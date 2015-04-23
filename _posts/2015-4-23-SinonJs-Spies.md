---
layout: post
title: SinonJs Spies
---

This series introduces automated testing libraries for Javascript applications with Mocha, Chai and SinonJs. The <a href="http://zsolt-nagy.github.io/Writing-Automated-Tests-with-Mocha-and-Chai/" target="_blank">first part</a> covers the fundamentals of Mocha and Chai. The <a href="http://zsolt-nagy.github.io/Asynchronous-Tests-and-Fixtures-with-Mocha-and-ChaiJs/" target="_blank">second part</a> describes how Mocha and Chai can test DOM manipulations and handle asynchronous tests. 

Mocha and Chai have mostly been covered. If you apply what you learned in the first two parts of the Automated Testing series, you should be fine. We will now extend our test suites with a utility belt serving multiple purposes:

- Collect and verify runtime information on tested code (spies)
- Isolate tested code from code that surrounds it (stubs, mocks)
- Isolate the tested system from systems, storages and timing constraints surrounding it (fake server, fake timer)

SinonJs provides solutions helping you in achieving all three points. We will focus on the first point in the rest of this article: collecting and verifying runtime information on tested code.

### Spies

Spies monitor conditions under which a function is executed. While keeping existing state and behavior, spies provide more information such as:

- the number of times a function was called during a test
- the arguments a function was called with
- the return value of a function

I will show you how to use three types of spies: anonymous spies, function spies and method spies. 

**Anonymous spies**: The following example could be a possible test case for Backbone Events providing event driven communication in Backbone applications:

```javascript
describe( 'SinonJs', function() {
	
	describe( 'SinonJs Anonymous Spies', function() {

		it( 'should demonstrate the usage of anonymous spies with Backbone.Events', 
		    function() {
				var spy = sinon.spy();
				var eventBus = _.extend( {}, Backbone.Events );

				// Spy subscribes
				eventBus.on( 'custom:event', spy );

				// Spy is called when the event is triggered
				eventBus.trigger( 'custom:event', 2, 5 );

				expect( spy.calledWith( 2, 5 ) ).to.be.true;
				expect( spy.calledOnce ).to.be.true;
				expect( spy.callCount ).to.equal( 1 );		    
		} );
	} );
} );
```

The empty Sinon spy constructor creates an anonymous spy capable of acting instead of any pieces of functionality. I personally found them useful when I wanted to know if an event was triggered with correct arguments, without executing the callback method. The example above demonstrates this use case by using Backbone's publish-subscribe channels. When `custom:event` is triggered, the anonymous spy is executed checking if the callback was called with the proper arguments exactly once. 

**Function spies**: function spies wrap a function keeping the original implementation and collecting runtime information. 

```javascript
var isInteger = function( n ) {
	return typeof n === 'number' && n % 1 === 0;
}

var fibonacci = function ( n ) {
    switch( n ) {
        case 0  : return 0;
        case 1  : return 1;
        default : return fibonacci( n - 1 ) + fibonacci( n - 2 );
    }
}

describe( 'SinonJs Function Spies', function() {

	it( 'should demonstrate usage of function spies testing the isInteger function', 
	    function() {

			var isIntegerSpy = sinon.spy( isInteger );
			
			isIntegerSpy( 5 );
			isIntegerSpy( 5.25 );

			expect( isIntegerSpy.calledTwice ).to.be.true;
			expect( isIntegerSpy.getCall( 0 ).returnValue ).to.be.true;
			expect( isIntegerSpy.getCall( 1 ).returnValue ).to.be.false;  
	} );

	it( 'should demonstrate spying on recursive functions', function() {
		// TODO: implement the test case now, checking the following condition:
		// The function was called 9 times
	} );
} );
```

The first test needs no further explanation, even the syntax of the SPY API is straightforward to read. It is worth noting that the first example can fully be solved using Mocha and Chai syntax from the first part of this series. 

The second example is a bit more complex. If you have not implemented this test case, I encourage you to solve it now as you may learn a valuable lesson. In case you have implemented this test case and your function was only called once instead of 9 times, you will soon have an eureka effect.

