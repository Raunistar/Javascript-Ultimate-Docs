# Event-Loop

## A. Fundamentals

### 1. What is Javascript Runtime Environment?

A Javascript Runtime Environment is the place where Javascript code runs and tools provided around the JS engine to make program useful.

>Javascript can not run alone.
>
>It needs a runtime environment.

- Execute code
- Handle I/O (network, files, browser)
- Interact with system or browser features.

> Javascript Engine !== Javascript runtime
>
> Runtime is bigger than the engine

#### Main Component of a JS Runtime

1. **Javascript Engine**: Executes the code.

   Example:
   - V8 -> Chrome, Node.js
   - SpiderMonkey -> Firefox
   - JavaScriptCore -> Safari

   The engine includes
   - Callstack
   - Memory Heap
   - Garbage Collector
   - JIT Compiler

2. **Web APIs (Browser Only)**: Provided by browser not Javascript.
   
   Example:
   - DOM
   - setTimeout/setInterval
   - fetch / XMLHttpRequest
   - Web Storage
   - WebSocket
   - Geolocation

   These API enable async behavior.

3. **Callback / Task Queue**: Where completed async callback wait before execution.
   
   Types:
   - **Macrotask Queue** (setTimeout, events)
   - **Microtask Queue** (Promises)

4. **Event Loop**: The coordinator
   
   It checks:
   1. Is Callstack empty?
   2. Are there any pending tasks?
   3. Which task should run next?

#### Runtime Examples

1. Browser Runtime
   
   ```txt
    Browser
     |-- JS Engine (V8)
     |-- Web APIs
     |-- Event Loop
     |-- Callback Queues
     |-- Rendering Engine
   ```

2. Node.js Runtime
   
   ```txt
    Node.js
     |-- V8 Engine
     |-- Libuv (Event loop + I/O)
     |-- Thread Pool
     |-- Node APIs (fs, path, os, net, timers ..)
   ```

#### Example:

```js
console.log("start");
setTimeout(()=>{
    console.log("Timer Done")
}, 0);
console.log("End")
```

#### Execution Steps:

1. JS Engine execute start
2. setTimeout goes to web APIs
3. End executes
4. Callback waits in queue
5. Event loop pushes callback into stack
6. Timer done prints

```code
Start
End
Timer done
```

#### Conclusion:

A Javascript runtime environment provides the Javascript engine along with APIs, event loop, and queues required to execute the code and handle async operations.

- Javascript engine executes code
- Runtime provides async capabilities
- Browser and Node.js are runtimes
- Event loop is part of runtime, not the JS Engine.
- DOM is a Web API provided by browser.
- setTimeout is part of runtime.

### 2. Call Stack

The Call stack is a data structure (LIFO: Last In First Out) used by Javascript engine to keep track of function calls and their execution order.

Whenever a function is called it pushes onto the stack. When it finishes it popped off.

#### Key Responsibility of Call Stack

- Execute sync Javascript code
- Maintains execution Context
- Ensure correct order of execution
- Handle function calls and returns

#### How Callstack Works

Example:

```js
function first(){
    second();
}
function second(){
    console.log("hello");
}
first();
```

Call Stack Flow

```code
1. Global() pushed
2. first()  pushed
3. second() pushed
4. console.log() pushed
5. console.log() popped
6. second() popped
7. first()  popped
8. Global() popped
```

#### Call Stack is synchronous only

The call stack:
- Does not handle async tasks
- Can not pause execution
- can not multitask

Example:

```js
setTimeout(()=>{
    console.log("Async");
}, 1000);
console.log("Sync")
```

Execution:

- setTimeout -> sent to Web APIs
- console.log("Sync") -> runs immediately
- Call stack never waits
  
#### Call Stack Overflow

Occurs when
- Function call themselves endlessly
- Stack memory limit exceeded

Example:

```js
function recurse(){
    recurse();
}
recurse();
```

Result:

```code
RangeError: Maximum call stack size exceeded
```

#### Call Stack vs Heap

|Call Stack | Heap |
|-----------|------|
|Stores function calls| Stores object and variables|
|LIFO | Unordered|
|Fast access | Slower|
|Limited Size | Larger|

#### Conclusion

The Call Stack is a LIFO data Structure used by the Javascript engine to manage function calls and execute sync code in order.

- A Javascript Engine can only have one callstack
- Js Executes one function at a time
- Blocking code blocks the stack
- Async code never blocks the stack (it is handled by web APIs)
 
### 3. Memory Heap

The memory heap is a large, unstructured region of memory where Javascript stores `Object`, `arrays`, `functions`, and `reference-type` values.

Unlike call stack, the heap is not ordered and not LIFO.

#### Why Do We Need a Heap?

Because:
- Object can be large
- Size is dynamic
- Lifetime is unpredictable

Stack memory is small and fixed -> not suitable for objects.

#### What Goes into Stack vs Heap

Stack:
- Primitive values (number, string, boolean)
- Function execution contexts
- References (pointer)

Heap:
- Objects
- Arrays
- Functions
- Closures
- DOM nodes

