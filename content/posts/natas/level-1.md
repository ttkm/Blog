---
title: "Natas Level 1"
date: 2026-02-16T15:05:00-06:00
draft: false
tags:
  - security
  - natas
  - overthewire
  - curl
---

- **Username:** `natas1`
- **Password:** `0nzCigAq7t2iALyvU9xcHlYN4MlkIwlq`
- **URL:** <http://natas1.natas.labs.overthewire.org>

Request the page with cURL:

```bash
curl -u natas1:0nzCigAq7t2iALyvU9xcHlYN4MlkIwlq http://natas1.natas.labs.overthewire.org
```

Example output:

```html
<html>
<head>
<!-- This stuff in the header has nothing to do with the level -->
<link rel="stylesheet" type="text/css" href="http://natas.labs.overthewire.org/css/level.css">
<link rel="stylesheet" href="http://natas.labs.overthewire.org/css/jquery-ui.css" />
<link rel="stylesheet" href="http://natas.labs.overthewire.org/css/wechall.css" />
<script src="http://natas.labs.overthewire.org/js/jquery-1.9.1.js"></script>
<script src="http://natas.labs.overthewire.org/js/jquery-ui.js"></script>
<script src=http://natas.labs.overthewire.org/js/wechall-data.js></script><script src="http://natas.labs.overthewire.org/js/wechall.js"></script>
<script>var wechallinfo = { "level": "natas1", "pass": "0nzCigAq7t2iALyvU9xcHlYN4MlkIwlq" };</script></head>
<body oncontextmenu="javascript:alert('right clicking has been blocked!');return false;">
<h1>natas1</h1>
<div id="content">
You can find the password for the
next level on this page, but rightclicking has been blocked!

<!--The password for natas2 is TguMNxKo1DSa1tujBLuZJnDUlCcUAPlI -->
</div>
</body>
</html>
```

For this level it's fairly simple since we are not using a browser. Looking at the source code, the author disabled right‑clicking on the page, but lucky for us we can easily see the password using cURL.

**Password for natas2:** `TguMNxKo1DSa1tujBLuZJnDUlCcUAPlI`
