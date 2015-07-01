# Preview Warning #

This preview document is a rough draft. It may contain incorrect or outdated information, so use at your own risk.

When this doc is published at http://code.google.com/closure/library/ , it will be deleted from this wiki and replaced with a link to the polished version. Wiki edits to this preview version may be incorporated into the polished version, or they may be lost. Edit at your own risk.

We welcome feedback! Use the comments field or closure-library-discuss@googlegroups.com to give feedback.

# Closure Cookbook: Adding Drag-and-Drop Behavior to Your Application #
## Overview ##

Creating drag-and-drop functionality for your web application
ordinarily involves detailed state management and the handling of many
different browser event types, as well as the implementation of the
dragging "animation" itself.

The Closure Library's **DragDrop** and **DragDropGroup** classes, found in
the fx package, abstract the gory details of drag-and-drop-relevant
browser events into a manageable set of
standardized event types.

By attaching listeners for these events you can
customize drag-and-drop behavior and hook it into your application's
logic. In addition, the Closure Library's drag-and-drop facilities
handle much of the bookkeeping involved in implementing this complex
behavior, and they handle updating the display of the dragged item as
it is being dragged.

## Concepts and Definitions ##

The method names in the drag-and-drop classes use some terminology
that overlaps with event model terminology. In the Closure Library's
drag-and-drop model, the terms "source", "target," and "group" have
specialized meanings that are worth defining from the outset:

### Target ###
An object on which the dragged element can be
dropped to trigger an effect of some sort. A target is represented by
a **goog.fx.DragDrop**
or **goog.fx.DragDropGroup** object.

### Source ###

A draggable object. A source is also
represented by a **goog.fx.DragDrop**
or **goog.fx.DragDropGroup**
object. A **goog.fx.DragDrop**
or **goog.fx.DragDropGroup** object becomes a source
when it has a valid target defined for it through a call to the
object's addTarget method. For example:
```
var source = new goog.fx.DragDrop('idOfElement', someData);
source.addTarget(target);
```

Calling addTarget(target) also
identifies `target` as a target. Note that drag-and-drop
sources and targets are event targets in the sense that they can
receive browser events or other events.

### Group ###
A set of sources and/or targets that share
common behaviors. A group is represented by
a **goog.fx.DragDropGroup** object. Elements in
a **DragDropGroup** will use the same set of
drag-and-drop event handlers and, if
the **DragDropGroup** is a source, the same set of
allowed targets. Items in the same **DragDropGroup**
are **not** dragged together as a unit. Group membership does not
physically group elements; rather, it applies the same logic in bulk
to a set of objects.

### Item ###
A single Element (or, optionally, a pairing of an
Element with string data) that can be dropped upon or dragged. An item
is represented by a **goog.fx.DragItem**
object. A **DragDrop** object has a
single **DragItem**, while
a **DragDropGroup** object can have any number
of **DragItems**.

## Initializing **DragDrop** Objects ##

To keep things simple, assume for the moment that you're only
using **DragDrop** objects (that is, you are not using
any **DragDropGroups**). To prepare
your **DragDrop** objects for use, perform the
following steps:
<ol>
<li>For each draggable source element or drop target element, call<br>
the <b>DragDrop</b> constructor like so:<br>
<pre><code>var source = new goog.fx.DragDrop(someElement, someData);<br>
</code></pre>
or<br>
<pre><code>var source = new goog.fx.DragDrop('idOfSomeElement', someData);<br>
</code></pre>
where <code>someElement</code> is a DOM <b>Element</b> that<br>
you want either to be draggable or to serve as a target on which some<br>
draggable element can be dropped, and where <code>someData</code> is an<br>
optional object that will be recorded in the object to be used however<br>
you wish.<br>
</li>

<li>
Define at least one drop target for<br>
every <b>DragDrop</b> object representing a draggable<br>
element by calling addTarget on the object:<br>
<br>
<pre><code>source.addTarget(someOtherDragDrop);<br>
</code></pre>

where <code>someOtherDragDrop</code> is<br>
another <b>DragDrop</b>
or <b>DragDropGroup</b>
object. A <b>DragDrop</b> can have any number of<br>
targets. The same <b>DragDrop</b> object can also be both<br>
a draggable source and a drop target. In other words, you can<br>
call addTarget on the same <b>DragDrop</b>
object that elsewhere serves as the argument<br>
to addTarget.  A <b>DragDrop</b> on<br>
which addTarget has been called will be flagged as a<br>
source and initialized appropriately, and<br>
a <b>DragDrop</b> which has served as an argument<br>
to addTarget will be identified as a target and<br>
initialized appropriately. Every <b>DragDrop</b> should<br>
either have at least one target or serve as a target for at least one<br>
other <b>DragDrop</b>: otherwise constructing the object<br>
will have no effect.<br>
<br>
If you wish to define special CSS classes for the source and/or target<br>
elements of a newly constructed object (to set the cursor to a grabby<br>
hand for draggable elements, for example)<br>
call setSourceClass()<br>
and/or setTargetClass():<br>
<br>
<pre><code>target.setTargetClass('target');<br>
source.setSourceClass('source');<br>
</code></pre>
</li>
<li>
Call init on all new <b>DragDrop</b> objects:<br>
<pre><code>source.init();<br>
</code></pre>
The init method adds the browser event listeners and CSS<br>
classes appropriate to the <b>DragDrop</b>'s role as a<br>
source and/or target.<br>
</li>
</ol>

