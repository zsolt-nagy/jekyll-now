---
layout: post
title: Specifying JSON APIs
---

JSON (JavaScript Object Notation) is a very good alternative to XML in today's web applications as a data transmission standard. We can get away with a lot less characters when it comes to data transmission. JSON is an obvious choice when developing client side rich web applications driven by an API. Furthermore, all major languages came up with JSON parsers. It is therefore not a surprise that JSON APIs are more popular than ever. Many teams still struggle with the proper usage of the standards. I will highlight some pitfalls and solutions below that can be used for specifying excellent APIs.

There are significant differences between specifying a good API and specifying an API well. Advice on writing a good API is out of the scope of this article. Web API Design: Crafting Interfaces that Developers Love from Brian Mulloy is a good resource in this topic. We will rather focus on how to specify an API well.

### Interfaces

Different products are built by different teams. In case of some full stack solutions, both the client side and server side code are written by the same team. Testing, code review and quality assurance is also handled internally. Regardless of the size of the product and the size of the developer team, documentation is important for good maintainability. When different teams work on a system sharing only a subset of the required engineering knowledge, proper documentation of interfaces go beyond the "important for maintainability" condition: good specification is vital for smooth cooperation of teams.

I have collaborated in a software development project where more than 20 people contributed to the same product, excluding the Android and iOS clients and other backend products sharing databases with us. Whenever a new milestone is launched, the API and the client side solution are developed in parallel. Having invested in proper client side mocking solutions, the Javascript-based client only accesses the real API once both teams are ready. Both teams work based on the API specification. Whenever a team discovers that the specification has to be changed, all teams sync up and update the specification before proceeding with the implementation. 

> As a side note, it is worth mentioning that there are different interfaces between different teams. An extreme example is a HTML template between an application logic development team and a styling team. A significantly more straightforward example is a set of Entity-Relation diagrams easing communication between teams accessing the same databases.

### API Specification Using Formal Languages

Formal languages are specified using grammars describing how to generate "words" of a language. All possible words that can be generated based on the grammar should be handled by both the client side and the server side solution. In addition, neither party should generate words that the grammar does not describe. In this section, I will propose a specification method based on <a href="http://en.wikipedia.org/wiki/JSON-WSP" target="_blank">JSON-WSP</a>.

> If you have not heard of formal languages before, don't think of a word as a word in this article: a word is a sequence of characters that can be generated by the rules of the language. In case of generating APIs, a word is a correct payload transmitted between the client and the application server.

It is not easy to write a good API specification. In fact, some specifications are not specific enough, while others are far too verbose. In other cases, types are handled loosely, violating the grammar by sending `"1"` instead of `1` for example, or `""` instead of `null`. None of these options are acceptable, and checking communication in both ends has to be automated.

Specifying the endpoints is straightforward. All we need is to create a table containing the resource URLs and a short description of what the GET, POST, PUT and DELETE requests are used for. The hard part is the specification of the request, response and fault payloads.

Specifying the types are vital. As said before, it is vital to differentiate values 0 and "0". Therefore, primitive types should be introduced, such as `<boolean>`, `<string>`. Instead of using the type `<number>`, it is advised to specify `<integer>`. Complex types should be built on top of primitive types.

The simplest complex type is an enumeration. For instance, if we have two statuses, active and deactivated, we can build the following type:

```javascript
<status> ::= ( 0 | 1 )
```

The `|` operator and the parentheses are meta elements used for specifying the language: `|` specifies a selection between the elements inside the parentheses, meaning that either the value `0` or the value `1` should stand in place of the `<status>` sense. Note that this is an exclusive selection. 

In some cases, such as emails or addresses, regular expressions can limit values. The type `<rx-name>` should denote a type specified by a regular expression. For instance,

```javascript
<rx-positiveint> = [1-9][0-9]*
```

specifies a positive integer. This example also shows that it is possible to specify types other than strings using regular expressions. The only sane restriction is that only primitive types should be constructed as `rx-` types. Other language constructs are available for complex types.

Suppose that you would like to specify an array of objects, each storing a name-email pair. 

```json
<user> ::= 
{
	"name"    : <string>
	"address" : <string>
}

"users":
[
	( <user>, )*
]
```

Similarly to regular expressions, the `*` denotes any number of repetitions, including zero. Specifying a positive length is possible by using `+`. Note that the comma at the end of the last `<user>` element is specified by the schema, but in practice, it should be omitted. Describing this fairly obvious behavior would not add much to the descriptive nature of this specification though. Specification of the `<user>` type is straightforward to interpret. Semantically naming the types you use makes the specification slightly more readable. However, if you prefer including the user object in the `answers` key, feel free to do so:

```json
<user> ::= {
	"name"    : <string>
	"address" : <string>
}

"users":
[
	(
		{
		    "name"    : <string>
		    "address" : <string>
	    }, 
	)*
]
```

### Outlook

Beyond improving the API specification, JSON-WSP can do a lot more. Similarly to WSDL, JSON-WSP is a web service protocol. It can be used for automatically checking requests and responses in both ends. In fact, a parseable description can also be used to automatically generate business objects on client side in form of Backbone Models and Collections. 
