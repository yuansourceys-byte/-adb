# -adb
远程操作
WirelessAdbRemote/
├── settings.gradle.kts
├── build.gradle.kts
├── gradle.properties
├── .github/workflows/build-apk.yml
└── app/
    ├── build.gradle.kts
    ├── proguard-rules.pro
    └── src/main/
        ├── AndroidManifest.xml
        ├── java/com/example/wirelessadb/
        │   ├── MainActivity.kt
        │   ├── AdbManager.kt
        │   └── Prefs.kt
        └── res/
            ├── layout/activity_main.xml
            ├── drawable/ic_launcher.xml
            ├── values/colors.xml
            ├── values/themes.xml
            └── values/strings.xml
