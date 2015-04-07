---
layout: post
title: Writing Automated Tests with Mocha and Chai
---

Many people claim that unit testing is a necessary condition for refactoring code. It is very hard to reliably refactor code without having good testing coverage. According to the definition of code refactoring, the process results in changes applied on the code without changing its external behavior. Good luck establishing a reliable refactoring process without automated testing. 

In addition to taking refactoring into consideration, having automated tests is essential for developing commercial Javascript applications. Even if you develop open source components, people will only place trust in your code if it is thoroughly tested. The only commonly accepted and repeatable proof of thorough testing is a collection of executable automated tests.

### Which testing libraries should I use?

The <a href="http://en.wikipedia.org/wiki/List_of_unit_testing_frameworks#JavaScript" target="_blank">list</a> is quite long. If you don't have unit tests in your repositories, any reliable and actively maintained testing library can be useful for you. My suggestion would be to focus on making your <a href="http://zsolt-nagy.github.io/Solid-Test-Driven-Development/" target="_blank">testing process</a> better instead of spending a lot of time on choosing the best testing library.

When I was included in making a decision about which testing libraries to use, it was evident to me that it was a battle between Jasmine and Mocha. The main reason why Mocha seemed to be a better choice was its modular nature. I prefer to afford the flexibility, extensibility and configurability provided by cooperating modular libraries. Mocha, accompanied by an assertion library and a mocking library does exactly that.

> We will not focus on mocking libraries in this article. It is still worth mentioning SinonJs. Sinon is a great tool for creating spies, stubs and mocks to provide you with runtime information on the object under test, and simplify its dependencies. In addition, Sinon lets you take full control of time and all server communication using fake timers and fake servers. This way you don't have to wait seconds or worst case minutes until the desired effect shows up, and you can also test your code in isolation, without communicating with the real server.

### Assertion styles

Assertions describe your expectations towards how your code should behave. The most popular assertion library used with Mocha is ChaiJs. According to the <a href="http://chaijs.com/guide/styles/" target="_blank">style guide</a>, ChaiJs offers three different assertion styles:

```javascript
// TDD assertion style: Assert
assert.equal( name, 'John' );

// BDD assertion styles: Expect, Should
expect( name ).to.equal( 'John' );
name.should.equal( 'John' );
```

All three styles are easily readable. Although it is a matter of taste, in my opinion, the Should assertion style is often slightly more natural to read and interpret than Expect. The Assert style lags behind the other two in terms of readability, simply because they cannot be read from left to right as sentences. Assert describes the relation between two values in a prefix notation, starting with the relation, followed by the operands. If your team prefers reading `> X 3` to `X > 3`, Assert will likely to be your choice. 

Readability is not everything. Notice that the only way to write assertions with `should`, is that the assertion library extends the Object prototype. This could lead to errors in rare extreme cases like trying to call the non-existing `should` method of `undefined`. More importantly, an assertion style not extending the Object prototype is simply cleaner. This is a strong reason in favor of the Expect style. As Expect is very close to the standard Jasmine syntax, choosing Expect makes your assertions familiar to Jasmine users. For all these reasons, I will stick to the Expect style in the rest of this article.

I did not consider that the assertion styles are grouped by the <a href="http://chaijs.com/guide/styles/" target="_blank">ChaiJs style guide</a> under the umbrella of BDD and TDD. This information is slightly misleading as any assertion syntax can be used for any testing processes. Recall my post on <a href="http://zsolt-nagy.github.io/Solid-Test-Driven-Development/" target="_blank">Test Driven Development</a> describing why your software development methodology should include some strategy that lets you write tests independently from the bias of implementation. The choice of assertion library has little to do in establishing these processes. 

### Building Blocks

```javascript
describe( 'Top level suite', function() {

	describe( 'Second level test suite', function() {

		before( function() {
			// executes once, before all tests 
		} );

		beforeEach( function() {
			// executes before each test of the suite
		} );

		after( function() {
			// executes once, after all tests
		} );

		afterEach( function() {
			// executes after each test of the suite
		} );

		it( 'should verify a behavior', function() {
			// a test containing assertions
		} );

		xit( 'should be a pending test', function() {
			// this test is pending. All assertions inside are ignored.
		} );	

	    it.skip( 'should also be a pending test', function() {
	        // it.skip is a longer, more semantic form of xit
	    } );		

		it( 'should also be a pending test' );
	} );

	describe( 'Another second level test suite', function() {

	} );
} );
```

The top level building block is called `describe`, responsible for describing a test suite. Test suites can be hierarchical, see the two second level test suites inside the top level test suite. Each test suite can have two types of setup and tear down functions: one running before/after each and every test, and one running before/after all tests. These functions are important so that each and every test case can focus on a piece of functionality instead of long setup and cleanup code. More importantly, each and every test can and should run in isolation with the help of correct setup and tear down functions.