The challenge of solving the recursive test is that the recipe of creating a spy under the reference `fibonacciSpy` and calling it does not work when trying to keep track of the call count. Think about it. Function spies keep the original behavior intact. The original behavior includes calling 'fibonacci' instead of `fibonacciSpy`. This means that the call count of `fibonacciSpy` will always be 1.

The solution is to replace the original `fibonacci` reference with its spied version inside the scope of the test. 

```javascript
it( 'should demonstrate spying on recursive functions', function() {
	var originalFibonacci = fibonacci;
	fibonacci = sinon.spy( fibonacci );
	fibonacci( 4 );

	expect( fibonacci.callCount ).to.equal( 9 );

	fibonacci = originalFibonacci;
} );
```


**Method spies**: instead of spying on a function, the observed element is a method of an object. Don't forget that spying on methods is a permanent change within an object. Therefore, the original behavior has to be *restored* before ending the test.

The following example demonstrates the usage of method spies useing Backbone models. The test case verifies that the parse method of the model is called after a successful fetch. This tests builds on the foundations of creating <a href="TODO" target="_blank">asynchronous tests</a>.

```javascript
describe( 'SinonJs Method Spies', function() {
    it( 'should call the parse method when fetching data', function( testIsDone ) {
        var m = new Backbone.Model();
        m.url = 'http://ip.jsontest.com';
        sinon.spy( m, 'parse' );
        m.fetch().done( function() {
             expect( m.parse.calledOnce ).to.be.true;
         } ).always( function() {
             m.parse.restore();
             testIsDone();
         } );
    } );
} );
```

**Spy API**: We have used just a fraction of the features Sinon spies provide. The following test suite introduces the rest. In order to make the test suite self documenting, I will omit the description "should demonstrate the usage of ... spy field or method". The description includes the tested spy field or method name, followed by its type. In case of functions, the type notation will be demonstrated in the form `arguments => returnValue`. The type `any` denotes an arbitrary Javascript type.

