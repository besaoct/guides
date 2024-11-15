# Integrating Google Maps and Configuring Gradle in Flutter

### **Google Maps Integration (Android)**

To integrate Google Maps in your Flutter project, follow these steps:

#### **Add Google Maps API Key**

   **Create a `strings.xml` File**:  
   In Android, the Google Maps API key is stored in the `strings.xml` file for security reasons.

   Navigate to `android/app/src/main/res/values/strings.xml` and add the following code:

   ```xml
   <resources>
      <string name="google_maps_api_key">YOUR_GOOGLE_MAPS_API_KEY_HERE</string>
   </resources>
   ```

   Replace `YOUR_GOOGLE_MAPS_API_KEY_HERE` with your actual Google Maps API key.

   **Add Meta-data for Google Maps API Key**:  
   Open `android/app/src/main/AndroidManifest.xml` and add the following inside the `<application>` tag:

   ```xml
   <meta-data
       android:name="com.google.android.geo.API_KEY"
       android:value="@string/google_maps_api_key" />
   ```

---

### **Permissions for Android**

#### **Location Permissions**

In Android, you need to request permissions for location access (both foreground and background). To enable this:

   **Modify `AndroidManifest.xml`**:  
   Add the following permission requests inside the `<manifest>` tag:

   ```xml
   <!-- Location permissions -->
   <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
   <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />

   <!-- Background location permission (required only if you need to access location in the background) -->
   <uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
   ```

  **Optional: External Storage Access**  
   If your app needs access to the camera or external storage for reading or writing files:

   ```xml
   <!-- Camera permission -->
   <uses-permission android:name="android.permission.CAMERA" />

   <!-- File access permissions for reading and writing external storage -->
   <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
   <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
   ```

---

### **Permissions for iOS**

For iOS, you need to update the `Info.plist` file with the required usage descriptions for camera, photo library, and location.

#### **Modify `Info.plist` for iOS Permissions**

   **Location Permissions**:
   Add the following keys for location access:

   ```xml
   <key>NSLocationWhenInUseUsageDescription</key>
   <string>We need your location to show it on the map.</string>
   <key>NSLocationAlwaysUsageDescription</key>
   <string>We need your location to show it on the map.</string>
   ```

   **Camera & Microphone Permissions**:
   Add the keys for camera, photo library, and microphone usage:

   ```xml
   <key>NSCameraUsageDescription</key>
   <string>We need access to your camera to take pictures or scan documents for tasks such as attendance marking or other security-related activities.</string>
   
   <key>NSPhotoLibraryUsageDescription</key>
   <string>We need access to your photo library to upload photos related to your shift, such as vehicle registration or other necessary documentation.</string>
   
   <key>NSMicrophoneUsageDescription</key>
   <string>We need access to your microphone for recording any audio related to your tasks during the shift.</string>
   ```

   **Background Location Permissions**:
   If you need background location tracking, add the following:

   ```xml
   <key>UIBackgroundModes</key>
   <array>
      <string>location</string>
   </array>
   ```

   **Google Maps API Key for iOS**:
   Add the API key for Google Maps in the `Info.plist`:

   ```xml
   <key>GMSApiKey</key>
   <string>YOUR_GOOGLE_MAPS_API_KEY_HERE</string>
   ```

   Replace `YOUR_GOOGLE_MAPS_API_KEY_HERE` with your actual Google Maps API key.

---

### **Gradle Configuration for Android**

To configure Gradle for your Flutter project, follow these steps:
   
   **Gradle Distribution URL**  
   Specify the Gradle distribution URL by adding the following line in your `android/gradle/wrapper/gradle-wrapper.properties` file:

   ```properties
   distributionUrl=https\://services.gradle.org/distributions/gradle-8.9-all.zip
   ```

  **Add Required Plugins**  
   Open `android/build.gradle` and add the necessary plugins for Flutter, Android, and Kotlin:

   ```gradle
   plugins {
       id "dev.flutter.flutter-plugin-loader" version "1.0.0"
       id 'com.android.application' version '8.7.0' apply false
       id 'org.jetbrains.kotlin.android' version '2.0.20' apply false
   }
   ```

  **Set Compile Options**  
   Set the Java version compatibility for your Android project:

   ```gradle
   compileOptions {
       sourceCompatibility = JavaVersion.VERSION_1_8
       targetCompatibility = JavaVersion.VERSION_1_8
   }
   ```

  **Kotlin Options**  
   If you're using Kotlin, set the JVM target version in your `android/app/build.gradle` file:

   ```gradle
   kotlinOptions {
       jvmTarget = JavaVersion.VERSION_1_8
   }
   ```

---

### **Conclusion**

With the above configurations:

- You will have integrated Google Maps in your Flutter app.
- The necessary permissions for location, camera, and external storage are requested on Android and iOS.
- Gradle configurations have been set to ensure compatibility with the required versions of Flutter, Kotlin, and Java.
  
This ensures a smooth setup for Flutter development with Google Maps integration and permission management across both Android and iOS.
