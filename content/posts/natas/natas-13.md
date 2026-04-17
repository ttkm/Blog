---
title: "Natas 13 - ???"
date: 2026-04-17T14:32:17-05:00
draft: true
toc: false
images:
tags:
  - security
  - natas
  - overthewire
  - curl
---

- **Username:** `natas13`
- **Password:** `trbs5pCjCrkuSknBBKHhaBxq6Wm1j3LC`
- **URL:** <http://natas13.natas.labs.overthewire.org>

```bash
curl -u natas13:trbs5pCjCrkuSknBBKHhaBxq6Wm1j3LC http://natas13.natas.labs.overthewire.org/
```

Body:

```html
<body>
<h1>natas13</h1>
<div id="content">
For security reasons, we now only accept image files!<br/><br/>


<form enctype="multipart/form-data" action="index.php" method="POST">
<input type="hidden" name="MAX_FILE_SIZE" value="1000" />
<input type="hidden" name="filename" value="8oo4uk0itp.jpg" />
Choose a JPEG to upload (max 1KB):<br/>
<input name="uploadedfile" type="file" /><br />
<input type="submit" value="Upload File" />
</form>
<div id="viewsource"><a href="index-source.html">View sourcecode</a></div>
</div>
</body>
```
