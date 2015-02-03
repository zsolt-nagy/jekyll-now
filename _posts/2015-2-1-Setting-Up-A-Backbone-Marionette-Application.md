---
layout: post
title: Setting up a Backbone+Marionette application
---

The below instructions describe how a new Backbone project can be set up. Even though there are automatized solutions to perform all these steps, I find it important to go through these steps at least once. There are many excellent developers in the industry who spend years developing and maintaining client side rich web applications. These people often deliver excellent solutions, but they also crash and burn when trying to set up a new project. This is your chance to fill in your knowledge gaps in the unlikely case that they exist. 

When dealing with the terminal, I will write commands in their pure form assuming that you have the rights to execute them. In case execution is denied from you due to lack of administrator rights, write `sudo` before the command to be executed. Important: some packages will be downloaded from online repositories. When the exact version is not specified, some paths may become incorrect with time.

Installing Node Modules with NPM
-----------------------------

Create a folder for your test project. Our project will be placed in the `MarionetteTest` folder in the below example.

Create a file called `package.json`. This file describes dependencies for your project. Copy-paste the following JSON object into the package.json file:

```json
{
    "name": "MarionetteTest",
    "version": "0.0.1",
    "readme": "Text description for your application",
    "devDependencies": {
        "chai": "latest",
        "chai-jq": "latest",
        "grunt": "latest",
        "grunt-cli": "latest",
        "grunt-contrib-compass": "latest",
        "grunt-contrib-copy": "latest",
        "grunt-contrib-requirejs": "latest",
        "grunt-contrib-uglify": "latest",
        "grunt-css": "latest",
        "grunt-mocha": "latest",
        "grunt-mocha-phantomjs": "latest",
        "grunt-shell": "latest",
        "mocha": "latest",
        "sinon": "latest",
        "sinon-chai": "latest"
    }
}
```
All keys and values inside the package.json are obvious except the devDependencies. These dependencies are node packages that are to be installed by the Node Package Manager. All these dependencies can be placed in the following two groups:

- tools for automated testing (Mocha, Chai, SinonJs)
- task runner (Grunt) to run the automated tests and and prepare the application for deployment. 

These dependencies are not used by the application directly, they just provide the infrastructure for testing and deploying the application. Look up each package here: https://www.npmjs.com/ to have an idea of what each package is good for.

