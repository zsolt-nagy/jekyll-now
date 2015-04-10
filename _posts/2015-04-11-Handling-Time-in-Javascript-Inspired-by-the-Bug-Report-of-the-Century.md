---
layout: post
title: Handling Time in Javascript - Inspired by the Bug Report of the Century
---

First of all, check out this <a href="https://github.com/angular/angular.js/issues/5017" target="_blank">bug report</a>. This is not the first bug report on the Date object I have ever seen, but it motivated me to share a couple of related observations and experiences with you. More importantly, I will also describe some solutions that let you get away with using dates in any Javascript.

### What went wrong in Australia in 1970?

A different timezone offset was applied right before and right after new year's eve. Proof:

<img src="http://zsolt-nagy.github.io/images/posts/dates_in_australia.png" alt="Timezone offsets around New Years eve in 1970 in Australia" />

After invoking the timestamp constructor of Date objects and console logging the timezone offset in minutes, a daylight saving inflection point got created. It is a bug we have to live with. Not a crucial one as I have never seen an application that would require you to use a date object before 1970.

### Date Object: The Bad Parts

If you know why `new Date("2")` is February 1st, 2001, then feel free to continue reading this post <a href="#timezone">here</a>.

**The constructor**: Keeping the different ways in mind on how to create a date is quite an adventure in itself. If you want to read the full summary, there are countless references about it. The below examples are executed in a browser console from Germany, Europe. I will use the most logical, sortable and human readable date format in the comments: "yyyy.mm.dd". 


```javascript
new Date();   
Thu Apr 09 2015 22:22:28 GMT+0200 (W. Europe Daylight Time)
// now

new Date(2000,0,1,0,0,0);  
Sat Jan 01 2000 00:00:00 GMT+0100 (W. Europe Standard Time)
// 2000.01.01 midnight local = 1999.12.31 23:00:00 GMT

new Date(2000); 
Thu Jan 01 1970 01:00:02 GMT+0100 (W. Europe Standard Time)
// 2000 milliseconds after 1970.01.01 midnight GMT
// In Germany, it's 1 hour later

new Date("2000");
Sat Jan 01 2000 01:00:00 GMT+0100 (W. Europe Standard Time)
// same as new Date(2000,0,1,0,0,0) = new Date(2000,0,1) = new Date("2000-01-01")

new Date(2000,0,0); 
Fri Dec 31 1999 00:00:00 GMT+0100 (W. Europe Standard Time)
// 2000.01.(-01) = 1999-12-31, midnight local
// Equals 1999-12-30 23:00:00 GMT

new Date(2000,12,31)
Wed Jan 31 2001 00:00:00 GMT+0100 (W. Europe Standard Time)
// 2000-13-31 = 2001-01-31 midnight local
```
 
Straightforward, right? Some rules:

 - Years can be arbitrary. Days start with 1. All other fields are zero based. Most people notice that months are odd because they start with zero. In practice, days pose as the real exception.
 - If you just supply one argument, depending on its type and value, it's treated as a millisecond-based timestamp, the year value (4 digit string) or the month value (1 or 2 digit string)
 - Months, days and time coordinates overflow in both directions with the side effect of offsetting other coordinates accordingly. For instance, the 0th day of January 2015 is December 31st 2014. The 32nd day of January is the 1st day of February.

If you are not familiar with the details, it does not matter. Just stick to proper type conversions and know one way to initialize the date object. 

<a name="timezone"></a>
**Implied timezone**: The date object forces the timezone of your system on you.

If you think you can get around it by calling methods with `UTC` in them, think again. You will be able to get and set <a href="http://en.wikipedia.org/wiki/Coordinated_Universal_Time" target="_blank">UTC</a> values without problems. Can your use third party widgets such as charting libraries or date/timepickers? Unlikely. This is what makes the Date object hard to use in some cases.

### Simple mistakes

**Handling Unix Timestamps**: Unix timestamps are in seconds. Javascript timestamps are in milliseconds. If you have ever created a date referring to a gloomy Saturday evening on January 17th, 1970, you might already know that you have to multiply your Unix timestamps by 1000.

**Trying to represent UTC/GMT manually**: only do it if your whole system is built based on the `UTC` getters and setters. Otherwise, your UTC date will be imterpreted as local date.