#### Example

```js
let a = 10;
let user = {name: "Avinash"};
```

Memory:

```code
Stack:
a -> 10
user -> reference(0x123)

Heap:
ox123 -> {name: "Avinash"}
```

#### Heap & References

```js
let obj1 = {count: 1};
let obj2 = obj1;

obj2.count = 5;
console.log(obj1.count); // 5
```

Both variables point to the same heap object.

#### Garbage Collection (GC)

Javascript automatically frees memory using Garbage Collection.

Most common method:

Mark and Sweep
1. Mark all reachable objects
2. Sweep (remove) unreachable objects

```js
let data = {value: 100};
data = null; // object becomes unreachable
```

GC removes it later.

#### Memory Leak

Common Causes:
- Global variables
- Uncleared timers
- Forgotten event listeners
- Closures holding references

Example:

```js
setInterval(()=>{
    console.log("Running Forever");
}, 1000);
```

If not cleared -> memory leak

#### Heap vs Stack

|Feature | Stack | Heap |
|--------|-------|------|
|Structure| LIFO | Unstructured|
|Speed|Faster|Slower|
|Size | Small | Large|
|Stores | Execution context | Object & References|

#### Conclusion

The memory heap is a large, unstructured memory area where Javascript stores objects and reference-type data, manged automatically by garbage collection.

- Objects live in heap
- Stack holds references
- Heap is garbage collected
- Memory leaks come from bad reference handling

### 4. Execution Context (Global + Function)

An Execution Context (EC) is the environment in which Javascript code is evaluated and executed. Every time Javascript runs code, it does so inside an execution context.

#### Types of Execution Context

1. Global Execution Context
2. Function Execution Context
3. Eval Execution Context(rare usually ignored)

#### GLobal Execution Context (GEC)

- Created once when the JS program starts
- Contains:
  - Global variables
  - Global Functions
  - this keyword

In Browser:

```js
this === window // true
```

In Node

```js
this === global // false (module scope)
```

#### Function Execution Context (FEC)

- Created every time a function is called.
- Each Function gets it own execution context

#### Execution Context Phase

Each execution context has tow phases:

**Phase 1:** Creation Phase (Memory Allocation)

During this phase:
- Variables -> undefined
- Functions -> stored fully in memory
- this is determined
- Scope chain is created

Example:

```js
console.log(a); // undefined
sayHi(); // works
var a = 10;
function sayHi(){
    console.log("Hi");
}
```

**Phase 2:** Execution Phase

- Code is executed line by line
- Variables get assigned actual values
- Functions are invoked

#### Execution Context Structure

```code
Execution Context {
    Variable Environment
    Lexical Environment
    this binding
}
```

#### Execution Context and Call Stack

```js
function a(){
    b();
}
function b(){
    console.log("Hello");
}

a();
```

Call Stack

```code
|b()| -> b goes to callstack
|a()| -> a goes to call stack
|GEC| -> Global Execution Context 
```

Each stack entry  = an execution context

#### Hoisting Explained Via Execution Context

```js
console.log(x); // undefined
var x = 5;
```

Why?

- x allocated in creation phase
- Assigned in execution phase

#### Conclusion

An execution context is an internal environment created by the Javascript engine to execute code, consisting of memory allocation, scope, and the value of this.

- Execution context !== function
- Created Before code execution
- Tow Phase: creation & execution
- Every function call creates a new context

### 5. Synchronous vs Asynchronous Javascript

#### What does Synchronous mean?

Synchronous code runs one step at a time, in order. Each operation blocks the next until it finishes.

Example:

```js
console.log("A");
console.log("B");
console.log("C");
```

Output:

```code
A
B
C
```

Each line waits for previous one.

#### What does Asynchronous mean?

Asynchronous code allows Javascript to:
- Start a task
- Move on to other work
- Comeback later when task is done

Example:

```js
console.log("Start");
setTimeout(()=>{
    console.log("Async task");
}, 0);
console.log("End");
```

Output:

```code
Start
End
Async Task
```

#### How Javascript Handles Async Code (High Level)

1. Sync code -> goes to call stack
2. Async task -> handled to Web APIs
3. When completed -> callback is placed in queue
4. Event loop moves the callback to call stack

#### Why Async is needed In JavaScript

Javascript is single threaded.

Without Async:
- UI Freezes
- App becomes unresponsive
- Network & file operations block everything

Async solves this.

#### Sync bs Async

|Feature | Sync | Async |
|--------| -----|-------|
|Execution| One At a time| Non-blocking|
|Waiting | Blocks | Doesn't Block|
|Performance| Slow for I/O| Efficient|
|Use Case | CPU Logic | Network, timers, I/O|
| Example| for loop | fetch, setTimeout|

#### Common Misconception

- Async != multi threaded
- Async = non-blocking via event loop

Javascript still usages main thread.

#### Conclusion

Synchronous Javascript executes code sequentially and blocks execution, while async Javascript allows long-running task to be handled without blocking the main thread using callbacks, promises, and the event loop.

