Caplin StyleGuide for JS
========================

A javascript style guide based on drawing heavily from other style guides (in particular [Felix's Node Style Guide](http://nodeguide.com/style.html), but favouring explicitness and minimisation of potential for error over terseness.


Religion
--------
Tabs not spaces. (Which one to go with doesn't really matter, but a decision has to be made).


Semicolons
----------
Semicolons everywhere they should be.  Failing to put in semicolons can cause bugs.

```
// DO NOT DO THIS - missing semicolon makes it all go bad.

var myFunc = function() {
  // myFunc code codes here.
}

(function() {
  // this code should be executed immediately.
})();
```
This piece of code uses an idiom for immediately executed functions.  However it fails badly because of the lack of a semicolon after myFunc (and note that when code is concatenated, you don't always have control over what is above you).  myFunc will be executed immediately, passing the immediately executed function as a disregarded parameter to myFunc, then the result is considered to be a function and attempted to be invoked.  TL;DR: always use semicolons.


Braces
------
Opening braces go on the same line as the statement:

*Right:*
```
if (true) {
  console.log('winning');
}
```

*Wrong:*
```
if (true)
{
  console.log('losing');
}
```


Naming
------

lowerCamelCase for variables and properties.  UpperCamelCase for classes. ALL_CAPS_WITH_UNDERSCORES for constants.  Long names are better than wrong or confusing names, but bear in mind that names much over four words become hard to read in camel case.

Avoid uncommon abbreviations or single character names.  Single character loop counters are acceptable, but always consider if you can be more descriptive.

Avoid meaningless or generic names like 'obj', 'temp', 'data'.  Type names are better than generic names but are still bad and should be avoided; 'func', 'string', 'str', 'num'.

Names should be in a single language for the entire codebase.  For us, this is English (although USAian or UKian spelling variants are acceptable outside of the public API).

We do not use hungarian notation, however collections should be named with plural forms and if something is a boolean, it should be obvious from the name (e.g. isSelected).  Clearly indicating variables containing DOM elements or regexes can help keep code clean too.

Anything that could be referenced from code in another file but that should not be (e.g. private member variables) must indicate this by having their name prepended with an underscore.

```
funtion MyClass() {
  this.publicMember = 23;
  this._privateMember = 44;
}
```

If your method has a name beginning with 'get', it should not usually modify any state.  An exception is for values that are lazily initialised.


Object / Array
--------------
Don't use the Array constructor to create an array containing particular values. Use the literal form for that.

```
var x = new Array("this", "is", "wrong");
var y = ["this", "is", "right"];
```

Short declarations can be on a single line, otherwise start a new line for each item.  When split over multiple lines, commas go at the end of the line not the beginning.

Quote keys in Object literals only when required.


Conditionals
------------
It's easy to assume that the behaviour of ```if (x)``` is the same as ```if (x == true)```.  This is NOT TRUE.  There are numerous differences.  In fact, the behaviour is the same as ```if (Boolean(x)) ```, a relationship often referred to as 'truthiness'.

The truthiness rules are not known to all developers, and developers relying on them when they were not 100% sure of what types a particular variable can take has caused bugs in our codebase.  Worse, when you look at code that depends on truthiness, it is not possible to know if the developer who wrote that code knew exactly what they were doing or not.

It is better to be explicit in what you check.

```
if (bob != null) {
  // This is explicit.
  console.log('this code is executed if bob is not null or undefined');
}
```
```
if (bob) {
  // This relies on truthiness.
  console.log('this code is executed if bob is not null, undefined, 0, false, NaN or the empty string').
}
```

Write what you mean.

If you are completely sure that a variable is a boolean (either it has not be passed into your scope or you have checked it yourself), it is acceptable to leave out the ```=== true```, as to put it in would be barbarous.

Complex conditionals should be assigned to a descriptive variable.

A common source of errors is missing an else in a complex set of conditionals.  What's worse is that it's difficult to tell from looking at the code afterwards whether the author intended the behaviour or not.  When writing nested if statements, inner if's or run-on ifs should always have else clauses, with a comment if they are empty.


Coercion
--------
Coercion is bad.  Avoid it. 

* Prefer ```===``` over ```==```
* Prefer ```!==``` over ```!=```

The only exception is if you wish to check that something is or is not null or undefined, you can check with == null or != null.

* Prefer ```String(number)``` to ```number + ""```.
* Prefer ```Number(string)``` to ``` + string ```.
* Prefer ```Boolean(string)``` to ``` !! string ```.

This does not create a new object, and it makes your code clearer.

It is acceptable to use coercion when appending to a string (e.g. for logging).


Numbers
-------

Use the Math methods for rounding.  There are short forms (eg. real|0), but they are less descriptive.


Returns
-------

Guarded returns at the top of a function are fine.  Other than that, balance the following two considerations:

# Having lots of returns in a function makes it hard to read.
# Having deeply nested conditional statements makes it hard to read.


Closures
-------
Avoid nested closures, they quickly become extremely hard to follow.


Constructors
------------
Remember to call your superconstructor in your own constructor. e.g.

```
function MyClass(arg1, arg2) {
  SuperClass.call(this, arg1);
  /* myclass construction code */
}
```
A constructor should leave the class in a consistent state (i.e. it should establish the Class Invariants).  If it cannot then it must throw an Error.


Extending Other Objects
-----------------------
General library code must not modify the prototype of other objects.

It is acceptable for application code or shim libraries to do this, but it's a step that should be taken with great trepidation.


Public API Methods
------------------
Methods that may be called by others should start by checking their preconditions (e.g. types of arguments, etc.) and throwing appropriate Errors if the preconditions are not met.

Use JSDoc comments for all methods that are part of the public API.

Your JSDoc comments should make a point of describing what happens if any of the arguments are null/undefined when the function is called, whether the function can ever return null/undefined, and under which circumstances errors will be thrown.


Errors
------
Throw only objects that are instances of the generic ```Error``` type.

Custom Error objects are fine, but must inherit from ```Error```, and set the name property.  Where appropriate reuse the standard Error types, e.g. TypeError, RangeError.

