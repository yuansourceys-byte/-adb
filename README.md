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
pluginManagement {
    repositories { google(); mavenCentral(); gradlePluginPortal() }
}
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories { google(); mavenCentral() }
}
rootProject.name = "WirelessAdbRemote"
include(":app")
plugins {
    id("com.android.application") version "8.5.2" apply false
    id("org.jetbrains.kotlin.android") version "1.9.24" apply false
}
org.gradle.jvmargs=-Xmx2048m -Dfile.encoding=UTF-8
android.useAndroidX=true
android.nonTransitiveRClass=true
kotlin.code.style=official
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
}

android {
    namespace = "com.example.wirelessadb"
    compileSdk = 34

    defaultConfig {
        applicationId = "com.example.wirelessadb"
        minSdk = 26               // dadb 用到 java.time，26 起步最稳
        targetSdk = 34
        versionCode = 1
        versionName = "1.0"
    }

    buildTypes {
        release {
            isMinifyEnabled = false
            proguardFiles(getDefaultProguardFile("proguard-android-optimize.txt"), "proguard-rules.pro")
        }
        debug { }
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }
    kotlinOptions { jvmTarget = "17" }

    buildFeatures { viewBinding = true }

    packaging {
        resources {
            excludes += setOf(
                "META-INF/AL2.0",
                "META-INF/LGPL2.1",
                "META-INF/versions/9/OSGI-INF/MANIFEST.MF"
            )
        }
    }
}

