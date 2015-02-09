---
layout: post
title: Underscore Templating
---

Most software are developed by applying the principles of object oriented programming or functional programming. We will practice the latter approach by solving a problem related to presentation and filtering of data. We will use the UnderscoreJs library for the solution.

Programming in a functional way has some advantages over object oriented programming. Side effects and state of objects do not have to be managed, all data are treated as immutable. Therefore your code becomes easier to test, as side effect free functions are generally easier to verify.

Suppose that an array of objects are given. The objects contain data on departures from a given airport. The following fields are given:
- flight id
- destination
- time of departure
- expected time of departure (optional)
- Gate (Optional)
- Status (Optional)

Whenever optional fields are not provided, both the keys and the values are missing.

> We will omit the key of missing properties for the sake of this example. In case of static data like this, I would rather suggest providing the key and equating it to  `null` so that we have a way to indicate that the property indicated by the key belongs to the object and only the value is missing. Using an empty string is another option, although it would require an implicit assumption that an empty string is a missing value. Instead of using assumptions, we are normally better off using `null` which provides a value indicating an absence of value by default.
> Using the value `undefined`  for indicating missing values is wrong and should be strictly prohibited. 

The following tasks will be solved in this example:

Suppose an array of departure objects is given under the reference `departures`. 
1. Display departures in a tabular format. Use underscore templating. 
2. Extract the column names from the `departures` object. Use these column names in the template instead of hard coded text. The template should contain headers written in natural language (e.g. "Departure time" instead of departureTime).
3. Display an empty string if the value of a field is absent

```javascript
var departures = [
	{
		id: 'KL 1255',
		destination: 'Amsterdam',
		departureTime: '21:55',
		gate: 'A13',
	},
	{
		id: 'OK 001',
		destination: 'Prague',
		departureTime: '20:40',
		gate: 'A13',
		status: 'Check-in'
	},
	{
		id: '4U 2011',
		destination: 'Stuttgart', 
		departureTime: '20:35',
		gate: 'A11',
		status: 'Check-in'
	},
	{
	    id: 'LX 911',
	    destination: 'Zurich',
	    departureTime: '20:15',
	    expectedDepartureTime: '21:15',
	    status: 'check-in'
	},
	{
		id: 'OS 133',
		destination: 'Vienna',
		departureTime: '19:25',
		gate: 'A06',
		status: 'Departed'	
	}
];
```

Let's start with the table template. The result should look somewhat like `console.table( departures )`, without the `(index)` column.

For the sake of simplicity, we will place the template in a `script` tag and append it to the bottom of the body. Alternatives would be to use a CommonJs module loader such as RequireJs and a template loader plugin.

```html
<html>
  <body>
    <div id="result"></div>
    <script type="text/javascript" 
            src="http://underscorejs.org/underscore-min.js">
    </script>
	<script type="text/template" id="departures-template">
	<table>
	    <tr>
	        <th><%= data.headers.id %></th>
	        <th><%= data.headers.destination %></th>
	        <th><%= data.headers.departureTime %></th>
	        <th><%= data.headers.expectedDepartureTime %></th>
	        <th><%= data.headers.gate %></th>
	        <th><%= data.headers.status %></th>
	    </tr>
	    <% _.each( data.departures, function( departure ) { %>
	    <tr>
	        <td><%= departure.id %></td>
	        <td><%= departure.destination %></td>
	        <td><%= departure.departureTime %></td>
	        <td><%= departure.expectedDepartureTime %></td>
	        <td><%= departure.gate %></td>
	        <td><%= departure.status %></td>
	    </tr>
	    <% } ); /* end each */ %>
	</table>
	</script>
  </body>
</html>
```

Once the template is available, all we need to do is create a template function for the table and evaluate it with the `departure` array:

```javascript
// Get the template string from the DOM
var templateString = document.getElementById( 'departures-template' ).innerHTML;

// Prepare a partially evaluated template function
var tableTemplate = _.template( templateString );

// Prepare the data for presentation
var data = {
    departures: departures,
    headers: {
	    id: 'Id',
	    destination: 'Destination',
	    departureTime: 'DepartureTime',
	    expectedDepartureTime: 'Expected Departure Time',
	    gate: 'Gate',
	    status: 'Status'
    }
};

// Evaluate the template function with the prepared data
var departuresTable = tableTemplate( data );

// Display the result
document.getElementById( 'result' ).innerHTML = departuresTable;
```

