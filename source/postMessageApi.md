---
title: postMessage API

language_tabs:
  - json

toc_footers:
  - <a href='http://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - apiDocs

search: true
---


# Overview - postMessage API

<aside class="notice">
This part of the docs is in-progress.
</aside>


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
