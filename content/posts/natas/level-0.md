---
title: "Natas Level 0"
weight: 3
date: 2026-02-16T15:00:00-06:00
draft: false
tags:
  - security
  - natas
  - overthewire
  - curl
---

Natas teaches the basics of server-side web security. This walkthrough uses cURL so you get used to the command line and useful options.

For the first level we're given:

- **Username:** `natas0`
- **Password:** `natas0`
- **URL:** <http://natas0.natas.labs.overthewire.org>

cURL is a command-line tool for transferring data to and from a server. To fetch the HTML source for natas0 we need the `-u` option, which sends the username and password for server authentication:

```bash
curl -u user:password URL
```

So we can request the HTML for natas0 with:

```bash
curl -u natas0:natas0 http://natas0.natas.labs.overthewire.org
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
<script src="http://natas.labs.overthewire.org/js/wechall-data.js"></script>
<script src="http://natas.labs.overthewire.org/js/wechall.js"></script>
<script>var wechallinfo = { "level": "natas0", "pass": "natas0" };</script>
</head>
<body>
<h1>natas0</h1>
<div id="content">
You can find the password for the next level on this page.

<!--The password for natas1 is 0nzCigAq7t2iALyvU9xcHlYN4MlkIwlq -->
</div>
</body>
</html>
```

For this level we only need to look at the body. When you do, you'll see a comment left for us there: the password for natas1.

**Password for natas1:** `0nzCigAq7t2iALyvU9xcHlYN4MlkIwlq`
