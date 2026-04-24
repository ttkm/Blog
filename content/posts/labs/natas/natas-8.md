---
title: "Natas 8 - Reversible Encoding"
date: 2026-04-06T14:09:25-05:00
draft: false
aliases:
  - /posts/natas/natas-8/

toc: false
images:
tags:
  - security
  - natas
  - overthewire
  - curl
---

- **Username:** `natas8`
- **Password:** `xcoXLmzMkoIP9D7hlgPlh9XD7OgLAe5Q`
- **URL:** <http://natas8.natas.labs.overthewire.org>

```bash
curl -u natas8:xcoXLmzMkoIP9D7hlgPlh9XD7OgLAe5Q http://natas8.natas.labs.overthewire.org
```

Body:

```html
<body>
<h1>natas8</h1>
<div id="content">


<form method=post>
Input secret: <input name=secret><br>
<input type=submit name=submit>
</form>

<div id="viewsource"><a href="index-source.html">View sourcecode</a></div>
</div>
</body>
```

After checking the body, inspect `index-source.html` first.

```bash
curl -u natas8:xcoXLmzMkoIP9D7hlgPlh9XD7OgLAe5Q http://natas8.natas.labs.overthewire.org/index-source.html | w3m -T text/html
```

If `w3m` seems random here, check Natas 6 where I explain why I use it for cleaner terminal output.

Body:

```php
<?

$encodedSecret = "3d3d516343746d4d6d6c315669563362";

function encodeSecret($secret) {
    return bin2hex(strrev(base64_encode($secret)));
}

if(array_key_exists("submit", $_POST)) {
    if(encodeSecret($_POST['secret']) == $encodedSecret) {
    print "Access granted. The password for natas9 is <censored>";
    } else {
    print "Wrong secret";
    }
}
?>
```

From this source, user input (`$_POST['secret']`) is transformed with:

> original > base64_encode > reverse string > bin2hex

Then the result is compared against `$encodedSecret`.  
So we reverse that same pipeline on `$encodedSecret` to recover the original secret.

> bin2hex > reverse string > base64 decode > original

We can use a tiny PHP snippet:

```php
<?

$encodedSecret = "3d3d516343746d4d6d6c315669563362";

$decoded = base64_decode(strrev(hex2bin($encodedSecret)));

echo $decoded;
?>
```


Result:

```text
oubWYf2kBq
```


Now submit that recovered secret with curl:

```bash
curl -u natas8:xcoXLmzMkoIP9D7hlgPlh9XD7OgLAe5Q http://natas8.natas.labs.overthewire.org -d "secret=oubWYf2kBq&submit="
```

Body:

```html
<body>
<h1>natas8</h1>
<div id="content">

Access granted. The password for natas9 is ZE1ck82lmdGIoErlhQgWND6j2Wzz6b6t
<form method=post>
Input secret: <input name=secret><br>
<input type=submit name=submit>
</form>

<div id="viewsource"><a href="index-source.html">View sourcecode</a></div>
</div>
</body>
```

**Password for natas9:** `ZE1ck82lmdGIoErlhQgWND6j2Wzz6b6t`