dependencies {
    implementation("androidx.core:core-ktx:1.13.1")
    implementation("androidx.appcompat:appcompat:1.7.0")
    implementation("com.google.android.material:material:1.12.0")
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.8.1")
    implementation("dev.mobile:dadb:1.2.7")   // 解析不到就换成 Maven Central 上最新版
}
name: Build APK
on:
  push: { branches: [ main ] }
  workflow_dispatch:
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'
      - uses: gradle/actions/setup-gradle@v4
        with:
          gradle-version: '8.7'
      - name: Build debug APK
        run: gradle assembleDebug --no-daemon
      - uses: actions/upload-artifact@v4
        with:
          name: app-debug
          path: app/build/outputs/apk/debug/*.apk
          <?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />

    <application
        android:allowBackup="true"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/Theme.WirelessAdbRemote">

        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:windowSoftInputMode="adjustResize">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest>
<resources>
    <string name="app_name">无线ADB遥控</string>
</resources>
<resources>
    <color name="brand">#2F6BFF</color>
    <color name="ok">#16A34A</color>
    <color name="err">#DC2626</color>
    <color name="muted">#6B7280</color>
</resources>
<resources>
    <style name="Theme.WirelessAdbRemote" parent="Theme.Material3.DayNight.NoActionBar">
        <item name="colorPrimary">@color/brand</item>
        <item name="colorOnPrimary">#FFFFFF</item>
    </style>
</resources>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="108dp" android:height="108dp"
    android:viewportWidth="108" android:viewportHeight="108">
    <path android:fillColor="#2F6BFF" android:pathData="M0,0h108v108h-108z" />
    <path android:fillColor="#FFFFFF"
        android:pathData="M54,20l16,24h-10v44h-12v-44h-10z" />
</vector>
<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fillViewport="true">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="14dp">

        <!-- ============ ① 连接 ============ -->
        <com.google.android.material.card.MaterialCardView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            app:cardCornerRadius="18dp"
            app:cardElevation="0dp"
            app:strokeWidth="1dp"
            app:strokeColor="#22000000">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:padding="16dp">

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="① 连接设备"
                    android:textSize="15sp"
                    android:textStyle="bold" />

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="10dp"
                    android:orientation="horizontal">

                    <com.google.android.material.textfield.TextInputLayout
                        android:layout_width="0dp"
                        android:layout_height="wrap_content"
                        android:layout_weight="1"
                        android:hint="目标手机 IP"
                        app:boxBackgroundMode="outline">

                        <com.google.android.material.textfield.TextInputEditText
                            android:id="@+id/etIp"
                            android:layout_width="match_parent"
                            android:layout_height="wrap_content"
                            android:inputType="text"
                            android:maxLines="1" />
                    </com.google.android.material.textfield.TextInputLayout>

                    <com.google.android.material.textfield.TextInputLayout
                        android:layout_width="104dp"
                        android:layout_height="wrap_content"
                        android:layout_marginStart="8dp"
                        android:hint="端口"
                        app:boxBackgroundMode="outline">

                        <com.google.android.material.textfield.TextInputEditText
                            android:id="@+id/etPort"
                            android:layout_width="match_parent"
                            android:layout_height="wrap_content"
                            android:inputType="number"
                            android:maxLines="1" />
                    </com.google.android.material.textfield.TextInputLayout>
                </LinearLayout>

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="10dp"
                    android:gravity="center_vertical"
                    android:orientation="horizontal">

                    <com.google.android.material.button.MaterialButton
                        android:id="@+id/btnConnect"
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="连接"
                        app:cornerRadius="14dp" />

                    <TextView
                        android:id="@+id/tvStatus"
                        android:layout_width="0dp"
                        android:layout_height="wrap_content"
                        android:layout_marginStart="12dp"
                        android:layout_weight="1"
                        android:text="未连接"
                        android:textSize="13sp" />
                </LinearLayout>
            </LinearLayout>
        </com.google.android.material.card.MaterialCardView>

        <!-- ============ ② 配对（Android 11+） ============ -->
        <com.google.android.material.card.MaterialCardView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="10dp"
            app:cardCornerRadius="18dp"
            app:cardElevation="0dp"
            app:strokeWidth="1dp"
            app:strokeColor="#22000000">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:padding="16dp">

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="② 无线调试配对（Android 11+ 才需要）"
                    android:textSize="15sp"
                    android:textStyle="bold" />

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="4dp"
                    android:alpha="0.7"
                    android:text="目标机：开发者选项 → 无线调试 → 使用配对码配对设备"
                    android:textSize="12sp" />

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="10dp"
                    android:orientation="horizontal">

                    <com.google.android.material.textfield.TextInputLayout
                        android:layout_width="0dp"
                        android:layout_height="wrap_content"
                        android:layout_weight="1"
                        android:hint="配对端口"
                        app:boxBackgroundMode="outline">

                        <com.google.android.material.textfield.TextInputEditText
                            android:id="@+id/etPairPort"
                            android:layout_width="match_parent"
                            android:layout_height="wrap_content"
                            android:inputType="number"
                            android:maxLines="1" />
                    </com.google.android.material.textfield.TextInputLayout>

                    <com.google.android.material.textfield.TextInputLayout
                        android:layout_width="0dp"
                        android:layout_height="wrap_content"
                        android:layout_marginStart="8dp"
                        android:layout_weight="1"
                        android:hint="6 位配对码"
                        app:boxBackgroundMode="outline">

                        <com.google.android.material.textfield.TextInputEditText
                            android:id="@+id/etPairCode"
                            android:layout_width="match_parent"
                            android:layout_height="wrap_content"
                            android:inputType="number"
                            android:maxLength="6"
                            android:maxLines="1" />
                    </com.google.android.material.textfield.TextInputLayout>
                </LinearLayout>

                <com.google.android.material.button.MaterialButton
                    android:id="@+id/btnPair"
                    style="@style/Widget.Material3.Button.OutlinedButton"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="10dp"
                    android:text="配对" />
            </LinearLayout>
        </com.google.android.material.card.MaterialCardView>

        <!-- ============ ③ 滑动控制 ============ -->
        <com.google.android.material.card.MaterialCardView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="10dp"
            app:cardCornerRadius="18dp"
            app:cardElevation="0dp"
            app:strokeWidth="1dp"
            app:strokeColor="#22000000">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:padding="16dp">

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="③ 滑动控制"
                    android:textSize="15sp"
                    android:textStyle="bold" />

                <com.google.android.material.button.MaterialButton
                    android:id="@+id/btnSwipeUp"
                    android:layout_width="match_parent"
                    android:layout_height="64dp"
                    android:layout_marginTop="12dp"
                    android:text="⬆  上滑"
                    android:textSize="18sp"
                    app:cornerRadius="16dp" />

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="6dp"
                    android:alpha="0.6"
                    android:text="命令（可改）"
                    android:textSize="11sp" />

                <EditText
                    android:id="@+id/etCmdUp"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:background="@android:color/transparent"
                    android:fontFamily="monospace"
                    android:inputType="textNoSuggestions"
                    android:maxLines="2"
                    android:textSize="12sp" />

                <com.google.android.material.button.MaterialButton
                    android:id="@+id/btnSwipeDown"
                    android:layout_width="match_parent"
                    android:layout_height="64dp"
                    android:layout_marginTop="14dp"
                    android:text="⬇  下滑"
                    android:textSize="18sp"
                    app:cornerRadius="16dp" />

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="6dp"
                    android:alpha="0.6"
                    android:text="命令（可改）"
                    android:textSize="11sp" />

                <EditText
                    android:id="@+id/etCmdDown"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:background="@android:color/transparent"
                    android:fontFamily="monospace"
                    android:inputType="textNoSuggestions"
                    android:maxLines="2"
                    android:textSize="12sp" />

                <com.google.android.material.button.MaterialButton
                    android:id="@+id/btnAutoSize"
                    style="@style/Widget.Material3.Button.TextButton"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="6dp"
                    android:text="自动匹配目标屏幕分辨率（wm size）" />
            </LinearLayout>
        </com.google.android.material.card.MaterialCardView>

        <!-- ============ ④ 自定义命令 ============ -->
        <com.google.android.material.card.MaterialCardView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="10dp"
            app:cardCornerRadius="18dp"
            app:cardElevation="0dp"
            app:strokeWidth="1dp"
            app:strokeColor="#22000000">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:padding="16dp">

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="④ 自定义命令"
                    android:textSize="15sp"
                    android:textStyle="bold" />

                <EditText
                    android:id="@+id/etCustomCmd"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="8dp"
                    android:fontFamily="monospace"
                    android:hint="例如 input keyevent KEYCODE_HOME"
                    android:inputType="textNoSuggestions"
                    android:maxLines="2"
                    android:textSize="13sp" />

                <com.google.android.material.button.MaterialButton
                    android:id="@+id/btnRunCustom"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="10dp"
                    android:text="执行"
                    app:cornerRadius="14dp" />
            </LinearLayout>
        </com.google.android.material.card.MaterialCardView>

        <!-- ============ ⑤ 日志 ============ -->
        <com.google.android.material.card.MaterialCardView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="10dp"
            app:cardCornerRadius="18dp"
            app:cardElevation="0dp"
            app:strokeWidth="1dp"
            app:strokeColor="#22000000">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:padding="16dp">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:gravity="center_vertical"
                    android:orientation="horizontal">

                    <TextView
                        android:layout_width="0dp"
                        android:layout_height="wrap_content"
                        android:layout_weight="1"
                        android:text="日志"
                        android:textSize="15sp"
                        android:textStyle="bold" />

                    <com.google.android.material.button.MaterialButton
                        android:id="@+id/btnClearLog"
                        style="@style/Widget.Material3.Button.TextButton"
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="清空" />
                </LinearLayout>

                <TextView
                    android:id="@+id/tvLog"
                    android:layout_width="match_parent"
                    android:layout_height="170dp"
                    android:fontFamily="monospace"
                    android:scrollbars="vertical"
                    android:textIsSelectable="true"
                    android:textSize="11sp" />
            </LinearLayout>
        </com.google.android.material.card.MaterialCardView>

    </LinearLayout>
</ScrollView>
package com.example.wirelessadb

import android.content.Context

class Prefs(context: Context) {

    private val sp = context.getSharedPreferences("settings", Context.MODE_PRIVATE)

    var ip: String
        get() = sp.getString("ip", "") ?: ""
        set(v) = sp.edit().putString("ip", v).apply()

    var port: String
        get() = sp.getString("port", "5555") ?: "5555"
        set(v) = sp.edit().putString("port", v).apply()

    var pairPort: String
        get() = sp.getString("pairPort", "") ?: ""
        set(v) = sp.edit().putString("pairPort", v).apply()

    var cmdUp: String
        get() = sp.getString("cmdUp", DEFAULT_UP) ?: DEFAULT_UP
        set(v) = sp.edit().putString("cmdUp", v).apply()

    var cmdDown: String
        get() = sp.getString("cmdDown", DEFAULT_DOWN) ?: DEFAULT_DOWN
        set(v) = sp.edit().putString("cmdDown", v).apply()

    var customCmd: String
        get() = sp.getString("customCmd", DEFAULT_CUSTOM) ?: DEFAULT_CUSTOM
        set(v) = sp.edit().putString("customCmd", v).apply()

    companion object {
        const val DEFAULT_UP = "input swipe 540 1800 540 700 300"
        const val DEFAULT_DOWN = "input swipe 540 700 540 1800 300"
        const val DEFAULT_CUSTOM = "input keyevent KEYCODE_BACK"
    }
}
package com.example.wirelessadb

import dadb.Adb
import dadb.AdbKeyPair
import dadb.pairing.AdbPairingClient
import java.net.InetSocketAddress
import java.net.Socket

/* ================================================================
 *  dadb API 备注（很重要，万一编译报错，改这里就行，30 秒搞定）
 *
 *  1) AdbKeyPair.generate()      —— 生成一对新的 ADB RSA 密钥
 *     AdbKeyPair.read(路径, 路径)  —— 想持久化时用（本示例为了少踩坑先不落盘）
 *
 *  2) Adb.create(keyPair, host, port)  —— 建立连接，内部自动完成
 *     CNXN / AUTH(签名) 握手；目标机第一次会弹"允许调试"框
 *
 *  3) AdbPairingClient(host, port, code, keyPair) —— Android 11+
 *     用 6 位配对码配对。若你的 dadb 版本参数不同，按 README 微调即可。
 *
 *  4) ShellResponse.output —— 命令的标准输出字符串
 * ================================================================ */