```javascript
describe( 'SinonJs spy API', function() {
	
	beforeEach( function() {
		this.isIntegerSpy = sinon.spy( isInteger );
	} );

	afterEach( function() {
		delete this.isIntegerSpy;
	} );

	it( 'callCount : integer', function() {
		this.isIntegerSpy( 1 );
		expect( this.isIntegerSpy.callCount ).to.equal( 1 );
	} );

	it( 'called :  boolean', function() {
		this.isIntegerSpy( 1 );
		expect( this.isIntegerSpy.called ).to.be.true;
	} );

	it( 'calledOnce : boolean', function() {
		this.isIntegerSpy( 1 );
		expect( this.isIntegerSpy.calledOnce ).to.be.true;
	} );

	it( 'calledTwice : boolean', function() {
		this.isIntegerSpy( 1 );
		this.isIntegerSpy( 2 );
		expect( this.isIntegerSpy.calledTwice ).to.be.true;
	} );

	it( 'calledThrice : boolean', function() {
		this.isIntegerSpy( 1 );
		this.isIntegerSpy( 2 );
		this.isIntegerSpy( 3 );
		expect( this.isIntegerSpy.calledThrice ).to.be.true;
	} );	

	it( 'reset', function() {
		this.isIntegerSpy( 1 );
		this.isIntegerSpy.reset();
		expect( this.isIntegerSpy.callCount ).to.equal( 0 );
	} );

	it( 'returned : arguments => boolean', function() {
		this.isIntegerSpy( 1 );
		this.isIntegerSpy( 1.25 );
		expect( this.isIntegerSpy.returned( true ) ).to.be.true;
		expect( this.isIntegerSpy.returned( false ) ).to.be.true;
		expect( this.isIntegerSpy.returned( null ) ).to.be.false;
	} );

	it( 'alwaysReturned : arguments => boolean', function() {
		this.isIntegerSpy( 1 );
		this.isIntegerSpy( 1.25 );
		expect( this.isIntegerSpy.alwaysReturned( true ) ).to.be.false;
		expect( this.isIntegerSpy.alwaysReturned( false ) ).to.be.false;
		expect( this.isIntegerSpy.alwaysReturned( null ) ).to.be.false;
	} );

	it( 'firstCall, secondCall, thirdCall, lastCall : spyCall', function() {
		this.isIntegerSpy( 1 );
		this.isIntegerSpy( 2 );
		this.isIntegerSpy( 3 );
		expect( this.isIntegerSpy.firstCall.returnValue ).to.be.true;
		expect( this.isIntegerSpy.secondCall.returnValue ).to.be.true;
		expect( this.isIntegerSpy.thirdCall.returnValue ).to.be.true;
		expect( this.isIntegerSpy.lastCall.returnValue ).to.be.true;
	} );

	it( 'getCall : spyCall', function() {
		this.isIntegerSpy( 1 );
		this.isIntegerSpy( 2 );
		this.isIntegerSpy( 3 );
		expect( this.isIntegerSpy.getCall( 0 ).returnValue ).to.be.true;
		expect( this.isIntegerSpy.getCall( 1 ).returnValue ).to.be.true;
		expect( this.isIntegerSpy.getCall( 2 ).returnValue ).to.be.true;
	} );

	it( 'calledBefore, calledAfter : spy => boolean', function() {
		var anonymousSpy = sinon.spy();

		this.isIntegerSpy( 1 );
		anonymousSpy();

		expect( this.isIntegerSpy.calledBefore( anonymousSpy ) ).to.be.true;
		expect( anonymousSpy.calledAfter( this.isIntegerSpy ) ).to.be.true;
	} );

	it( 'calledOn : object => boolean', function() {
		var context = {};
		this.isIntegerSpy( 1 );
		_.bind( this.isIntegerSpy, context )();

		expect( this.isIntegerSpy.calledOn( context ) ).to.be.true;
	} );

	it( 'alwaysCalledOn: object => boolean', function() {
		var context = {};
		this.isIntegerSpy( 1 );
		_.bind( this.isIntegerSpy, context )();

		expect( this.isIntegerSpy.alwaysCalledOn( context ) ).to.be.false;
	} );

	it( 'calledWith: arguments => boolean', function() {
		this.isIntegerSpy( 1, true, 'abc' );
		this.isIntegerSpy( 1, false );

		// partial matching from left to right
		expect( this.isIntegerSpy.calledWith( 1, true, 'abc' ) ).to.be.true;
		expect( this.isIntegerSpy.calledWith( 1, false ) ).to.be.true;
		expect( this.isIntegerSpy.calledWith( 1 ) ).to.be.true;
		expect( this.isIntegerSpy.calledWith( 1, true ) ).to.be.true;
		expect( this.isIntegerSpy.calledWith( 'abc' ) ).to.be.false;
		expect( this.isIntegerSpy.calledWith() ).to.be.true;
	} );

	it( 'alwaysCalledWith: arguments => boolean', function() {
		this.isIntegerSpy( 1, true, 'abc' );
		this.isIntegerSpy( 1, false );

		// partial matching from left to right has to match on all calls
		expect( this.isIntegerSpy.alwaysCalledWith( 1, true, 'abc' ) ).to.be.false;
		expect( this.isIntegerSpy.alwaysCalledWith( 1, false ) ).to.be.false;
		expect( this.isIntegerSpy.alwaysCalledWith( 1 ) ).to.be.true;
		expect( this.isIntegerSpy.alwaysCalledWith( 1, true ) ).to.be.false;
		expect( this.isIntegerSpy.alwaysCalledWith( 'abc' ) ).to.be.false;
		expect( this.isIntegerSpy.alwaysCalledWith() ).to.be.true;
	} );

	it( 'neverCalledWith: arguments => boolean', function() {
		this.isIntegerSpy( 1, true, 'abc' );
		this.isIntegerSpy( 1, false );

		// partial matching from left to right has to match on all calls
		expect( this.isIntegerSpy.neverCalledWith( 1, true, 'abc' ) ).to.be.false;
		expect( this.isIntegerSpy.neverCalledWith( 1, false ) ).to.be.false;
		expect( this.isIntegerSpy.neverCalledWith( 1 ) ).to.be.false;
		expect( this.isIntegerSpy.neverCalledWith( 1, true ) ).to.be.false;
		expect( this.isIntegerSpy.neverCalledWith( 'abc' ) ).to.be.true;
		expect( this.isIntegerSpy.neverCalledWith() ).to.be.false;
	} );

	it( 'calledWithExactly: arguments => boolean', function() {
		this.isIntegerSpy( 1, true, 'abc' );
		this.isIntegerSpy( 1, false );

		// full matching of the whole argument list and nothing else
		expect( this.isIntegerSpy.calledWithExactly( 1, true, 'abc' ) )
			.to.be.true;
		expect( this.isIntegerSpy.calledWithExactly( 1, false ) )
			.to.be.true;
		expect( this.isIntegerSpy.calledWithExactly( 1 ) ).to.be.false;
		expect( this.isIntegerSpy.calledWithExactly( 1, true ) )
			.to.be.false;
		expect( this.isIntegerSpy.calledWithExactly( 'abc' ) ).to.be.false;
		expect( this.isIntegerSpy.calledWithExactly() ).to.be.false;
	} );

	it( 'alwaysCalledWithExactly: arguments => boolean', function() {
		this.isIntegerSpy( 1, true, 'abc' );
		this.isIntegerSpy( 1, false );

		// full matching of the whole argument list on all calls
		expect( this.isIntegerSpy.alwaysCalledWithExactly( 1, true, 'abc' ) )
			.to.be.false;
		expect( this.isIntegerSpy.alwaysCalledWithExactly( 1, false ) )
			.to.be.false;
		expect( this.isIntegerSpy.alwaysCalledWithExactly( 1 ) ).to.be.false;
		expect( this.isIntegerSpy.alwaysCalledWithExactly( 1, true ) )
			.to.be.false;
		expect( this.isIntegerSpy.alwaysCalledWithExactly( 'abc' ) )
			.to.be.false;
		expect( this.isIntegerSpy.alwaysCalledWithExactly() ).to.be.false;
	} );

	it( 'threw : (() | string | object) => boolean', function() {
		var injectError = function() {
			invalidReference;
		}
		var errorSpy = sinon.spy( injectError );

		try {
			errorSpy();
		} catch( e ) {
			this.e = e;
		}

		expect( errorSpy.threw() ).to.be.true;
		expect( errorSpy.threw( 'ReferenceError' ) ).to.be.true;
		expect( errorSpy.threw( 'Error' ) ).to.be.false;
		expect( errorSpy.threw( this.e ) ).to.be.true;
	} );

	it.skip( 'calledWithNew - may result in false positives' );

	it( 'alwaysThrew : (() | string | object) => boolean', function() {
		var injectError = function( shouldThrow ) {
			if( shouldThrow ) invalidReference;
		}
		var errorSpy = sinon.spy( injectError );

		try {
			// swapping the two calls negates the 1st, 2nd and 4th assertions.
			errorSpy( false );
			errorSpy( true );
		} catch( e ) {
			this.e = e;
		}

		expect( errorSpy.alwaysThrew() ).to.be.false;
		expect( errorSpy.alwaysThrew( 'ReferenceError' ) ).to.be.false;
		expect( errorSpy.alwaysThrew( 'Error' ) ).to.be.false;
		expect( errorSpy.alwaysThrew( this.e ) ).to.be.false;
	} );

	it( 'thisValues : array', function() {
		var context = {};
		this.isIntegerSpy( 1 );
		_.bind( this.isIntegerSpy, context )();

		expect( this.isIntegerSpy.thisValues[1] ).to.equal( context );
	} );

	it( 'args : array', function() {
		this.isIntegerSpy( 1, true, 'abc' );
		expect( this.isIntegerSpy.args[0] ).to.deep.equal( [ 1, true, 'abc' ] );
	} );

	it( 'exceptions : array', function() {
		var injectError = function( shouldThrow ) {
			if( shouldThrow ) invalidReference;
		}
		var errorSpy = sinon.spy( injectError );

		try {
			errorSpy( false );
			errorSpy( true );
		} catch( e ) {
			this.e = e;
		}

		expect( errorSpy.exceptions[0] ).to.be.undefined;
		expect( errorSpy.exceptions[1] ).to.equal( this.e );
	} );

	it.skip( 'printf (string, array) => string', function() {
		// This function just formats a string injecting spy results.
		// Don't use it inside assertions!
	} );
} );

```

