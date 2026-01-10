# Module 1: JavaScript Fundamentals & Node.js Core

## JavaScript Runtime & Event Loop

### Runtime Environment

**Browser vs Node.js**

The JavaScript runtime environment determines what APIs and capabilities are available to your code. Browsers provide DOM manipulation, Web APIs (fetch, localStorage), and the window object for web applications, while Node.js offers a server-side environment with C++ bindings, a global object, and the libuv event loop for backend operations. Both environments share the V8 JavaScript engine and the core language, but implement different event loops optimized for their respective use cases—browsers focus on UI responsiveness, while Node.js prioritizes I/O throughput.

- **Browser**: DOM, Web APIs (fetch, localStorage), window object
- **Node.js**: No DOM, C++ bindings, global object, libuv event loop
- **Both**: V8 engine, JavaScript language, event loop (different implementations)

**V8 Architecture Components**

V8 is Google's high-performance JavaScript and WebAssembly engine that compiles JavaScript to optimized machine code. It uses a multi-tiered compilation pipeline where source code is parsed into an Abstract Syntax Tree (AST), interpreted to bytecode by Ignition, and hot functions are optimized by TurboFan. This architecture enables fast startup times through interpretation while achieving near-native performance for frequently executed code through just-in-time (JIT) compilation.

```
Source Code
    ↓
[Parser] → AST (Abstract Syntax Tree)
    ↓
[Ignition Interpreter] → Bytecode (Baseline)
    ↓
[TurboFan Compiler] → Optimized Machine Code (Hot Functions)
    ↓
[Orinoco GC] → Memory Management
```

**Parser**

The parser converts raw JavaScript source code into a structured Abstract Syntax Tree (AST) that represents the code's grammatical structure. It performs lexical analysis (tokenization) to identify keywords, identifiers, and operators, then syntactic analysis to build the AST with node types representing functions, statements, and expressions. This step also validates syntax and reports errors before execution begins, making it crucial for catching bugs early.

- **Lexer/Tokenizer**: Keywords, identifiers, operators
- **Parser**: AST construction with node types
- **Syntax validation and error reporting

**Ignition (Interpreter)**

Ignition is V8's baseline interpreter that generates bytecode from the AST for fast startup and initial execution. It uses a register-based bytecode architecture (as opposed to stack-based) which is more efficient for modern CPUs. While unoptimized, Ignition collects profiling data about function execution frequency and types, which informs TurboFan's optimization decisions. This design allows applications to start quickly while identifying hot code paths for later optimization.

- Generates bytecode from AST
- Register-based bytecode architecture
- Fast startup, unoptimized execution
- Collects profiling data for TurboFan

**TurboFan (Optimizing Compiler)**

TurboFan is V8's optimizing just-in-time (JIT) compiler that transforms frequently executed (hot) bytecode into highly optimized machine code. It uses execution counting to identify hot functions and applies advanced optimizations like inlining, escape analysis, and type specialization. When assumptions break (e.g., type changes or hidden class transitions), TurboFan performs deoptimization to fall back to interpreted bytecode, ensuring correctness while maintaining performance for stable code paths.

- JIT compilation for hot functions (execution counting)
- Optimization tiers: unoptimized → optimized → deoptimized
- Optimizations: inlining, escape analysis, type specialization
- Deoptimization triggers: type changes, hidden class transitions

**Orinoco Garbage Collector**

Orinoco is V8's generational garbage collector that manages memory automatically by reclaiming objects that are no longer reachable. It uses a generational approach with New Space for young objects (using the Scavenge algorithm with Cheney's semi-space copying) and Old Space for mature objects (using Mark-Sweep-Compact). The collector runs incrementally and concurrently to avoid blocking the main thread, which is critical for maintaining smooth UI responsiveness and server throughput.

