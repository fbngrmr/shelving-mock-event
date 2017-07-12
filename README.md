# Event mock

[![Build Status](https://travis-ci.org/dhoulb/mock-event.svg?branch=master)](https://travis-ci.org/dhoulb/mock-event)

Fully unit tested mock implementation of the browser Event API. Conforms as closely as possible to [https://developer.mozilla.org/en-US/docs/Web/API/Event](Event).

Mocked Event and EventTarget classes that conform to browser's Event API. Can be used to simulate events in server-side code for testing, or in other places where browser APIs are not available. 

Supports the following functionality:

- Event listeners, e.g. `addEventListener('click', () => {})` and `removeEventListener()`
- Event handlers, e.g. `target.onclick = () => {}`
- Event dispatch, e.g. `dispatchEvent(new Event('click', { bubbles: true }))`
- Parent heirarchy with bubbling (and capturing) like DOM elements
- Stopping propagation with `stopPropagation()` and `stopImmediatePropagation()`
- Preventing default with `preventDefault()`

## Examples

### Create an event target and dispatch events on it

This full example shows how you would create an `EventTarget` (with a parent `EventTarget` that would also receive bubbled events), and dispatch an `Event` on that target:

```js
import { Event, EventTarget } from 'shelving-mock-event';

// Create a parent target.
const parent = new EventTarget();

// Create an target and pass in the parent target.
// Enable an `onclick` event handler for the target.
const target = new EventTarget(parent, ['click']); 

// Attach event listeners/handlers.
// As-per the event capturing/bubbling model, they will be called in this order.

	// Add event listener on parent.
	// Will be called during capturing phase, i.e. before listeners/handlers on the target.
	parent.addEventListener('click', () => { console.log('Called 1'); }, true);

	// Add event handler on target.
	target.onclick = (e) => { console.log('Called 2'); }
	
	// Add event listener on target.
	target.addEventListener('click', () => { console.log('Called 3'); });
	
	// Add event listener on parent target.
	// Will be called during bubbling phase, i.e. after listeners/handlers on the target.
	parentTarget.addEventListener('click', () => { console.log('Called 5'); });

// Fire the event on the target.
target.dispatchEvent(new Event('click', { bubbles: true }));
```

### Creating a custom event targets

`EventTarget` is not normally used directly, but is implemented by a specific class. Javascript ES6 classes make this easy as you can simply extend `EventTarget`:

```js
// Create a custom Javascript class.
class MyThing extends EventTarget
{
	// Construct.
	constructor(parent)
	{
		// Call super() to initialise this.
		// Pass through any parent element.
		// Initialise `onclick` and `onboom` properties.
		super(parent, ['click', 'boom']);
	}
}

// Make a new MyThing.
const thing = new MyThing();

// Add a handler on the thing.
thing.onboom = () => { console.log('Called handler!'); }

// Add a listener on the thing.
thing.addEventListener('boom', () => { console.log('Called listener!'); });

// Dispatch event.
thing.dispatchEvent(new Event('boom'));
```

## API

### `Event`

```typescript
event = new Event(type: string, { bubbles = false, cancelable = false })
```

Instantiate Events and dispatch them on EventTargets using `target.dispatchEvent(new Event('click'))`. Conforms to the [Event](https://developer.mozilla.org/en-US/docs/Web/API/Event) interface.


- `type` (string)  

	The name of the event, e.g. `'click'`, `'blur'` or `'my-custom-event'`

- `bubbles` (boolean)  

	Whether the event should bubble up to the parent EventTarget when dispatched. The default is `false`

- `cancellable` (boolean)  

	Whether the event can be cancelled with `event.preventDefault()`. The default is `false`

#### Properties

- `event.type` (string) (read only)  

	The name of the event, e.g. `'click'`, `'blur'` or `'my-custom-event'` (matches what was passed in to the constructor.

- `event.cancellable` (boolean) (read only)  

	Whether this event can be cancelled or not (matches what was passed in to the constructor).

- `event.bubbles` (boolean) (read only)  

	Whether this event will bubble or not (matches what was passed in to the constructor).

- `event.target` (boolean) (read only)  

	A reference to the original `EventTarget` that `dispatchEvent()` was called on.

- `event.currentTarget` (boolean) (read only)  

	A reference to the current `EventTarget` that this event is firing on.

- `event.eventPhase` (constant) (read only)  

	The current phase of the event. One of: `Event.NONE`, `Event.CAPTURING_PHASE`, `Event.AT_TARGET`, `Event.BUBBLING_PHASE`

- `event.timeStamp` (number) (read only)  

	The time this event was created, in milliseconds.

- `event.defaultPrevented` (boolean) (read only)  

	Whether or not event.preventDefault() has been called on the event.

#### Methods

- `event.preventDefault()`  

	Cancels the event (if it is cancelable).

- `event.stopPropagation()`  

	Stops the propagation (bubbling or capturing) of events on targets further along in the event dispatch order.

- `event.stopImmediatePropagation()`  

	Stops the propagation (bubbling or capturing) of events on targets further along in the event dispatch order _and_ any further events on the current target.

### `EventTarget`

```typescript
target = new EventTarget(parent?: EventTarget, handlers?: [string])
```

An object which can have events dispatched on it using `target.dispatchEvent(event)`. Conforms to the [EventTarget](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget) interface.

- `parent` (EventTarget)  

	The parent element for this element that events will bubble up to (if specified)

- `handlers` (array of strings)  

	Allow event handlers on this EventTarget (e.g. `el.onclick = function(){}`). Handlers that are not explicitly named in the constructor will not be called when an event is dispatched. Handlers can only be functions.

#### Properties

- `target.onclick` (function)

	Set this property to a `function` and that function will be called when an event is dispatched with e.g. `target.dispatchEvent(new Event('click'))`. Any event can have a handler, but they must be explicitly specified in the constructor. If handler is not `function` or `undefined` then `TypeError` will be thrown.

#### Methods

```typescript
target.addEventListener(type: string, callback: function, capturing = false)
```

Add an event listener to an `EventTarget`. Event listeners are callback functions that are called when the named event is dispatched on that `EventTarget` (or is dipatched on one of its children and bubbles up to the target).

- `type` (string)  

	The name of the event to listen for, e.g. `'click'`, `'blur'` or `'my-custom-event'`
 
- `callback` (function)  

	The callback function that gets called when `type` events are dispatched on this target. The callback function will receive the following arguments:
	
	- `event`: the `Event` object that was dispatched on the event target.

- `capturing` (boolean)  

	Whether the listener should be attached to the capturing phase (before listeners on the target) or the bubbling phase (after listeners on the target). The default is `false` (bubbling phase).

```typescript
target.removeEventListener(type: string, callback: function, capturing = false)
```

Remove a specific event listener from an `EventTarget`.

- `type` (string)  

	The name of the event you wish to stop listening for, e.g. `'click'`, `'blur'` or `'my-custom-event'`

- `callback` (function)  

	The callback function you wish to remove. Must be a reference to the same callback function that was added with `addEventListener()`

- `capturing` (boolean)  

	Must match the `capturing` setting that was used when the event was added with `addEventListener()`