/** 进程内共享同一对密钥：同一次运行里配对/连接用的是同一把钥匙 */
object AdbKeys {
    @Volatile private var cached: AdbKeyPair? = null

    @Synchronized
    fun get(): AdbKeyPair = cached ?: AdbKeyPair.generate().also { cached = it }
}

class AdbManager {

    private var adb: Adb? = null

    val connected: Boolean
        @Synchronized get() = adb != null

    /** 先用 3 秒超时探一下端口，避免 dadb 卡死在错误 IP 上 */
    fun reachable(host: String, port: Int, timeoutMs: Int = 3000): Boolean =
        runCatching {
            Socket().use { it.connect(InetSocketAddress(host, port), timeoutMs) }
        }.isSuccess

    @Synchronized
    fun connect(host: String, port: Int) {
        disconnect()
        adb = Adb.create(AdbKeys.get(), host, port)
    }

    @Synchronized
    fun disconnect() {
        runCatching { adb?.close() }
        adb = null
    }

    /** 在目标设备上跑一条 shell 命令，返回输出 */
    fun shell(command: String): String {
        val a = adb ?: throw IllegalStateException("尚未连接设备")
        return try {
            a.shell(command).output
        } catch (t: Throwable) {
            disconnect()   // 连接已断，清掉状态，下次需要重连
            throw t
        }
    }

