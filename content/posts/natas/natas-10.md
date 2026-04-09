---
title: "Natas 10 - ???"
date: 2026-04-09T16:23:56-05:00
draft: true
toc: false
images:
tags:
  - security
  - natas
  - overthewire
  - curl
---

- **Username:** `natas10`
- **Password:** `t7I5VHvpa14sJTUGV0cbEsbYfFP2dmOu`
- **URL:** <http://natas10.natas.labs.overthewire.org>

```bash
curl -u natas10:t7I5VHvpa14sJTUGV0cbEsbYfFP2dmOu http://natas10.natas.labs.overthewire.org
```

Body:

```html
<body>
<h1>natas10</h1>
<div id="content">

For security reasons, we now filter on certain characters<br/><br/>
<form>
Find words containing: <input name=needle><input type=submit name=submit value=Search><br><br>
</form>


Output:
<pre>
</pre>

<div id="viewsource"><a href="index-source.html">View sourcecode</a></div>
</div>
</body>
```

Again, inspect `index-source.html`.

```bash
curl -u natas10:t7I5VHvpa14sJTUGV0cbEsbYfFP2dmOu http://natas10.natas.labs.overthewire.org/index-source.html | w3m -T text/html
```

```php
<?
$key = "";

if(array_key_exists("needle", $_REQUEST)) {
    $key = $_REQUEST["needle"];
}

if($key != "") {
    if(preg_match('/[;|&]/',$key)) {
        print "Input contains an illegal character!";
    } else {
        passthru("grep -i $key dictionary.txt");
    }
}
?>
```


