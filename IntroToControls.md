# Preview Warning #

This preview document is a rough draft. It may contain incorrect or outdated information, so use at your own risk.

When this doc is published at http://code.google.com/closure/library/ , it will be deleted from this wiki and replaced with a link to the polished version. Wiki edits to this preview version may be incorporated into the polished version, or they may be lost. Edit at your own risk.

We welcome feedback! Use the comments field or closure-library-discuss@googlegroups.com to give feedback.

# The Closure UI Framework: An Introduction to Controls #

## Overview ##

Closure's Component class (described in [An Introduction to Components](IntroToComponents.md)) supplies methods and patterns for managing the life cycle
of a user interface widget, but it includes no actual implementation
of interface functionality.

Component's subclass Control fills this gap by
providing a canonical Component implementation that has built-in support for many of the standard behaviors one expects from an interface
element.

The Control class also provides state management abstractions that make the logic of
a Control instance highly configurable, and includes methods for
manipulating Control content (like button labels and menu item names) and
visibility. Control's configurable logic means that you can often
adapt Control or one of its existing subclasses to your application without having to write your own
subclass.

If you need a simple widget like a button, this introduction will help
you both to use the Control class out
of the box and to configure it to suit your needs. It will also give
you the foundation you need to go on to learn how to write your
own Control subclasses, how to use custom renderers, and how to
incorporate Controls like menu items
and toolbar buttons into Containers
like menus and toolbars.

## Quick Example ##

You can instantiate a button-like Control and add it to a document with this code:

```
var c = new goog.ui.Control('Click me');
c.render(goog.dom.getElement('parentForC'));
```

To make your application react to interaction with the Control, attach event handlers like this one:

```
goog.events.listen(c, goog.ui.Component.EventType.ACTION,
    function (e) {
        c.setCaption('You clicked me!');
    });
```

[Control demo](http://closure-library.googlecode.com/git/closure/goog/demos/control.html) shows that a Control instance appears and behaves much like a button.

In many cases, the functionality built into Control (or one of its subclasses)
will be sufficient, and you won't need to do anything beyond the above techniques to make use the class.

If you do, however, want to customize the class for your particular needs, you might want to study the concepts
and more detailed examples described below.

## Concepts and Definitions ##

As it was mentioned in the overview, there are two main aspects of the Control: its behavior and its appearance.

### State Management Concepts (behavior abstractions) ###

Control state management is embodied in three basic concepts: **state**, **state transition** and **transition event**.
#### State ####
An abstraction that reduces the
details of mouse and keyboard interaction with
a Control to a small, manageable set of
properties. The goog.ui.Control class
provides methods for keeping track of different aspects of
a Control's state. For example, with
the method isActive() you can test whether
a Control is "Active" (or
"pressed"). State constants are enumerated
in goog.ui.Component.State. The
Control
class uses the concept of state to allow easy customization of various
aspects of a Control's behavior, including when to fire
events (see <a href='#def2-3'>States with Transition Events</a> below)
and what CSS styles to apply to
a Control's DOM
element. <a href='#ex3'>See Example 3: Configuring State-based<br>
Behavior</a> below for more details.<br>

<h4>State Transition</h4>

A change in the<br>
boolean value of some aspect of<br>
a Control's state (when<br>
a Control changes from being unselected<br>
to being selected, for example).<br>
<br>
<h4>Transition Events</h4>

A goog.events.Event<br>
object dispatched in response to a state transition in<br>
a Control. A Control<br>
can be configured to fire events only when particular kinds of state<br>
transitions take place (see <a href='#def2-3'>States with Transition<br>
Events</a> below). It is simpler for your code to track transition<br>
events than to track mouse actions or key<br>
presses. Controls also fire some event<br>
types that are not state transition events (see <a href='#evs'>Event<br>
Types</a> below).<br>
<br>
<h3>Types of States</h3>

