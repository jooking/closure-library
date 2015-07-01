# Preview Warning #

This preview document is a rough draft. It may contain incorrect or outdated information, so use at your own risk.

When this doc is published at http://code.google.com/closure/library/ , it will be deleted from this wiki and replaced with a link to the polished version. Wiki edits to this preview version may be incorporated into the polished version, or they may be lost. Edit at your own risk.

We welcome feedback! Use the comments field or closure-library-discuss@googlegroups.com to give feedback.

The sample code at <a href='http://closure-library.googlecode.com/git/closure/goog/demos/samplecomponent.html'>samplecomponent.html</a> is now outdated, where they differ, use the code from this document instead.

# The Closure UI Framework: An Introduction to Components #

## Overview ##

Closure's goog.ui package provides a large selection of
ready-made user interface widgets: buttons, menus, toolbars,
autocompletes, popup calendars, and more. But more than this, it
provides an architecture for new interface components.

The base class goog.ui.Component forms
the foundation of both Closure's existing UI components and the
extensibility of the goog.ui package.
The Component class defines a standard
pattern that can help you build components that are easy to use and
maintain.

In practice, most interface elements you design or use will probably
be subclasses of either Control
or Container. Both of these classes are
extensions of Component, however, and
some understanding of the Component
class will help you work with them more effectively. This document
provides an introduction to the Component class by
describing how to create a Component
subclass.

The core pattern defined by
the Component class is a sequence of
stages for the creation and destruction of
a Component subclass
instance. Subclasses implement methods for each of these stages, along
with a couple of support methods. The first six methods listed below
correspond to the stages in a component's life cycle (in chronological
order), while the last is a required support method.

<table>
<tr><th>Method</th><th>Life Cycle Stage (or Purpose)</th></tr>
<tr><td>constructor</td> <td>Component instance creation</td></tr>

<tr><td>createDom()</td> <td>Component DOM<br>
structure building</td></tr>
<tr><td>decorateInternal() (optional)</td></tr>
<tr><td>enterDocument()</td> <td>Post DOM-building<br>
initialization (such as attaching event listeners)</td></tr>
<tr><td>exitDocument()</td> <td>Post DOM-removal cleanup<br>
(such as detaching event listeners)</td></tr>
<tr><td>dispose()</td> <td>Component disposal</td></tr>

<tr><td>canDecorate()</td> <td>Indicates whether the<br>
component can use a pre-existing element</td></tr>
</table>
## Quick Example ##

If you develop a UI component using
Closure's Component pattern, you end up
with a widget that you can include in a page with just a few lines of
JavaScript. For example, in
the <a href='http://closure-library.googlecode.com/git/closure/goog/demos/samplecomponent.html'>samplecomponent.html</a>
demo in the demos directory you can see a simple
component that changes color when clicked. The following code
initializes the first instance of this component and places it on the
page:

```
var box1 = new goog.demos.SampleComponent();
box1.render(goog.dom.getElement('target1'));
```

## Component Construction and Initialization ##

By implementing separate methods to create an instance of
the Component, to build out its DOM
structure, and to attach its event listeners, you enable simple
initialization for your own Component
subclass.

### Creating the Component Instance ###

The first stage in the life of
a Component instance is, of course, its
creation with a call to its constructor. Your own subclasses
of Component must define a constructor
that calls the Component
constructor. Furthermore, every subclass constructor must take an
optional DomHelper parameter
(see <a href='dom-manipulation.html'>Using Closure to Create, Insert,<br>
and Retrieve DOM Objects</a>) and pass that parameter along to
the Component constructor, as
illustrated below. For example, the following code from the demo
JavaScript file
(<a href='http://closure-library.googlecode.com/git/closure/goog/demos/samplecomponent.js'>samplecomponent.js</a>)
shows the SampleComponent constructor,
along with the call to inherits() to
subclass Component:

