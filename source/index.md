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

If you are feeling confused or stuck during the process, check out [Tutorials](#tutorials).



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

At the moment, our [JS API](http://api-static.edmdesigner.com/EDMdesignerAPI.js)'s dependence is [jQuery](https://jquery.com/) (v1.3 or later). It's only because of the convenient ajax requests, but probably we will change it to native ajax in the near future. 

We suggest you to use the [JS API](http://api-static.edmdesigner.com/EDMdesignerAPI.js) as much as it's possible.


We chose PHP as the language for the server side examples, because most of the people can at least read it.



#Authentication

In this chapter you will learn about the authentication and authorization process in our API.
Basically you will need to generate a user level or admin level access token with which you can make API calls.


## Generating an access token

First of all, you will need an API key and a corresponding magic word to generate an access token. You can manage your API keys and magic words on our dashboard.

Basically you need to HTTP POST from your server to one of our [API endpoints](#making-requests) the following data to the **/api/token** route:

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

**http(s)://api.end.point/request/route?user=youruserid&token=youraccesstoken**

For exmaple:

https://api-a.edmdesginer.com/json/project/list?user=userId&token=123456789

If you use our [JS API](http://api-static.edmdesigner.com/EDMdesignerAPI.js) then you won't have to manually tinker with concatenating the correct request urls, the JS API module will do it for you. You will just need to call the corresponding functions. (You will find these functions as well later in the documentation.)

The other very good thing about using this JS API is, that the failover mechanism is built-in so you won't need to implement failover manually.

If the token is not valid any more or the userId that you sent is not correct, then the HTTP response's status code will be [403](https://en.wikipedia.org/wiki/HTTP_403) and your request won't be served.

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
 - token.php sends the necessary info to one of our endpoints
 - our endpoint responds with a token
 - JS API will use this token for the subsequent API calls

There is a special user associated to each API key. It's name is templater and its projects are the default projects in the system. It is always existent and cannot be deleted. The [list defaults route](#project-list-defaults) will always respond with the templater's projects. (Our concrete example on the right is generating a token for the templater user.)






## Admin level access token

```javascript
//First include jQuery somewhere in your page, because our JS API is depending on that
//Use the JS API from: https://api-static.edmdesigner.com/EDMdesignerAPI.js
//A concrete example with token.php as the handshaking route and admin as the user
//Be very careful, authorize your users on the server side very carefully!!!
initEDMdesignerPlugin("token.php","admin", function onSuccess(edmDesignerApi) {
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

The generation of an admin level access token is almost the same as generating a regular [user level access token](#user-level-access-token). The only difference is that you have to use the string "admin" as the user. This is not an existent user in our system, we just use it to generate an admin level access token.

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
Throughout the documentation you will be reading about requests with which you can make certain operations on projects. Read more about project management [here](#project-handling). These projects have the following properties:

Field | Type | Required | Description
------|------|----------|------------
_id | ObjectId | true | The MongoDB id of the project. (Autogenerated, so if you create a project, you don't have to give it.)
user | ObjectId | true | A reference to the owner of the project.
title | String | true | The title of the project.
description | String | false | Optional extra information.
createdOn | Date | false | The time when the project was created at.
lastModified | Date | false | The time when the project was modified for the last time.
document | Mixed | false | The document descriptor object. Read more about [here](documentDescriptors.html).
customData | Mixed | false | A custom object in which you can put extra info of the project. For example if it's a template or a campaign.






## The user model

```javascript

var mongoose      = require("mongoose");

var Schema = mongoose.Schema,
  ObjectId = Schema.ObjectId;

// User schema
var userSchema = new Schema({
  id: {type: String, required: true},
  group: {type: ObjectId, ref: "Groups"},

  createTime: {type: Date, default: Date.now},

  customData: {type: Schema.Types.Mixed}
});

userSchema.id({username: 1});

module.exports = mongoose.model("User", userSchema);
```


```php
//Switch to the javascript tab to see the original mongoose schema and model of our Users.
```

You will need to associate your users with ours. The best was is to use your own userIds as username in our system, so the association will be trivial.
It's best if you create a user in our system at the first time when your user tries to open our editor. Read more about user management [here](#user-handling).

Field | Type | Required | Description
------|------|----------|------------
id | String | true | This will be the id of the user, so you must make it sure, that it's unique per API key. We suggest you to use the userId from your system.
group | ObjectId | true | A reference to the user's group. (Autogenerated.)
createdTime | Date | false | The time when the user was created at.
customData | Mixed | false | A custom object in which you can put extra info of the user.


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
initEDMdesignerPlugin("token.php", "TestUser", function(edmDesignerApi) {
  //In the following examples we assume that you put the code snippets here,
  //where edmDesignerApi is on the closure.
}, function onError(error) {
  console.log(error);
});
```
The following functionalities are reachable with a [user level access token](#user-level-access-token).

Read more about the project model [here](#the-project-model).

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








## Project - getOne

Get one project by _id.

```javascript
initEDMdesignerPlugin("token.php", "TestUser", function(edmDesignerApi) {
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

Get a specified project as json. The result will contain almost every information about the selected project.

HTTP | JSONP | JS API
-----|-------|-------
GET /json/project/getProject/:id | GET /jsonp/project/getProject/:id | edmDesignerApi.getProject(projectId, callback, onErrorCB)

### Parameters:

Field | Type | Required | Description
------|------|----------|------------
:id | String | true | The id of the project you want to get.

### Response:
The project as a json. Read more about the project model [here](#the-project-model).






## Project - list defaults

```javascript
edmDesignerApi.getDefaultTemplates(function(result) {
  //the result is an array, containing your default projects
}, function onError(error) {
  console.log(error);
});
```
You can get the default templates provided by your templater user by calling this function. It means that if you have a user called templater, this route will result its projects. (Every API key has an associated templater user by default.)

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

There are limitations of the JSONP technique you should know about. One limitation is that the client browser will always issue HTTP GET requests to the server, and another is that HTTP error status codes cannot be used to inform the client of the success or failure of an operation.

This is because if the server’s response has a status code other than 200 OK, the client will not load the received Javascript code, and the callback function will never be called. Read more about JSONP [here](https://en.wikipedia.org/wiki/JSONP).



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
  //the result is the newly created project's object or error object with err property
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
    
initEDMdesignerPlugin("token.php", "templater", function(edmDesignerApi) {
  edmDesignerApi.createProject({title: "test-title", description: "test-desc"}, function(result) {
    templaterProjectId = result._id;
  });
}, onErrorCB);


//later somewhere in the code...
initEDMdesignerPlugin("token.php", "TestUser", function(edmDesignerApi) {

  edmDesignerApi.createFromDefaults(templaterProjectId, {title: "createFromDefaults example", description: "Created with createFromDefaults function"}, function(result) {
    //the result is the newly created project's object or error object with err property
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
initEDMdesignerPlugin("token.php", "TestUser", function(edmDesignerApi) {
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

Updates the title or/and the description of the specified project. You can add custom informations to this project too.

HTTP | JSONP | JS API
-----|-------|-------
POST /json/project/updateInfo | GET /jsonp/project/updateInfo | edmDesignerApi.updateProjectInfo(projectId, data, callback, onErrorCB)




### Parameters:

Field | Type | Required | Description
------|------|----------|------------
data | [Project](#the-project-model) | true | You don't need to set the document field, because that won't be updated if you use this route
data.override | Boolean | false | If true, then the customData will be overridden with the new one. If false, then it will be merged into the existing one. The default value of it is false.

### Response:
{success: true} or it can be an error object:
  - err Description of the error {String} or an error code {Number}.










## Project - remove

```javascript
initEDMdesignerPlugin("token.php", "TestUser", function(edmDesignerApi) {
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
DELETE /json/project/remove/:id | GET /jsonp/project/remove/:id | edmDesignerApi.removeProject(projectId, callback, onErrorCB)

### Parameters:

Field | Type | Required | Description
------|------|----------|------------
:id | Param in url | true | The id of the project to remove. Example: /json/project/remove/32


### Response:
A number, it is 1 if the project was successfully deleted

or it can be an error object:
  - err Description of the error {String} or an error code {Number}.














## Project - generate HTML

```javascript
initEDMdesignerPlugin("token.php", "TestUser", function(edmDesignerApi) {
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

HTTP | JSONP | JS API
-----|-------|-------
GET /json/project/generate/:id | GET /jsonp/project/generate/:id | edmDesignerApi.generateProject(projectId, callback, onErrorCB)

### Parameters:

Field | Type | Required | Description
------|------|----------|------------
:id | ProjectId | true | The id of the project to generate html from. This parameter is the part of the route.


### Response:
A bulletproof responsive HTML version of the given template

or it can be an error object:
  - err Description of the error {String} or an error code {Number}.



## Project - generate without sanitizing
Generates the responsive HTML e-mail based on the projectId. The generated html won't be sanitized. (good for those, who wants to use complex placeholders the normal [generate route](#generate) would erase. You should call this route from your server ONLY! You have to take care about sanitizing if you publish the HTML on the web. We recommend to you to use [Google Caja](https://code.google.com/p/google-caja/wiki/JsHtmlSanitizer).

There is no JSONP version of this route, so it's not reachable from the JS API either.

### Route
  GET /json_v1.0.0/apiKey/:apiKey/user/:userid/project/:projectid/generateWithoutSanitizing

### Parameters (in the route):
  * :apiKey {String} The key of the instance the target user belongs to
  * :userId {String} The id of the project's owner. Plesase note that this id should be the same you used for the token generation.
  * :projectid {String} The id of the project. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list the projects of the user with the [/json/project/list](#list-projects) route.

### Response
A not sanitized responsive HTML version of the given template

or it can be an error object:
  - err Description of the error {String} or an error code {Number}.









## Project - export HTML

Please see [generate HTML](#project-generate-html).

## Project - preview

To create a preview you only need to use the [generate HTML](#project-generate-html) route as the src attribute of an iframe. If you want to simulate different devices, just set the width of the iframe properly. (Eg. 400px wide for smartphones, 1000px wide for desktop...)

For further info, see [generate HTML](#project-generate-html).


## Project - Open

```javascript
initEDMdesignerPlugin("token.php", "TestUser", function(edmDesignerApi) {
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

This function is only a wrapper, which will create a correct iframe src from the params. For detailed information about setting up the iframe's src, please see the [next chapter](#editor-iframe).

**edmDesignerApi.openProject(projectId, [languageCode], [settings], callback, onErrorCB)**

### Parameters:

  * projectId {String} The id of the project. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list the projects of the user with the [edmDesignerAPI.listProjects](#edmdesignerapilistprojectscallback-onerrorcb) function.
  * languageCode {String} /Optional/ A two character [ISO 639-1 code](http://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) of a selected language. With this parameter you can choose the language which the opened project will use. You can find the [list of the available languages](#available-languages) at the end of this page. Default language is english.
  * settings {Object} /Optional/ With this object, it is possible to set some basic settings:
    * autosave {Number} With this property you can set the frequency of the autosave (in millisecond). Please note that the autosave only save the project if there were changes after the last save, with this property you can only set how often should the autosave handler check if there are any kind of changes in the template. If you want to turn off the autosave functionality, then you have to set this property to zero (0). It is possible to manually trigger the saving mechanism with the [save project message](#save-project).
  * callback {Function} A function to be called if the request succeeds
  * onErrorCB {Function} A function to be called if the request fails

The callback will be called with an object on which there are two properties:

 - src: the correct src of the iframe
 - iframe: an iframe which you can already insert to the dom





# Editor iframe

If you want to open a project in our editor, you just have to put an iframe into your page with the correct src. If you don't want to tinker with creating the correct url manually, then you can use the function defined in the [previous section](#project-open).

Here is a real life example for the src:

**https://api-static.edmdesigner.com/?user=YOURUSERID&projectId=YOURPROJECTID&token=YOURTOKEN&language=en&autosave=1000**

Param | Required | Description
---|---|---
user | true | The id of the user.
projectId | true | The project you want to open.
token | true | The [access token of the user](#user-level-access-token)
language | false | The [language](#available-languages) of the editor. (It defaults to "en".)
autosave | false | The number of milliseconds to spend between the last change and the autosave. If it's 0, then the autosave is disabled. (The default value is 1000 and it cannot be lower than 1000.)

As you can see, that the editor is served from api-static.edmdesigner.com. Everything on this subdomain is static and served by [Amazon's CDN (Cloudfront)](http://edmdesigner.com/blog/update-edmdesigner-uses-amazon-cloudfront). This means that the editor will load fast all over the world. It makes API calls to our endpoints with build in failover mechanism, so it is very, very robust as well.








# User handling

```javascript
initEDMdesignerPlugin("token.php", "admin", function(edmDesignerApi) {
  edmDesignerApi.listGroups(function(result) {
    //the result is an array, containing your groups
  }, onErrorCB);
});

function onErrorCB(error) {
  console.log(error);
}
```

The following functionalities are reachable with an [admin level access token](#admin-level-access-token).

Read more about the user model [here](#the-user-model).







## User - list

```javascript
//do not forget, that you have to use an admin level access token!
edmDesignerApi.listUsers(function(result) {
  //the result is an array, containing your users
  //A user is an object with
  //id (the user's id)
  //group (The group which the user belongs) and
  //createTime (Time of creation) properties
  //customData (The custom informations you saved for the user)
  console.log(result);
}, onError(error) {
  console.log(error);
});

function 
```

Lists the users associated to the API key you are using.

HTTP | JSONP | JS API
-----|-------|-------
GET /json/user/list | GET /jsonp/user/list | edmDesignerApi.listUsers(callback, onErrorCB)

### Parameters:

Field | Type | Required | Description
------|------|----------|------------
query | [Query Object](#the-mongoose-query-object) | false | A mongoose query object.


### Response:
An array of your [users](#the-user-model).

Or in case of of limit and skip parameters sent an Object consists of:

  - totalCount {Number} The length of the unfiltered list
  - result {Array} The list of the [users](#the-user-model], same as above

Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.














## User - create

```javascript
var groupId = ""
edmDesignerApi.createUser({id: "exampleuserId", group: result._id}, function(resultUser) {
  // the resultUser is an object with an id property
  console.log(resultUser);
}, function onErrorCB(error) {
  console.log(error);
});
```

Creates a new user

HTTP | JSONP | JS API
-----|-------|-------
POST /json/user/create | GET /jsonp/user/create | edmDesignerApi.createUser(data, callback, onErrorCB)

### Parameters:

Field | Type | Required | Description
------|------|----------|------------
data | [User](#the-user-model) | true | The only required property of it is the id.


###Response:
User object:

  - id {String} The id of the user (your id, not the MongoDB _id)

Or it can be an error object:

  - err Description of the error {String} or an error code {Number}.










## User - get one

```javascript
initEDMdesignerPlugin("token.php", "admin", function(edmDesignerApi) {
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

Gets a specified user


HTTP | JSONP | JS API
-----|-------|-------
GET /json/user/getOne/:id | GET /json/user/getOne/:id | edmDesignerApi.getUser(userId, callback, onErrorCB)

### Parameters:
Field | Type | Required | Description
------|------|----------|------------
:id | String | true | The id of the user. (Which was given by you previously at [user creation](#user-create).)

### Response:
A [user](#the-user-model) object.

Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

















## User - update

Updates a specified user. The group (which the user belongs) can be changed. You can add custom informations to the user too.

```javascript
initEDMdesignerPlugin("token.php", "admin", function(edmDesignerApi) {
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

HTTP | JSONP | JS API
-----|-------|-------
GET /json/user/getOne/:id | GET /json/user/getOne/:id | edmDesignerApi.updateUser(userId, data, callback, onErrorCB)

### Parameters:
Field | Type | Required | Description
------|------|----------|------------
:id | String | true | The id of the user. (Which was given by you previously at [user creation](#user-create).)
data | [User](#the-user-model) | true | The new user data
data.override | Boolean | false | If it is true then the customData will be overridden with the newly given data, if it is false then the newly given customData will be merged with the previous ones. By default it is false.


### Response:
The user object.








## User - remove

```javascript
initEDMdesignerPlugin("token.php", "admin", function(edmDesignerApi) {
  edmDesignerApi.createUser({id: "exampleUserId"}, function(result) {
  
    edmDesignerApi.deleteUser(result.id, function(resultUser) {
      //the resultUser is an object with an id property
      console.log(resultUser);
    }, onErrorCB);
  
  }, onErrorCB);
});

function onErrorCB(error) {
  console.log(error);
}
```

Deletes a specified user.

HTTP | JSONP | JS API
-----|-------|-------
DELETE /json/user/delete/:id | GET /jsonp/user/delete/:id | edmDesignerApi.deleteUser(userId, callback, onErrorCB)

### Parameters:

Field | Type | Required | Description
------|------|----------|------------
:id | String | true | The id of the user. (Which was given by you previously at [user creation](#user-create).)

### Response:
User object:

  - id {String} The id of the deleted user

Or it can be an error object:

  - err Description of the error {String} or an error code {Number}.


# Gallery handling

```javascript
window.addEventListener("message", function(event) {
  if (event.origin !== "https://api-static.edmdesigner.com") {
    return;
  }

  var msg = {};
  try {
    msg = JSON.parse(event.data);

    if (msg.action === "edit" && msg.elementJson.type === "IMAGE") {
      event.source.postMessage(JSON.stringify({
        _dynId: msg._dynId,
        action: "setProps",
        props: {
          src: "http://upload.wikimedia.org/wikipedia/en/2/24/Lenna.png"
        }
      }), event.origin);
    } else if (msg.action === "editBackground" && (msg.elementJson.type === "BOX" || msg.elementJson.type === "BUTTON" || msg.elementJson.type === "GENERAL")) {
      event.source.postMessage(JSON.stringify({
        _dynId: msg._dynId,
        action: "setProps",
        props: {
          background: {
            image: {
              src: "http://upload.wikimedia.org/wikipedia/en/2/24/Lenna.png"
            }
          }
        }
      }), event.origin);
    }
  } catch(e) {
    //do nothing
  }
}, false);
```

Basically there are two ways to use gallery in our editor:

 - you can use the built-in gallery
 - or you integrate our editor with your own gallery.


We suggest you to use your own gallery for several reasons:

 - The first is that if you use our built-in gallery, then the images first will be uploaded to our server and then we post it to your server which you set up on the dashboard. This way the upload speeds can be slower radically.
 - The second reason is that you have to keep our and your gallery in synchron, which is probably takes much more time then using your already existing gallery from your own system.


 If you want to use your own gallery, then we have to disable the gallery for your API keys. Log in to the [dashboard](https://dashboard.edmdesigner.com/login) and go to editor settings. At the runtime modifiers part, you can disable the built-in gallery. Be careful, try it first with one of your test api keys. 

 When the gallery is disabled, you can hook on editor events which we send through parent.postMessage. In the example on the right hand side (javascript tab) we hook on every image and background related edit event. What we do is basically we immediately set the src property or the background src whenever we get an "edit" or "editBackground" event. We do it with the setProps action. You have to send back the _dynId of the element, so the editor will update the property of the same element the user started to edit.

 If you want to pop-up your own gallery, you need to do that in the if parts and you can send a postMessage back to the iframe when your user picked the image he/she wanted. This way you can use your own gallery or even your favorite image manipulator.


If you still want to use our built-in gallery, you can read its [docs](./builtInGallery.html) and a [tutorial](./tutorialGalleryConfig.html) about it.



# Code element handling

```javascript
var placeholderJson = { type: "BOX", ... }

window.addEventListener("message", function(event) {
  var msg = {};

  try {
    msg = JSON.parse(event.data);
  } catch(e) {
    return; // do nothing
  }

  if (msg.action === "edit" && msg.elementJson.subType === "CODE") {
    event.source.postMessage(JSON.stringify({
      _dynId: msg._dynId,
      action: "setProps",
      props: {
        customData: { code: "YOUR CODE", disabledCode: false },
        root: placeholderJson
      }
    }), event.origin);
  }
}, false);
```

With the Code element, you can insert custom code into your templates.

### Properties:

Field | Type | Required | Description
------|------|----------|------------
customData | object | true | Options for the Code element
root | object | false | You can overwrite the Code element's placeholder with a Template JSON object. The root element's type has to be [BOX](./documentDescriptors.html#box) or [MULTICOLUMN](./documentDescriptors.html#multicolumn)

### customData properties:

Field | Type | Required | Description
------|------|----------|------------
code | string | true | Your custom code, which will be inserted into the template by default
disabledCode | boolean | false | If you set this option true the exported template will include the placeholder instead of your custom code


# Header script

```javascript
window.addEventListener("message", function(event) {
  try {
    var msg = JSON.parse(event.data);
  } catch(e) {
    return; // do nothing
  }

  if (msg.action === "editHeaderScript") {
    event.source.postMessage(JSON.stringify({
      _dynId: msg._dynId,
      action: "setProps",
      props: {
        headerScript: "<script>console.log('Hello')</script>"
      }
    }), event.origin);
  }
}, false);
```

With Header script, you can insert any valid HTML code into the template's `<head>` tag. For example you can use this feature to place JavaScript code wrapped up in a `<script>` tag. (But do not forget that majority of email softwares will sanitize `<script>` tags)

### Properties:

Field | Type | Required | Description
------|------|----------|------------
headerScript | string | true | valid-HTML code









#Available Languages

At the moment we support the following languages:

 - de (German)
 - en (English)
 - fr (French)
 - es (Spanish)
 - hu (Hungarian)
 - it (Italian)
 - nl (Dutch)
 - pl (Polish)
 - pt (Portuguese)
 - pt-BR (Portuguese [Brazilian])
 - ro (Romanian)
 - ru (Russian)
 - zh-HANS (Chinese [Simplified])

If the language you want to use is not available yet then please contact us with the following email address: info@edmdesigner.com




#Supported browsers

 - Internet Explorer 10+
 - Google Chrome 33+
 - Mozilla Firefox 27+
 - Opera 19+
 - Safari 5+

























  

