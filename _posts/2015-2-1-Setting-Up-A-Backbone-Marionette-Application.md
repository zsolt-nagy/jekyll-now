---
layout: post
title: Setting up a Backbone+Marionette application
---

The below instructions describe how a new Backbone project can be set up. Even though there are automatized solutions to perform all these steps, I find it important to go through these steps at least once. There are many excellent developers in the industry who spend years developing and maintaining client side rich web applications. These people often deliver excellent solutions, but they also crash and burn when trying to set up a new project. This is your chance to fill in your knowledge gaps in the unlikely case that they exist. I will concentrate on the tools and the reasons for using them instead of giving you a step by step tutorial containing a boilerplate that matches my taste. 

When dealing with the terminal, commands will be presented in their pure form, assuming that you have the rights to execute them. In case execution is denied from you due to lack of administrator rights, write `sudo` before the command to be executed. Important: some packages will be downloaded from online repositories. When the exact version is not specified, some paths may become incorrect with time.



Installing Node Modules with NPM
-----------------------------

Create a folder for your test project. Our project will be placed in the `MarionetteTest` folder in the below example.

Create a file called `package.json`. This file describes dependencies for your project. Copy-paste the following JSON object into the package.json file:

{% highlight json %}
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
        "mocha": "latest",
        "sinon": "latest",
        "sinon-chai": "latest"
    }
}
{% endhighlight %}

All keys and values inside the package.json are obvious except the devDependencies. These dependencies are node packages that are to be installed by the Node Package Manager. All these dependencies can be placed in the following two groups:

- tools for automated testing (Mocha, Chai, SinonJs),
- task runner (Grunt) to run the automated tests and and prepare the application for development, testing and deployment

These dependencies are not used by the application directly, they just provide the infrastructure for testing and deploying the application. Look up each package here: https://www.npmjs.com/ to have an idea of what each package is good for.

> The dependencies object specifies that npm should grab the latest version from each package. It is also possible to specify an exact match, or even the latest version compatible with a given version. When relying on a dependency, it is very important to make sure that updating dependencies does not introduce a breaking change. For a full list of the dependency-specification syntax, check out the [package.json documentation](https://docs.npmjs.com/files/package.json). 

If you don't have the Node Package Manager, grab it from here: http://nodejs.org/. In case you already have npm, you might need to update it to its latest version. Execute `npm update -g` to do so.

Execute the command `npm install` from the folder where your `package.json` resides. This will install the above described node modules for you and place them in the `node_modules` folder. 

>Hint: in case you are using git, I suggest putting the `node_modules` folder in the git ignore list, otherwise you will end up committing unnecessary files to the online repository and you may end up with conflicts.

Setting up the application folder with Bower and RequireJs
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
        "marionette": "latest"
    }
}
```

Executing bower install will install all dependencies of our application in the folder specified by your `.bowerrc` file. Two important remarks:

- Bower requires `git` to be installed and globally accessible in your terminal or command line. 
- In case of packages like Marionette, all dependencies are implicitly downloaded. This is why both backbone.wreqr and backbone.babysitter appear in your libs folder.

An alternative option is to leave the dependencies object empty in `bower.json`. Then execute the following command in the terminal:

```
bower install jquery backbone underscore requirejs marionette --save
```

The save option will prefill the dependencies in `bower.json` with the versions downloaded by Bower. In case there is a conflict between the versions required by the dependencies, Bower lets you choose betwen them in the terminal. 

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
    'jquery'    : '../lib/jquery/dist/jquery',
    'underscore': '../lib/underscore/underscore',
    'backbone'  : '../lib/backbone/backbone',
    'backbone.babysitter': 
        '../lib/backbone.babysitter/lib/backbone.babysitter',
    'backbone.wreqr': 
        '../lib/backbone.wreqr/lib/backbone.wreqr',
    'backbone.marionette': 
        '../lib/marionette/lib/core/backbone.marionette'
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
      deps: [ 
          'backbone', 
          'backbone.babysitter', 
          'backbone.wreqr' 
      ]
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

I will leave the rest of the Backbone-Marionette related questions open. 

<a name="mocha-tests"></a>

Setting up the test environment
---------------------------------

Create a `test` folder in your application and a `spec` folder inside the `test` folder. Create a file in the test folder and call it `test.html`. This file will contain all the results of our automated tests. Paste the following contents in the file:

```html
<html>
    <head>
        <link rel="stylesheet" href="../node_modules/mocha/mocha.css" />
        <script type="text/javascript" 
                src="../node_modules/mocha/mocha.js"></script>
        <script type="text/javascript" 
                src="../node_modules/chai/chai.js"></script>
        <script type="text/javascript" 
                src="../node_modules/sinon/lib/sinon.js"></script>
        <script type="text/javascript">
			var expect = chai.expect;
        			var should = chai.should;
        			mocha.setup('bdd');

        			window.onload = function( )
        			{
        			    mocha.run(); 
        			}
        </script>
        <script type="text/javascript" 
                src="spec/example.spec.js"></script>
	</head>
	<body>
		<h1>Automated tests</h1>
		<div id="mocha"></div>
	</body>