```
goog.demos.SampleComponent = function(opt_label, opt_domHelper) {
  goog.base(this, opt_domHelper);

  /**
   * The label to display.
   * @type String
   * @private
   */
  this.initialLabel_ = opt_label || 'Click Me';

  /**
   * The current color.
   * @type String
   * @private
   */
  this.color_ = 'red';

  /**
   * Keyboard handler for this object. This object is created once the 
   * component's DOM element is known.
   *
   * @type goog.events.KeyHandler|Null
   * @private
   */
  this.kh_ = null;
};
goog.inherits(goog.demos.SampleComponent, goog.ui.Component);
```

### Building the Component's DOM Representation ###

After the Component instance has
been created, it must build a Document Object Model structure to serve
as the actual interface for
the Component in the document. To
handle this phase in your Component subclass' life cycle, you must
implement a `createDom()` method, and you may also
implement the optional `decorateInternal()` method:

<table>
<tr><td><code>createDom()</code></td> <td>Builds<br>
the Component's DOM tree and stores its<br>
root in <code>this.element_</code>.</td></tr>
<tr><td><code>decorateInternal(element)</code>
(optional)</td><td>Takes an Element<br>
that is already present in the document, performs any transformations<br>
necessary to turn it into a structure equivalent to the structure<br>
created by <code>createDom()</code>, and stores it<br>
in <code>this.element_</code>.</td></tr>

</table>

The `createDom()` method builds
the Component's DOM structure from
scratch, while the `decorateInternal()` method takes an
existing DOM structure and turns it into a representation of
the Component. As noted, both methods
must store the root Element of the
resulting DOM structure by
calling `this.setElementInternal(element)`. The private `element_` field is defined
on Component,
and Component's own initialization code
expects to find the DOM Element for the
widget in this field. This example
from <a href='http://closure-library.googlecode.com/git/closure/goog/demos/samplecomponent.js'>samplecomponent.js</a>

shows the creation of the single div used
by SampleComponent:

```
goog.demos.SampleComponent.prototype.createDom = function() {
  this.decorateInternal(this.dom_.createElement('div'));
};

goog.demos.SampleComponent.prototype.decorateInternal = function(element) {
  this.setElementInternal(element);
  if (!this.getLabelText()) {
    this.setLabelText(this.initialLabel_);
  }
  ...
};
```

If all you need is a single
empty div Element, you can
use the default version of `createDom()` supplied
in Component. Component

also provides a default version of `decorateInternal()` that sets
the `element_` field and does nothing more.

### Post-DOM-Building Initialization of the Component ###

Some initialization tasks require the presence of
the Component's DOM structure in the
document. For example, if you are creating a popup menu and you need
to attach an event handler to the div with
id "menu", you won't be able to retrieve
the div by id unless it already exists in the
document. If your Component requires
any such initialization, you must implement an `enterDocument()`
method:

<table>
<td><code>enterDocument()</code></td> <td>Performs any initialization<br>
tasks that require the presence of the Component's DOM structure in<br>
the document (such as attaching event listeners)</td>
</table>

Any implementation of `enterDocument()` must call the
superclass `enterDocument()` method, which propagates the
call to all of the component's children (see "Component Composition"
below). The `enterDocument()` method is a good place to
attach event handlers to Elements,
especially Elements that need to be
retrieved via `getElementById()` (that is, elements that
are not retrieved via some sort of internal bookkeeping on the part of
the component, but that instead depend on the presence of the element
in the document). This can be done using the event handler defined by the superclass, goog.ui.Component (accessed via this.getHandler()). The SampleComponent
class in
the <a href='http://closure-library.googlecode.com/git/closure/goog/demos/samplecomponent.html'>samplecomponent.html</a>
example uses `enterDocument()` to attach a click listener:

```
goog.demos.SampleComponent.prototype.enterDocument = function() {
  goog.base(this, 'enterDocument');
  this.getHandler().listen(this.getContentElement(), goog.events.EventType.CLICK, this.onDivClicked_);
};
```

