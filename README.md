
# react-native-scan-doc (WIP)

## Read before you proceed:
Most of the scan doc package out there cost a fortune. Last time I looked into one it cost tens of thousands euros a year.

Getting Opencv package to play nicely with RN out-of-box is almost impossible. Part of the reason is also that I try to avoid using install OpenCV manager for Android. Believe me I had tried make it as dummy as possible to the best of my knowledge. But the reality is setting up opencv to work nicely with Android itself is a challenging enough task. 

I suggest you look at a few examples to understand how OpenCV works with Android first then proceed to manual installation.
https://blog.nishtahir.com/2015/11/11/setting-up-for-android-ndk-development/

Try out my [Android Native Scan Doc](https://github.com/beast/android-opencv-scan-doc) sample if you want to first check out how the scan doc helps you to crop a doc in image.

Again, I am open to have some help from the community to improve the package documentation.

I myself unfortunately is tied up with other work of my own. Will look into this seriously once I got time and resource. Thanks.

## Getting started

`$ yarn add https://github.com/simokhalil/react-native-scan-doc`

### Mostly automatic installation (You still need to setup OpenCV yourself, See below))

`$ react-native link react-native-scan-doc`

### Setup OpenCV
##### 1. Download
- Download OpenCV package and extract it : https://sourceforge.net/projects/opencvlibrary/files/opencv-android/3.2.0/opencv-3.2.0-android-sdk.zip/download

##### 2. Setup OpenCV Android SDK
- Import OpenCV module in Android Studio (File / New / Import Module): `<path_to_downloaded_package>/sdk/java`
- In the project explorer change the project view from Android to Project. Open Project -> openCVLibrary320 -> build.gradle
- Change the compileSdkVersion, targetSdkVersion and buildToolsVersion value to the latest version you use. Then sync the project
- Configure Module as dependency: `File / Project Structure / app / Tab: Dependencies / + / Module Dependency / openCVLibrary320`

##### 3. Add Native libraries
- Now we need to add native JNI libraries in our project. These libraries should be added in jniLibs directory. Create a new jniLibs directory in `app -> src -> main`.
- Open the extracted OpenCV SDK directory. Switch to `OpenCV-android-sdk/sdk/native/libs` directory.
- You will find directories for many CPU architectures. Copy the required architecture directory to the jniLibs directory.
- In each copied directory, delete all files except `libopencv_java3.so`

##### 4. Migrate from ndkCompile to ndk-build
If you're using the deprecated ndkCompile, you should migrate to using either CMake or ndk-build. Because ndkCompile generates an intermediate Android.mk file for you, migrating to ndk-build may be a simpler choice.

To migrate from ndkCompile to ndk-build, proceed as follows:
- Compile your project with ndkCompile at least once by selecting `Build > Make` Project. This generates the Android.mk file for you.
- Locate the auto-generated Android.mk file by navigating to `project-root/module-root/build/intermediates/ndk/debug/Android.mk`
- Relocate the Android.mk file to `/app/src/mainjniLibs/Android.mk`. This makes sure that Gradle doesn't delete the script file when running the clean task
- Open the Android.mk file and edit any paths in the script such that they are relative to the current location of the script file. Here is an example : 

    ```
    LOCAL_PATH := $(call my-dir)
    include $(CLEAR_VARS)
    
    LOCAL_MODULE := app
    LOCAL_LDFLAGS := -Wl,--build-id
    LOCAL_SRC_FILES := \
        ./arm64-v8a/libopencv_java3.so \
        ./armeabi/libopencv_java3.so \
        ./armeabi-v7a/libopencv_java3.so \
        ./mips/libopencv_java3.so \
        ./mips64/libopencv_java3.so \
        ./x86/libopencv_java3.so \
        ./x86_64/libopencv_java3.so \
    
    LOCAL_C_INCLUDES += ../../debug/jni
    LOCAL_C_INCLUDES += ../jni
    LOCAL_C_INCLUDES += ./
    
    include $(BUILD_SHARED_LIBRARY)
    ```

- Link Gradle, by adding the following to the app's build.gradle (in `android` section):

    ```
    android {
        ...
        sourceSets { main { jni.srcDirs = ['src/main/jni', 'src/main/jniLibs/'] } }
        externalNativeBuild {
            ndkBuild {
                path 'src/main/jniLibs/Android.mk'
            }
        }
    }
    ```

- Finally, disable ndkCompile by opening the build.properties file and removing the following line, and sync the project:

    ```
    // Remove this line
    android.useDeprecatedNdk = true
    ```

##### 5. Init OpenCV in your main Activity
Add the following to the top of your MainActivity Class

```
public class MainActivity extends ReactActivity {
    
    private static final String TAG = "MainActivity";
    
    static {
        if(!OpenCVLoader.initDebug()){
            Log.d(TAG, "OpenCV not loaded");
        } else {
            Log.d(TAG, "OpenCV loaded");
        }
    }
    
    ...<existing code>
}
```

## Features
### Android
- [x] Scan document.
- [x] Perspective Transform.
  
### iOS
- [ ] Scan document.
- [ ] Perspective Transform.

## Requirements
### Android
API 16+
### iOS
iOS 8+
### React Native
RN 0.38+

### Manual installation

#### iOS

1. In XCode, in the project navigator, right click `Libraries` ➜ `Add Files to [your project's name]`
2. Go to `node_modules` ➜ `react-native-scan-doc` and add `RNScanDoc.xcodeproj`
3. In XCode, in the project navigator, select your project. Add `libRNScanDoc.a` to your project's `Build Phases` ➜ `Link Binary With Libraries`
4. Run your project (`Cmd+R`)<

#### Android

1. Open up `android/app/src/main/java/[...]/MainActivity.java`
  - Add `import my.fin.RNScanDocPackage;` to the imports at the top of the file
  - Add `new RNScanDocPackage()` to the list returned by the `getPackages()` method
2. Append the following lines to `android/settings.gradle`:
  	```
  	include ':react-native-scan-doc'
  	project(':react-native-scan-doc').projectDir = new File(rootProject.projectDir, 	'../node_modules/react-native-scan-doc/android')
  	```
3. Insert the following lines inside the dependencies block in `android/app/build.gradle`:
  	```
      compile project(':react-native-scan-doc')
  	```

## Usage
```javascript
import RNScanDoc from 'react-native-scan-doc';
  
const croppedImage = await RNScanDoc.scan(photo_path, 600, 800, 90, 'PNG', null);
console.log('Got scanned document : ', croppedImage.path);
```
  
