[UnderscoreJs](http://www.underscorejs.org) is a utility-belt library for Javascript providing a set of functions well known from the world of functional programming. These utility functions allow software development in a stateless way, without side-effects.  Javascript objects have to be treated as *immutable data*, otherwise UnderscoreJs would lose its real purpose.

> Immutable data cannot be changed. While in object oriented programming, the state of an object can be mutated by its methods, programs written in a functional style treat function parameters as value types that never change during the execution of a function. 

According to [npmjs.org](https://www.npmjs.org/), at the time of writing this article, UnderscoreJs is the most depended upon Node module, while its fork, [lodash](https://www.npmjs.org/package/lodash) is ranked as fourth in the same list. As Underscore is very popular, most Javascript developers have experimented with it. Especially in the first couple of projects, it is very easy to write syntactically correct, semantically incorrect code. In addition, bad habits can be anchored over time, resulting in suboptimal solutions to stay in the code.

Collection utilities
--------

If there is one pattern I regularly noticed with beginners, it was relying on the `_.each` too much. This is understandable as `each` is probably the first collection utility people learn as

- it is at the top of the list of [collection utilities](http://underscorejs.org/#collections)
- most other collection utilities can be expressed with `each` and some external state variables. 



In practice however, I only suggest using the `each` function whenever every element of a collection has to be processed. When thinking about when to exactly use `each`, assembling templates comes to my mind. In most other cases, it is easy to find another Underscore utility function, or when everything else fail, I find a `for` loop more readable than an `_.each` construct.

For instance, suppose we have a collection of items in a shopping cart.

```javascript
var cart = [
    { name: 'Apple Doughnut', price: 1 },
    { name: 'Cherry Tomatoes', price: 2.49 },
    { name: 'Smoked Salmon', price: 3.99 }
];
```

Let's solve the following queries using `_.each`:
1. Create a function that returns the total value of the cart.
2. Create a function that returns the price of an item in the cart.

```javascript
// Question 1
var getTotalValue = function( items ) {
    var sum = 0;
    
    var addToSum = function( item ) { 
        sum += item.price;
    }
    
    _.each( items, addToSum );
    
    return sum;
}
```

Although `items` is still treated as immutable, the implementation is not very elegant as the `sum` is computed as a side effect of running `_.each`. Functional programming is not about side effects, right? There is a better way to solve the same problem without side effects.

```javascript
var getTotalValue = function( items ) {
    var addItems = function( memo, item ) { 
        return memo + item.price;
    }
    
    return _.foldl( items, addItems, 0 );
}
```

The above function is now side-effect free.

The second example can also be solved with or without `_.each`. 

```javascript
var getPrice1 = function( items, itemName ) {
    var price;
    
    _.each( items, function( item ) {
        if( item.name == itemName ) {
            price = item.price;
        }
    } );
    
    return price;
}

var getPrice2 = function( items, itemName ) {
    var item =  _.findWhere( items, { name: itemName } );
    
    return item.price || null; 
}
```

It is not a surprise that the first implementation solves the problem using the side effect of executing the anonymous function inside `_.each`. There is also an efficiency-concern: as soon as the target item is found, it would make sense to break out from the iteration, but apparently `_.each` does not make it possible. Inserting a return statement in the anonymous function inside `getPrice1` stops execution of the function, but then `_.each` will start executing the same function on the following element of the collection. 

> When we want to break an `_.each` iteration, `_.find` comes to the rescue. The difference between the two functions is that `_.find` stops iteration as soon as its iterator function returns `true`, breaking the iteration.

Anonymous functions
--------

In the above example, the function `getPrice1` contains another debatable solution: an anonymous function. When establishing coding standards, it is important to know whether allowing a code construct increases or decreases the maintainability of your code. Reading the code still comes natural, maybe even more natural than the application of a reference to a named function defined elsewhere. The missing function name in the stack trace could make debugging harder. In practice, this has never been a concern for me when trying to locate a bug.

The real danger comes with nesting Underscore functions. For instance, the following problem requires the application of a combination of Underscore functions:

Suppose an array of Integers is given referenced by `arr`. Return the array of unique mod 10 remainders of the numbers in the array in descending order.

```javascript
_.sortBy( 
    _.uniq( 
        _.map( arr, function( item ) { 
            return item % 10; 
        } ) 
    ), 
    function( item ) { 
        return -item; 
    } 
);
```

Most people interpret this code ugly to read. Even with logically sane indentation, it is tough to follow which function belongs where. Unlike most functional languages, UnderscoreJs is not a good fit for describing a chain of functions without additional syntactic sugar. Even if it's easy to read for some people, eventually we can describe more complex scenarios with more complex functions that will become hard to read.

I have already shown one option to make code more readable: by naming functions and partial results:

```javascript
var mod10 = function( num ) { return num % 10; }
var arrMod10 = _.map( arr, mod10 );
var remainders = _.uniq( arrMod10 );
var sorter = function( item ) { return -item; }
var result = _.sortBy( remainders, sorter );
```

The code has become more easily readable and debuggable. This solution is satisfactory at first glance, but we can do better to make the code more compact. Underscore provides us with a function to `chain` a sequence of functions. 

```javascript
var mod10 = function( num ) { return num % 10; }
var sorter = function( item ) { return -item; }
var result = _.chain( arr )
              .map( mod10 )
              .uniq()
              .sortBy( sorter )
              .value();
```

Chaining passes intermediate results as the first argument of the following function. The `value` function is called at the end of the chain to return the final result. 

It is up to individual interpretation whether it makes sense to name the functions `mod10` and `sorter` or it's better to leave them as anonymous functions. 


Object Oriented notation
-------

All the examples so far have made use of the functional notation of Underscore. For any `utilityFunction` transforming `data` and having `argument1`, `argument2`, ... as arguments, this notation looks like the following:

```javascript
_.utilityFunction( data, argument1, argument2, ... );
```

It is also possible to apply a syntactic sugar that lets us express the same transformation in an object oriented way:

```javascript
_( data ).utilityFunction( argument1, argument2, ... );
```

`_( data )` creates a wrapped object, allowing us to execute any utility functions directly. This notation is not directly compatible with chaining. Therefore in Underscore, the object oriented notation of the chaining example looks like the following:

```javascript
var mod10 = function( num ) { return num % 10; }
var sorter = function( item ) { return -item; }
_( _( _( arr ).map( mod10 ) ).uniq() ).sortBy( sorter );
```

Some people prefer the functional notation while others favor writing code in an object oriented way. On one hand, Underscore gave us functional programming support for a purpose. On the other hand, the object oriented notation expresses the idea of applying a utility function on a given data object more clearly. I still lean towards using the functional notation first for clarity, second for implicit compatibility with Lodash. 

> Lodash has several different builds. The underscore build is fully compatible with Underscore. Other builds have major differences. Whenever using the Object Oriented notation on a variable `data`,  `_( data )` returns a `LodashWrapper` object. Lodash Wrappers provide [intuitive chaining](https://lodash.com/docs#_) which makes it possible to chain functions without calling `chain`. This also means that at the end of the chain, `value()` has to be called to retrieve the return value of the last function in the chain.

In builds of Lodash not compatible with Underscore, the following two expressions are equivalent and both return a Lodash Wrapper:

```javascript
var mod10 = function( num ) { return num % 10; }
var sorter = function( item ) { return -item; }

var lodashWrapper1 = _( _( _( arr ).map( mod10 ) ).uniq() ).sortBy( sorter );
var lodashWrapper2 = _( arr ).map( mod10 ).uniq().sortBy( sorter );
```

`lodashWrapper1.value()` and `lodashWrapper2.value()` return the exact same array of integers. 

As a consequence, when replacing Underscore with Lodash without special attention, the object oriented notation will result in unexpected errors manifesting in Lodash Wrappers returned when an array, a number or an object is expected.


Dynamic type checking
-------

There are multiple type checkers built in Underscore such as `isUndefined`, `isDate`, `isNumber` etc. When not used with care, they can encourage Javascript errors appear in the console. On one hand it is good because errors can be detected while testing the application. If these errors end up in a production environment, it's another story.

Let's check the implementation of `_.isUndefined` according to the [annotated source code](http://underscorejs.org/docs/underscore.html) of Underscore:

```javascript
_.isUndefined = function(obj) {
    return obj === void 0;
};
```

Suppose a variable is not undefined, but undeclared. In this case, the `isUndefined` function does not return a boolean value, but throws a `ReferenceError` instead. This is very good for debugging, as long as we make sure these errors are caught. One might ask who would make such stupid mistakes in the code to check if an undeclared variable is undefined? Apparently, it does not have to be you or any client side developer. It can also happen that someone "refactors" the API your single page application is using, and under certain special circumstances, a key in a GET request becomes omitted. 

I am not saying that these type checkers are wrong in general, they are just wrong in the wrong hands. Always use them defensively and never assume that you can get away with checking the type of a variable before checking if it is undeclared. In case you study the annotated source code, this should not be a concern for you. However, without proper knowledge, people often make assumptions. One assumption is that these type checkers work like the `typeof` operator in Javascript, which returns
- `"undefined"` for an undeclared variable,
- `"object"` for a variable having a value `null`.

Know the functions you are using!
- any Underscore type checker throws a reference error if its argument is not declared,
- `_.isObject( null )` returns `false`.

[Fixing the typeof operator](http://javascriptweblog.wordpress.com/2011/08/08/fixing-the-javascript-typeof-operator/) has been a concern for a while. Underscore gives you a more consistent approach which does not let you get away with checking undeclared types anymore.

Underscore templating after version 1.7.0
------

First and foremost, it is very important to emphasize that there is a potentially breaking change in version 1.7.0 of Underscore. Prior to version 1.7.0, the following code would return `"underscore.js"`:

```javascript
_.template( 'underscore.<%= extension %>', { extension: 'js' } )
```

This approach is now forbidden. A template function has to be created first, and the template variables can only be passed to the template function. This results in a more clear structure and encourages better performance via caching pre-compiled templates.

```javascript
var templateFunction = _.template( 'underscore.<%= extension %>' );
templateFunction( { extension: 'js' } );
```

Before upgrading to version 1.7.0, don't forget to check if your templates are written in the correct way. As the one step templating has never been really encouraged, it is likely that this change only breaks a small fraction of applications using Underscore templating.

Should I use Underscore templating at all?
-----

I have been told multiple times that Underscore templating lacks the functionality one would expect for building complex applications. My answer has always been that there is no better way to keep your templates simple and minimalistic as a complex web-application. While templates are quite hard to test and debug in general, Javascript code isn't. Therefore, the best advice I can give you is to stick to the principle of separating concerns and keep presentation logic out from the templates.

> Whether you use a Model-View-Controller, Model-View-Presenter or a Model-View-ViewModel framework, this separation is not only possible, but also encouraged. 

Normally, all you need in a template is 
- a set of template variables based on your model state and prepared by your presenter, controller or view model
- sequence of DOM elements
- selection ( `if`-`else`)
- iteration ( `_.each` )

This is all a templating solution needs and Underscore provides all these options in a simple way. The reason why I ended up using Underscore templating is that lately I have been using BackboneJs which comes equipped with underscore functions such as `_.template` by default.

Summary
-------

UnderscoreJs is very popular in the Javascript community. A utility belt library encouraging functional programming is a very good choice for writing elegant, descriptive, maintainable code when used with care. The following advice was given for writing solid code using Underscore:
- Think twice when using `_.each` just to rely on its side-effects. Most often than not, there is another collection utility for the same purpose.
- Tidy up your code by naming anonymous functions or use chaining to avoid building a pyramid-like code structure, also known as the Pyramid of Doom. 
- Unless UnderscoreJs is not used for functional programming purposes, the functional notation results in more natural, functional style code than the Object Oriented notation. 
- Underscore templates are partially evaluated from version 1.7.0.
- Underscore templating has everything you need from a templating engine when respecting separation of concerns.
