# Old Android

### Activity  
entry point/lifecycle container for a screen/application interaction.
└── Fragment  
└── XML Layout
### Fragments 
A **Fragment** is a **reusable portion of UI inside an Activity**.
It cannot exist alone — it **must live inside an Activity**.

### Service
A **Service** performs **background work without a UI**.
Modern Replacement: **WorkManager, Foreground Service, Coroutines**
### JobScheduler
`JobScheduler` schedules **background tasks** to run later.
Example:
Run task when:  
- device is charging  
- device has WiFi  
- device is idle
Example usage: Sync data every few hours , Upload logs when network available

Modern replacement: **WorkManager**

### View
A **View** is the basic UI building block in Android.
Modern replacement: **Jetpack Compose**


### Intent
An **Intent** is a message used to communicate between Android components.
Barely noticeable because intent in Modern Android work inder the hood
You use it to:
- start an Activity
- start a Service
- send data between screens


# All Android app components

1/ An **Activity** represents a **single screen with a user interface**. It handle display UI, user input, navigation to other screen, and **Manage lifecycle events**. **Each Activity has a lifecycle that the Android system controls.**
Ex: Login screen, home page screen.
Lifecycle: onCreate()  , onStart()  ,onResume()  ,onPause()  , onStop()  , onDestroy()

2/ A **Service** performs **long-running operations in the background** without a UI.
Ex: Any background work like location tracking, music playback

3/ A **BroadcastReceiver** listens for **system-wide broadcast events**.
Ex: Battery low, Network connectivity change
A BroadcastReceiver is used to respond to broadcast messages from the system or other applications.

4/ A **ContentProvider** manages **shared application data**.
Ex: Sharing media files from gallery, sharing a link