The tests contain a text description also appearing in the report. This description should describe the functionality under test in a compact sentence. The second argument of an `it` test case is a function containing one or more assertions that have to be verified by the code.

Notice the three pending tests. Tests cases without implementation or `xit` test cases are automatically pending. Don't forget the advice of <a href="http://zsolt-nagy.github.io/Solid-Test-Driven-Development/" target="_blank">this article</a> about failing tests: never ever accept them. If you are aware of a failing test that you cannot repair immediately, make it pending until you have the resources to make it pass.

### Unit Testing advice

Many testers have trouble structuring and writing correct tests. The principles of writing maintainable code also apply for tests. I will enumerate some advice on writing maintainable tests:

```javascript
describe( 'Maintainable unit test cases...', function( ) {
    it( 'should test one type of behavior' );
    it( 'should not depend on any other test cases.' );
    it( 'should execute with exact same initial state as any other tests in the suite.' );
    it( 'should not matter in which order are these tests executed.' );
} );
```

I have read advice suggesting that you should write exactly one assertion per test case. This makes some sense given that if your first assertion fails, other assertions won't even be evaluated. On the other hand, common sense often overrode this suggestion, especially when I had to test pieces of functionality acting as a lookup table. For instance, if your task was to check how a roman numeral to decimal converter converts single digits, you have the following choices:

```javascript
// Strictly one assertion per test suite
it( 'should convert I to 1', function() {
	expect( romanToDecimal( 'I' ) ).to.equal( 1 );
} );
it( 'should convert V to 5', function() {
	expect( romanToDecimal( 'V' ) ).to.equal( 5 );
} );
//...

// Allow multiple assertions
it( 'should convert single digits', function() {
	expect( romanToDecimal( 'I' ) ).to.equal( 1 );
	expect( romanToDecimal( 'V' ) ).to.equal( 5 );
	// ...
} );
```

Note that at an expense of readability, it would be possible to write just one assertion for the "it should convert single digits" test case as well. After all it is possible to come up with a boolean expression and check if that expression is true. This however only proves the point that it is not the number of assertions that should be limited, but rather the number of different pieces of functionality. 

### Assertions

If you have not written automated tests before, in order to take full advantage of this post, I suggest <a href="http://zsolt-nagy.github.io/Setting-Up-A-Backbone-Marionette-Application/#mocha-tests" target="_blank">setting up Mocha and Chai</a> yourself so that you would have a starting point for writing an application guided by tests. 

The following lookup table will show how basic assertions can be written in Expect style. Feel free to copy-paste the assertions and run them in tests. Alternatively, download the examples of this section from <a href="https://github.com/zsolt-nagy/mocha-chai-sinon-cheatsheet" target="_blank">this repository</a>.

Let's start with type checking using the `a` and `an` functions. Note that opposed to the typeof operator, `null` is not an object.

```javascript
it( 'should check types', function() {
	expect( '1' ).to.be.a( 'string' );
	expect( 1 ).to.be.a( 'number' );
	expect( true ).to.be.a( 'boolean' );
	expect( {} ).to.be.an( 'object' );
	expect( null ).to.be.a( 'null' );
	expect( undefined ).to.be.a( 'undefined' );
	expect( null ).to.be.a( 'null' );
	expect( [] ).to.be.an( 'array' );
	expect( 0/0 ).to.be.a( 'number' );
} );
```

Chain objects such as `at`, `be`, `been` etc. can be written in any positions for the purpose of increasing readability. 

```javascript
it( 'should accept any number of chain objects', function() {
	expect( 'mocha' ).a( 'string' );
	expect( 'mocha' ).to.be.a( 'string' );
	expect( 'mocha' ).at.be.been.have.is.of.that.to.with.a( 'string' );
} );
```

Placing a `not` between the checked value and the checking function call negates the result. The assertion passes whenever the same assertion without `not` would fail.

```javascript
it( 'should handle negation', function() {
	expect( 5 ).to.not.be.a( 'string' );
} );
```

Due to loose typing, we sometimes have to deal with conditions that are truthy. A value is troothy whenever negating the value twice becomes true due to the implicit type cast done by the negation operator. Truthy values are `ok` while falsy values are `not.ok`.

```javascript
it( 'should check truthy and falsy values', function() {
	var o = {};

	// x is truthy iff !!x is true.
	expect( 'John' ).to.be.ok;
	expect( true ).to.be.ok;
	expect( o ).to.be.ok;
	expect( 1 ).to.be.ok;
	expect( 0 ).to.be.not.ok;

	// y is falsy iff !!x is false.		
	expect( null ).to.be.not.ok;
	expect( o.member ).to.be.not.ok;
	expect( "" ).to.be.not.ok;	
} );
```

