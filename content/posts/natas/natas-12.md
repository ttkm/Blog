---
title: "Natas 12 - ???"
date: 2026-04-11T13:36:15-05:00
draft: true
toc: false
images:
tags:
  - security
  - natas
  - overthewire
  - curl
---

- **Username:** `natas12`
- **Password:** `yZdkjAYZRd3R7tq7T5kXMjMJlOIkzDeB`
- **URL:** <http://natas12.natas.labs.overthewire.org>

```bash
curl -u natas12:yZdkjAYZRd3R7tq7T5kXMjMJlOIkzDeB http://natas12.natas.labs.overthewire.org/
```

Body:

```html
<body>
<h1>natas12</h1>
<div id="content">

<form enctype="multipart/form-data" action="index.php" method="POST">
<input type="hidden" name="MAX_FILE_SIZE" value="1000" />
<input type="hidden" name="filename" value="pp6m1a1b01.jpg" />
Choose a JPEG to upload (max 1KB):<br/>
<input name="uploadedfile" type="file" /><br />
<input type="submit" value="Upload File" />
</form>
<div id="viewsource"><a href="index-source.html">View sourcecode</a></div>
</div>
</body>
```


