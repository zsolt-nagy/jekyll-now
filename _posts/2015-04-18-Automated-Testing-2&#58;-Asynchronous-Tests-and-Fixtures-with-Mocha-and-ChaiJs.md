---
layout: post
Automated Testing 2: Asynchronous Tests and Fixtures with Mocha and ChaiJs
---

This is the second part on automated testing with Mocha, ChaiJs and SinonJs. If you have never used Mocha or ChaiJs before, I suggest reading <a href="http://zsolt-nagy.github.io/Writing-Automated-Tests-with-Mocha-and-Chai/">the first part</a>. We will continue where we left off last time. The syntax of Mocha and Chai should mostly be clear. There are some frequently occurring scenarios though that require additional knowledge. These are:

- Fixtures for testing DOM state and manipulations,
- Asynchronous tests.


### Fixtures

In most cases, we write Javascript code that manipulates the DOM. Therefore, we have to find a way to create DOM nodes that our tests can access. In the example below, we will create a div for the purpose of holding these DOM nodes for the tests. These nodes are called <a href="http://en.wikipedia.org/wiki/Test_fixture" target="_blank">test fixtures</a>, and they are responsible for storing a fixed (DOM) state of the tested code.

Let's insert a hidden div in `index.html` below `#mocha`: `#fixtures` will stores the fixed states used by the tests.

```javascript
    <body>
        <h1>Automated tests</h1>
        <div id="mocha"></div>
        <div id="fixtures" style="display: none; visibility: hidden;"></div>
    </body>
```

Fixtures have to be set up before running tests and they also have to be cleaned up afterwards. Make sure each test can run in isolation.


```javascript
// Code under test
var seasonsView = {
    template : '<ul><li class="js-season">Spring</li>' +
                   '<li class="js-season">Summer</li>' +
                   '<li class="js-season">Fall</li>' + 
                   '<li class="js-season">Winter</li></ul>',
    el       : '#seasons',
    render   : function() {
        $( '#seasons' ).empty().prepend( this.template );
    }  
};

describe("Test with fixture", function()
{
    before( function()
    {
        this.$fixture = $('<div id="view-fixture"></div>');
    } );

    beforeEach( function() {
       this.$fixture.empty().appendTo( $( '#fixtures' ) );
       this.$fixture.prepend( '<div id="seasons"></div>' );
       seasonsView.render();
    } );

    afterEach( function() {
        $( '#fixtures' ).empty();
    } );

    it( 'should display Spring, Summer, Autumn and Winter', function() {
        var mapFunction = function( item ) { 
            return item.innerHTML;
        }

        var $seasons = this.$fixture.find( '.js-season' ); 
        var seasonNames = _.map( $seasons, mapFunction); console.log(seasonNames);
        expect( seasonNames ).to.contain( 'Spring' );
        expect( seasonNames ).to.contain( 'Summer' );
        expect( seasonNames ).to.contain( 'Autumn' );
        expect( seasonNames ).to.contain( 'Winter' );
    });

    it( 'should display four seasons.', function() {
        var seasonNodes = this.$fixture.find( '.js-season' );
        expect( seasonNodes ).to.have.length( 4 );
    });
} );

```

Quiz question of the day: why does the above described test fail? If you noticed the small error before, congratulations! You will likely notice a lot of accidental errors in blog posts and in books. These skills will make you spot more errors in your code as well. If you have not noticed it yet, this is your chance to check the code: you have all knowledge needed to find the error on your own. I will reveal the small bug at the end of the chapter. Notice that an eagle eye is required to write or review reliable code.

Now that you have found the bug, let's concentrate on the contents:

- The tested code is a simple static view. It looks somewhat similar to a plain Backbone View instance, and it's good enough for the sake of illustrating how fixtures work. 
- Before the execution of the first test, the `#view-fixture` node is added to the DOM.
- Before each and every test, the contents of `#view-fixture` are emptied and the empty div is appended to the main `#fixtures` node. In addition, our `seasonsView` object is rendered inside the added node
- After each and every test, `#fixtures` is emptied, removing the non-empty `#view-fixture` from the DOM
- Each test is executed between a `beforeEach` and an `afterEach` call. The tests have full access to the contents of the fixture in the DOM
- The fixture node and its contents don't show up in the test page because `#fixtures` is styled as invisible. Nothing will bother you while the tests are executed

