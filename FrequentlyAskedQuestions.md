_Question not answered here? Ask the [Closure Library discussion list](https://groups.google.com/group/closure-library-discuss)._



## How does it compare to other JavaScript libraries and toolkits? ##

More so than other libraries, the Closure Library is geared towards collaborative development and maintanence of complex JavaScript applications, like the sort you see at Google.  Towards that goal, it emphasizes object-oriented style programming, namespacing, dependency declaration, type checking (in conjunction with the [Closure Compiler](http://code.google.com/closure/compiler)), interfaces, and encapsulation.  The library is split into a very broad set of modules and classes.

The library is lower level and _very_ extensive.  It's a benefit to Closure Library developers, but for newcomers, the steeper learning curve can be intimidating.  But our documentation keeps getting fuller, the source is _very_ well documented, and if you **really** get stuck, the authors read and answer questions on the [discussion list](https://groups.google.com/group/closure-library-discuss).

The Closure Library started with the particular design goals of supporting some of the most complex web applications, and presumes a strong familiarity with the JavaScript language and OO-style development.  Closure Library's developers don't purport it to be a "tool for everyone" -- we encourage developers to choose "the best tool for the job."

But for the class of complex applications the library targets, it works wonderfully, and we'll point to the millions of users using Gmail, Reader, Docs, and other Google web applications as proof.

## When I compile with type-checking on, I get warnings about "Bad type annotation. Unknown type" from closure library files. ##

Here is one example of this from [r2029](https://code.google.com/p/closure-library/source/detail?r=2029):
```
    closure/closure/goog/events/events.js:896: ERROR - Bad type annotation. Unknown type goog.debug.ErrorHandler
    * @param {goog.debug.ErrorHandler} errorHandler Error handler with which to
    ^
```

**Does this mean I have to pull those types into my code to get it to compile correctly?**

No, this just means you need to include the closure library deps.js file in your compilation.

The raw compiler option would look something like:
```
    --js closure-library/closure/goog/deps.js.
```

NOTE: If you're using closurebuilder.py (or calcdeps.py), it is not sufficient to simply specifiy "--root closure-library". You have to specifically add the deps file by passing a compiler option:

```
    -f "--js closure-library/closure/goog/deps.js."
```

If you are writing your own JavaScript library based on Closure, you may also need to [generate a deps file for your library](http://code.google.com/closure/library/docs/depswriter.html) and then specifically include it as well.

**This seems really clunky and strange.  Why do I have to do this?  Isn't this a bug?**

At the closure compiler and and closure library level, this is correct behavior bases on the principles:

  * Only goog.require() what your javascript code directly needs and uses to run.

Every time you goog.require() an item, you cause the file that goog.provide()s it to be loaded (or included in the compilation). You also transitively include all the files it depends on.  This happens even if you do not use anything from any of those files. This slows down loading and compilation, and can unnecessarily increase the size of the compiled output.  So, only goog.require() what is needed for your code run.

But to the javascript engine [Type annotations](https://developers.google.com/closure/compiler/docs/js-for-compiler) are just _comments_, not _code_. If a type appears only in annotations, such as inputs to methods in your current file, the engine does not need it to run the javascript in this file, and that type should not be goog.require()d.

Presumably, when that method is called from another file, that other file will have already directly/indirectly require()'d the type to create the object. If that method is never called, that type may not even be needed, it might never be require()'d, and its file might be correctly excluded from compilation.

This is a good optimization, but then the type checker in the closure compiler can't find the excluded (or sometimes just not yet loaded) type and it correctly complains.

**How does including the deps.js file solve this?**

Closure Library contains a deps.js file. The deps file serves two purposes:
  * It lists the location of files, so we can bootstrap them for uncompiled testing.
  * It forward-declares all type names even if their files are not included in compilation.

Unused type names can be optimized away, whereas statics and other file contents cannot.

**Why isn't the deps.js file included like the rest of the js files?**

The deps.js file does not goog.provide() or goog.require() anything, so the tools (correctly) ignore it.