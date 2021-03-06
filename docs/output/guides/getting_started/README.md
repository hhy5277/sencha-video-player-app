# Getting Started

## 1. Requirements

### 1.1 Chrome Platform

Chrome Platform Apps are native applications written in HTML5, JavaScript and CSS. These applications are integrated into the user’s operating environment providing access to APIs and services not available to traditional web apps including Networking APIs, USB and Serial port access, and integrated user identity.

This tutorial assumes you have the Chrome Platform Apps already enabled and ready to use.

To read more on getting started we recommend you to visit [Google Chrome Apps developer site](http://developer.chrome.com/trunk/apps/about_apps.html)

### 1.2. Ext JS 4 SDK

Download [Ext JS 4 SDK][sdk-download]. Unzip the package to a new directory called "sdk" within your application root directory.

## 2. Application Structure

### 2.1 Basic Structure

Although not mandatory, all suggestions listed below should be considered as best-practice guidelines to keep your application well organized, extensible and maintainable.
The following is the recommended directory structure for an Ext JS application:

    - appname
        - app
            - namespace
                - Class1.js
                - Class2.js
                - ...
        - sdk
        - resources
            - css
            - images
            - ...
        - app.js
        - index.html
		- background.js
		- index.js
		- manifest.json
		- sandbox.html

- `appname` is a directory that contains all your application's source files
- `app` contains all your classes, the naming style of which should follow the convention listed in the [Class System](http://docs.sencha.com/ext-js/4-1/#!/guide/class_system) guide
- `sdk` contains the Ext JS 4 SDK files
- `resources` contains additional CSS and image files which are responsible for the look and feel of the application, as well as other static resources (XML, JSON, etc.)
- `index.html` is the entry-point HTML document
- `app.js` contains your application's logic
- `background.js` called by Chrome Platform to start your application
- `index.js` creates the sandbox iframe for Ext JS to run (read more below)
- `manifest.json` describe your application as part of Chrome Platform
- `sandbox.html` sandboxed iframe entry point of your application

### 2.2 Content Security Policy

Chrome Platform Apps run in a controlled environment that enforces compliance with Content Security Policy.
Developer should be aware, that there are some functions and methods either disabled, or not available. To name a few : alert(), localStorage and Flash.
Here is the full list of [Disabled Web Features](http://developer.chrome.com/trunk/apps/app_deprecated.html)

Since Ext JS need some higher privileges to execute, the solution found is to create an iframe that acts as a sandbox environment. This iframe has no access to native APIs, so we have the following architecture:

- `index.js` is the Chrome Platform app entry point. It creates the iframe which contains the Ext application and has access to native APIs.
- `app.js` is the Ext JS app entry point. It can execute all the Ext code and render the views, but has no access to native APIs.

So the parent page needs the iframe to render Ext JS components, and the iframe needs the parent page to have access to native APIs. They both comunicate using the
HTML5 Post Message API.

### 2.3 Chrome App entry point 

First of all you need to setup the manifest.json and background.js. These two files are mandatory for chrome applications.
If you would rather create just an extension, please have a look at the documentation provided on [Getting started with chrome extensions](http://developer.chrome.com/extensions/).

In your index.html, define the sandbox iframe as follow:

	<iframe id="sandbox-frame" class="sandboxed" sandbox="allow-scripts" src="sandbox.html"></iframe>

Iframe points to sandbox.html file where we set up files requred for your ExtJs application and frame communication.
Sample file would contain:

    <!DOCTYPE HTML>
    <html>
    <head>
        <link rel="stylesheet" type="text/css" href="resources/css/app.css" />'
        <script src="sdk/ext-all-dev.js"></script>'
        <script src="lib/ext/data/PostMessage.js"></script>'
        <script src="lib/ChromeProxy.js"></script>'
        <script src="app.js"></script>
    </head>
    <body></body>
    </html>

And on index.js, we get reference to this iframe for communication between frames:

	var iframe = document.getElementById('sandbox-frame');

	iframeWindow = iframe.contentWindow;

With this you'll have the start point for your Ext Application. You can check [additional Ext JS guides](http://docs.sencha.com/ext-js/4-1/#!/guide) on how to create your
first views, controllers and models.

### 3 Native APIs

In order to Ext JS app gain access to native APIs, like query the network with UPNP for media servers, it needs to communicate the parent page. This is done by Post Message, and Ext
has a proxy for that.

	Ext.data.PostMessage.request({
	    key: 'extension-baseurl',
	    success: function(data) {
	        //...
	    }
	});
	
This will send a message to the parent page with the key 'extension-baseurl'. The parent page now should listen for this message and reply back:

	window.addEventListener('message', function(e) {
	    var data= e.data,
	        key = data.key;

	    console.log('[index.js] Post Message received with key ' + key);

	    switch (key) {
	        case 'extension-baseurl':
	            extensionBaseUrl(data);
	            break;
            
	        case 'upnp-discover':
	            upnpDiscover(data);
	            break;
            
	        case 'upnp-browse':
	            upnpBrowse(data);
	            break;
            
	        case 'play-media':
	            playMedia(data);
	            break;
            
	        case 'download-media':
	            downloadMedia(data);
	            break;
            
	        case 'cancel-download':
	            cancelDownload(data);
	            break;
        
	        default:
	            console.log('[index.js] unidentified key for Post Message: "' + key + '"');
	    }
	}, false);
	
	function extensionBaseUrl(data) {
	    data.result = chrome.extension.getURL('/');
	    iframeWindow.postMessage(data, '*');
	}
	
At the example above the Ext app requests the base chrome URL. The parent page receives the request, assigns the result, and sends the data back.

[sdk-download]: http://www.sencha.com/products/extjs/