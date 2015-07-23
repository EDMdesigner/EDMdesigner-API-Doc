---
title: API Reference - the basics

language_tabs:
  - javascript
  - php


toc_footers:
  - <a href='http://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - apiDocs

search: true
---

#Overview - the basics

[EDMdesigner](http://www.edmdesigner.com) is a drag and drop tool for creating responsive HTML e-mail templates very quickly and painlessly, radically increasing your click-through rate. This document gives you the basic information what you will need to integrate it to your system (e.g. a CRM, CMS, WebShop or anything else you can imagine). Also, you can find other documentations at the [other resources](#other-resources) section.


To start developing with our API, please request access to our dashboard, where you can manage your API keys. (Please contact us at info@edmdesigner.com.)

With an API key and the corresponding "magic word" you can generate an access token with which you can make request to our API. You will need to send the access token with each API requests to our servers.

At a certain point your users will open our editor. When the editor's iframe is opened, you will be able to modify its behaviour through our [postMessage API](./postMessageApi.html). That's the client side of our API. The whole code (js, css and html as well) of the editor is [served with Amazon's CDN (Cloudfront)](http://edmdesigner.com/blog/update-edmdesigner-uses-amazon-cloudfront) so it should load super fast from all over the world.



#Making requests

There are two main ways to make requests to our server. The first is when you send a simple HTTP request to one of our API endpoints, the second is when you send a [JSONP](https://en.wikipedia.org/wiki/JSONP) request.

In the latter case you will be able to create most of the integration on the client side. For this purpose we have created a Javascript wrapper with built-in failover between our endpoints. You can find that [here](http://api-static.edmdesigner.com/EDMdesignerAPI.js).

**Our API endpoints are the followings:**

 - api-a.edmdesigner.com
 - api-b.edmdesigner.com
 - api-c.edmdesigner.com

Every time when you make an API request, you should do it with failover to the other endpoints. (Again, in our JS API, this failover mechanism is built-in.)

Whenever you send a request to one of our API endpoints, you should send a userId and a token with it. These are access tokens, about which you can learn more about in the [authentication](#authentication) chapter. The lifetime of a token is one day and if you try to make an API call without it, then the response will have [403](https://en.wikipedia.org/wiki/HTTP_403) status code.

There are some API requests, which are not reachable via a JSONP call. This is because those API calls considered to be dangerous, so you should call them from your server-side and do the appropriate authentication and authorization on your users.

It might happen that you try to make extremely lot of API calls, especially when something is messed up with your integration. If it happens, then after a while we will send back a response with [429](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes#429) status code and won't serve your requests for a while. So integrate carefully, and don't try to send too many requests. (Our API is [rate limited](https://en.wikipedia.org/wiki/Rate_limiting).)

At the moment, our [JS API](http://api-static.edmdesigner.com/EDMdesignerAPI.js)'s dependence is jQuery. It's only because of the convenient ajax requests, but probably we will change it to native ajax in the near future.

We suggest you to use the [JS API](http://api-static.edmdesigner.com/EDMdesignerAPI.js) as much as it's possible.


We chose PHP as the language for the server side examples, because most of the people can at least read it.



#Authentication

In this chapter you will learn about the authentication and authorization process in our API.
Basically you will need to generate a user level or admin level access token with which you can make API calls.


## Generating an access token

First of all, you will need an API key and a corresponding magic word to generate an access token. You can manage your API keys and magic words on our dashboard.

Basically you need to HTTP POST from your server to one of our [API endpoints](#making-requests) the followin data to the **/api/token** route:

Param | Description
---|---
id | one of your API keys
uid | your user's id
ip | ipv4 address of your client
ts | a timestamp (now)
hash | The hash has to be the string representation of the md5 hash of the concatenation of the API key, the ipv4 address, the timestamp and the magic word: <br/> **hash = md5(API_KEY + ipv4 + timestamp + magic)**

If everything goes well, our server will return a JSON containing a token property which is the access token. If not, then you the response's status code might be 403 or you might receive a JSON with an err property in it.

You might want to save the access token to your user's session for later use. Our tokens are valid for one day, so if your session's lifetime is less or equal to that, then you are good to go. If your session's lifetime is more than that, you will need to regenerate the access tokens from time to time.

It is very important to build-in a failover mechanism to the token generation process as well. So if you don't get proper response from one of our endpoints, you should retry it with the other [endpoints](#making-requests).

It's also extremely important to authorize your users on your server side, so they won't be able to generate access tokens in each other's name.


In the following chapters you will see examples for the token generation process.



## Using the access token

Once you have an access token, you can make requests to our API endpoints. The only thing you have to do is to add the userId (what you used when you created the token) and the token itself to the query string as a parameter of the HTTP of JSONP request:

**http(s)://api.end.point/request/route?userId=youruserid&token=youraccesstoken**

For exmaple:

https://api-a.edmdesginer.com/json/project/list?user=userId&token=123456789

If you use our [JS API](http://api-static.edmdesigner.com/EDMdesignerAPI.js) then you won't have to manually tinker with concatenating the correct request urls, the JS API module will do it for you. You will just need to call the corresponding functions. (You will find these functions as well later in the documentation.)

The other very good thing about using this JS API is, that the failover mechanism is built-in so you won't need to implement failover manually.

If the token is not valid any more or the userId that you sent is not correct, then the HTTP response's statuc code will be [403](https://en.wikipedia.org/wiki/HTTP_403) and your request won't be served.

## User level access token

```javascript
//First include jQuery somewhere in your page, because our JS API is depending on that
//Use the JS API from: https://api-static.edmdesigner.com/EDMdesignerAPI.js
initEDMdesignerPlugin("##handshaking_route##","##userId##", function onSuccess(edmDesignerApi) {
  //the edmDesignerApi object will contain all the functions which you can use
}, function onError(error) {
  console.log(error);
});


//A concrete example with token.php as the handshaking route and templater as the user
initEDMdesignerPlugin("token.php","templater", function onSuccess(edmDesignerApi) {
  //the edmDesignerApi object will contain all the functions which you can use
  //all of the things will happen in the name of the templater user here,
  //so authorize your users carefully!!!
}, function onError(error) {
  console.log(error);
});
```


```php
//token.php
//This example does not implement failover mechanism for easier understanding.

<?php
if ($_POST["userId"]) {
  //here you have to authorize your user!!!
  //don't let your users to generate access tokens in each other's name
  // - they could mess up each others' projects

  $publicId = "YOUR_API_KEY";
  $magic = "YOUR_MAGIC_WORD";
  $ip = $_SERVER["REMOTE_ADDR"];
  $timestamp = time();
  $hash = md5($publicId . $ip . $timestamp . $magic);
  $url = "http://api-a.edmdesigner.com/api/token"; //should fail over to api-b & api-c as well.
  $data = array(
        "id"  => $publicId,
        "uid" => $_POST["userId"],
        "ip"  => $ip,
        "ts"  => $timestamp,
        "hash"  => $hash
  );
  // use key 'http' even if you send the request to https://...
  $options = array(
      "http" => array(
          "header"  => "Content-type: application/x-www-form-urlencoded\r\n",
          "method"  => "POST",
          "content" => http_build_query($data),
      )
  );
  $context  = stream_context_create($options);
  $result = file_get_contents($url, false, $context);
  print($result);
}
?>
```

In this section you will see an example for basic user level token generation. This is the basic permission level. With a user level access token you can manage the thing associated to that user, nothing more. There are also admin level calls, which you can read about in the next [section](#admin-level-access-token).

You can follow the whole process in the example on the right. On the [javascript tab](?javascript#user-level-access-token), you can see examples for the [JS API usage](?javascript#user-level-access-token), and on the [php tab](?php#user-level-access-token), you can see a very simple example implementation of the [server side](?php#user-level-access-token) part of the handshaking process. (This is your part of the token generation).

To use our [JS API](https://api-static.edmdesigner.com/EDMdesignerAPI.js), you just have to include it in your page with a script tag. (&lt;script src="https://api-static.edmdesigner.com/EDMdesignerAPI.js"&gt;&lt;/script&gt;) 

By doing this, you will have a global initEDMdesignerPlugin function.
The first parameter is the route where the JS module will post the userId, which is the second parameter. If your user has rights to generate a token, then you will have to post all the necessary info to our endpoint. In the example, there is no proper authorization, since that depends on your system.

The whole process in the example is the following:

 - the JS API sends the userId to token.php
 - token.php sends the neccessary info to one of our endpoints
 - our endpoint responds with a token
 - JS API will use this token for the subsequent API calls

There is a special user associated to each API key. It's name is templater and its projects are the default projects in the system. It is always existent and cannot be deleted. The [list defaults route](#project-list-defaults) will always respond with the templater's projects. (Our concrete example on the right is generating a token for the templater user.)






## Admin level access token

```javascript
//First include jQuery somewhere in your page, because our JS API is depending on that
//Use the JS API from: https://api-static.edmdesigner.com/EDMdesignerAPI.js
//A concrete example with token.php as the handshaking route and admin as the user
//Be very careful, authorize your users on the server side very carefully!!!
initEDMdesignerPlugin("token.php","templater", function onSuccess(edmDesignerApi) {
  //the edmDesignerApi object will contain all the functions which you can use
  //you will be able to reach all the admin functionality here!
  //so authorize your users carefully!!!
}, function onError(error) {
  console.log(error);
});
```

```php
<?php
if ($_POST["userId"]) {
  //here you have to authorize your user!!!
  //don't let your users to generate access tokens in each other's name
  //don't let your users to generate an admin token if they don't have the needed permissions for it!!!
  // - they could mess up each others' projects

  $publicId = "YOUR_API_KEY";
  $magic = "YOUR_MAGIC_WORD";
  $ip = $_SERVER["REMOTE_ADDR"];
  $timestamp = time();
  $hash = md5($publicId . $ip . $timestamp . $magic);
  $url = "http://api-a.edmdesigner.com/api/token"; //should fail over to api-b & api-c as well.
  $data = array(
        "id"  => $publicId,
        "uid" => $_POST["userId"], //which supposed to be "admin"
        "ip"  => $ip,
        "ts"  => $timestamp,
        "hash"  => $hash
  );
  // use key 'http' even if you send the request to https://...
  $options = array(
      "http" => array(
          "header"  => "Content-type: application/x-www-form-urlencoded\r\n",
          "method"  => "POST",
          "content" => http_build_query($data),
      )
  );
  $context  = stream_context_create($options);
  $result = file_get_contents($url, false, $context);
  print($result);
}
?>
```

The generation of an admin level access token is almost the same as generating a regular token. The only difference is that you have to use the string "admin" as the user. This is not an existent user in our system, we just use it to generate an admin level access token.

You have to be very careful with this! Always authorize your users on your side of the handshaking process (token generation).


# Models
## The project model
```javascript
//This is not a code example!!!
//This is the original file in which the mongoose schema and model is defined.
var mongoose = require("mongoose");

var Schema = mongoose.Schema;

var projectSchema = new Schema({
  user: {type: Schema.ObjectId, ref: "User", required: true},
  title: {type: String, required: true},
  description: {type: String},

  createdOn: {type: Date, default: Date.now},
  lastModified: {type: Date, default: Date.now},

  document: {type: Schema.Types.Mixed},

  customData: {type: Schema.Types.Mixed}
});

module.exports = mongoose.model("Project", projectSchema);
```

```php
//Switch to the javascript tab to see the original mongoose schema and model of our Projecs.
```
Throughout the documentation you will be reading about requests with which you can make certain operations on projects. These projects have the follwing properties:

Field | Type | Required | Description
------|------|----------|------------
_id | ObjectId | true | The MongoDB id of the project. (Autogenerated, so if you create a project, you don't have to give it.)
user | ObjectId | true | A reference to the owner of the project.
title | String | true | The title of the project.
description | String | false | Optional extra information.
createdOn | Date | false | The time when the project was created at.
lastModified | Date | false | The time when the project was modified for the last time.
document | Mixed | false | The document descriptor object. Learn more about it. TODO LINK
customData | Mixed | false | A custom object in which you can put extra info of the project. For example if it's a template or a campaign.


## The mongoose query object

```javascript
//A relatively complex example for the query object.
var query = {
  find: {
    "customData.campaign": true,
    "title": {
      $regex: "foo.*"
    }
  },
  select: ["title", "lastModified", "createdOn"],
  skip: 0,
  limit: 10,
  sort: {
    lastModified: -1
  }
};
```

In the documentation there will be several routes with which you can make queries. It basically means that you can set the parameters of the db search. You can place your settings object to the query of the get request.

Field | Type | Required | Description
------|------|----------|------------
find | Object | false | Please check the [moongoose docs](http://mongoosejs.com/docs/queries.html).
select | Array | false | Please check the [moongoose docs](http://mongoosejs.com/docs/queries.html).
skip | Number | false | Please check the [moongoose docs](http://mongoosejs.com/docs/queries.html).
limit | Number | false | Please check the [moongoose docs](http://mongoosejs.com/docs/queries.html).
sort | Object | false | Please check the [moongoose docs](http://mongoosejs.com/docs/queries.html).

You can set the parameters of the db search (you have to use the moongose.js query syntax, please check their [documentation](http://mongoosejs.com/docs/guide.html) for more information)


# Project handling
```javascript
initEDMdesignerPlugin("TestUser", function(edmDesignerApi) {
  //In the follwing examples we assume that you put the code snippets here,
  //where edmDesignerApi is on the closure.
}, function onError(error) {
  console.log(error);
});
```
The following section is about the functionalities which are reachable with a token generated to a user.

## Project - list
```javascript
//example without query
edmDesignerApi.listProjects(function(result) {
  //handling the result.
}, function onError(error) {
  console.log(error);
});

//example with query
var settings = {
  find: {
    "customData.campaign": true,
    "title": {
      $regex: "foo.*"
    }
  },
  select: ["title", "lastModified", "createdOn"],
  skip: 0,
  limit: 10,
  sort: {
    lastModified: -1
  }
};
edmDesignerApi.listProjects(settings, function(result) {
  //handling the result.
}, function onError(error) {
  console.log(error);
});
```

Lists the projects of the actual user.

HTTP | JSONP | JS API
-----|-------|-------
GET /json/project/list | GET /jsonp/project/list | edmDesignerApi.listProjects(settings, callback, onErrorCB)

### Parameters:
Field | Type | Required | Description
------|------|----------|------------
settings | [Query Object](#the-mongoose-query-object) | false | Optional query object

### Response
The response is an array of projects. or it can be an error object:
  - err Description of the error {String} or an error code {Number}.




## Project - list and count

```javascript
//example without query
edmDesignerApi.listAndCountProjects(function(result) {
  //result should be an array of projects
}, function onError(error) {
  console.log(error);
});

//example with query
var settings = {
  find: {
    "customData.campaign": true,
    "title": {
      $regex: "foo.*"
    }
  },
  select: ["title", "lastModified", "createdOn"],
  skip: 0,
  limit: 10,
  sort: {
    lastModified: -1
  }
};
edmDesignerApi.listAndCountProjects(settings, function(result) {
  //result.count should exist and should be a Number
  //result.response should exist and should be an array of projects
}, function onError(error) {
  console.log(error);
});
```

Lists and counts the projects of the actual user, which satisfies the query.

HTTP | JSONP | JS API
-----|-------|-------
GET /json/project/listAndCount | GET /jsonp/project/listAndCount | edmDesignerApi.listAndCountProjects(settings, callback, onErrorCB)

### Parameters:

Field | Type | Required | Description
------|------|----------|------------
settings | [Query Object](#the-mongoose-query-object) | false | Optional query object

### Response
Compared to [list projects](#list-projects), listAndCountProjects' response is an object with two properties: count and response. The first is the number of entries which satisfies the query, the second is the actual array of items. __The count property will hold a value if and only if you set the limit and skip property.__ Otherwise its output will be the same like at [list projects](#list-projects).

...or it can be an error object:
  - err Description of the error {String} or an error code {Number}.






## Project - list defaults

```javascript
edmDesignerApi.getDefaultTemplates(function(result) {
  //the result is an array, containing your default projects
}, function onError(error) {
  console.log(error);
});
```
You can get the default templates povided by your templater user by calling this funciton. It means that if you have a user called templater, this route will result its projects. (Every API key has an associated templater user by default.)

HTTP | JSONP | JS API
-----|-------|-------
GET /json/project/defaults | GET /json/project/defaults | edmDesignerApi.getDefaultTemplates(callback, onErrorCB)

### Parameters:

Field | Type | Required | Description
------|------|----------|------------
query | [Query Object](#the-mongoose-query-object) | true | A mongodb query obj.


###Response:

* A list of the default projects - satisfying the criteria in the query.
* Or in case of of limit and skip parameters sent an Object consists of:
  * totalCount {Number} The length of the full list
  * result {Array} The list of the projects, same as above
* or it can be an error object:
  * err Description of the error {String} or an error code {Number}.







## Project - create

```javascript
var exampleProject = {
  title: "test title",
  description "test description",
  document: {} //an empty doc
};
edmDesignerApi.createProject(exampleProject, function(result) {
  console.log(result._id);
}, function onError(error) {
  console.log(error);
});
```

Creates a new project (a new e-mail template).

HTTP | JSONP | JS API
-----|-------|-------
POST /json/project/create | GET /jsonp/project/create | edmDesignerApi.createProject(data, callback, onErrorCB)



### Parameters:

Field | Type | Required | Description
------|------|----------|------------
title | String | true | The title of the template / campaign
description | String | false | Optional description of the project
document | Object | true | The document descriptor object. (See docs. TODO LINK)
customData | Object | false | Extra info about the project.

### Response:
The result object has an _id field with is the id of the new project. (This is a [MongoDB](http://www.mongodb.org/) id.)
or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

!!!!JSONP LIMITATIONS!!!



## Project - duplicate

```javascript
var projectId = "#somehow you have to set the projectId.";
edmDesignerApi.duplicateProject(projectId, function(result) {
  //result._id is the id of the new project
}, function onErrorCB(error) {
  console.log(error);
});
```

Creates the exact copy of the project with the ID specified by the projectId.

HTTP | JSONP | JS API
-----|-------|-------
GET /json/project/duplicate/:id | GET /jsonp/project/duplicate/:id | edmDesignerApi.duplicateProject(projectId, callback, onErrorCB)

### Parameters (in the route):

Field | Type | Required | Description
------|------|----------|------------
:id | String | true | The string representation of the project id. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you [listed the projects of the user](#list-projects).


### Response:
Should be a [project](#the-project-model) object.
or it can be an error object:
  - err Description of the error {String} or an error code {Number}.







## Project - create from own
```javascript
var projectId = "#somehow you have to set the projectId.";
edmDesignerApi.createFromOwn(projectId, {title: "createFromOwn example", description: "Created with createFromOwn function"}, function(result) {
  //the result is the newly created project's object or error obejct with err property
});
```

This function is similar to the [duplicate project](#duplicate-project) function, expect whit this function you can define the the title and the description of the new (copy) project. The user can only create the new project from one of his/her own project.

HTTP | JSONP | JS API
-----|-------|-------
POST /json/project/createFromOwn | GET /jsonp/project/createFromOwn | edmDesignerApi.createFromOwn(projectId, data, callback, onErrorCB)

### Parameters:

Field | Type | Required | Description
------|------|----------|------------
_id | ObjectId | true | The id of the object to copy.
data | [Project](#the-project-model) | true | You should not set the document property, since it will be copied from the other project. (Basically, you can overwrite the title, description and customData of the copied project.)


### Response:
The new project's json.
or it can be an error object:
  - err Description of the error {String} or an error code {Number}.









## Project - create from defaults

```javascript
var templaterProjectId = null;
    
initEDMdesignerPlugin("templater", function(edmDesignerApi) {
  edmDesignerApi.createProject({title: "test-title", description: "test-desc"}, function(result) {
    templaterProjectId = result._id;
  });
}, onErrorCB);

initEDMdesignerPlugin("TestUser", function(edmDesignerApi) {

  edmDesignerApi.createFromDefaults(templaterProjectId, {title: "createFromDefaults example", description: "Created with createFromDefaults function"}, function(result) {
    //the result is the newly created project's object or error obejct with err property
  });

function onErrorCB(error) {
  console.log(error);
}
```

The user can create a new project based on one of the templater user's projects.

HTTP | JSONP | JS API
-----|-------|-------
POST /json/project/createFromDefaults | GET /json/project/createFromDefaults | edmDesignerApi.createFromDefaults(projectId, data, callback, onErrorCB)

### Parameters:

Field | Type | Required | Description
------|------|----------|------------
_id  | ObjectId | true | The id of the default project from which you want to create the new one. The owner has to be the templater user of that project.
data | [Project](#the-project-model) | true | You should not set the document property since the copied project's document will be used.

### Response:
A [project](#the-project-model).
or it can be an error object:
  - err Description of the error {String} or an error code {Number}.











## Project - update info

```javascript
initEDMdesignerPlugin("TestUser", function(edmDesignerApi) {
  edmDesignerApi.createProject({title: "test-title", description: "test-desc"}, function(result) {
    edmDesignerApi.updateProjectInfo(result._id, {title: "New title", description: "New description", customData: {foo: "BAR"}}, function(result) {
      //the result is an object with success {Boolean} property or err {String} property.
    });
  });
}, onErrorCB);

function onErrorCB(error) {
  console.log(error);
}
```

Generates the bulletproof responsive HTML e-mail based on the projectId.

HTTP | JSONP | JS API
-----|-------|-------
 | | edmDesignerApi.updateProjectInfo(projectId, data, callback, onErrorCB)

### Parameters:

Field | Type | Required | Description
------|------|----------|------------

  * projectId {String} The id of the project. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list the projects of the user with the [edmDesignerAPI.listProjects](#edmdesignerapilistprojectscallback-onerrorcb) function.
  * data {Object}
    * data.title {String} The new title of the project
    * data.description {String} The new description of the project
    * data.customData {Object} You can upload custom informations to this project. You can save any kind of information. It is up to you, how you want to use it!
    * data.override {Boolean} If it is true then the customData will be overrided with the newly given data, if it is false then the newly given customData will be merged with the previous ones. By default it is false.
  * callback {Function} A function to be called if the request succeeds
  * onErrorCB {Function} A function to be called if the request fails



### Update information
Updates the title or/and the description of the specified project. You can add custom informations to this project too.

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/project/updateInfo

#### Parameters (you should post):
  * projectId {String} /REQUIRED/ The id of the project. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list the projects of the user with the [/json/project/list](#list-projects) route.
  * title {String} The title of the new project.
  * description {String} The description of the new project.
  * customData {Object} You can upload custom informations to this project. You can save any kind of information. It is up to you, how you want to use it!
  * override {Boolean} If it is true then the customData will be overrided with the newly given data, if it is false then the newly given customData will be merged with the previous ones. By default it is false.

#### Answer:
  - success {Boolean} It should be true if the update was successful

or it can be an error object:
  - err Description of the error {String} or an error code {Number}.













## Project - remove

```javascript
initEDMdesignerPlugin("TestUser", function(edmDesignerApi) {
  edmDesignerApi.createProject({title: "test-title", description: "test-desc"}, function(result) {
    edmDesignerApi.removeProject(result._id, function(result) {
      //callback hell...
    });
  });
}, onErrorCB);

function onErrorCB(error) {
  console.log(error);
}
```

Removes a project.

HTTP | JSONP | JS API
-----|-------|-------
 | | edmDesignerApi.removeProject(projectId, callback, onErrorCB)

### Parameters:

Field | Type | Required | Description
------|------|----------|------------

  * projectId {String} The id of the project. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list the projects of the user with the [edmDesignerAPI.listProjects](#edmdesignerapilistprojectscallback-onerrorcb) function.
  * callback {Function} A function to be called if the request succeeds
  * onErrorCB {Function} A function to be called if the request fails

### Remove
Removes a project.

#####Type
  + DELETE

#####Route
  + //api.edmdesigner.com/json/project/remove/:id

#### Parameters (in the route):
  * :id {String} The id of the project. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list the projects of the user with the [/json/project/list](#list-projects) route.

####Response:
A number, it is 1 if the project was successfully deleted

or it can be an error object:
  - err Description of the error {String} or an error code {Number}.













## Project - generate HTML

```javascript
initEDMdesignerPlugin("TestUser", function(edmDesignerApi) {
  edmDesignerApi.createProject({title: "test-title", description: "test-desc"}, function(result) {
    edmDesignerApi.generateProject(result._id, function(result) {
      //the result is a robust responsive HTML e-mail code.
    });
  });
}, onErrorCB);

function onErrorCB(error) {
  console.log(error);
}
```

Generates the bulletproof responsive HTML e-mail based on the projectId.

TODO: preview!!!
+ insert export menu item
+ insert preview menu item !!!

HTTP | JSONP | JS API
-----|-------|-------
 | | edmDesignerApi.generateProject(projectId, callback, onErrorCB)

### Parameters:

Field | Type | Required | Description
------|------|----------|------------

  * projectId {String} The id of the project. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list the projects of the user with the [edmDesignerAPI.listProjects](#edmdesignerapilistprojectscallback-onerrorcb) function.
  * callback {Function} A function to be called if the request succeeds
  * onErrorCB {Function} A function to be called if the request fails



### Generate
Generates the bulletproof responsive HTML e-mail based on the projectId.

#####Type
  + GET

#####Route
  + //api.edmdesigner.com/json/project/generate/:id

#### Parameters (in the route):
  * :id {String} The id of the project. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list the projects of the user with the [/json/project/list](#list-projects) route.

####Response
A bulletproof responsive HTML version of the given template

or it can be an error object:
  - err Description of the error {String} or an error code {Number}.



### Generate without sanitizing
Generates the responsive HTML e-mail based on the projectId. The generated html won't be sanitized. (good for those, who wants to use complex placeholders the normal [generate route](#generate) would erase.

#####Type
  + GET

#####Route
  + //api.edmdesigner.com/json_v1.0.0/apiKey/:apiKey/user/:userid/project/:projectid/generateWithoutSanitizing

#### Parameters (in the route):
  * :apiKey {String} The key of the instance the target user belongs to
  * :userId {String} The id of the project's owner. Plesase note that this id should be the same you used for the token generation.
  * :projectid {String} The id of the project. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list the projects of the user with the [/json/project/list](#list-projects) route.

####Response
A not sanitized responsive HTML version of the given template

or it can be an error object:
  - err Description of the error {String} or an error code {Number}.


## Project - export HTML

See [generate HTML](#project-generate-html).

## Project - preview

See [generate HTML](#project-generate-html).


## Project - get title
Gets the title of the selected project.

//TODO this is a stupid route

#####Type
  + GET

#####Route
  + //api.edmdesigner.com/json/project/title/:id

#### Parameters (in the route):
  * :id {String} The id of the project. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list the projects of the user with the [/json/project/list](#list-projects) route.

####Response
Title object:
  - title {String} The title of the selected project

or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

## Project - Open

```javascript
initEDMdesignerPlugin("TestUser", function(edmDesignerApi) {
  edmDesignerApi.createProject({title: "test-title", description: "test-desc"}, function(result) {
    edmDesignerApi.openProject(result._id, function(result) {
      $("body").append(result.iframe);
    });
  });
}, onErrorCB);

function onErrorCB(error) {
  console.log(error);
}
```

Opens a project.

//TODO - reference to the next chapter

HTTP | JSONP | JS API
-----|-------|-------
 | | edmDesignerApi.openProject(projectId, [languageCode], [settings], callback, onErrorCB)

### Parameters:

Field | Type | Required | Description
------|------|----------|------------

  * projectId {String} The id of the project. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list the projects of the user with the [edmDesignerAPI.listProjects](#edmdesignerapilistprojectscallback-onerrorcb) function.
  * languageCode {String} /Optional/ A two character [ISO 639-1 code](http://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) of a selected language. With this parameter you can choose the language which the opened project will use. You can find the [list of the available languages](#languages) at the end of this page. Default language is english.
  * settings {Object} /Optional/ With this object, it is possible to set some basic settings:
    * autosave {Number} With this property you can set the frequency of the autosave (in millisecond). Please note that the autosave only save the project if there were changes after the last save, with this property you can only set how often should the autosave handler check if there are any kind of changes in the template. If you want to turn off the autosave functionality, then you have to set this property to zero (0). It is possible to manually trigger the saving mechanism with the [save project message](#save-project).
  * callback {Function} A function to be called if the request succeeds
  * onErrorCB {Function} A function to be called if the request fails



# Editor iframe

//TODO - talk about the iframe and its query string
//TODO - talk about the CDN based thing
//TODO - reference to the previous point


---

# User handling

```javascript
initEDMdesignerPlugin("TestAdmin", function(edmDesignerApi) {
  edmDesignerApi.listGroups(function(result) {
    //the result is an array, containing your groups
  }, onErrorCB);
});

function onErrorCB(error) {
  console.log(error);
}
```

You have to generate an admin token!







## User - list

```javascript
initEDMdesignerPlugin("TestAdmin", function(edmDesignerApi) {
  edmDesignerApi.listUsers(function(result) {
    //the result is an array, containing your users
    //A user is an object with
    //id (the user's id)
    //group (The group which the user belongs) and
    //createTime (Time of creation) properties
    //customData (The custom informations you saved for the user)
    console.log(result);
  }, onErrorCB);
});

function onErrorCB(error) {
  console.log(error);
}
```

### edmDesignerApi.listUsers(callback, onErrorCB)
Lists the users you have
#### Parameters:
  * callback {Function} A function to be called if the request succeeds
  * onErrorCB {Function} A function to be called if the request fails

### List
Lists the users you have

#####Type
  + GET

#####Route
  + //api.edmdesigner.com/json/user/list
  
#### Parameters (optionals):
  - skip {Number} index where from the listing should begin, without limit parameter it will be ingnored
  - limit {Number} length of the items to be listed, without skip parameter it will be ingnored
  - select [Array] list of the properties to be listed ["name", "created"]
  - sort {Object} MongoDb sort object: {property : "asc/desc"}

####Response:
An array of your users. Every user is an object with this parameters:
  - id {String} The id of the user
  - group {String} The MongoDB _id of the group the user belongs to
  - createTime {String} The time when the user was created
  - customData {Object} The custom informations you saved for the user

Or in case of of limit and skip parameters sent an Object consists of:
  - totalCount {Number} The length of the unfiltered list
  - result {Array} The list of the users, same as above

Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.












## User - create

```javascript
initEDMdesignerPlugin("TestAdmin", function(edmDesignerApi) {
  edmDesignerApi.createGroup({name: "exampleGroup", featureSwitch: {}}, function(result) {
  
    edmDesignerApi.createUser({id: "exampleuserId", group: result._id}, function(resultUser) {
      // the resultUser is an object with an id property
      console.log(resultUser);
    }, onErrorCB);
  
  }, onErrorCB);
});

function onErrorCB(error) {
  console.log(error);
}
```

### edmDesignerApi.createUser(data, callback, onErrorCB)
Creates a new user
#### Parameters:
  * data {Object}
    * data.id {String} /REQUIRED/ The id you want to use for this new user
    * data.group {String} The id of the group you want this user to belong. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list your groups with the [edmDesignerAPI.listGroups](#edmdesignerapilistgroupscallback-onerrorcb) function.
    * data.customData {Object} You can aupload custom informations to this user. You can save any kind of information. It is up to you, how you want to use it!
  * callback {Function} A function to be called if the request succeeds
  * onErrorCB {Function} A function to be called if the request fails


### Create user
Creates a new user

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/user/create

#### Parameters (you should post):
  * id {String} /REQUIRED/ The id you want to use for this new user
  * group {String} The id of the group you want this user to belong. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list your groups with the [/json/groups/list](#list-groups) route.
  * customData {Object} You can upload custom informations to this user. You can save any kind of information. It is up to you, how you want to use it!

####Response:
User object:
  - id {String} The id of the user (your id, not the MongoDB _id)

Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.








## User - create multiple

```javascript
initEDMdesignerPlugin("TestAdmin", function(edmDesignerApi) {
  edmDesignerApi.createGroup({name: "exampleGroup", featureSwitch: {}}, function(result) {
  
    edmDesignerApi.createMultipleUser([{id: "exampleUserId", group: result._id}, {id: "exampleUser2Id}, {id: "exampleUser3"}], function(resultObj) {
      // the resultObj is an object with the following properties:
      // created {Array} It contains the users whose have been created
      // failed {Array} It contains the users whose creation has failed
      // alreadyHave {Array} It contains the users whose you already created
      console.log(resultObj);
    }, onErrorCB);
  
  }, onErrorCB);
});

function onErrorCB(error) {
  console.log(error);
}
```

### edmDesignerApi.createMultipleUser(data, callback, onErrorCB)
Creates multiple user
#### Parameters:
  * data {Array} It contains user objects. User object should have the following properties:
    * id {String} /REQUIRED/ The id you want to use for this new user
    * group {String} The id of the group you want this user to belong. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list your groups with the [edmDesignerAPI.listGroups](#edmdesignerapilistgroupscallback-onerrorcb) function.
    * customData {Object} You can aupload custom informations to this user. You can save any kind of information. It is up to you, how you want to use it!
  * callback {Function} A function to be called if the request succeeds
  * onErrorCB {Function} A function to be called if the request fails

### Create multiple user
Creates multiple user

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/user/multipleCreate

#### Parameters (you should post):
  * users {Array} It contains user objects. User object should have the following properties:
    * id {String} /REQUIRED/ The id you want to use for this new user. Please note that if there is no id then the server will automatically ignore that input! (In this way you can get an empty array as an answer)
    * group {String} The id of the group you want this user to belong. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list your groups with the [/json/groups/list](#list-groups) route.
    * customData {Object} You can upload custom informations to the user. You can save any kind of information. It is up to you, how you want to use it!

####Response:
Three different arrays:
  - created {Array} Contains the ids of the users who was successfully created
    - id {String} The id of the user 
  - alreadyHave {Array} Contains the users you already had
    - id {String} The id of the user
    - group {String} The MongoDB _id of the group the user belongs to
    - createTime {String} The time when the user was created
  - falied {Array} Contains the ids of the users whose creation failed
    - id {String} The id of the user

Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.










## User - get one

```javascript
initEDMdesignerPlugin("TestAdmin", function(edmDesignerApi) {
  edmDesignerApi.createUser({id: "exampleUserId"}, function(result) {
  
    edmDesignerApi.getUser(result.id, function(resultUser) {
      /**the resultUser is an object with
      id (the user's id)
      group (The group which the user belongs) and
      createTime (Time of creation) properties
      customData (the custom data you saved for this user) */
      console.log(resultUser);
    }, onErrorCB);
  
  }, onErrorCB);
});

function onErrorCB(error) {
  console.log(error);
}
```

### edmDesignerApi.getUser(userId, callback, onErrorCB)
Gets a specified user
#### Parameters:
  * userId {String} The id of the user.
  * callback {Function} A function to be called if the request succeeds
  * onErrorCB {Function} A function to be called if the request fails


### Get one user
Gets a specified user

#####Type
  + GET

#####Route
  + //api.edmdesigner.com/json/user/getOne/:id

####Parameters (in the route):
   * :id {String} The id of the user.

####Response:
User object:
  - id {String} The id of the user
  - group {String} The MongoDB _id of the group the user belongs to
  - createTime {String} The time when the user was created
  - customData {Object} The custom informations you saved for the user

Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

















## User - update

```javascript
initEDMdesignerPlugin("TestAdmin", function(edmDesignerApi) {
  edmDesignerApi.createUser({id: "exampleUserId"}, function(result) {
    edmDesignerApi.createGroup({name: "exampleGroup", featureSwitch: {}}, function(resultGroup) {
    
      edmDesignerApi.updateUser(result.id, {group: resultGroup._id}, function(resultUser) { 
        //the resultUser is an object with an id property
        console.log(resultUser);
      }, onErrorCB);
    
    }, onErrorCB);
  }, onErrorCB);
});

function onErrorCB(error) {
  console.log(error);
}
```

### edmDesignerApi.updateUser(userId, data, callback, onErrorCB)
Updates a specified user. Only the group (which the user belongs) can be changed. You can add custom informations to this user too.
#### Parameters:
  * userId {String} The id of the user. 
  * data {Object}
    * data.group {String} The id of the group you want this user to belong. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list your groups with the [edmDesignerAPI.listGroups](#edmdesignerapilistgroupscallback-onerrorcb) function.
    * data.customData {Object} You can upload custom informations to this user. You can save any kind of information. It is up to you, how you want to use it!
    * data.override {Boolean} If it is true then the customData will be overrided with the newly given data, if it is false then the newly given customData will be merged with the previous ones. By default it is false.
  * callback {Function} A function to be called if the request succeeds
  * onErrorCB {Function} A function to be called if the request fails


### Update user
Updates a specified user. Only the group (which the user belongs) can be changed. You can add custom informations to the user too.

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/user/update

#### Parameters (you should post):
   * id {String} The id of the user. 
   * group {String} The id of the group you want this user to belong. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list your groups with the [/json/groups/list](#list-groups) route.
   * customData {Object} You can upload custom informations to this user. You can save any kind of information. It is up to you, how you want to use it!
   * override {Boolean} If it is true then the customData will be overrided with the newly given data, if it is false then the newly given customData will be merged with the previous ones. By default it is false.

####Response:
User object:
  - id {String} The id of the updated user

Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.












## User - remove

```javascript
initEDMdesignerPlugin("TestAdmin", function(edmDesignerApi) {
  edmDesignerApi.createUser({id: "exampleUserId"}, function(result) {
  
    edmDesignerApi.deleteUser(result.id, function(resultUser) {
      //the resultUser is an object with an id propert
      console.log(resultUser);
    }, onErrorCB);
  
  }, onErrorCB);
});

function onErrorCB(error) {
  console.log(error);
}
```

### edmDesignerApi.deleteUser(userId, callback, onErrorCB)
Deletes a specified user
#### Parameters:
  * userId {String} The id of the user.
  * callback {Function} A function to be called if the request succeeds
  * onErrorCB {Function} A function to be called if the request fails

### Delete user
Deletes a specified user

#####Type
  + DELETE

#####Route
  + //api.edmdesigner.com/json/user/delete/:id
  
#### Parameters (in the route):
   * :id {String} The id of the user.

####Response:
User object:
  - id {String} The id of the deleted user

Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.





___

## Project - getOne

```javascript
initEDMdesignerPlugin("TestAdmin", function(edmDesignerApi) {
  edmDesignerApi.createProject({title: "test-title", description: "test-desc"}, function(result) {
    edmDesignerApi.getProject(result._id, function(result) {
      console.log(result);
      //the result is the whole project
      // _id {String} The id of the project
      // title {String} The title of the project.
      // description {String} Description about the project
      // createdOn {String} Creation time
      // lastModified {String} Time of the last modification
      // document {Object} The json format of the template
          // usedColors {Array} List of the colors which are used in the template
         // generalSettings {Object} The default settings of the template
         // root {Object} The actual structure of the template
         // header {Object} The header of the template
         // footer {Object} The footer of the template 
      // customData {Object} Your custom data
    });
  });
}, onErrorCB);

function onErrorCB(error) {
  console.log(error);
}
```

###edmDesignerApi.getProject(projectId, callback, onErrorCB)
Get a specified project as json. The result will contain almost every information about the selected project.
#### Parameters:
  * projectId {String}  The id of the project. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list the projects of the user with the [edmDesignerAPI.listProjects](#edmdesignerapilistprojectscallback-onerrorcb) function.
  * callback {Function} A function to be called if the request succeeds
  * onErrorCB {Function} A function to be called if the request fails






















#Gallery handling

TODO talk about disabling the built-in gallery

If you want to host the uploaded images yourself and want to use your other hosted images as well, then there are a few routes to fulfil this functionality.

Basic operations: The user uploads an image in the api to our server, which uploads it to the given server. This requires you to implement an upload route on your server and [configure](#configure-api-servers-gallery) our server (you have to do the configuration only once). The user can delete the images as well, so there should be an delete route on your server (it should be in the [gallery configuration](#configure-api-servers-gallery)) .

We will create a md5 hash from a timestamp and your magic word (The right order of the string concatenation: timestam + magic word). This two (the hash and the timestamp) will be sent to you in each of our requests (in the query). You recreate the hash in your server side and compare the two hashes. With this method you can ensure that the request really came from us.


## Upload route
You should implement a route on your server which we can use for image upload.

Type: POST

We will upload the images as the following. There will be "userId", "hash" and "time" fields in the query of the request:
  - The "userId" will contain the id of the user (who wants to upload the image).
  - The "hash" will be a string we use for authenticate ourself. We concatenate timestamp with your magic word (the right order of the concatenation: timestam + magicword) and create an md5 hash from it. With this hash you can ensure that the request came from us. 
  - The "time" will be the timestamp we used for creating the hash 

The image will be uplaoded as a file (the field name will be "file") and there will be another field "userId", it will contain the same data as the query field "userId". There is a third field, called "originalFileName" which contains the name of the original file before the user started to upload it.  
Please note that if you use php on your server side then checking the content-tpye of the uploaded image with the $_Files["file"]["type"] is not possible (it will be Application/octet-stream) but if you use our hash to authenticate us, then you do not have to worry about the uploaded file. We will make sure that it will be what it has to be.

If you progress our request by php, reading our sent data may need a special way like the example below: 

  $json = file_get_contents("php://input");
  $values = json_decode($json, true);

  $someValue = $values["someValue"];

Further info about this issue can be read in this article: 
http://stackoverflow.com/questions/11990767/post-empty-when-curling-json

  
The response to our post request should be a json object. The object should have the following properties:
  * url {String} /REQUIRED/ The url where the newly uploaded image can be found.
  * secure_url {String} Http secure version og the image url
  * thumb_url {String} An url where the thumbnail version of the image is available (Tha gallery can work a lot faster if you can provide a thumbnail)
  * name {String} The name of the image. If it is not given then we will you the last segment of the url
  * width {Number} The original width of the image (It can save a lot of process if you can provide this information)
  * height {Number} The original height of the image (It can save a lot of process if you can provide this information)

Or if you want to response with some kind of error (for example an error message), then you have to send an error object with the following property:
  * err

For example:  
  var error = {
    err: "Not my user!"
  };


## Delete route
You should implement a route on your server side which we can use for deleting images.

Type: POST

There will be "hash" and "time" fields in the query of the request:
  - The "hash" will be a string we use for authenticate ourself. We concatenate timestamp with your magic word (the right order of the concatenation: timestam + magicword) and create an md5 hash from it. With this hash you can ensure that the request came from us.
  - The "time" will be the timestamp we used for creating the hash 

We will post an object with two parameters:
  * url {String} The url of the deleted image
  * userId {String} The id of the user who deleted the image.

If you progress our request by php, reading our sent data may need a special way like the example below: 

  $json = file_get_contents("php://input");
  $values = json_decode($json, true);

  $someValue = $values["someValue"];

Further info about this issue can be read in this article: 
http://stackoverflow.com/questions/11990767/post-empty-when-curling-json

  
Your response to our post request should be 200. (HTTP status code)  
Or if you want to response with some kind of error (for example an error message), then you have to send an error object with the following property:
  * err

For example:  
  var error = {
    err: "Not my user!"
  };


## Copy route
IF you want to copy a project from a user to an other and want to copy the images (which are used in the project) too, you need to use the [create from to with images route](#creat-from-to-with-images). This route will collect the urls of the images (which you host) of the project and send it to you. After that you should make a copy of this images to the target user and sent us back the parameters of this new images.

Type: POST  

There will be "hash" and "time" fields in the query of the request:
  - The "hash" will be a string we use for authenticate ourself. We concatenate timestamp with your magic word (he right order of the concatenation: timestam + magicword) and create an md5 hash from it. With this hash you can ensure that the request came from us. 
  - The "time" will be the timestamp we used for creating the hash 


We will post an object whit the following parameters:
  * urls {Array} It contains the urls of the images which are used in the template
  * from_userId {String} The id of the users from who we want to copy the template to the target user
  * target_userId {String} The id of the user who will get the new template. (This user needs the new urls)


If you progress our request by php, reading our sent data may need a special way like the example below: 

  $json = file_get_contents("php://input");
  $values = json_decode($json, true);

  $someValue = $values["someValue"];

Further info about this issue can be read in this article: 
http://stackoverflow.com/questions/11990767/post-empty-when-curling-json

  
Your response to our post request should be an object. The object should have an "urls" property which should be another object. This object should have the following "key - value" pairs: the old url (which you get from the urls array we sent with the post request) should be the key and the value should be the parameters of the new image (like the one you need to send beack in the [upload route](#upload-route):
  * url {String} /REQUIRED/ The url where the newly uploaded image can be found.
  * secure_url {String} Http secure version og the image url
  * thumb_url {String} An url where the thumbnail version of the image is available (Tha gallery can work a lot faster if you can provide a thumbnail)
  * name {String} The name of the image. If it is not given then we will you the last segment of the url
  * width {Number} The original width of the image (It can save a lot of process if you can provide this information)
  * height {Number} The original height of the image (It can save a lot of process if you can provide this information)

For example: 
If we uploaded the following array: ["www.foo.bar", "http://test.com/image.jpg"] then your asnwer should be something like this object:
  
  var answer = { 
    urls: {
      "www.foo.bar": {
        url: "www.copy.of.foo.bar"
      },
      "http://test.com/image.jpg": {
        url: "http://test.com/copyimage.jpg",
        secure_url: "https://test.com/copyimage.jpg",
        thumb_url: "http://test.com/thumb/copyimage.jpg",
        name: "copyimage.jpg",
        width: 600,
        height: 200
      }
    }
  };

Or if you want to response with some kind of error (for example an error message), then you have to send an error object with the following property:
  * err

For example:  
  var error = {
    err: "Not my user!"
  };


##Configure api server's gallery
Configure the api's gallery.

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/gallery/config
  
#### Parameters (you should post):
  * uploadRoute {String} /REQUIRED/ The [route](#upload-route) which we can use for uploading the images. (This route should be implemented on your server.)
  * deleteRoute {String} /REQUIRED/ The [route](#delete-route) which we can use for deleting the images. (This route should be implemented on your server.)
  * copyRoute {String} The [route](#copy-route) which we can use for the [create from to width images route](#create-from-to-width-images) (If you want to use this functionality, you need to implement this route on your server)
  * noUploadText {Object} It contains the language code (see [available languages](#available-languages)) - html(string) pairs. The given content will be displayed if the upload is not allowed. 
  * limitReached {Object} It contains the language code (see [available languages](#available-languages)) - html(string) pairs. The given content will be displayed if a user reached the upload limit. It is not required to impose a limit on the upload. If you want to know how you can set the limit, please read the [feature switch](#feature-switch) chapter.
  * noUrl {Boolean} With this boolean you can forbid the image handling from absolute urls. If you set this parameter true, then none of your users will be able to use images from absoulte url.

####Response:
The data you posted:
  - uploadRoute {String}
  - deleteRoute {String}
  - copyRoute {String}
  - noUploadText {Object}
  - limitReached {Object}
  - noUrl {Boolean}
or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

##Get configuration
Gets the configuration of the api gallery.

#####Type
  + GET

#####Route
  + //api.edmdesigner.com/json/gallery/config

####Response:
Config object: 
  - uploadRoute {String} The [route](#upload-route) you set for the uploading or nothing if you not configured our server yet.
  - deleteRoute {String} The [route](#delete-route) you set for the deleting or nothing if you not configured our server yet.
  - copyRoute {String} The [route](#copy-route) you set for the [create from to width images route](#create-from-to-width-images) (only if you previously set it)
  - noUploadText {Object} (only if you previously set it in)
  - limitReached {Object} (only if you previously set it)
  - noUrl {Boolean} (only if you previously set it true)

or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

##Set the galleryStamper's url
Sets the url of the galleryStamper. If this mode is enabled, the API call this route for image stamping, e.g. in case of video play icon.

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/gallery/galleryStamperUrl

#### Parameters (you should post):
  - url {String} The url to set.

####Response:
Config object: 
  - url {String} The set url

or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

##Get the galleryStamper's url
Gets the url of the galleryStamper. If this mode is enabled, the API call this route for image stamping, e.g. in case of video play icon.

#####Type
  + GET

#####Route
  + //api.edmdesigner.com/json/gallery/galleryStamperUrl

####Response:
Config object: 
  - url {String} The set url

or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

##List images
Lists all images of a specified user

#####Type
  + GET

#####Route
  + //api.edmdesigner.com/json/gallery/list/:id

#### Parameters (optionals):
  - skip {Number} index where from the listing should begin, without limit parameter it will be ingnored
  - limit {Number} length of the items to be listed, without skip parameter it will be ingnored
  - select [Array] list of the properties to be listed ["name", "created"]
  - sort {Object} MongoDb sort object: {property : "asc/desc"}

####Parameters (in the route):
  * :id {String} The id of the target user

####Response:
Array of urls

Or in case of of limit and skip parameters sent an Object consists of:
  - totalCount {Number} The length of the full list
  - result {Array} The list of the urls, same as above

or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

##Add images
Add one or more images to one or more specified users

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/gallery/add

####Parameters (you should post):
  * images {Array} /REQUIRED/ should contain the images you want to give to the users. Every image should be an object with the following parameters:
    * url {String} /REQUIRED/ The url of the images (you probably host)
    * secure_url {String} Http secure version og the image url
    * thumb_url {String} An url where the thumbnail version of the image is available (Tha gallery can work a lot faster if you can provide a thumbnail)
    * name {String} The name of the image. If it is not given then we will you the last segment of the url
    * width {Number} The original width of the image (It can save a lot of process if you can provide this information)
    * height {Number} The original height of the image (It can save a lot of process if you can provide this information)
  * users {Array} /REQUIRED/ should contain the ids of the users  

####Response:
Three different array:
  - added {Array} contains the image user pairs which were successfully created. An object in this array looks like this:
    - image {Object} One image from the posted images array
    - users {Array} An array containing the ids of the users whose get the image successfully
  - alreadyHave {Array} contains the image user pairs where the user alread had the given image. An object in this array looks like this:
    - image {Object} One image from the posted images array
    - users {Array} An array containing the ids of the users whose already had this image
  - failed {Array} contains the image user pairs where the adding the image to the given users failed. An object in this array looks like this:
    - image {Object} One image from the posted images array
    - users {Array} An array containing the ids of the users whose cannot get the image because of somekind of error
    - error {String} Description of the error

or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

##Delete images
Delete one or more images from one or more specified users

#####Type
  + DELETE

#####Route
  + //api.edmdesigner.com/json/gallery/delete

####Parameters (you should post):
  * images {Array} /REQUIRED/ should contain the urls of the images you want to remove from the given users.
  * users {Array} /REQUIRED/ should contain the ids of the users.

####Response:
Three different array:
  - deleted {Array} contains the image user pairs where the image was successfully deleted. An object in this array looks like this:
    - image {Object} One image from the posted images array
      - url {String} The url of the image 
    - users {Array} An array containing the ids of the users whose
  - dontHave {Array} contains the image user pairs where the user doesn't have the given image. An object in this array looks like this:
    - image {Object} One image from the posted images array
      - url {String} The url of the image 
    - users {Array} An array containing the ids of the users whose doesn't have this image
  - failed {Array} contains the image user pairs where the deleting the image from the given users failed. An object in this array looks like this:
    - image {Object} One image from the posted images array
    - users {Array} An array containing the ids of the users whose image cannot be deleted because of somekind of error
    - error {String} Description of the error

or it can be an error object:
  - err Description of the error {String} or an error code {Number}.


#Available Languages
You can set the localization which the api will use. (if you want to know how, please check the [edmDesignerApi.openProject](#edmdesignerapiopenprojectprojectid-languagecode-settings-callback-onerrorcb) function!)  
Available languages:
 - English (code: 'en')
 - Hungarian (code: 'hu')


If the language you want to use is not available yet then please contact us with the following email address: info@edmdesigner.com


























  

