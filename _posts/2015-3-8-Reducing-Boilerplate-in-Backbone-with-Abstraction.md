---
layout: post
title: Reducing Boilerplate in Backbone with Abstraction
---

Backbone is one of the least opiniated libraries that help you organize your code. Backbone just gives you models, collections, routers, views, a history object and events. The rest is up to you. This means that if you enjoy typing and copy-pasting, you will end up with a lot of boilerplate code. If you stick to DRY (don't repeat yourself) principles, this post is here to remind you that Backbone is not as verbose as you would think it is.

### Inheritance with Extend

Models, collections and views have many responsibilities in common. More often than not, these pieces of functionality can be abstracted. For instance, suppose that all Backbone Views have a method for scrolling to the top of their contents. Also suppose that Backbone Models have a feature to serialize the attributes of nested models and collections. Instead of defining a `scrollToTop` method in each and every view, then a `deepToJSON` method in each and every model, it makes more sense to create base classes and use it throughout the application.

```javascript
var BaseView = Backbone.View.extend( { 
    scrollToTop : function( ) { ... }
} );

var BaseModel = Backbone.Model.extend( {
	deepToJSON : function( ) { ... }
} );
```

Even though the Backbone documentation allows extending the prototype of Backbone objects, this possibility is never used in the code bases I contribute to. The main reason is that each extension should be named semantically. When Backbone Views are extended, the extension is not the same entity anymore. Prototype extensions can easily make your code confusing to read, and when applied without care, two extensions may even get into conflict with each other. Naming extensions forces developers to apply them with care. 

### Abstract Constructors

Suppose that you are developing a platform game and you use models for storing data on your characters. Each character has the same properties: 

- x and y location coordinates,
- vx and vx velocities,
- collision detection with a surface.

Unless our goal is to write a boring game, each character should move according to its own strategy. When animating a character, we have to update their position and speed in a method. This method cannot be implemented in the character class because it depends on the strategy of the actual characters. 

```javascript
var CharacterAbstractModel = Backbone.Model.extend( {
	defaults: function() {
	    x  : 0.0,
	    y  : 0.0,
	    vx : 0.0,
	    vy : 0.0
	},
	animate : function() {
	    throw new Error( 'animate() is not implemented.' );
	}
} );

var KoopaModel = CharacterAbstractModel.extend( {
	animate : function( ) {
	    // implement the strategy for this character
	}
} );
```

The animate method throws an error indicating that the animate method should be overriden in constructors derived from `CharacterAbstractModel`. The word Abstract in the name also indicates that this constructor should not be instantiated. Throwing an error is not an optional way of documenting the methods that have to be implemented. In order to give developers a proper overview, documentation should be more explicit. 


### Mixins

Creating extended constructors from base constructors works whenever the built hierarchy feels natural. Humans tend to accept things more easily if they can easily name them. For instance, an Audi is a Car. A Website Traffic Report is a Report. Other problem domains tend to be different and their extensibility is less flexible. One of the best examples is when we face with different mixins that we may or may not need. 

For instance, in a Backbone View, the following concerns may reappear:

- validation widgets
- tooltips 
- abstractions for two way data binding with Backbone Stickit
- a toolbar
- a preloader animation
- etc.

Even if we just take the above five examples into account, there are 120 different combinations for creating Backbone Views with or without these features. In addition, a name `ValidatingStickyTooltipView` does not seem to be a natural name either.Adding all these features to one base constructor would solve the naming problem, but it would violate the single responsibility principle by bloating base constructors.

The solution is to use mixins and include the responsibilities on demand, without modifying base constructors. This approach makes it possible to define mixins respecting the single responsibility principle. For instance, a Backbone View requiring all 5 mixins would be defined as follows:

```javascript
var RegistrationFormView = BaseView.extend( 
	_.extend( 
		{},
		ValidationMixin,
		TooltipsMixin,
		StickitMixin, 
		ToolbarMixin,
		PreloaderMixin,
		{
			// implementation of the register form
		} ) );
```

Recall the implementation of the `extend` underscore function. We have to start with extending the empty object, otherwise the contents of all mixins and the Registration Form View implementation would be injected into `ValidationMixin` because `extend` does not create a shallow copy of its first argument. 

### Abstraction made simple

Backbone lets you organize your code in a modular and extensible way without writing much boilerplate. Whenever you are thinking about reaching the copy-paste shortcuts to create a slightly modified version of one of your existing functions, think again. An abstracted base constructor or a mixin lets you increase the maintainability of your code. 
