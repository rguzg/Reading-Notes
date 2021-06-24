# The Node.js Event Loop, Timers, and process.nextTick()
https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/

*Writers notes are in italics*

# What is the event loop?
- The event loop is what allows NodeJS to perform non-blocking I/O operations, even though the JavaScript engine is single-threaded
- The event loop offloads operations to the OS kernel. When the operation's completed, the kernel tells the JavaScript engine so that the callback supplied by the JavaScript file can be added to the poll queue and eventually executed *I'm guessing that in Windows, this involves using APCs*.

# Event loop explained
- When NodeJS starts, the event loop initializes and then the input script is processed. This input script might make async API calls, schedule timers or call `process.NextTick()`. After the script is processed, the event loop starts doing its own thing.
- A simplified overview of the event loop's order of operations is the following: `timers -> pending callbacks -> idle, prepare -> poll (handle incomming connections, data, etc) -> check -> close callbacks -> back to timers`
- Each operations is referred to as a phase. Each phase has a queue of callbacks to execute. 
- Each phase is special in its onw way, but generally, when the event loop enters a given phase it will perform any operations specific to that phase and then execute callbacks until the queue is emptied or until the maximum number of callbacks is executed. When the queue has been emptied or the maximum number of callbacks has been reached, the event loop moves to the next phase. *The way the event loop works sounds quite similar to the way interrupts are handled in Windows NT. Each event loop phase could be an IRQL and callbacks could be the routines associated with each IRQL or they could even be RPCs. The initial phase of the event loop could be an equivalent to the passive IRQL, but I digress...*
- Any of the operations executed by the event loop can schedule more operations. If these new operations are created in the poll phase, then after the kernel finishes working on the operations, new poll events are queued even if other polling events are being processed. This can cause the poll phase to run much longer than a timer's threshold. *This is similar to how Windows NT handles the RPC queue in the Dispatch/DPC IRQL*

# Phases Overview
- **Timers**: Executes callbacks scheduled by `setTimeout()` and `setInterval()` *Timers is the highest thing in the Event Loop just like the Clock IRQL is one of the highest IRQLs*
- **Pending Callbacks**: Executes I/O callbacks deferred to the next loop iteration
- **Idle, Prepare**: Used internally
- **Poll**: Retrieve new I/O events; execute I/O related callbacks (these are pretty much all callbacks except the ones related to timers, close callbacks and `setImmediate()`); Node will block here when appropriate
- **Check**: `setImmediate()` callbacks are executed here
- **Close**: some close callbacks are executed here, e.g. `socket.on('close)`

Between each run of the event loop, NodeJS checks if it's waiting for any asynchronous I/O or timers and shuts down cleanly if there're not any

# Phases in detail
## Timers
- A timer specifies the threshold after which a provided callback may be executed
- Timer callbacks will run as early as they can be scheduled to run after the specified amount of time has passed. The OS scheduling might also delay the running of the callbacks. *Timers don't execute after an specific amount of time. There're several things than can delay them!*

### Example

```javascript
const fs = require('fs');

function someAsyncOperation(callback) {
  // Assume this takes 95ms to complete
  fs.readFile('/path/to/file', callback);
}

const timeoutScheduled = Date.now();

setTimeout(() => {
  const delay = Date.now() - timeoutScheduled;

  console.log(`${delay}ms have passed since I was scheduled`);
}, 100);

// do someAsyncOperation which takes 95 ms to complete
someAsyncOperation(() => {
  const startCallback = Date.now();

  // do something that will take 10ms...
  while (Date.now() - startCallback < 10) {
    // do nothing
  }
});
```

