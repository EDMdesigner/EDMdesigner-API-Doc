---
title: API Reference - Advanced

language_tabs:
  - javascript
  - php


toc_footers:
  - <a href='http://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - apiDocs

search: true
---


# Overview - advanced topics

<aside class="notice">
This part of the docs is in-progress.
</aside>

Advanced topics.






# Custom Strings

```javascript
//Custom String object example:
var customStringsObject = {
  id: "validId",
  title: {
    en: "EDM beer"
  },
  items: [
    {
       replacer: "##Svijany##",
       label: "good beer 1"      
    },
    {
       replacer: "##Poutník##",
       label: {
          en: "good beer 2"
       }
    }
  ]
};
```

The custom strings are an option to provide predefined texts to your users what they can insert while editing text contents.
You can add custom strings to 
- all of your users 
- to specified users
- or to users of a specified group.

The custom strings can be added by bunch of custom string items. Each bunch contains a title what will be displayed as the label of the drop-down list in the text editor, and an id, what provides the identification. By this id it will be possible to remove the bunch.
The id can not contain any special characters, the title is a string or a localization object. In case of localization object it what must contain 'en' property.
Within an item the label property has the same rules.

  


## Add Custom Strings to all of the users of the apiClientInstance

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




## Add Custom Strings to specified users

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


## Add Custom Strings to specified groups of the apiClientInstance

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



## Get Custom Strings of all of the users of the apiClientInstance

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



## Get Custom Strings of a specified user

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


## Get Custom Strings of a specified group

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



## Remove a general Custom Strings by it's id

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



## Remove a user's Custom Strings by it's id

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



## Remove a group's Custom Strings by it's id

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











# Group handling
You have to generate an admin token!
## Group - list
### edmDesignerApi.listGroups(callback, onErrorCB)
Lists the groups you have
#### Parameters:
  * callback {Function} A function to be called if the request succeeds
  * onErrorCB {Function} A function to be called if the request fails
 

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
```javascript
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
```
### edmDesignerApi.createGroup(data, callback, onErrorCB)
Creates a new group
#### Parameters:
  * data {Object}
    * data.name {String} /REQUIRED/ The name you want to give to the new group
    * data.featureSwitch {Object} The features that are available for users belong to this group. There is an ever-expanding [list](#feature-switch) of possible features which you can choose from.
    * data.customData {Object} You can add custom informations to this group. You can save any kind of information. It is up to you, how you want to use it!
  * callback {Function} A function to be called if the request succeeds
  * onErrorCB {Function} A function to be called if the request fails


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

```javascript
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
```

### edmDesignerApi.getGroup(groupId, callback, onErrorCB)
Gets a specified group
#### Parameters:
   * groupId {String} The id of the group. Note that it has to be a valid MongoDB _id. It's best if you use the values that you got when you list your groups with the [edmDesignerAPI.listGroups](#edmdesignerapilistgroupscallback-onerrorcb) function.
   * callback {Function} A function to be called if the request succeeds
   * onErrorCB {Function} A function to be called if the request fails


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

```javascript
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
```

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









#Feature Switch
You can allow or forbid some features of our application to a group of your users.
The list of the features you can set at the moment:  
  - gallery {Object}: contains the features of the gallery
    - noUpload {Boolean} If you set this parameter true then the users belong to this group cannot use the upload functionality. If you configured the gallery noUploadText (see [gallery config](#configure-api-servers-gallery)) text then it will appear on the upload tab, otherwise a default text will be used.
    - noUrl {Bollean} If you set this parameter true, then the users of this group cannot use any image not hosted by you. The image from url tab of the gallery will disappear if the feature does not allowed.
    - limit {Number} The number of images the user can upload. If you do not want to set any limit then leave this parameter undefined. If a user reach the limit, then he won't be able to upload any more image and if you configured the gallery limitReached (see [gallery config](#configure-api-servers-gallery)) text then it will be displayed otherwise a default text will be used.  Please note that the 0 limit and the noUpload are almost equivalent except a different message will appear on the upload tab if you use one or the other.

This list is constantly expanding with time!






#Feature configuration
This is a new development which will be used everywhere from the next api version. Right now it is only used in a few features [texteditor buttons](#wysiwyg-texteditor-buttons).
It is quite similar to the [feater switch](#feature-switch), but it is a little bit "smarter". Here you can configure the features on four different level. Each level inherits the upper levels configuration but has greater priority.  Basicly the configurationions are merged into each other, but the lower levels always override the inherited configurations, if they have different configuration, but not everything, only the affected settings. The inheritance chain is the following:

apiClient -> apiClientInstance(apiKey) -> group -> user.

For example, we have a configuration named "feature" on apiClient level with value 5. We have to users with different groups, in one of this two groups we override this feature with value 8. So the user who belongs to this group will get a feature configuration with feature 8 (he inherited 5 from the apiClient, but his group overrided it to 8, so in the en he inherited feature 8) while the other user will get feature 5 (he inherited it from the apiClient's feature configuration).

If you want to disable a feature on one of this four levels, you have to give the feature a false value. (it is do not give any value to a feature then that feauter will be inherited from the upper levels, so it is important to give the feature a false value).

For example, we have a configuration named "feature" on apiClient level with value {"number": 5}. We have to users with different groups, in one of this two groups we override this feature with value false. So the user who belongs to this group won't get the feature configuration (he inherited {"number": 5} from the apiClient, but his group overrided it to false, so in the en he does not inherited this feature. this basicly means that this user cannot use the given feature) while the other user will get feature {"number": 5} (he inherited it from the apiClient's feature configuration, so he can use this feature).














# Favorite elements

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



### Structure
The representing object for a Complex elem should have the following properties:  
  - doc {Object} /REQUIRED/ it should be a json object (which represent our templates)
    - type /REQUIRED/ It must be ["BOX"](#box), ["MULTICOLUMN"](#multicolumn) or ["FULLWIDTH_CONTAINER"](#full-width-container)
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



## Upload Complex Elems to a Group
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


## Upload Complex elems to specified users 
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



## Upload Complex elems to all users 
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











#Project handler admin routes

##Create to
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







##Create from to
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







##Create from to with images
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








##Get Project
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






##Set Project
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






##Download as Zip
It is possible to export your template as a zip. You can use a simple user level token.

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
  - example request: //api.edmdesigner.com/json_v1.0.0/apiKey/yourapikey/user/youruserid/project/projectId/downloadAsZip?user=youruserid&token=youruserleveltoken&data={"config": {"needHtml": true, "needJson": false, "withoutSanitizing": false} }

####Response:
A zip which can contain the followings (it depends on the configuration, by default it will contain everything):
  - template.html {file} The html code of the template. It can be a sanitized version or not, it depends of your configuration. By default it will be sanitized.
  - project.json {file} The json version of the project.
  - images folder with the images of the template.
Please note that every images source (href, src, etc.) in the json and html file are replaced with the paths of the images in the images folder.













# Custom Data
If you want to save any kind of plus information for some of your data you can do it with using the customData field
You can save custom informations to:  
  - yourself /apiClientInstance/ ([addCustomData](#add-custom-data-to-yourself))
  - users (update user [server side](#update-user) or [client side](#edmdesignerapiupdateuseruserid-data-callback-onerrorcb))
  - groups (update group [server side](#update-one-group) or [client side](#edmdesignerapiupdategroupgroupid-data-callback-onerrorcb))
  - project (update project [server side](#update-information) or [client side](#edmdesignerapiupdateprojectinfoprojectid-data-callback-onerrorcb))  


## Add Custom Data to yourself
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




## Get my custom data
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