    /** Android 11+：用配对码把我们的公钥写进目标机的 adb_keys */
    fun pair(host: String, port: Int, code: String) {
        AdbPairingClient(host, port, code, AdbKeys.get()).start()
    }
}
package com.example.wirelessadb

import android.os.Bundle
import android.text.method.ScrollingMovementMethod
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import androidx.core.widget.doAfterTextChanged
import androidx.lifecycle.lifecycleScope
import com.example.wirelessadb.databinding.ActivityMainBinding
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.launch
import kotlinx.coroutines.withContext
import java.text.SimpleDateFormat
import java.util.Date
import java.util.Locale

class MainActivity : AppCompatActivity() {

    private lateinit var b: ActivityMainBinding
    private lateinit var prefs: Prefs
    private val adb = AdbManager()
    private val logBuf = StringBuilder()
    private val fmt = SimpleDateFormat("HH:mm:ss", Locale.getDefault())

    private val colorOk = 0xFF16A34A.toInt()
    private val colorErr = 0xFFDC2626.toInt()
    private val colorMuted = 0xFF6B7280.toInt()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        b = ActivityMainBinding.inflate(layoutInflater)
        setContentView(b.root)
        prefs = Prefs(this)

        // ---- 恢复上次的输入 ----
        b.etIp.setText(prefs.ip)
        b.etPort.setText(prefs.port)
        b.etPairPort.setText(prefs.pairPort)
        b.etCmdUp.setText(prefs.cmdUp)
        b.etCmdDown.setText(prefs.cmdDown)
        b.etCustomCmd.setText(prefs.customCmd)