How is the event loop going to behave when this script is executed?
- When the event loops starts executing, the 100ms timer hasn't elapsed, so the event loop goes through the other event loop phases until it reaches the poll phase
- When entering the poll phase, it has an empty queue (because `fs.readFile()` hasn't completed), so it'll wait for the number of milliseconds remaining until the soonest timer's threshold is reached *the documentation says that the event loop is "waiting" but I think technically what's happening is that the event loop continues to execute, but nothing really happens because no timers are expiring and nothing is being added to the poll queue. As NodeJS knows that there's a timer scheduled, the event loop continues to idle instead of shutting down.*
- While waiting for the timer, 95ms pass and`fs.readFile()` completes, so its callback is added to the poll queue and then executed. The callback takes 10ms to complete.
- After the callback is executed, as the threshold for the 100ms timer has elapsed, the event loop jumps back to the timers phase, there the callback for the timer will be added to the queue and then executed.

This example also showcases how timer callbacks may not be executed at the exact time stated when the timer is called. In this particular case, the timer callback executed 105ms after the `setTimeout()` was called, instead of 100ms.

To prevent the poll phase from starving the event loop (as it can add new callbacks to the queue while other callbacks are being executed), there is a hard maximum before it stops polling for more events.

## Pending Callbacks
- Some callbacks are deferred to the next event loop iteration. Those callbacks are executed here.
- One example of a deferred callback is when a socket receives an `ECONNREFUSED` error. Some *nix systems want to wait to report the error. The error reporting will be queued to execute in this phase.

## Poll
- The poll phase has two main functions:
    - Calculating how long it should block and poll for I/O, then *a reminder that I/O is stuff that happens asynchronously. Here NodeJS calculates for how long the event loop thread will be on a waiting state, waiting (valga la redundancia), for I/O operations to complete*
    - Processing events in the poll queue

- When the event loop enters the poll phase and there're no timers scheduled, one of two things will happen:
    - If the poll queue is not empty, the event loop will iterate through its queue of callbacks executing them synchronously until either the queue becomes empty or the the system-dependent hard limit is reached.
    - If the poll queue is empty, one of two things will happen:
        - If scripts have been scheduled by `setImmediate()` then the event loop will end the poll phase and continue to check phase to execute those scheduled scripts
        - If scripts have not been scheduled, the event loop will wait for callbacks to be added to the queue, then execute them immediately

Once the poll queue is empty, the event loop will check for timers that have expired. If one or more timers are ready, the event loop will go back to the timers phase and execute those timers' callbacks. *Is this what happens when there're timers scheduled when entering the poll phase?*

## Check
- This phase is where `setImmediate()` callbacks are executed. 
- This phase allows scripts to execute callbacks immediately after the poll queue is completed. If the poll phase becomes idle and there're callbacks inside the check queue, the event loop will move on to the check phase instead of waiting.
- If during the poll phase becomes idle *AKA, it's blocking and waiting for I/O and there's nothing on the poll queue*, it will end and continue to the check phase instead of continuing to wait for poll events

## Close callbacks
If a socket or handle is closed abruptly, the `'close'` event will be emitted here

# setImmediate() vs setTimeout()
- `setImmediate()` executes a callback after the current poll phase completes
- `setTimeout()` schedules a callback to be run after a minimum threshold in milliseconds has elapsed

The order in which timers are executed depends on the context in which they are called. If both are called from within the main module, the timing will be bound by the performance of the process (which can be impacted by other applications running on the machine) *Preemptive multitasking FTW? or just set the process' priority to real-time :Kappa:*

For example, the following script is not within an I/O cycle *I think this means that `setImmediate()` and `setTimeout()` are called when NodeJS is processing the synchronous parts of the code*, so the order in which the two timers are executed is non-deterministic, as it's bound to the process performance *AKA it depends on when the kernel decides to make the timer expire*

```javascript
// timeout_vs_immediate.js
setTimeout(() => {
  console.log('timeout');
}, 0);

setImmediate(() => {
  console.log('immediate');
});
```
```bash
$ node timeout_vs_immediate.js
timeout
immediate

$ node timeout_vs_immediate.js
immediate
timeout
```

However, if you move the two calls within an I/O cycle *AKA, they're not executed during the synchronous part of the code*, the immediate callback is always executed first. 

*This is quite the interesting puzzle to find out why this happens. I'm guessing it happens because `setImmediate()` and `setTimeout()` are called inside a poll callback. When the poll callback is executed, the callback for `setImmediate()` is added to the check queue and NodeJS starts waiting for the `setTimeout()` timer to expire. After the poll callback is executed, the poll queue becomes empty and as stated in the explanation for the poll phase, the event loop checks for any `setImmediate()` callbacks first before checking on any expired timers.*

# process.nextTick()
- `process.nextTick()` is not part of the event loop. The `nextTickQueue` will be processed after the current operation is completed, regardless of the current phase of the event loop. An operation is defined as a transition from the C/C++ handler and handling the JavaScript that needs to be executed *i.e. an operation finishing is the time between the JavaScript engine determining that a callback needs to be executed and the callback being executed*
- Any time `process.nextTick()` is called in a given phase, all callbacks to `process.nextTick()` will be resolved before the event loop continues. This can be a problem, because it allows you to starve your I/O callbacks by making recursive `process.nextTick()` calls, which prevents the evet loop from reaching the poll phase
- `process.nextTick()` exists as a design philosophy where an API should always be asynchronous even where it doesn't have to be. For example, in this function, the callback is called asynchronously using `process.nextTick()`:

```javascript
function apiCall(arg, callback) {
  if (typeof arg !== 'string')
    // The second argument of process.nextTick is the arguments that are passed to the callback. This was done so that an arrow function
    // containing the callback didn't have to be added
    return process.nextTick(callback,
                            new TypeError('argument should be string'));
}
```

- On the example above, if the argument check fails, the callback will be called with an error, but only **after the rest of the synchronous code has be run and before the event loop starts**
- Calling `process.nextTick()` just registers the callback inside of the nextTickQueue, this allows recursive calls to `process.nextTick()` without reaching a `RangeError: Maximum call stack size exceeded from v8`
- This philosophy can lead to some problematic situations, like this one:
```javascript
let bar;

// this has an asynchronous signature, but calls callback synchronously
function someAsyncApiCall(callback) { callback(); }

// the callback is called before `someAsyncApiCall` completes.
someAsyncApiCall(() => {
  // since someAsyncApiCall hasn't completed, bar hasn't been assigned any value
  console.log('bar', bar); // undefined
});

bar = 1;
```
- In the example, `someAsyncApiCall` calls `callback` synchronously, which means that when the callback is executed `bar` will still not be in scope, because the script has not run to completion yet.
- This problem can be solved by placing `callback` inside `process.nextTick()`. By doing this, the script will be able to run to completion and `bar` will be in scope when the callback is run
- `process.nextTick()` can be useful when you need to inform the user about an error before the event loop continues
- Another real world example:

```javascript
const server = net.createServer(() => {}).listen(8080);

server.on('listening', () => {});
```

- When only a port is passed, the port is bound immediately. So, the `listening` callback could be called immediately, the problem is that the `.on('listening')` callback will not have been set by that time. To get around this, the `listening` event is queued in a `process.nextTick()` to allow the script to run to completion. This allows the user to set any event handlers they want.

## process.nextTick() vs setImmediate()    
- The names of these two functions are backwards, `process.nextTick()` fires immediately on the same phase and `setImmediate()` fires on the following iteration (tick) of the event loop.
- The names should be backwards, but doing that would break a lot of npm packages
- It's recommended to use `setImmediate()` because it's easier to reason about

## Why use process.nextTick()?
- It can be used to handle errors, cleanup unneeded resources or try a request again before the event loop continues. *For example, in the case of a request, the request could be attempted again to prevent another part of the program from crashing because the request failed*
- Sometimes it's necessary to run a callback after the call stack has unwound but before the event loop continues. An example of this is:
```javascript
const server = net.createServer();
server.on('connection', (conn) => { });

server.listen(8080);
server.on('listening', () => { });
```
- Say that `listen()` is run at the beginning of the event loop, but the execution of the listening callback is placed inside a `setImmediate()` call. Unless a hostname is passed, binding to the port will happen during the next time the event loop reaches the poll phase (this is because just opening a socket without having to resolve the hostname is a process that happens really quickly), as `setImmediate()` callbacks are called after the poll phase completes, there's a chance that a connection could have been received before the connection callback was assigned. *i.e. process.nextTick() could be used to prevent race conditions. BTW, remember that we already saw why the listening callback can't be called synchronously*
- Another example is:
```javascript
const EventEmitter = require('events');
const util = require('util');

function MyEmitter() {
  EventEmitter.call(this);
  this.emit('event');
}
util.inherits(MyEmitter, EventEmitter);

const myEmitter = new MyEmitter();
myEmitter.on('event', () => {
  console.log('an event occurred!');
});
```
- You can't emit an event from the constructor immediately because the callback for the event hasn't been assigned yet. So, you can use `process.nextTick()` to set a callback to emit the event after the constructor has finished:
```javascript
const EventEmitter = require('events');
const util = require('util');

function MyEmitter() {
  EventEmitter.call(this);

  // use nextTick to emit the event once a handler is assigned
  process.nextTick(() => {
    this.emit('event');
  });
}
util.inherits(MyEmitter, EventEmitter);

const myEmitter = new MyEmitter();
myEmitter.on('event', () => {
  console.log('an event occurred!');
});
```