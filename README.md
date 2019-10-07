
Engage Digital Messaging - Android
==========

Engage Digital provides a mobile messaging component that allows users of your app to easily communicate with your customer support agents. You can send text messages
and receive push notifications and automatic server-provided replies.

The component integrates nicely in any Android phone or tablet, allows presenting Engage Digital Messaging through Fragment or Activity and has rich customization options to fit
perfectly in your application.

For more information about Engage Digital Messaging, please see [Engage Digital Messaging reference](http://mobile-messaging.dimelo.com)


API Reference
-------------

Please refer to [Engage Digital Messaging SDK Android API Reference](https://rawgit.com/ringcentral/engage-digital-messaging-android/master/JavaDoc/index.html) for advanced use.

Supported Versions
------------------

The Engage Digital Messaging SDK Android is currently supporting **Android 4.0.3 (API 15) and above**.


Getting Started
---------------

Follow these three **mandatory** steps to integrate the Engage Digital Messaging in your application:

1. [Install the Dimelo library using Gradle](#how-to-install-with-gradle-build-system-using-android-studio).
2. [Choose your authentication method and initialize the SDK](#authentication-and-sdk-initialization).
3. [Display Engage Digital Messaging in your application](#displaying-the-mobile-messaging).

#### **Note: Specify requirement for Apache HTTP Legacy library for dimelo version < 1.7.1**
If your app is targeting API level 28 (Android 9.0) or above, you must include the following declaration within the ```<application>``` element of ```AndroidManifest.xml```.

```xml
<uses-library
      android:name="org.apache.http.legacy"
      android:required="false" />
```

\
These are minimal steps to make Engage Digital Messaging work in your app.\
Read on how to customize the appearance and the behavior of Engage Digital Messaging to perfectly fit in your app:
- [Customize Engage Digital Messaging appearance](#customizing-mobile-messaging-appearance)
- [Add push notifications support](#push-notifications)
- [Enable message location using the Google APIs](#activate-location-messages)
- [Add a Dimelo listener to react to SDK triggered events](#reacting-to-mobile-messaging-events)

You can see an example of Engage Digital Messaging implementation by downloading the [Sample App](https://github.com/ringcentral-tutorials/engage-digital-messaging-android-demo).


How To Install With Gradle build system (Using Android Studio)
-----------------------------

Add these to your Grade file:

#### repositories
```
  repositories {
      maven {
                url "https://raw.github.com/ringcentral/engage-digital-messaging-android/master"
    }
  }
```

#### dependencies
```
  dependencies {
        compile 'com.dimelo.dimelosdk:dimelosdk:1.7.+'
  }
```


Migration to Dimelo 1.7.0
-----------------------------
Dimelo 1.7.0 uses a new mandatory domain name setting (first part of your Dimelo Digital URL: **domain-name**.engagement.dimelo.com), so these changes **must** be taken into consideration:
* `setApiSecret(String apiSecret)` is now deprecated in favor of `initWithApiSecret(String secret, String domainName, DimeloListener dimeloListener)`.
* `setApiKey(String apiKey)` is now deprecated in favor of `initWithApiKey(String apiKey, String domainName, DimeloListener dimeloListener)`.
* `setHostname(String  hostname)` is not available anymore.


Authentication and SDK initialization
--------------

With each HTTP request, Dimelo sends a JWT ([JSON Web Token](http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html)).
This token contains user-specific info that you specify (`userIdentifier`, `userName` etc.) and a HMAC signature. User identifier allows Dimelo to identify author of messages from different in the agent's backend.
If user identifier is missing (`null`), then an autogenerated unique installation identifier is used to identify the user (created automatically).

If your app rely on a uniq imutable technical identifier to identify user use `userIdentifier` to also identify them at the Agent interface level.

Use `userName` to provide to agent a human name for the user.

We support two kinds of authentication modes: with **built-in secret** and with a **server-side secret**.

#### 1. Setup with a built-in secret

This is a convenient mode for testing and secure enough when user identifiers are unpredictable.

Here's how to create the Dimelo instance and initialize it using a built-in secret:
```java
Dimelo dimelo = Dimelo.setup(Context);
dimelo.initWithApiSecret(SOURCE_API_SECRET, DIMELO_DOMAIN_NAME, DIMELO_LISTENER);
/*
  SOURCE_API_SECRET can be found in your source configuration
  DIMELO_DOMAIN_NAME is your domain name (e.g. DIMELO_DOMAIN_NAME.engagement.dimelo.com)
  DIMELO_LISTENER is an optionnal parameter that we will cover later in this document
*/
```

Then it will create and sign JWT automatically when needed (as if it was provided by the server).

You simply set necessary user-identifying information and JWT will be computed on the fly.
You do not need any cooperation with your server in this setup.

The security issue here is that built-in secret is rather easy to extract from the app's binary build.
Anyone could then sign JWTs with arbitrary user identifying info to access other users'
chats and impersonate them. To mitigate that risk make sure to use this mode
only during development, or ensure that user identifiers are not predictable (e.g. random UUIDs).

#### 2. Setup with a server-side secret (better security but more complicated)

This is a more secure mode. Dimelo will provide you with two keys: a public API key and a secret key.
The public one will be used to configure `Dimelo` instance and identify your app.
The secret key will be stored on your server and used to sign JWT token on your server.

Here's how create the Dimelo instance and initialize it using a server-side secret:
```java
Dimelo dimelo = Dimelo.setup(Context);
dimelo.initWithApiKey(SOURCE_API_KEY, DIMELO_DOMAIN_NAME, DIMELO_LISTENER);
/*
  SOURCE_API_KEY can be found in your source configuration
  DIMELO_DOMAIN_NAME is your domain name (e.g. DIMELO_DOMAIN_NAME.engagement.dimelo.com)
  DIMELO_LISTENER is an optionnal parameter that we will cover later in this document
*/
```

Once this is done you will have to set `jwt` property manually with a value received from your server.
This way your server will prevent one user from impersonating another.

1. Set authentication-related properties (`userIdentifier`, `userName`, `authenticationInfo`).
2. Get a dictionary for JWT using `Dimelo.getJwtDictionary()`. This will also contain public API key, `installationIdentifier` etc.
3. Send this dictionary to your server.
4. Check the authenticity of the JWT dictionary on the server and compute a valid signed JWT token.
Use a corresponding secret API key to make a HMAC-SHA256 signature.
*Note:* use raw binary value of the secret key (decoded from hex) to make a
signature. Using hex string as-is will yield incorrect signature.
5. Send the signed JWT string back to your app.
6. In the app, set the `Dimelo.jwt` property with a received string.

/!\ `Dimelo.setUserIdentifier();`, `Dimelo.setAuthenticationInfo();` and `Dimelo.setUserName();` must only be called **BEFORE** `Dimelo.setJwt();` otherwise your JWT will be emptied and your request will end up in a 404 error.

You have to do this authentication only once per user identifier,
but before you try to use Engage Digital Messaging. Your application should prevent
user from opening Engage Digital Messaging until you receive a JWT token.


Displaying Engage Digital Messaging
-------------------

Dimelo provides different ways to display Engage Digital Messaging.

#### As an Activity:

Achieved by calling `Dimelo.openChatActivity()` (wich will internally call `Context.startActivity`).
This method will display a full screen Engage Digital Messaging with a Toolbar containing a title.
The title and the background (drawable or color) of the Toolbar are customizable.
The Navigation Icon can be displayed and customized.
The user can close Engage Digital Messaging (the Activity) by pressing the Navigation Icon or the back button of his device.

By default, the app name is used for the title and the primaryColor (appCompat) is used as the background color of the toolbar.

To make it work, you **must** declare the Activity in your `AndroidManifest.xml` with a name equals to `com.dimelo.dimelosdk.main.ChatActivity`
This is the easiest way to display Engage Digital Messaging.

If your application doesn't inherit from `AppCompat`, the toolbar and status bar of `Dimelo` will be black.
In order to fix this, your application needs to set the `ChatActivity` theme to `@style/DimeloTheme` (`AndroidManifest.xml`).
By default, DimeloTheme will set the Toolbar and status bar to blue.

You can change those by overriding `dimelo_color_primary` and `dimelo_color_primary_dark` in color file resources (`dimelo_color_primary` for toolbar and `dimelo_color_primary_dark` for status bar).

#### As a Fragment:

Your app must use an `AppCompat` theme to be able to use `Dimelo` as a `Fragment`.
Achieved by calling `Dimelo.newChatFragment()` and using the Android `FragmentManager` and `FragmentTransaction`.
This is the most flexible way to display Engage Digital Messaging as you can manually place, open and close it like any Fragment.
No Toolbar is displayed.
##### Note: forwarding onBackPressed() events using "Chat.onBackPressed()" is necessary to display the best user experience; "true" will be returned if the event has been consumed


#### As a Nested Fragment:
Engage Digital Messaging support fragment nesting (i.e: Engage Digital Messaging fragment can be displayed inside a fragment parent).
However, in this specific case, the host application needs to care about the following steps.

1. Using Fragment.setUserVisibleHint in order to notify Engage Digital Messaging about its visibility state.
This will prevent Engage Digital Messaging to do background work when unnecessary

2. Engage Digital Messaging uses intents for displaying attachments. Forwarding "onActivityResult" will allow Engage Digital Messaging to correctly receive the results.

3. Engage Digital Messaging is compiled against Android sdk 23 and handle dynamic permissions.
To allow optimal behaviors, forwarding "onRequestPermissionsResult" is necessary.

##### Note: forwarding onBackPressed() events using "Chat.onBackPressed()" is necessary to display the best user experience; "true" will be returned if the event has been consumed

Examples are provided within the SampleApp


Customizing Engage Digital Messaging Appearance
---------------------------

[See how to customize Engage Digital Messaging using the Android Resource folders](Customization.md).

You can also customize it programmatically:
#### As an Activity:
1) Implementing `Dimelo.OnActivitySetupAppearanceListener(Chat.Customization chatActivityCustomization)` and modifying `chatActivityCustomization` attributes.
2) Calling Dimelo.setChatCustomizationListener()

The ChatActivity will call the listener back when creating its layout.

You do not need to call customization.apply() as it will be called for you.

#### As a Fragment:
1) Calling `Chat.getCustomization()` and receiving an istance of `Chat.Customization`
2) Modifiying its attributes.
3) Calling customization.apply() to register the changes and update Engage Digital Messaging.

We provide a lot of properties for you to make Engage Digital Messaging look native to your application.

For your convenience, properties are organized in two groups: Basic and Advanced.
In many cases it would be enough to adjust the Basic properties only.

You can customize the inputbar color, the font and the color of any text in Engage Digital Messaging view.
If you are displaying Engage Digital Messaging as an activity you can also cutomize the ActionBar colors and title

Advanced options include background and padding for text bubbles.
We use 3 kinds of messages. Each kind can be customized independently.

1. User's text message.
3. Agent's text message.
3. System text notification (automatic reply from the server).

All bubble images must be 9-part sliced resizable images or a ([Drawable xml](http://developer.android.com/guide/topics/resources/drawable-resource.html)) to fit arbitrary content.

Text bubbles can be colored using properties `userMessageBackgroundColor`, `agentMessageBackgroundColor` etc.

If you provide a custom bubble image for text, you should also update
message bubble padding properties to arrange your text correctly within a bubble.

Check the [Engage Digital Messaging SDK Android API Reference](https://rawgit.com/ringcentral/engage-digital-messaging-android/master/JavaDoc/index.html) to learn about all customization options.


Push Notifications
------------------

Engage Digital Messaging can receive push notifications from Engage Digital Messaging server.
To make them work, a couple of steps must be done on your part:

### Using Google Cloud Messaging (GCM):
1. Register to Google GCM service by using for example `GoogleCloudMessaging.register(senderId)`
2. Set `Dimelo.deviceToken` property with the value returned by the `GoogleCloudMessaging.register(senderId)`
3. Your app must register for remote notifications for example by declaring and implementing a `Receiver` and a `Service` (Android APIs).
4. Optionally implement `Dimelo.BasicNotificationDisplayer` abstract class.
   It allows you to specify a title, an icone and how to display Engage Digital Messaging when the user click the notification.
   If you want to handle the entire process of displaying notifications you can directly implement `Dimelo.NotificationDisplayer` interface.
5. Let Dimelo consume the notification using `Dimelo.consumeReceivedRemoteNotification()`.
   If this method returns `true`, it means that Dimelo recognized the notification as its own and you should not
   process the notification yourself. Engage Digital Messaging will be updated automatically with a new message.
6. If your Android version is at least Android N, you'll receive an interactive push notification with direct reply. To disable this, use `Dimelo.interactiveNotification = false;`.

If the notification is received while the app is running, the sdk will display the notification only if Engage Digital Messaging is not visible by the user
You can override the behavior by implementing `dimeloShouldDisplayNotificationWithText` from the listener `DimeloListener`

Prior to Android 5, the notification will be displayed as a Ticker (one line scrolling notification) and is not clickable.

Starting from Android 5, the notification will be displayed as a ([Heads-up](http://developer.android.com/guide/topics/ui/notifiers/notifications.html#Heads-up)).
Clicking of the Heads-up will, by default, open the application.
If you specify the ([parent activity using the support meta-data tag](http://developer.android.com/training/implementing-navigation/ancestral.html)), clicking the Heads-up will open Engage Digital Messaging and provide the up-navigation.

If Engage Digital Messaging is opened directly upon clicking the notification, then the `IntentService` managing the notification must ensure to properly configure the `Dimelo` instance (at the minimum, calling `Dimelo.setup` and `dimelo.setApiSecret`).

If you'd like to have the full control on the notification (appearance and behavior on click) you can implement the `Dimelo.NotificationDisplayer` interface.

<br>

### Using [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging/android/client) (FCM):
*Note:* This is an example on how to initialize the Dimelo instance:
```java
public class MainActivity extends AppCompatActivity {

  @Override
  public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.your_layout);
    // Setup Dimelo
    Dimelo dimelo = setupDimelo(this);
  }

  public static Dimelo setupDimelo(Context context) {
        Dimelo.setup(context);
        Dimelo dimelo = Dimelo.getInstance();
        dimelo.initWithApiSecret(SOURCE_API_SECRET, DIMELO_DOMAIN_NAME, DIMELO_LISTENER);
        dimelo.setUserName("USER_NAME");
        dimelo.setUserIdentifier("USER_IDENTIFIER");
        JSONObject authInfo = new JSONObject();
        try {
            authInfo.put("CustomerId", "0123456789");
            authInfo.put("Dimelo", "Rocks!");
        } catch (JSONException e) {
        }
        dimelo.setAuthenticationInfo(authInfo);
        return dimelo;
  }

  // ...
}
```
1. Download the `google-services.json` file from your [firebase console](https://support.google.com/firebase/answer/7015592) and copy it in your project's `/app` folder
2. Declare a `FirebaseMessagingService` [service](https://firebase.google.com/docs/reference/android/com/google/firebase/messaging/FirebaseMessagingService) in your project's `AndroidManifest.xml`:
```xml
<service
    android:name=".MyFirebaseMessagingService">
    <intent-filter>
        <action android:name="com.google.firebase.MESSAGING_EVENT"/>
    </intent-filter>
</service>
```
3. Create a class that extends FirebaseMessagingService:
```java
public class MyFirebaseMessagingService extends FirebaseMessagingService
```
4. Retrieve the device token and pass it to your Dimelo instance by overriding the `onNewToken` method:
```java
@Override
public void onNewToken(String token) {
  if (Dimelo.isInstantiated())
    Dimelo.getInstance().setDeviceToken(token);
}
```
5.  Finally intercept the notification and pass it to your Dimelo instance by overring the `onMessageReceived` method:
```java
@Override
public void onMessageReceived(RemoteMessage remoteMessage) {
  // You have to configure the Dimelo instance before calling the Dimelo.consumeReceivedRemoteNotification() method.
  MainActivity.setupDimelo(MyFirebaseMessagingService.this);
  if (Dimelo.consumeReceivedRemoteNotification(MyFirebaseMessagingService.this, remoteMessage.getData(), null)){
    // The notification will be handled by the Dimelo instance
  }
  else {
    // It is not a Dimelo notification.
  }
}
```

*Note:* You can also retrieve the current device token by calling `FirebaseInstanceId.getInstance().getInstanceId()`. Here is an example on how to retrieve the device token and pass it to the Dimelo instance (as [described here](https://firebase.google.com/docs/cloud-messaging/android/client#retrieve-the-current-registration-token)):
```java
FirebaseInstanceId.getInstance().getInstanceId()
        .addOnCompleteListener(new OnCompleteListener<InstanceIdResult>() {
            @Override
            public void onComplete(@NonNull Task<InstanceIdResult> task) {
                if (!task.isSuccessful()) {
                    Log.w(TAG, "getInstanceId failed", task.getException());
                    return;
                }

                // Get new Instance ID token
                String token = task.getResult().getToken();
                Dimelo.getInstance().setDeviceToken(token);
            }
        });
```

Activate location messages
-----------------------------
Activating location messages allow users to send a map containing their location.

**Note**: This feature uses [Google Places API](https://developers.google.com/places/android-api/)  which is by default limited to 1000 requests per day. you can increase this limitation to 150 000 requests per day ([Enable billing to get 150 000 requests per 24 hour period](https://developers.google.com/places/android-api/usage#enable-billing))


1. Connect to [Google API console](https://console.developers.google.com/)
2. Access to API Manager pannel
3. Search for **Google Maps Android API** and **Google Places API for Android**
4. Enable both APIs
5. Access to Credentials pannel and create an API key for Android
6. Add your API key to your application manifest
```
<application
  <meta-data
  android:name="com.google.android.geo.API_KEY"
  android:value="YOUR_API_KEY">
</application>
```


Reacting To Engage Digital Messaging Events
-----------------------

You can react to various events in Engage Digital Messaging by implementing a `DimeloListener`.

Two particular events that might be interesting to you are `dimeloDidBeginNetworkActivity()` and `dimeloDidEndNetworkActivity()`.

Use `-onOpen:` and `-onClose:` events to get informations using `dimelo` parameter when Engage Digital Messaging view is just opened or closed.

Please refer to [Engage Digital Messaging SDK Android API Reference](https://rawgit.com/ringcentral/engage-digital-messaging-android/master/JavaDoc/index.html) documentation for more information.