        b.tvLog.movementMethod = ScrollingMovementMethod()

        // ---- 编辑即保存（这就是"命令随便改"） ----
        b.etIp.doAfterTextChanged { prefs.ip = it?.toString().orEmpty() }
        b.etPort.doAfterTextChanged { prefs.port = it?.toString().orEmpty() }
        b.etPairPort.doAfterTextChanged { prefs.pairPort = it?.toString().orEmpty() }
        b.etCmdUp.doAfterTextChanged { prefs.cmdUp = it?.toString().orEmpty() }
        b.etCmdDown.doAfterTextChanged { prefs.cmdDown = it?.toString().orEmpty() }
        b.etCustomCmd.doAfterTextChanged { prefs.customCmd = it?.toString().orEmpty() }

        // ---- 点击事件 ----
        b.btnConnect.setOnClickListener { if (adb.connected) doDisconnect() else doConnect() }
        b.btnPair.setOnClickListener { doPair() }
        b.btnSwipeUp.setOnClickListener { runCmd(b.etCmdUp.text.toString(), "上滑") }
        b.btnSwipeDown.setOnClickListener { runCmd(b.etCmdDown.text.toString(), "下滑") }
        b.btnRunCustom.setOnClickListener { runCmd(b.etCustomCmd.text.toString(), "自定义") }
        b.btnAutoSize.setOnClickListener { autoFitScreen(verbose = true) }
        b.btnClearLog.setOnClickListener { logBuf.clear(); b.tvLog.text = "" }