Oh, and the injected bug... back when I moved to Malta, I inquired about the seasons and mentioned fall. One of my colleagues replied: "we don't have fall, we have autumn". For this reason, you will only find Autumn in the <a href="https://github.com/zsolt-nagy/mocha-chai-sinon-cheatsheet" target="_blank">cheat sheet</a>, making the first test pass.


<a name="asynchronous"></a>
### Asynchronous Tests

In some cases, we may reach the last line of a test before an asynchronous callback is executed. Even if an assertion in the callback fails, the test itself may still pass. Therefore, we need to tell the Mocha test case when it should terminate. 

Let's fetch some data from a server! Suppose that you would like to fetch a Backbone Model for the purpose of checking its attributes. Note that we are not writing a unit test here as we ask a real server to send us some data. Mocha and Chai may have use cases beyond unit testing. 

We will use the service http://time.jsontest.com/ and our test will check the existence of the following keys: `time`, `milliseconds_since_epoch` and `date`. Our first attempt will look like this:

```javascript
// Tested code
var TimeModel = Backbone.Model.extend({
    url: 'http://time.jsontest.com'
});

describe( 'Asynchronous Tests', function() {
    before( function() {
        this.timeModel = new TimeModel();
    });

    it( 'should find the correct keys after fetching the model', 
        function( testIsDone ) {
            var callback = function() { 
                var expectedKeys = 
                    [ 'date', 'milliseconds_since_epoch', 'time' ];  
                expect( this.timeModel.attributes ).to
                    .have.keys( expectedKeys );    

                // Tell Mocha when the test should finish
                testIsDone();        
            }
            this.timeModel.fetch().done( 
                _.bind( callback, this ) 
            );
    });
});
```

We fetch `timeModel` in the very last line of the test. If we hadn't called the `testIsDone` function in `callback`, Mocha would have terminated the test before the reaching the assertion. The test would have passed for the wrong reason. This would have been the most dangerous testing mistake, making the test completely useless. 

As a side note, most people call the `testIsDone` function `done`. Under normal circumstances, I would have "done" the same too. Just in this specific example, I already had a different `done` function belonging to the return xhr of `this.timeModel.fetch()` and I didn't feel like overloading the name in the explanation. 

The `before`, `beforeEach`, `after` and `afterEach` functions can also be asynchronous. Just define the callback function as an argument and call it whenever you think setup or cleanup is done. If you fail to define a setup or tear down function asynchronous and it contains a callback, one of the following two anomalies may occur:

- a test executes before or during the execution of the callback in the setup method. In other words, the test is executed before its setup is completed;
- a callback in an after or afterEach method is executed during the setup method belonging to another test or the other test itself. Prepare for anything unexpected.

It is worth mentioning that there are other, more common use cases for asynchronous tests: animations. Suppose that a user action causes an "fade in" animation, completed in 1 second. Alternatively, suppose that there is a network connection error on your site while fetching data from the server, resulting your server call to time out. The displayed erroneous state should still be tested.

### Mocha and Chai are not enough

While we won't have much trouble with fixtures, you might already suspect that asynchronous tests are not that straightforward to handle. If you wanted to start testing your application solely based on this advice, depending on your application, you may end up with tests executing for minutes:

- Testing animations take time
- Fetching data from a real server takes time
- Testing a server timeout takes even more time
- Setting up and cleaning up the dependencies of a tested object may or may not take much time. They still shift your focus away from the object you test. In addition, faults in other objects can cause failures in your tests, breaking the principles of unit testing

Unit tests have to be executed quickly and have to focus on client side exclusively. Mocha and Chai can't solve these problems on their own. Therefore, stay tuned for the upcoming parts of the Automating Testing series, where I will introduce SinonJs, allowing you to do the following:

- freeze the time and move it forward in an instant as much as possible
- create a fake server to handle server requests and respond them with any payload in any order at any time
- define spies, stubs and mocks to substitute dependencies of your tested object with a simplified version. In addition, you will also get some runtime information on tested objects

We will continue with spies in a week.
