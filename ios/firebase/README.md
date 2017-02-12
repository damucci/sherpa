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

Firebase provides a series of query methods that allow you to filter, order, and limit the data that is retrieved.

```swift
func queryStarting(atValue startValue: Any?) -> FIRDatabaseQuery
func queryStarting(atValue startValue: Any?, childKey: String?) -> FIRDatabaseQuery
func queryEnding(atValue endValue: Any?) -> FIRDatabaseQuery
func queryEnding(atValue endValue: Any?, childKey: String?) -> FIRDatabaseQuery
func queryEqual(toValue value: Any?) -> FIRDatabaseQuery
func queryEqual(toValue value: Any?, childKey: String?) -> FIRDatabaseQuery

func queryOrdered(byChild key: String) -> FIRDatabaseQuery
func queryOrderedByKey() -> FIRDatabaseQuery
func queryOrderedByValue() -> FIRDatabaseQuery
func queryOrderedByPriority() -> FIRDatabaseQuery

func queryLimited(toFirst limit: UInt) -> FIRDatabaseQuery
func queryLimited(toLast limit: UInt) -> FIRDatabaseQuery
```

For example, in a blogging app, if I wanted to get retrieve the last 5 blog posts about cooking, I would use a query like this:

```swift
let ref = FIRDatabase.database().reference().child("blog_posts")
let query = ref.queryOrdered(byChild: "post_date")
               .queryEqual(toValue: "cooking", childKey: "primary_category")
               .queryLimited(toLast: 5)

query.observeSingleEvent(of: .value, with: { snapshot in
    print("Results: ", snapshot)
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

* You can combine a single `StartingAt` and single `EndingAt` filter, or you can use a single `EqualTo` filter. Any other attempt at combining filters will result in a runtime exception. If you need to filter any further, you'll need to bake the desired filtering into the structure of the data. For example, if I wanted to retrieve the last 5 cooking blog posts that only a specific user posed, I would group posts under user specific nodes.

```swift
let ref = FIRDatabase.database().reference().child("blog_posts").child(userIdentifier)
let query = ref.queryOrdered(byChild: "post_date")
               .queryEqual(toValue: "cooking", childKey: "primary_category")
               .queryLimited(toLast: 5)
```

* You can only use a single `OrderBy` modifier or else you will get a runtime exception. Any additional ordering will need to be done locally.
* You can only use a single `LimitedTo` modifier or else you will get a runtime exception, though, its difficult to imagine when you would want to.

### Updating Remote Resources

```swift
```

### Database Persistence

## File Storage
## Push Notifications
## Analytics
## Pricing
