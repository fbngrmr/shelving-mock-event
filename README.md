# Event mock

Unit-tested mock/polyfill for Event and EventTarget from the browser DOM.

Compatable with Event and EventTarget interfaces from the browser/DOM API, but can be used in server-side code on Node or other places where the browser APIs are not available. Supports the following functionality:

- EventTarget listener callbacks, e.g. `addEventListener('click', () => {})` and `removeEventListener()`
- EventTarget handler callbacks, e.g. `target.onclick = () => {}`
- Event dispatch, e.g. `el.dispatchEvent(new Event('click', { bubbles: true }))`
- Parent heirarchy with bubbling (and capturing) like DOM targets
- Stopping propagation, e.g. `event.stopPropagation()` and `event.stopImmediatePropagation()`
- Preventing default, e.g. `event.preventDefault()`

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
`EventTarget` is not usually used directly, but is implemented in specific class. Javascript ES6 classes make this easy as you can simply extend `EventTarget`:

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

### `Event` (class)
Instantiate Events and dispatch them on EventTargets using `target.dispatchEvent(new Event('click'))`. Conforms to the [https://developer.mozilla.org/en-US/docs/Web/API/Event](Event) interface.

`event = new Event(type: string, { bubbles = false, cancelable = false })`
Create a new event.

	`type` (string)
	The name of the event, e.g. `'click'`, `'blur'` or `'my-custom-event'`

	`bubbles` (boolean)
	Whether the event should bubble up to the parent EventTarget when dispatched. The default is `false`

	`cancellable` (boolean)
	Whether the event can be cancelled with `event.preventDefault()`. The default is `false`

#### Properties

`event.type` (string) (read only)
The name of the event, e.g. `'click'`, `'blur'` or `'my-custom-event'` (matches what was passed in to the constructor.

`event.cancellable` (boolean) (read only)
Whether this event can be cancelled or not (matches what was passed in to the constructor).

`event.bubbles` (boolean) (read only)
Whether this event will bubble or not (matches what was passed in to the constructor).

`event.target` (boolean) (read only)
A reference to the original `EventTarget` that `dispatchEvent()` was called on.

`event.currentTarget` (boolean) (read only)
A reference to the current `EventTarget` that this event is firing on.

`event.eventPhase` (constant) (read only)
The current phase of the event. One of: `Event.NONE`, `Event.CAPTURING_PHASE`, `Event.AT_TARGET`, `Event.BUBBLING_PHASE`

`event.timeStamp` (number) (read only)
The time this event was created, in milliseconds.

`event.defaultPrevented` (boolean) (read only)
Whether or not event.preventDefault() has been called on the event.

#### Methods

`event.preventDefault()`
Cancels the event (if it is cancelable).

`event.stopPropagation()`
Stops the propagation (bubbling or capturing) of events on targets further along in the event dispatch order.

`event.stopImmediatePropagation()`
Stops the propagation (bubbling or capturing) of events on targets further along in the event dispatch order _and_ any further events on the current target.

### `EventTarget` (class)
An object which can have `Event` instances dispatched on it using `target.dispatchEvent(event)`. Conforms to the [https://developer.mozilla.org/en-US/docs/Web/API/EventTarget](EventTarget) interface.

`target = new EventTarget(parent?: EventTarget, handlers?: [string])`
Create a new EventTarget.

	`parent` (EventTarget)
	The parent element for this element that events will bubble up to (if specified)

	`handlers` (array of strings)
	Allow event handlers on this EventTarget (e.g. `el.onclick = function(){}`). Handlers that are not explicitly named in the constructor will not be called when an event is dispatched. Handlers can only be functions.

#### Properties

`target.onclick` (function)
Set this property to a `function` and that function will be called when an event is dispatched with e.g. `target.dispatchEvent(new Event('click'))`. Any event can have a handler, but they must be explicitly specified in the constructor. If handler is not `function` or `undefined` then `TypeError` will be thrown.

#### Methods

`target.addEventListener(type: string, callback: function, capturing = false)`
Add an event listener to an `EventTarget`. Event listeners are callback functions that are called when the named event is dispatched on that `EventTarget` (or is dipatched on one of its children and bubbles up to the target).

	`type` (string)
	The name of the event to listen for, e.g. `'click'`, `'blur'` or `'my-custom-event'`

	`callback` (string)
	The callback function that gets called when `type` events are dispatched on this target. The callback function will receive the following arguments:

		`event`: the `Event` object that was dispatched on the event target.

	`capturing` (boolean)
	Whether the listener should be attached to the capturing phase (before listeners on the target) or the bubbling phase (after listeners on the target). The default is `false` (bubbling phase).

`target.removeEventListener(type: string, callback: function, capturing = false)`
Remove a specific event listener from an `EventTarget`.

	`type` (string)
	The name of the event you wish to stop listening for, e.g. `'click'`, `'blur'` or `'my-custom-event'`

	`callback` (string)
	The callback function you wish to remove. Must be a reference to the same callback function that was added with `addEventListener()`

	`capturing` (boolean)
	Must match the `capturing` setting that was used when the event was added with `addEventListener()`