---
layout: page
permalink: /develop/js-style-guide/
title: JavaScript Style Guide
author: Liviu Ionescu

date: 2017-10-09 02:47:00 +0300

---

After many years dealing with style guides for various languages, the conclusion is that the style itself is less important than applying it consistently. 

So, for the xPack JavaScript source files to be consistent, the first requirement is to pass the [Standard JS](https://standardjs.com) validation tools.

After this, the main recommendations are:

- use the **ECMAScript 6** specifications (ES 6),
- if the module does something reasonably complex, the module public functions **must be asyncronous**, 
- asynchronous functions must **use promises** (and definitely **avoid callbacks**), 
- **reentrancy** should be seriously considered (avoid module-global variables).

<div style="clear: both;"></div>

## From _Understanding ECMAScript 6_

The main specifications to be followed are those of ES 6; they override all other older specifications and style guides.

### Block bindings

Use `const` by default, and only use `let` when you know a variable’s value needs to change

### Functions

Use default parameters.

```javascript
const makeRequest = function (url, timeout = 2000, callback = function() {}) {
  // the rest of the function
}

const add = function (first, second = getValue(first)) {
  return first + second
}
```

Use _rest_ parameters.

```javascript
const pick = function (object, ...keys) {
  let result = Object.create(null)
  for (let i = 0, len = keys.length; i < len; i++) {
    result[keys[i]] = object[keys[i]]
  }
  return result
}
```

The `Function` constructor.

```javascript
var add = new Function("first", 'second = first',
                 'return first + second')
console.log(add(1, 1))     // 2
console.log(add(1))        // 2
```

JavaScript has two different internal-only methods for functions: `[[Call]]` and `[[Construct]]`. When a function is called without new, the `[[Call]]` method is executed, which executes the body of the function as it appears in the code. When a function is called with new, that’s when the `[[Construct]]` method
is called. The `[[Construct]]` method is responsible for creating a new object, called the instance, and then executing the function body with this set to the instance. Functions that have a `[[Construct]]` method are called constructors.

```javascript
const Person = function (name) {
  if (this instanceof Person) {
    this.name = name
  } else {
    throw new Error('You must use new with Person.')
  }
}
var person = new Person('Nicholas')
var notAPerson = Person.call(person, 'Michael')    // works!
```

Block-level functions

```javascript
'use strict'

if (true) {
  console.log(typeof doSomething) // throws an error
  let doSomething = function () {
    // empty
  }
  doSomething()
}
console.log(typeof doSomething)
```

Arrow functions are functions defined with a new syntax that uses an arrow (`=>`)

- **No this, super, arguments, and new.target bindings** The values of `this`, `super`, arguments, and new.target inside the function are defined by the closest containing non-arrow function
- **Cannot be called with new** Arrow functions do not have a [[Construct]] method and therefore cannot be used as constructors. Arrow functions throw an error when used with new.
- **No prototype** Because you can’t use new on an arrow function, there’s no need for a prototype. The prototype property of an arrow function doesn’t exist.
- **Can’t change this** The value of this inside the function can’t be changed. It remains the same throughout the entire life cycle of the function.
- **No arguments object** Because arrow functions have no arguments binding, you must rely on named and rest parameters to access function arguments.
- **No duplicate named parameters** Arrow functions cannot have duplicate named parameters in strict or non-strict mode, as opposed to non-arrow functions, which cannot have duplicate named parameters only in strict mode.

```javascript
let sum = (num1, num2) => num1 + num2
// effectively equivalent to:
let sum = function(num1, num2) {
  return num1 + num2
};
```

### Enhanced Object Functionality

It is now possible to modify an object’s prototype after it has been created thanks to ECMAScript 6’s `Object.setPrototypeOf()` method.

In addition, you can use the `super` keyword to call methods on an object’s prototype. The `this` binding inside a method invoked using `super` is set up to automatically work with the current value of `this`.

## From [_npm's "funny" coding style_](https://docs.npmjs.com/misc/coding-style)

As the name implies, some of them are _funny_, but some are still useful.

### Line length

Keep lines shorter than **80** characters

### Indentation

Indentation is **two spaces**

### Curly braces

Curly braces belong on the same line

```javascript
const f = function () {
  while (foo) {
    bar()
  }
}
```

### Semicolons

Don't use semicolons, except when required; for example to prevent the expression from being interpreted as a function call or property access, respectively.

```javascript
;(x || y).doSomething()
;[a, b, c].forEach(doSomething)
```

### Comma first

Put the **comma at the start** of the next line, directly below the token that starts the list

```javascript
const magicWords = [ 'abracadabra'
                 , 'gesundheit'
                 , 'ventrilo'
                 ]
  , spells = { 'fireball' : function () { setOnFire() }
             , 'water' : function () { putOut() }
             }
  , a = 1
  , b = 'abc'
  , etc
  , somethingElse
```

### Quotes

Use single quotes for strings except to avoid escaping

```javascript
const ok = 'String contains "double" quotes'
const alsoOk = "String contains 'single' quotes or apostrophe"
const paramOk = `Back quotes string with ${parameter}`
```

### Whitespaces

Put a single space in front of `(` for anything other than a function call

### Functions

Use named functions.

- always create a new `Error` object with your message (`new Error('msg')`)
- logging is done using the [`npmlog`](https://github.com/npm/npmlog) utility

### Case, naming, etc

- use **lowerCamelCase** for multiword identifiers when they refer to objects, functions, methods, properties, or anything not specified in this section
- use **UpperCamelCase** for class names (things that you'd pass to "new")
- use **all-lower-hyphen-css-case** for multiword filenames and config keys
- use named functions, they make stack traces easier to follow
- use **CAPS\_SNAKE\_CASE** for constants, things that should never change and are rarely used

### null, undefined, false

- boolean variables and functions should always be either `true` or `false`
- when something is intentionally missing or removed, set it to `null`
- don't set things to `undefined`
- boolean objects are verboten

### Exceptions

For the native Node.js callback usage, never to ever ever throw anything. It's worse than useless. Just send the error message back as the first argument to the callback.

But for the modern ES 6 promise usage, exceptions are fine.

## From [Node.js modules](https://nodejs.org/dist/latest-v6.x/docs/api/modules.html)

### Modules

Modules are a way of preventing multiple JavaScript units to polute the global namespace.

Objects defined at root level in a module are not global, but belong to the module; the usual name for this is _module-global_. 

### Caching

Modules are cached after the first time they are loaded. This means (among other things) that every call to require('foo') will get exactly the same object returned, if it would resolve to the same file. Multiple calls to `require('foo')` will not cause the module code to be executed multiple times.

From this point of view, modules behave like [singletons](https://en.wikipedia.org/wiki/Singleton_pattern); they can also be compared to static classes in C++.

Leaving status at the module level can be either a blessing or a curse, depending on the environment used to run the module. In server environments, using module-global variables is like using static variables in a multi-threaded environment, if not handled correctly it may have unexpected results.

### Exports 

To make some functions and objects visible outside the module, you can add them as properties to the special `modules.exports` object:

```javascript
const PI = Math.PI
module.exports.area = (r) => PI * r * r
module.exports.circumference = (r) => 2 * PI * r
```

Although you can rewrite the `module.export` to be a sinle function (such as a constructor), still prefer to add them as properties to the object and refer to them explicitly in the `require()` line:

```javascript
const square = require('./square.js').square
const mySquare = square(2)
console.log(`The area of my square is ${mySquare.area()}`)

module.exports.area = (width) => {
  return {
    area: () => width * width
  }
}
```

If you want to export a complete object in one assignment instead of building it one property at a time, assign it to `module.exports`.

### Accessing the main module

When a file is run directly from Node.js, `require.main` is set to its module. That means that you can determine whether a file has been run directly by testing

```javascript
require.main === module
```

### The module wrapper

Before a module's code is executed, Node.js will wrap it with a function wrapper that looks like the following:

```javascript
module = ... // an object for the current module
module.export = {} // an empty object
exports = module.export // a reference to the exports; avoid using it
__filename = '/x/y/z/abc.js'
__dirname = '/x/y/z'
(function (exports, require, module, __filename, __dirname) {
  // Your module code actually lives in here
});
```

In each module, the `module` variable is a reference to the object representing the current module. `module` isn't actually a global but rather local to each module.

The `module.exports` object is created by the Module system. Sometimes this is not acceptable; many want their module to be an object of their own. To do this, assign the desired export object to `module.exports`. 

For convenience, `module.exports` is also accessible via the `exports` module-global. Note that assigning a value to `exports` will simply rebind the local exports variable, which is probably not what you want to do; if the relationship between `exports` and `module.exports` seems like magic to you, ignore `exports` and only use `module.exports`.

Note that assignment to `module.exports` must be done immediately. It cannot be done in any callbacks.

### Global objects

These are really objects, available in all modules. (see Node.js [Globals](https://nodejs.org/api/globals.html))

- `global`
- `process`
- `console`
  - `console.log('msg')` - writes 'msg' to stdout
  - `console.warn('msg')` - writes 'msg' to stderr
  - `console.error('msg')` - writes 'Error: msg' to stderr
  - `console.assert(value, 'msg')`

## The xPack project preferences

### Prefer ES6 solutions

This is Rule no. 1, that overrides all other rules. Definitely **avoid using old style code**.

### Use classes as much as possible

Even if the new syntax is mostly syntactic sugar, and internally things behave as strangly as they did in the first JavaScript versions, still **use the new class syntax** at large; it is much cleaner and improves readability.

### Use promises instead of callbacks

Really. No callbacks at all. Use promises.

### Use async/await for anynchronous calls

Once `async`/`await` became standard, and the V8 engine added support for them, there is no reason for not using `async`/`await`.

**Wrap old legacy code using callbacks into promises** and execute them with `await`.

### Use static class members for sharing

Modules are singletons; using module variables is like using static variables in a multi-threaded environment; they may provide a way of sharing common data between multiple instances of objects created inside the same module, but if not handled correctly this may have unexpected results. 

The general recommendation is to **make the modules reentrant**. In practical terms, **do not use module-global variables** at all; make the module **export a class, and create instances of it** whenever needed; for sharing data between instances, **use static class members**.

### Do not restrict export to a single function or class

Bad style:

```javascript
module.exports = function () {
	return /[\u001b\u009b][[()#;?]*(?:[0-9]{1,4}(?:;[0-9]{0,4})*)?[0-9A-ORZcf-nqry=><]/g;
};
...
const func = require('name')
```

Apart from being unnamed, returning a single function prevents future extensions, for example exporting a second function from the same module would mandate all modules that use the first function via `require()` to be updated to `require().func1`, which may cause many headaches to developers.

```javascript
module.exports.func1 = function () {
	return /[\u001b\u009b][[()#;?]*(?:[0-9]{1,4}(?:;[0-9]{0,4})*)?[0-9A-ORZcf-nqry=><]/g;
};
module.exports.func2 = function () { ... }
...
const func = require('name').func1
```

The recommendation is to always return functions or preferably classes as properties of the `module.exports` object, and get them individually by name.

### Prefer static classes to group methods

Prepare your module to export multiple functions; group them (by functionality) either below a parent object, or, even better, in classes with static members.

The main advantage of this scheme is that adding new exports will only change the interface incrementaly, minimising the risk to break backward compatibility.

## Make node exports/imports look like ES6 exports/imports

Assuming classes are prefered, the EC6 syntax for export/import would look like:

```javascript
export class WscriptAvoider { ... }
...
import { WscriptAvoider } from 'wscript-avoider.js'
```

So, to stay close to this syntax, the recommendation is to preserve the original `module.exports` object, and add properties to it, preferably classes, even if they have only static members.

To import them, the syntax uses the explicit classs name:

```javascript
const WscriptAvoider = require('wscript-avoider').WscriptAvoider
WscriptAvoider.quitIfWscript(appName)
```

Cases like `import { WscriptAvoider as Xyz } from 'wscript-avoider.js'` would be naturaly represented as:

```javascript
const Xyz = require('wscript-avoider').WscriptAvoider
Xyz.quitIfWscript(appName)
```

In case the class is not static, instantiate it as usual.

## Links to other style guides

Prefered:

- [JavaScript "Standard" Style](https://standardjs.com)

Other links:

- [10 Best JavaScript Style Guides](http://noeticforce.com/best-javascript-style-guide-for-maintainable-code)
- [airbnb JavaScript Style Guide](https://github.com/airbnb/javascript)
- [Idiomatic.js JavaScript Style Guide](https://github.com/rwaldron/idiomatic.js/)
- [Google ES6 Style Guide](https://google.github.io/styleguide/jsguide.html)
- [Crockford Code Conventions for the JavaScript Programming Language](http://javascript.crockford.com/code.html)
- [felixge/node-style-guide](https://github.com/felixge/node-style-guide)
- [RisingStack/node-style-guide](https://github.com/RisingStack/node-style-guide)

## Linting Tools

Prefered (used automatically by Standard):

- [ESLint](http://eslint.org/), but indirectly via the 'Standard JS' tools.

Other links:

- [A Comparison of JavaScript Linting Tools](https://www.sitepoint.com/comparison-javascript-linting-tools/)
- [JSHint](http://www.jshint.com/)
