---
title: API Reference

language_tabs:
  - javascript
  - php

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='http://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

#Overview

[EDMdesigner](http://www.edmdesigner.com) is a drag and drop tool for creating responsive HTML e-mail templates very quickly and painlessly, radically increasing your click-through rate. This documentation is to give you a detailed description about the EDMdesigner-API with which you can integrate our editor into any web-based system (e.g. a CRM, CMS, WebShop or anything else you can imagine).


To start developing with our API, please request access to our dashboard, where you can manage your API keys. (Please contact us at info@edmdesigner.com.)

With an API key and the corresponding "magic word" you can generate an access token with which you can make request to our API. You will need to send the access token with each API requests to our servers.

At a certain point your users will open our editor. When the editor is opened, you will be able to send to its iframe postMessages. That's the client side of our API. The whole code (js, css and html as well) of the editor is [served with Amazon's CDN (Cloudfront)](http://edmdesigner.com/blog/update-edmdesigner-uses-amazon-cloudfront) so it should load super fast from all over the world. You can find the client side api in a different documentation. (TODO LINK)

Also, you can find a documentation about our document format in which we store our templates.

This documentation is only about the requests sent to our servers.


****************************************
Client side API - todo
****************************************

****************************************
Documentation of our document format - todo
****************************************


#Making requests

General stuff. Most of our api routes are reachable with a http and a jsonp request. Some of them are not, because we consider them as dangerous operations so they are only reachable via simple http requests and they should be called only from your servers.

With failover mechanism...
 - a, b, c

403

JS API - failover is built in.
Node.js lib.
PHP lib.

HTTP
JSONP
JS API
Dependency: jQuery

We suggest you to use our JS API as much as it's possible.


Our server side examples are in PHP because most of the people can at least read it.




#Authentication

```javascript
  //<script src="path_to_your_jquery.js"></script>
  //<script src="http://api.edmdesigner.com/EDMdesignerAPI.js?route=##handshaking_route##"></script>
  //<script>
    initEDMdesignerPlugin("##userId##", function onSuccess(edmDesignerApi) {
      ...
    }, function onError(error) {
      console.log(error);
    });
  //</script>
```


```php
//token.php
<?php
if ($_POST["userId"]) {
  $publicId = "TESTAPIKEY";
  $magic = "TESTMAGICWORD";
  $ip = $_SERVER["REMOTE_ADDR"];
  $timestamp = time();
  $hash = md5($publicId . $ip . $timestamp . $magic);
  $url = "http://api-a.edmdesigner.com/api/token"; //could be api-b & api-c as well.
  $data = array(
        "id"  => $publicId,
        "uid" => $_POST["userId"],
        "ip"  => $ip,
        "ts"  => $timestamp,
        "hash"  => $hash
  );
  // use key 'http' even if you send the request to https://...
  $options = array(
      'http' => array(
          'header'  => "Content-type: application/x-www-form-urlencoded\r\n",
          'method'  => 'POST',
          'content' => http_build_query($data),
      ),
  );
  $context  = stream_context_create($options);
  $result = file_get_contents($url, false, $context);
  print($result);
}
?>
```

The Handshaking Process...

To implement the handshaking on your server, you need an API_KEY, a magic word (wich is delivered with your API_KEY), your user's IPv4 address and a timestamp. The logic that handles handshaking has to receive the userId too, because it has to be sent to our server as well. Our API implementation automatically sends the userId in a POST HTTP request. You can set this user id by the first parameter of the initEDMdesignerPlugin.

After concatenating the API_KEY, the IPv4 address of your user, the timestamp (as a string) and your magic word, you have to create an md5 hash of the resulting string.

  
  hash = md5(API_KEY + ipv4 + timestamp + magic)


After this, you have to send the API_KEY, the userId, the IPv4 address, the timestamp and the generated md5 hash to our server at api.edmdesigner.com/api/token through HTTP.

If everything goes well, you get back a token, with which your user can be identified.

The only route which does not need a token. Actually this is the one with which you generate your access tokens. Should store them in a session. Tokens are alive for one day - stored in a redis server - their ttl is one day. (Storing them in session is only a good idea if your sessions are alive for less then one day.) We might decrease the ttl, so it's also a good idea to build in a regeneration method for tokens.


Initializing the client side api js...


First of all, jQuery has to be loaded before you load our API. In the second line you can see a route parameter in the script's src. By that you can tell the script where to look for the handshaking implementation. For example, if you implemented it in your index.php, you have to replace ##handshaking_route## with index.php. The first parameter is a user id from your system. It can be any string. The last parameter is an error callback which will be called in every request if the request fails. This is not required thus you can set an error callback in one of your function calls (see later) and in that case it will be called instead of this one.


In the resulting object (edmDesignerApi) you will find some functions through which you can interact with our system.


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
In the following sections you will be reading about requests with which you can make certain operations on projects. These projects have the follwing properties:

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
You can set the parameters of the db search. You need to place your settings object to the query of the get request. Please note that it is an optional feature. If you don't want to use it, call the route without any kind of settings (in the query) and it will send back all of the user projects!

Field | Type | Required | Description
------|------|----------|------------
find | Object | false | dfdfdf
select | Array | false | 
skip | Number | false |
limit | Number | false |
sort | Object | false |

You can set the parameters of the db search (you have to use the moongose.js query syntax, please check their [documentation](http://mongoosejs.com/docs/guide.html) for more information)


# User functions
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

___

### Get title
Gets the title of the selected project.

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


























---

# Load project in editor

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

//TODO - talk about the iframe.

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



---

#Admin functions

You have to generate an admin token!

## Group - list
### edmDesignerApi.listGroups(callback, onErrorCB)
Lists the groups you have
#### Parameters:
  * callback {Function} A function to be called if the request succeeds
  * onErrorCB {Function} A function to be called if the request fails
 

#### Example:
  
  <script>
    initEDMdesignerPlugin("TestAdmin", function(edmDesignerApi) {
      edmDesignerApi.listGroups(function(result) {
        //the result is an array, containing your groups
      }, onErrorCB);
    });
    
    function onErrorCB(error) {
      console.log(error);
    }
  </script>

### List groups
Lists the groups you have

#####Type
  + GET

#####Route
  + //api.edmdesigner.com/json/groups/list
#### Parameters (optionals):
  - skip {Number} index where from the listing should begin, without limit parameter it will be ingnored
  - limit {Number} length of the items to be listed, without skip parameter it will be ingnored
  - select [Array] list of the properties to be listed ["name", "created"]
  - sort {Object} MongoDb sort object: {property : "asc/desc"}
####Response:
An array of your groups. Every group is an object with this parameters:
  - _id {String} MongoDB id of the group
  - featureSwitch {Object} The features that are available for users belong to this group. There is an ever-expanding [list](#feature-switch) of possible features which you can choose from.
  - name {String} The name of the group
  - customData {Object} Th custom informations you saved for the group

Or in case of of limit and skip parameters sent an Object consists of:
  - totalCount {Number} The length of the full list
  - result {Array} The list of the groups, same as above

Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.










## Group - create
### edmDesignerApi.createGroup(data, callback, onErrorCB)
Creates a new group
#### Parameters:
  * data {Object}
    * data.name {String} /REQUIRED/ The name you want to give to the new group
    * data.featureSwitch {Object} The features that are available for users belong to this group. There is an ever-expanding [list](#feature-switch) of possible features which you can choose from.
    * data.customData {Object} You can add custom informations to this group. You can save any kind of information. It is up to you, how you want to use it!
  * callback {Function} A function to be called if the request succeeds
  * onErrorCB {Function} A function to be called if the request fails
 

#### Example:
  
  <script>
    initEDMdesignerPlugin("TestAdmin", function(edmDesignerApi) {
      edmDesignerApi.createGroup({name: "exampleGroup", featureSwitch: {exampleFeature1: true, exampleFeature3: true}}, function(result) {
        //the resultGroup is an object with
        //name {String} (the new group's name)
        //_id  {String} (the new group's id) and
        //featureSwitch {Object} (the group's features) properties
        console.log(result);
      }, onErrorCB);
    });
    
    function onErrorCB(error) {
      console.log(error);
    }
  </script>


### Create group
Creates a new group

#####Type
  + POST

#####Route 
  + //api.edmdesigner.com/json/groups/create

#### Parameters (you should post):
  * name {String} /REQUIRED/ The name you want to give to the new group
  * featureSwitch {Object} The features that are available for users belong to this group. There is an ever-expanding [list](#feature-switch) of possible features which you can choose from.
  * customData {Object} You can upload custom informations to this group. You can save any kind of information. It is up to you, how you want to use it!

####Response
An object containing the MongoDB _id of the newly created group:
  - _id {String}

Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.













## Group - get one
### edmDesignerApi.getGroup(groupId, callback, onErrorCB)
Gets a specified group
#### Parameters:
   * groupId {String} The id of the group. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list your groups with the [edmDesignerAPI.listGroups](#edmdesignerapilistgroupscallback-onerrorcb) function.
   * callback {Function} A function to be called if the request succeeds
   * onErrorCB {Function} A function to be called if the request fails

#### Example:
  
  <script>
    initEDMdesignerPlugin("TestAdmin", function(edmDesignerApi) {
      edmDesignerApi.createGroup({name: "exampleGroup", featureSwitch: {}}, function(result) {
      
        edmDesignerApi.getGroup(result._id, function(resultGroup) { 
          //the resultGroup is an object with
          //name {String} (the new group's name)
          //_id  {String} (the new group's id) and
          //featureSwitch {Object} (the group's features) properties
          //customData {Object} The custom infromations you saved for this group
          console.log(resultGroup);
        }, onErrorCB);
      
      }, onErrorCB);
    });
    
    function onErrorCB(error) {
      console.log(error);
    }
  </script>

### Get one group
Gets a specified group

#####Type
  + GET

#####Route
  + //api.edmdesigner.com/json/groups/read/:id

#### Parameters (in the route):
   * id {String} The id of the group. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list your groups with the [/json/groups/list](#list-groups) route.

####Response:
A group object:
  - _id {String} MongoDB id of the group
  - featureSwitch {Object} The features that are available for users belong to this group.There is an ever-expanding [list](#feature-switch) of possible features which you can choose from.
  - name {String} The name of the group
  - customData {Object} The custom informations you saved for this group

Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.
















## Group - update
### edmDesignerApi.updateGroup(groupId, data, callback, onErrorCB)
Updates a specified group's name or the features it provides or both of these two at the same time. You can even add custom informations to the group.
#### Parameters:
  * groupId {String} The id of the group. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list your groups with the [edmDesignerAPI.listGroups](#edmdesignerapilistgroupscallback-onerrorcb) function.
  * data {Object}
     * data.name {String} The name you want to give to the group
     * data.featureSwitch {Object} The features that are available for users belong to this group. There is an ever-expanding [list](#feature-switch) of possible features which you can choose from.
     * data.customData {Object} You can upload custom informations to this group. You can save any kind of information. It is up to you, how you want to use it!
     * data.override {Boolean} If it is true then the customData will be overrided with the newly given data, if it is false then the newly given customData will be merged with the previous ones. By default it is false.
  * callback {Function} A function to be called if the request succeeds
  * onErrorCB {Function} A function to be called if the request fails

#### Example:
  
  <script>
    initEDMdesignerPlugin("TestAdmin", function(edmDesignerApi) {
      edmDesignerApi.createGroup({name: "exampleGroup", featureSwitch: {feature1: true}}, function(result) {
      
        edmDesignerApi.updateGroup(result._id, {name: "newName"}, function(resultGroup) { 
          //the resultGroup is an object with
          //success {Boolean} or err {String} property
          console.log(resultGroup);
        }, onErrorCB);
      
      }, onErrorCB);
      
      edmDesignerApi.createGroup({name: "exampleGroup2", featureSwitch: {feature1: true}}, function(result) {
      
        edmDesignerApi.updateGroup(result._id, {featureSwitch: {feature1: true, newFeature: true}}, function(resultGroup) { 
          //the resultGroup is an object with
          //success {Boolean} or err {String} property
          console.log(resultGroup);
        }, onErrorCB);
        
      }, onErrorCB);
      
      edmDesignerApi.createGroup({name: "exampleGroup3", featureSwitch: {feature1: true}}, function(result) {
      
        edmDesignerApi.updateGroup(result._id, {name: "newExampleName", featureSwitch: {feature1: true, newFeature: true, customData: {foo: "BAR"}}}, function(resultGroup) { 
          //the resultGroup is an object with
          //success {Boolean} or err {String} property
          console.log(resultGroup);
        }, onErrorCB);
      
      }, onErrorCB);
    
      function onErrorCB(error) {
        console.log(error);
      }
    });
  </script>

### Update one group
Updates a specified group's name or the features it provides or both of these two at the same time. You can even add custom informations to the group.

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/groups/update

#### Parameters (you should post):
   * _id {String} /REQUIRED/ The id of the group. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list your groups with the [/json/groups/list](#list-groups) route.
   * name {String} The name you want to give to the group
   * featureSwitch {Object} The features that are available for users belong to this group. There is an ever-expanding [list](#feature-switch) of possible features which you can choose from.
   * customData {Object} You can upload custom informations to this group. You can save any kind of information. It is up to you, how you want to use it!
   * override {Boolean} If it is true then the customData will be overrided with the newly given data, if it is false then the newly given customData will be merged with the previous ones. By default it is false.

####Response:
An object:
  - success {Boolean} It should be true if the update was successful

Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.





---






## User - list
### edmDesignerApi.listUsers(callback, onErrorCB)
Lists the users you have
#### Parameters:
  * callback {Function} A function to be called if the request succeeds
  * onErrorCB {Function} A function to be called if the request fails

#### Example:
  
  <script>
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
  </script>

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
### edmDesignerApi.createUser(data, callback, onErrorCB)
Creates a new user
#### Parameters:
  * data {Object}
    * data.id {String} /REQUIRED/ The id you want to use for this new user
    * data.group {String} The id of the group you want this user to belong. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list your groups with the [edmDesignerAPI.listGroups](#edmdesignerapilistgroupscallback-onerrorcb) function.
    * data.customData {Object} You can aupload custom informations to this user. You can save any kind of information. It is up to you, how you want to use it!
  * callback {Function} A function to be called if the request succeeds
  * onErrorCB {Function} A function to be called if the request fails

#### Example:
  
  <script>
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
  </script>


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
### edmDesignerApi.createMultipleUser(data, callback, onErrorCB)
Creates multiple user
#### Parameters:
  * data {Array} It contains user objects. User object should have the following properties:
    * id {String} /REQUIRED/ The id you want to use for this new user
    * group {String} The id of the group you want this user to belong. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list your groups with the [edmDesignerAPI.listGroups](#edmdesignerapilistgroupscallback-onerrorcb) function.
    * customData {Object} You can aupload custom informations to this user. You can save any kind of information. It is up to you, how you want to use it!
  * callback {Function} A function to be called if the request succeeds
  * onErrorCB {Function} A function to be called if the request fails

#### Example:
  
  <script>
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
  </script>


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
### edmDesignerApi.getUser(userId, callback, onErrorCB)
Gets a specified user
#### Parameters:
  * userId {String} The id of the user.
  * callback {Function} A function to be called if the request succeeds
  * onErrorCB {Function} A function to be called if the request fails

#### Example:

  <script>
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
  </script>

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

___

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






## User - update
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

#### Example:
  
  <script>
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
  </script>


## User - remove
### edmDesignerApi.deleteUser(userId, callback, onErrorCB)
Deletes a specified user
#### Parameters:
  * userId {String} The id of the user.
  * callback {Function} A function to be called if the request succeeds
  * onErrorCB {Function} A function to be called if the request fails

#### Example:
  
  <script>
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
  </script>

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
###edmDesignerApi.getProject(projectId, callback, onErrorCB)
Get a specified project as json. The result will contain almost every information about the selected project.
#### Parameters:
  * projectId {String}  The id of the project. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list the projects of the user with the [edmDesignerAPI.listProjects](#edmdesignerapilistprojectscallback-onerrorcb) function.
  * callback {Function} A function to be called if the request succeeds
  * onErrorCB {Function} A function to be called if the request fails

#### Example:

  <script>
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
  </script>
___


# Server side routes
Almost every client side functions have a corresponding route on server side.


## Admin routes

##Authentication
To authenticate these routes, you need to generate a token to a "fake" user called admin.You should send this token and the admin string on every request's query. (like this: ?user=admint&token=token ).Please note that every route need to be authenticated expect the one which generate the token. (//api.edmdesigner.com/api/token)

Example: //api.edmdesigner.com/json/groups/list?user=admin&token=adminToken

### Create handshaking
Create the handshaking between PHP and API needs to be done before any further call
    
    $url = "http://api.edmdesigner.com/api/token";
    $data = array(
      "id"  => $publicId,
      "uid" => $user,
      "ip"  => $ip,
      "ts"  => $timestamp,
      "hash"  => $hash
    );
    
    $options = array(
        'http' => array(
            'header'  => "Content-type: application/x-www-form-urlencoded\r\n",
            'method'  => 'POST',
            'content' => http_build_query($data),
        )
    );

    $context  = stream_context_create($options);
    $result = file_get_contents($url, false, $context);
    
    $token = json_decode($result, TRUE);

___


## Group handler routes

### Upload Complex Elems 
Upload a list of Complex elems to the specified group. Every user who belongs to this group will be able to use these Complex elems. Please note that every upload will overwrite the previous uploads!  
If you want to know what is a Complex elem, please read the [complexElements](#complexelements) part of the documentation!

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/complexElem/addComplexElemToGroup

#### Parameters (you should post):
   * groupId {String} /REQUIRED/ The id of the group. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list your groups with the [/json/groups/list](#list-groups) route.
   * items {Array} /REQUIRED/ The list of Complex elems which the users of the specified group will be able to use. __Please note that if you upload a new list, the old list will be overwrited!__ If you want to know how a Complex elem object should look like, please read [this](#structure) part of the documentation.

####Response:
An object with 3 child objects:
  - result {Array} list of the complex elems which were successfully added to the group
  - fails {Array} if some of the items failed it contains the reason, if none of them failed it is null 
  - err {String} If there is any reason why the whole process failed otherwise null

___

## User handler routes

### Upload Complex elems to specified users 
Upload a list of Complex elems to a specified user or users. Please note that every upload will overwrite the previous uploads!  
If you want to know what a Complex elem is good for, please read the [complexElements](#complexelements) part of the documentation!

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/complexElem/addComplexElemToUsers

#### Parameters (you should post):
   * userIds {Array} /REQUIRED/ List of the ids of the users you want to upload the Complex elems.
   * items {Array} /REQUIRED/ The list of Complex elems which the selected users will be able to use. __Please note that if you upload a new list, the old list will be overwrited!__ If you want to know how a Complex elem object should look like, please read [this](#structure) part of the documentation.

####Response:
An object:
  - error {String} If the whole process is failed, otherwise null.
  - fails {Array} If a part of the items failed to insert, if all items inserted, it is null.

___


### Upload Complex elems to all users 
Upload a list of Complex elems to all users. Please note that every upload will overwrite the previous uploads!  
If you want to know what a Complex elem is good for, please read the [complexElements](#complexelements) part of the documentation!

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/complexElem/addComplexElems

#### Parameters (you should post):
   * userIds {Array} /REQUIRED/ List of the ids of the users you want to upload the Complex elems.
   * items {Array} /REQUIRED/ The list of Complex elems which the selected users will be able to use. __Please note that if you upload a new list, the old list will be overwrited!__ If you want to know how a Complex elem object should look like, please read [this](#structure) part of the documentation.

####Response:
An object:
  - error {String} If the whole process is failed, otherwise null.
  - fails {Array} If a part of the items failed to insert, if all items inserted, it is null.
  - result {Array} List of all inserted Complex elems

___








#Gallery handling
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

___

#Project handler admin routes

###Create to
Creates a project/template to a specified user

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/project/createTo

####Parameters (you should post):
  * userId {String} /REQUIRED/ The target user's id
  * title {String} The title of the new template, if it is not given, then the project's name will be 'Untitled'
  * description {String} Description of the template, by defaullt it is an empty string
  * document {Object} An object, which represents a template. By setting this param, you can create new projects based on your prepared custom templates
  * customData {Object} You can upload custom informations to this project. You can save any kind of information. It is up to you, how you want to use it!

####Response:
Project object:
  - _id {String} MongoDB _id of the newly created project

or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

###Create from to
Creates a project/template to a specified user using an another template

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/project/createFromTo

####Parameters (you should post):
  * userId {String} /REQUIRED/ The target user's id
  * _id {String} /REQUIRED/ The MongoDB _id of the template we want to copy to the target user
  * title {String} The title of the new template, if it is not given, then the project's name will be 'Untitled'
  * description {String} Description of the template, by defaullt it is an empty string
  * document {Object} An object, which represents a template. By setting this param, you can create new projects based on your prepared custom templates
  * customData {Object} You can upload custom informations to this project. You can save any kind of information. It is up to you, how you want to use it!

####Response:
Project object:
  - _id {String} MongoDB _id of the newly created project

or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

###Create from to with images
It is quiet similar to the [create from to route](#create-from-to), basic with this route you can do the same thing except, here not only the project will be copied, but the images (which are used in the template) too. If you want to use this route, first you need to implement a [copy route](#copy-route) on your server side and [config](#configure-api-servers-gallery) it's address to the gallery.

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/project/createFromToWithImages

####Parameters (you should post):
  * userId {String} /REQUIRED/ The target user's id
  * _id {String} /REQUIRED/ The MongoDB _id of the template we want to copy to the target user
  * title {String} The title of the new template, if it is not given, then the project's name will be 'Untitled'
  * description {String} Description of the template, by defaullt it is an empty string
  * customData {Object} You can upload custom informations to this project. You can save any kind of information. It is up to you, how you want to use it!

####Response:
Project object:
  - _id {String} MongoDB _id of the newly created project

or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

###Get Project
Get a specified project as json. The result will contain almost every information about the selected project.

#####Type
  + GET

#####Route
  + //api.edmdesigner.com/json/project/getProject/:id

####Parameters (in the route):
  * :id {String} The id of the target project

####Response:
Project object:
  - _id {String} MongoDB _id of the newly created project
  - title {String} The title of the project.
  - description {String} Description about the project
  - createdOn {String} Creation time
  - lastModified {String} Time of the last modification
  - document {Object} The json format of the template
    - usedColors {Array} List of the colors which are used in the template
    - generalSettings {Object} The default settings of the template
    - root {Object} The actual structure of the template
    - header {Object} The header of the template
    - footer {Object} The footer of the template 
  - customData {Object} Your custom data

or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

###Set Project
Sets the root of a selected user's selected project's document. 

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/project/setProject

####Parameters (you should post):
  * userId {String} /REQUIRED/ the id of the user whose project you want to update
  * projectId {Sring} /REQUIRED/ the id of the project you want to update
  * documentRoot {Object} /REQUIRED/ The EDM template json you want to set to the selected project. It should start with [Root](#root) elem

####Response:
Success object:
  - success {boolean}

or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

###Download as Zip
It is possible to export your template as a zip. 

#####Type
  + GET

#####Route
  + //api.edmdesigner.com//json_v1.0.0/apiKey/:apiKey/user/:user/project/:project/downloadAsZip

####Parameters (in the route):
  * apiKey {String} the key of the instance the selected user belongs to.
  * user {Sring}  the id of the user
  * project {String} the id of the project you want to download

####Configuration
You can set which files you want to have in your exported zip and how you want to export them. To set some configuration you need to put a config json to the data property in the query part of your request. Your config json should have the following properties:
  - needHtml {Boolean} if it is false then there will be no template.html in your exported zip
  - needJson {Boolean} if it is false then there will be no project.json in your exported zip
  - withoutSanitizing {Boolean} if it is true then the template.html won't be sanitized.
 
Example:
  - the config json: {"config": {"needHtml": true, "needJson": false, "withoutSanitizing": false} }
  - example request: //api.edmdesigner.com/json_v1.0.0/apiKey/yourapikey/user/youruserid/project/projectId/downloadAsZip?user=admin&token=adminToken&data={"config": {"needHtml": true, "needJson": false, "withoutSanitizing": false} }

####Response:
A zip which can contain the followings (it depends on the configuration, by default it will contain everything):
  - template.html {file} The html code of the template. It can be a sanitized version or not, it depends of your configuration. By default it will be sanitized.
  - project.json {file} The json version of the project.
  - images folder with the images of the template.
Please note that every images source (href, src, etc.) in the json and html file are replaced with the paths of the images in the images folder.



##complexElements
The complex elem is a possible tool to save a BOX or MULITCOLL element with all of it contents to be able to reuse it. This feature is created for make the template editing more fast so more effective.
A complex elem can binded to a user, to a group of users or to all of the users.
When a user uses the editor these binding displays all together and can be used all of them.

The administrator of the application can upload complex elems to all of the above possibilities.
The user of the editor can save complex elems only for him/herself, those will be added to the admin defined items.
The editor users can also delete those complex elems what are binded to him/herself as a user.

These are the 3 ways to upload complex elems as an admin:
  - to all your user ([general complexElems upload](#upload-complex-elems-to-all-users)) 
  - to a specified group ([upload complexElems to group](#upload-complex-elems)) 
  - to specified user or users ([upload complexElems to user](#upload-complex-elems-to-specified-users))

___

### Structure
The representing object for a Complex elem should have the following properties:  
  - doc {Object} /REQUIRED/ it should be a json object (which represent our templates)
    - type /REQUIRED/ It must be ["BOX"](#box), ["MULTICOLUMN"](#multicolumn) or ["FULLWIDTH_CONTAINER"](#full-width-container)
    - generalSettings
  - id {String} /optional/ this id is what will identify the item for you if you would like to manage it via admin
  - title {Object} /optional/ it should contains language code - title string pairs. For example: 'en': 'Green-white complexElem'. If you miss to give it, we it will receive a default name. The title will appear on the list of the complex element. If you don't want to use any other localization then please use the 'en' language code, the default will always be the 'en' regardless of the actual language!  


___


### Localization
It is possible to use different localization with the same complex elem but __not required__.
You can localize the title of the complex elem. The title of a complex elem will appear on the list, where the user will be able to choose from.  
If you want to support more than one language you just have to put the language code - text pair to the title object (see [structure](#structure)).
An example title object: 

  title: { 'en': 'example title',
     'hu': 'példa cím'
     /...
    }

___

### Custom Data
If you want to save any kind of plus information for some of your data you can do it with using the customData field
You can save custom informations to:  
  - yourself /apiClientInstance/ ([addCustomData](#add-custom-data-to-yourself))
  - users (update user [server side](#update-user) or [client side](#edmdesignerapiupdateuseruserid-data-callback-onerrorcb))
  - groups (update group [server side](#update-one-group) or [client side](#edmdesignerapiupdategroupgroupid-data-callback-onerrorcb))
  - project (update project [server side](#update-information) or [client side](#edmdesignerapiupdateprojectinfoprojectid-data-callback-onerrorcb))  

___

### Add Custom Data to yourself
You can save any kind of custom information to your apiClientInstance database entry

####Type
  + POST

####Route
  + //api.edmdesigner.com/json/general/saveCustomData

#### Parameters (you should post):
  * customData {Object} The custom informations you want to save
  * override {Boolean} If it is true then the customData will be overrided with the newly given data, if it is false then the newly given customData will be merged with the previous ones. By default it is false.

####Response
Http status code 200
  
Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

### Get my custom data
You can get the custom informations you previously saved for yourself

####Type
  + GET

####Route
  + //api.edmdesigner.com/json/general/getMyCustomData

####Response
An object containing your custom data
  - customData {Object} The custom informations you saved for yourself
  
Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

### Custom Strings
The custom strings are an option to provide predefined texts to your users what they can insert while editing text contents.
You can add custom strings to 
- all of your users 
- to specified users
- or to users of a specified group.

The custom strings can be added by bunch of custom string items. Each bunch contains a title what will be displayed as the label of the drop-down list in the text editor, and an id, what provides the identification. By this id it will be possible to remove the bunch.
The id can not contain any special characters, the title is a string or a localization object. In case of localization object it what must contain 'en' property.
Within an item the label property has the same rules.

###Example of Custom String object:

  var customStringsObject = {
    id: 'validId',
    title: {
      'en': 'EDM beer'
    },
    items: [
                              {
                                 replacer: '##Svijany##',
                                 label: 'good beer 1'            
                              },
                              {
                                 replacer: '##Poutník##',
                                 label: {
                                    en: 'good beer 2'
                                 }
                             }
    ]
  };
___

### Add Custom Strings to all of the users of the apiClientInstance

####Type
  + POST

####Route
  + //api.edmdesigner.com/json/customStrings/add

#### Parameters (you should post):
  * id {String} The identification of the custom strings bunch
  * title {String or  localization Object with 'en' property} the displayed name of the bunch
  * items {Array} Array of customStrings Object [see here](#example-of-custom-string-object)

####Response
Http status code 200
- id {String} the saved custom strings id
- customStrings {Object} the saved custom stings object
- overwritten {Boolean} describes if the save updated an existing bunch or created a new one 
  
Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

### Add Custom Strings to specified users

####Type
  + POST

####Route
  + //api.edmdesigner.com/json/customStrings/addToUsers

#### Parameters (you should post):
  * users {Array} Array of the userIds
  * id {String} The identification of the custom strings bunch
  * title {String or  localization Object with 'en' property} the displayed name of the bunch
  * items {Array} Array of customStrings Object, [see here](#example-of-custom-string-object)

####Response
Http status code 200
- updatedUsers {Array} list of objects with the property of:
       - id {String} the user id
       - updated {Boolean} describes if the save updated an existing bunch or created a new one 
- id {String} the saved custom strings id
- customStrings {Object} the saved custom stings object
  
Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.


### Add Custom Strings to specified groups of the apiClientInstance

####Type
  + POST

####Route
  + //api.edmdesigner.com/json/customStrings/addToGroup

#### Parameters (you should post):
  * group {String} MongoDb ObjectId (24 chars length id of the group)
  * id {String} The identification of the custom strings bunch
  * title {String or  localization Object with 'en' property} the displayed name of the bunch
  * items {Array} Array of customStrings Object, [see here](#example-of-custom-string-object)

####Response
Http status code 200
- id {String} the saved custom strings id
- customStrings {Object} the saved custom stings object
- overwritten {Boolean} describes if the save updated an existing bunch or created a new one 
  
Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.
___

### Get Custom Strings of all of the users of the apiClientInstance

####Type
  + GET

####Route
  + //api.edmdesigner.com/json/customStrings/getCustomStringsOfApiInstance

#### Parameters (you should post):
  no parameters needed

####Response
Http status code 200
- err {null} null  value
- the custom string objects as the property names are the id of the custom strings bunch

example: 
{
err: null,
customStringID1: {
       title: {
                    en: 'csTitle'
         },
        items: [
            ...items of custom string Objects....
        ]
   }
}
  
Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.
___

### Get Custom Strings of a specified user

####Type
  + GET

####Route
  + //api.edmdesigner.com/json/customStrings/getCustomStringsOfUser

#### Parameters (you should post):
 * userId {String} the id of the user

####Response
Http status code 200
- err {null} null  value
- the custom string objects as the property names are the id of the custom strings bunch

example: 
{
err: null,
customStringID1: {
       title: {
                    en: 'csTitle'
         },
        items: [
            ...items of custom string Objects....
        ]
   }
}
  
Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.
___

### Get Custom Strings of a specified group

####Type
  + GET

####Route
  + //api.edmdesigner.com/json/customStrings/getCustomStringsOfGroup

#### Parameters (you should post):
 * groupId {String} mongoDb ObjectID of the group

####Response
Http status code 200
- err {null} null  value
- the custom string objects as the property names are the id of the custom strings bunch

example: 
{
err: null,
customStringID1: {
       title: {
                    en: 'csTitle'
         },
        items: [
            ...items of custom string Objects....
        ]
   }
}
  
Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.
___

### Remove a general Custom Strings by it's id

####Type
  + DELETE

####Route
  + //api.edmdesigner.com/json/customStrings/:id

#### Parameters (you should post):
 * :id {String} the id of the custom string bunch

####Response
Http status code 200
- id {String} the id of the deleted custom strings bunch
Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.
___

### Remove a user's Custom Strings by it's id

####Type
  + DELETE

####Route
  + //api.edmdesigner.com/json/customStrings/:id/removeFromUser/:user

#### Parameters (you should post):
 * :id {String} the id of the custom string bunch
 * :user {String} the id of the user what you want to remove from

####Response
Http status code 200
- id {String} the id of the deleted custom strings bunch
- userId {Sting} the id of the user what you removed from
Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.
___

### Remove a group's Custom Strings by it's id

####Type
  + DELETE

####Route
  + //api.edmdesigner.com/json/customStrings/:id/removeFromUser/:group

#### Parameters (you should post):
 * :id {String} the id of the custom string bunch
 * :group {String} the MongoDb ObjectId of the group what you want to remove from

####Response
Http status code 200
- id {String} the id of the deleted custom strings bunch
- group {Sting} the id of the group what you removed from
Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.
___


##Authentication
To authenticate these routes, userId and a token (generated to the userId) are needed. Those to should be sent on the request's query.(somehow like this: ?user=userID&token=token ). Please note that every route need to be authenticated expect the one which generate the token. (//api.edmdesigner.com/api/token)

For example: //api.edmdesginer.com/json/project/list?user=userId&token=123456789



# API key management


## API key - list
Lists your api keys

#####Type
  + GET

#####Route
  + //api.edmdesigner.com/json/apiKey
  
#### Parameters
 - time {String/Number} Must have. The current timestamp.
 - email {String} Must have. The email to registered to your account.
 - hash {String} Must have. A concatated string as the following: md5(email + timestamp + your global magic word) You can check your global magic word at https://dashboard.edmdesigner.com/#profile
 - findObject {Object} Optional. With this object the result can be filtered and modified.
 - findObject.skip {Number} index where from the listing should begin, without limit parameter it will be ingnored
 - findObject.limit {Number} length of the items to be listed, without skip parameter it will be ingnored
 - findObject.select [Array] list of the properties to be listed ["name", "created"]
 - findObject.sort {Object} MongoDb sort object: {property : "asc/desc"}
 
####Response:
An array of your api keys. Every item is an object with this parameters:
  - _id {String} MongoDB id of the api key
  - featureSwitch {Object} The features that are available for users belong to this apikey. There is an ever-expanding [list](#feature-switch) of possible features which you can choose from.
  - name {String} The name of the api key
  - customData {Object} Th custom informations you saved for the api key
  - magic {String} the magic word of the api key
  - galleryUploadRoute {String} the upload route of the api key
  - galleryDeleteRoute {String} the delete route of the api key
  - galleryCopyRoute {String} the copy route of the api key
  - apiKey {String} the id of the api key

Or in case of of limit and skip parameters sent an Object consists of:
  - totalCount {Number} The length of the full list
  - result {Array} The list of the api keys, same as above

Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

## API key - create
Create new api keys

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/apiKey
  
#### Parameters
 - time {String/Number} Must have. The current timestamp.
 - email {String} Must have. The email to registered to your account.
 - hash {String} Must have. A concatated string as the following: md5(email + timestamp + your global magic word) You can check your global magic word at https://dashboard.edmdesigner.com/#profile
 - items {Array} Must have. An array of the new apiKey object what consists of the following:
 - id: {String} Must have. The id of the api key.
 - name: {String} Optional. The name of the api key.

####Response:
- created {Array} the list of the created api keys
- failed {Array} The list of the failed api keys
- alreadyHave {Array} The list of the api keys what could not be created bacause the keys existed before


## API key - update
Updating skins of existing api key

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/apiKey/:apiKeyId/skin (Where apiKeyId is the id of the api key)
  
#### Parameters
 - time {String/Number} Must have. The current timestamp.
 - email {String} Must have. The email to registered to your account.
 - hash {String} Must have. A concatated string as the following: md5(email + timestamp + your global magic word) You can check your global magic word at https://dashboard.edmdesigner.com/#profile


___

#Feature Switch
You can allow or forbid some features of our application to a group of your users.
The list of the features you can set at the moment:  
  - gallery {Object}: contains the features of the gallery
    - noUpload {Boolean} If you set this parameter true then the users belong to this group cannot use the upload functionality. If you configured the gallery noUploadText (see [gallery config](#configure-api-servers-gallery)) text then it will appear on the upload tab, otherwise a default text will be used.
    - noUrl {Bollean} If you set this parameter true, then the users of this group cannot use any image not hosted by you. The image from url tab of the gallery will disappear if the feature does not allowed.
    - limit {Number} The number of images the user can upload. If you do not want to set any limit then leave this parameter undefined. If a user reach the limit, then he won't be able to upload any more image and if you configured the gallery limitReached (see [gallery config](#configure-api-servers-gallery)) text then it will be displayed otherwise a default text will be used.  Please note that the 0 limit and the noUpload are almost equivalent except a different message will appear on the upload tab if you use one or the other.

This list is constantly expanding with time!

___

#Feature configuration
This is a new development which will be used everywhere from the next api version. Right now it is only used in a few features [texteditor buttons](#wysiwyg-texteditor-buttons).
It is quite similar to the [feater switch](#feature-switch), but it is a little bit "smarter". Here you can configure the features on four different level. Each level inherits the upper levels configuration but has greater priority.  Basicly the configurationions are merged into each other, but the lower levels always override the inherited configurations, if they have different configuration, but not everything, only the affected settings. The inheritance chain is the following:

apiClient -> apiClientInstance(apiKey) -> group -> user.

For example, we have a configuration named "feature" on apiClient level with value 5. We have to users with different groups, in one of this two groups we override this feature with value 8. So the user who belongs to this group will get a feature configuration with feature 8 (he inherited 5 from the apiClient, but his group overrided it to 8, so in the en he inherited feature 8) while the other user will get feature 5 (he inherited it from the apiClient's feature configuration).

If you want to disable a feature on one of this four levels, you have to give the feature a false value. (it is do not give any value to a feature then that feauter will be inherited from the upper levels, so it is important to give the feature a false value).

For example, we have a configuration named "feature" on apiClient level with value {"number": 5}. We have to users with different groups, in one of this two groups we override this feature with value false. So the user who belongs to this group won't get the feature configuration (he inherited {"number": 5} from the apiClient, but his group overrided it to false, so in the en he does not inherited this feature. this basicly means that this user cannot use the given feature) while the other user will get feature {"number": 5} (he inherited it from the apiClient's feature configuration, so he can use this feature).

___

#JSON document descriptors
In some cases you - as an integrator - migth want to use the raw document format instead of using the editor and fetch the document from the created project. Some example cases could be when you want to set custom headers and footers to some of your users or you want to create complex or dynamic element.

We can distiguish two main element descriptors, from which a whole document can be composed. These two main types are the containers and the leaf elements.

Every element has its type property. The possible values are the followings: "BOX", "MULTICOLUMN", "ROOT", "TEXT", "TITLE", "IMAGE", "BUTTON and "FULLWIDTH_CONTAINER".

##Containers
Containers are used to group some elements and define the layout of the project. You can put any kind of elements into containers, so these elements can be other containers and leaf elements as well.
###Box
Boxes are great to hold other elements together, give them margins, paddings, borders and/or background.
Example:

  {
    type: "BOX",
    margin: {
      top: 0,
      right: 20,
      bottom: 0,
      left: 20
    },
    padding: {
      top: 10,
      right: 10,
      bottom: 10,
      left: 10
    },
    border: {
      top: {
        style: "solid",
        width: 1,
        color: "#aabbaa"
      },
      right: {
        style: "dotted",
        width: 2,
        color: "#aabbaa"
      },
      bottom: {
        style: "dashed",
        width: 3,
        color: "#aabbaa"
      },
      left: {
        style: "none",
        width: 0,
        color: "#aabbaa"
      }
    },
    background: {
      color: "#00aaff",
      image: {
        src: "http://startups.hu/images/logos/edmdesigner.png",
        repeat: "no-repeat",
        position: "top center"
      }
    },
    children: [
      //puth the child elements here
    ]
  }
  
The properties what you can see in the example above, are optional.

###Multicolumn
Multicolumns are the main elements to create complex layouts. They can have multiple columns (we support max. 5 in our editor) and every columns can have children. Example:

  {
    type: "MULTICOLUMN",
    cols: [
      [
        //children of the first column
      ],
      [
        //children of the second column
      ]
    ]
  }

###Root
Every document starts with this elem, as this is the top parent elem. It contains a full width container as a child what has a box inside as a child. It has no other property out of the children field. ({type: "ROOT", ... }) In the near future - when we will introduce the full width containers - it will change a little bit.  

Example:

  {
    type: "ROOT",
    children: [
      {
        type: "FULLWIDTH_CONTAINER,
        leftChildren: [
          type: "BOX",
          ... any box properties ...
        ],
        order: "LTR",
        twoCell: false
      }
    ]
  }

###Full width container
The main feature of this element type will be that the background color can be full width, not only 600px as the root element.
It can only be a direct child of the root, so can be placed only in the highest level of the document structure. A fullwidth container can have 1 cell, or one left cell and one right cells divided from the middle. The cells can have their own background color. The elem can be set "left to right" or "right to left". In case of "left to right" setting the left cells will be ordered on the top if there is no enough room for both in the same line. In case of "right to left" the right cell will do so. If the element is set to be 1 celled, the order property determines that which cell is displayed. The content of the not visible cell always will be stored so will never be lost only not displayed.    

Example:

  {
    type: "FULLWIDTH_CONTAINER",
    order: "RTL", (or "LTR")
    twoCell: "true", (or "false")
    leftBackgroundColor: "#aaaaaa",
    rightBackgroundColor: "#FF0000",
    leftChildren: [
      ...any element except root and fullwidth container...
    ],
    rightChildren: [
      ...any element except root and fullwidth container...
    ]
  }

##Leaf elements
Leaf elements cannot have any child elements. These elements are typically the content elements.

###Text
The text element must contain 2 fields: 'text' what can contain html code, but you should not put very complex things in i, and the 'type' with constant 'TEXT'. 
In our editor we enable the simple text formatting options (bold, italic, underlined) and lists (ul, ol) and some other very simple things like h1, h2, h3.
The 'linkColor' and 'linkUnderLine' specifies the links styles for those <a> tags what has no inline style. The defaults values can be found in the example below. 
The 'textStyles' specifies the style for those (non H1, H2, H3) text contents what have no inline specified style. The defaults values can be found in the example below. 
The 'h1Styles', 'h2Styles' and 'h3Styles' specifies the style for those (H1, H2, H3) tags what have no inline specified style. The defaults values can be found in the example below. 
The spacing specifies a padding values around the whole text div. The defaults values can be found in the example below. 

 Example:

  {
    text: "yo <i>this</i> is Sparta",
    type: "TEXT",
    linkColor: "#5555ff",
          linkUnderLine: true,
          textStyles: {
              lineHeight: 14,
              color": "#000000",
              size": 14,
              family": "arial"
          },
           h3Styles: {
              lineHeight: 32,
              color": "#000000",
              size": 32,
              family": "arial"
          },
           h2Styles: {
              lineHeight: 26,
              color": "#000000",
              size": 26,
              family": "arial"
          },
           h1Styles: {
              lineHeight: 20,
              color": "#000000",
              size": 20,
              family": "arial"
          },
    "spacing": {
                    "left": "5",
                    "bottom": "5",
                    "right": "5",
                    "top": "5"
                }
  }

###Title
Title elements are just like texts, but we enable the users to set only h1, h2 and h3 as formatting.

Example:

  {
    text: "yo <i>this</i> is Sparta",
    type: "TITLE",
    linkColor: "#5555ff",
          linkUnderLine: true,
           h3Styles: {
              lineHeight: 32,
              color": "#000000",
              size": 32,
              family": "arial"
          },
           h2Styles: {
              lineHeight: 26,
              color": "#000000",
              size": 26,
              family": "arial"
          },
           h1Styles: {
              lineHeight: 20,
              color": "#000000",
              size": 20,
              family": "arial"
          },
    "spacing": {
                    "left": "5",
                    "bottom": "5",
                    "right": "5",
                    "top": "5"
                }
  }

###Image
Images are relatively complex elements. Just like boxes, they can have paddings, margins, borders, background color (no background image), but they can't have children. They have their own, special properties as well:
  - src: the url of the image
  - altText: the alt text
  - link: you can wrapp in a link to your image with this propery
  - originalWidth: the original width of the image
  - originalHeight: the original height of the image
  - width: the scaled width of the image (how width should it be in your newsletter)
  - height: the scaled height of the image (how heigh should it be in your newsletter) - this is a must have property if you want your newsletters to render nice on outlooks

###Button
Button is relatively complex as well. It can have the following box properties: paddings, margins, borders, backgrouns (color and background image as well). In addition it can have radius. Baically button is a fancy link, so it has the following properties as well:
  - text: the text what sould appear (it can be a simple html snippet as well)
  - href: the link.
  - sizeType: "FIXED" - exact amount of pixels, "FIT_TO_TEXT": the width will be based on the width of the text
  - width: num of pixels if the sizeType is "FIXED"

___

#Iframe messaging
The [edmDesignerApi.openProject](#edmdesignerapiopenprojectprojectid-languagecode-settings-callback-onerrorcb) function will return an iframe. It is possible to communicate with this iframe with the [window.postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Window.postMessage) native method.

###Special elements and features
There are some speciel elements and functionalities what can be used with iframe messaging. You can configure most of this features in our [dashboard](dashboard.edmdesigner.com). The basic concept behind this feature is that this configurable elements and buttons will post a message (with [window.postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Window.postMessage) native method which was mentioned above) to the parent window (it should be your site). You should handle the message on your side, it does not matter how, and send back the content with the corresponding action name ([see below](#possbile-messages-to-send)). For example: a popup window where the user can take some action, set what kind of content he wants to use, and you should send back this content wiht the right action name.

___

###Wysiwyg texteditor buttons
You can insert custom strings to the cursor position in the wysiwyg editor. With this feature you can configure texteditor buttons, which will appear in the wysiwyg editor's toolbar. There can be as many different texteditor buttons as you want. Each button can have its own label. If a user clicks one of this buttons, it posts a message to the parent window with [window.postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Window.postMessage) native method. This messages can be configured too in our [dashboard](dashboard.edmdesigner.com).  
The basic usecase of a texteditor button: The user edits one of his text or title, and wants to insert some custom string so he clicks to one of the texteditor buttons. It posts the configured [message](#texteditor-button-message-format) to the parent window (which should be your site) where you should provide an interface where he can choose which custom string he wants to use. The selected string should be posted back with the right [action name](#insert-to-cursor-in-wysiwyg-editor) to our iframe.  
__Please note that if you want the content to be unchanged/untouched in the generated html code then you should generate that code with the "[generate without sanitizing](#generate-without-sanitizing)" route (instead of the normal [generate](#generate) route. That way the generated code won't be safe enough, so after you replaced your placeholders YOU SHOULD SANITIZE the code (with [google caja](https://code.google.com/p/google-caja/wiki/JsHtmlSanitizer))__

###Texteditor button message format
When a user clicks a texteditor button, we post you (parent window) a message. This message is a stringified json with two parameters:
  - action {String} The action name you configured for this type of code elements
  - content {String} The selected text in the editor (if there is any. It will be null if there is no selected text). Plesae note that the selected text will always be overriden with the content you send in the [repsonse](#insert-to-cursor-in-wysiwyg-editor). If you want to keep the selected text then you should send it back in/with your content.

example: __"{\"action\": \"the action you configured for this texteditor button\", \"content\": \"the selected text or null if there is no selected text\"}"__

###Texteditor button's structure
A texteditor button configuration has the following parameters:
  - id : The id/type of the texteditor button. This is what distinguish one button from another.
  - action : The message our iframe will send to you when a user clicks the texteditor button belonging to this configuration. (You can find how you have to respond to this message [here](#insert-to-cursor-in-wysiwyg-editor)) 
  - label : The name of the button. This will appear as the label of the button. It is localizable and works quite similar like the [header/footer localization](#localization). Please note that the "en" value is the default value, so it should always be set.

You can configure your texteditor buttons for:
  * [every instance](dashboard.edmdesigner.com/#textEditorButton/general)
  * [every user belonging to one instance](dashboard.edmdesigner.com/#textEditorButton/instance)
  * [a group of user](dashboard.edmdesigner.com/#textEditorButton/group)
  * [some selected user](dashboard.edmdesigner.com/#textEditorButton/user)

A user inherits a texteditor button settings just like it is explained in the [feature configuration](#feature-configuration) part of the documentation

___

###Possbile messages to send
List of the events you can send to the iframe (as a message).  

###Save Project
It is possible to manualy ask the iframe to save the actual (opened) project. As a response the iframe will post a [save result](#save-result) message.  
The message you need to send: __saveProject__

###Force selected element to be deselected
It is possible to force the editor to deselect the actual selected element. There won't be any kind of response (from the api iframe) to this message.
The message you need to send: __loseSelected__

###Insert to cursor in wysiwyg editor
The response for the incoming messages from [texteditor buttons](#wysiwyg-texteditor-button). You need to post a stringified json as the message string. It should have two properties:
  - action {String} The name of the action: __InsertToCursor__
  - content {String} The content (placeholders) the user wants to insert to the cursor 

The message you need to send: __{"action": "InsertToCursor", "content": "your content goes here"}__
___

###Possible response messages
List of the events the iframe can send to your application (as a message).

###Save result
It is the answer for the [save project](#save project) request. It can have to different statuses:  
In case of success the answer message will be the following: __Save result: success__  
In case of failure the answer message will be the following: __Save result: failed__  

###Project Loading Started
The iframe will send you a message at the begining of the project loading.
The message will be the following: __ProjectLoadingStarted__

###Project Loading Finished
The iframe will send you a message when the project loading is done. It can have different statuses:
In case the project loading was succesful, the message will be the following: __ProjectLoadingSuccess__
In case the project loading failed, the message will be the following: __ProjectLoadingFailed__



If the language you want to use is not available yet then please contact us with the following email address: info@edmdesigner.com



#Available Languages
You can set the localization which the api will use. (if you want to know how, please check the [edmDesignerApi.openProject](#edmdesignerapiopenprojectprojectid-languagecode-settings-callback-onerrorcb) function!)  
Available languages:
 - English (code: 'en')
 - Hungarian (code: 'hu')

Talk about the iframe src as well!!!


#Example implementations
  * [Node.js example](https://github.com/EDMdesigner/EDMdesigner-API-Example-Node.js)
  * [PHP example](https://github.com/EDMdesigner/EDMdesigner-API-Example-PHP)
  * [PHP admin example](https://github.com/EDMdesigner/EDMDesigner-API-Example-PHP-Admin)
  