</html>
```

* Mocha is our test runner. We include the test runner Javascript file and the default CSS so that the test results will look nice in the test html page
* Chai is the assertion library of our choice. We enable the expect (i.e. Expect X to be/have Y) and should (i.e. X should be/have Y) assertion styles and use the BDD (Behavior Driven Development) mocha setup
* `sinon.js` is a helpful library allowing us to create spies, stubs and mocks. In addition, sinon lets us create a fake server, catch server requests and respond those requests with fake payload
* `mocha.run()` should be called to run the included test spec files and verify the assertions inside, grouped by test cases and test suites
* `example.spec.js` is the only test spec file that we currently included. We will soon provide some dummy tests to see the test page in action
* The document body should include a node with id mocha

Create the file test/example.spec.js and paste the following content there:

```javascript
describe( 'Example Test Suite', function( )
{
    it( 'should pass', function() 
    {
        expect( true ).to.be.true;
    };
    it( 'should be pending' );
}
```

View `test.html` in the browser. You should see your automated tests. One of them should pass, while the other one should be pending.

Configure the Grunt task runner
-------------------------------

Now that both the application and the test page are set up, it is time to think about what kind of tasks need to be executed in order to develop, maintain, test and deploy the application.

Create a file called `Gruntfile.js` in the `MarionetteTest` folder. Paste the following code there:

```javascript
module.exports = function( grunt ) { 

    grunt.initConfig( {
    } );
    
    grunt.registerTask('default', [] );
}
```

The task runner is not running any errors at the moment. Test the Gruntfile by running it from the terminal by running the grunt task. Reminder: we installed grunt as a Node module in the section "Installing Node Modules with NPM".

```
~/MarionetteTest$ grunt

Done, without errors.
```

The list of tasks that can be added to the Gruntfile is quite big. In addition, even if you don't find a task suitable for automating a process related to development, testing or deployment, you can also execute arbitrary commands. What kind of tasks and commands are suggested?

- In most applications, SASS or LESS is used for writing maintainable CSS. Both `.sass` and `.less` can be compiled into `.css` that the browser can understand. The task responsible for converting `.sass` files into `.css` is `"grunt-contrib-compass"`
- When developing, operating and maintaining large Backbone/Marionette applications, typically a lot of files are created. For performance reasons, it makes sense to concatenate and minify the code and deploy the application in a way that the browser loads only one Javascript file from the server. The task `"grunt-contrib-uglify"` is responsible for this process. As a side benefit, as the name suggests, your code will also become quite ugly, therefore the outside world will have a hard time understanding details and organizing principles of your code. For smooth debugging however, deploying source maps to at least the QA server is suggested.
- Live reload: development is made easier if your browser is automatically reloaded whenever you change a file. 
- Execution of automated tests: Mocha tests can also be executed in the command line, using a headless browser like PhantomJs.
- RequireJs module compilation and dependency resolution can also be performed by the Gruntfile
- When you stick to coding standards, JsHint can be executed automatically to make sure your code meets your defined quality standards

Let's add automatic RequireJs compilation and uglification to the Gruntfile.

```javascript
module.exports = function( grunt ) { 

    // Configure tasks
    grunt.initConfig( {

	requirejs: {
	    compile: {
	        options: {
                    name           : "main",
	            baseUrl        : "app/js/",
	            mainConfigFile : "app/js/main.js",
	            out            : "app/js/main.min.js",
	            deps           : [ '../lib/requirejs/require' ],
	            optimize       : "none",
	            preserveLicenseComments: false,
	            generateSourceMaps : false
	        }
	    }
        },

	uglify : {
            dist: {
	        src  : [ 'app/js/main.min.js' ],
	        dest : 'app/js/main.min.js'
            }
	}    

    } );

    // Load tasks from the node_modules folder
    grunt.loadNpmTasks( 'grunt-contrib-requirejs' );
    grunt.loadNpmTasks( 'grunt-contrib-uglify' );
    
    // define tasks list
    grunt.registerTask( 
    	'default', 
    	[ 'requirejs', 'uglify' ] );
    grunt.registerTask( 
    	'require', [ 'require' ] );
}
```

Setting up Grunt consists of three parts: configuration options are set up, then the used modules are loaded from the `node_modules` folder, then the tasks are defined. A registered task can contain any number of loaded and configured tasks or commands. 
- When executing `grunt`, the file `main.min.js` is created containing compiled and uglified code,
- When executing `grunt require`, only the requireJs task is executed.

How to proceed from here
------------------------

First of all, I suggest following this tutorial step by step in order to familiarize yourself with the tools that make your life easier. The alternative is to hunt for scripts, include them one by one, and manually prepare your product for deployment. What about testing? Well, the alternative of automated testing is a documentation of manual test scenarios. Once a development leader at the military of a small country was asked to cut the budget of testing by half. Then he asked which half of the 1000 pages of manual test cases he should stop executing, the first half or the second half? 

Experiment with each tool until you are confident with handling it. This tutorial does not explain everything you have to know about these tools, therefore it is worth researching alternative configuration options. In addition, it is also worth trying out alternatives, such as Browserify instead of Bower, or another CommonJs compatible module loader instead of RequireJs. The EcmaScript 6 module system can also be used to provide polyfills for compatibility. 

This tutorial left many points open. For instance, the testing tools were hardly introduced, let alone the testing method (Test Driven, Acceptance Test Driven or Behavior Driven Development). Furthermore, even though Backbone and Marionette is accessible in the entry point of the application, we are far from being done when it comes to the actual setup of Backbone and Marionette objects. I will leave these tasks up to you for experimenting.
