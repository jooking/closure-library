# Resources for Closure Library #

## Documentation ##

Documentation of the library exists on this wiki, on the [code.google.com site](http://code.google.com/closure/library), in the [generated documentation](http://docs.closure-library.googlecode.com/git/index.html).

But the most important documentation is **in the source**.  The code is documented with [JSDoc comments](http://code.google.com/closure/compiler/docs/js-for-compiler.html) -- each method, function, and class has its parameters documented _with types_, and the compiler enforces these types.

The book, [Closure: The Definitive Guide](http://my.safaribooksonline.com/9781449381882), is now available.

Larry Hosken wrote a [Closure Library tutorial](http://lahosken.san-francisco.ca.us/frivolity/prog/closure_ffdemo/).

Closure Cheet Sheet: http://www.closurecheatsheet.com/

Closure Please http://closureplease.com/

## Tools ##

Closure Library is closely tied to [Closure Compiler](http://code.google.com/closure/compiler), which, in addition to "minifying javascript", provides type checking, dead code removal, and more.

[Closure Inspector](http://code.google.com/closure/compiler/docs/inspector.html) is a tool to help debug code after it has been run through the compiler.

[plovr](http://plovr.com/) is a Java build tool that dynamically recompiles JavaScript and Closure Template code.

[github.com/dturnbull/closure-script](https://github.com/dturnbull/closure-script) is a Ruby build tool that dynamically recompiles JavaScript and Closure Template code.  Also available as a JRuby jar.

[protobuf-plugin-closure](http://code.google.com/p/protobuf-plugin-closure/) is a set of protoc compiler plugins adding support for closure-library goog.proto2.

[closure-library-css-util](http://code.google.com/p/closure-library-css-util/) is a CSS minification tool compatible with goog.setCssNameMapping(..., 'BY\_WHOLE').

## BrowserChannel Servers ##

[github.com/dturnbull/browserchannel](https://github.com/dturnbull/browserchannel) is a complete Ruby implementation.

[libevent-browserchannel-server](http://code.google.com/p/libevent-browserchannel-server/) is a C++ server implementation.

## Non-BrowserChannel Comet ##

[Minerva](http://ludios.org/minerva/) is a work in progress Comet server and client; the server uses Python+Twisted.  It features the same robustness as BrowserChannel.

## Discussion Group ##

There is a [discussion list](https://groups.google.com/group/closure-library-discuss) where library authors and users can help answer questions.