<p><dl>
<dt><a></a>Supported States</dt><dd>The set of state<br>
flags that can be set and tested (see isSupportedState()<br>
and setSupportedState() below). Events will not be<br>
dispatched for state flags that are not supported (see "States with<br>
Transition Events" below), and unsupported state flags will not<br>
exhibit AutoState behavior (see "AutoStates" below). The default set<br>
of supported state flags is DISABLED, HOVER, ACTIVE, FOCUSED. With<br>
support for just these four state flags, a simple button-like control<br>
can be enabled or disabled, can change its appearance when moused over<br>
or clicked, and can receive keyboard focus. However, the application<br>
developer may choose to add or remove support for particular state<br>
flags using setSupportedState().<br>
</dd>
<br>

<dt><a></a>AutoStates</dt><dd>The set of states for<br>
which a Control will use the default<br>
behavior implemented in the Control<br>
class. AutoState implementations translate primitive user-interaction<br>
events like mouseover and mousedown into the higher level abstraction<br>
of states, automatically setting the state and appearance of<br>
the Control based on user actions. For<br>
example, a control whose set of AutoStates includes HOVER will<br>
automatically highlight on mouseover and un-highlight on<br>
mouseout. While the Control class<br>
implements default behavior for all the state flags enumerated<br>
in goog.ui.Component.State, you can<br>
choose whether or not to use this built-in implementation for a given<br>
state flag by choosing whether to include the flag in the set of<br>
AutoStates (see setAutoStates()<br>
in <a href='#otherconfig'>Other Configuration Methods</a> below).</dd>
<br>

<dt><a></a>States with Transition Events</dt><dd>The set<br>
of state flags that trigger the dispatch<br>
of goog.events.Events in response to<br>
changes in their values. For example, a control that is set to<br>
dispatch transition events for the HOVER state will dispatch a<br>
HIGHLIGHT event on mouseover and an UNHIGHLIGHT event on mouseout. A<br>
state flag can be supported without dispatching transition events<br>
(state changes will be made silently for such flags), but in order to<br>
dispatch transition events a state flag must be supported.<br>
</dd><br>

<h4>Control Display Concepts (presentational appearance abstractions)</h4>

<p><dl>
<dt><a></a>Renderer</dt> <dd>An object that handles the<br>
display of<br>
a Control. A Control's<br>
renderer is notified of all changes in<br>
the Control's state, and in response<br>
changes the CSS classes and ARIA (i.e., accessibility) state of<br>
the Control's DOM element to reflect<br>
the new states. Because a single renderer instance can be reused for<br>
an entire class of Controls, a renderer<br>
class typically provides a getInstance() method to<br>
retrieve a singleton instance of the<br>
class. The Control class uses an<br>
instance of ControlRenderer as its default renderer.</dd><br>

<dt><a></a>Control DOM element</dt><dd>The DOM element<br>
at the root of the DOM structure that represents<br>
a Control. A renderer can either create<br>
a DOM structure for the Control from<br>
scratch (through its createDom() method) or it can use an<br>
existing DOM structure (through its decorate()<br>
method). </dd><br>

<dt><a></a>Content</dt><dd>A text caption or DOM<br>
structure to display in the Control. A<br>
button's content, for example, might be its label. The default control<br>
renderer provides basic implementations of setContent()<br>
and (for text content) setCaption() methods for<br>
manipulating Control<br>
content. Control subclasses with more<br>
complicated content can implement their own methods for manipulating<br>
the various parts of the content.  For example,<br>
a Control might include both a text label and a thumbnail image, and so might provide both a setCaption() and a setThumbnail() method.</dd><br>

<dt><a></a>Decorator<br>
Registry</dt><dd>The goog.ui.registry namespace contains<br>
a map from CSS classnames to factory functions that<br>
produce Components for use in<br>
decorating existing DOM elements. This registry allows you to use CSS<br>
classes in HTML to indicate a Control<br>
class to use for a particular element. Register your own factories by<br>
calling the goog.ui.registry.setDecoratorByClassName()<br>
function. Get a decorator for an element by<br>
calling goog.ui.registry.getDecorator(element), or just<br>
call the convenience method goog.ui.decorate(element),<br>
which does this step for you and then decorates the<br>
element.</dd><br>

<dt><a></a>State Styles</dt><dd>As the user interacts<br>
with a Control,<br>
the Control changes the appearance of<br>
its DOM representation by adding and subtracting styles to reflect its<br>
current state. So, for example, if an element is active, it would have<br>
the class name goog-control-active. If it is active and<br>
checked, it would have two class<br>
names: goog-control-active<br>
and goog-control-checked. See<br>
the <a href='#styles'>Styles</a> table for a complete list of these<br>
state styles.</dd><br>

<h2>Detailed Examples</h2>

<h3>Example 1: Creating a Control's DOM Structure from Scratch</h3>

To create a simple button Control that<br>
builds its own DOM structure, we call<br>
the Control constructor and pass in the<br>
button label:<br>
<br>
<pre><code>var c1 = new goog.ui.Control('Hello, world!');<br>
</code></pre>

With a call to<br>
the Control's render()<br>
method we then tell the Control to<br>
build its DOM structure, add it to the document as a child of the<br>
element parentForC1, and attach event handlers:<br>
<br>
<pre><code>c1.render(getElement('parentForC1'));<br>
</code></pre>

If you go to<br>
the <a href='http://closure-library.googlecode.com/git/closure/goog/demos/control.html'>Control demo</a> you can see this "Hello World" button in action.<br>
<br>
The event log on the right side of the page will show you what events<br>
this Control is firing. Try mousing over the Control and clicking on<br>
it.<br>
<br>
You should see only three kinds of<br>
events: ENTER, LEAVE and ACTION. These three event types are fired by<br>
all Controls.<br>
<br>
All Controls can also<br>
fire SHOW and HIDE events, although we have<br>
not included code here to toggle the visibility of<br>
our Control. By default<br>
a Control will not have any transition<br>
events enabled, and so we only see default event types in this<br>
example.<br>
<br>
<h3><a></a>Example 2: Decorating a DOM Element with a Control</h3>
<p>If we have the following &lt;span&gt; element:</p>

<pre><code>&lt;span id="parentForC2"<br>
    class="goog-inline-block goog-control"<br>
    style="vertical-align:middle;"&gt;<br>
    Decorated Example<br>
&lt;/span&gt;<br>
</code></pre>

then we can attach a Control to it with the following code, rather than building a DOM element from scratch:<br>
<br>
<pre><code>var elem = goog.dom.getElement('parentForC2');<br>
var c2 = goog.ui.registry.getDecorator(elem);<br>
c2.decorate(elem);<br>
</code></pre>

The presence of the goog-control CSS class in the HTML<br>
tag allows us to get our Control<br>
instance using the goog.ui.Control.getDecorator()<br>
method. The getDecorator() method consults the decorator<br>
registry to see whether a Control factory has been registered for any<br>
of the &lt;span&gt;'s CSS classes. It finds<br>
the Control factory that is registered<br>
by default with the class name goog-control, and uses<br>
that factory to return a Control<br>
instance.<br>
<br>
We could condense this process to just two lines using the convenience<br>
method goog.ui.decorate():<br>
<br>
<pre><code>var elem = goog.dom.getElement('parentForC2');<br>
var c2 = goog.ui.decorate(elem);<br>
</code></pre>

We don't have to use the decorator registry at all, however. Instead<br>
of calling getDecorator() we could just<br>
initialize c2 as follows:<br>
<br>
<pre><code>var c2 = new goog.ui.Control();<br>
</code></pre>

Once we have a Control instance we call<br>
its decorate() method, which prepares the existing DOM<br>
element for use as a Control and then<br>
calls enterDocument() to attach event handlers. Much of<br>
the work of decorate() is delegated to the<br>
renderer. These examples use the default renderer<br>
(goog.ui.ControlRenderer), which<br>
retrieves the content for the Control<br>
from the decorated element and initializes the Control's<br>
state based on the element's CSS classes.<br>
<br>
<h3>Example 3: Configuring State-based Behavior</h3>

Our next example demonstrates how to use<br>
a Control's state to customize several<br>
aspects of its behavior and appearance. We instantiate and initialize<br>
the Control for this example with the<br>
following code:<br>
<br>
<pre><code>var elem = goog.dom.getElement('parentForC3');<br>
var c3 = new goog.ui.Control();<br>
c3.decorate(elem);<br>
</code></pre>

This code attaches the Control to the<br>
following &lt;span&gt;:<br>
<br>
<pre><code>&lt;span id="parentForC3"<br>
   class="goog-inline-block goog-control goog-control-disabled"<br>
   style="vertical-align:middle;"&gt;<br>
   Decorated Example<br>
&lt;/span&gt;<br>
</code></pre>

<h4>Styling with States</h4>
<p>
The above HTML is almost identical to the HTML for Example 2, but it<br>
assigns an additional CSS class to the &lt;span&gt;<br>
element: goog-control-disabled. If you view<br>
the Control <a href='http://closure-library.googlecode.com/git/closure/goog/demos/control.html'>on<br>
the demo page</a> (the second button from the top), you will see that it is<br>
indeed disabled. The decorate() method initializes the<br>
state of the component from the classnames of its element. If it finds<br>
any of the state-setting classnames listed in<br>
the <a href='#styles'>Styles</a> table below, the logical state of<br>
the Component will be set to match the<br>
classnames.<br>
</p>
<p>
The Control will also automatically<br>
update CSS classes in response to state changes. You can thus control<br>
the visual appearance of the various states of<br>
your Control entirely from within style<br>
sheets.<br>
</p>
<h4><a></a>Controlling State Transition Events</h4>
<p>
We can control the state transition events dispatched by<br>
our Control by<br>
calling setDispatchTransitionEvents().<br>
The <a href='http://closure-library.googlecode.com/git/closure/goog/demos/control.html'>demo</a>
Control<br>
discussed in Example 2 above<br>
uses setDispatchTransitionEvents() to enable all<br>
transition events:</p>

<pre><code>c2.setDispatchTransitionEvents(goog.ui.Component.State.ALL, true);<br>
</code></pre>

All of the state constants<br>
in goog.ui.Component.State are<br>
bitmasks, and the first argument<br>
to setDispatchTransitionEvents should be some bitwise<br>
combination of these constants. The value<br>
of goog.ui.Component.State.ALL, not surprisingly, has all<br>
of its bits set (0xFF), and so this argument tells<br>
the Control to dispatch state<br>
transition events for all possible state transitions. The second<br>
parameter indicates whether to enable (true) or disable<br>
(false) transition events for the indicated state<br>
flag.<br>
<br>
We can exert finer-grained control over state transition events,<br>
however. To enable transition events for a subset of states, just pass<br>
to setDispatchTransitionEvents() the bitwise 'or' of<br>
state constants. For example, if you wanted<br>
your Control to fire transition events<br>
only when the 'enabled' state property or the 'focused' property<br>
changes, you could use this code:<br>
<br>
<pre><code>var transitions = goog.ui.Component.State.DISABLED | goog.ui.Component.State.FOCUSED;<br>
c3.setDispatchTransitionEvents(transitions, true);<br>
</code></pre>

(Note: if we made the second argument false, instead of<br>
enabling transition events for these state flags, we would disable<br>
them.)<br>
<br>
Our <var>transitions</var> variable excludes, for<br>
example, goog.ui.Component.State.HOVER. Thus if you<br>
moused over a Control configured with the <var>transitions</var>
flags you would not see 'highlight' and 'unhighlight' events<br>
dispatched.<br>
<br>
You may notice that "Hello, world!" Control still dispatches 'enter'<br>
and 'leave' events even though it has no state transition events<br>
enabled. These event types<br>
(goog.ui.Component.EventType.ENTER<br>
and goog.ui.Component.EventType.LEAVE) are not state<br>
transition events. Rather than indicating a change in the logical<br>
state of the Control, they represent<br>
properties of the view -- in this case, raw user mouse actions. What's<br>
the difference? Well, the Control<br>
class' management of HOVER state adds a layer of useful<br>
logic to the raw mouse actions by not changing the HOVER<br>
state when the Control is disabled<br>
(more on this below). You can find a complete list of state transition<br>
events in the <a href='#evs'>Event Types</a> table below.<br>
<br>
<h4>Controlling AutoStates</h4>

The Control builds-in many useful<br>
default behaviors for particular states. You may find, however, that<br>
you need to disable or alter some of these behaviors for your<br>
application. You can do this by modifying the set of AutoStates --<br>
that is, the set of states for which the default behavior is<br>
enabled -- with a call to setAutoStates(). For example:<br>
<br>
<pre><code>var disabledAutoStates = goog.ui.Component.State.HOVER;<br>
c4.setAutoStates(disabledAutoStates, false);<br>
</code></pre>

Like setDispatchTransitionEvents(), setAutoStates()<br>
takes some bitwise combination of<br>
the goog.ui.Component.State<br>
constants. Go to the <a href='http://code.google.com/closure/library/samples/control.html'>demo</a>, enable<br>
the "AutoState Example" button, and try rolling over it. Not only do<br>
no 'highlight' or 'unhighlight' events fire, but the highlighting<br>
behavior has also been completely removed. Note that disabling state<br>
transition events for HOVER in the previous example did<br>
not disable the highlighting behavior; although<br>
the Control wasn't telling the outside<br>
world that it was being highlighted, its own internal handling<br>
of HOVER state went on as usual. By disabling<br>
the HOVER <em>AutoState</em>, however, we have disabled<br>
both transition events and internal handling of HOVER<br>
state.<br>
<br>
<h4>Triggering Application Actions with State Transition Events</h4>

Once you understand how to use state transition events, using them to<br>
control your application is simply a matter of attaching event<br>
listeners. For example, the "hint" fields underneath the buttons in<br>
the <a href='http://closure-library.googlecode.com/git/closure/goog/demos/control.html'>demo</a> are controlled by event<br>
listener attachments like this one:<br>
<br>
<pre><code>goog.events.listen(c4,<br>
    goog.ui.Component.EventType.ENABLE,<br>
    function(e) {<br>
      goog.dom.setTextContent(goog.dom.getElement('hint3'),<br>
          'Click to activate.'); <br>
    });<br>
</code></pre>

Listening for state transition events like HOVER rather<br>
than user interaction events like ENTER<br>
and LEAVE can drastically simply the logic of your event<br>
listeners. In the "State Example" button in<br>
the <a href='http://code.google.com/closure/library/samples/control.html'>demo</a>, for example, we<br>
disable HOVER events and instead use ENTER<br>
events to trigger the mouseover hint. But because we haven't<br>
implemented our own check to prevent highlighting when<br>
the Control is disabled, we clobber our<br>
"You need to enable this component" warning when we mouseover the<br>
disabled Control. The HOVER<br>
state handling would have managed this logic for us.<br>
<br>
<a href='http://code.google.com/closure/library/samples/control.html'>control.html</a>

<h2>Control Initialization and Customization Reference</h2>

A Control instance can be customized<br>
through both constructor parameters and a wide variety of<br>
configuration methods.<br>
<br>
<h3>Constructor Parameters</h3>

The full signature of the goog.ui.Control constructor is:<br>
<br>
<pre><code>goog.ui.Control(content, opt_renderer, opt_domHelper)<br>
</code></pre>

<dl>
<dt><var>content</var></dt><dd><p>Either text content to<br>
apply to the Control (a label for<br>
button Controls, the menu text for menu<br>
item Controls, etc.), a DOM node to<br>
include in the Control's DOM structure, or an array of<br>
DOM nodes. Any content supplied in the content parameter<br>
is ignored if the Control is used to decorate an existing<br>
DOM structure.</p></dd>
<dt><var>opt_renderer</var></dt><dd><p>Optional. A<br>
renderer instance that will control the appearance of<br>
the Control.<br>
Control<br>
supplies a default renderer for use when this parameter is<br>
omitted.</p></dd>
<dt><var>opt_domHelper</var></dt><dd><p>Optional. A<br>
goog.dom.DomHelper<br>
instance to use for the construction of<br>
the Control. You may want to use this<br>
parameter if you are working with multiple<br>
frames. </p></dd>
</dl>

<h3>Control Configuration Methods</h3>
<h4>State Configuration Methods</h4>

The following methods all modify some aspect of<br>
the Control's state. In all cases, if<br>
the relevant state property is not supported, or if the state property<br>
already has the requested value, the method will have no effect. If<br>
state transitions for the relevant state property are turned on,<br>
the Control will fire a state<br>
transition event of the appropriate type (see <a href='#evs'>Event<br>
Types</a> below). If any listener functions for this event type return<br>
false, the specified state change will not be applied.<br>
<br>
<p>
In general you do not need to call these methods in response to mouse<br>
and keyboard events: AutoState behaviors will handle translating user<br>
actions into state for you.<br>
</p>
<p>
Whenever a state change takes place the renderer is given the<br>
opportunity to alter the appearance of<br>
the Control in response to the<br>
change. The default renderer does so by adding or removing the<br>
classnames listed in the <a href='#styles'>Styles</a> table below.<br>
</p>
<dl>
<dt>setHighlighted(highlight)</dt><dd>Highlights or<br>
unhighlights the Control (that is,<br>
modifies its goog.ui.Component.State.HOVER state<br>
property). </dd><br>

<dt>setActive(active)</dt><dd>Activates or deactivates<br>
the Control (that is, modifies<br>
its goog.ui.Component.State.ACTIVE state property). For<br>
example, the AutoState behavior activates<br>
a Control when it is being pressed and<br>
deactivates it when it is released. </dd><br>

<dt>setFocused(focused)</dt><dd> Applies or removes<br>
styling indicating that the Control has<br>
keyboard focus, and modifies<br>
the Control's<br>
goog.ui.Component.State.FOCUSED<br>
state property. Note that, unlike the other state-setting methods,<br>
this method is called as a <em>result</em> of<br>
the Control's element having received<br>
or lost keyboard focus, rather than <em>causing</em> the element<br>
to acquire or lose focus. Calling setFocused(true)<br>
doesn't guarantee that the Control's<br>
key event target has keyboard focus, only that it is styled as<br>
such.</dd><br>

<dt>setEnabled(enable)</dt><dd>Enables or disables<br>
the Control (that is, modifies<br>
its goog.ui.Component.State.DISABLED state property). If<br>
the state change is successful,<br>
the goog.ui.Component.State.DISABLED state property will<br>
be set to the <em>negation</em> of enable (The semantics<br>
of the DISABLED state flag differs slightly from that of other state<br>
flags: while with most state flags we apply extra CSS styling when<br>
a Control enters a state, we actually<br>
apply a special "disabled" style when the component leaves the enabled<br>
state, and remove the "disabled" style when the component enters the<br>
enabled state.)</dd><br>

<dt>setSelected(select)</dt><dd>Selects or deselects<br>
the Control (that is, modifies<br>
its goog.ui.Component.State.SELECTED state<br>
property). Note that goog.ui.Component.State.SELECTED is<br>
not a supported state by default. It might be supported in<br>
a Control such as an item in a list of<br>
selectable items (a "multi-select" list).</dd><br>

<dt>setChecked(check)</dt><dd>Checks or unchecks<br>
the Control (that is, modifies<br>
its goog.ui.Component.State.CHECKED state property). Note<br>
that goog.ui.Component.State.CHECKED is not a supported<br>
state by default. It might be supported in<br>
a Control such as a toggle<br>
button.</dd><br>

<dt>setOpen(open)</dt><dd>Opens (expands) or closes<br>
(collapses) the Control (that is,<br>
modifies its goog.ui.Component.State.OPENED state<br>
property). Note that goog.ui.Component.State.OPENED is<br>
not a supported state by default. It might be supported in<br>
a Control such as a tree view in which<br>
items can be expanded or collapsed.</dd><br>

<h4>Other Configuration Methods</h4>

<dt>setRenderer(renderer)</dt><dd>Sets the renderer used<br>
to control the appearance of<br>
the Control. Changing the renderer<br>
after the Control has decorated its<br>
element or has been rendered produces an error.</dd><br>

<dt>addClassName(className)</dt><dd>Adds the given CSS<br>
class name to the list of classes to be applied to<br>
the Control's root element. This method<br>
is useful when you create a number<br>
of Controls, and you want to add some<br>
additional styling to just one of them (to make its text content bold,<br>
for example). addClassName(className) adds the extra<br>
styling but otherwise leaves the behavior and DOM structure of the<br>
control intact.</dd><br>

<dt>removeClassName(className)</dt><dd>Removes the given<br>
CSS class name from the list of classes to be applied to<br>
the Control's root element.</dd><br>

<dt>setContent(content)</dt><dd>Sets<br>
the Control's content<br>
to content. The content parameter may be a<br>
string, an element, or an array of<br>
nodes. See <a href='#def3-3'>Content</a> above.</dd><br>

<dt>setCaption(caption)</dt><dd>An alias<br>
for setContent() that takes a string parameter.</dd><br>

<dt>setVisible(visible, opt_force)</dt><dd>Shows<br>
the Control if visible<br>
is true and hides it if visible<br>
is false. Does nothing if<br>
the Control already has the requested<br>
visibility. Otherwise dispatches a SHOW or HIDE event. Event listener<br>
functions can cancel the visibility change by returning<br>
false. If opt_force is true, doesn't check whether the<br>
component already has the requested visibility, and doesn't dispatch<br>
any events.</dd><br>

<dt>setState(state, enable)</dt><dd>Sets the state<br>
property indicated by state to the boolean value<br>
of enable, and updates<br>
the Control's styling accordingly. Does<br>
nothing if the Control is already in<br>
the correct state or if it doesn't support the specified<br>
state.</dd><br>

<dt>setSupportedState(state, support)</dt><dd>Enables<br>
(if support is true) or disables<br>
(if support is false) support for the state<br>
property indicated by state.  Changing the set of states<br>
supported by the Control after it has<br>
been rendered is an error. See <a href='#def2-1'>Supported States</a>
above.</dd><br>

<dt>setAutoStates(states, enable)</dt><dd>Enables<br>
(if enable is true) or disables<br>
(if enable is false) automatic event<br>
handling and default logic for the given<br>
state(s). See <a href='#ex3-3'>Controlling AutoStates</a>
above. </dd><br>

<dt>setDispatchTransitionEvents(states,<br>
enable)</dt><dd>Enables or disables transition events for the<br>
given state(s). Controls handle state transitions internally by<br>
default, and only dispatch state transition events if explicitly<br>
requested to do so by calling this<br>
method. See <a href='#ex3-2'>Controlling State Transition Events</a>
above.</dd><br>

<h3>Control Accessor Methods</h3>
<dl>
<dt>canDecorate(element)</dt><dd>Returns true<br>
if element can be decorated by<br>
this Control.</dd><br>

<dt>getCaption()</dt><dd>Returns the text caption of<br>
the Control. If<br>
the Control's content is a string,<br>
equivalent to getContent()</dd><br>

<dt>getContent()</dt><dd>Returns the text caption or DOM<br>
structure displayed in<br>
the Control.</dd><br>

<dt>getContentElement()</dt><dd>Returns the DOM element<br>
into which child components are to be rendered, or null<br>
if the Control hasn't been rendered<br>
yet.</dd><br>

<dt>getExtraClassNames()</dt><dd>Returns any additional<br>
class names to be applied to the component's root element, or null if<br>
no extra class names have been added.</dd><br>

<dt>getHandler()</dt><dd>Returns the event handler for<br>
this component. Lazily created the first time this method is<br>
called.</dd><br>

<dt>getKeyEventTarget()</dt><dd>Returns the DOM element<br>
on which the Control is listening for<br>
keyboard events (null if none).</dd><br>

<dt>getKeyHandler()</dt><dd>Returns the keyboard event<br>
handler for this component. Lazily created the first time this method<br>
is called.</dd><br>

<dt>getRenderer()</dt><dd>Returns the renderer used by<br>
this Control to create its DOM<br>
structure or to decorate an existing element.</dd><br>

<dt>getState()</dt><dd>Returns the component's state as a<br>
bitwise combination<br>
of goog.ui.Component.State<br>
constants.</dd><br>

<dt>hasState(state)</dt>
<blockquote><dd>Returns true if the Control is in<br>
the specified state, false otherwise. The state<br>
parameter must be one of the state constants<br>
in goog.ui.Component.State.</dd><br></blockquote>

<dt>isActive()</dt><dd>Returns true if<br>
the Control is active (pressed), false<br>
otherwise.</dd><br>

<dt>isAutoState(state)</dt>
<dd>Returns true if the Control<br>
provides default event handling for state, false<br>
otherwise. The state parameter must be one of the state<br>
constants<br>
in goog.ui.Component.State.</dd><br>

<dt>isChecked()</dt><dd>Returns true if<br>
the Control is checked, false<br>
otherwise.</dd><br>

<dt>isDispatchTransitionEvents(state)</dt><dd>Returns<br>
true if the Control is set to dispatch<br>
transition events for the given state, false otherwise.</dd><br>

<dt>isEnabled()</dt><dd>Returns true if<br>
the Control is enabled, false<br>
otherwise.<br>
</dd><br>

<dt>isFocused()</dt><dd>Returns true if<br>
the Control is styled to indicate that<br>
it has keyboard focus, false otherwise.</dd><br>

<dt>isHighlighted()</dt><dd>Returns true if<br>
the Control is currently highlighted,<br>
false otherwise.</dd><br>

<dt>isOpen()</dt><dd>Returns true if<br>
the Control is open (expanded), false<br>
otherwise.</dd><br>

<dt>isSelected()</dt><dd>Returns true if<br>
the Control is selected, false<br>
otherwise.</dd><br>

<dt>isSupportedState(state)</dt><dd>Returns true if<br>
the Control<br>
supports state, false otherwise. The state<br>
parameter must be one of the state constants<br>
in goog.ui.Component.State.</dd><br>

<dt>isTransitionAllowed(state, enable)</dt><dd>Returns<br>
true if the transition into or out of the given state is allowed to<br>
proceed, false otherwise. Also dispatches a state transition event if<br>
the Control is configured to dispatch<br>
transitions for state. The enable parameter<br>
indicates whether we are checking a transition into or out<br>
of state. A state transition is allowed if all of the<br>
following conditions are true:<br>
<br>
<b>the Control supports the state,<br></b>the Control isn't already in the target state, and<br>
<b>either the Control is configured not to dispatch events for this state transition, or the transition event dispatched by isTransitionAllowed() is not canceled by any event listener.</b>

This method is "protected" in the sense that it is primarily for use<br>
within the class. It is called by all state configuration<br>
methods</dd><br>

<dt>isVisible(opt_noCheckParent)</dt><dd>Returns true if<br>
the Control's visibility is set to<br>
visible, false if it is set to hidden.</dd><br>


<h3>goog.ui.registry Functions</h3>
<dl>
<dt>setDecoratorByClassName(className,<br>
decoratorFunction)</dt><dd>Associates a CSS class name with a<br>
factory function that returns a new instance of<br>
a Control or<br>
a Control subclass. Subsequent calls<br>
to goog.ui.registry.getDecoratorByClassName(element) will<br>
return the instance produced by decoratorFunction<br>
if element has className as one of its CSS<br>
classes. Calls to goog.ui.decorate() will also use<br>
decorator instances produced<br>
by decoratorFunction.</dd><br>

<dt>getDecoratorByClassName(className)</dt><dd>Returns the Control<br>
instance created by the decorator factory function registered for the<br>
given CSS class name, or null if no decorator factory function is<br>
found.</dd><br>

<dt>getDecorator()</dt><dd>Returns an instance<br>
of Control or a subclass suitable to decorate the given<br>
element, based on its CSS class.</dd><br>

<dt>reset()</dt><dd>Resets the global renderer registry.</dd>


<h2>Event Types</h2>
The Control class has built-in<br>
support for dispatching the following types of events:<br>
<br>
<table><thead><th>Event Type</th><th>Associated State</th><th>Associated State Supported by Default</th><th>Dispatched by Default</th></thead><tbody>
<tr><td><code>goog.ui.Component.EventType.SHOW</code></td><td>None            </td><td>NA                                   </td><td>Yes                  </td></tr>
<tr><td><code>goog.ui.Component.EventType.HIDE</code></td><td>None            </td><td>NA                                   </td><td>Yes                  </td></tr>
<tr><td><code>goog.ui.Component.EventType.ENTER</code></td><td>None            </td><td>NA                                   </td><td>Yes                  </td></tr>
<tr><td><code>goog.ui.Component.EventType.LEAVE</code></td><td>None            </td><td>NA                                   </td><td>Yes                  </td></tr>
<tr><td><code>goog.ui.Component.EventType.ACTION</code></td><td>None            </td><td>NA                                   </td><td>Yes                  </td></tr>
<tr><td><code>goog.ui.Component.EventType.DISABLE</code></td><td><code>goog.ui.Component.State.DISABLED</code></td><td>Yes                                  </td><td>No                   </td></tr>
<tr><td><code>goog.ui.Component.EventType.ENABLE</code></td><td><code>goog.ui.Component.State.DISABLED</code></td><td>Yes                                  </td><td>No                   </td></tr>
<tr><td><code>goog.ui.Component.EventType.HIGHLIGHT</code></td><td><code>goog.ui.Component.State.HOVER</code></td><td>Yes                                  </td><td>No                   </td></tr>
<tr><td><code>goog.ui.Component.EventType.UNHIGHLIGHT</code></td><td><code>goog.ui.Component.State.HOVER</code></td><td>Yes                                  </td><td>No                   </td></tr>
<tr><td><code>goog.ui.Component.EventType.ACTIVATE</code></td><td><code>goog.ui.Component.State.ACTIVE</code></td><td>Yes                                  </td><td>No                   </td></tr>
<tr><td><code>goog.ui.Component.EventType.DEACTIVATE</code></td><td><code>goog.ui.Component.State.ACTIVE</code></td><td>Yes                                  </td><td>No                   </td></tr>
<tr><td><code>goog.ui.Component.EventType.FOCUS</code></td><td><code>goog.ui.Component.State.FOCUSED</code></td><td>Yes                                  </td><td>No                   </td></tr>
<tr><td><code>goog.ui.Component.EventType.BLUR</code></td><td><code>goog.ui.Component.State.FOCUSED</code></td><td>Yes                                  </td><td>No                   </td></tr>
<tr><td><code>goog.ui.Component.EventType.SELECT</code></td><td><code>goog.ui.Component.State.SELECTED</code></td><td>No                                   </td><td>No                   </td></tr>
<tr><td><code>goog.ui.Component.EventType.UNSELECT</code></td><td><code>goog.ui.Component.State.SELECTED</code></td><td>No                                   </td><td>No                   </td></tr>
<tr><td><code>goog.ui.Component.EventType.CHECK</code></td><td><code>goog.ui.Component.State.CHECKED</code></td><td>No                                   </td><td>No                   </td></tr>
<tr><td><code>goog.ui.Component.EventType.UNCHECK</code></td><td><code>goog.ui.Component.State.CHECKED</code></td><td>No                                   </td><td>No                   </td></tr>
<tr><td><code>goog.ui.Component.EventType.OPEN</code></td><td><code>goog.ui.Component.State.OPENED</code></td><td>No                                   </td><td>No                   </td></tr>
<tr><td><code>goog.ui.Component.EventType.CLOSE</code></td><td><code>goog.ui.Component.State.OPENED</code></td><td>No                                   </td><td>No                   </td></tr></tbody></table>

<h2>Styles</h2>
The Control class' default renderer<br>
automatically toggles the following CSS classes on<br>
a Control's root element:<br>
<br>
<table><thead><th>Classname</th><th>Added When:</th><th>Interpreted in Static HTML?</th></thead><tbody>
<tr><td>goog-control</td><td>Element is created or decorated</td><td>By getDecorator()          </td></tr>
<tr><td>goog-control-rtl</td><td>enterDocument() is executed, if the element has a 'direction' style  attribute of 'rtl' indicating a right-to-left language</td><td>No                         </td></tr>
<tr><td>goog-control-disabled</td><td>Element becomes disabled</td><td>When element is decorated, to set state.</td></tr>
<tr><td>goog-control-hover</td><td>Element is highlighted</td><td>When element is decorated, to set state</td></tr>
<tr><td>goog-control-active</td><td>Element becomes active</td><td>When element is decorated, to set state</td></tr>
<tr><td>goog-control-selected</td><td>Element is selected</td><td>When element is decorated, to set state</td></tr>
<tr><td>goog-control-checked</td><td>Element is checked</td><td>When element is decorated, to set state</td></tr>
<tr><td>goog-control-focused</td><td>Element gets focus</td><td>When element is decorated, to set state</td></tr>
<tr><td>goog-control-open</td><td>Element is opened</td><td>When element is decorated, to set state</td></tr></tbody></table>

<h2>Reference Links</h2>

<a href='http://docs.closure-library.googlecode.com/git/class_goog_ui_Control.html'>goog.ui.Control</a>