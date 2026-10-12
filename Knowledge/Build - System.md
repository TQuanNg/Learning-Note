**Kotlin Symbol Processing (KSP)** is a **lightweight, high-performance API for annotation processing in Kotlin**, designed as a modern replacement for Java’s **Annotation Processing Tool (APT)** and Kotlin’s **kapt**.

In short: **KSP lets libraries generate code by analyzing Kotlin source code directly—faster and more accurately than kapt.**

**kapt** works by generating Java stubs → ❌ slow
**KSP solves this by processing Kotlin symbols directly**, without converting to Java.


**AGP = Android Gradle Plugin**
It is the **tool that connects Android Studio with Gradle (the build system)** so your Android app can **build, compile, package, and run**.

When you click **Run ▶**, AGP handles:
✅ Compile Kotlin/Java code  
✅ Process AndroidManifest.xml  
✅ Compile XML layouts & resources  
✅ Manage dependencies  
✅ Generate BuildConfig & R classes  
✅ Create APK or AAB files  
✅ Handle build variants (debug/release)

**APK (Android Package Kit)** is the **installable app file** for Android.

**AndroidManifest.xml** : Defines apps's components, and permissions
Ex: To allow internet connection to the app:
```
<uses-permission android:name="android.permission.INTERNET" />
```


### Baseline Profiles
A performance optimization that helps your app start faster and feel smoother immediately after installation.
How it works: **AOT** = Ahead Of Time compilation
Without it: **JIT**: Just-in-time compilation
This improve code execution speed by about 30% from the first launch by avoiding interpretation and just-in-time (JIT) compilation steps for included code paths.