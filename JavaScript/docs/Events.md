# Javascript Events

## What is Event?

An event is a notification that something happened in the browser.

Example:

- User clicks a button
- User type in an input box
- Page finishes loading
- Dom Element Loaded
- Network request completes
- An element become visible

Event allows Javascript to react to user actions or system actions.

## Event Lifecycle

For any DOM event (like "click", "scroll"...)

Event Lifecycle Steps:

1. **Event creation inside the browser**
2. **Determine the event target**: The exact DOM element where the event triggered.
3. **Capturing phase** (top -> target)
4. **Target Phase** (runs the event listener at the target)
5. **Bubbling Phase** (target -> top)
6. **Event propagation ends**

This determines the order in which event listener are executed.

## What is Event Capturing?

In event lifecycle fist event is being created and then window determines the target of that event, so the event travels from root element (window) down to target element.

```txt
window -> document -> html -> body -> parent -> child -> event target
```

**Capturing listener runs before bubbling listener**, but only if you explicitly enable capturing.

### How can we enable event capturing?

- To enable event capturing you have to pass true as third parameter in the `addEventListener(event, handler, true)`.
- Now when you enable event capturing in that case the events parent can handle that event for target too.

```js
<div id="parent">
  <button id="child">Click Me</button>
</div>

<script>
document.getElementById("parent").addEventListener("click", () => {
  console.log("Parent capturing");
}, true); // <-- capturing enabled

document.getElementById("child").addEventListener("click", () => {
  console.log("Child capturing");
}, true);
</script>
```

```txt
Parent capturing
Child Capturing
```

So what happens is when you enable event capturing then in that case event is still traveling toward the target and if parent has the event listener then it will be triggered as event will come to parent and then if child also has event listener then child's event listener will be triggered too but after parent because in capturing phase event is traveling from top to bottom.

## What is Event Bubbling?

Event Bubbling is the default event flow.

```txt
event target -> parent -> body -> html -> document -> window
```

So event created, Propagated from window to target, and then again that event will propagate from target to window and then on reaching window the event will be decomposed by browser.

Example:

```js
<div id="parent">
  <button id="child">Click</button>
</div>

<script>
document.getElementById("parent").addEventListener("click", () => {
  console.log("Parent bubbling");
});

document.getElementById("child").addEventListener("click", () => {
  console.log("Child bubbling");
});
</script>
```

Output:

```txt
Child bubbling
Parent bubbling
```

Now order is reversed since parent and child both has listener for that event both of their listener will be called but first child then parent because we are in bubbling phase.

## What is Event Delegation?

Event Delegation = Attach One event listener on a parent container.

So the core Idea of Event Delegation is that instead of handing same event of multiple child components attach one event listener to parent component and handle them for each of the child, and this is possible because event bubble up from child to parent.

Why to use event delegation?

- Improve performance (instead of adding many event listener we will have one event listener)
- Works for dynamic child elements
- Cleaner and maintainable code.

Example:

```js
<ul id="list">
  <li>Apple</li>
  <li>Orange</li>
  <li>Banana</li>
</ul>

<script>
document.getElementById("list").addEventListener("click", function (e) {
  if (e.target.tagName === "LI") {
    console.log("You clicked:", e.target.textContent);
  }
});
</script>
```

- Even a new `<li>` element are added the listener still works.

## What are Different types of events?

### User Interface Events

- `click`, `dblclick`
- `mousedown`, `mouseup`,`mousemove`, `mouseover`, `mouseout`
- `wheel`

### Keyboard Events

- `keydown`, `keyup`

### Form Events

- `input`, `change`, `submit`, `focus`, `hashchange`

### Touch Events (mobile)

- `touchstart`, `touchend`, `touchmove`

### Drag and Drop events

- `dragstart`, `dragover`, `drop`

### Network/ custom browser events

- `online`, `offline`, `readystatechange`

## How to Handle Events Efficiently?

