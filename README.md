EDMdesigner-API
===============

[EDMdesigner](http://www.edmdesigner.com) is a drag and drop tool for creating responsive HTML e-mail templates very quickly and painlessly, radically increasing your click-through rate. This documentation is to give you a detailed description about the EDMdesigner-API with which you can integrate our editor into any web-based system (e.g. a CRM, CMS, WebShop or anything else you can imagine).

To start developing with our api, please request an API_KEY (and a corresponding magic word) at info@edmdesigner.com.

Basically, the only thing you have to do is to implement one side of the handshaking on your server. When the handshaking is done, you can use the sent object, in which you can find the functions that are communicating with our server.

The handshaking is built into the initialization process, so if you implemented the handshaking on your side, everything will work automatically.

We provide example implementations that include the handshaking as well. You can find the continously broading list at the end of this page at the Example implementations section.

##### Table of content

1. __[Initializin](#initializing)__  
2. __[Handshaking](#handshaking)__  
3. __[Api functions](#api-functions)__  
  1. [User functions](#user-functions)  
  2. [Admin functions](#admin-functions)  
4. __[Server side routes](#server-side-routes)__  
  1. [Admin routes](#admin-routes)  
    * _[Authentication](#authentication)_  
    * _[Group handler routes](#group-handler-routes)_  
    * _[User handler routes](#user-handler-routes)_  
    * _[Gallery handling](#gallery-handling)_  
    * _[Project handler routes](#project-handler-admin-routes)_  
    * _[Headers & Footers](#headers--footers)_
  2. [User routes](#user-routes)  
    *  _[Authentication](#authentication-1)_  
    *  _[Project routes](#project-routes)_  
5. __[Feature switch](#feature-switch)__
6. __[JSON document descriptors](#json-document-descriptors)__
7. __[Available languages](#available-languages)__  
8. __[Examples](#example-implementations)__
9. __[Dependencies](#dependencies)__  


Initializing
------------
	
	<script src="path_to_your_jquery.js"></script>
	<script src="http://api.edmdesigner.com/EDMdesignerAPI.js?route=##handshaking_route##"></script>
	<script>
		initEDMdesignerPlugin("##userId##", function(edmDesignerApi) {
			...
		}, onErrorCB);
		
		function onErrorCB(error) {
			console.log(error);
		}
	</script>
	
First of all, jQuery has to be loaded before you load our API. In the second line you can see a route parameter in the script's src. By that you can tell the script where to look for the handshaking implementation. For example, if you implemented it in your index.php, you have to replace ##handshaking_route## with index.php. The first parameter is a user id from your system. It can be any string. In case you don't want to handle separate user accounts, just pass there your API_KEY. The last parameter is an error callback which will be called in every request if the request fails. This is not required thus you can set an error callback in one of your function calls (see later) and in that case it will be called instead of this one.

In the resulting object (edmDesignerApi) you will find some functions through which you can interact with our system.

### Handshaking
To implement the handshaking on your server, you need an API_KEY, a magic word (wich is delivered with your API_KEY), your user's IPv4 address and a timestamp. The logic that handles handshaking has to receive the userId too, because it has to be sent to our server as well. Our API implementation automatically sends the userId in a POST HTTP request. You can set this user id by the first parameter of the initEDMdesignerPlugin.

After concatenating the API_KEY, the IPv4 address of your user, the timestamp (as a string) and your magic word, you have to create an md5 hash of the resulting string.

	
	hash = md5(API_KEY + ipv4 + timestamp + magic)


After this, you have to send the API_KEY, the userId, the IPv4 address, the timestamp and the generated md5 hash to our server at api.edmdesigner.com/api/token through HTTP.

If everything goes well, you get back a token, with which your user can be identified.

# API functions
## User functions
### edmDesignerApi.listProjects(callback, onErrorCB)
Lists the projects of the actual user.
#### Parameters:
  * callback {Function} A function to be called if the request succeeds
  * onErrorCB {Function} A function to be called if the request fails

#### Example:
	
	<script>
		initEDMdesignerPlugin("TestUser", function(edmDesignerApi) {
			edmDesignerApi.listProjects(function(result) {
				console.log(result);
			});
		}, onErrorCB);
		
		function onErrorCB(error) {
			console.log(error);
		}
	</script>

___	

### edmDesignerApi.createProject(data, callback, onErrorCB)
Creates a new project (a new e-mail template).
#### Parameters:
  * data {Object}
    * data.title {String} The title of the new project.
    * data.description {String} The description of the new project.
    * data.document {Object} An object, which represents a template. By setting this param, you can create new projects based on your prepared custom templates.
  * callback {Function} A function to be called if the request succeeds
    * the result param is an object in which you can find an _id property, which identifies the newly created project
    * result._id {String} The id of the new project. (This is a [MongoDB](http://www.mongodb.org/) id.)
  * onErrorCB {Function} A function to be called if the request fails

#### Example:
	
	<script>
		initEDMdesignerPlugin("TestUser", function(edmDesignerApi) {
			edmDesignerApi.createProject(function({title: "test-title", description: "test-desc", document: {}}, result) {
				console.log(result._id);
			});
		}, onErrorCB);
		
		function onErrorCB(error) {
			console.log(error);
		}
	</script>

___	
	
### edmDesignerApi.duplicateProject(projectId, callback, onErrorCB)
Creates the exact copy of the project with the ID specified in projectId.
#### Parameters:
  * projectId {String} The id of the project. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list the projects of the user with the [edmDesignerAPI.listProjects](#edmdesignerapilistprojectscallback-onerrorcb) function.
  * callback {Function} A function to be called if the request succeeds
  * onErrorCB {Function} A function to be called if the request fails

#### Example:
	
	<script>
		initEDMdesignerPlugin("TestUser", function(edmDesignerApi) {
			edmDesignerApi.createProject({title: "test-title", description: "test-desc"}, function(result) {
				edmDesignerApi.duplicateProject(result._id, function(result) {
					//callback hell...
				});
			});
		}, onErrorCB);
		
		function onErrorCB(error) {
			console.log(error);
		}
	</script>

___	

### edmDesignerApi.removeProject(projectId, callback, onErrorCB)
Removes a project.
#### Parameters:
  * projectId {String} The id of the project. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list the projects of the user with the [edmDesignerAPI.listProjects](#edmdesignerapilistprojectscallback-onerrorcb) function.
  * callback {Function} A function to be called if the request succeeds
  * onErrorCB {Function} A function to be called if the request fails

#### Example:
	
	<script>
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
	</script>

___	

### edmDesignerApi.openProject(projectId, [languageCode], callback, onErrorCB)
Opens a project.
#### Parameters:
  * projectId {String} The id of the project. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list the projects of the user with the [edmDesignerAPI.listProjects](#edmdesignerapilistprojectscallback-onerrorcb) function.
  * languageCode {String} /Optional/ A two character [ISO 639-1 code](http://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) of a selected language. With this parameter you can choose the language which the opened project will use. You can find the [list of the available languages](#languages) at the end of this page. Default language is english.
  * callback {Function} A function to be called if the request succeeds
  * onErrorCB {Function} A function to be called if the request fails

#### Example:
	
	<script>
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
	</script>

___	

### edmDesignerApi.generateProject(projectId, callback, onErrorCB)
Generates the bulletproof responsive HTML e-mail based on the projectId.
#### Parameters:
  * projectId {String} The id of the project. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list the projects of the user with the [edmDesignerAPI.listProjects](#edmdesignerapilistprojectscallback-onerrorcb) function.
  * callback {Function} A function to be called if the request succeeds
  * onErrorCB {Function} A function to be called if the request fails

#### Example:
	
	<script>
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
	</script>

___	
	
### edmDesignerApi.getDefaultTemplates(callback, onErrorCB)
You can get the default templates povided by EDMdesigner by calling this funciton.
#### Parameters:
  * callback {Function} A function to be called if the request succeeds
  * onErrorCB {Function} A function to be called if the request fails
 
#### Example:

	<script>
		initEDMdesignerPlugin("TestUser", function(edmDesignerApi) {
			edmDesignerApi.getDefaultTemplates(function(result) {
			//the result is an array, containing the default projects, provided by EDMdesigner
			}, onErrorCB);
		});
		
		function onErrorCB(error) {
			console.log(error);
		}
	</script>

___

##Admin functions
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

___

### edmDesignerApi.createGroup(data, callback, onErrorCB)
Creates a new group
#### Parameters:
  * data {Object}
    * data.name {String} /REQUIRED/ The name you want to give to the new group
    * data.featureSwitch {Object} The features that are available for users belong to this group. There is an ever-expanding [list](#feature-switch) of possible features which you can choose from.
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

___

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
					console.log(resultGroup);
				}, onErrorCB);
			
			}, onErrorCB);
		});
		
		function onErrorCB(error) {
			console.log(error);
		}
	</script>

___

### edmDesignerApi.updateGroup(groupId, data, callback, onErrorCB)
Updates a specified group's name or the features it provides or both of these two at the same time.
#### Parameters:
  * groupId {String} The id of the group. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list your groups with the [edmDesignerAPI.listGroups](#edmdesignerapilistgroupscallback-onerrorcb) function.
  * data {Object}
     data.name {String} The name you want to give to the group
     * data.featureSwitch {Object} The features that are available for users belong to this group. There is an ever-expanding [list](#feature-switch) of possible features which you can choose from.
  * callback {Function} A function to be called if the request succeeds
  * onErrorCB {Function} A function to be called if the request fails
 
#### Example:
	
	<script>
		initEDMdesignerPlugin("TestAdmin", function(edmDesignerApi) {
			edmDesignerApi.createGroup({name: "exampleGroup", featureSwitch: {feature1: true}}, function(result) {
			
				edmDesignerApi.updateGroup(result._id, {name: "newName"}, function(resultGroup) {	
					//the resultGroup is an object with
					//name {String} (the new group's name)
					//_id  {String} (the new group's id) and
					//featureSwitch {Object} (the group's features) properties
					console.log(resultGroup);
				}, onErrorCB);
			
			}, onErrorCB);
			
			edmDesignerApi.createGroup({name: "exampleGroup2", featureSwitch: {feature1: true}}, function(result) {
			
				edmDesignerApi.updateGroup(result._id, {featureSwitch: {feature1: true, newFeature: true}}, function(resultGroup) {	
					//the resultGroup is an object with
					//name {String} (the new group's name)
					//_id  {String} (the new group's id) and
					//featureSwitch {Object} (the group's features) properties
					console.log(resultGroup);
				}, onErrorCB);
				
			}, onErrorCB);
			
			edmDesignerApi.createGroup({name: "exampleGroup3", featureSwitch: {feature1: true}}, function(result) {
			
				edmDesignerApi.updateGroup(result._id, {name: "newExampleName", featureSwitch: {feature1: true, newFeature: true}}, function(resultGroup) {	
					//the resultGroup is an object with
					//name {String} (the new group's name)
					//_id  {String} (the new group's id) and
					//featureSwitch {Object} (the group's features) properties
					console.log(resultGroup);
				}, onErrorCB);
			
			}, onErrorCB);
		
			function onErrorCB(error) {
				console.log(error);
			}
		});
	</script>

___
	
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
				console.log(result);
			}, onErrorCB);
		});
		
		function onErrorCB(error) {
			console.log(error);
		}
	</script>

___

### edmDesignerApi.createUser(data, callback, onErrorCB)
Creates a new user
#### Parameters:
  * data {Object}
    * data.id {String} /REQUIRED/ The id you want to use for this new user
    * data.group {String} The id of the group you want this user to belong. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list your groups with the [edmDesignerAPI.listGroups](#edmdesignerapilistgroupscallback-onerrorcb) function.
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

___

### edmDesignerApi.createMultipleUser(data, callback, onErrorCB)
Creates multiple user
#### Parameters:
  * data {Array} It contains user objects. User object should have the following properties:
    * id {String} /REQUIRED/ The id you want to use for this new user
    * group {String} The id of the group you want this user to belong. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list your groups with the [edmDesignerAPI.listGroups](#edmdesignerapilistgroupscallback-onerrorcb) function.
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

___

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
					createTime (Time of creation) properties*/
					console.log(resultUser);
				}, onErrorCB);
			
			}, onErrorCB);
		});
		
		function onErrorCB(error) {
			console.log(error);
		}
	</script>

___

### edmDesignerApi.updateUser(userId, data, callback, onErrorCB)
Updates a specified user. Only the group (which the user belongs) can be changed.
#### Parameters:
  * userId {String} The id of the user. 
  * data {Object}
    * data.group {String} The id of the group you want this user to belong. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list your groups with the [edmDesignerAPI.listGroups](#edmdesignerapilistgroupscallback-onerrorcb) function.
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

___

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
			"id"	=> $publicId,
			"uid"	=> $user,
			"ip"	=> $ip,
			"ts"	=> $timestamp,
			"hash"	=> $hash
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
### List groups
Lists the groups you have

#####Type
  + GET

#####Route
  + //api.edmdesigner.com/json/groups/list

####Answer:
An array of your groups. Every group is an object with this parameters:
  - _id {String} MongoDB id of the group
  - featureSwitch {Object} The features that are available for users belong to this group. There is an ever-expanding [list](#feature-switch) of possible features which you can choose from.
  - name {String} The name of the group

Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

### Create group
Creates a new group

#####Type
  + POST

#####Route 
  + //api.edmdesigner.com/json/groups/create

#### Parameters (you should post):
  * name {String} /REQUIRED/ The name you want to give to the new group
  * featureSwitch {Object} The features that are available for users belong to this group. There is an ever-expanding [list](#feature-switch) of possible features which you can choose from.

####Answer
An object containing the MongoDB _id of the newly created group:
  - _id {String}

Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

### Get one group
Gets a specified group

#####Type
  + GET

#####Route
  + //api.edmdesigner.com/json/groups/read/:id

#### Parameters (in the route):
   * id {String} The id of the group. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list your groups with the [/json/groups/list](#list-groups) route.

####Answer:
A group object:
  - _id {String} MongoDB id of the group
  - featureSwitch {Object} The features that are available for users belong to this group.There is an ever-expanding [list](#feature-switch) of possible features which you can choose from.
  - name {String} The name of the group

Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

### Update one group
Updates a specified group's name or the features it provides or both of these two at the same time.

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/groups/update

#### Parameters (you should post):
   * _id {String} /REQUIRED/ The id of the group. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list your groups with the [/json/groups/list](#list-groups) route.
   * name {String} The name you want to give to the group
   * featureSwitch {Object} The features that are available for users belong to this group. There is an ever-expanding [list](#feature-switch) of possible features which you can choose from.

####Answer:
A newly updated group object:
  - _id {String} MongoDB id of the group
  - featureSwitch {Object} The features that are available for users belong to this group. There is an ever-expanding [list](#feature-switch) of possible features which you can choose from.
  - name {String} The name of the group

Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

### Upload Headers 
Upload a list of headers to the specified group. Every user who belongs to this group will be able to use these headers. Please note that every upload will overwrite the previous uploads!  
If you want to know what a header is good for, please read the [headers & footers](#headers--footers) part of the documentation!

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/groups/uploadHeaders

#### Parameters (you should post):
   * groupId {String} /REQUIRED/ The id of the group. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list your groups with the [/json/groups/list](#list-groups) route.
   * headers {Array} /REQUIRED/ The list of headers which the users of the specified group will be able to use. __Please note that if you upload a new list, the old list will be overwrited!__ If you want to know how a header object should look like, please read [this](#structure) part of the documentation.

####Answer:
An object with two arrays:
  - success {Array} The list of the headers which were successfully added to the group
  - failed {Array} The list of the object: 
    - header {Object} The header which is not valid therefore it was not added to the group
    - error {String} The reason why the header is not valid

Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

### Upload Footers 
Upload a list of footers to the specified group. Every user who belongs to this group will be able to use these footers. Please note that every upload will overwrite the previous uploads!  
If you want to know what a footer is good for, please read the [headers & footers](#headers--footers) part of the documentation!

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/groups/uploadFooters

#### Parameters (you should post):
   * groupId {String} /REQUIRED/ The id of the group. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list your groups with the [/json/groups/list](#list-groups) route.
   * footers {Array} /REQUIRED/ The list of footers which the users of the specified group will be able to use. __Please note that if you upload a new list, the old list will be overwrited!__ If you want to know how a footer object should look like, please read [this](#structure) part of the documentation.

####Answer:
An object with two arrays:
  - success {Array} list of the footers which were successfully added to the group
  - failed {Array} list of the object: 
    - footer {Object} The footer which is not valid therefore it was not added to the group
    - error {String} The reason why the footer is not valid

Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

### Upload Complex Elems 
Upload a list of Complex elems to the specified group. Every user who belongs to this group will be able to use these Complex elems. Please note that every upload will overwrite the previous uploads!  
If you want to know what is a Complex elem, please read the [complexElems](#complexElems) part of the documentation!

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/complexElem/addComplexElemToGroup

#### Parameters (you should post):
   * groupId {String} /REQUIRED/ The id of the group. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list your groups with the [/json/groups/list](#list-groups) route.
   * items {Array} /REQUIRED/ The list of Complex elems which the users of the specified group will be able to use. __Please note that if you upload a new list, the old list will be overwrited!__ If you want to know how a Complex elem object should look like, please read [this](#structure) part of the documentation.

####Answer:
An object with 3 child objects:
  - result {Array} list of the complex elems which were successfully added to the group
  - fails {Array} if some of the items failed it contains the reason, if none of them failed it is null 
  - err {String} If there is any reason why the whole process failed otherwise null

___

## User handler routes

### List
Lists the users you have

#####Type
  + GET

#####Route
  + //api.edmdesigner.com/json/user/list

####Answer:
An array of your users. Every user is an object with this parameters:
  - id {String} The id of the user
  - group {String} The MongoDB _id of the group the user belongs to
  - createTime {String} The time when the user was created

Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

### Create user
Creates a new user

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/user/create

#### Parameters (you should post):
  * id {String} /REQUIRED/ The id you want to use for this new user
  * group {String} The id of the group you want this user to belong. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list your groups with the [/json/groups/list](#list-groups) route.

####Answer:
User object:
  - id {String} The id of the user (your id, not the MongoDB _id)

Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

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

####Answer:
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

___

### Get one user
Gets a specified user

#####Type
  + GET

#####Route
  + //api.edmdesigner.com/json/user/getOne/:id

####Parameters (in the route):
   * :id {String} The id of the user.

####Answer:
User object:
  - id {String} The id of the user
  - group {String} The MongoDB _id of the group the user belongs to
  - createTime {String} The time when the user was created

Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

### Update user
Updates a specified user. Only the group (which the user belongs) can be changed.

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/user/update

#### Parameters (you should post):
   * id {String} The id of the user. 
   * group {String} The id of the group you want this user to belong. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list your groups with the [/json/groups/list](#list-groups) route.

####Answer:
User object:
  - id {String} The id of the updated user

Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

### Delete user
Deletes a specified user

#####Type
  + DELETE

#####Route
  + //api.edmdesigner.com/json/user/delete/:id
  
#### Parameters (in the route):
   * :id {String} The id of the user.
 
####Answer:
User object:
  - id {String} The id of the deleted user

Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

### Upload Headers 
Upload a list of headers to a specified user or users. Please note that every upload will overwrite the previous uploads!  
If you want to know what a header is good for, please read the [headers & footers](#headers--footers) part of the documentation!

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/user/uploadHeaders

#### Parameters (you should post):
   * users {Array} /REQUIRED/ List of the ids of the users you want to upload the headers.
   * headers {Array} /REQUIRED/ The list of headers which the selected users will be able to use. __Please note that if you upload a new list, the old list will be overwrited!__ If you want to know how a header object should look like, please read [this](#structure) part of the documentation.

####Answer:
An object with two arrays:
  - success {Array} list of the users whose successfully got the headers
  - failed {Array} list of the object: 
    - user {String} The id of the user who did not get the headers
    - error {String} The reason why the user did not get the headers
  - badHeaders {Array} list of the object: 
    - header {Object} The header which is not valid therefore it was not added to the users
    - error {String} The reason why the header is not valid

Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

### Upload Footers 
Upload a list of footers to a specified user or users. Please note that every upload will overwrite the previous uploads!  
If you want to know what a footer is good for, please read the [headers & footers](#headers--footers) part of the documentation!

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/user/uploadFooters

#### Parameters (you should post):
   * users {Array} /REQUIRED/ List of the ids of the users you want to upload the footers.
   * footers {Array} /REQUIRED/ The list of footers which the selected users will be able to use. __Please note that if you upload a new list, the old list will be overwrited!__ If you want to know how a footer object should look like, please read [this](#structure) part of the documentation.

####Answer:
An object with two arrays:
  - success {Array} The list of the users whose successfully got the footers
  - failed {Array} The list of the object: 
    - user {String} The id of the user who did not get the footers
    - error {String} The reason why the user did not get the footers
  - badFooters {Array} list of the object: 
    - footer {Object} The footer which is not valid therefore it was not added to the users
    - error {String} The reason why the footer is not valid

Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

### Add custom string to all users
Add customstrings to all of your users.
Note that every calls overwrites the previous, so if you want to remove all the custom strings, just do the call with 	$customStrings['items'] or dont send the items at all
	
		$url = "http://api.edmdesigner.com/json/user/addCustomStrings?token=".$token."&user=".$user;
		//the $token is the string received from token validation and the $user is an existing userId
		
		$customStrings = array();
		$customStrings['items'] = array();
		$customStrings['items'][] = array('label' => 'testLabel1', 'replacer' => 'testReplacer1');
		$customStrings['items'][] = array('label' => 'testLabel2', 'replacer' => 'testReplacer2');
		$customStrings['items'][] = array('label' => 'testLabel3', 'replacer' => 'testReplacer3');
		
		$options = array(
		    'http' => array(
		        'header'  => "Content-type: application/x-www-form-urlencoded\r\n",
		        'method'  => 'POST',
		        'content' => http_build_query($data),
		    )
		);

		$context  = stream_context_create($options);
		$result = file_get_contents($url, false, $context);
		$savedResult = json_decode($result, TRUE);
		
___

### Add custom string to specified user
Add customstrings to just some of your users. 
Note that every calls overwrites the previous, so if you want to remove all the custom strings, just do the call with 	$customStrings['items'] or dont send the items at all
	
		$url = "http://api.edmdesigner.com/json/user/addCustomStringsToUser?token=".$token."&user=".$user;
		//the $token is the string received from token validation and the $user is an existing userId
		
		$customStrings = array();
		$customStrings['userId'] = 'your user id'; // It must be one of your existing user's id, otherwise returns error
		$customStrings['items'] = array();
		$customStrings['items'][] = array('label' => 'testLabel1', 'replacer' => 'testReplacer1');
		$customStrings['items'][] = array('label' => 'testLabel2', 'replacer' => 'testReplacer2');
		$customStrings['items'][] = array('label' => 'testLabel3', 'replacer' => 'testReplacer3');
		
		$options = array(
		    'http' => array(
		        'header'  => "Content-type: application/x-www-form-urlencoded\r\n",
		        'method'  => 'POST',
		        'content' => http_build_query($data),
		    )
		);

		$context  = stream_context_create($options);
		$result = file_get_contents($url, false, $context);
		$savedResult = json_decode($result, TRUE);

___

### Upload Complex elems to specified users 
Upload a list of Complex elems to a specified user or users. Please note that every upload will overwrite the previous uploads!  
If you want to know what a Complex elem is good for, please read the [complexElems](#complexElems) part of the documentation!

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/complexElem/addComplexElemToUsers

#### Parameters (you should post):
   * userIds {Array} /REQUIRED/ List of the ids of the users you want to upload the Complex elems.
   * items {Array} /REQUIRED/ The list of Complex elems which the selected users will be able to use. __Please note that if you upload a new list, the old list will be overwrited!__ If you want to know how a Complex elem object should look like, please read [this](#structure) part of the documentation.

####Answer:
An object:
  - error {String} If the whole process is failed, otherwise null.
  - fails {Array} If a part of the items failed to insert, if all items inserted, it is null.

___


### Upload Complex elems to all users 
Upload a list of Complex elems to all users. Please note that every upload will overwrite the previous uploads!  
If you want to know what a Complex elem is good for, please read the [complexElems](#complexElems) part of the documentation!

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/complexElem/addComplexElems

#### Parameters (you should post):
   * userIds {Array} /REQUIRED/ List of the ids of the users you want to upload the Complex elems.
   * items {Array} /REQUIRED/ The list of Complex elems which the selected users will be able to use. __Please note that if you upload a new list, the old list will be overwrited!__ If you want to know how a Complex elem object should look like, please read [this](#structure) part of the documentation.

####Answer:
An object:
  - error {String} If the whole process is failed, otherwise null.
  - fails {Array} If a part of the items failed to insert, if all items inserted, it is null.
  - result {Array} List of all inserted Complex elems

___


##Gallery handling
If you want to host the uploaded images yourself and want to use your other hosted images as well, then there are a few routes to fulfil this functionality.

Basic operation: The user uploads an image in the api to our server, which uploads it to the given server. This requires you to implement an upload route on your server and [configure](#configure-api-servers-gallery) our server (you have to do the configuration only once). The user can delete the images as well, so there should be an delete route on your server (it should be in the [gallery configuration](#configure-api-servers-gallery)) .

___

### Upload route
You should implement a route on your server which we can use for image upload.

Type: POST

We will upload the images as the following. There will be a "userId" field in the query of the request, it will contain the id of the user (who wants to upload the image). The image will be uplaoded as a file (the field name will be "file") and there will be another field "userId", it will contain the same data as the query field "userId"

The response to our post request should be an image object. The object should have the following properties:
  * url {String} /REQUIRED/ The url where the newly uploaded image can be found.
  * secure_url {String} Http secure version og the image url
  * thumb_url {String} An url where the thumbnail version of the image is available (Tha gallery can work a lot faster if you can provide a thumbnail)
  * name {String} The name of the image. If it is not given then we will you the last segment of the url
  * width {Number} The original width of the image (It can save a lot of process if you can provide this information)
  * height {Number} The original height of the image (It can save a lot of process if you can provide this information)

___

### Delete route
You should implement a route on your server side which we can use for deleting images.

Type: POST

We will post an objact with two parameters:
  * url {String} The url of the deleted image
  * userId {String} The id of the user who deleted the image.

Your response to our post request should be 200. (HTTP status code) 

___

###Configure api server's gallery
Configure the api's gallery.

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/gallery/config
  
#### Parameters (you should post):
  * uploadRoute {String} /REQUIRED/ The [route](#upload-route) which we can use for uploading the images. (This route should be implemented on your server.)
  * deleteRoute {String} /REQUIRED/ The [route](#delete-route) which we can use for deleting the images. (This route should be implemented on your server.)
  * noUploadText {Object} It contains the language code (see [available languages](#available-languages)) - html(string) pairs. The given content will be displayed if the upload is not allowed. 
  * limitReached {Object} It contains the language code (see [available languages](#available-languages)) - html(string) pairs. The given content will be displayed if a user reached the upload limit. It is not required to impose a limit on the upload. If you want to know how you can set the limit, please read the [feature switch](#feature-switch) chapter.
  * noUrl {Boolean} With this boolean you can forbid the image handling from absolute urls. If you set this parameter true, then none of your users will be able to use images from absoulte url.
 
####Answer:
The data you posted:
  - uploadRoute {String}
  - deleteRoute {String}
  - noUploadText {Object}
  - limitReached {Object}
  - noUrl {Boolean}
or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

###Get configuration
Gets the configuration of the api gallery.

#####Type
  + GET

#####Route
  + //api.edmdesigner.com/json/gallery/config
  
####Answer:
Config object: 
  - uploadRoute {String} The [route](#upload-route) you set for the uploading or nothing if you not configured our server yet.
  - deleteRoute {String} The [route](#delete-route) you set for the deleting or nothing if you not configured our server yet.
  - noUploadText {Object} (only if you previously set it in)
  - limitReached {Object} (only if you previously set it)
  - noUrl {Boolean} (only if you previously set it true)
or it can be an error
or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

###List images
Lists all images of a specified user

#####Type
  + GET

#####Route
  + //api.edmdesigner.com/json/gallery/list/:id

####Parameters (in the route):
  * :id {String} The id of the target user
  
####Answer:
Array of urls

or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

###Add images
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
  
####Answer:
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

###Delete images
Delete one or more images from one or more specified users

#####Type
  + DELETE

#####Route
  + //api.edmdesigner.com/json/gallery/delete

####Parameters (you should post):
  * images {Array} /REQUIRED/ should contain the urls of the images you want to remove from the given users.
  * users {Array} /REQUIRED/ should contain the ids of the users.
  
####Answer:
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

##Project handler admin routes

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

####Answer:
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

####Answer:
Project object:
  - _id {String} MongoDB _id of the newly created project

or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

###Default templates
Every api partner get a templater user. (id = "templater"). This speciel user have a few project by default which can be used for anything you want. With this route you can list this projects.

#####Type
  + GET

#####Route
  + //api.edmdesigner.com/json/project/defaults

####Answer:
List of projects. Every project is an object with the following parameters:
  - _id {String} MongoDB _id of the project
  - title {String} Title of the template
  - document {Object} An object, which represents a template
  - description {String} Description of the template

or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

##Headers & footers
You can upload headers and footers which your users will be able to use. A header will appear on the top of their templates and the footer will appear on the bottom.
The users won’t be able to edit these two elements or remove them.  
If you upload any header and/or footer, then the affected users have to use one. If you want to have a group of users who don’t have to use any kind of header and/or footer, then do not upload header and/or footer to all of your users ([general upload](#upload-header-2)) or upload the group an empty header and/or footer. This way they can choose the empty one and then there will be no header and/or footer on their templates.  
You can upload different kind of headers and/or footers. If you do so, then the affected users will be able to choose which header they want to use, but they still have to use one of them.   
The newly created document will get a header and/or footer if the user (who created the project) has any header and/or footer. A user can have header and/or footer if it was uploaded directly to him or to his group or if there are headers and/or footers which were uploaded to every users. (The header and/or footer priority is the following: user’s personal headers and/or footers <  user’s group’s headers and/or footers < general headers and/or footers)

There is three different ways to upload headers. You can upload it:
  - to all your user ([general header upload](#upload-headers-2)) 
  - to a specified group ([upload headers to group](#upload-headers)) 
  - to specified user or users ([upload headers to user](#upload-headers-1))

Footers can be uploaded in the same ways:
  - to everyone ([general footer upload](#upload-footers-2))
  - to a specified group ([upload footers to a group](#upload-footers))
  - to specified user or users ([upload footers to users](#upload-footers-1))


### Structure
The representing object fore header or footer should have the following properties:
  - document {Object} /REQUIRED/ it should be a json object (which represent our templates)
    - root /REQUIRED/ There should be the structure of the header or footer
    - generalSettings
  - id {String} /REQUIRED/ this id is what we use for distinguish the headers from each other so __it should be unique!__ Please note that the id you want to use for the users headers should be different from the ids of the general headers or the ids of the user's group's headers. The same is true for the footers.
  - title {Object} it should contains language code - title string pairs. For example: 'en': 'Green-white header'. The title will appear on the dropdown list. If you don't want to use any other localization then please use the 'en' language code, the default will always be the 'en' regardless of the actual language!  
  - placeholders {Object} it is needed when you want to have more than one supported language on your headers or footers. If you want to know how do the placeholders work please read the [localization](#localization) part of the [headers & footers](#headers--footers) chapter.

___

### Localization
It is possible to use different localization with the same headers and/or footers but __not required__.  
If you want to support only one language then you should "hardcode" the content of the header or footer to the representing json and it should work perfectly. In that case you do not have to use the placeholders object (see [stucture](#structure)).  
If you want to support more than one language, then you have to use the placeholders object. You should not write the actual content to the json instead you should use a placeholder (for example if you have a title element in your header or footer, then the text of the title should be the placeholder instead of the actual content. We will replace the placeholders in our application). You can give any kind of name to this placeholder, but it should begin and end with two '#' character (for example: ##title##.). After that you have to insert the placeholder to the placeholders object as follows: the key should be the name of the placeholder, and the value should be an object conatining the language code - value pairs of the actual content.  
Example placeholders object: 

	placeholders: { '##title##': {
				'en': 'The header is a good thing',
				'hu': 'A fejléc egy jó dolog',
				//...
			},
	 '##example-palceholder##: {
	 			'en': 'It is an example',
	 			'hu': 'Ez egy példa',
	 			//...
	 			}
	 }
	 
You can localize the name (title) of the headers and/or footers too. The title of a header or footer is the text which will appear on the dropdown list, where the user will be able to choose from them.  
If you want to support more than one language you just have to put the language code - text pair to the title object (see [structure](#structure)).
An example title object: 

	title: { 'en': 'example title',
		 'hu': 'példa cím'
		 /...
		}
		
If you support a language but you do not want to give the header or footer a different title in the other language, you do not have to, but please note that this way the default title will be used on the dropdown list for this header, and the default is always the 'en' (english) version of the title.  

Example:  
_We support the english and the hungarian languages. We upload two headers with the following two title objects: title: {'en': 'first 	header'} and title: {'en': 'second header', 'hu': 'Második fejléc'}. If someone select the hungarian language, then in his dropdown 		list there will be two selectable header: 'first header' and 'Második fejléc', but if someone choose the english version, then 		the dropdown list will look like the following: 'first header' and 'second header'._  

If you do not give an 'en' (english) title to a header, then it will be generated automatically in our application, so it is suggested to always upload an 'en' version of the title too.

___

### Upload Headers 
Upload a list of headers which __all of your users__ will be able to use. Please note that every upload will overwrite the previous uploads!  
If you want to know what a header is good for, please read the [headers & footers](#headers--footers) part of the documentation!

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/general/uploadHeaders

#### Parameters (you should post):
   * headers {Array} The list of headers which all of your users will be able to use. __Please note that if you upload a new list, the old list will be overwrited!__ If you want to know how a header object should look like, please read [this](#structure)

####Answer:
An object with two arrays:
  - success {Array} The list of the headers which were successfully saved
  - failed {Array} The list of the object: 
    - header {Object} The header which is not valid therefore it was not saved
    - error {String} The reason why the header is not valid

Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

### Upload Footers 
Upload a list of footers which __all of your users__ will be able to use. Please note that every upload will overwrite the previous uploads!  
If you want to know what a footer is good for, please read the [headers & footers](#headers--footers) part of the documentation!

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/general/uploadFooters

#### Parameters (you should post):
   * footers {Array} The list of footers which all of your users will be able to use. __Please note that if you upload a new list, the old list will be overwrited!__ If you want to know how a footer object should look like, please read [this](#structure) part of the documentation.

####Answer:
An object with two arrays:
  - success {Array} The list of the footers which were successfully saved
  - failed {Array} The list of the object: 
    - footer {Object} The footer which is not valid therefore it was not saved
    - error {String} The reason why the footer is not valid

Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___


##complexElements
The complex elem is a possible tool to save a BOX or MULITCOLL element with all of it contents to be able to reuse it. This feature is created for make the template editing more fast so more effective.
A complex elem can binded to a user, to a group of users or to all of the users.
When a user uses the editor these binding displays all together and can be used all of them.

The administrator of the application can upload complex elems to all of the above possibilities.
The user of the editor can save complex elems only for him/herself, those will be added to the admin defined items.
The editor users can also delete those complex elems what are binded to him/herself as a user.

These are the 3 ways to upload complex elems as an admin:
  - to all your user ([general complexElems upload](#upload-complexElems)) 
  - to a specified group ([upload complexElems to group](#upload-complexElems)) 
  - to specified user or users ([upload complexElems to user](#upload-complexElems))

___

### Structure
The representing object for a Complex elem should have the following properties:
  - doc {Object} /REQUIRED/ it should be a json object (which represent our templates)
    - type /REQUIRED/ It must be "BOX" or "MULTICOLUMN"
    - generalSettings
  - id {String} /optional/ this id is what will identify the item for you if you would like to manage it via admin
  - title {Object} /optional/ it should contains language code - title string pairs. For example: 'en': 'Green-white complexElem'. If you miss to give it, we it will receive a default name. The title will appear on the list of the complex element. If you don't want to use any other localization then please use the 'en' language code, the default will always be the 'en' regardless of the actual language!  

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


##User routes

##Authentication
To authenticate these routes, userId and a token (generated to the userId) are needed. Those to should be sent on the request's query.(somehow like this: ?user=userID&token=token ). Please note that every route need to be authenticated expect the one which generate the token. (//api.edmdesigner.com/api/token)

For example: //api.edmdesginer.com/json/project/list?user=userId&token=123456789

##Project routes

### List projects
Lists the projects of the actual user.

#####Type
  + GET

#####Route
  + //api.edmdesigner.com/json/project/list

####Answer:
List of projects. Every project is an object with the following parameters:
  - _id {String} MongoDB _id of the project
  - title {String} Title of the template
  - description {String} Description of the template

or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___	

### Create
Creates a new project (a new e-mail template).

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/project/list

#### Parameters (you should post):
  * title {String} The title of the new project.
  * description {String} The description of the new project.
  * document {Object} An object, which represents a template. By setting this param, you can create new projects based on your prepared custom templates.

#### Answer:
  - _id {String} MongoDB _id of the newly created project

or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___	
	
### Duplicate
Creates the exact copy of the project with the ID specified in projectId.

#####Type
  + GET

#####Route
  + //api.edmdesigner.com/json/project/duplicate/:id

#### Parameters (in the route):
  * :id {String} The id of the project. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list the projects of the user with the [/json/project/list](#list-projects) route.

#### Answer:
Project object:
  - _id {String} The MongoDB _id of the new template
  - title {String} The title of the new template
  - description {String} The description of the new template
  - document {Object} An object, which represents the new template

or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___	

### Remove
Removes a project.

#####Type
  + DELETE

#####Route
  + //api.edmdesigner.com/json/project/remove/:id

#### Parameters (in the route):
  * :id {String} The id of the project. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list the projects of the user with the [/json/project/list](#list-projects) route.

####Answer:
A number, it is 1 if the project was successfully deleted

or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___	

### Generate
Generates the bulletproof responsive HTML e-mail based on the projectId.

#####Type
  + GET

#####Route
  + //api.edmdesigner.com/json/project/generate/:id

#### Parameters (in the route):
  * :id {String} The id of the project. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list the projects of the user with the [/json/project/list](#list-projects) route.

####Answer
A bulletproof responsive HTML version of the given template

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

####Answer
Title object:
  - title {String} The title of the selected project

or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

### Update information
Updates the title or/and the description of the specified project

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/project/updateInfo

#### Parameters (you should post):
  * projectId {String} /REQUIRED/ The id of the project. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list the projects of the user with the [/json/project/list](#list-projects) route.
  * title {String} The title of the new project.
  * description {String} The description of the new project.

#### Answer:
  - _id {String} MongoDB _id of the newly created project
  - title {String} The title of the new template
  - description {String} The description of the new template
  - document {Object} An object, which represents the new template

or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

Feature Switch
---------------
You can allow or forbid some features of our application to a group of your users.
The list of the features you can set at the moment:  
  - gallery {Object}: contains the features of the gallery
    - noUpload {Boolean} If you set this parameter true then the users belong to this group cannot use the upload functionality. If you configured the gallery noUploadText (see [gallery config](#configure-api-servers-gallery)) text then it will appear on the upload tab, otherwise a default text will be used.
    - noUrl {Bollean} If you set this parameter true, then the users of this group cannot use any image not hosted by you. The image from url tab of the gallery will disappear if the feature does not allowed.
    - limit {Number} The number of images the user can upload. If you do not want to set any limit then leave this parameter undefined. If a user reach the limit, then he won't be able to upload any more image and if you configured the gallery limitReached (see [gallery config](#configure-api-servers-gallery)) text then it will be displayed otherwise a default text will be used.  Please note that the 0 limit and the noUpload are almost equivalent except a different message will appear on the upload tab if you use one or the other.

This list is constantly expanding with time!

---

JSON document descriptors
-------------------------
In some cases you - as an integrator - migth want to use the raw document format instead of using the editor and fetch the document from the created project. Some example cases could be when you want to set custom headers and footers to some of your users or you want to create complex or dynamic element.

We can distiguish two main element descriptors, from which a whole document can be composed. These two main types are the containers and the leaf elements.

Every element has its type property. The possible values are the followings: "BOX", "MULTICOLUMN", "ROOT", "TEXT", "TITLE", "IMAGE", "BUTTON".

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
Now, the root element is actually a box, only the type property differs. ({type: "ROOT", ... }) In the near future - when we will introduce the full width containers - it will change a little bit.

###Full width container
Not implemented yet.
The main feature of this element type will be that the background color can be full width, not only 600px as the root element.

##Leaf elements
Leaf elements cannot have any child elements. These elements are typically the content elements.

###Text
The text element is super simple. It can contain html code, but you should not put very complex things in it. In our editor we enable the simple text formatting options (bold, italic, underlined) and lists (ul, ol) and some other very simple things like h1, h2, h3. Example:

	{
		text: "yo <i>this</i> is Sparta"
	}

###Title
Title elements are just like texts, but we enable the users to set only h1, h2 and h3 as formatting.

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

Available Languages
-------------------
You can set the localization which the api will use. (if you want to know how, please check the [edmDesignerApi.openProject](#edmdesignerapiopenprojectprojectid-callback-onerrorcb) function!)  
Available languages:
 - English (code: 'en')
 - Hungarian (code: 'hu')

If the language you want to use is not available yet then please contact us with the following email address: info@edmdesigner.com  


Example implementations
-----------------------
  * [Node.js example](https://github.com/EDMdesigner/EDMdesigner-API-Example-Node.js)
  * [PHP example](https://github.com/EDMdesigner/EDMdesigner-API-Example-PHP)
  * [PHP admin example](https://github.com/EDMdesigner/EDMDesigner-API-Example-PHP-Admin)
  
Dependencies
------------
jQuery