        setStatus("未连接", colorMuted)
    }

    // ================= 连接 / 断开 =================

    private fun doConnect() {
        val host = b.etIp.text.toString().trim()
        val port = b.etPort.text.toString().trim().toIntOrNull() ?: 5555
        if (host.isEmpty()) { toast("请输入目标手机 IP"); return }

        prefs.ip = host
        prefs.port = port.toString()
        b.btnConnect.isEnabled = false
        setStatus("连接中…", colorMuted)

        lifecycleScope.launch {
            val err = withContext(Dispatchers.IO) {
                if (!adb.reachable(host, port)) {
                    return@withContext "无法连通 $host:$port（检查同一 Wi-Fi / 无线调试是否开启 / 端口是否正确）"
                }
                runCatching { adb.connect(host, port) }.exceptionOrNull()
            }

            b.btnConnect.isEnabled = true
            if (err == null) {
                b.btnConnect.text = "断开"
                setStatus("已连接 $host:$port", colorOk)
                log("已连接 $host:$port")
                autoFitScreen(verbose = false)
            } else {
                val m = err.message ?: err.toString()
                setStatus("连接失败", colorErr)
                log("连接失败：$m")
                toast("连接失败：$m")
            }
        }
    }

    private fun doDisconnect() {
        adb.disconnect()
        b.btnConnect.text = "连接"
        setStatus("未连接", colorMuted)
        log("已断开")
    }

    // ================= 配对 =================

    private fun doPair() {
        val host = b.etIp.text.toString().trim()
        val port = b.etPairPort.text.toString().trim().toIntOrNull()
        val code = b.etPairCode.text.toString().trim()
        if (host.isEmpty() || port == null || code.length != 6) {
            toast("请填好：目标 IP、配对端口、6 位配对码")
            return
        }
        b.btnPair.isEnabled = false
        lifecycleScope.launch {
            val err = withContext(Dispatchers.IO) { runCatching { adb.pair(host, port, code) }.exceptionOrNull() }
            b.btnPair.isEnabled = true
            if (err == null) {
                log("配对成功 ✔ 现在去目标机「无线调试」页面看连接用的 IP:端口，填上面后点连接")
                toast("配对成功")
            } else {
                log("配对失败：${err.message}")
                toast("配对失败：${err.message}")
            }
        }
    }

    // ================= 命令执行 =================

    private fun runCmd(cmd: String, label: String) {
        val c = cmd.trim()
        if (c.isEmpty()) { toast("命令是空的"); return }
        if (!adb.connected) { toast("请先连接设备"); return }

        lifecycleScope.launch {
            val r = withContext(Dispatchers.IO) { runCatching { adb.shell(c) } }
            r.onSuccess { out -> log("[$label] $c\n${out.trim()}") }
                .onFailure {
                    log("[$label] 执行失败：${it.message}")
                    b.btnConnect.text = "连接"
                    setStatus("连接已断开", colorErr)
                }
        }
    }

    /** 读目标机 wm size，按分辨率重写上下滑命令 */
    private fun autoFitScreen(verbose: Boolean) {
        if (!adb.connected) { if (verbose) toast("请先连接设备"); return }
        lifecycleScope.launch {
            val r = withContext(Dispatchers.IO) { runCatching { adb.shell("wm size") } }
            r.onSuccess { out ->
                val m = Regex("(\\d+)\\s*x\\s*(\\d+)").find(out)
                if (m == null) { log("解析分辨率失败：$out"); return@onSuccess }
                val w = m.groupValues[1].toInt()
                val h = m.groupValues[2].toInt()
                val x = w / 2
                val yTop = (h * 0.20).toInt()
                val yBottom = (h * 0.80).toInt()
                b.etCmdUp.setText("input swipe $x $yBottom $x $yTop 300")
                b.etCmdDown.setText("input swipe $x $yTop $x $yBottom 300")
                log("已按 ${w}x${h} 生成滑动命令")
                if (verbose) toast("已按 ${w}x${h} 生成命令")
            }.onFailure { log("获取分辨率失败：${it.message}") }
        }
    }

    // ================= 小工具 =================

    private fun setStatus(text: String, color: Int) {
        b.tvStatus.text = text
        b.tvStatus.setTextColor(color)
    }

    private fun log(msg: String) {
        logBuf.append('[').append(fmt.format(Date())).append("] ").append(msg).append('\n')
        if (logBuf.length > 30000) logBuf.delete(0, logBuf.length - 30000)
        b.tvLog.text = logBuf.toString()
    }

    private fun toast(s: String) = Toast.makeText(this, s, Toast.LENGTH_SHORT).show()

    override fun onDestroy() {
        adb.disconnect()
        super.onDestroy()
    }
}
