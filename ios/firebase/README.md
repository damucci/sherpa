# Firebase: Best Practices and Key Learnings

The purpose of this document is to capture some of our thinking around the best practices and key take aways of using Google Firebase.

## Realtime Database

### Data Modeling

### Requesting Remote Resources

Requesting a single resource similar to a typical REST api request

```swift
let ref = FIRDatabase.database().reference().child("user_profile").child(userIdentifier)
ref.observeSingleEvent(of: .value, with: { snapshot in
    // async callback logic
})
```

Creating an observer callback that will be called for the initial value at the destination node, if one exists, as well as whenever data below that node changes in any way.

```swift
let ref = FIRDatabase.database().reference().child("user_profile").child(userIdentifier)
let handle = ref.observe(.value, with: { snapshot in
    // async listener logic
})
```

#### Gotchas

* These callbacks never time out, even in the case of the REST like observeSingleEvent
* Observation callbacks will continue to be triggered even if the associated FIRDatabaseReference is discarded
* Observation handles need to be removed from a FIRDatabaseReference with the exact path on which they were created

### Querying

Firebase provides a series of query methods that allow you to [filter, order, and limit] (https://firebase.google.com/docs/database/ios/lists-of-data#sorting_and_filtering_data) the data that is retrieved.

For example, in a blogging app, if you wanted to get retrieve the last 5 blog posts about cooking, you would use a query like this:

```swift
let ref = FIRDatabase.database().reference().child("blog_posts")
let query = ref.queryOrdered(byChild: "post_date")
               .queryEqual(toValue: "cooking", childKey: "primary_category")
               .queryLimited(toLast: 5)

query.observeSingleEvent(of: .value, with: { snapshot in
    // async callback logic
})
```

Paging can be accomplished by using the data from the previous page to limit the results of the next page.

```swift
// Request initial page
let ref = FIRDatabase.database().reference().child("blog_posts")
let query1 = ref.queryOrdered(byChild: "post_date")
                .queryEnding(atValue: Date().timeIntervalSince1970, childKey: "post_date")
                .queryLimited(toLast: 5)

// Request next page
let first = ... // first entry from previous page
let ref = FIRDatabase.database().reference().child("blog_posts")
let query1 = ref.queryOrdered(byChild: "post_date")
                .queryEnding(atValue: first.postDate.timeIntervalSince1970, childKey: "post_date")
                .queryLimited(toLast: 5)

```

#### Gotchas

* You can combine a single `StartingAt` and single `EndingAt` filter, or you can use a single `EqualTo` filter. Any other attempt at combining filters will result in a runtime exception. If you need to filter any further, you'll need to bake the desired filtering into the structure of the data. For example, if you wanted to retrieve the last 5 cooking blog posts that only a specific user posted, you would have to group posts under user-specific nodes.

```swift
let ref = FIRDatabase.database().reference().child("blog_posts").child(userIdentifier)
let query = ref.queryOrdered(byChild: "post_date")
               .queryEqual(toValue: "cooking", childKey: "primary_category")
               .queryLimited(toLast: 5)
```

* You can only use a single `OrderBy` modifier or else you will get a runtime exception. Any additional ordering will need to be done locally.
* You can only use a single `LimitedTo` modifier or else you will get a runtime exception, though, its difficult to imagine when you would want to.
* TODO: Querying/Paging breaks with database persistence enabled

### Updating Remote Resources

```swift
let ref = FIRDatabase.database().reference().child("characters")
ref.updateChildValues([
    "Luke" : [
        "homeworld": "Tatooine",
        "species" : "human"
    ]
], withCompletionBlock: { (error, reference) in
    // Async update complete
})
```

```swift
let ref = FIRDatabase.database().reference()
ref.updateChildValues([
    "characters/Luke/groups/rebels"  : true ,
    "groups/rebels/members/Luke": true
], withCompletionBlock: { (error, reference) in
    // Async update complete
})
```

```swift
let ref = FIRDatabase.database().reference().child("affiliations")
ref.runTransactionBlock({ data -> FIRTransactionResult in
    guard let value = data.value as? NSMutableDictionary else { return .success(withValue: data) }

    if let post = posts["rebels"] as? NSMutableDictionary {
        if let likes = post["likes"] as? Int {
            post["likes"] = likes + 1
        } else {
            post["likes"] = 1
        }
    }
    data.value = value
    return .success(withValue: data)
}, andCompletionBlock: { (error, success, reference) in
    // Async update complete
})
```

#### Gotchas

* Neither `updateChildValues` nor `runTransactionBlock` requests timeout when the device has no network connectivity. Instead, they update the local store immediately, triggering observer callbacks. Contrastingly, update completion callbacks do not get triggered until their request updates the server.
* When database persistence is enabled, `updateChildValues` requests get cached across app launches, without their completion blocks, while waiting for network connectivity. Conversely, `runTransactionBlock` requests do not persist, resulting in Firebase advising you not to use them when database persistence is enabled.

### Database Persistence

## File Storage
## Push Notifications
## Analytics
## Pricing