While it is not absolutely necessary for this particular listener to
be attached in the `enterDocument()` method (the listener
could be attached to the `element_` member before that
element was actually added to the document), attaching event listeners
for your component in `enterDocument()` is a generally safe
practice.

### Initialization Convenience Methods: render() and decorate() ###

The Component base class provides
convenience methods to handle the entire initialization
process. Any Component subclass that
properly implements `createDom()`
and `enterDocument()` can be initialized with a call to the
base class `render()` method, as illustrated in
the <a href='http://closure-library.googlecode.com/git/closure/goog/demos/samplecomponent.html'>samplecomponent.html</a> demo:

```
var box1 = new goog.demos.SampleComponent();
box1.render(goog.dom.getElement('target1'));
```

The `render()` method calls `createDom()`, places the
resulting `element_` in the document, and then calls
`enterDocument()`. If an Element
parameter is supplied to `render()` it will place
the `element_` in the document as a child of the
parameter Element (as in the above
example). If no parameter is supplied, the Component's DOM structure
will be placed in the document as a child of
the body Element.

The <a href='http://closure-library.googlecode.com/git/closure/goog/demos/samplecomponent.html'>samplecomponent.html</a>
page presents another example as well:

```
// Shows label taken from DIV text
var box2 = new goog.demos.SampleComponent();
box2.decorate(goog.dom.getElement('target2'));
```

The `decorate()` method takes an existing element as a
parameter, passes that Element in a
call to `decorateInternal()`, and then
calls `enterDocument()`.

Note that a Component subclass
should never itself call `render()`
or `decorate()`.  Note also that if you do not
use `render()` or `decorate()` to initialize and
place your component instances, you <em>must</em> include your own
call to `enterDocument()` after the component's element has
been added to the document.

These two methods, `render()` and `decorate()`,
serve as two examples of the process of component-building defined by
the Component pattern. Although they
provide useful convenience methods, you can define and use your
own Component classes without
invoking `render()` or `decorate()`.

### Summary ###

Unless you are willing to use a default version from the superclass,
your subclass of Component must
implement each of these construction and initialization methods:

  * constructor function: The constructor must take an optional DomHelper parameter and pass it along to the Component constructor.
  * `createDom()`: This method builds the DOM Element `element` for the component from scratch, and sets `this.element_ = element` Component provides a default version that creates a single div.
  * `decorateInternal(element)`: This method takes the Dom Element `element`, sets `this.element_ = element` and performs whatever transformations need to be performed in order to prepare the Dom structure (adding default label text to a div if no text is present, for instance, as in the example above). Component provides a default version that sets `this.element_` and does nothing else.
  * `enterDocument()`: This method performs any initialization of the component that requires the DOM structure to be complete and present in the document (such as attaching event listeners to elements retrieved with `getElementById(id)`). It must call `enterDocument()` on Component.

## Component Destruction ##

Any good Component should be able to
clean up after itself on request, so that users of
the Component can easily tell it to
free up its resources. Component cleanup involves:

  * removing any event listeners on the component's DOM element and sub-elements (handled by goog.ui.Component)
  * removing any object references in the Component instance's members
  * removing from the DOM any Elements that were created with `createDom()`

Component subclasses may therefore need to implement one or both of the following two methods:

<table>
<tr><td><code>exitDocument()</code></td> <td>Reverse changes made in 'enterDocument()', goog.ui.Component's exitDocument implementation makes a call to 'removeAll' for the Component's event handler accessed via 'this.getHandler()' </td></tr>
<tr><td><code>dispose()</code></td> <td>Component disposal, including<br>
object reference cleanup</td></tr>
</table>

