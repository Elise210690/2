Prerequisites (verified on Kubuntu 18.04.3):
- Build kstars from source according to https://indilib.org/forum/general/210-howto-building-latest-libindi-ekos.html
- Install Android SDK
- Download and install NDK version android-ndk-r17c.
- Install JDK java-8-openjdk-amd64:
    sudo apt-get install openjdk-8-jdk
- Install Qt open source for Android (newest version 5.13.1) under your home directory to be writable.
- After installation of Qt, enter the correct paths to the NDK, JDK and SDK in Tools>Options>Devices>Android and make sure Qt shows no warnings regarding these paths.
- Qt does not work with the newest SDK Tools, you must use v25 as suggested here:
  https://stackoverflow.com/questions/42754878/qt-creator-wont-list-any-available-android-build-sdks/42811774#42811774

  Basically, download https://dl.google.com/android/repository/tools_r25.2.5-linux.zip and replace the ANDROID_SDK_DIR/tools directory, 
  or just use the Android SDK download tool to use this version.
- Some tools are needed for the compilation: sudo apt-get install dos2unix ccache subversion ant libconfig-yaml-perl

Edit /etc/environment and add the following environmental variables before building:
    PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:$ANT_HOME/bin"
    JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64"
    QT_ANDROID="/$HOME/Qt/5.13.1/android_armv7"
    CMAKE_ANDROID_NDK="/$HOME/Android/Sdk/ndk/android-ndk-r17c"
    ANDROID_NDK="/$HOME/Android/Sdk/ndk/android-ndk-r17c"
    ANDROID_SDK_ROOT="/$HOME/Android/Sdk"
    ANDROID_API_LEVEL=17
    ANDROID_PLATFORM=17
    KSTARS_ROOT="/$HOME/Projects/kstars"
    ANT_HOME="/usr/share/ant"

If you want to generate signed release package set the following variables:
    export ANDROID_KEYSTORE=your_keystore_file
    export ANDROID_KEYSTORE_ALIAS=your_keystore_alias
    export KSTARS_ROOT=KStars Path (e.g. /$HOME/Projects/kstars)

Configure git (if you haven't done this already):
    git config --global user.email "you@example.com"
    git config --global user.name "Your Name"

Make a build directory in a separate location form source (e.g. /$HOME/Projects/build/kstars-android):
    - Run build_kf5.sh and verify that the script builds everything without any problem.
    NOTE: Ki18n often fails to build the first time, but it should succeeds when you rebuild kf5 again.

    $KSTARS_ROOT/packaging/android/build_kf5.sh

Configure out-of-source build (MinSizeRel build type is recommended for Android):
    cmake -B. -H$KSTARS_ROOT -DBUILD_KSTARS_LITE=ON -DCMAKE_TOOLCHAIN_FILE=$CMAKE_ANDROID_NDK/build/cmake/android.toolchain.cmake \
      -DEIGEN3_INCLUDE_DIR=/usr/include/eigen3/ -DCMAKE_INSTALL_PREFIX=$(pwd)/android/export -DCMAKE_BUILD_TYPE=MinSizeRel \
      -DECM_DIR=/usr/share/ECM/cmake/ -DCMAKE_PREFIX_PATH=$QT_ANDROID \
      -DCMAKE_AR=${ANDROID_NDK}/toolchains/llvm/prebuilt/linux-x86_64/bin/llvm-ar

Compile:
    make -j4
    
Download and convert the translations:
    make convert_translations_to_android
    
Install:
    make -j4 install
    
Generate the Android debug package:
    make create-apk-debug-kstars

Install the Android debug package to your phone:
    make install-apk-debug-kstars


