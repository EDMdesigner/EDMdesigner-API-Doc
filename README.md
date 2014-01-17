EDMdesigner-API
===============

Documentation of the EDMdesigner API for integration projects. By using our API you can easily integrate EDMdesigner's editor to any web based environment.

Dependencies
------------
jQuery

Handshaking
-----------

Figure about the handshaking process
Ide kell a nodejs-es és a php-s példa

initEDMdesignerPlugin


Initializing
------------
	
	
	<script src="http://api.edmdesigner.com/EDMdesignerAPI.js"></script>
	<script>
		initEDMdesignerPlugin("TestUser", function(edmDesignerApi) {
			...
		});
	</script>
	
Itt lehet beszélni arról, hogy Testuser stb.

### Handshaking
api.edmdesigner.com/api/token
	
createProject

	return {
		listProjects:		function(callback) { ... },
		createProject:		function(data, callback) { ... },
		duplicateProject:	function(projectId, callback) { ... },
		removeProject:		function(projectId, callback) { ... },
		openProject:		function(projectId, callback) { ... },
		generateProject:	function(projectId, callback) { ... }
	};

User azonosítás!

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
