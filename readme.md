---
layout: main
permalink: /index.html
title: Caplin Style Guide for Javascript
---



Caplin Style Guide for JavaScript
=================================

A javascript style guide drawing heavily from other style guides (in particular [Felix's Node Style Guide](http://nodeguide.com/style.html), and somewhat inspired by [idiomatic.js](https://github.com/rwldrn/idiomatic.js/)) but favouring explicitness and minimisation of potential for error over terseness.

As with any style guide, some of these rules are debateable. Having a consistent style throughout the codebase is more valuable than getting any specific rule 'right'.

> Break any of these rules sooner than say anything outright barbarous.
>
> *Politics and the English Language*, George Orwell

This guide is written in markdown.  The readable version is [here](http://caplin.github.io/StyleGuide/), and the source is [here](https://github.com/caplin/StyleGuide).


Religion
--------
Tabs not spaces. So it is written, so let it be done.

Configuring your editor to show whitespace characters will help you be consistent.  I have their opacity set to 30/255 so that they don't interfere visually.


EcmaScript target
-----------------
As long as we support IE8, caplin code is limited to the shimmable subset of ecmascript 5.

*OK:*

```javascript
var boundFunction = func.bind(this);
var x = Object.create(prototypeForX);
var arr = [1, 2, 3].map(function(x) {return x * 2;});

```

*Wrong (not shimmable):*
```javascript
var x = {};
Object.defineProperty(x, 'fred', {
  get: function() {
    console.log('fred has been got');
    return 23;
  }
});

```

General library code must not modify the prototype of other objects.  It is acceptable for application code or shim libraries to do this, but it is a step that should be taken with great trepidation.

Using strict mode is encouraged.  You do this by putting `"use strict";` at the top of your scope.


Semicolons
----------
Semicolons after every statement except control flow ending with curly brackets (if/while/etc).  Failing to put in semicolons can cause bugs:

```javascript
// DO NOT DO THIS - missing semicolon makes it all go bad.

var myFunc = function() {
  // myFunc definition code codes here.
}

(function() {
  // this code should be executed immediately.
})();
```

This piece of code uses an idiom for immediately executed functions.  However it fails badly because of the lack of a semicolon after myFunc (and note that when code is concatenated, you don't always have control over what is above you).  myFunc will be executed immediately, passing the following function as a disregarded parameter, then the result is considered to be a function and attempted to be invoked.


Braces
------
Opening braces go on the same line as the statement:

*Right:*

```javascript
if (true) {
  console.log('winning');
}
```

*Wrong:*

```javascript
if (true)
{
  console.log('losing');
}
```

Code blocks must always be surrounded in braces, even if they are only a single line.

*Wrong:*

```javascript
if (true) console.log('losing');
```


Object / Array
--------------
Don't use the Array constructor to create an array containing particular values. Use the literal form for that.

```javascript
var x = new Array("this", "is", "wrong");
var y = ["this", "is", "right"];
```
This is because the Array constructor treats its arguments inconsistently.  A single numeric argument defines the size of the array, while any other kind of argument list provides initial values.  Avoid confusion by always using the literal notation to provide initial array values.

Short declarations can be on a single line, otherwise start a new line for each item.  When split over multiple lines, commas go at the end of the line not the beginning.

*Right:*

```javascript
var myArray = [
  'first',
  'second',
  'third'
];
```

*Wrong:*

```javascript
var myArray = [
  'first'
  , 'second'
  , 'third'
];
```

Don't leave trailing commas.

Only quote keys in Object literals when required.


Conditionals
------------
It's easy to assume that the behaviour of `if (x)` is the same as `if (x == true)`.  This is **NOT TRUE**.  There are numerous differences.  In fact, the behaviour is the same as `if (Boolean(x))`, a relationship often referred to as 'truthiness'.

The truthiness rules are not well known by all developers, and relying on them without being sure of exactly which types may be in the relevant variables leads to bugs.  Worse, the errors can be difficult to track down and when you look at code that depends on truthiness, it is not possible to know if the developer who wrote that code knew exactly what they were doing or not.

It is better to be explicit in what you check.

*OK:*

```javascript
if (bob != null) {
  // This is explicit (enough).
  console.log('this code is executed if bob is not null or undefined');
}
```

*Wrong:*

```javascript
if (bob) {
  // This relies on truthiness.
  console.log('Executed if bob is not null, undefined, 0, false, NaN or the empty string').
}
```

Write what you mean.

If you are completely sure that a variable is a boolean (either it has not been passed into your scope or you have checked it yourself), it is acceptable to leave out the `=== true`, as to put it in would be barbarous.

Complex conditionals should be assigned to a descriptive variable.

A common source of errors occurs when a developer writes a chain of if/else if/else if's and leaves out an else, or they write a bunch of nested ifs, and one of the inner ifs misses an else.  What's worse is that it's difficult to tell from looking at the code afterwards whether the author intended the behaviour or not.  When writing nested if statements, inner if's or run-on ifs should always have else clauses with a comment if they are empty.


Coercion
--------
Coercion is bad.  Avoid it. 

* Prefer `===` to `==`
* Prefer `!==` to `!=`

The only exception is if you wish to check that something is or is not null or undefined, you can check with `== null` or `!= null`.

* Prefer `String(number)` to `number + ""`.
* Prefer `Number(string)` to ` + string `.
* Prefer `Boolean(string)` to ` !! string `.

This does not create a new object, and it makes your code clearer.

It is acceptable to use coercion when appending to a string (e.g. for logging).

Usually use the Math methods for rounding numbers to integers.  There are short forms (eg. `real|0`), but they are less descriptive.


Functions
---------

Guarded returns at the top of a function are fine.  Other than that, balance the following two considerations:

* Having lots of returns in a function makes it hard to follow.
* Having deeply nested conditional statements in a function makes it hard to read.  Anything that shifts your code to the right should be meaningful for that code.

Avoid nested closures, they quickly become extremely hard to follow.


Naming
------

`lowerCamelCase` for variables and properties.  `UpperCamelCase` for classes. `ALL_CAPS_WITH_UNDERSCORES` for constants.  Long names are better than wrong or confusing names, but bear in mind that names much over four words become hard to read in camel case.

Avoid uncommon abbreviations or single character names.  Single character loop counters are acceptable, but always consider if you can be more descriptive.

Avoid meaningless or generic names like 'obj', 'temp', 'data'.  Type names are better than generic names but are still bad and should be avoided; 'func', 'string', 'str', 'num'.

Names should be in a single language for the entire codebase.  For us, this is English (although USAian or UKian spelling variants are acceptable outside of the public API).

We do not mandate Hungarian Notation, however collections should be named with plural forms and if something is a boolean, it should be obvious from the name (e.g. isSelected).  Clearly indicating variables containing DOM elements or regexes can help keep code clean too.

Anything that could be referenced from code in another file but that should not be (e.g. private member variables) must indicate this by having their name prepended with an underscore.

```javascript
funtion MyClass() {
  this.publicMember = 23;
  this._privateMember = 44;
}
```

If your method has a name beginning with 'get', it should not usually modify any state.  An exception is for values that are lazily initialised.


Constructors
------------
Remember to call your superconstructor in your own constructor. e.g.

```javascript
function MyClass(arg1, arg2) {
  SuperClass.call(this, arg1);
  /* myclass construction code */
}
```
A constructor should leave the class in a consistent state (i.e. it should establish the class invariants).  If it cannot then it must throw an Error.


Public API
----------
Use JSDoc comments for anything part of the public API (i.e. may be used by code outside our own codebase).

Your JSDoc comments should make a point of describing what happens if any of the arguments are null/undefined when the function is called, whether the function can ever return null/undefined, and under which circumstances errors will be thrown.

Methods that are part of the public API should start by checking their preconditions (e.g. types of arguments, etc.) and throwing appropriate Errors if the preconditions are not met.


Errors
------
Throw only objects that are instances of the generic `Error` type.

Custom Error objects are fine, but must inherit from `Error`, and set the name property.  Where appropriate reuse the standard Error types, e.g. TypeError, RangeError.

Error messages should include all the kinds of things you might need to debug the issue.  For example, if something is the wrong type, your error message should say what type it is as well as what type it should be.


Revision Control
----------------

Do not check in commented out code.  Remembering old versions of code is what the revision control system is for.  If there is code commented out and checked in then nobody knows why it's there.

Write good checkin comments.  Mention the issue title and ID, who worked on the issue and what you did.  If someone is confused by a line of code you wrote, they will look at your check in comment, make it help them.


General Hygeine
---------------

Keep your lines, methods, classes and files short (aim for <100 characters, <10 lines, <200 lines).

Do not use globals.  Avoid using singletons.

Actively look for ways to fail faster.

Use jsHint with the [provided settings](https://raw.github.com/caplin/StyleGuide/gh-pages/.jshintrc).

Write your code to be testable and test it! Writing code that receives objects it needs rather than creating them can make the code easier to test.  If you find yourself writing 'new', double check that your code is as testable as it should be.

Prefer feature detection over browser detection.  Wheverever possible, keep code that depends on the browser (e.g. uses the DOM or host objects not common to other js environments) separate from pure javascript.