The `getCall` method and the `firstCall`, `secondCall`, `thirdCall` and `lastCall` properties return a spy call having the following interface:

```javascript
describe( 'SinonJs Spy Call API', function() {
	
	beforeEach( function() {
		this.isIntegerSpy = sinon.spy( isInteger );
		this.isIntegerSpy( 5, true, 'abc' );
		this.spyCall = this.isIntegerSpy.firstCall;
	} );

	afterEach( function() {
		delete this.firstCall;
		delete this.isIntegerSpy;
	} );

	it( 'calledOn : object => boolean', function() {
		expect( this.spyCall.calledOn( this ) ).to.be.true;
	} );

	it( 'calledWith : arguments => boolean', function() {
		expect( this.spyCall.calledWith() ).to.be.true;
		expect( this.spyCall.calledWith( 5 ) ).to.be.true;
		expect( this.spyCall.calledWith( 5, true ) ).to.be.true;
		expect( this.spyCall.calledWith( 5, true, 'abc' ) ).to.be.true;
	} );

	it( 'calledWithExactly : arguments => boolean', function() {
		expect( this.spyCall.calledWithExactly() ).to.be.false;
		expect( this.spyCall.calledWithExactly( 5 ) ).to.be.false;
		expect( this.spyCall.calledWithExactly( 5, true ) ).to.be.false;
		expect( this.spyCall.calledWithExactly( 5, true, 'abc' ) ).to.be.true;
	} );

	it( 'notCalledWith : arguments => boolean', function() {
		expect( this.spyCall.notCalledWith() ).to.be.false;
		expect( this.spyCall.notCalledWith( 5 ) ).to.be.false;
		expect( this.spyCall.notCalledWith( 5, true ) ).to.be.false;
		expect( this.spyCall.notCalledWith( 5, true, 'abc' ) ).to.be.false;
	} );

	it( 'threw : (() | string | object) => boolean', function() {
		var injectError = function() {
			invalidReference;
		}
		var errorSpy = sinon.spy( injectError );

		try {
			errorSpy();
		} catch( e ) {
			this.e = e;
		}

		var firstCall = errorSpy.getCall( 0 );

		expect( firstCall.threw() ).to.be.true;
		expect( firstCall.threw( 'ReferenceError' ) ).to.be.true;
		expect( firstCall.threw( 'Error' ) ).to.be.false;
		expect( firstCall.threw( this.e ) ).to.be.true;		
	} );

	it( 'thisValue : object', function() {
		expect( this.spyCall.thisValue ).to.equal( this );
	} );

	it( 'args : array', function() {
		expect( this.spyCall.args ).to.deep.equal( [ 5, true, 'abc' ] );
	} );

	it( 'exception : object', function() {
		expect( this.spyCall.exception ).to.be.undefined;	
	} );

	it( 'returnValue : any', function() {
		expect( this.spyCall.returnValue ).to.be.true;
	} );
} );
```


