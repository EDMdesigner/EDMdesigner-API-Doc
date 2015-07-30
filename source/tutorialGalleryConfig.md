---
title: Integration Tutorial - gallery configuration

language_tabs:
  - php


toc_footers:
  - <a href='http://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - apiDocs

search: true
---

#Overview

<aside class="warning">
We suggest you to use your <a href="./index.html#gallery-handling">own gallery</a> with postMessages.
</aside>

In my last two tutorial posts I was talking about a basic way to integrate EDMdesigner into an arbitrary web-based system. This time, you will learn about how to configure the gallery to make it work. It won’t take long, I promise.

Also, it's possible to disable our built-in  gallery so you can use your own. I will write about that soon!

If you want to learn more about our old gallery handling, please read the related parts in our [docs](./builtInGallery.html).



# Gallery hook configuration
We do not store the images of our partners on our server side. If someone uploads an image to our gallery, we immediately upload it to our partner’s server. It means that there has to be an upload hook to be given on our side. There are two ways to configure it. The first option happens on the dashboard, but you can do it programmatically through our API as well.

<a href="./images/dashboard04.png" target="_blank"><img src="./images/dashboard04.png" /></a>

On the screenshot above you can see the part of our dashboard on which you can configure the hooks and some other things as well. The two most important ones are the upload and the delete hooks. Without them, the gallery cannot be functional. If you want to implement copy functionality, then you should set up a copy hook as well. There are three other things that you can set up here. The first two are messages that will appear if someone reached the limit what you can set up in their groups, so I will talk about those in a separate post. The last option is that you can forbid your users to set urls of the images from anywhere. This way you can force your users to use only those pictures which are hosted by you. It can be useful if you don’t want your email to be spammy. (If there are lot of different domains referenced in your email, that is generally bad for your spam score.)

You might want to configure your gallery programatically. It is useful when you create a system or plugin or whatever that you want to set up on different domains. If you include gallery setup in your install script, that will save you a lot of time. You can do it via our API. You need to generate an admin token on your server side and you can call the gallery config routes.




# Upload hook

```php
<?php
//ini_set('display_errors',1);
//ini_set('display_startup_errors',1);
//error_reporting(-1);
require_once dirname(__FILE__) . "/../../includes/config.php";
require_once dirname(__FILE__) . "/utils.php";
header("Content-type: application/json;charset=utf-8");
function printError($msg) {
	print "{\"err\": \"" . $msg . "\"}";
	exit;
}
if (!isset($_FILES["file"])) {
	printError("File input is needed.");
}
//security check
$hash = $_REQUEST["hash"];
$time = $_REQUEST["time"];
if (!checkHookSecurity($hash, $time)) {
	printError("Security error.");
}
if ($_FILES["file"]["size"] > 5000000) {
	printError("File size is too large.");
}
if ($_FILES["file"]["error"] > 0) {
	printError("Upload error, error code: " . $_FILES["file"]["error"]);
}
if (file_exists("upload/" . $_FILES["file"]["name"])) {
	printError("File already exists.");
}
//$contentType = mime_content_type($_FILES["file"]);
//$finfo = new finfo(FILEINFO_MIME_TYPE);
$extension = "";
if(getImageType($_FILES["file"]["tmp_name"])) {
	$extension = "." . getImageType($_FILES["file"]["tmp_name"]);
}
$userId = null;
if($_GET["userId"] === "templater") {
	$userId = "templater";
} else {
	$userId = explode(" - ", $_GET["userId"]);
	$userId = $userId[1];
}
$dirName = dirname(dirname(dirname(__FILE__))) . "/temp/user/" . $userId;
if (!file_exists($dirName)) {
	mkdir($dirName, 0755, true);
}
$savedFileName = "img_" . rand(1,100000000);
move_uploaded_file($_FILES["file"]["tmp_name"], $dirName . "/" . $savedFileName . $extension);
echo "{\"url\": \"" . SENDSTUDIO_APPLICATION_URL . "/admin/temp/user/" . $userId . "/" . $savedFileName . $extension . "\"}";
?>
```
The example on the right hand side is a real life example from our addon for Interspire Email Marketer (IEM). I think it's quite self explanatory if you take a look at the code, but let me emphasise two things:

 - The first is that you have to set the content type to **application/json** as you can see at the beginning of the file.
  - Also, it means that the response has to be a valid JSON.
 - The second thing is the [checkHookSecurity](#gallery-hook-security) function. You should implement something similar, otherwise others will be able to upload random content to your server.

Check the original file from our [IEM addon](https://github.com/EDMdesigner/IEM-EDMdesigner-Addon/blob/master/edmdesigner/UploadImage.php).

# Delete hook

```php
<?php
//ini_set('display_errors',1);
//ini_set('display_startup_errors',1);
//error_reporting(-1);
require_once dirname(__FILE__) . "/../../includes/config.php";
require_once dirname(__FILE__) . "/utils.php";
header("Content-type: application/json;charset=utf-8");
function printError($err) {
	print '{"err": ' . $err . '}';
	exit(1);
}
$json = file_get_contents("php://input");
$values = json_decode($json, true);
if($values == null) {
	printError("Null " . $json);
}
if (!isset($values["url"]) || !isset($values["userId"])) {
	printError("Could not delete resource. 0");
}
$hash = $values["hash"];
$time = $values["time"];
if (!checkHookSecurity($hash, $time)) {
	printError("Security error.");
}
$count = 0;
$path = str_replace(SENDSTUDIO_APPLICATION_URL . "/admin", "", $values["url"], $count);
$path = dirname(__FILE__) . "/../../" . $path;
if ($count == 1) {
	if (unlink($path)) {
		echo "{\"success\": true}";
	} else {
		printError("Could not delete resource. 1");
	}
} else {
	http_response_code(404);
}
?>
```

The rules that you have to follow are almost the same as in the case of the [upload hook](#upload-hook).

 - You have to set the content type to **applicatin/json** and send a valid JSON as response.
 - Also, you have to implement the [checkHookSecurity](#gallery-hook-security) function.


Check the original file from our [IEM addon](https://github.com/EDMdesigner/IEM-EDMdesigner-Addon/blob/master/edmdesigner/DeleteImage.php).

# Gallery hook security

```php
//utils.php
//check out the checkHookSecurity calls in the previous examples!
<?php
function checkHookSecurity($hash, $time) {
    $magic = getApiKeyAndMagic();
    $magic = $magic["EDMdesignerMagic"];
    return strcmp(md5($time . $magic), $hash) == 0;
}
?>
```

When we send requests to the hooks that you provided, we send a userId, a hash and a timestamp. The hash is the md5 hash of the concatenation of your magic word and the timestamp we send. This way you can make sure that **we** sent the request and you can send back 403 or an error message if someone is trying with some nasty things.

Check the original file from our [IEM addon](https://github.com/EDMdesigner/IEM-EDMdesigner-Addon/blob/master/edmdesigner/utils.php#L184).


If you want to learn more about our gallery handling, please read the related parts in our [docs](./index.html#gallery-handling).
