# react-native-pjsip

## Updated by Aniruddh Kukadiya

 - Registerd service to add below code in AndroidManifest.xml file 

```
    <service
        android:name="com.carusto.ReactNativePjSip.PjSipService"
        android:enabled="true"
        android:exported="true" />
```

Update audio changes after Android 10(Android Q) in PJSIP service file

 - When call on make in speaker
```
mAudioManager.setMode(AudioManager.MODE_IN_COMMUNICATION);
```
 - When call on make in earpiece
 ```
 mAudioManager.setSpeakerphoneOn(false);
 ```
 
Added local notification while call because of app wakeup while after Android 8 (Android Oreo) in PJSIP service file
 
  - Notifiaction description need to pass in parameter
 
 ```
  public void createNotification(String destination) {
        if (Build.VERSION.SDK_INT > Build.VERSION_CODES.O) {
            Log.e(TAG, "createNotification: ");
            String NOTIFICATION_CHANNEL_ID = "calling_pjsip";
            String channelName = "Background Service";
            NotificationChannel chan = new NotificationChannel(NOTIFICATION_CHANNEL_ID, channelName, NotificationManager.IMPORTANCE_DEFAULT);
            chan.setLightColor(Color.BLUE);
            chan.setLockscreenVisibility(Notification.VISIBILITY_PRIVATE);

            NotificationManager manager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
            assert manager != null;
            manager.createNotificationChannel(chan);

            NotificationCompat.Builder notificationBuilder = new NotificationCompat.Builder(this, NOTIFICATION_CHANNEL_ID);
            Notification notification = notificationBuilder.setOngoing(true)
                    .setContentTitle("Calling...")
                    .setContentText(destination)
                    .setPriority(NotificationManager.IMPORTANCE_DEFAULT)
                    .setCategory(Notification.CATEGORY_CALL)
                    .setSmallIcon(android.R.drawable.sym_call_outgoing)
                    .build();

            startForeground(2, notification);
        } else {
            startForeground(1, new Notification());
        }
    }
 ```
 
  - Call above funtintion when call start and update with you app name

```
createNotification("Message show in notification");
```



A [PJSIP](http://www.pjsip.org/) module for React Native.

## Support

- Currently support for iOS and Android.
- Support video and audio communication.
- Ability to use Callkit and PushNotifications.
- You can use it to build an iOS/Android app that can communicate with SIP server.
- Android version is based on [react-native-pjsip-builder](https://github.com/datso/react-native-pjsip-builder)
- iOS version is based on [Vialer-pjsip-iOS](https://github.com/VoIPGRID/Vialer-pjsip-iOS)

## Installation

- [iOS](https://github.com/datso/react-native-pjsip/blob/master/docs/installation_ios.md)
- [Android](https://github.com/datso/react-native-pjsip/blob/master/docs/installation_android.md)

## Usage

First of all you have to initialize module to be able to work with it.

There are some interesting moment in initialization.
When application goes to background, PJSIP module is still working and able to receive calls, but your javascipt is totally suspended.
When User open your application, javascript start to work and now your js application need to know what status have your account or may be you have pending incoming call.

So thats why first step should call start method for pjsip module.

```javascript
import {Endpoint} from 'react-native-pjsip';

let endpoint = new Endpoint();
let state = await endpoint.start(); // List of available accounts and calls when RN context is started, could not be empty because Background service is working on Android
let {accounts, calls, settings, connectivity} = state;

// Subscribe to endpoint events
endpoint.on('registration_changed', account => {});
endpoint.on('connectivity_changed', online => {});
endpoint.on('call_received', call => {});
endpoint.on('call_changed', call => {});
endpoint.on('call_terminated', call => {});
endpoint.on('call_screen_locked', call => {}); // Android only
```

Account creating is pretty strainghforward.

```javascript
let configuration = {
  "name": "John",
  "username": "sip_username",
  "domain": "pbx.carusto.com",
  "password": "****",
  "proxy": null,
  "transport": null, // Default TCP
  "regServer": null, // Default wildcard
  "regTimeout": null // Default 3600
  "regHeaders": {
    "X-Custom-Header": "Value"
  },
  "regContactParams": ";unique-device-token-id=XXXXXXXXX"
};
endpoint.createAccount().then((account) => {

});

```

To be able to make a call first of all you should createAccount, and pass account instance into Endpoint.makeCall function.
This function will return a promise that will be resolved when PjSIP initializes the call.

```javascript
let options = {
  headers: {
    "P-Assserted-Identity": "Header example",
    "X-UA": "React native"
  }
}

let call = await endpoint.makeCall(account, destination, options);
call.getId() // Use this id to detect changes and make actions

endpoint.addListener("call_changed", (newCall) => {
  if (call.getId() === newCall.getId()) {
     // Our call changed, do smth.
  }
}
endpoint.addListener("call_terminated", (newCall) => {
  if (call.getId() === newCall.getId()) {
     // Our call terminated
  }
}
```

