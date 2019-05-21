protobuf-android-native
=======================
The goal of this project is to provide a pre-compiled version of libprotobuf and libprotobuf-lite.
This builds a static library to use protobuf with C++ on android.

# Build
* Clone this repo `git clone https://github.com/sbsb99/protobuf-android-native.git`
* Clone submodules `git submodule init && git submodule update`
* Change directory `cd android_library`
* Run gradle `./gradlew externalNativeBuildCleanRelease externalNativeBuildRelease`
Libraries and headers are located in `output\${VERSION}\`

# Protobuf Version
* Checkout protobuf submodule `cd protobuf; git checkout v3.6.1`

# ABI Setting
* Modify the `android.defaultConfig.ndk.abiFilters` in the `android_library/protobuf`

# NDK Version
* Open `android_library` project in Android Studio.
* `File->Project Structure->SDK Location` Set Android NDK location
* or Modify the `ndk.dir` in the `android_library/local.properties`