Equality is a fundamental assertion. In fact, most assertions can be expressed using equality and native Javascript, without significantly reducing readability. Equality works like the `===` operator meaning that both values and types should equal. Two reference types are equal whenever they point at the exact same object, array or function. Their contents do not matter.

```javascript
it( 'should check equality', function() {
	var o = {};
	o.o = o;

	expect( 2*2 ).to.equal( 4 );
	expect( NaN ).to.not.equal( NaN );
	expect( 5 ).to.not.equal( '5' );
	// different object references are not equal
	expect( { a: 3 } ).to.not.equal( { a: 3 } ); 
	// references pointing at the same object are equal
	expect( o.o ).to.equal( o );
} );
```

When comparing reference values, we often use equality in a sense that all primitive values inside the compared objects in any depth should be equal to each other respectively. This is called a deep equality check, denoted by `deep.equal` or `eql`. 

I generally like solutions that explain themselves even if the reader is not familiar with the used library. As the difference between `eql` and `equal` requires additional knowledge about Chai, I prefer using the longer, but self-explanatory `deep.equal` form.

```javascript
it( 'should check deep equality of objects and arrays', function() {
	var o1 = { b:5, c: { d:6, e: [1,2,3] } };
	var o2 = { c: { d:6, e: [1,2,3] }, b:5 };
	expect( o1 ).to.deep.equal( o2 );
	expect( o1 ).to.eql( o2 );
	expect( o1 ).to.not.equal( o2 );
} );
```

Multiple checks can be chained using the `and`. Note that `and` does not shield its right part from the effects of `not` on the left. Therefore, applying `and` and `not` in the same assertion is discouraged due to readability reasons. 

```javascript
it( 'should chain multiple checks on the same object', function() {
	var name = 'Mocha';
	expect( name ).to.be.a( 'string' ).and
				  .to.equal( 'Mocha' );
} );
```

Beyond truthy and falsy values, `true` and `false` explicitly checks if a value is actually a boolean. 

```
it( 'should check true and false values', function() {
	var o = {};

	expect( true ).to.be.true;
	expect( true ).to.not.be.false;
	expect( o ).to.not.be.true;
	expect( o ).to.not.be.false;
} );
```

Checking against `null` and `undefined` was already solved in the very first assertion example of this section. As syntactic sugar, these checks can be shortened. In addition, `exist` checkes if a value is defined and is not `null`.

```javascript
it( 'should detect null, undefined and existing values', function() {
	var o = {};

	expect( o ).to.exist;
	expect( null ).to.not.exist;
	expect( null ).to.be.null;
	expect( null ).to.not.be.undefined;
	expect( o.member ).to.not.exist;
	expect( o.member ).to.not.be.null;
	expect( o.member ).to.be.undefined;
} );
```

When checking the structure of an object or an array, we only know how to check deep equality so far. Sometimes we are just interested in the existence of a couple of keys and/or values inside the tested value. Property checks and deep property checks are possible in the following way:

```javascript
it( 'should check object properties', function() {
	var player = {
		name: 'Mario', 
		coins: 55,
		inventory: {
			mushrooms: 2,
			stars: 3
		}
	};

	// check the existence of a key
	expect( player ).to.have.property( 'coins' );
	// check the existence of a key + check its value
	expect( player ).to.have.property( 'name', 'Mario' );
	// deep properties
	expect( player ).to.have.deep.property( 'inventory.stars', 3 );
} );
```

Length is a special property deserving some syntactic sugar. Regular property checks are also available for checking length if that is your preference.

```javascript
it( 'should check the length propoerty', function() {
	expect( [] ).to.have.length( 0 );
	expect( 'Mocha' ).to.have.length( 5 );
	expect( [] ).to.have.property( 'length', 0 );
	expect( 'Mocha' ).to.have.property( 'length', 5 );
} );
```

Property checks take the prototype chain into consideration. If you want to avoid the prototype chain and restrict property checks to own properties, use `ownProperty` instead of `property`.

```javascript
it( 'should check own properties', function() {
	var o = Object.create( { name: 'Mocha' } );

	expect( o ).to.have.property( 'name' );
	expect( o ).to.not.have.ownProperty( 'name' );
} );
```

The following assertions check if a value is a member of an array. This is purely syntactic sugar as a `contain` check can be expressed with comparing the value of an `indexOf` lookup with -1.

```javascript
it( 'should check if an array contains an element', function() {
	expect( [1, 3, 5] ).to.contain( 3 );
	expect( [1, 3, 5] ).to.not.contain( 4 );

	// contain expressed without contain
	expect( [1, 3, 5].indexOf( 3 ) ).to.not.equal( -1 );
} );
```

