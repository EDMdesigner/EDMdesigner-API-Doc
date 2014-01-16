EDMdesigner-API
===============

Documentation of the EDMdesigner API for integration projects

Handshaking

initEDMdesignerPlugin


Routes

api.edmdesigner.com/EDMdesignerAPI.js

api.edmdesigner.com/api/token

api.edmdesigner.com/api/listProjects
api.edmdesigner.com/api/createProject
api.edmdesigner.com/api/openProject/:id
api.edmdesigner.com/api/duplicateProject/:id
api.edmdesigner.com/api/updateProjectInfo/:id
api.edmdesigner.com/api/removeProject/:id
api.edmdesigner.com/api/generateProject/:id

	<script>
		initEDMdesignerPlugin("pluginTest", function(edmPlugin) {
			...
		});
	</script>
	
createProject

	return {
		listProjects:		function(data, callback) { ... },
		createProject:		function(data, callback) { ... },
		duplicateProject:	function(projectId, callback) { ... },
		removeProject:		function(projectId, callback) { ... },
		openProject:		function(projectId, callback) { ... },
		generateProject:	function(projectId, callback) { ... }
	};

initEDMdesignerPlugin("pluginTest", function(edmPlugin) {
listProjects:		generateRequestFunction("listProjects"),

	createProject:		generateRequestFunction("createProject", true),
	duplicateProject:	generateRequestFunctionWithId("duplicateProject"),
	openProject:		generateRequestFunctionWithId("openProject"),
	removeProject:		generateRequestFunctionWithId("removeProject"),

	generateProject:	generateRequestFunctionWithId("generateProject")
