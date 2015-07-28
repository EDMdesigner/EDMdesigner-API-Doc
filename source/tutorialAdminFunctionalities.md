---
title: Integration Tutorial - admin functionalities

language_tabs:
  - html
  - php


toc_footers:
  - <a href='http://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - apiDocs

search: true
---

#Overview
In this post you will read about how to create users programmatically and you will get some advice how to associate your users to EDMdesigner’s users.


In the previous [tutorial post](./tutorialTheBasics.html) I was talking about the very basics of using EDMdesigner’s API. This time I will be talking about topics, with which it’s already possible to really integrate EDMdesigner into your system, meaning that there will be direct association between your users in your database and EDMdesigner users.

# Create EDMdesigner users for all of your users
One way to do it is that you create an initialization script which creates EDMdesigner users for all of your users in your database. For this, you should create users in our system when you create a new user in yours, so if a user registers to your system, then a user should be created in our system as well. To provide one to one relationship between the users in your and in our system, the user name of EDMdesigner users should be the user id of your users,  their e-mail address or something else which is unique per API key. The important thing is that based on the user name you provided to our system, you can find out which is that user in your system.


# Create users on the fly
An other way could be to check if the actual user ever used EDMdesigner and if not, you can register that user on the fly. I think it’s easier to implement, and that extra checking is nothing compared to the ease of the implementation.
Since there are several ways to implement it actually, I will only focus on creating users from code, nothing else, so I won’t talk about concrete implementations.

# Admin level access token

```html
<script src="path_to_your_jquery"></script>
<!-- at the moment, our JS API depends on jQuery... -->
<script src="http://api-static.edmdesigner.com/EDMdesignerAPI.js"></script>
<script>
	//you can take a look at token.php if you change to the php tab
	initEDMdesignerPlugin("token.php", "admin", function(edmDesignerApi) {
	  //actual calls on edmApi's admin functions... eg. listing, createing users, etc.
	  //USE THE JS API with admin level access token only on administrative interfaces
	  //don't forget to check on the server side if your user has the necessarry priviledges
	});
</script>
```

```php
//token.php
<?php
if ($_POST["userId"]) {
	$publicId = "YOUR_DEV_API_KEY";
	$magic = "YOUR_MAGIC_WORD";

	$ip = $_SERVER["REMOTE_ADDR"];
	$timestamp = time();

	$hash = md5($publicId . $ip . $timestamp . $magic);

	$url = "http://api-a.edmdesigner.com/api/token"; //In real life, you should implement
	//a fail over mechanism to api-b.edmdesigner.com and api-c.edmdesigner.com

	//Also, make some authorization based on data in your session,
	//because there is a chance that someone tries to be tricky.
	//For example you might want to prevent most of your users
	//to edit templater's projects...


	// !!!IMPORTANT!!!
	// if ($_POST["userId"] == "admin") -> DO NOT FORGET TO CHECK
	// IF YOUR USER HAS ADMIN PRIVILEDGES!!!

	$data = array(
		"id"	=> $publicId,
		"uid"	=> $_POST["userId"],
		"ip"	=> $ip,
		"ts"	=> $timestamp,
		"hash"	=> $hash
	);

	// use key 'http' even if you send the request to https://...
	$options = array(
	    "http" => array(
	        "header"  => "Content-type: application/x-www-form-urlencoded\r\n",
	        "method"  => "POST",
	        "content" => http_build_query($data),
	    ),
	);

	$context  = stream_context_create($options);
	$result = file_get_contents($url, false, $context);

	print($result);
}
?>
```


The first thing you have to do, is to create an “admin token”. This is a special kind of token which is used for admin (or superuser) functionalities. These are: creating, updating, removing users, managing groups, etc. Now, I will concentrate only on user creation. The token creation is very similar to what we saw in the last post but in this case we have to send a “admin”  as the username to our token generator route. This “admin” user is non-existent in our DB – not like “templater” , which is existent -, it’s only used to make difference between admin and normal user functionalities. This also means that you can’t create a user with this name in our system.

It is very, very important that you make sure that only your administrators can create admin token. You have to check in your session that the actual user of yours have rights to use the admin functionalities in our API.

If you use the Javascript API, then do it only on administrative interfaces, where it’s sure that only your admins can use it.




# Making API calls from your 

```html
//Please switch to the php tab
```

```php
//createUserExample.php
//you can also use cURL if you want to.
<?php
//in production you should failover to api-b.edmdesigner.com and api-c.edmdesigner.com
$url = "http://api-a.edmdesigner.com/json/user/create?user=admin&token=YOUR_ADMIN_TOKEN";
$data = array("id" => "USER_ID_FROM_YOUR_SYSTEM");

// use key 'http' even if you send the request to https://...
$options = array(
    "http" => array(
        "header"  => "Content-type: application/x-www-form-urlencoded\r\n",
        "method"  => "POST",
        "content" => http_build_query($data),
    ),
);
$context  = stream_context_create($options);
$result = file_get_contents($url, false, $context);

var_dump($result);
?>
```

There are cases when you need deeper integration – eg. when a user registers, you register it to EDMdesigner as well. In these cases you should make the API calls from your server side.

The Javascript API works with JSONP calls, which has it’s limitations. From the server side you can call the JSON versions of the routes. Also, there are some extra routes which you can call from the server side but not with JSONP. These routes are usually dangerous to call from the client side, since there should be some post processing on the result. An example for this is when you call our HTML generator without sanitizing. It can be useful when you have some kind of query language and based on that you fetch info from your own DB just before sending. If our sanitizer would replace the queries that you inserted into the template, probably you would go crazy. This means that you have to take care about sanitizing the final result.

So from the server side routes we will use the one with which you can create a new user. Basically you have to post some data to the following route:

**//api.edmdesigner.com/json/user/create?user=admin&token=YOUR_ADMIN_TOKEN**
 
The only required thing you should post is the id, which can be the userId in your system. This way you can create a one-to-one association between your and our users. You will be able to reach this user’s functionalities with this id you gave.
One very useful thing is that you can associate customData to every user (and to every projects as well). This way you can customize the way you integrate EDMdesigner into your system even better. You have to post a customData field when you create your user, which you will reach and modify later.

Please check out our PHP example for [admin functionalities on GitHub](https://github.com/EDMdesigner/EDMDesigner-API-Example-PHP-Admin).

If you want to learn more about the access token generation, please read the corresponding part in our [docs](./index.html#generating-an-access-token).