- **Generational GC**: New Space (young) vs Old Space (mature)
- **Scavenge algorithm** for New Space (Cheney's algorithm, semi-space)
- **Mark-Sweep-Compact** for Old Space
- **Incremental and concurrent marking** (non-blocking)

```javascript
// Hidden Classes (Maps) - V8 optimization
const obj1 = { x: 1, y: 2 };
const obj2 = { x: 3, y: 4 }; // Same hidden class (fast)
obj1.z = 5; // Hidden class transition (slower)

// Inline Caching (IC) - Property access optimization
function getX(o) { return o.x; }
getX(obj1); // Monomorphic IC (fast)
getX(obj2); // Reuse IC (same shape)
getX({ x: 1, y: 2, z: 3 }); // Megamorphic IC (slow)
```

**Performance Implications**

V8's optimizations rely heavily on predictable code patterns. Object shape consistency (same property order) allows V8 to reuse hidden classes and inline caches, dramatically improving property access speed. Hidden class transitions cause deoptimization, forcing fallback to slower paths. Memory fragmentation increases garbage collection pressure, so it's important to avoid adding properties dynamically after initialization. Understanding these patterns helps write JavaScript that performs well in V8-optimized environments like Node.js and Chrome.

- Object shape consistency matters (same property order)
- Hidden class transitions cause deoptimization
- Memory fragmentation increases GC pressure
- Avoid adding properties dynamically after initialization

---

### Event Loop Deep Dive

**Event Loop Phases (Node.js)**

The Node.js event loop is a single-threaded mechanism that handles asynchronous I/O operations by delegating blocking tasks to the system kernel and processing callbacks in a specific order. It consists of six phases that execute in sequence: Timers, Pending Callbacks, Idle/Prepare, Poll, Check, and Close Callbacks. Between each phase, `process.nextTick()` callbacks execute, providing the highest priority queue. This architecture enables Node.js to handle thousands of concurrent connections efficiently without blocking the main thread.

```
┌─────────────────────────────┐
│     Timers Phase           │ → setTimeout, setInterval callbacks
├─────────────────────────────┤
│   Pending Callbacks Phase   │ → I/O callbacks (except timers, close)
├─────────────────────────────┤
│   Idle/Prepare Phase       │ → Internal Node.js use only
├─────────────────────────────┤
│      Poll Phase            │ → I/O polling, new I/O callbacks
├─────────────────────────────┤
│      Check Phase           │ → setImmediate callbacks
├─────────────────────────────┤
│   Close Callbacks Phase    │ → socket.on('close'), cleanup
└─────────────────────────────┘
          ↑           ↓
          └───────────┘
     process.nextTick() runs between EVERY phase
```

**Execution Order**

The event loop follows a strict execution order that ensures predictable behavior for asynchronous code. First, the current call stack executes synchronously. Then all microtasks (Promises, queueMicrotask) run before moving to macrotasks. The `process.nextTick()` queue has the highest priority and runs between every phase of the event loop. Understanding this order is crucial for writing correct asynchronous code and avoiding race conditions.

1. Execute current call stack
2. Execute all microtasks (Promises, queueMicrotask)
3. Execute `process.nextTick()` queue (highest priority microtask)
4. Run event loop phase
5. Repeat

```javascript
// Event loop execution order example
console.log('1. Start');

setTimeout(() => console.log('2. setTimeout'), 0);

setImmediate(() => console.log('3. setImmediate'));

Promise.resolve().then(() => console.log('4. Promise'));

process.nextTick(() => console.log('5. nextTick'));

console.log('6. End');

// Output: 1, 6, 5, 4, 2, 3 (in I/O cycle)
// Output: 1, 6, 5, 4, 3, 2 (in main module)
```

**process.nextTick() vs setImmediate()**

`process.nextTick()` and `setImmediate()` are both scheduling mechanisms but serve different purposes. `process.nextTick()` is a microtask that runs between every phase of the event loop, making it the highest priority queue—use it to ensure execution before I/O operations. `setImmediate()` is a macrotask that runs in the Check phase after the Poll phase, making it ideal for breaking up long-running CPU operations. The naming is counterintuitive: `nextTick` runs before `setImmediate` in most cases, and their relative order can vary depending on whether the code is in the main module or an I/O callback.

| Feature | process.nextTick() | setImmediate() |
|---------|-------------------|----------------|
| Type | Microtask | Macrotask |
| Queue | Runs between every phase | Runs in Check phase |
| Priority | Higher than Promises | Lower than microtasks |
| Use case | Ensure execution before I/O | Break up long operations |
| I/O cycle | After setImmediate | Before nextTick |

---

### Asynchronous JavaScript Patterns

**Callbacks**

Callbacks are functions passed as arguments to other functions, executed after an asynchronous operation completes. They follow the error-first callback pattern where the first argument is an error (null if successful) and subsequent arguments contain the result. While simple and widely supported, callbacks can lead to "callback hell" with deeply nested code, making error handling and control flow difficult to manage in complex applications.

```javascript
function fetchData(callback) {
  setTimeout(() => {
    callback(null, { id: 1, name: 'John' });
  }, 1000);
}

fetchData((err, data) => {
  if (err) return console.error(err);
  console.log(data);
});
```

**Promises**

Promises represent the eventual completion or failure of an asynchronous operation, providing a cleaner alternative to callbacks with chainable `.then()` and `.catch()` methods. A promise can be in one of three states: pending, fulfilled, or rejected. Promise utilities like `Promise.all()` (all must resolve), `Promise.race()` (first to settle), and `Promise.allSettled()` (all settle regardless of outcome) enable powerful composition patterns for handling multiple asynchronous operations in parallel.

```javascript
// Promise states: pending → fulfilled/rejected
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve({ id: 1, name: 'John' });
  }, 1000);
});

promise
  .then(data => console.log(data))
  .catch(err => console.error(err));

// Promise.all() - All must resolve
Promise.all([p1, p2, p3])
  .then(results => console.log(results));

// Promise.race() - First to settle
Promise.race([p1, p2])
  .then(result => console.log(result));

// Promise.allSettled() - All settle (resolve/reject)
Promise.allSettled([p1, p2])
  .then(results => console.log(results));
```

**async/await**

`async/await` is syntactic sugar built on top of Promises that makes asynchronous code look and behave like synchronous code. An `async` function always returns a Promise, and `await` pauses execution until the Promise resolves or rejects. This pattern eliminates callback nesting and makes error handling with try/catch blocks straightforward. For parallel execution, combine `await` with `Promise.all()` to run multiple asynchronous operations concurrently rather than sequentially.

```javascript
async function fetchData() {
  try {
    const response = await fetch('/api/users');
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Error:', error);
    throw error;
  }
}

// Parallel execution
async function fetchAll() {
  const [users, posts] = await Promise.all([
    fetch('/api/users').then(r => r.json()),
    fetch('/api/posts').then(r => r.json())
  ]);
  return { users, posts };
}
```

**Custom Promise.all() Implementation**

Understanding how `Promise.all()` works internally helps grasp Promise composition fundamentals. The implementation creates a new Promise that resolves when all input promises resolve, or rejects immediately when any promise rejects. It maintains an array to store results in order, tracks completion count, and uses `Promise.resolve()` to handle non-Promise inputs. This pattern demonstrates key Promise concepts: chaining, error propagation, and coordinating multiple asynchronous operations.

```javascript
function promiseAll(promises) {
  return new Promise((resolve, reject) => {
    if (promises.length === 0) {
      resolve([]);
      return;
    }

    const results = new Array(promises.length);
    let completed = 0;

    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(value => {
          results[index] = value;
          completed++;
          if (completed === promises.length) {
            resolve(results);
          }
        })
        .catch(reject);
    });
  });
}
```

---

### Advanced JavaScript Concepts

**Closures**

A closure is a function that retains access to its lexical scope even when executed outside that scope. This enables powerful patterns like data encapsulation, function factories, and the module pattern. Closures are created whenever a function is defined inside another function, allowing the inner function to access variables from the outer function after the outer function has returned. This mechanism is fundamental to JavaScript's functional programming capabilities and is widely used for creating private state and maintaining context.

```javascript
// Closure: Function retains access to lexical scope
function createCounter() {
  let count = 0;
  return {
    increment: () => ++count,
    decrement: () => --count,
    getCount: () => count
  };
}

const counter = createCounter();
console.log(counter.increment()); // 1
console.log(counter.getCount());  // 1

// Module pattern (IIFE + closure)
const Module = (() => {
  let privateVar = 0;
  return {
    increment: () => ++privateVar,
    get: () => privateVar
  };
})();
```

**Hoisting**

Hoisting is JavaScript's behavior of moving declarations to the top of their scope during compilation. `var` declarations are hoisted and initialized to `undefined`, while `let` and `const` are hoisted but remain in the Temporal Dead Zone (TDZ) until the declaration line, causing a ReferenceError if accessed early. Function declarations are fully hoisted, making them callable before their definition, while function expressions are not hoisted. Understanding hoisting helps avoid subtle bugs and write more predictable code.

```javascript
// var: hoisted and initialized to undefined
console.log(x); // undefined (not ReferenceError)
var x = 5;

// let/const: hoisted but in Temporal Dead Zone (TDZ)
// console.log(y); // ReferenceError
let y = 5;

// Function declarations: fully hoisted
console.log(hoisted()); // 'hoisted'
function hoisted() { return 'hoisted'; }

// Function expressions: not hoisted
// console.log(notHoisted()); // TypeError
const notHoisted = () => 'not hoisted';
```

**this Binding**

The `this` keyword in JavaScript refers to the execution context of a function, which depends on how the function is called. Implicit binding occurs when a function is called as a method of an object, setting `this` to that object. Explicit binding uses `call()`, `apply()`, or `bind()` to specify `this` directly. Arrow functions have lexical `this`, meaning they inherit `this` from their enclosing scope. Understanding `this` binding is crucial for object-oriented programming, event handling, and working with libraries that rely on context.

```javascript
// this depends on call site
const obj = {
  name: 'John',
  greet() { console.log(this.name); }
};

obj.greet(); // 'John' (implicit binding)

const greet = obj.greet;
greet(); // undefined (default binding in strict mode)

// call/apply: explicit this binding
greet.call({ name: 'Jane' }); // 'Jane'
greet.apply({ name: 'Bob' }); // 'Bob'

// bind: permanent this binding
const boundGreet = greet.bind({ name: 'Alice' });
boundGreet(); // 'Alice'

// Arrow functions: lexical this
const obj2 = {
  name: 'John',
  greet: () => console.log(this.name) // this from outer scope
};
```

**Prototypes & Prototypal Inheritance**

JavaScript uses prototypal inheritance, where objects inherit directly from other objects rather than through class hierarchies. Every object has an internal `[[Prototype]]` link (accessible via `__proto__` or `Object.getPrototypeOf()`) pointing to its prototype object. When accessing a property, JavaScript traverses the prototype chain until found or reaching `null`. ES6 classes provide syntactic sugar over this prototype system, making inheritance more familiar to developers from class-based languages while maintaining the underlying prototype mechanism.

```javascript
// Prototype chain
function Person(name) {
  this.name = name;
}

Person.prototype.greet = function() {
  console.log(`Hello, ${this.name}`);
};

const john = new Person('John');
john.greet(); // 'Hello, John'

// ES6 classes (syntactic sugar)
class Person {
  constructor(name) {
    this.name = name;
  }

  greet() {
    console.log(`Hello, ${this.name}`);
  }

  static species = 'Homo sapiens';
}

// Inheritance
class Employee extends Person {
  constructor(name, title) {
    super(name);
    this.title = title;
  }

  introduce() {
    console.log(`${this.name} is a ${this.title}`);
  }
}
```

**Memory Leak Patterns**

Memory leaks occur when objects are unintentionally retained in memory, preventing garbage collection. Common patterns include global variables that never get cleared, event listeners that aren't removed, closures that capture large objects, uncleared timers, and DOM references to removed elements. These issues are particularly problematic in long-running applications like servers and single-page applications. Using `WeakMap` and `WeakSet` can help, as they allow garbage collection when keys are no longer referenced elsewhere.

```javascript
// 1. Global variables
let globalCache = {}; // Never cleared

// 2. Event listeners not removed
const btn = document.getElementById('btn');
btn.addEventListener('click', handler);
// Missing: btn.removeEventListener('click', handler);

// 3. Closures retaining large objects
function createHandler(largeData) {
  return function() {
    // largeData retained even if not used
    console.log('clicked');
  };
}

// 4. Timers not cleared
const timer = setInterval(() => {
  // Missing: clearInterval(timer);
}, 1000);

// 5. DOM references
let elements = document.querySelectorAll('.item');
// Elements removed from DOM but referenced in array

// Prevention: WeakMap/WeakSet
const cache = new WeakMap();
cache.set(largeObj, metadata); // Automatically GC'd
```

---

## Node.js Core Architecture

### Architecture Fundamentals

**Single-Threaded Event-Driven Model**

Node.js uses a single-threaded event-driven architecture where one main thread handles JavaScript execution through the event loop. Non-blocking I/O operations are delegated to libuv, which manages asynchronous operations and callbacks. CPU-intensive tasks can be offloaded to a thread pool (default 4 threads) for file system, DNS, compression, and crypto operations. System calls are delegated to the OS kernel, which handles the actual I/O efficiently. This model enables high concurrency with minimal overhead, making Node.js ideal for I/O-bound applications like web servers and real-time services.

- One main thread (event loop) for JavaScript execution
- Non-blocking I/O via libuv
- Thread pool (4 threads default) for CPU-intensive tasks
- System calls delegated to OS kernel

**libuv Architecture**

libuv is a multi-platform support library with a focus on asynchronous I/O that powers Node.js's event loop and non-blocking operations. It provides an abstraction over platform-specific I/O APIs like epoll (Linux), kqueue (macOS/BSD), and IOCP (Windows). The library manages the event loop, thread pool for blocking operations, and handles asynchronous file system, DNS, and network operations. This abstraction layer allows Node.js to run consistently across different operating systems while leveraging each platform's most efficient I/O mechanism.

```
┌─────────────────────────────────┐
│     JavaScript (V8)             │
└─────────────┬───────────────────┘
              │
┌─────────────▼───────────────────┐
│   libuv Event Loop (Single)     │
├─────────────────────────────────┤
│   Thread Pool (4 threads)       │
│   - File system operations      │
│   - DNS lookups                 │
│   - Compression                 │
│   - Crypto operations           │
└─────────────────────────────────┘
              │
┌─────────────▼───────────────────┐
│   OS Kernel (epoll/kqueue/IOCP) │
└─────────────────────────────────┘
```

**System Call Wrapping**

Node.js wraps system calls with non-blocking behavior using the `O_NONBLOCK` flag for file descriptors. When an operation cannot complete immediately, it returns `EAGAIN` or `EWOULDBLOCK` errors, which libuv handles by queuing the operation for later completion. The platform-specific event notification mechanisms—epoll on Linux, kqueue on macOS/BSD, and IOCP on Windows—allow the kernel to notify Node.js when I/O operations are ready. This design enables efficient handling of thousands of concurrent connections without thread-per-connection overhead.

- Non-blocking I/O: O_NONBLOCK flag for file descriptors
- EAGAIN/EWOULDBLOCK: Handle incomplete operations
- Platform-specific: epoll (Linux), kqueue (macOS), IOCP (Windows)

**Thread Pool Operations**

The libuv thread pool handles blocking operations that cannot be performed asynchronously by the OS kernel. These include file system operations (except on some platforms where async file I/O is available), DNS lookups, compression/decompression, and cryptographic operations. By offloading these CPU-intensive or blocking tasks to worker threads, the main event loop remains free to handle incoming requests and callbacks. The thread pool size can be configured via the `UV_THREADPOOL_SIZE` environment variable, with a default of 4 threads.

```javascript
// Operations delegated to thread pool
const fs = require('fs');

// File I/O (thread pool)
fs.readFile('large-file.txt', (err, data) => {
  console.log(data);
});

// DNS lookup (thread pool)
const dns = require('dns');
dns.lookup('example.com', (err, address) => {
  console.log(address);
});

// Compression (thread pool)
const zlib = require('zlib');
zlib.gzip(data, (err, compressed) => {
  console.log(compressed);
});

// Crypto (thread pool)
const crypto = require('crypto');
crypto.pbkdf2('password', 'salt', 100000, 64, 'sha512', (err, key) => {
  console.log(key);
});
```

---

### Event Loop Phases Deep Dive

**Timers Phase**

The Timers phase executes callbacks scheduled with `setTimeout()` and `setInterval()`. Timers are stored in a binary heap for efficient minimum-delay lookup, with a minimum delay of 1ms in Node.js (though the actual delay can be longer due to event loop phases). Timing is not guaranteed to be exact because callbacks run only after the current operation completes and all microtasks finish. This phase is ideal for scheduling delayed operations where approximate timing is acceptable.

- Executes setTimeout/setInterval callbacks
- Minimum delay: 1ms (Node.js implementation)
- Binary heap for timer management
- Not guaranteed exact timing

```javascript
setTimeout(() => console.log('Timer'), 0);
// Runs after microtasks and nextTick queue
```

**Pending Callbacks Phase**

The Pending Callbacks phase executes I/O callbacks from the previous event loop iteration, excluding timers, close callbacks, and `setImmediate()`. This phase handles callbacks for operations like TCP errors, DNS resolution failures, and other deferred I/O errors. It's important to note that `process.nextTick()` runs between every phase, including before this one, which can affect callback execution order. Understanding this phase helps debug timing-sensitive I/O operations.

- I/O callbacks from previous loop iteration
- Excludes: timers, close callbacks, setImmediate
- process.nextTick() runs between phases

**Idle/Prepare Phase**

The Idle and Prepare phases are used internally by Node.js and are not exposed to user code. The Idle phase is used for V8's idle tasks and garbage collection, while the Prepare phase prepares for the upcoming Poll phase. These phases typically execute very quickly or not at all, as they're primarily for internal Node.js housekeeping. Developers don't need to interact with these phases directly, but understanding their existence provides a complete picture of the event loop.

- Internal Node.js use only
- prepare phase for poll phase setup

**Poll Phase**

The Poll phase is where most I/O callbacks are executed and new I/O events are polled. Its behavior depends on whether timers are scheduled: if no timers exist, it blocks until I/O occurs or the timeout expires; if timers exist, it's non-blocking and moves to the Check phase. This phase is crucial for Node.js's performance, as it efficiently waits for I/O without consuming CPU cycles. File system callbacks, network callbacks, and most user-defined I/O operations run here.

- I/O polling with timeout calculation
- New I/O event callbacks execution
- Blocking behavior:
  - If no timers: blocks until I/O or timeout
  - If timers: non-blocking, moves to Check phase

```javascript
// Poll phase behavior
const fs = require('fs');
fs.readFile('file.txt', () => {
  console.log('I/O callback in Poll phase');
});
```

**Check Phase**

The Check phase executes callbacks scheduled with `setImmediate()`, making it the ideal place to run code that should execute after I/O polling completes. This phase runs after the Poll phase, ensuring that I/O callbacks have had a chance to run. `setImmediate()` is useful for breaking up long-running CPU operations to avoid blocking the event loop, as it allows the event loop to process other callbacks between chunks of work.

- setImmediate() callbacks execution
- Runs after poll phase completes
- Ideal for CPU-intensive tasks

```javascript
setImmediate(() => {
  console.log('Check phase');
});
```

**Close Callbacks Phase**

The Close Callbacks phase executes cleanup callbacks like `socket.on('close')` and handles process exit scenarios. This phase runs after all other phases, ensuring that cleanup operations occur after all other work is complete. It's important for proper resource management, especially in long-running servers where connections and resources need to be cleaned up gracefully. Understanding this phase helps implement proper shutdown procedures and resource cleanup.

- socket.on('close') and similar cleanup callbacks
- Process exit handling
- Runs after all other phases

---

### Modules System

**CommonJS (require/module.exports)**

CommonJS is Node.js's original module system, using `require()` for importing and `module.exports` for exporting. Modules are cached after the first `require()`, so subsequent imports return the same instance. The `exports` object is a reference to `module.exports`, allowing shorthand exports like `exports.foo = 'bar'`. CommonJS modules load synchronously, which is suitable for server-side applications but not ideal for browser environments where asynchronous loading is preferred.

```javascript
// Exporting
module.exports = { foo: 'bar' };
// or
exports.foo = 'bar';

// Importing
const { foo } = require('./module');
const module = require('./module');

// Module caching
require.cache[require.resolve('./module')];
```

**ES Modules (import/export)**

ES Modules (ESM) is the modern JavaScript standard for modules, using `import` and `export` statements with static analysis for better optimization. Named exports allow selective importing with tree-shaking, while default exports provide a single primary export. Dynamic imports using `import()` enable code splitting and lazy loading. ESM is asynchronous by design, making it suitable for both browser and server environments. Node.js supports ESM via the `.mjs` extension or `"type": "module"` in package.json.

```javascript
// Exporting
export const foo = 'bar';
export default function() {}

// Importing
import { foo } from './module.js';
import module from './module.js';

// Dynamic import
const module = await import('./module.js');
```

**Module Resolution Algorithm**

Node.js resolves module paths through a specific algorithm that determines which file to load. First, it checks if the module is a core module (fs, path, http, etc.). For relative/absolute paths, it resolves the file directly. For bare module names, it traverses upward through `node_modules` directories. It then checks file extensions (.js, .json, .node), the package.json main field, and finally index.js. Understanding this algorithm helps debug module resolution issues and optimize dependency loading.

1. Check if module is core module (fs, path, etc.)
2. Check for relative/absolute path
3. Check node_modules (upward traversal)
4. Check file extensions (.js, .json, .node)
5. Check package.json main field
6. Check index.js

**Module Caching**

Node.js caches modules after the first `require()` call, ensuring that subsequent imports return the same instance. This singleton behavior is important for maintaining state across imports and avoiding duplicate initialization. The cache is stored in `require.cache` and can be cleared (though rarely needed) by deleting entries. Module caching improves performance by avoiding repeated parsing and execution, but developers must be aware that module-level state is shared across all imports.

```javascript
// Single instance per module
// Cached after first require
const mod1 = require('./module');
const mod2 = require('./module');
console.log(mod1 === mod2); // true

// Clear cache (rarely needed)
delete require.cache[require.resolve('./module')];
```

---

### Streams & Buffers

**Buffer Class**

Buffers are fixed-size memory allocations for handling binary data in Node.js, essential for working with files, network protocols, and streams. They can be created from strings, arrays, or allocated directly with `Buffer.alloc()` (zero-filled) or `Buffer.allocUnsafe()` (uninitialized, faster). Buffers support encoding conversions (utf8, base64, hex), concatenation, and slicing. Understanding buffers is crucial for efficient data processing in Node.js, especially when dealing with large files or network protocols.

```javascript
// Create buffer
const buf1 = Buffer.from('hello');
const buf2 = Buffer.alloc(10);
const buf3 = Buffer.allocUnsafe(10);

// Operations
buf1.toString('utf8');
buf1.toJSON();
Buffer.concat([buf1, buf2]);
```

**Stream Types**

Streams are abstract interfaces for working with streaming data in Node.js, enabling efficient processing of large datasets without loading everything into memory. There are four stream types: Readable (data source), Writable (data destination), Duplex (both readable and writable), and Transform (modifies data as it passes through). Streams support backpressure handling, pausing/resuming, and piping. They're essential for building scalable applications that process data incrementally.

```javascript
const { Readable, Writable, Duplex, Transform } = require('stream');

// Readable stream
const readable = new Readable({
  read(size) {
    this.push('data');
    this.push(null); // End
  }
});

readable.on('data', chunk => console.log(chunk));
readable.on('end', () => console.log('Done'));

// Writable stream
const writable = new Writable({
  write(chunk, encoding, callback) {
    console.log(chunk.toString());
    callback();
  }
});

writable.write('hello');
writable.end();

// Duplex stream (both readable and writable)
const duplex = new Duplex({
  read(size) { /* ... */ },
  write(chunk, encoding, callback) { /* ... */ }
});

// Transform stream (modify data)
const transform = new Transform({
  transform(chunk, encoding, callback) {
    this.push(chunk.toString().toUpperCase());
    callback();
  }
});
```

**Backpressure Handling**

Backpressure occurs when a writable stream cannot process data as fast as the readable stream produces it. Node.js streams handle backpressure automatically through the `pipe()` method, which pauses the readable stream when the writable stream's internal buffer is full and resumes when drained. Manual backpressure handling involves checking the return value of `writable.write()` and pausing/resuming the readable stream accordingly. Proper backpressure handling prevents memory overflow and ensures efficient data flow.

```javascript
const fs = require('fs');

const readable = fs.createReadStream('large-file.txt');
const writable = fs.createWriteStream('output.txt');

// Automatic backpressure with pipe()
readable.pipe(writable);

// Manual backpressure handling
readable.on('data', chunk => {
  const canWrite = writable.write(chunk);
  if (!canWrite) {
    readable.pause();
    writable.once('drain', () => readable.resume());
  }
});
```

**Stream Chaining**

Stream chaining connects multiple streams together to create data processing pipelines. Each stream in the chain receives data from the previous stream, processes it, and passes it to the next. This enables powerful composition patterns like reading a file, compressing it, and writing to another location in a single pipeline. Chaining is memory-efficient because data flows through the pipeline incrementally without loading everything into memory at once.

```javascript
const fs = require('fs');
const zlib = require('zlib');

fs.createReadStream('input.txt')
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream('input.txt.gz'))
  .on('finish', () => console.log('Done'));
```

---

### File System Operations

**Sync vs Async**

Node.js provides both synchronous and asynchronous versions of file system operations. Async operations are non-blocking and preferred in production to avoid blocking the event loop, using callbacks or Promises. Sync operations block the event loop until complete and should only be used in startup scripts or CLI tools. The `fs.promises` API provides Promise-based versions of async operations, enabling cleaner code with async/await. Choosing the right approach is crucial for maintaining application responsiveness.

```javascript
const fs = require('fs');

// Async (non-blocking, preferred)
fs.readFile('file.txt', 'utf8', (err, data) => {
  if (err) throw err;
  console.log(data);
});

// Sync (blocking, avoid in event loop)
try {
  const data = fs.readFileSync('file.txt', 'utf8');
  console.log(data);
} catch (err) {
  console.error(err);
}

// Promise-based (fs.promises)
const { readFile } = require('fs').promises;
const data = await readFile('file.txt', 'utf8');
```

**Path Module**

The `path` module provides utilities for working with file and directory paths across different operating systems. It handles platform-specific separators (backslash on Windows, forward slash on Unix), path normalization, and resolution. Functions like `join()`, `basename()`, `dirname()`, `extname()`, and `resolve()` help construct and manipulate paths safely. Using the path module is essential for writing cross-platform code that works consistently on Windows, macOS, and Linux.

```javascript
const path = require('path');

// Cross-platform paths
const filePath = path.join('dir', 'subdir', 'file.txt');
console.log(path.basename(filePath)); // 'file.txt'
console.log(path.dirname(filePath));  // 'dir/subdir'
console.log(path.extname(filePath));  // '.txt'
console.log(path.resolve('dir', '..')); // '/absolute/path'
```

**File Watching**

File watching enables applications to respond to changes in the file system automatically. The `fs.watch()` method monitors files or directories for changes, emitting events when modifications occur. Recursive directory watching (Node.js 10.12+) monitors entire directory trees. File watching is useful for development tools, hot reloading, and real-time processing of user uploads. However, it can be resource-intensive and may not be reliable on all platforms, so consider using dedicated libraries like `chokidar` for production use.

```javascript
const fs = require('fs');

// Watch file for changes
fs.watch('file.txt', (eventType, filename) => {
  console.log(`${eventType}: ${filename}`);
});

// Watch directory recursively (Node.js 10.12+)
fs.watch('dir', { recursive: true }, (eventType, filename) => {
  console.log(`${eventType}: ${filename}`);
});
```

**Directory Operations**

Directory operations include reading, creating, and traversing directory structures. Recursive directory traversal is a common pattern for processing entire project structures. The `fs.readdirSync()` method reads directory contents, while `fs.statSync()` provides file/directory metadata. Combining these with recursion enables processing of nested directories. Understanding directory operations is essential for build tools, file processors, and applications that work with project structures.

```javascript
const fs = require('fs');
const path = require('path');

// Recursively read directory
function readDirRecursive(dir, fileList = []) {
  const files = fs.readdirSync(dir);
  files.forEach(file => {
    const filePath = path.join(dir, file);
    const stat = fs.statSync(filePath);
    if (stat.isDirectory()) {
      readDirRecursive(filePath, fileList);
    } else {
      fileList.push(filePath);
    }
  });
  return fileList;
}

const allFiles = readDirRecursive('./src');
console.log(allFiles);
```

---

## Quick Reference

**Event Loop Priority (Highest to Lowest)**

Understanding event loop priority is crucial for predicting execution order and debugging timing issues. `process.nextTick()` has the highest priority, running between every phase. Promise microtasks run before any macrotask. Timers execute in the Timers phase, followed by I/O callbacks in the Pending phase. `setImmediate()` runs in the Check phase, and close callbacks run last. This hierarchy ensures predictable behavior for asynchronous operations.

1. process.nextTick()
2. Promise microtasks
3. setTimeout/setInterval (Timers phase)
4. I/O callbacks (Pending phase)
5. setImmediate (Check phase)
6. Close callbacks

**Common Async Patterns**

Async patterns in JavaScript have evolved from callbacks to Promises to async/await, each offering improved readability and error handling. The promisify utility converts callback-based functions to Promises. `Promise.all()` executes operations in parallel, while sequential execution uses `await` in a loop. Choosing the right pattern depends on the use case: parallel for independent operations, sequential for dependent operations, and race for timeout scenarios.

```javascript
// Callback → Promise
function promisify(fn) {
  return function(...args) {
    return new Promise((resolve, reject) => {
      fn(...args, (err, result) => {
        err ? reject(err) : resolve(result);
      });
    });
  };
}

// Promise.all() parallel execution
const results = await Promise.all([p1, p2, p3]);

// Promise.race() first to settle
const result = await Promise.race([p1, p2]);

// Sequential execution
for (const item of items) {
  await processItem(item);
}
```

**V8 Optimization Tips**

V8 optimizes JavaScript code based on predictable patterns. Initializing all object properties in the constructor maintains consistent hidden classes, enabling faster property access. Avoiding dynamic property additions prevents hidden class transitions and deoptimization. Using arrays for ordered data and avoiding megamorphic property access (many different object shapes) improves performance. These patterns help V8 generate optimized machine code and avoid costly deoptimizations.

- Initialize all object properties in constructor
- Maintain consistent object shapes
- Avoid adding properties after object creation
- Use arrays for ordered data
- Avoid megamorphic property access

**Node.js Best Practices**

Following Node.js best practices ensures performant, maintainable, and scalable applications. Use async/await over callbacks for cleaner async code. Prefer `fs.promises` over callback-based file operations for better error handling. Use streams for large file operations to avoid memory issues. Avoid blocking the event loop with synchronous operations. Use Worker Threads for CPU-intensive tasks that can't be offloaded to the thread pool. Implement proper error handling in async flows to prevent unhandled promise rejections.

- Use async/await over callbacks
- Prefer fs.promises over callback-based fs
- Use streams for large file operations
- Avoid blocking the event loop (sync operations)
- Use Worker Threads for CPU-intensive tasks
- Implement proper error handling in async flows
