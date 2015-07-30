# Other resources

There are several resources from where you can learn more about EDMdesigner: the official documentation, the tutorials and the example implementations.


## Docs

Document | Description
---|---
[Basics](./index.html) | This documentation covers the very basics of the integration, like:<ul><li>generating an access token</li><li>making API calls</li><li>managing and opening projects</li><li>managing users</li><li>gallery handling</li></ul>
[Advanced](./advanced.html) | This documentation covers more advanced topics. Probably you will only need some of these functionalities. <ul><li>Group handling</li><li>String placeholders</li><li>Dynamic data</li><li>Favorite elements' management</li><li>Working with custom data</li><li>Advanced project handling</li><li>API key management</li></ul>
[postMessage API](./postMessageApi.html) | By reading this documentation you can learn about how to manipulate the editor itself from the client side. Basically this client side API is based on the [postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) function with which you can send messages between frames from different domains. <ul><li>Basic messages and responses (eg. manual save)</li><li>Hooking on editor events (eg. using your own gallery instead of the built-in one)</li><li>Dynamic data</li><li>Setting elements' properties</li></ul>
[Element descriptors](./documentDescriptors.html) | A detailed description of our element types used in projects and favorite elements. We generate all of our outputs based on these JSON descriptors.
[Old built-in gallery](./builtInGallery.html) | Don't use it. Use the new postMessage based solution.
[Migrating to the CDN based editor](./migratingToTheCDNBasedEditor.html) | In this article you will learn about how to migrate from the good old version of the editor to the superfast and robust [CDN based version](http://edmdesigner.com/blog/update-edmdesigner-uses-amazon-cloudfront).



## Tutorials

Tutorial | Description
---|---
[The Basics](./tutorialTheBasics.html) | In this tutorial you will learn about our dashboard, the access token generation and the very basic functionalities, like creating and opening projects.
[Admin Functionalities](./tutorialAdminFunctionalities.html) | The biggest take away of this tutorial that you will see how to create new users in our system, so you will be able to associate one user for each of users in your system.
[Old Built-in Gallery Configuration](./tutorialGalleryConfig.html) | In this tutorial you will learn about how to set up the gallery properly, how to implement the hooks on your side and you will also learn about the security of these hooks.


## Example implementations
Our PHP examples are very well maintained, but the others are probably not up-to-date. Still, they can be a good starting point, but you might want to take a look at the PHP examples as well in the meanwhile.

  * [PHP example](https://github.com/EDMdesigner/EDMdesigner-API-Example-PHP)
  * [PHP admin example](https://github.com/EDMdesigner/EDMDesigner-API-Example-PHP-Admin)
  * [Node.js example](https://github.com/EDMdesigner/EDMdesigner-API-Example-Node.js)
  * [Cold Fusion example](https://github.com/EDMdesigner/EDMdesigner-API-Example-ColdFusion) - Thank you Tom Chiverton from Extravision! :)
  * [Django example](https://github.com/EDMdesigner/EDMDesigner-API-Example-Django) - Thank you Ferenc Vehmann from ZenHeads! :)

 If you have another example implementation, please let us know, we will fork it and will put a link here.



