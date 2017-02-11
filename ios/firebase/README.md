# Firebase: Best Practices and Key Learnings

The purpose of this document is to capture some of our thinking around the best practices and key take aways of using Google Firebase.

```swift
```


### Requesting Remote Resources

#### Usages

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

### Updating Remote Resources