**Trying to represent another timezone**: in short, don't do it! You don't have access to all sufficient information in Javascript. Timezone offsets depend on daylight saving. Daylight saving depends on governmental decisions. I have seen multiple libraries, where the main developer tried to introduce timezones, and got stuck with it as soon as he realized that he needs to take care of daylight saving.

**Applying the wrong timezone offset**: in some cases, you have to supply a fake time to third party libraries by offsetting the Date object belonging to the timestamp with the timezone offset. What's wrong with the following line of code?

```javascript
var timestamp = 954547200000;
var offset = new Date().getTimezoneOffset() * 60 * 1000;
d = new Date( timestamp + offset )
```

The problem is that the above code will work for half a year each year. This code works exactly when the current timezone offset is the same as the timezone offset of the local date represented by the timestamp. I have seen this mistake multiple times. It's quite a big achievment to locate this bug in legacy code for the first time, but believe me, it becomes disturbing when you see how many times it re-occurred in multiple repositories.

### Client-Server communication

> A Unix timestamp represents the exact same time instant all over the world.

If you freeze the time and enter `new Date().getTime()` in your console, you will see the exact same timestamp anywhere in the world. The client will be able to interpret and display that timestamp both in the local and GMT timezones.

If you need to display timezones, one option is to use the <a href="http://www.w3.org/TR/NOTE-datetime" target="_blank">ISO 8601</a> standard. This is the serialized form of Javascript Date objects as well:

```javascript
var d = new Date(2011,11,11);
// Result: Sun Dec 11 2011 00:00:00 GMT+0100 (W. Europe Standard Time)

d.getTimezoneOffset();
// Result: -60

json = JSON.stringify({ date: d });
// Result: "{"date":"2011-12-10T23:00:00.000Z"}"

iso8601String = JSON.parse( json );
// Result: Object {date: "2011-12-10T23:00:00.000Z"}
```

In the JSON date string value, `Z` means UTC. It can be replaced by a timezone designator of form `+hh:mm` or `-hh:mm`. `JSON.stringify` is not a lossless representation of your Date object. As soon as you serialize your date, the outside world will not know your own timezone, as the string represents a UTC date. In addition, parsing an ISO 8601 string results in a string, not a date. While there are workarounds for understanding this format automatically while parsing, in practice, all you need to do is supply the parsed value as the only argument of a new Date object. 

If you also have to modify the dates on client side without asking the server, you need a timezone database on client side. There is simply no other way in knowing when the daylight saving start and end dates occur. You can reinvent the wheel by implementing your own solution, or you can use excellent and stable client side libraries for handling time properly. According to my knowledge, the best library for this purpose is <a href="http://momentjs.com/docs/" target="_blank">MomentJs</a>. One more piece of advice: update your timezone database at least once per year. 

Although MomentJs understands ISO 8601 perfectly, you may still want to consider unix timestamps and a timezone string for an exact representation between the client and the server. See <a href="http://momentjs.com/timezone/" target="_blank">http://momentjs.com/timezone/</a> for more examples.

### Third party libraries

GitHub contains a lot of jQuery widgets. Private repositories contain even more. There is a new charting library in the news almost every week. Many libraries use dates, some of them force your own timezone on you. Suppose that you would like to display data in GMT. This is a reasonable request in case you have offices all around the world and you would like to avoid communication traps. 

All widgets that work with the Date object force you  to fake the date to display the correct UTC coordinates. For instance, the timestamp 954547200000 belongs to April 1st, 2000 midnight. If you pass this timestamp to the Date constructor, you get March 31st in case you are left of the GMT zone. Unless you take care of these anomalies by adding the timezone offset to the timestamp, people in the United States will unhappily conclude that their dates are shifted by one day. 

In addition, some charting libraries like an old version of jqPlot may duplicate the hour, day or month when daylight saving starts, and skip a month when daylight saving ends in case you apply the timezone offset in the wrong way. The exact opposite happens on the other half of the globe. 

### What do I need to remember?

- If local timezone is enough for you, feel free to stick to the Date object
- In case of local timezones and GMT, use the UTC methods of the Date object and apply timezone offsets with special care. Alternatively, use MomentJs
- In case of arbitrary timezone support, use MomentJs
- Client-server communication works either with timestamps with or without the specified timezone, or you can also use ISO 8601 if the exact location does not matter
- Fake the time for third party libraries if you want to use them in GMT