You must perform the above initialization steps in this order. Calls
to addTarget not only add to the list of targets for
a **DragDrop**, but also determine whether
a **DragDrop** is identified as a target and whether
it is identified as a source. Whether a **DragDrop**
is a target and whether it is a source in turn affects the kind of
initialization performed on the **DragDrop** by
the init method. In particular, the low-level event
listeners used to detect dragging are only attached
to **DragDrop** objects that are identified as
sources.

At any time after the **DragDrop** objects are
created, you can attach listeners for high-level drag drop events, as
described in "Attaching Event Handlers" below.

See the [drag-and-drop demo](https://code.google.com/p/closure-library/source/browse/closure/goog/demos/dragdrop.html) in the the Closure Library demos directory for an example initialization
procedure.

## Initializing <b>DragDropGroup</b> Objects ##

You create a **DragDropGroup** in exactly the same
manner as a **DragDrop**, with one exception:
the **DragDropGroup** does not take
an **Element** (or any) parameter. Instead, you first
construct an empty **DragDropGroup** object, and then
you add **Elements** to that group individually with
calls to its addItem method:
```
var group = new goog.fx.DragDropGroup();
group.addItem(someElement, 'Some data');
group.addItem(someOtherElement, 'Some more data');
```
or
```
var group = new goog.fx.DragDropGroup();
group.addItem('idOfSomeElement', 'Some data');
group.addItem('idOfSomeOtherElement', 'Some more data');
```

Except for this one difference, **DragDropGroup**
objects are used in the same way as **DragDrop**
objects. If a **DragDropGroup** is a source, all of
its items will be draggable. If it is added as a target to a
source **DragDrop**
or **DragDropGroup**, all of its items will serve as
drop targets for that source.

The [drag-and-drop demo](http://code.google.com/p/closure-library/source/browse/#git/closure/goog/demos/dragdrop.html)
in the the Closure Library demos directory gives examples of
initializing **DragDropGroups**.

## Attaching Event Handlers ##

Both **DragDrop**
and **DragDropGroup** inherit
from **goog.events.EventTarget**
(via **fx.AbstractDragDrop**). Event listeners can
therefore be attached to them with `goog.events.listen`
(see the [Event Handling](http://code.google.com/closure/library/docs/events_tutorial.html) tutorial for an introduction
to `goog.events.listen`). As a result, you don't need to
worry about dealing with native browser events for your source and
target objects. Calling `init()` on
a **DragDrop** or **DragDropGroup**
object automatically creates event listeners for low-level browser
events like mousedown. These listeners will intercept the
browser events, interpret them in the context of the current drag
state, and fire the more abstract event types defined
in `goog.fx.AbstractDragDrop.EventType`.

To implement drag-and-drop rollovers, drop-triggered procedures, and
the other aspects of the drag-and-drop behavior, implement and attach
listeners for these event types:

**`goog.fx.AbstractDragDrop.EventType.DRAGSTART` is dispatched to
the source when the dragging is initiated. This event is
fired**before**the clone of the source is created (see
below)**`goog.fx.AbstractDragDrop.EventType.DRAGEND` is
dispatched to the source when the user releases the mouse button. If
the drag ends with the mouse pointer over a valid target, a `DROP` event
will be dispatched to the target and a DRAG event will be dispatched
to the source.
**`goog.fx.AbstractDragDrop.EventType.DRAGOVER` is
dispatched both to the target and to the source when the mouse pointer
enters a valid drop target during a drag.**`goog.fx.AbstractDragDrop.EventType.DRAGOUT` is
dispatched both to the target and to the source when the mouse pointer
leaves a valid target during a drag
**`goog.fx.AbstractDragDrop.EventType.DRAG` is dispatched
to the source when the drag ends (i.e., when the user releases the
mouse button)**`goog.fx.AbstractDragDrop.EventType.DROP` is dispatched
to the target if the mouse pointer is over a valid target when the
drag ends

A clone of the dragged item is automatically created when the user
starts dragging a draggable element, and the clone follows the mouse
until the user lets go. The dragging animation is handled
automatically by the Closure Library: you don't need to implement handlers to
enable this animation. The clone is instantiated after the "dragstart"
event is fired, so any changes to the appearance of the dragged item
in a "dragstart" handler will appear in both the dragged clone and the
source element.

The event objects dispatched by **DragDrop**
and **DragDropGroup** objects are instances
of `goog.fx.DragDropEvent`. A `DragDropEvent`
object will always have values for the first three fields listed
below -- `type`, `dragSource`,
and `dragSourceItem`. Events of type "dragover" and "dragout"
include the "target" parameters
(`dropTarget`, `dropTargetItem`,
and `dropTargetElement`), while "drop" and "drag" events will
include both these target parameters and specific position information
for the event.

<ul>
<li><var>type</var>: The type of the event. One of the six types<br>
defined<br>
in <b><code>goog.fx.AbstractDragDrop.EventType</code></b>.</li>
<li><var><code>dragSource</code></var>: <b>DragDrop</b>
or <b>DragDropGroup</b> object associated with the<br>
element being dragged.</li>
<li><var>dragSourceItem</var>: The particular item being dragged: one<br>
of the <b>DragItem</b> objects belonging to the<br>
dragSource.</li>
<li><var>dropTarget</var> (optional): <b>DragDrop</b>
or <b>DragDropGroup</b> object associated with the<br>
element on which the dragged element has been dropped.</li>
<li><var>dropTargetItem</var> (optional): The particular item on which<br>
the dragged element was dropped: one of<br>
the <b>DragItem</b> objects belonging to dropTarget.</li>
<li><var>dropTargetElement</var> (optional): The particular element on<br>
which the dragged element was dropped.</li>
<li><var>clientX</var> (optional): X-Position of the mouse pointer<br>
relative to the screen.</li>
<li><var>clientY</var> (optional): Y-Position of the mouse pointer<br>
relative to the screen.</li>
<li><var>viewportX</var> (optional): X-Position of the mouse pointer<br>
relative to the viewport.</li>
<li><var>viewportY</var> (optional): Y-Position of the mouse pointer<br>
relative to the viewport.</li>
</ul>


Unknown end tag for &lt;/p&gt;


<p>
To trigger an action when an element is dragged onto one of its<br>
possible targets, then, listen for "drop" events on the<br>
target <b>DragDrop</b>
or <b>DragDropGroup</b> objects. Implement visual effects<br>
by listening for other events on sources and targets.<br>
<br>
The <a href='http://code.google.com/p/closure-library/source/browse/#git/closure/goog/demos/dragdrop.html'>drag-and-drop<br>
demo</a> in the demos directory illustrates some of the<br>
possibilities. The following snippet from the demo contains the event<br>
handler code for a source <b>DragDropGroup</b>
(<var>list2</var>) and a target <b>DragDrop</b>
(<var>button2</var>):<br>
<pre>
goog.events.listen(list2, 'dragstart', dragStart);<br>
goog.events.listen(list2, 'dragend', dragEnd);<br>
<br>
...<br>
<br>
goog.events.listen(button2, 'dragover', dragOver);<br>
goog.events.listen(button2, 'dragout', dragOut);<br>
goog.events.listen(button2, 'drop', drop);<br>
<br>
...<br>
<br>
function dragOver(event) {<br>
event.dropTargetItem.element.style.background = 'red';<br>
}<br>
<br>
function dragOut(event) {<br>
event.dropTargetItem.element.style.background = 'silver';<br>
}<br>
<br>
function drop(event) {<br>
event.dropTargetItem.element.style.background = 'silver';<br>
var str = [<br>
event.dragSourceItem.data,<br>
' dropped onto ',<br>
event.dropTargetItem.data,<br>
' at ',<br>
event.viewportX,<br>
'x',<br>
event.viewportY<br>
];<br>
alert(str.join(''));<br>
}<br>
<br>
...<br>
<br>
function dragStart(event) {<br>
goog.style.setOpacity(event.dragSourceItem.element, 0.5);<br>
}<br>
<br>
function dragEnd(event) {<br>
goog.style.setOpacity(event.dragSourceItem.element, 1.0);<br>
}<br>
</pre>
</p>
<h2>Reference Links</h2>

<li><a href='http://docs.closure-library.googlecode.com/git/class_goog_fx_AbstractDragDrop.html'>goog.fx.AbstractDragDrop<br>
API Reference</a>

<li><a href='http://docs.closure-library.googlecode.com/git/class_goog_fx_DragDrop.html'>goog.fx.DragDrop<br>
API Reference</a>

<li><a href='http://docs.closure-library.googlecode.com/git/class_goog_fx_AbstractDragDrop.html'>goog.fx.AbstractDragDrop<br>
API Reference</a>

<li> <a href='http://docs.closure-library.googlecode.com/git/class_goog_fx_DragDropEvent.html'>goog.fx.DragDropEvent<br>
API Reference</a>

<li> <a href='http://docs.closure-library.googlecode.com/git/class_goog_fx_DragDropGroup.html'>goog.fx.DragDropGroup<br>
API Reference</a>

<li><a href='http://docs.closure-library.googlecode.com/git/class_goog_fx_DragDropItem.html'>goog.fx.DragDropItem API Reference</a>