### Matchers

I left out a couple of functions from the cheat sheet as they require the introduction of matchers. SinonJs matchers provides you with tools for describing your expectations in a compact way. These expectations are then used in assertions to verify whether the arguments or the return value match the expectation described in the matcher.

Using matchers is not mandatory. Some matchers are easier to express in other ways, while others rather act as syntactic sugar. One use case I highly recommend is checking if tested functions are always called with the same argument types and they also return the same argument type in each call.

```javascript
describe( 'SinonJs Spy and SpyCall Matchers', function() {
	beforeEach( function() {
		var getArgs = function() { return arguments; }
		this.getArgsSpy = sinon.spy( getArgs );
	} );	

	afterEach( function() {
		delete this.getArgsSpy;
	} );

	it( 'calledWithMatch : (matcher1, ..., matcherN) => boolean', function() {
		this.getArgsSpy( 5 );
		this.getArgsSpy( 'abc' );
		
		expect( this.getArgsSpy.calledWithMatch( sinon.match.number ) )
			.to.be.true;
		expect( this.getArgsSpy.calledWithMatch( sinon.match.string ) )
			.to.be.true;
		expect( this.getArgsSpy.calledWithMatch( sinon.match.bool ) )
			.to.be.false;
	} );

	it( 'alwaysCalledWithMatch : (matcher1, ..., matcherN) => boolean', 
		function() {
		
		this.getArgsSpy( 5 );
		this.getArgsSpy( 'abc' );
		
		expect( this.getArgsSpy.alwaysCalledWithMatch( sinon.match.number ) )
			.to.be.false;
		expect( this.getArgsSpy.alwaysCalledWithMatch( sinon.match.string ) )
			.to.be.false;
		expect( this.getArgsSpy.alwaysCalledWithMatch( sinon.match.bool ) )
			.to.be.false;
		expect( this.getArgsSpy.alwaysCalledWithMatch( sinon.match.any ) )
			.to.be.true;			
	} );

	it( 'neverCalledWithMatch : (matcher1, ..., matcherN) => boolean', 
		function() {
		
		this.getArgsSpy( 5 );
		this.getArgsSpy( 'abc' );
		
		expect( this.getArgsSpy.neverCalledWithMatch( sinon.match.number ) )
			.to.be.false;
		expect( this.getArgsSpy.neverCalledWithMatch( sinon.match.string ) )
			.to.be.false;
		expect( this.getArgsSpy.neverCalledWithMatch( sinon.match.bool ) )
			.to.be.true;		
	} );

	it( 'returned : matcher => boolean', function() {
		this.getArgsSpy( 5 ); 
		expect( this.getArgsSpy.returned( sinon.match.bool ) ).to.be.false;	
	} );

	it( 'spyCall.calledWithMatch: matcher => boolean', function() {
		this.getArgsSpy( 5 );
		this.getArgsSpy( true );
		var callSpy = this.getArgsSpy.firstCall;

		expect( callSpy.calledWithMatch( sinon.match.number ) ).to.be.true;
		expect( callSpy.calledWithMatch( sinon.match.bool ) ).to.be.false;
	} );

	it( 'spyCall.notCalledWithMatch: matcher => boolean', function() {
		this.getArgsSpy( 5 );
		this.getArgsSpy( true );
		var callSpy = this.getArgsSpy.firstCall;

		expect( callSpy.notCalledWithMatch( sinon.match.number ) ).to.be.false;
		expect( callSpy.notCalledWithMatch( sinon.match.bool ) ).to.be.true;
	} );

	it( 'Type matchers', function() {
		this.getArgsSpy( 5 );
		var callSpy = this.getArgsSpy.firstCall;

		expect( callSpy.calledWithMatch( sinon.match.any ) ).to.be.true;
		expect( callSpy.calledWithMatch( sinon.match.defined ) ).to.be.true;
		expect( callSpy.calledWithMatch( sinon.match.truthy) ).to.be.true;
		expect( callSpy.calledWithMatch( sinon.match.falsy ) ).to.be.false;
		expect( callSpy.calledWithMatch( sinon.match.bool ) ).to.be.false;
		expect( callSpy.calledWithMatch( sinon.match.number ) ).to.be.true;
		expect( callSpy.calledWithMatch( sinon.match.string ) ).to.be.false;
		expect( callSpy.calledWithMatch( sinon.match.object ) ).to.be.false;
		expect( callSpy.calledWithMatch( sinon.match.func ) ).to.be.false;
		expect( callSpy.calledWithMatch( sinon.match.array ) ).to.be.false;
		expect( callSpy.calledWithMatch( sinon.match.regexp ) ).to.be.false;
		expect( callSpy.calledWithMatch( sinon.match.date ) ).to.be.false;

		expect( callSpy.calledWithMatch( sinon.match.typeOf( 'number' ) ) )
			.to.be.true;
	} );

	it( 'Value == equality matchers', function() {
		var data = { value: 5 };
		this.getArgsSpy( 5 );
		this.getArgsSpy( data );
		var firstCallSpy = this.getArgsSpy.firstCall;
		var lastCallSpy = this.getArgsSpy.lastCall;

		expect( firstCallSpy.calledWithMatch( sinon.match( 5 ) ) ).to.be.true;
		expect( lastCallSpy.calledWithMatch( sinon.match( data ) ) ).to.be.true;
	} );

	it( 'Value and type === equality matchers', function() {
		this.getArgsSpy( 5 );
		var firstCallSpy = this.getArgsSpy.firstCall;

		expect( firstCallSpy.calledWithMatch( sinon.match.same( 5 ) ) )
			.to.be.true;
		expect( firstCallSpy.calledWithMatch( sinon.match.same( '5' ) ) )
			.to.be.false;
	} );

	it( 'instanceOf matcher', function() {
		this.getArgsSpy( new Error() );
		var firstCallSpy = this.getArgsSpy.firstCall;

		expect( firstCallSpy.calledWithMatch( 
			sinon.match.instanceOf( Object ) ) ).to.be.true;
	} );

	it( 'has property, has own property matchers', function() {
		this.getArgsSpy( new Error() );
		var firstCallSpy = this.getArgsSpy.firstCall;

		expect( firstCallSpy.calledWithMatch( sinon.match.has( 'stack' ) ) )
			.to.be.true;
		expect( firstCallSpy.calledWithMatch( sinon.match.has( 'toString' ) ) )
			.to.be.true;
		expect( firstCallSpy.calledWithMatch( 
			sinon.match.hasOwn( 'stack' ) ) ).to.be.true;
		expect( firstCallSpy.calledWithMatch( 
			sinon.match.hasOwn( 'toString' ) ) ).to.be.false;
	} );

	it( 'matcher and/or matcher', function() {
		this.getArgsSpy( 5 );
		this.getArgsSpy( '5' );
		var numberOrString = sinon.match.number.or( sinon.match.string );
		var numberAndFive = sinon.match.number.and( sinon.match( 5 ) );

		expect( this.getArgsSpy.alwaysCalledWithMatch( numberOrString ) )
			.to.be.true;
		expect( this.getArgsSpy.alwaysCalledWithMatch( numberAndFive ) )
			.to.be.false;
	} );

	it( 'custom matcher', function() {
		this.getArgsSpy( 2015 );
		var yearMatcher = sinon.match( function( year ) {
			return typeof year === 'number' &&
				   year >= 1970 &&
				   year <= 2015;
		} );

		expect( this.getArgsSpy.alwaysCalledWithMatch( yearMatcher ) )
			.to.be.true;
	} );
} );
```

All the above tests have been added to the <a href="https://github.com/zsolt-nagy/mocha-chai-sinon-cheatsheet" target="_blank">Mocha-Chai-SinonJs reference</a> on GitHub. Feel free to clone it or fork it. 

After writing all 72 tests, I discovered that I started utilizing the expressive power of Mocha, Chai and SinonJs better than before. This exercise was very much like a short term memory training exercise. I suggest building on these foundations if you write automated tests on a regular basis. Alternatively, whenever you get stuck, you can look up these tests as a reference.

### Code Isolation

Spying is just scratching the surface of what Sinon can provide you with. Creating stubs and mocks will allow you to write more reliable tests by allowing you to focus on code that matters and substitute code that does not matter in a test. In other words, tested code will work in a vacuum, surrounded by simplified dependencies that always act correctly under special, tested circumstances.  