The Component base class provides
an `exitDocument()` method, this method calls `exitDocument()` on all
of the Component's children and removes all event listeners registered
on the Component's event handler. If a Component subclass
only attaches event listeners and doesn't make any other changes in
an `enterDocument()` method, this superclass method is
sufficient. But if the subclass does other changes aside from attaching
event handlers in 'enterDocument()', it should also implement
an `exitDocument()` method that undoes any changes
performed in `enterDocument()`. It must always be possible
to call `exitDocument()`, remove
a Component's element from the
document, and then add the element back in somewhere else and
call `enterDocument()`. All implementations
of `exitDocument()` should call the
superclass `exitDocument()` in order to make sure any child
components have `exitDocument()` called on them.

This `exitDocument()` method from
the <a href='http://closure-library.googlecode.com/git/closure/goog/demos/samplecomponent.html'>samplecomponent.html</a>
demo removes the SampleComponent
instance's event listeners:

```
  goog.demos.SampleComponent.prototype.exitDocument = function() {
    // Undo any non event handler related changes here

    // Call superclass 'exitDocument' function at the end of our exit document
    goog.base(this, 'exitDocument');
  };
```

The Component base class also provides
a `dispose()` method that may be sufficient. The base
class `dispose()`:

  * calls `exitDocument()`
  * calls `dispose()` on all child components
  * removes `element_` from the document, and
  * nulls out all of the component's member object references.

If your subclass has further object references that need to be erased
you must implement a `dispose()` method that nulls them out
and then calls the superclass `dispose()`. SampleComponent
has the following dispose method:

```
goog.demos.SampleComponent.prototype.dispose = function() {
  if (!this.getDisposed()) {
    if (this.kh_) {
      this.kh_.dispose();
    }
    goog.base(this, 'dispose');
  }
};
```

<a href='http://closure-library.googlecode.com/git/closure/goog/demos/samplecomponent.js'>samplecomponent.js</a>

## Component Composition ##

With the `addChild()` and `addChildAt()`
methods, Component introduces the
ability to build hierarchical structures of components. As has been
mentioned above, the `exitDocument()` method
of Component calls
the `exitDocument()` method of all child components, and a
call
to Component's `dispose()`

similarly trickles down to children. Organizing components
hierarchically thus offers the convenience of disposing of multiple
components with a single
call. Component also calls
the `enterDocument()` method of all of
child Components,
and `render()` defers calling `enterDocument()`
for any component if its parent component has not yet entered the
document. These conditions guarantee that `enterDocument()`

will be called for all of the items within a
given render-ed component hierarchy in a single cascade
when it is called on the top component in the hierarchy,
with `enterDocument()` being called on the child components
only after their parents are known to be placed in the document. If
you do not use `render()` to initialize and place a
component, you must enforce a similar guarantee yourself. You must
only call a child's `enterDocument()` method after both the
parent and the child have been placed in the document.

The `addChild()` method has the following signature:

```
addChild(child, opt_render)
```

Passing an `opt_render` parameter of true will
automatically render the child component's element.

To add a child to a component with a call
to `addChild(element)`, the following
preconditions must be met:

  * the child component's element must be a descendant of the parent component's element
  * the child must not yet be in the document, unless the parent is also in the document and opt\_render is false
  * the child must not already have a parent

The `addChildAt()` method allows you to insert a child
component at a particular spot in the parent's child list. It has all
of `addChild()`'s preconditions, and has the following
signature:
```
addChildAt(child, insertionIndex, opt_render)
```
The child will be inserted at index `insertionIndex` in the
child list. The value of `insertionIndex` must be
between 0 and the current child count, inclusive.

## Other Methods ##

The above-described methods form the core of the design pattern
defined by Component. Subclasses must
implement the following support method as well:

  * `canDecorate(existingElement)`: This method must return a boolean value indicating whether or not the component can decorate Element `existingElement`. The superclass default always returns true.

In addition to the methods a subclass
of Component should implement, there
are a number of public convenience methods that you would only want to
override in extremely unusual cases. For a complete list of
the Component class's public methods,
consult
the [goog.ui.Component API Reference](http://docs.closure-library.googlecode.com/git/class_goog_ui_Component.html).