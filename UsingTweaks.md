# Introduction #

The goog.tweak package provides a mechanism for defining debug-time configuration settings, such as timeouts, enabled features, prefetch parameters, etc. When enabled, they can be set through the UI and through query parameters. When disabled, the Closure Compiler will fully strip out the relevant code, leaving no overhead behind.

Check out the [demo page](http://closure-library.googlecode.com/git/closure/goog/demos/tweakui.html).

# Details #

## How a Tweak is Different From a @define ##

  * There's a UI, making all settings easily discoverable and settable.
  * Values can be set using query parameters, no need to recompile to change a value.
  * The syntax is different.

## What Happens When You Click Reload ##

The registry collects all tweak settings that are not set to their default values and creates a query string that encodes their values. Eg)
  * Boolean Values: `?jstweaks=1&noasserts=0`
  * String Values: `?loglevel=info&appcacheversion=1234`
  * Numeric Values: `?numentries=100`
  * Boolean Groups: `?features=fun,-lazy,html5`
By default the query parameters are set to the lowercase tweak label. This can be overridden.

When the page refreshes, the tweaks will set their values from the query parameters.

## Registering a Tweak ##

All tweaks must be registered in the global scope.
All tweak IDs and default value parameters must be literals.

**Booleans (checkboxes):**
```
goog.tweak.registerBoolean('NoAsserts', 'Strip calls to assert().');
goog.tweak.registerBoolean('NoAsserts', 'Strip calls to assert().', true /* defaultValue */);
// The default value is false if not specified.
```

**Strings (Text Box):**
```
goog.tweak.registerString('TestUserName', 'The username for the test account.');
goog.tweak.registerString('TestUserName', 'The username for the test account.', 'tester' /* defaultValue */);
// The default value is '' if not specified.
```

**Strings (Combo Box):**
```
goog.tweak.registerString('LogLevel', 'The minimum severity of log messages to print.', '', {
    validValues: ['All', 'Info', 'Warning', 'Off']
});
// The default value is set to the first value in this set if it is not already in it.
```

**Numbers (Text Box):**
```
goog.tweak.registerNumber('ThreadCount', 'The number of threads in the inbox.');
goog.tweak.registerNumber('ThreadCount', 'The number of threads in the inbox.', 100 /* defaultValue */);
```

**Numbers (Combo Box):**
```
goog.tweak.registerNumber('ThreadCount', 'The number of threads in the inbox.', 0, {
    validValues: [0, 5, 15, 50]
});
// The default value is set to the first value in this set if it is not already in it.
```

**Boolean Group (child entries are grouped into a single query parameter in the form: ?group=enabled1,-disabled2,enabled3)**
```
goog.tweak.beginBooleanGroup('Features', 'Application features to enable');
goog.tweak.registerBoolean('Speech', 'Enables voice control module');
goog.tweak.registerBoolean('Animation', 'Enables the animations module');
goog.tweak.endBooleanGroup();
```

**Adding a button to the UI:**
```
goog.tweak.registerButton('DoSomething', 'Does something', function() { doStuff(); });
```

**More customizations:**
```
goog.tweak.registerBoolean('NoDatabase', 'Do not use a database.', false, {
  // These apply to all tweak types.
  paramName: 'nodb', // The query parameter for this setting.
  restartRequired: false,
  callback: function(tweak) {alert(tweak.getValue())} // Called when the setting is changed via the UI.
});

// Applies only to booleans within a group:
goog.tweak.registerBoolean('NoDatabase', 'Do not use a database', false, {
  token: 'db' // The token in the query parameter (eg: ?features=db)
});
```

## How to Get a Tweak's Value: ##
```
var value1 = goog.tweak.getBoolean('NoCache');
var value2 = goog.tweak.getNumber('ThreadCount');
var value3 = goog.tweak.getString('LogLevel');
```

## How to Enable the UI: ##
```
var tweakElem = goog.tweak.TweakUi.createCollapsible();
// tweakElem will be undefined when tweaks are stripped by the compiler.
if (tweakElem) {
  document.body.appendChild(tweakElem);
}
```

## Organizing Tweaks into Groups (namespaces): ##
The following tweaks will be grouped into a collapsible group named "foo.bar":
```
goog.tweak.registerBoolean('foo.bar.Tweak1', 'Description');
goog.tweak.registerBoolean('foo.bar.Tweak2', 'Description');
goog.tweak.registerBoolean('foo.bar.Tweak3', 'Description');
```

## How to Override the Default Value of a Tweak: ##
There are 5 ways to set tweaks:
  1. The "default" value -- set when by the `goog.tweak.register` call.
  1. The "JSCompiler" override -- set by compiler options.
  1. The "config file" override -- set using `goog.tweak.overrideDefaultValue()` calls.
  1. The "query param" override -- set by the TweakUI upon clicking "Apply Tweaks".
  1. The "run-time" override -- set by the TweakUI the instant a value is changed. Applies only to tweaks that have set `restartRequired` to false.

The precedence goes like this:

> `default < config file < JSCompiler < query param < run-time`

But in-code, the order of appearance looks like

> `JSCompiler/query param < config file < default < run-time`

## Using a config.js file: ##
```
// The compiler will use this value instead of the one passed to goog.tweak.registerString() when inlining values.
// Both values must be literals.
goog.tweak.overrideDefaultValue('LogLevel', 'FINE'); 
```

The idea here is that projects can have a `config.js` file that sets the values they want for tweaks defined in shared libraries.
Eg: things like
```
goog.tweak.overrideDefaultValue('lib.EnableXhrMonitor', false); 
goog.tweak.overrideDefaultValue('lib.NoTimeouts', true); 
```
  * These calls must be made before the corresponding tweak is registered.
  * Multiple `config.js` files can override the same tweak. The last call to overrideDefaultValue() will be the one that is used.
  * Setting the tweak values via the compiler takes precedence over config.js overrides.

## How to Add Tweaks Dynamically: ##
```
// This can be useful for tweaks that control server-side settings.
var tweakRegistry = goog.tweak.getRegistry();
if (tweakRegistry) {
  var boolEntry = new goog.tweak.BooleanSetting('name', otherArgs...);
  tweakRegistry.register(boolEntry);
}
```

## Compiler Support: ##
**Compiler Checks**

  * Check can be enabled by setting `tweakProcessing = TweakProcessing.CHECK`
  * The ID and default value parameters must be string literals for:
    * `goog.tweak.register{Boolean|Number|String|Button}`
    * `goog.tweak.get{Boolean|Number|String}`
    * `goog.tweak.overrideDefaultValue`
  * The ID parameter of `goog.tweak.get{Boolean|Number|String}` and `goog.tweak.overrideDefaultValue` must correspond to a `goog.tweak.register{Boolean|Number|String}` function.
  * All goog.tweak.register{Boolean|Number|String|Button}` functions and goog.tweak.overrideDefaultValue must appear in the top-level context

**Tweak Stripping**

  * This is enabled by setting `tweakProcessing = TweakProcessing.STRIP`
  * All calls to `goog.tweak.get{Boolean|Number|String}` will be replaced with the default value for the tweak.
  * All other calls functions within goog.tweak will be stripped.

**Overriding a Tweak Value**

  * This can be done through calls to `setTweakToBooleanLiteral`, `setTweakToStringLiteral`, or `setTweakToNumberLiteral`
  * When combined with `tweakProcessing = TweakProcessing.STRIP`, this will set the value returned for calls to `goog.tweak.getBoolean('TweakId')`.
  * When `tweakProcessing = TweakProcessing.CHECK`, this will override the default value for the tweak.