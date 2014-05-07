# Native Promise Only (NPO)

A polyfill for native ES6 Promises as close as possible (no extensions) to the strict spec definitions.

## Intent

The aim of this project is to be the smallest (and maybe the fastest!?) polyfill for Promises, staying as close as possible to what's specified in both [Promises/A+](http://promisesaplus.com) and the upcoming ES6 specification (**link needed**).

An equally important goal is to avoid exposing any capability for promise-state to be mutated externally. The next section explains the trade-offs of that balance.

## Known Limitations

On the bright side, a promise object from this polyfill **will be** an instance of the `Promise` constructor, which makes identification of genuine promises easier:

```js
var p = new Promise(..);

p instanceof Promise; // true
```

However, these promise instances don't inherit (delegate to) a *meaningful* `Promise.prototype` object for their methods. That is:

```js
var p = new Promise(..);

Object.getOwnPropertyNames( p ); // [ then, catch ]
Object.getOwnPropertyNames( Promise.prototype ); // [ constructor ]
```

As such, these promises are not really "sub-classable" in the ES6 `class` / `extends` sense, though theoretically you should be able to do that in ES6 with the built-in Promises.

The reason for this deviation is that there's a choice between having delegated methods on the `.prototype` or having private state. Since **the spirit of promises was always to ensure trustability** -- that promises were immutable (from the outside) to everyone except the initial resolver/deferred -- private state is a critically important feature to preserve.

Many other ES6 promise shims/libs seem to have forgotten that important point, as many of them either expose the state publicly on the object instance or provide public accessor methods which can externally mutate a promise's state. Both of these deviations are **intolerable** in my opinion, so this library chose the opposite trade-off: *no ES6 sub-classing*.

Any trade-off is a shame, but this one is the least of a few evils, and probably won't prove to limit very many, as there are only a limited number of use-cases for `extend`ing `Promise` in the ES6 sub-class sense.

## ES5 Assumption

This polyfill assumes various ES5 capabilities, such as `Function#bind` and `Array#forEach`. If you need to use this polyfill in older JS environments, make sure to provide polyfills/shims [such as these](https://github.com/es-shims/es5-shim).

*Native Promise Only* needs (all of which have compliant ES5 shims):

* `Array.isArray`
* `Array#forEach`
* `Array#some`
* `Function#bind`

## Usage

To use this polyfill in the browser, include the "npo.js" file with your site's scripts. It's a polyfill, which means it will not overwrite `Promise` if it exists as a global already, so it's safe to include unconditionally.

To use with AMD, import the "npo.js" file module.

To use the polyfill in node, run:

```
npm install native-promise-only
```

Then require the module into your node code like this:

```
var Promise = require("native-promise-only");
```

Other than the above "Known Limitations" discussion and some browser bugs (such as [these](https://gist.github.com/getify/bd11ccf1eff2efdac0fb)) which **this polyfill doesn't have**, your promises should operate the same in all JS environments.

Exactly like native promises, here's a quick example of how you create and use the polyfilled promises:

```js
var p = new Promise(function(resolve,reject){
	setTimeout(function(){
		resolve("Yay!");
	},100);
});

p.then(function(msg){
	console.log(msg); // Yay!
});
```

For more on how to use and enjoy native (or polyfilled!) promises, check out [JavaScript Promises](http://www.html5rocks.com/en/tutorials/es6/promises/) by [Jake Archibald](http://twitter.com/jaffathecake).

## Still Want More?

This project intentionally adheres pretty strictly to the narrow core of [Promises/A+](http://promisesaplus.com) as adopted/implemented by ES6 into the native `Promise()` mechanism.

But it's quite likely that you will experience a variety of scenarios in which using *only* native promises might be tedious, limiting, or more trouble than it's worth. There's good reason why most other **Promises/A+** "compliant" libs are actually superset extensions on the narrow core: **because async flow-control is often quite complex in the real world.**

*Native Promise Only* will not add any of these extra flourishes. Sorry.

**However, I have another project**: [asynquence](http://github.com/getify/asynquence) (async + sequence). It's an abstraction on top of the promises concept (promises are hidden inside), designed to drastically improve the readability and expressiveness of your async flow-control code.

You simply express your async flow-control and *asynquence* creates and chains all the promises for you underneath. **Super simple.**

*asynquence* has a custom implementation for the hidden "promises" it uses, and as such does not need native `Promises`, nor does it need/include this polyfill.

Get your feet wet with native promises first, but then when you go looking for something more, consider [asynquence](http://github.com/getify/asynquence) (which is vastly more powerful and is still only 2k!).

## Tests/Compliance

<a href="http://promisesaplus.com/" float="right">
    <img src="http://promisesaplus.com/assets/logo-small.png" alt="Promises/A+ logo"
         title="Promises/A+ 1.1 compliant" align="right" />
</a>

*Native Promise Only* is "spec compliant" in the sense of passing all tests in the [Promises/A+ Test Suite](https://github.com/promises-aplus/promises-tests).

To run the test suite after cloning this repo, run `npm install` to install the dev-dependencies, then run `npm test`.

However, there are definitely other tests that need to be added, for example testing the `Promise()` constructor's behavior, as well as the `Promise.*` static helpers (`resolve(..)`, `reject(..)`, `all(..)`, and `race(..)`), none of which are covered by Promises/A+ test suite.

Developing a more comprehensive test-suite to augment the  Promises/A+ test suite **is now another primary goal** of this project.

## License

The code and all the documentation are released under the MIT license.

http://getify.mit-license.org/