> The dependencies object specifies that npm should grab the latest version from each package. It is also possible to specify an exact match, or even the latest version compatible with a given version. When relying on a dependency, it is very important to make sure that updating dependencies does not introduce a breaking change. For a full list of the dependency-specification syntax, check out the [package.json documentation](https://docs.npmjs.com/files/package.json). 

If you don't have the Node Package Manager, grab it from here: http://nodejs.org/. In case you already have npm, you might need to update it to its latest version. Execute `npm update -g` to do so.

Execute the command `npm install` from the folder where your `package.json` resides. This will install the above described node modules for you and place them in the `node_modules` folder. 

>Hint: in case you are using git, I suggest putting the `node_modules` folder in the git ignore list, otherwise you will end up committing unnecessary files to the online repository and you may end up with conflicts.

Setting up the application folder with Bower and Grunt
---------------------------------------

Create an `app` folder inside your application and a `lib` folder inside app. We will place all frameworks and libraries we will use inside `app/lib`. We will use Bower to install the frameworks and libraries we will use to develop our application. Bower is a package manager for client-side development. 

Install Bower as a global Node package so that you can invoke it from any folder.

```
npm install bower -global
```

Bower can be configured with a JSON object. Create a `.bowerrc` file in your app folder and paste the below contents in it. 

```json
{
  "directory" : "lib",
  "json"      : "bower.json",
  "endpoint"  : "https://Bower.herokuapp.com",
  "searchpath"  : "",
  "shorthand_resolver" : ""
}
```

It is time to define the actual dependencies. Place a `bower.json` file in the same folder:

```json
{
    "name": "Marionette Test",
    "version": "0.0.1",
    "dependencies": {
        "jquery": "latest",
        "backbone": "latest",
        "underscore": "latest",
        "requirejs": "latest",
        "bootstrap": "latest",
        "marionette": "latest"
    }
}
```

Executing bower install will install all dependencies of our application in the folder specified by your `.bowerrc` file. Two important remarks:

- Bower requires `git` to be installed and globally accessible in your terminal or command line. 
- In case of packages like Marionette, all dependencies are implicitly downloaded. This is why both backbone.wreqr and backbone.babysitter appear in your libs folder.

> **Question**: What's the difference between NPM and Bower?
> **Answer**: After reading this section, you may conclude that Bower works very much like NPM. Although NPM is the largest available Javascript-based module library, Bower is optimized for client side development. The main advantage of Bower is that it does not allow multiple versions of the same dependency to end up in your application code. Downloading multiple copies of the same dependency is highly inefficient on client side. Let alone injecting a breaking change in a conflicting dependency. There are solutions for using NPM for client side dependency management, but we will stick to Bower in this tutorial.

> **Question:** How are dependencies of dependencies loaded by Bower and NPM exactly?
> **Answer:** Suppose we load the Marionette package both with Bower and with NPM. As you saw in the above example, Bower placed dependencies of Marionette directly in the `lib` folder, outside the folder of the retrieved Marionette package. NPM on the other hand places all dependencies of Marionette inside the Marionette folder. Suppose the following two lines were added to the devDependencies list of your package.json file.  
> ```json
>     "backbone.marionette": "latest",
    "underscore": "latest"
> ```
> After executing `npm install`, you will find two underscore folders: 
> 
> - one directly in `node_modules`,
> - one in `backbone.marionette`, inside its `node_modules` folder.

It is now time to organize the client side dependencies into modules and load them. Create an `index.html` file in the `app` folder and paste the following code there:

```html
<!DOCTYPE html>
<html>
    <head>
        <title>Marionette Test</title>
    </head>
    <body>
        <script data-main="js/main.js" 
                src="lib/requirejs/require.js">
        </script>
    </body>
</html>
```

`app/js/main.js` describes the RequireJs configuration:

```javascript
requirejs.config({
  paths: {
    'jquery': '../lib/jquery/dist/jquery',
    'underscore': '../lib/underscore/underscore',
    'backbone': '../lib/backbone/backbone',
    'backbone.babysitter': '../lib/backbone.babysitter/lib/backbone.babysitter',
    'backbone.wreqr': '../lib/backbone.wreqr/lib/backbone.wreqr',
    'backbone.marionette': '../lib/marionette/lib/core/backbone.marionette'
  },
  shim: {
    underscore: {
      exports: '_'
    },
    backbone: {
      exports: 'Backbone',
      deps: [ 'jquery', 'underscore' ]
    },
    'backbone.marionette': {
      exports: 'Backbone.Marionette',
      deps: [ 'backbone', 'backbone.babysitter', 'backbone.wreqr' ]
    }
  },
  deps: [ 'jquery', 'underscore' ]
} );
```

We have achieved the following by applying a RequireJs configuration:

- there is exactly one script tag in the index.html file, including the entry point of the application containing the RequireJs configuration. The alternative would be specifying all the Javascript files in Script tags in the order that does not break dependencies. This would not only be hard to read, but it is also quite upsetting to order your script tags so that dependencies come before a given package
- In the `paths` configuration, aliases have been created as shorter references
- `shims` specify dependencies of included packages in case a given package is not RequireJs compatible. 
- `deps` specify the dependencies of our application

> Since version 1.1.1 published in February 2014, Backbone is RequireJs compatible. This implies that in case you are using version 1.1.1 or later, specifying the shim for Backbone is not mandatory.

Run the application
-----------------------

A tedious tutorial-based configuration process is mostly not satisfying as the reader has to keep on reading and applying the steps without knowing if any mistakes were made in the process. Therefore, before continuing with the setup of our task runner and the setup of the test environment, we will insert a small code snippet into app/js/main.js to make sure that our application setup is basically ready for development. The following code snippet is going to be the entry point of our application:

```javascript
require([
    'jquery',
    'backbone',
    'backbone.marionette'
], function(
    $,
    Backbone,
    Marionette
) {

	$( document ).ready( function( )
	{
		console.log( 'Document is ready.' );
		console.log( 'jQuery: ', $ );
		console.log( 'Backbone: ', Backbone );
		console.log( 'Marionette: ', Marionette );
	} );

} );
```

This entry point verifies that both requirejs, jQuery, Backbone and Marionette are accessible once the DOM is ready.