- Sync blocks the execution
- Async usages event loop
- Async improves responsiveness
- Js remains single threaded.

### 6. Blocking vs Non-Blocking Execution

#### What is Blocking?

Blocking means, A task occupies the call stack and prevents any other code from running until it finishes.

When JS is blocked:

- UI Freezes (browser)
- Server can't handle other requests (Node.js)
- Event loop can't do anything

Example:

```js
console.log("Start");
const start = Date.now();
while(Date.now() - start < 3000){
    // blocking loop
}
console.log("End");
```

For 3 Seconds:
- No Clicks
- No times
- No async callback
- Everything frozen

#### What is Non-Blocking?

Non-blocking means, A task is delegated to the runtime (Web APIs/libuv) so the call stack remains free. Javascript continues executing other code while waiting.

Non-Blocking Example:

```js
console.log("Start");
setTimeout(()=>{
    console.log("Done");
}, 3000);
console.log("End");
```

Output:

```code
Start
End
Done
```

Main thread is never blocked.

#### Execution Flow

Blocking:

```code
Call Stack
  |-- Long task
  |-- Everything waits
```

Non-Blocking

```code
Call Stack
  |-- Long task (3s)
  |-- Everything waits
Event Loop
  |-- Executes callback later
```

- Sync code can be non blocking and async code can be blocking it depends on design (good or bad).

Example: Async Blocking

### 7. What is Event-Driven Programming?

## B. Core Event Loop Concepts

### 1. What is Event Loop?

### 2. Call Stack and Event Loop Interaction

### 3. Task Queue (Callback Queue)

### 4. Microtask Queue (Promise/MutationObserver)

### 5. Macrotask Queue (Timers/ I/O, Events)

### 6. Microtask Priority over Macrotask

### 7. Tick vs Microtask (Node.js vs Browser differences)

## C. Web APIs (Browser Environment)

### 1. What are Web APIs?

### 2. Timer API (setTimeout, setInterval)

### 3. DOM Events API

### 4. Fetch API & Network Layer

### 5. Promise & Microtask API Implementation

### 6. Mutation Observer

### 7. IntersectionObserver

### 8. WebSocket API

### 9. Animation APIs (requestAnimationFrame)

### 10.  Storage APIs (localStorage, sessionStorage)

### 11. Geolocation API

### 12. FileReader & File API

### 13. Clipboard API

### 14. Web Crypto API

> These APIs run outside the JS engine and notify JS through the event loop

## D. Browser Concurrency & Workers

### 1. What are Web Workers?

### 2. Dedicated Workers

### 3. Shared Workers

### 4. Service Workers

### 5. Message passing Models

### 6. Worker Thread Limitations (No DOM access)

### 7. Worklet (CSS paint API, Audio Worklet)

### 8. WebAssembly in Workers

## E. Node.js Event Loop (Libuv)

### 1. What is Libuv?

### 2. Node's Event Loop Phases

   - timers
   - pending callbacks
   - idle/prepare
   - poll
   - checks
   - close callbacks

### 3. Process.nextTick()

### 4. Microtask Queue in Node

### 5. Macrotask Queue in Node

### 6. setImmediate vs setTimeout

### 7. Handling I/O with thread pool

### 8. Node.js Worker threads

### 9. Node.js Clustering (multi-process concurrency)

## F. Deep Internal Concepts

### 1. How Promise Actually Works

### 2. Job Queue vs Microtask Queue

### 3. Task Source Categories (HTML spec)

### 4. Long Task and Event Loop Lag

### 5. Event Loop Starvation

### 6. Zero-delay timers and what 0ms != 0ms

### 7. Performance.now(), queueMicrotask()

### 8. Render Queue vs Event Loop

### 9. Event loop in different JS engine (V8, SpiderMonkey)

## G. Browser Rendering Pipeline

### 1. Critical Rendering Path

### 2. Frame Lifecycle

### 3. Layout, Paint, Composite

### 4. requestAnimationFrame timing

### 5. Microtask Running Before Rendering

### 6. Main Thread vs Compositor thread

## H. High Performance Pattern

### 1. Debouncing

### 2. Throttling

### 3. Idle Callbacks (requestIdleCallback)

### 4. Web Workers for CPU-heavy tasks

### 5. Avoiding Layout thrashing

### 6. Using microtask for predictable async behavior

### 7. Avoiding event loop blocking

### 8. Capturing and Bubbling Optimization

## I. Architecture Level understanding

### 1. Event driven architecture in frontend

### 2. Event emitter in Node.js

### 3. Concurrency model differences: Browser vs Node

### 4. MicroServices and Event Loop in Distributed System

### 5. Queues + Event loop synergy (Kafka Like Pattern)

## J. System Design 

### 1. Explain Event Loop step By Step With output Prediction

### 2. Priority Between microtask and Macrotask

### 3. What Node.js uses Libuv?

### 4. Can Javascript be multithreaded?

### 5. Compare Event loop with python asyncio

### 6. How Browser Implement Parallelism
