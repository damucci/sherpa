# Firebase: Best Practices and Key Learnings

The purpose of this document is to capture some of our thinking around the best practices and key take aways of using Google Firebase.


## High Level Topics:

- [ ] [Networking and Local Persistence] (remote_and_local_persistence.md)
- [ ] Push Notifications
- [ ] Paging
- [ ] Testing


## Additional Learnings

### Billing

Firebase offers [three different billing plans] (https://firebase.google.com/pricing/), each with different limits on connections, storage, backups, etc:
* SPARK - a free option with fairly reasonable limits for a development team
* FLAME - a $25 per month option the might be enough for a public/private Beta
* BLAZE - a pay as you go option that enables you to scale

With all of these options in mind, we would generally recommend that going straight for the BLAZE, pay-as-you-go option, since it is the only one that ensures that your apps access will never be turned off during heavy usages. For example, suppose while your app is out in production, it gains organic growth and unexpectedly sees a large surge of traffic. If this surge is large enough to hit your billing limits, your app's access to the offending Firebase features will be revoked for all users until the end of the billing cycle, likely having a severely damaging effect on your app's growth. As another example, suppose that while you're developing a feature that will allow the user to upload and download photos, your team starts testing this feature before you start limiting the size of photos or downscaling them prior to uploading. In this case, the team would likely hit the *GB downloaded* limit and then be effectively blocked from working on that feature until the billing period ended. 