The header text is currently hard coded. The second part of this example will make the table generic. Column names will be generated based on the key names. 

```javascript
var keys = _.chain( departures )
            .map( _.keys )
            .flatten()
            .uniq()
            .value();
```

We extract the keys of each object in the `departures` array with the `map` function. The result is then flattened, and finally, multiple occurrences of the same key are filtered. Chaining makes the functions more readable. Without chaining, the order of the functions would be the exact opposite as above:

```javascript
var keys = _.uniq( _.flatten( _.map( departures, _.keys ) ) );
```

We now need a function that translates the keys into text that we can display in the template:

```javascript
var stringValue = function( camelCase ) {
    var words = camelCase.replace(/([A-Z])/g, ' $1');
    return words.length ? 
           words[0].toUpperCase() + words.slice(1) :
           '';
}
```

Finally, the keys should be translated to (key, value) pairs and the result should be passed to the template.

```javascript
var keyToSchemaItem = function( key ) { 
    return { 
        key: key, 
        value: stringValue( key ) 
    };
};
var tableSchema = _.map( keys, keyToSchemaItem );
```

Let's prepare all data for presentation and change the template. The table columns will become dynamic based on the contents of `tableSchema`. Replace the value of `data` and the template:

```javascript
// Prepare the data for presentation
var data = {
    departures: departures,
    schema: tableSchema
};
``` 

```html
	<script type="text/template" id="departures-template">
	<table>
	    <tr>
		    <% _.each( data.schema, function( column ) { %>
		        <th><%= column.value %></th>
			<% } ); /* end each */ %>
	    </tr>
	    <% _.each( data.departures, function( departure ) { %>
	    <tr>
	        <% _.each( data.schema, function( column ) { %>
		        <td><%= departure[ column.key ] %></td>
		    <% } ); /* end each */ %>
	    </tr>
	    <% } ); /* end each */ %>
	</table>
	</script>
```

After evaluating the already prepared template function `tableTemplate` with `data` and inserting it into the DOM, the table appears. The order of the columns has changed, but the task did not specify any order anyway.

The third task is about displaying a `-` instead of non-existing values.  Ideally where should this task be done? 

> An option would be to pass the `-` value in the data object and replace all falsy values in the template with the formatted value. I would rather suggest preparing the data in advance and leaving the template as simple as possible. Therefore, the `-` values will be inserted into the `departures` object.

Let's create a placeholder object containing all the keys that occur in the table, having  `'-'`  values.

```javascript
var keyToPair = function(key) {
    return [ key, '-' ];
};
var placeholder = 
	_.chain(keys)
	 .map(keyToPair)   // key -> [key, '-']
	 .object()         // {..., key: '-', ... }
	 .value();
```

Then mix the placeholder object in each object of the array.

```javascript
var addEmptyValues = function(departure) {
    return _.extend( {},  placeholder, departure );
};
var presentedDepartures = _.map( departures, addEmptyValues );
```

>  **Why do we extend the empty object?**
>  `_.extend` works in a way that it *extends* its first argument with the remaining list of arguments. If a key is found in multiple objects, the rightmost occurrence will overwrite all others. The composed object is then returned by `_.extend`. However, on top of returning the result, all the keys-value pairs from the second attribute to the last attribute will be mixed into the first attribute as a side effect. As we should enjoy writing side effect free code, using `_.extend` with the empty object as the first argument is suggested.

After clarifying how the `addEmptyValues` function works, the mapping becomes straightforward. The`presentedDepartures` array now holds the data that we want to present. All you need to do is change `data.departures` to this value:

```javascript
// Prepare the data for presentation
var data = {
    departures: presentedDepartures,
    schema: tableSchema
};
```

We have reached the end of this task. It is very important to emphasize that you benefit from this example the most if you implement it yourself.  Forget about the "knowledge is power" mantra. Only applied knowledge is power.
