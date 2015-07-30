---
title: API Reference - built-in gallery

language_tabs:
  - php


toc_footers:
  - <a href='http://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - apiDocs

search: true
---

#Gallery handling

<aside class="notice">
This part of the docs is in-progress.
</aside>

<aside class="warning">
We suggest you to use your <a href="./index.html#gallery-handling">own gallery</a> with postMessages.
</aside>

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



<!--
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

-->

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