Especially, but not limited to serializing and unserializing JSON payloads when <a href="http://zsolt-nagy.github.io/Specifying-JSON-APIs/" target="_blank">communicating with an API</a>, checking the existence of each and every key is a very important test. It is even more important to check that no other keys are sent to the server as they may cause unwanted log lines or in some cases, the endpoint call may even be rejected. The `have.keys` check exhaustively checks all keys for exact matching, while the `contain.keys` check only checks the existence of the enumerated keys.

```javascript
it( 'should check keys of an object', function() {
	var jsonPayload =
	{
		name:    'John',
		email:   'john@user.com',
		country: 'NZL'
	}

	// exact matching of all keys
	expect( jsonPayload ).to.have.keys( [ 'name', 'email', 'country' ] );
	// exclusion of keys
	expect( jsonPayload ).to.not.have.keys( [ 'repeat_email' ] );
	// inclusion of keys
	expect( jsonPayload ).to.contain.keys( [ 'name' ] );	
} );
```

More syntactic sugar is coming, replacing the `>`, `<`, `>=`, `<=` and `instanceof` operators, matching arbitrary regular expressions, checking substrings in strings or satisfying an arbitrary function returning booleans. 

```javascript
it( 'should compare ordered values', function() {
	expect( 2 ).to.be.above( 1 );
	expect( 1 ).to.not.be.above( 1 );
	expect( 1 ).to.be.at.least( 1 );

	expect( 1 ).to.be.below( 2 );
	expect( 1 ).to.not.be.below( 1 );
	expect( 1 ).to.be.at.most( 1 );

	expect( 1 ).to.be.within( 0, 2 );
} );

it( 'should check the instanceof relationship', function() {
	var C = function( ) { this.a = 1; };
	var D = function( ) { this.b = 2; };
	D.prototype = new C();

	var c = new C();
	var d = new D();

	expect( c ).to.be.an.instanceof( C );
	expect( c ).to.not.be.an.instanceof( D );
	expect( d ).to.be.an.instanceof( C );
	expect( d ).to.be.an.instanceof( D );
	expect( c ).to.be.an.instanceof( Object );
	expect( d ).to.be.an.instanceof( Object );		
} );

it( 'should match regular expressions', function() {
	expect( 'John' ).to.match( /^(John|Jack)$/ );
} );

it( 'should contain a substring', function() {
	expect( 'John' ).to.have.string( 'hn' );
} );

it( 'should satisfy a condition described by a function', function() {
	var isShortName = function( name )
	{
		return typeof name === 'string' && 
		       name.length >= 2 && 
		       name.length <= 5;
	};

	expect( 'John' ).to.satisfy( isShortName );
	expect( 'Michael' ).to.not.satisfy( isShortName );
} );
```

An assertion fails whenever an error is thrown inside it. As we don't want failing tests for checking erroneous states, expecting a function to throw an error is a valid use case for testing error states. There are obviously workarounds for throwing an error whenever a specific type of error is not caught in native Javascript, but the error check is more easily readable.

```javascript
it( 'should throw an error', function() {
	var parse = function( response ) {
	    if( response.error ) {
	        throw new Error( response.error.message );
	    }
	    return response;
	};

	var testWrapper = function() {
	    parse( { error: { message: 'Server Error' } } );
	}

	expect( testWrapper ).to.Throw( 'Server Error' );
} );
```

### Preparing your cheat sheet

All the above examples are available in <a href="https://github.com/zsolt-nagy/mocha-chai-sinon-cheatsheet/blob/master/spec/cheatsheet.spec.js" target="_blank">this test suite</a>. The executed test suite can be viewed in a browser, where each test case can be expanded and collapsed in order to view the assertions inside the test cases.

<img src="http://zsolt-nagy.github.io/images/posts/cheatsheet.png" alt="Mocha-Chai Cheat Sheet" />

Steps for preparing the test file:

1. Clone the <a href="https://github.com/zsolt-nagy/mocha-chai-sinon-cheatsheet" target="_blank">mocha-chai-sinon-cheatsheet</a> repository.
2. Execute `npm install` in the cloned folder
3. Display `index.html` in a browser

If you would like to extend the cheat sheet, feel free to submit a pull request.

### What is next?

Due to the length of this article, I did not deal with important topics in Mocha and Chai. In the second part, I will describe how to deal with asynchronous tests and I will also introduce fixtures. As a side effect, an example will exercise the `before`, `beforeEach` and `after` functions of test suites.

The third and fourth part will deal with SinonJs. First we learn how to spy, stub and mock objects, followed by a demonstration of how to use a fake timer and a fake server.



