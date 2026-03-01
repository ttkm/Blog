---
title: "Natas Level 3"
date: 2026-02-16T15:15:00-06:00
draft: false
tags:
  - security
  - natas
  - overthewire
  - curl
---

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

We have to snoop around and see if there's any sensitive data we can find. The hint is in the comment: "Not even Google will find it this time." One thing that can be hidden from Google is instructions for bots. If you haven't dealt with web crawlers before, see [robots.txt](https://en.wikipedia.org/wiki/Robots.txt). Sites often have a `robots.txt` file that tells crawlers what not to fetch. Let's see if it exists:

```bash
curl -u natas3:3gqisGdR0pjm6tpkDKdIWO2hSvchLeYH http://natas3.natas.labs.overthewire.org/robots.txt
```

```text
User-agent: *
Disallow: /s3cr3t/
```

The site owner told robots not to crawl `/s3cr3t/`. Let's peek inside that folder:

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

There's `users.txt` again. Fetch it:

```bash
curl -u natas3:3gqisGdR0pjm6tpkDKdIWO2hSvchLeYH http://natas3.natas.labs.overthewire.org/s3cr3t/users.txt
```

```text
natas4:QryZXc2e0zahULdHrtHxzyYkj59kUxLQ
```

A lesson to be learned: always remove any possible sensitive information, unless you like vulnerabilities.

**Password for natas4:** `QryZXc2e0zahULdHrtHxzyYkj59kUxLQ`