1. Avoid adding to many listeners: use event delegation
2. debounce / throttle expensive events

   ```js
    function debounce(fn, delay) {
      let timer;
      return function (...args) {
        clearTimeout(timer);
        timer = setTimeout(() => fn.apply(this, args), delay);
      };
    }

    input.addEventListener("input", debounce(() => {
      console.log("Searching...");
    }, 500));
   ```

3. Remove listener if no longer needed.

   ```js
    button.addEventListener("click", handler);
    button.removeEventListener("click", handler);
   ```

4. Use Passive listener to improve scroll performance

    So what happens is when some events are high frequency events that for each event if handler is called sometime browser has to pause the event till handler finishes and that result to frame drops and bad ux.

    so to handle that we can make an event handler passive in that case browser will not wait for the handler execution it will continue.

    ```js
    document.addEventListener("scroll", () => {
        console.log("scrolling");
    }, { passive: true });
    ```

5. If you need to listen to event once the. you can use once listener

   ```js
    button.addEventListener("click", () => console.log("Clicked only once"), {
      once: true,
    });
   ```

## What are custom events?

A custom event is an event you create yourself.

Useful when component need to communicate without direct reference.

Create and Dispatch Custom Events

```js
<button id="btn">Fire Custom Event</button>

<script>
const customEvent = new CustomEvent("myCustomEvent", {
  detail: { message: "Hello from custom event!" },
});

document.addEventListener("myCustomEvent", (e) => {
  console.log("Received:", e.detail.message);
});

document.getElementById("btn").addEventListener("click", () => {
  document.dispatchEvent(customEvent);
});
</script>
```

Output:

```txt
Received: Hello from custom event!
```

## DOM Event control Methods

1. event.preventDefault()

   Prevent the browser's default action from happening.

   Examples:
   - Clicking a link -> navigates to a URL
   - Submitting a form -> reloads the page
   - Right-click -> Opens the context menu
   - Dragging -> starts drag operation
   - Keyboard shortcuts (CTRL + s)
  
   When to use
   - Custom form handling
   - Prevent page from reload
   - Prevent link navigation
   - Disable context menu show something els
   - Prevent checkbox from toggling temporarily
   - Custom drag & drop

   Example

   ```html
    <a href="https://google.com" id="link">Go to Google</a>

    <script>
    document.getElementById("link").addEventListener("click", function(event) {
      event.preventDefault();
      console.log("Navigation blocked!");
    });
    </script>
   ```

2. event.stopPropagation()

   Stops the event from continuing to bubble up to capture down the DOM tree.

   After this call:
   - No Parent elements will receive the event
   - But other listener on the same element will still run.
  
   When to use:
     - Prevent parent handler from triggering
     - Avoid double action in nested clickable elements
     - Control event flow manually.
  
   Example

    ```html
      <div id="parent">
        <button id="child">Click Me</button>
      </div>
  
      <script>
      parent.addEventListener("click", () => console.log("Parent clicked"));
      child.addEventListener("click", (e) => {
        e.stopPropagation();
        console.log("Child clicked only");
      });
      </script>
    ```
  
    Clicking the button print:
  
    ```txt
      Child clicked only
    ```

3. event.stopImmediatePropagation()

    It stops:
    1. Event propagation to parent
    2. And prevents any additional listener on the same element from running.

    stopImmediatePropagation() is stronger than stopPropagation() since it does not allow any handler to execute again.

    When to use:
    - You want Exactly One Listener to respond for just one time. 
    - You need to cancel other listeners on the same element.

    Example:

    ```js
    button.addEventListener("click", () => console.log("Listener 1"));
    button.addEventListener("click", () => console.log("Listener 2"));
    button.addEventListener("click", (e) => {
      e.stopImmediatePropagation();
      console.log("Listener 3 (only this runs)");
    });
    ```

    Output:

    ```txt
    Listener 3 (only this runs)
    ```
