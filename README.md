# hackdays-32-web-push
The Push API enables web apps to receive notifications pushed from a server and will display those notifications even if the web app is in the background. It is supported by Chrome and Firefox, and Apple is sending strong signals that it will support it in 2022. This powerful feature will help progressive web apps drive engagement by letting app owners push messages directly onto the notification list of their users' devices.

To leverage this communication channel, this project will stand up a platform that will allow web apps to enqueue notifications via a custom SDK to Google Cloud Pub/Sub (or some other GCP altenative?) for processing. Once processed, the message will be forwarded to the appropriate browser manufacturer's notification platform which will then deliver it to the subscribed browser.

Note on Macs, you may have to explicitly allow Chrome notifications by setting a permission at System Preferences > Notifications.

## Requirements

https://github.com/users/chrisahardie/projects/1/views/1

## Issues list

https://github.com/chrisahardie/hackdays-32-web-push/issues

## Solution components

### Web application

**Landing page**

* This page lets the user subscribe to notifications via a service worker. Once subscribed, the web app's landing page can be backgrounded, but it will still accept push notifications.

The subscription event will publish subscription details to the API for storage.  The subscription details will include:

* first name
* last name
* [endpoint](https://developer.mozilla.org/en-US/docs/Web/API/PushSubscription/endpoint)
* [auth](https://web.dev/push-notifications-web-push-protocol/#inputs)
* [p256dh](https://web.dev/push-notifications-web-push-protocol/#inputs)

* This page lets the user unsubscribe to notifications via its service worker. This will also entail consuming a delete operation to remove the subscription from the database.

**Admin page**

* This page lets an admin configure [push notification settings](https://developer.mozilla.org/en-US/docs/Web/API/Notification/actions):
    * actions
    * badge
    * body
    * data
    * dir
    * icon
    * image
    * lang
    * requireInteraction
    * silent
    * tag
    * title
    * vibrate

* This page lets an admin send a push notification to all users
* This page lets an admin send push notifications to a single user

### Web Api

* Supplies a [VAPID](https://blog.mozilla.org/services/2016/04/04/using-vapid-with-webpush/) public key
* Accepts subscription details from the web app for storage in a database
* Enqueues messages to the platform via SDK
* Returns a list of all users
* Manages a VAPID private key and public key in a config

### Database

* Very simple schema, a single table should suffice
    * id
    * first name
    * last name
    * endpoint
    * auth
    * p256dh
    * created
    * lastUsed

### SDK

* Abstracts the cloud integration from the client's web app
* Enqueues messages to Google Cloud Pub/Sub
* Generates vapid keys

### Google Cloud Pub/Sub

* Durable queue used to let service components communicate asynchronously

### Serverless function

* Reads messages off Google Cloud Pub/Sub, and dispatches them to notification endpoint. (`web-push` is a good NPM library for this if this is a Node function)
