# Firebase: Remote and Local Persistence

Firebase has two primary features for storing data in the cloud. The [Firebase Realtime Database] (https://firebase.google.com/docs/database/ios/start) is a cloud-hosted database in which the data is stored as JSON and synchronized in realtime to every connected client. [Firebase Storage] (https://firebase.google.com/docs/storage/ios/start) service is a convenient wrapper around a Google Cloud Storage bucket that integrates nicely with the Realtime Database.

### There is no way to know if a piece of data has been synced to the Realtime Database
### With unreliable connectivity, Realtime Database callbacks are not guaranteed to be executed and do not time out
### Update requests are persisted, Transaction blocks are not
### File Storage requests function more like traditional REST web-services
