EDMDesigner-API-Example-PHP-Admin
=================================

Example project about integrating the the admin functionalities.

EDMdesigner-API
===============

[EDMdesigner](http://www.edmdesigner.com) is a drag and drop tool for creating responsive HTML e-mail templates very quickly and painlessly, radically increasing your click-through rate. This documentation is to give you a detailed description about the EDMdesigner-API with which you can integrate our editor into any web-based system (e.g. a CRM, CMS, WebShop or anything else you can imagine).

To start developing with our api, please request an API_KEY (and a corresponding magic word) at info@edmdesigner.com.

Basically, the only thing you have to do is to implement one side of the handshaking on your server. When the handshaking is done, you can use the sent object, in which you can find the functions that are communicating with our server.

The handshaking is built into the initialization process, so if you implemented the handshaking on your side, everything will work automatically.

We provide example implementations that include the handshaking as well. You can find the continously broading list at the end of this page at the Example implementations section.

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
  * projectId {String} The id of the project. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you listed the projects of the user with the edmDesignerAPI.listProjects function.
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
  * projectId {String} The id of the project. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you listed the projects of the user with the edmDesignerAPI.listProjects function.
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

### edmDesignerApi.openProject(projectId, callback, onErrorCB)
Opens a project.
#### Parameters:
  * projectId {String} The id of the project. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you listed the projects of the user with the edmDesignerAPI.listProjects function.
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
  * projectId {String} The id of the project. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you listed the projects of the user with the edmDesignerAPI.listProjects function.
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
    * data.featureSwitch {Object} The features that are available for users belong to this group. Please note that now it doesn't have any function, but later there will be a list of possible features which you can choose from.
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
   * groupId {String} The id of the group. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you listed your groups with the edmDesignerAPI.listGroups function.
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
   * groupId {String} The id of the group. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you listed your groups with the edmDesignerAPI.listGroups function.
   * data {Object}
     * data.name {String} The name you want to give to the group
     * data.featureSwitch {Object} The features that are available for users belong to this group. Please note that now it doesn't have any function, but later there will be a list of possible features which you can choose from.
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
    * data.group {String} The id of the group you want this user to belong. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you listed your groups with the edmDesignerAPI.listGroups function.
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
    * group {String} The id of the group you want this user to belong. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you listed your groups with the edmDesignerAPI.listGroups function.
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
    * data.group {String} The id of the group you want this user to belong. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you listed your groups with the edmDesignerAPI.listGroups function.
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

## Group manipulating routes
### List groups
Lists the groups you have

#####Type
  + GET

#####Route
  + //api.edmdesigner.com/json/groups/list

####Answer:
An array of your groups. Every group is an object with this parameters:
  - _id {String} MongoDB id of the group
  - featureSwitch {Object} The features that are available for users belong to this group. Please note that now it doesn't have any function, but later there will be a list of possible features which you can choose from.
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
  * featureSwitch {Object} The features that are available for users belong to this group. Please note that now it doesn't have any function, but later there will be a list of possible features which you can choose from.

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
   * id {String} The id of the group. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you listed your groups with the /json/groups/list route.

####Answer:
A group object:
  - _id {String} MongoDB id of the group
  - featureSwitch {Object} The features that are available for users belong to this group. Please note that now it doesn't have any function, but later there will be a list of possible features which you can choose from.
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
   * _id {String} /REQUIRED/ The id of the group. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you listed your groups with the /json/groups/list route.
   * name {String} The name you want to give to the group
   * featureSwitch {Object} The features that are available for users belong to this group. Please note that now it doesn't have any function, but later there will be a list of possible features which you can choose from.

####Answer:
A newly updated group object:
  - _id {String} MongoDB id of the group
  - featureSwitch {Object} The features that are available for users belong to this group. Please note that now it doesn't have any function, but later there will be a list of possible features which you can choose from.
  - name {String} The name of the group

Or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

## User manipulating routes

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
  * group {String} The id of the group you want this user to belong. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you listed your groups with the /json/groups/list route.

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
    * group {String} The id of the group you want this user to belong. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you listed your groups with the /json/groups/list route.

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
   * group {String} The id of the group you want this user to belong. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you listed your groups with the /json/groups/list route.

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

##Gallery handling
If you want to host the uploaded images yourself and want to use your other hosted images as well, then there are a few routes to fulfil this functionality.

Basic operation: The user uploads an image in the api to our server, which uploads it to the given server. This requires you to implement an upload route on your server and configure our server (you have to do the configuration only once).

###Configure api server
Set the api server the route where it can upload the images. (This route should be implemented on your server.)

#####Type
  + POST

#####Route
  + //api.edmdesigner.com/json/gallery/config
  
#### Parameters (you should post):
  * route {String} The route which we can use for uploading the images.
 
####Answer:
The data you posted:
  - route {String}
or it can be an error object:
  - err Description of the error {String} or an error code {Number}.

___

###Get configuration
Gets the configuration of the api gallery. It returns the route you set for the uploading

#####Type
  + GET

#####Route
  + //api.edmdesigner.com/json/gallery/config
  
####Answer:
Config object: 
  - route {String} The route you set for the uploading or nothing if you not configured our server yet. 
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
  * :id {String} The id of the project. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you listed the projects of the user with the /json/project/list route.

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
  * :id {String} The id of the project. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you listed the projects of the user with the /json/project/list route.

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
  * :id {String} The id of the project. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you listed the projects of the user with the /json/project/list route.

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
  * :id {String} The id of the project. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you listed the projects of the user with the /json/project/list route.

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
  * projectId {String} /REQUIRED/ The id of the project. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you listed the projects of the user with the /json/project/list route.
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


Example implementations
-----------------------
  * [Node.js example](https://github.com/EDMdesigner/EDMdesigner-API-Example-Node.js)
  * [PHP example](https://github.com/EDMdesigner/EDMdesigner-API-Example-PHP)
  * [PHP admin example](https://github.com/EDMdesigner/EDMDesigner-API-Example-PHP-Admin)
  
Dependencies
------------
jQuery

