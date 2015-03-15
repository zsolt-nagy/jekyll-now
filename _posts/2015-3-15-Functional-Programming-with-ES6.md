---
layout: post
title: Functional Programming with ECMAScript6
---

The ES6 standard is expected this June. This means significant changes in writing Javascript code in general. Among other features, ECMAScript6 is equipped with a strong utility belt supporting functional programming. All the new features will make programming more enjoyable. Happy developers are often more productive than unhappy ones. Some unhappy developers tend to switch to technologies that are evolving faster. Others invent their own solutions for making the community happier. These inventions are rarely compatible with each other, resulting in more alternatives to evaluate and a lot to learn. Therefore ES6 came about the right time as it will keep most people entertained while it also helps the community speak the same language.

After a short explanation on when the expressive power of ES6 can be used in a real environment, we will dive into the features loosely or tightly related to writing code in a functional style. 

### When can I start using ES6?

ES6 can be used here and now for experimental purposes. In addition, if you are writing server side code with NodeJs, you can decide on using the `harmony` flag at any time you think it is stable. The real question is however when ES6 features can be used for client side development. 

The problem with browsers is that we often have to support browsers that will never implement the ES6 standards. If you have to support IE8 today, it will take years until you will be able to drop support for all browsers not equipped with ES6.

Does this mean that if we are lucky, we may be able to start writing code in ES6 by the time ES8 is published? As always there is a solution. You can start using all the new features of the language here and now with transpilers like <a href="https://babeljs.io/" target="_blank">BabelJs</a>. You can transform your application written ES6  into executable ES5 code doing simple transformations. 

The syntax is already here, we only have to wait for the performance benefits such as tail recursion. While babel comes with some sort of a <a href="https://github.com/babel/babel/issues/256" target="_blank">support</a> for tail recursion, the native features will definitely be faster. Therefore, as soon as ES6 is made fully available for some browsers, running native ES6 code for these browsers will be advised. 

The following examples can be translated and execute using BabelJs.

### Arrow Functions and Array Comprehension in ES6

As a warm up, the first task is going to be to filter an array and return the even numbers only. 

```javascript
var arr = [1,2,3,4,5];

// ES5
arr.filter( function( x ) { return x%2 === 0; } );

// First ES6 solution
[ for ( x of arr ) if ( x%2 === 0 ) x ];

// Second ES6 solution
arr.filter( x => x%2 === 0 );
```

The first ES6 solution introduces a new language construct called array comprehension. The `for-of` operator enumerates all items of the array while the condition of the `if` statement filters the enumeration.

The second solution is done by using arrow functions transforming `x` to a boolean:

- `( x => x%2 ===0)(1)` is `false`
- `( x => x%2 ===0)(2)` is `true`

Arrow functions are convenient shorthands. 


Let's calculate the sum of squares of all elements of an array. The solution with UnderscoreJs would look like this:

```javascript
_.chain( arr )
 .map( function(x) { return x*x; } )
 .foldl( function( a, b ) { return a+b; } )
 .value();
```

The ES6 version is a lot more compact regardless of whether we use array comprehension or arrow functions only:

```javascript
// ES6 solution with array comprehension
[ for (x of arr) x*x ].reduce( (x,y) => (x+y) );

// ES6 solution with arrow functions only
arr.map( x => x*x ).reduce( (x,y) => (x+y) );

// ES6 solution in the form of a function
var sumOfSquares = ( arr ) => 
		arr.map( x => x*x )
		   .reduce( (x,y) => (x+y) );
```

The above two examples show that array comprehension can be used instead of mapping or filtering. 

### Destructuring, Spread Operator

It is also possible to solve the sum of squares problem using recursion.

```javascript
var sumOfSquares = arr =>
    arr.length === 0 ? 0 : arr[0]*arr[0] + sumOfSquares( arr.splice( 1 ) ); 
```

When using ES6, tail recursion will be taken into consideration, making it worth considering recursive solutions. However, as mentioned before, at the time of writing this article, BabelJs does not come with this feature.

There are other features though that make it easier to handle arrays and an indefinite number of arguments.

- Destructuring:

```javascript
var a = 1, b = 2;
[a,b] = [b,a];
console.log( a, b );
// 2, 1

[a,,b] = [1,2,3,4,5];
console.log( a, b );
//1, 3
```

- Spread operator

```javascript
var array = [1,2,3,4,5];
[head, ...tail] = array;
console.log( head, tail );
// 1, [2,3,4,5]

var maxValue = Math.max( ...array );
console.log( maxValue );
// 5
```

The above constructs can be used to make the "sum of squares" example more compact:

```javascript
var sumOfSquares = ([ head, ...tail ]) =>
    typeof head === 'undefined' ? 0 : head*head + sumOfSquares( tail ); 
```

### Generators and Iterators

Languages like Haskell use lazy evaluation: values are only evaluated on demand. LazyJs, a fork of Underscore is also available for lazy evaluation in ES5. We do not even need solutions like LazyJs in ES6 as we can use generators instead.


```javascript
function* fibonacci( limit = Infinity ) 
{
  let a = 0, b = 1;

  while( a < limit ) 
  {
    yield a;
    [a, b] = [b, a + b];
  }
}

let iterator = fibonacci( 100 );

console.log( [ for ( elem of iterator ) elem ] );
//  [0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89]

let iterator2 = fibonacci();
console.log( 
	iterator2.next().value, 
	iterator2.next().value,
	iterator2.next().value );
//  0 1 1
```

Before exploring the generator function itself, it is worth noting the `limit = Infinity` construct. This is a way to describe the default values of an argument, and it is definitely shorter than manually describing the default value in case it is absent.

The function `fibonacci` is a generator. When the `yield` keyword is reached, the next value of the sequence is created. Then the execution continues and further values can be generated. The sequence is terminated as soon as the end of the generator function is reached and there are no more values to yield.

The `[a, b] = [b, a + b]` assignment is a shorthand for doing two assignments using temporary variables. See more examples for the destructuring section.

Calling the `fibonacci` generator function returns an iterable object. An iterable object can be exhaustively enumerated in a `for-of` loop. Alternatively, we can also call the `next()` method of an iterable object to access the next element of the sequence.

### Use ES6

Whenever performance-wise there are no significant drawbacks, using ES6 is strongly suggested as it boosts developer productivity. Experiment with all the features of ES6 and start using the ones you are comfortable with.
