Folder structure:
```
notification/  
	NotificationManager
		- Build + show notification
		- Create PendingIntent for notification taps
		- Handle deeplink payload setup
	NotificationChannelManager: optional for different notification topics
		- create channels 
	NotificationWorker
		- scheduled execution
		- retry: WorkManager
		- delayed notification
	NotificationData
		- notification models
	NotificationReceiver: optional for Handle notification ACTION BUTTON clicks
	
```




Require permission(Android 13+)
#### Step 2 — Request runtime permission

From AndroidManifest:
\<uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>

```
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
		ActivityCompat.requestPermissions(
		this,
		arrayOf(Manifest.permission.POST_NOTIFICATIONS),1001
)}
```

#### Step 3: Create Notification Channel
Channels allow users to control notification behavior:  
- sound  
- vibration  
- importance  
- lock screen visibility
**Important**: Channel **importance** affects heads-up behavior
```
val notificationManager =  
    context.getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager  
  
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {  
    val channel = NotificationChannel(  
        CHANNEL_ID,  
        CHANNEL_NAME,  
        NotificationManager.IMPORTANCE_DEFAULT  
    )  
    notificationManager.createNotificationChannel(channel)  
}
```

#### Step 4: Build Notification
setSmallIcon()  
- required  
- notification will not display properly without it  
  
setPriority()  
- controls notification interruption level  
- mainly for pre-Android 8 behavior  
- channel importance is more important on Android 8+  
  
setContentIntent()  
- action when user taps notification  
- usually launches Activity through PendingIntent  
  
setAutoCancel(true)  
- removes notification after user taps it

Example:
```
val notification = NotificationCompat.Builder(  
	context,  
	NotificationChannelManager.TRADE_CHANNEL_ID  
)  
	.setSmallIcon(R.drawable.ic_launcher_foreground)  
	.setContentTitle(data.title)  
	.setContentText(data.message)  
	.setPriority(NotificationCompat.PRIORITY_HIGH)  
	.setContentIntent(pendingIntent)  
	.setAutoCancel(true)  
	.build()
```

#### Step 5:  Show Notification
notificationId:  
- same ID → updates existing notification  
- different ID → creates new notification
```
NotificationManagerCompat.from(context)  
	.notify(notificationId, notification)
```

#### NotificationWorker
WorkManager preferred for:
- delayed notifications
-  retry handling
-  guaranteed background execution
CoroutineWorker: runs using Coroutines, managed by WorkManager
doWork(): the task that WorkManager will execute
```
class NotificationWorker(  
	context: Context,  
	params: WorkerParameters  
	) : CoroutineWorker(context, params) {  
	  
		override suspend fun doWork(): Result {  
		  
		return try {  
		  
			val title = inputData.getString("title") ?: return Result.failure()  
			val message = inputData.getString("message") ?: return Result.failure()  
			  
			val notificationData = NotificationData(  
			id = System.currentTimeMillis().toInt(),  
			title = title,  
			message = message  
		)  
		  
		AppNotificationManager(applicationContext)  
		.showNotification(notificationData)  
		  
		Result.success()  
		  
		} catch (e: Exception) {  
			// Retry automatically if needed  
			Result.retry()  
		}  
	}  
}
```

## That's all the setup, and what would happen for notification
## Now when should it happen:


# Local Notification


# Push Notification
Typically use Firebase Cloud Messaging

Step 1: Add dependency
Step 2: Register in manifest
Step 3: Create Service
```
class MyFirebaseMessagingService : FirebaseMessagingService() {  
  
	override fun onMessageReceived(message: RemoteMessage) {  
	  
		val title = message.notification?.title  
		val body = message.notification?.body  
		  
		showNotification(title, body)  
	}  
}
```

