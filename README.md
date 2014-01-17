EDMdesigner-API
===============

[EDMdesigner](http://www.edmdesigner.com) is a drag and drop tool for creating responsive HTML e-mail templates very fast and painlessly, by which your click-through rate will radically increase. This documentation is intended to give you a detailed description about the EDMdesigner-API with which you can integrate our editor into arbitrary web-based systems. (For example a CRM, CMS, WebShop or anything else what you can imagine.)

To start developing with our api, please require an API_KEY at info@edmdesigner.com.

Basically, the only thing you have to do is, to implement one side of the handshaking on your server. When the handshaking is done, you can use sent object, on which you can find the functions which are communicating with our server.

The handshaking is built into the initialization process, so if you implemented the handshaking on your side, everything goes automatically.

We provide example implementations which include the handshaking as well. You can find the continously broading list at the end of this page at the Example implementations section.

Initializing
------------
	
	<script src="path_to_your_jquery.js"></script>
	<script src="http://api.edmdesigner.com/EDMdesignerAPI.js?route=##handshaking_route##"></script>
	<script>
		initEDMdesignerPlugin("##userId##", function(edmDesignerApi) {
			...
		});
	</script>
	
First of all, jQuery has to be loaded before you load our API. In the second line you can see a route parameter in the script's src. By that, you can tell the script where to look for the handshaking implementation. For example if you implemented it in your index.php, you have to replace ##handshaking_route## with index.php.
The first parameter is an user id from your system. It can be any string. If you don't want to handle separate user accounts, just pass there your API_KEY.

On the resulting object (edmDesignerApi) you will find some functions through which you can interact our system.

### Handshaking
To implement the handshaking on your server, you need an API_KEY, a magic word (wich is delivered with your API_KEY), your user's ipv4 address and a timestamp. The logic which handles handshaking has to get the userId as well, because it has to be sent to our server as well. Our API implementation automatically sends the userId in a POST HTTP request. You set this user id by the first parameter of the initEDMdesignerPlugin.

After concatenating the API_KEY, the ipv4 address of your user, the timestamp (as string) and your magic word, you have to create an md5 hash of the resulting string.

	
	hash = md5(API_KEY + ipv4 + timestamp + magic)


After this you have to send the API_KEY, the userId, the ipv4 address, the timestamp and the generated md5 hash to our server at: api.edmdesigner.com/api/token through http.

If everything goes well, you get back a token, with which your user can be identified.

## API functions	
### edmDesignerApi.listProjects(callback)
Lists projects of the actual user.
#### Parameters:
  * callback

#### Example:
	
	<script>
		initEDMdesignerPlugin("TestUser", function(edmDesignerApi) {
			edmDesignerApi.listProjects(function(result) {
				console.log(result);
			});
		});
	</script>
	
This snippet writes the projects of the TestUser to the console.

### edmDesignerApi.createProject(data, callback)
Creates a new project. (A new e-mail template.)
#### Parameters:
  * data {Object}
    * data.title {String} The title of the new project.
    * data.description {String} The description of the new project.
  * callback(result) {Function}
    * the result param is an object on which you can find an _id property, which identifies the newly created project
    * result._id {String} The id of the new project. (This is a [MongoDB](http://www.mongodb.org/) id.)

#### Example:
	
	<script>
		initEDMdesignerPlugin("TestUser", function(edmDesignerApi) {
			edmDesignerApi.listProjects(function(result) {
				console.log(result._id);
			});
		});
	</script>
	

The snippet above creates a new e-mail template and the _id of the newly created project is written to the console.
	
### edmDesignerApi.duplicateProject(projectId, callback)
Creates the exact copy to the project with projectId.
#### Parameters:
  * projectId {String} The id of the project. Note that, it has to be a valid MongoDB _id. It's best if you use the values what you got when you listed the projects of the user with the edmDesignerAPI.listProjects function.
  * callback

#### Example:
	
	<script>
		initEDMdesignerPlugin("TestUser", function(edmDesignerApi) {
			edmDesignerApi.listProjects(function(result) {
				console.log(result);
			});
		});
	</script>
	
This.

### edmDesignerApi.removeProject(projectId, callback)
Removes a project.
#### Parameters:
  * projectId
  * callback

#### Example:
	
	<script>
		initEDMdesignerPlugin("TestUser", function(edmDesignerApi) {
			edmDesignerApi.listProjects(function(result) {
				console.log(result);
			});
		});
	</script>
	
This.

### edmDesignerApi.openProject(projectId, callback)
Opens a project.
#### Parameters:
  * projectId
  * callback

#### Example:
	
	<script>
		initEDMdesignerPlugin("TestUser", function(edmDesignerApi) {
			edmDesignerApi.listProjects(function(result) {
				console.log(result);
			});
		});
	</script>
	
This.

### edmDesignerApi.generateProject(projectId, callback)
Generates the bulletproof responsive HTML e-mail based on the projectId.
#### Parameters:
  * projectId
  * callback

#### Example:
	
	<script>
		initEDMdesignerPlugin("TestUser", function(edmDesignerApi) {
			edmDesignerApi.listProjects(function(result) {
				console.log(result);
			});
		});
	</script>
	
This.

Example implementations
-----------------------
  * [Node.js example](https://github.com/EDMdesigner/EDMdesigner-API-Example-Node.js)
  * [PHP example](https://github.com/EDMdesigner/EDMdesigner-API-Example-PHP)
  
Dependencies
------------
jQuery

