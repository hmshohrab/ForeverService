# ForeverService - Android

In the world of Android development, there are instances where you might need to create a service
that runs indefinitely, even when the app is in the background or closed. This article aims to guide
you on how to create such a "Forever Service." We will explore the code snippet provided and
understand how it works.

## Service compatible Android 23-33 using Kotlin

### 1- Define a class for your service by extending the Service class.

### This class will handle the service logic.

### 2- Implement the necessary methods in your service class -onStartCommand()-Called when the service is started.

### on Bind!) - Called when a component wants to bind to the service jep det end bindings.

### - onDestroy() -Called when the service is stopped or destroyed.

### 3- In the onStartCommand() method, you can check if the service is running as a normal background service or in the foreground:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android
    package="com.example.yourservice">
    <uses-permission android:name="android.permission.FOREGROUND SERVICE" />
    <application>
        <service
            android:name=".YourService"
            android:enabled="true"
            android:exported="false" />
    </application>
</manifest>
```

```kotlin
override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
    val action = intent?.action

    if (action == ACTION_STOP_SERVICE) {
        Log.i(TAG_DEBUG , "ACTION_STOP_SERVICE time = ${Date()}")

        // The service was stopped by the user
        stopSelf()
        return START_NOT_STICKY
    }

    // Check if the service is running as a foreground service or a normal background service
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
        // Running as a foreground service
        initNotification()
        Log.i(TAG_DEBUG , "initNotification time = ${Date()}")

    } else {
        // Running as a normal background service
        // You can do any background work here
        Toast.makeText(this, "Service started", Toast.LENGTH_SHORT).show()
    }

    return START_STICKY
}
```
## To start the service, you can use the following code:
```kotlin
val intent - Intent(context, YourService::class.java)
context.startService(intent)
```
## To stop the service when the user clicks a button, you can send an intent with the ACTION_STOP_SERVICE action:
```kotlin
val stopintent - Intent(context, YourService::class.java)
stopintent.action = YourService.ACTION_STOP_SERVICE
context.startService(stopintent)
```

### Now, to handle the case where the service is removed by garbage collector and you want it to start again,
### you'll need to use a mechanism to restart the service when it's terminated. One approach is to use a BroadcastReceiver that listens for the BOOT COMPLETED action,
### which is sent when the device starts up or garbage collector restarts. Then, you can start your service from the onReceive() method of the receiver

## To implement this, follow these steps:

### 1- Create a class for your BroadcastReceiver:
```kotlin
class RestartServiceReceiver : BroadcastReceiver(){

    override fun onReceive(context: Context?, intent: Intent?) {
        if (intent?.action == Intent.ACTION_BOOT_COMPLETED) {
            // Start your service when the device boots up
            val serviceIntent = Intent(context, ForeverService::class.java)
            context?.startService(serviceIntent)
        }
    }
}
```

## To implement this, follow these steps:

### 2- Register the receiver in your AndroidManifest.xml file:
```kotlin
<receiver
    android:name=". RestartServiceReceiver" android:enabled="true"
    android:exported="true">
        <intent-filter>
            <action android:name="android.intent.action.BOOT COMPLETED" />
        </intent-filter>
</receiver>
```
## Handle service restart using system callbacks:
### Override the onTaskRemoved) and onStartCommand() methods in your service to handle situations where the service is terminated and needs to be restarted.

```kotlin
override fun onTaskRemoved(rootIntent: Intent) {
    val restartServiceIntent = Intent(applicationContext, ForeverService::class.java).also {
        it.setPackage(packageName)
    }
    val restartServicePendingIntent: PendingIntent = PendingIntent.getService(this, 1, restartServiceIntent, PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE )
    applicationContext.getSystemService(Context.ALARM_SERVICE)
    val alarmService: AlarmManager = applicationContext.getSystemService(Context.ALARM_SERVICE) as AlarmManager
    alarmService.set(AlarmManager.ELAPSED_REALTIME, SystemClock.elapsedRealtime() + 1000, restartServicePendingIntent)
    Log.i(TAG_DEBUG , "onTaskRemoved time = ${Date()}")

}
```