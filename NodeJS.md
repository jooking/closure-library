## Closure Library on Node.js in 60 seconds ##

As of July 2013, Closure Library comes with a script loader that makes goog.provide/require work in Node.js. To use it, just use Node.js's require() to load the bootstrap script.
```
$ cd path/to/closure-library
$ nodejs
> require('./closure/goog/bootstrap/nodejs')
{}
> goog.require
[Function]
> goog.require('goog.string.linkify')
undefined
> goog.string.linkify.linkifyPlainText('Hello http://www.world.com')
'Hello <a rel="nofollow" target="_blank" href="http://www.world.com">http://www.world.com</a>'
```

## Details ##

### Can I use Closure Library to share code between the browser and my NodeJS server? ###

Yes. The bootstrap script's primary goal is to support browser/server code-sharing. If you are accustomed to the Closure Library workflow for managing your dependencies, then that same workflow should "just work" on Node.js.

### How does it work? ###

When you load the Closure Library's Node.js bootstrapper, it replaces goog.require() with an implementation that uses Node.js's require() to load scripts.

The main Closure namespace ('goog') is registered in the global scope, so it will automatically be available to all files in your Node.js program.

### Does Closure-Library-on-Node.js behave differently than Closure Library in the browser? ###

The big difference is that goog.require() is now synchronous, rather than asynchronous. We consider this a happy benefit. :)

Obviously, any libraries in Closure Library that use the DOM will not work on NodeJS.

### Does Closure-Library-on-Node.js behave differently than other Node.js programs? ###

Yes, scoping and local modules work very differently.

In most Node.js modules, the default scope is "file scope". So if you have a file with the code
```
var x = 1;
```
then `x` will only be visible in the current file.

The Closure Library bootstrap script makes `goog` a global variable that is visible to all files. `goog.global` points to the global scope, rather than to the file scope. If you goog.require a symbol, that symbol will be available to all files, not just the one you're in.

As a corollary, this means that unlike most Node.js modules, you cannot have two conflicting versions of Closure Library in the same Node.js program when you use the main bootstrap script.

### Can I goog.require my own files on Node.js? ###

First, you need to [create a deps file](https://developers.google.com/closure/library/docs/depswriter) for your code.

Then, use Node.js require() to load the deps file. This will tell Closure Library where to find your code.

Lastly, you need to ensure your code is properly registered in the global scope. This can be a bit tricky, depending on how disciplined you are about declaring global variables. If you always declare global variables with goog.provide, then this should work fine out-of-the-box.

If you sometimes declare global variables with 'var', and you want those variables available to all scripts (like they are in the browser), then you may need to refactor your code to use goog.provide and/or use goog.nodeGlobalRequire.

Again, notice that any files loaded in this way will be global, rather than the usual Node.js standard operating procedure of local modules. This may lead to conflicts if you have two versions of the same code.