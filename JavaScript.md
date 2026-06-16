<div align="center">

# JavaScript Is Not What You Think

### *The broken, beautiful, and deeply misunderstood parts of JS nobody warns you about*

**Most developers learn JavaScript by copying examples. This is for the ones who want to understand why it actually works — and why it sometimes spectacularly doesn't.**

</div>

## What's Inside

- **Types** — JavaScript's type system is a lie you need to understand
- **Coercion** — The silent transformer that causes 80% of JS bugs
- **`this`** — The most context-dependent keyword in any mainstream language
- **Scope & Closures** — The invisible boxes your variables live in
- **Hoisting** — Code that runs before it exists
- **Prototypes** — The real inheritance system hiding behind `class`
- **Equality** — Why `==` is a minefield and `===` is just safer
- **The Event Loop** — How JS does one thing at a time, asynchronously
- **Promises & Async** — What `await` is actually doing
- **Error Handling** — Why most try/catch is wrong
- **`var` vs `let` vs `const`** — The differences that bite
- **Floating Point** — Why `0.1 + 0.2 !== 0.3`
- **Memory & Leaks** — What you're keeping alive without knowing
- **The Module System** — ESM vs CJS and why it still breaks
- **The Weird Stuff** — `typeof null`, `NaN !== NaN`, and other gifts

---

## 01 — JavaScript Has 8 Types. You're Probably Using 3 of Them.

JavaScript is a dynamically typed language. Variables don't have types — **values do.** There are exactly 8 primitive/reference types.

```js
// The 7 primitive types
typeof undefined    // "undefined"
typeof true         // "boolean"
typeof 42           // "number"
typeof "hello"      // "string"
typeof 42n          // "bigint"
typeof Symbol()     // "symbol"
typeof null         // "object"  ← THE LIE

// The 1 reference type
typeof {}           // "object"
typeof []           // "object"  ← also object
typeof function(){} // "function" ← technically object, special-cased
```

`typeof null === "object"` is a **25-year-old bug** that can never be fixed because it would break the entire web. `null` is not an object. It never was. It was a mistake in the original 10-day implementation of JavaScript that became permanent.

**The right way to check for null:**

```js
// Wrong
typeof value === "object"       // catches null, arrays, everything

// Right
value === null                   // identity check, not type check

// Right for "is this a plain object?"
value !== null && typeof value === "object" && !Array.isArray(value)
```

> Primitives are **copied by value**. Objects (everything else) are **copied by reference**. This single distinction is the source of more JS bugs than anything else.

```js
// Primitives — copied
let a = 5;
let b = a;
b = 10;
console.log(a); // 5 — untouched

// Objects — shared reference
let obj1 = { x: 1 };
let obj2 = obj1;
obj2.x = 99;
console.log(obj1.x); // 99 — you changed the original
```

---

## 02 — Type Coercion: The Silent Saboteur

JavaScript will **automatically convert types** in almost every operation. This is called implicit coercion, and it's responsible for the legendary JavaScript memes — but also for real production bugs.

### The `+` operator is schizophrenic

```js
1 + 1         // 2      — addition
"1" + 1       // "11"   — concatenation (string wins)
1 + "1"       // "11"   — same
1 + true      // 2      — true becomes 1
1 + false     // 1      — false becomes 0
1 + null      // 1      — null becomes 0
1 + undefined // NaN    — undefined becomes NaN
1 + []        // "1"    — [] becomes ""
1 + {}        // "1[object Object]"
```

The rule: **if either operand is a string, `+` becomes concatenation.** For everything else, JS tries to convert to number first.

### The `-`, `*`, `/` operators are more predictable

```js
"5" - 2       // 3    — string coerces to number
"5" * "2"     // 10   — both coerce
"5" - "x"     // NaN  — can't convert "x"
null - 1      // -1   — null → 0
undefined - 1 // NaN  — undefined → NaN
[] - 1        // -1   — [] → "" → 0
```

### Coercion in `if` statements (falsy values)

Only **6 values** are falsy in JavaScript. Everything else is truthy.

```js
// The 6 falsy values
if (false)     { } // never runs
if (0)         { } // never runs — watch out for zero as valid data
if (-0)        { } // never runs — yes, -0 exists
if (0n)        { } // never runs — BigInt zero
if ("")        { } // never runs — empty string
if (null)      { } // never runs
if (undefined) { } // never runs
if (NaN)       { } // never runs

// Everything else is truthy — including these surprises:
if ([])        { } // RUNS — empty array is truthy
if ({})        { } // RUNS — empty object is truthy
if ("0")       { } // RUNS — non-empty string, even "false"
if ("false")   { } // RUNS
```

This trips up every developer at least once:

```js
// Checking if an array is empty — the wrong way
const items = [];
if (!items) console.log("empty"); // Never prints — [] is truthy

// The right way
if (items.length === 0) console.log("empty");
```

### Explicit coercion — always do this

```js
// String to number
Number("42")      // 42
Number("")        // 0    — careful
Number("  ")      // 0    — whitespace too
Number(null)      // 0
Number(undefined) // NaN
Number("abc")     // NaN
parseInt("42px")  // 42   — stops at first non-numeric
parseFloat("3.14")// 3.14

// To string
String(42)        // "42"
String(null)      // "null"
String(undefined) // "undefined"
(42).toString()   // "42"

// To boolean
Boolean(0)        // false
Boolean("")       // false
Boolean(null)     // false
Boolean([])       // true  — remember this one
!!value           // idiomatic double-negation
```

---

## 03 — `this` Is a Runtime Decision, Not a Write-Time One

`this` is the most misunderstood keyword in JavaScript. It doesn't refer to the function. It doesn't refer to where the function was written. It refers to **how and where the function was called**.

### Rule 1: Default binding — `this` is the global object (or undefined in strict mode)

```js
function greet() {
  console.log(this); // window (browser) / global (Node) / undefined (strict)
}
greet();

"use strict";
function strictGreet() {
  console.log(this); // undefined — no implicit global binding in strict mode
}
strictGreet();
```

### Rule 2: Implicit binding — `this` is the object before the dot

```js
const user = {
  name: "Alex",
  greet() {
    console.log(this.name); // "Alex" — this = user
  }
};
user.greet(); // works

// But the moment you detach the function from the object:
const fn = user.greet;
fn(); // undefined or error — this is now global/undefined
```

This is the callback trap that has caught every JavaScript developer:

```js
const timer = {
  seconds: 0,
  start() {
    setInterval(function() {
      this.seconds++; // BUG: this is window/undefined, not timer
    }, 1000);
  }
};
```

### Rule 3: Explicit binding — `call`, `apply`, `bind`

```js
function greet(greeting) {
  console.log(`${greeting}, ${this.name}`);
}

const user = { name: "Alex" };

greet.call(user, "Hello");      // "Hello, Alex" — call with args
greet.apply(user, ["Hello"]);   // "Hello, Alex" — apply with array
const bound = greet.bind(user); // returns new function with this locked
bound("Hello");                  // "Hello, Alex"
```

### Rule 4: `new` binding — `this` is the new object

```js
function Person(name) {
  this.name = name; // this = the new object being created
}
const p = new Person("Alex"); // p.name === "Alex"
```

### Rule 5: Arrow functions — `this` is lexically inherited (never re-bound)

Arrow functions **do not have their own `this`**. They capture `this` from the surrounding scope at write-time. This fixes the callback trap:

```js
const timer = {
  seconds: 0,
  start() {
    setInterval(() => {
      this.seconds++; // this = timer — captured from start()'s scope
    }, 1000);
  }
};

// But arrow functions cannot be constructors or have their this changed:
const arrowFn = () => {};
new arrowFn();         // TypeError
arrowFn.call({ x: 1}); // 'this' inside is still whatever it was lexically
```

**The order of precedence:** `new` > explicit (`bind`/`call`/`apply`) > implicit (object.method) > default (global/undefined)

---

## 04 — Scope: The Invisible Boxes Variables Live In

JavaScript uses **lexical scope** — the scope of a variable is determined by where it's written in the source code, not where it runs.

### Function scope vs. block scope

```js
// var is function-scoped
function example() {
  if (true) {
    var x = 1; // lives in the function, not the block
  }
  console.log(x); // 1 — x leaked out of the if block
}

// let and const are block-scoped
function example2() {
  if (true) {
    let y = 1;
    const z = 2;
  }
  console.log(y); // ReferenceError — y doesn't exist here
  console.log(z); // ReferenceError — z doesn't exist here
}
```

### Closures: functions that remember their birth environment

A closure is a function that retains access to its lexical scope even when executed outside that scope. This is not a trick. It's a fundamental feature.

```js
function makeCounter() {
  let count = 0; // this variable is captured by the returned function

  return function() {
    count++;
    return count;
  };
}

const counter = makeCounter();
counter(); // 1
counter(); // 2
counter(); // 3
// count is private — nothing outside can touch it
```

### The classic loop closure bug

```js
// The famous bug — everyone writes this at some point
for (var i = 0; i < 3; i++) {
  setTimeout(function() {
    console.log(i); // prints 3, 3, 3 — not 0, 1, 2
  }, 100);
}
// By the time the callbacks run, the loop is done and i = 3
// All three callbacks share the SAME i (var is function-scoped)

// Fix 1: use let (creates new binding per iteration)
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // 0, 1, 2 ✓
}

// Fix 2: IIFE to capture current value
for (var i = 0; i < 3; i++) {
  (function(j) {
    setTimeout(() => console.log(j), 100); // 0, 1, 2 ✓
  })(i);
}
```

### Closure memory — the thing nobody thinks about

Closures keep the entire outer scope alive as long as the inner function lives. This is useful, but it's also how memory leaks happen.

```js
function createLeak() {
  const massiveArray = new Array(1000000).fill("data");

  return function() {
    // Only uses one value, but massiveArray stays in memory
    // because this function closes over the whole scope
    return massiveArray[0];
  };
}
```

---

## 05 — Hoisting: Code That Runs Before It Exists

JavaScript processes declarations before executing code. The result is that some variables and functions seem to exist before they're written.

### Function declarations are fully hoisted

```js
greet(); // "Hello" — works even before the declaration

function greet() {
  console.log("Hello");
}
```

### `var` is hoisted but not initialized

```js
console.log(name); // undefined — hoisted but not yet assigned
var name = "Alex";
console.log(name); // "Alex"

// What the engine actually sees:
var name;          // declaration hoisted to top of function
console.log(name); // undefined
name = "Alex";     // assignment stays in place
console.log(name); // "Alex"
```

### `let` and `const` are hoisted but not accessible — the Temporal Dead Zone

```js
console.log(x); // ReferenceError: Cannot access 'x' before initialization
let x = 5;
```

This is called the **Temporal Dead Zone (TDZ)** — the period between the start of the block and the declaration line. The variable exists (it's been hoisted) but cannot be accessed. This is intentional and better than `var`'s silent `undefined`.

```js
// The TDZ is a region, not just the line:
{
  // TDZ for 'x' starts here
  typeof x; // ReferenceError — even typeof isn't safe here
  // TDZ for 'x' ends here ↓
  let x = 5;
}
```

### Class declarations are also in the TDZ

```js
const p = new Person(); // ReferenceError — class not initialized yet
class Person {}
```

---

## 06 — Prototypes: The Real Inheritance System

JavaScript does not have classical inheritance. It has **prototypal inheritance** — objects delegate to other objects. The `class` keyword introduced in ES6 is syntactic sugar over this system, not a replacement for it.

### Every object has a prototype

```js
const obj = {};
Object.getPrototypeOf(obj); // → Object.prototype

const arr = [];
Object.getPrototypeOf(arr); // → Array.prototype
// Array.prototype's prototype is Object.prototype
// Object.prototype's prototype is null — end of chain
```

### Property lookup walks the chain

```js
const animal = {
  breathe() { return "breathing"; }
};

const dog = Object.create(animal); // dog's prototype is animal
dog.bark = function() { return "woof"; };

dog.bark();    // "woof"    — found on dog directly
dog.breathe(); // "breathing" — found by walking up to animal
dog.toString() // "[object Object]" — found on Object.prototype

// "dog" itself has no breathe or toString — the engine climbs the chain
```

### What `class` actually compiles to

```js
// What you write
class Animal {
  constructor(name) { this.name = name; }
  speak() { return `${this.name} speaks`; }
}
class Dog extends Animal {
  bark() { return "woof"; }
}

// What it means under the hood
function Animal(name) { this.name = name; }
Animal.prototype.speak = function() { return `${this.name} speaks`; };

function Dog(name) { Animal.call(this, name); }
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;
Dog.prototype.bark = function() { return "woof"; };
```

Understanding this matters when:
- You debug inheritance issues in devtools
- You write code that checks `instanceof`
- You need to extend built-ins like `Array` or `Error`
- You work with frameworks that manipulate prototypes directly

### Prototype mutation is dangerous and powerful

```js
// Adding to a built-in prototype — generally a bad idea
Array.prototype.sum = function() {
  return this.reduce((a, b) => a + b, 0);
};
[1, 2, 3].sum(); // 6 — works everywhere

// The problem: it shows up in for...in loops
for (let key in [1, 2, 3]) {
  console.log(key); // "0", "1", "2", "sum" — unexpected
}
// This is why for...in on arrays is dangerous
```

---

## 07 — Equality: Why `==` Is a Decision, Not a Typo

JavaScript has two equality operators. They are not interchangeable.

### `===` (strict equality) — no coercion, always safe

```js
1 === 1       // true
1 === "1"     // false — different types
null === null // true
null === undefined // false
NaN === NaN   // false — NaN is not equal to anything, including itself
```

### `==` (abstract equality) — coerces types first, then compares

```js
1 == "1"      // true — string coerces to number
0 == false    // true — false coerces to 0
0 == ""       // true — "" coerces to 0
"" == false   // true — both become 0
null == undefined // true — special case in the spec
null == 0     // false — null only equals undefined with ==
null == false // false — null only equals undefined
[] == false   // true — [] → "" → 0, false → 0
[] == 0       // true
[] == ""      // true
[] == []      // false — different references
```

The algorithm for `==` is 12 steps in the ECMAScript spec. Most developers can't recite them. That's the argument for always using `===`.

### The NaN problem

`NaN` (Not a Number) is the result of an invalid numeric operation. It has a unique property: **it is the only value in JavaScript not equal to itself.**

```js
NaN === NaN   // false
NaN == NaN    // false

// How to actually check for NaN
Number.isNaN(NaN)       // true — strict, only true for actual NaN
Number.isNaN("hello")   // false — does NOT coerce
isNaN("hello")          // true — global isNaN coerces first (dangerous)
isNaN(undefined)        // true — converts to NaN first

// The weird one
Object.is(NaN, NaN)  // true — Object.is has special NaN handling
Object.is(0, -0)     // false — Object.is also distinguishes 0 and -0
0 === -0             // true  — === does not
```

---

## 08 — The Event Loop: One Thread, Zero Blocking

JavaScript is **single-threaded** — it can only do one thing at a time. But it handles async operations (network, timers, I/O) without freezing. Understanding how is not optional.

### The architecture

```
  Call Stack           Web APIs              Task Queue
  ┌─────────┐         ┌──────────────┐       ┌──────────────┐
  │ main()  │  ──────▶│ setTimeout   │──────▶│ callback1()  │
  │ greet() │         │ fetch()      │       │ callback2()  │
  │ ...     │         │ addEventListener│     └──────────────┘
  └─────────┘         └──────────────┘
        ▲                                            │
        └────────────────────────────────────────────┘
                     Event Loop watches:
                  "Is the call stack empty? 
                   Then push next task."
```

### The Microtask Queue — higher priority than tasks

```js
// Task queue (macrotasks): setTimeout, setInterval, I/O
// Microtask queue: Promise callbacks, queueMicrotask

console.log("1 - synchronous");

setTimeout(() => console.log("2 - setTimeout"), 0);

Promise.resolve().then(() => console.log("3 - promise"));

queueMicrotask(() => console.log("4 - microtask"));

console.log("5 - synchronous");

// Output order: 1, 5, 3, 4, 2
// Why: sync first, then ALL microtasks drain before ANY macrotask runs
```

### Blocking the thread — what "never do this" looks like

```js
// This freezes the entire browser tab:
function blockFor(ms) {
  const end = Date.now() + ms;
  while (Date.now() < end) {} // synchronous spin — nothing else can run
}

blockFor(3000); // UI frozen, network stalled, events queued
```

### setTimeout(fn, 0) is not "immediate"

```js
setTimeout(() => console.log("after"), 0);
console.log("before");

// Output: "before", "after"
// 0ms doesn't mean right now — it means "as soon as the stack is clear
// and any queued microtasks have drained"
// The actual minimum delay is ~4ms in most browsers
```

---

## 09 — Promises, Async/Await, and What's Actually Happening

Promises represent an eventual value. `async/await` is syntax sugar over Promises. Neither is magic — understanding one means understanding both.

### Promise states

```js
// A Promise is always in one of three states:
// pending   — initial, not yet resolved or rejected
// fulfilled — operation succeeded, has a value
// rejected  — operation failed, has a reason

const p = new Promise((resolve, reject) => {
  // resolve(value)  → fulfills the promise
  // reject(reason)  → rejects the promise
  // calling both is ignored — only first call matters
  resolve(42);
  reject("error"); // ignored
});
```

### Promise chaining — the key mental model

```js
fetch("/api/user")
  .then(response => response.json())    // returns a new Promise
  .then(user => user.name)             // transforms the value
  .then(name => console.log(name))     // consumes the value
  .catch(err => console.error(err))    // catches any error in the chain
  .finally(() => console.log("done")); // always runs, passes value through
```

Each `.then()` returns a **new Promise**. Errors skip `.then()` handlers and fall through to the nearest `.catch()`.

### async/await desugared

```js
// This:
async function getUser() {
  const response = await fetch("/api/user");
  const user = await response.json();
  return user.name;
}

// Is equivalent to this:
function getUser() {
  return fetch("/api/user")
    .then(response => response.json())
    .then(user => user.name);
}
```

`await` pauses the **current async function** only. The rest of the program keeps running. It does not block the thread.

### The common async mistakes

```js
// Mistake 1: forgetting await
async function getData() {
  const data = fetch("/api");   // Bug: data is a Promise, not a Response
  console.log(data);            // Promise { <pending> }
}

// Mistake 2: await in a regular forEach — it doesn't work
async function processAll(items) {
  items.forEach(async (item) => {
    await doWork(item); // awaits inside the callback, NOT in processAll
  });
  // processAll finishes immediately — doesn't wait for forEach callbacks
}

// Fix: use for...of
async function processAll(items) {
  for (const item of items) {
    await doWork(item); // now awaited in sequence
  }
}

// Or parallel with Promise.all:
async function processAll(items) {
  await Promise.all(items.map(item => doWork(item))); // concurrent
}

// Mistake 3: unhandled promise rejections
async function risky() {
  throw new Error("failure");
}
risky(); // UnhandledPromiseRejection — no try/catch, no .catch()
```

### Promise combinators

```js
// All must succeed — rejects on first failure
await Promise.all([fetch("/a"), fetch("/b"), fetch("/c")]);

// First to settle wins (resolve or reject)
await Promise.race([fetch("/fast"), fetch("/slow")]);

// All settle regardless — gives results array with status + value/reason
await Promise.allSettled([fetch("/a"), fetch("/may-fail")]);

// First to RESOLVE wins — rejects only if ALL reject
await Promise.any([fetch("/a"), fetch("/b")]);
```

---

## 10 — Error Handling: Most try/catch Is Wrong

JavaScript errors are objects. They have a `message`, `name`, `stack`, and sometimes more. Most code handles them incorrectly.

### The silent swallow — the worst pattern

```js
// This is the worst thing you can do:
try {
  doSomething();
} catch (e) {
  // silently ignored
}
```

Errors disappear. Bugs become invisible. Don't do this.

### Catching what you expect, not everything

```js
// Better pattern: only catch errors you can handle
try {
  JSON.parse(userInput);
} catch (e) {
  if (e instanceof SyntaxError) {
    return { error: "Invalid JSON" }; // expected — handle it
  }
  throw e; // unexpected — rethrow, let it surface
}
```

### Custom error types — underused and extremely useful

```js
class NetworkError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.name = "NetworkError";
    this.statusCode = statusCode;
  }
}

class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = "ValidationError";
    this.field = field;
  }
}

// Now you can handle errors by type:
try {
  await fetchUser(id);
} catch (e) {
  if (e instanceof NetworkError && e.statusCode === 404) {
    return null; // user not found — expected
  }
  if (e instanceof ValidationError) {
    showFieldError(e.field, e.message);
    return;
  }
  throw e; // unknown — rethrow
}
```

### Error handling with async/await

```js
// Pattern 1: try/catch
async function getUser(id) {
  try {
    const user = await fetchUser(id);
    return user;
  } catch (e) {
    console.error("Failed:", e);
    return null;
  }
}

// Pattern 2: promise .catch at the callsite (sometimes cleaner)
const user = await fetchUser(id).catch(() => null);

// Pattern 3: Go-style tuple (explicit, no exception flow)
async function safe(promise) {
  try {
    return [null, await promise];
  } catch (e) {
    return [e, null];
  }
}
const [err, user] = await safe(fetchUser(id));
if (err) { /* handle */ }
```

### The error stack is gold — preserve it

```js
// DON'T wrap and throw a new generic error:
try {
  await doWork();
} catch (e) {
  throw new Error("Something went wrong"); // original stack is gone
}

// DO add context and preserve the cause:
try {
  await doWork();
} catch (e) {
  throw new Error("Failed during doWork", { cause: e }); // ES2022 cause
}
// error.cause now points to the original error with its original stack
```

---

## 11 — `var` vs `let` vs `const`: The Differences That Bite

These are not stylistic choices. They have meaningfully different behavior.

| Feature | `var` | `let` | `const` |
|---|---|---|---|
| Scope | Function | Block | Block |
| Hoisted | Yes (as `undefined`) | Yes (TDZ) | Yes (TDZ) |
| Re-declarable | Yes | No | No |
| Re-assignable | Yes | Yes | No |
| Global property | Yes | No | No |

```js
// var leaks out of blocks
{
  var x = 1;
  let y = 2;
  const z = 3;
}
console.log(x); // 1 — var escaped
console.log(y); // ReferenceError
console.log(z); // ReferenceError

// var attaches to window (browser)
var globalVar = "I'm global";
window.globalVar; // "I'm global"

let blockVar = "I'm not";
window.blockVar; // undefined

// const is NOT deeply immutable
const obj = { x: 1 };
obj.x = 99;     // works — you changed the object, not the binding
obj = {};        // TypeError — you tried to change the binding

// For deep immutability:
const frozen = Object.freeze({ x: 1 });
frozen.x = 99;  // silently fails (or throws in strict mode)
frozen.x;       // still 1
// Note: Object.freeze is shallow — nested objects are still mutable
```

---

## 12 — Floating Point: Why `0.1 + 0.2 !== 0.3`

This isn't a JavaScript bug. It's an IEEE 754 floating-point arithmetic reality that exists in virtually every programming language.

```js
0.1 + 0.2         // 0.30000000000000004
0.1 + 0.2 === 0.3 // false

// Why? Numbers are stored in binary.
// 0.1 in binary is 0.0001100110011... (infinite repeating)
// Same for 0.2. The rounding error compounds.

// The safe comparison pattern:
Math.abs((0.1 + 0.2) - 0.3) < Number.EPSILON // true

// For money: NEVER use floats
// 0.1 dollars is not representable in binary
// Use integers (cents) or a decimal library
const price = 199; // store as cents
const display = (price / 100).toFixed(2); // "1.99" for display only

// Other float surprises:
Number.MAX_SAFE_INTEGER      // 9007199254740991 (2^53 - 1)
9007199254740991 + 1         // 9007199254740992 — ok
9007199254740991 + 2         // 9007199254740992 — wrong, same as +1!
// Beyond MAX_SAFE_INTEGER, integers lose precision

// BigInt for large integers:
9007199254740991n + 2n       // 9007199254740993n — correct
```

---

## 13 — Memory & Leaks: What You're Keeping Alive Without Knowing

JavaScript has automatic garbage collection, but the GC can only free memory it knows is no longer referenced. It's your job to stop holding references to things you're done with.

### Common leak 1: Forgotten timers

```js
// This interval keeps running and holds a reference to data forever
function startPolling(data) {
  setInterval(() => {
    processData(data); // data stays in memory as long as interval runs
  }, 1000);
}
// If you never call clearInterval(), data never gets freed

// Correct:
function startPolling(data) {
  const id = setInterval(() => {
    processData(data);
  }, 1000);
  return () => clearInterval(id); // return cleanup function
}
```

### Common leak 2: Event listeners never removed

```js
// Each call to attachHandler adds a new listener, never removed
function attachHandler(element) {
  element.addEventListener("click", () => {
    console.log("clicked");
  });
}
// If element is removed from DOM but JS holds a reference to it,
// neither element nor its listeners get garbage collected

// Correct:
const handler = () => console.log("clicked");
element.addEventListener("click", handler);
// When done:
element.removeEventListener("click", handler);
// Or use AbortController:
const controller = new AbortController();
element.addEventListener("click", handler, { signal: controller.signal });
controller.abort(); // removes all listeners added with this signal
```

### Common leak 3: Closures holding large objects

```js
function processImage(imageData) {
  // imageData is potentially huge
  const width = imageData.width;
  const height = imageData.height;

  return function getInfo() {
    return `${width}x${height}`;
    // This closure holds ALL of imageData in memory, not just width/height
    // Because the closure captures the entire outer scope
  };
}

// The fact that imageData is never referenced inside getInfo
// doesn't matter — the entire scope is kept alive
```

### WeakMap and WeakRef — for when you want GC to decide

```js
// WeakMap keys are held weakly — if key has no other references, GC can free it
const cache = new WeakMap();

function processUser(user) {
  if (cache.has(user)) return cache.get(user);
  const result = expensiveComputation(user);
  cache.set(user, result);
  return result;
}
// When user object is garbage collected, its cache entry is automatically removed
// This would cause a memory leak with a regular Map
```

---

## 14 — The Module System: ESM vs CJS and Why It Still Hurts

JavaScript has two module systems. They are not fully compatible. This is a source of ongoing pain in the ecosystem.

### CommonJS (CJS) — Node's original, synchronous

```js
// Exporting
const name = "Alex";
module.exports = { name };
// or
exports.name = "Alex";

// Importing
const { name } = require("./module");
const fs = require("fs"); // works synchronously

// require() is a runtime function call — dynamic by nature
const moduleName = condition ? "a" : "b";
const m = require(moduleName); // perfectly valid
```

### ES Modules (ESM) — the standard, static, asynchronous

```js
// Exporting
export const name = "Alex";
export default function greet() {}
export { name, greet };

// Importing
import { name } from "./module.js"; // .js extension required in ESM
import greet from "./module.js";    // default import
import * as utils from "./module.js"; // namespace import

// Dynamic import — async, returns a Promise
const { name } = await import("./module.js");
```

### The key differences that cause headaches

```js
// CJS exports a copy of primitive values at export time
// ESM exports a live binding — the value can change

// module.js (CJS)
let count = 0;
module.exports = { count, increment: () => count++ };

const { count, increment } = require("./module.js");
increment();
console.log(count); // 0 — you got a copy, not a live binding

// module.js (ESM)
export let count = 0;
export const increment = () => count++;

import { count, increment } from "./module.js";
increment();
console.log(count); // 1 — live binding, reflects the update

// CJS in ESM files — the interop problem
import cjsModule from "./legacy.cjs";   // works — gets module.exports
import { name } from "./legacy.cjs";    // might fail — named imports from CJS are not guaranteed
// This is why you see "default import interop" issues all the time
```

---

## 15 — The Weird Parts: A Catalog of JavaScript Oddities

These are real, they exist in production code, and they're worth knowing.

```js
// NaN is a number
typeof NaN           // "number"
Number.isNaN(NaN)   // true — it is NaN and it is "of type number"

// -0 exists
-0 === 0             // true — === can't tell them apart
Object.is(-0, 0)     // false — Object.is can
(-0).toString()      // "0" — hides its sign when stringified
1 / -0               // -Infinity — reveals itself under division

// typeof before declaration is safe for var but not let/const
typeof undeclaredVar  // "undefined" — no error (for var/undeclared)
typeof myLet          // ReferenceError if in TDZ (for let/const)

// Array holes
const arr = [1,,3];  // array with a hole at index 1
arr[1]               // undefined
1 in arr             // false — the index doesn't exist
arr.map(x => x * 2) // [2, empty, 6] — map skips holes
arr.join(",")        // "1,,3"
Array(3)             // [empty × 3] — all holes

// Automatic semicolon insertion (ASI)
function getValue() {
  return        // ASI inserts semicolon here!
  {             // this block is unreachable
    value: 42
  }
}
getValue()       // undefined — return statement was terminated early
// Fix: opening brace must be on the same line as return

// arguments object — only in non-arrow functions
function logArgs() {
  console.log(arguments); // array-like but not an array
  console.log(arguments[0]);
  // Array.from(arguments) to convert to real array
  // Rest params are better: function logArgs(...args)
}

// with statement — never use, exists for historical reasons
with (obj) {     // adds obj to scope chain
  console.log(x); // is x from obj or from outer scope? unclear
}
// Banned in strict mode — this is the right call

// eval — executes a string as code
eval("2 + 2"); // 4 — but introduces security, performance, scope nightmares
// Banned by many linters and CSP policies. Never use.

// delete on variables does nothing
var x = 1;
delete x;    // false — can't delete variable declarations
x;           // still 1

delete window.someGlobalVar; // true — can delete global properties added without var/let/const

// in operator checks property existence (including prototype chain)
"toString" in {};  // true — it's on Object.prototype
"length" in [];    // true
0 in [1, 2, 3];    // true — checks if index 0 exists

// instanceof crosses context boundaries poorly
[] instanceof Array              // true — in same window/realm
// In iframes:
iframeArray instanceof Array     // false — different Array constructor!
// Safer:
Array.isArray(iframeArray)       // true — works across realms

// Sparse array length
const a = [];
a[99] = "hello";
a.length;         // 100
a[0];             // undefined
```

---

## 16 — Strict Mode: The Better JavaScript Hidden Inside JavaScript

`"use strict"` opts into a more restrictive and sane version of JavaScript. ES6 modules and `class` bodies are always in strict mode. Outside of those, you opt in.

```js
"use strict";

// 1. Undeclared variables throw instead of creating globals
x = 5;                    // ReferenceError — x is not declared

// 2. Writing to read-only properties throws
const obj = Object.freeze({ x: 1 });
obj.x = 2;               // TypeError — silent fail in sloppy mode

// 3. Duplicate parameter names banned
function fn(a, a) {}     // SyntaxError — valid in sloppy mode

// 4. this is undefined in unbound functions (not global)
function test() {
  console.log(this);     // undefined — not window
}
test();

// 5. with statement is banned
with (obj) {}            // SyntaxError

// 6. delete on undeletable properties throws
delete Object.prototype; // TypeError — silent fail in sloppy mode

// 7. arguments doesn't track parameter mutations
function fn(x) {
  x = 99;
  console.log(arguments[0]); // in sloppy: 99 (linked). In strict: original value
}
```

Always use strict mode. Or use ES modules/TypeScript which enforce it automatically.

---

## 17 — JavaScript Is a Design Accident Running the World

JavaScript was written in 10 days in May 1995. It was not designed to be the primary language of a global software platform. It was designed to make buttons on web pages slightly more interactive.

Then the world ran it at scale anyway.

That history explains everything that feels wrong about it:
- `null` being an object — unfixed from the prototype
- `this` changing based on call site — Java-influenced late-binding
- `var` leaking out of blocks — C-influenced syntax without C semantics
- Implicit coercion — designed for HTML authors, not engineers
- Prototype-based inheritance — Scheme-influenced, hidden under `class` sugar
- Two module systems — CommonJS emerged when native modules didn't exist; ESM arrived 20 years later

None of this was malice. It was urgency, influence from multiple languages, and a spec that had to stay backward compatible with every previous mistake forever.

> **The web can never have a breaking change.** Every decision made in 1995 is locked in by billions of pages that still load today. JavaScript gets better by adding, never by removing.

That constraint produced one of the most resilient, flexible, and unpredictable platforms ever built.

---


## 18 — Destructuring: Unpacking Is a Full Language Feature

Destructuring looks like a shorthand. It's actually a pattern-matching system with its own rules, defaults, renaming, and nesting.

### Array destructuring

```js
const [a, b, c] = [1, 2, 3];
// a = 1, b = 2, c = 3

// Skip elements with empty commas
const [, second, , fourth] = [1, 2, 3, 4];
// second = 2, fourth = 4

// Default values
const [x = 10, y = 20] = [5];
// x = 5, y = 20 — y used its default because undefined was provided

// Rest element — must be last
const [first, ...rest] = [1, 2, 3, 4];
// first = 1, rest = [2, 3, 4]

// Swap variables without a temp
let m = 1, n = 2;
[m, n] = [n, m];
// m = 2, n = 1
```

### Object destructuring

```js
const { name, age } = { name: "Alex", age: 30, role: "dev" };
// name = "Alex", age = 30 — role is ignored

// Rename while destructuring
const { name: userName, age: userAge } = user;
// userName = "Alex", userAge = 30
// name and age are NOT created as variables

// Default values
const { score = 0, level = 1 } = { score: 42 };
// score = 42, level = 1

// Rename AND default
const { title: postTitle = "Untitled" } = {};
// postTitle = "Untitled"

// Rest — grab remaining properties
const { name: n, ...rest } = { name: "Alex", age: 30, role: "dev" };
// n = "Alex", rest = { age: 30, role: "dev" }
```

### Nested destructuring — powerful but easy to overdo

```js
const { address: { city, zip } } = {
  address: { city: "Berlin", zip: "10115" }
};
// city = "Berlin", zip = "10115"
// WARNING: if address is undefined, this throws

// Safe nested destructure
const { address: { city } = {} } = user;
// If address is missing, {} is used, city = undefined — no throw
```

### Destructuring in function parameters — the most common use

```js
// Before destructuring:
function greet(user) {
  console.log(user.name, user.age);
}

// After — cleaner, self-documenting:
function greet({ name, age }) {
  console.log(name, age);
}

// With defaults:
function createUser({ name = "Anonymous", role = "viewer" } = {}) {
  return { name, role };
}
createUser();            // { name: "Anonymous", role: "viewer" }
createUser({ name: "Alex" }); // { name: "Alex", role: "viewer" }
// The = {} at the end means the whole argument is optional
```

> Destructuring runs at the assignment site. If the source value is `undefined`, defaults kick in. If the source is `null`, it throws — `null` cannot be destructured.

---

## 19 — Spread and Rest: The Three Dots That Do Two Different Things

`...` is two operators depending on context. Rest *collects* values into an array. Spread *expands* an iterable into individual values.

### Spread

```js
// Spread arrays
const a = [1, 2, 3];
const b = [4, 5, 6];
const combined = [...a, ...b]; // [1, 2, 3, 4, 5, 6]
const copy = [...a];           // shallow copy — not the same reference

// Spread into function arguments
Math.max(...[1, 5, 3]); // 5 — equivalent to Math.max(1, 5, 3)

// Spread objects (ES2018)
const defaults = { color: "blue", size: "md" };
const overrides = { size: "lg", weight: "bold" };
const merged = { ...defaults, ...overrides };
// { color: "blue", size: "lg", weight: "bold" }
// Later keys overwrite earlier keys

// Shallow copy an object
const copy = { ...original };

// Add or override a specific key
const updated = { ...user, age: 31 };
```

### Rest parameters — the function side

```js
function sum(...numbers) {
  return numbers.reduce((a, b) => a + b, 0);
}
sum(1, 2, 3, 4, 5); // 15

// Rest must be the last parameter
function log(label, ...messages) {
  messages.forEach(m => console.log(`[${label}] ${m}`));
}

// The difference from arguments:
// arguments: array-like, all args, exists in non-arrow functions
// rest params: real array, only uncaptured args, works everywhere
```

### What spread doesn't do: deep copy

```js
const original = { a: { b: 1 } };
const copy = { ...original };

copy.a.b = 99;
console.log(original.a.b); // 99 — you mutated the original

// Spread is one level deep. For deep copies:
structuredClone(original); // ES2022 — the correct answer for most cases
JSON.parse(JSON.stringify(original)); // works, but loses Date/undefined/functions
```

---

## 20 — Optional Chaining and Nullish Coalescing: The Safety Operators

These two operators were added in ES2020 and immediately became essential. They solve problems every developer was solving manually.

### `?.` — Optional chaining

Stops evaluation and returns `undefined` instead of throwing when it hits `null` or `undefined`.

```js
// Before optional chaining:
const city = user && user.address && user.address.city;

// After:
const city = user?.address?.city;

// Works on methods:
const name = user?.getName?.(); // calls only if getName exists

// Works on bracket notation:
const first = arr?.[0];
const val = obj?.["dynamic-key"];

// Works with function calls:
callback?.(); // calls callback only if it's not null/undefined

// The short-circuit is the key feature:
// user?.address?.city
// If user is null: returns undefined immediately, never accesses .address
// If user.address is null: returns undefined, never accesses .city
```

### `??` — Nullish coalescing

Returns the right side only when the left side is `null` or `undefined`. Unlike `||`, it does NOT trigger on `0`, `""`, or `false`.

```js
// || fires on ANY falsy value — often wrong:
const port = config.port || 3000;
// Problem: if config.port is 0 (a valid port), you get 3000 instead

// ?? fires only on null/undefined — usually what you want:
const port = config.port ?? 3000;
// If config.port is 0, you get 0. If it's null/undefined, you get 3000.

// More examples:
""    ?? "default"  // ""    — empty string is kept
0     ?? 42         // 0     — zero is kept
false ?? true       // false — false is kept
null  ?? "fallback" // "fallback"
undefined ?? "fallback" // "fallback"
```

### `??=`, `||=`, `&&=` — Logical assignment (ES2021)

```js
// Assign only if null/undefined
user.name ??= "Anonymous";
// Equivalent to: user.name = user.name ?? "Anonymous"

// Assign only if falsy
config.debug ||= false;

// Assign only if truthy
obj.cache &&= transform(obj.cache);
```

---

## 21 — Symbols: The Invisible Keys

`Symbol` is a primitive type that creates a guaranteed-unique value. Every Symbol is unique, even if it has the same description.

```js
const id = Symbol("id");
const id2 = Symbol("id");
id === id2; // false — always unique

// Symbols as object keys
const user = {
  name: "Alex",
  [id]: 12345     // Symbol as computed key
};

user[id];        // 12345
user.id;         // undefined — string "id" !== Symbol("id")

// Symbols don't show up in normal enumeration
Object.keys(user);   // ["name"] — no symbol
JSON.stringify(user); // '{"name":"Alex"}' — symbol dropped

// To find symbol keys:
Object.getOwnPropertySymbols(user); // [Symbol(id)]
```

### Why use Symbols?

```js
// 1. Truly private-ish keys — not accessible without the symbol reference
const _private = Symbol("private");
class MyClass {
  constructor() {
    this[_private] = "internal state";
  }
}

// 2. Prevent key collisions in shared objects
// Library A and Library B can both add a "meta" key without conflicting:
const metaA = Symbol("meta");
const metaB = Symbol("meta");
obj[metaA] = "library A data";
obj[metaB] = "library B data";

// 3. Well-known Symbols — hooks into built-in JS behavior
class Range {
  constructor(start, end) { this.start = start; this.end = end; }

  [Symbol.iterator]() {     // makes Range iterable (for...of, spread)
    let current = this.start;
    const end = this.end;
    return {
      next() {
        return current <= end
          ? { value: current++, done: false }
          : { value: undefined, done: true };
      }
    };
  }
}

for (const n of new Range(1, 5)) {
  console.log(n); // 1 2 3 4 5
}
[...new Range(1, 3)]; // [1, 2, 3]
```

### Well-known Symbols — the JS protocol hooks

```js
Symbol.iterator    // makes object iterable
Symbol.toPrimitive // controls type coercion
Symbol.hasInstance // controls instanceof behavior
Symbol.toStringTag // changes Object.prototype.toString output

class Money {
  constructor(amount, currency) {
    this.amount = amount;
    this.currency = currency;
  }

  [Symbol.toPrimitive](hint) {
    if (hint === "number") return this.amount;
    if (hint === "string") return `${this.amount} ${this.currency}`;
    return this.amount;
  }
}

const price = new Money(42, "USD");
+price;          // 42     (number hint)
`${price}`;      // "42 USD" (string hint)
price + 10;      // 52 (default hint → number)
```

---

## 22 — Iterators and Generators: Custom Sequences

Any object with a `[Symbol.iterator]` method that returns an iterator is iterable. `for...of`, spread, destructuring, and `Array.from` all use this protocol.

### The iterator protocol

```js
// An iterator is an object with a next() method
const iterator = {
  values: [10, 20, 30],
  index: 0,
  next() {
    if (this.index < this.values.length) {
      return { value: this.values[this.index++], done: false };
    }
    return { value: undefined, done: true };
  }
};

iterator.next(); // { value: 10, done: false }
iterator.next(); // { value: 20, done: false }
iterator.next(); // { value: 30, done: false }
iterator.next(); // { value: undefined, done: true }
```

### Generators: the shorthand for iterators

A generator function returns a generator object — an iterator you can pause and resume with `yield`.

```js
function* range(start, end) {
  for (let i = start; i <= end; i++) {
    yield i; // pause here, hand value to caller, resume next time
  }
}

for (const n of range(1, 5)) {
  console.log(n); // 1, 2, 3, 4, 5
}

// Generators are lazy — nothing runs until you pull a value
function* infinite() {
  let i = 0;
  while (true) yield i++;
}

const gen = infinite();
gen.next().value; // 0
gen.next().value; // 1
gen.next().value; // 2
// This never exhausts — you pull values one at a time
```

### Generators can receive values back

```js
function* conversation() {
  const name = yield "What's your name?";
  const age = yield `Hi ${name}! How old are you?`;
  return `${name} is ${age} years old.`;
}

const gen = conversation();
gen.next();           // { value: "What's your name?", done: false }
gen.next("Alex");     // { value: "Hi Alex! How old are you?", done: false }
gen.next(30);         // { value: "Alex is 30 years old.", done: true }
```

### Practical uses

```js
// Infinite IDs
function* idGenerator() {
  let id = 1;
  while (true) yield id++;
}
const nextId = idGenerator();
nextId.next().value; // 1
nextId.next().value; // 2

// Paginated data fetching — pull pages lazily
async function* fetchPages(url) {
  let page = 1;
  while (true) {
    const data = await fetch(`${url}?page=${page}`).then(r => r.json());
    if (data.length === 0) return;
    yield data;
    page++;
  }
}

for await (const page of fetchPages("/api/posts")) {
  process(page);
}
```

---

## 23 — Map and Set: The Data Structures You Should Use More

Most JavaScript code uses plain objects for everything. `Map` and `Set` exist for good reasons and should be used more often than they are.

### Map — keys of any type, insertion-ordered

```js
// Object keys are always strings (or Symbols)
// Map keys can be anything — objects, functions, primitives

const map = new Map();
const keyObj = { id: 1 };

map.set(keyObj, "user data"); // object as key
map.set(42, "the answer");
map.set(true, "boolean key");

map.get(keyObj);   // "user data"
map.get(42);       // "the answer"
map.size;          // 3

// Iteration — always in insertion order
for (const [key, value] of map) {
  console.log(key, value);
}

// Convert from/to object
const fromObj = new Map(Object.entries({ a: 1, b: 2 }));
Object.fromEntries(map); // back to object (only works for string keys)
```

### When to use Map instead of a plain object:

```js
// Use Map when:
// 1. Keys are not strings
const nodeData = new Map(); // DOM node → metadata
nodeData.set(document.querySelector("button"), { clicks: 0 });

// 2. Key insertion order matters and you need to iterate
// Object key order is not reliable in all JS engines for non-string keys

// 3. You need to frequently check size
map.size; // O(1)
Object.keys(obj).length; // O(n) — iterates all keys to count

// 4. You're adding and removing keys frequently
// Objects are optimized for known shapes — lots of dynamic
// add/delete can cause performance issues in V8

// Use plain object when:
// Keys are all strings, shape is known upfront, JSON serialization needed
```

### Set — unique values, any type

```js
const set = new Set([1, 2, 3, 2, 1]);
// Set {1, 2, 3} — duplicates removed automatically

set.add(4);
set.has(2);    // true
set.delete(1);
set.size;      // 3

// Deduplication pattern
const unique = [...new Set(array)];

// Set iteration is insertion-ordered
for (const value of set) {
  console.log(value);
}

// Set operations (not built-in, but easy to implement)
const a = new Set([1, 2, 3, 4]);
const b = new Set([3, 4, 5, 6]);

const union        = new Set([...a, ...b]);        // {1,2,3,4,5,6}
const intersection = new Set([...a].filter(x => b.has(x))); // {3,4}
const difference   = new Set([...a].filter(x => !b.has(x))); // {1,2}
```

### WeakMap and WeakSet — GC-friendly storage

```js
// WeakMap: keys must be objects, held weakly (GC can collect them)
// WeakSet: values must be objects, held weakly
// Neither is iterable — you can't list the contents

const seen = new WeakSet();

function processOnce(obj) {
  if (seen.has(obj)) return;
  seen.add(obj);
  doWork(obj);
  // When obj has no other references, WeakSet releases it automatically
}

// Perfect for private instance data without memory leaks
const _private = new WeakMap();

class Counter {
  constructor() {
    _private.set(this, { count: 0 });
  }
  increment() {
    _private.get(this).count++;
  }
  value() {
    return _private.get(this).count;
  }
}
// When the Counter instance is GC'd, its private data is too
```

---

## 24 — Proxy and Reflect: Intercept Everything

`Proxy` wraps an object and lets you intercept almost every fundamental operation — reads, writes, deletes, function calls. This is how Vue 3's reactivity works.

```js
const handler = {
  get(target, key) {
    console.log(`Reading ${key}`);
    return Reflect.get(target, key); // forward to original
  },
  set(target, key, value) {
    if (typeof value !== "number") throw new TypeError("Must be number");
    return Reflect.set(target, key, value); // forward
  },
  deleteProperty(target, key) {
    console.log(`Deleting ${key}`);
    return Reflect.deleteProperty(target, key);
  }
};

const obj = new Proxy({ x: 1, y: 2 }, handler);
obj.x;       // logs "Reading x", returns 1
obj.z = "a"; // throws TypeError
obj.z = 99;  // works — 99 is a number
```

### Validation proxy

```js
function createTypedObject(schema) {
  return new Proxy({}, {
    set(target, key, value) {
      const expectedType = schema[key];
      if (expectedType && typeof value !== expectedType) {
        throw new TypeError(`${key} must be ${expectedType}, got ${typeof value}`);
      }
      target[key] = value;
      return true;
    }
  });
}

const user = createTypedObject({ name: "string", age: "number" });
user.name = "Alex"; // ok
user.age = 30;      // ok
user.age = "old";   // TypeError: age must be number, got string
```

### Reflect — the object operation API

`Reflect` is not a constructor. It's an object with static methods that mirror the internal operations of the language. Every trap in a Proxy has a corresponding `Reflect` method as the safe way to forward the operation.

```js
Reflect.get(obj, "key")         // obj.key
Reflect.set(obj, "key", value)  // obj.key = value
Reflect.has(obj, "key")         // "key" in obj
Reflect.deleteProperty(obj, "key") // delete obj.key
Reflect.ownKeys(obj)            // Object.getOwnPropertyNames + Symbols
Reflect.apply(fn, thisArg, args) // fn.apply(thisArg, args)
Reflect.construct(Cls, args)    // new Cls(...args)

// The rule: in a Proxy trap, always forward via Reflect
// Using the direct operation (target.key) bypasses other Proxy traps
// Reflect forwards properly, preserving the full interception chain
```

---

## 25 — Tagged Template Literals: Strings With Superpowers

You know template literals. You probably don't know they can have a function prefix that completely transforms how they're processed.

```js
// Normal template literal
const name = "Alex";
`Hello, ${name}!`; // "Hello, Alex!"

// Tagged template literal — the tag function intercepts processing
function tag(strings, ...values) {
  // strings: array of raw string parts
  // values: array of interpolated values
  console.log(strings); // ["Hello, ", "!"]
  console.log(values);  // ["Alex"]
  return strings[0] + values[0].toUpperCase() + strings[1];
}

tag`Hello, ${name}!`; // "Hello, ALEX!"
```

### Real-world use cases

```js
// 1. SQL with automatic escaping (prevents SQL injection)
function sql(strings, ...values) {
  const query = strings.reduce((acc, str, i) =>
    acc + str + (values[i] !== undefined ? `$${i}` : ""), "");
  return { query, params: values };
}

const id = userInput;
sql`SELECT * FROM users WHERE id = ${id}`;
// { query: "SELECT * FROM users WHERE id = $1", params: [userInput] }
// The id is never interpolated directly — it's a parameter

// 2. HTML sanitization
function html(strings, ...values) {
  const escape = s => String(s)
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;");
  return strings.reduce((acc, str, i) =>
    acc + str + (values[i] !== undefined ? escape(values[i]) : ""), "");
}

const userComment = "<script>alert('xss')</script>";
html`<p>${userComment}</p>`;
// "<p>&lt;script&gt;alert('xss')&lt;/script&gt;</p>" — safe

// 3. String.raw — access the raw string with no escape processing
String.raw`Hello\nWorld`; // "Hello\\nWorld" — \n not converted to newline
```

---

## 26 — Getters and Setters: Computed Properties

Getters and setters let you define properties that look like plain values from the outside but run functions when accessed.

```js
const circle = {
  radius: 5,

  get area() {
    return Math.PI * this.radius ** 2;
  },

  get circumference() {
    return 2 * Math.PI * this.radius;
  }
};

circle.area;          // 78.54... — looks like a property, runs code
circle.radius = 10;
circle.area;          // 314.15... — recomputed

// area is not stored — it's computed every time
// It cannot be set (no setter defined)
```

### Setters with validation

```js
class Temperature {
  #celsius = 0;  // private field

  get celsius()  { return this.#celsius; }
  get fahrenheit() { return this.#celsius * 9/5 + 32; }
  get kelvin()   { return this.#celsius + 273.15; }

  set celsius(value) {
    if (value < -273.15) throw new RangeError("Below absolute zero");
    this.#celsius = value;
  }

  set fahrenheit(value) {
    this.celsius = (value - 32) * 5/9; // reuses celsius setter with validation
  }
}

const t = new Temperature();
t.celsius = 100;
t.fahrenheit;  // 212
t.kelvin;      // 373.15
t.celsius = -300; // RangeError: Below absolute zero
```

### Defining getters/setters on existing objects

```js
Object.defineProperty(obj, "fullName", {
  get() { return `${this.first} ${this.last}`; },
  set(value) {
    [this.first, this.last] = value.split(" ");
  },
  enumerable: true,
  configurable: true
});
```

---

## 27 — Private Class Fields: Real Encapsulation

Before ES2022, "private" fields were a convention (`_name`). Anyone could still access them. Private class fields (`#`) are enforced by the engine.

```js
class BankAccount {
  #balance = 0;   // truly private — inaccessible outside the class
  #owner;

  constructor(owner, initialBalance) {
    this.#owner = owner;
    this.#balance = initialBalance;
  }

  deposit(amount) {
    if (amount <= 0) throw new Error("Must be positive");
    this.#balance += amount;
    return this;
  }

  withdraw(amount) {
    if (amount > this.#balance) throw new Error("Insufficient funds");
    this.#balance -= amount;
    return this;
  }

  get balance() { return this.#balance; }

  toString() {
    return `${this.#owner}: $${this.#balance}`;
  }
}

const account = new BankAccount("Alex", 1000);
account.balance;          // 1000
account.#balance;         // SyntaxError — genuinely inaccessible
account.deposit(500).withdraw(200);
account.balance;          // 1300

// Private static fields and methods
class IdGenerator {
  static #nextId = 1;

  static generate() {
    return IdGenerator.#nextId++;
  }
}
IdGenerator.generate(); // 1
IdGenerator.generate(); // 2
IdGenerator.#nextId;    // SyntaxError
```

### `#` in checks — testing for private field existence

```js
class Node {
  #value;
  constructor(v) { this.#value = v; }

  static isNode(obj) {
    return #value in obj; // true only if obj has the #value private field
  }
}

Node.isNode(new Node(1));  // true
Node.isNode({});           // false
```

---

## 28 — Regular Expressions: The Language Within the Language

JavaScript RegExp is a full pattern-matching language. Most developers use 10% of it.

```js
// Literal syntax vs constructor
const pattern = /hello/i;                  // literal — compiled at parse time
const dynamic = new RegExp("hello", "i"); // constructor — for dynamic patterns

// Flags
/pattern/i   // case insensitive
/pattern/g   // global — find all matches
/pattern/m   // multiline — ^ and $ match line boundaries
/pattern/s   // dotAll — . matches \n too
/pattern/u   // unicode — enables full Unicode support
/pattern/d   // indices — includes match position info (ES2022)
```

### The methods and what they return

```js
// test — boolean, fast when you just need to know if it matches
/^\d+$/.test("123");  // true
/^\d+$/.test("12x");  // false

// match — returns first match (without /g) or all matches (with /g)
"hello world".match(/\w+/);   // ["hello", index: 0, ...]
"hello world".match(/\w+/g);  // ["hello", "world"] — no index with /g

// matchAll — returns iterator of all matches WITH capture groups
for (const match of "2024-01-15".matchAll(/(\d{4})-(\d{2})-(\d{2})/g)) {
  console.log(match[0]); // "2024-01-15" — full match
  console.log(match[1]); // "2024" — group 1
  console.log(match[2]); // "01" — group 2
}

// replace / replaceAll
"hello".replace(/l/, "r");   // "herlo" — first match only
"hello".replace(/l/g, "r");  // "herro" — all matches
"hello".replaceAll("l", "r"); // "herro" — string version

// Replace with a function — powerful
"hello world".replace(/\b\w/g, c => c.toUpperCase()); // "Hello World"

// split
"one,two,,three".split(/,+/); // ["one", "two", "three"]
```

### Named capture groups

```js
// Without named groups:
const match = "2024-01-15".match(/(\d{4})-(\d{2})-(\d{2})/);
const year  = match[1]; // fragile — number depends on group position

// With named groups:
const match = "2024-01-15".match(/(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/);
const { year, month, day } = match.groups; // explicit and readable

// Named groups in replace:
"2024-01-15".replace(
  /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/,
  "$<day>/$<month>/$<year>"
); // "15/01/2024"
```

### Lookahead and lookbehind — match without consuming

```js
// Positive lookahead: match X only if followed by Y
/\d+(?= dollars)/.exec("100 dollars and 50 cents"); // "100"

// Negative lookahead: match X only if NOT followed by Y
/\d+(?! dollars)/.exec("100 dollars and 50 cents"); // "50"

// Positive lookbehind: match X only if preceded by Y
/(?<=\$)\d+/.exec("$100 and €200"); // "100"

// Negative lookbehind: match X only if NOT preceded by Y
/(?<!\$)\d+/.exec("$100 and 200"); // "200"
```

> A common RegExp mistake: using `.` when you mean `[^]` or `[\s\S]`. By default `.` does not match newlines. Use the `/s` (dotAll) flag or `[\s\S]` if you need to.

---

## 29 — The Date Object: Broken by Design

JavaScript's `Date` is one of the most criticized APIs in any language. It was borrowed from Java's `java.util.Date` in the original 10-day implementation — and Java later deprecated those methods too.

```js
// Months are 0-indexed. Days are 1-indexed. This is not a joke.
new Date(2024, 0, 1);  // January 1, 2024 — month 0 = January
new Date(2024, 11, 31); // December 31, 2024 — month 11 = December

// Parsing is implementation-dependent
new Date("2024-01-15");    // UTC midnight
new Date("2024/01/15");    // local time — different result
new Date("January 15, 2024"); // works in most engines, but not guaranteed

// Getting date parts:
const d = new Date();
d.getFullYear();  // 4-digit year
d.getMonth();     // 0-11 (!) — add 1 for human month number
d.getDate();      // 1-31
d.getDay();       // 0=Sunday, 1=Monday, ..., 6=Saturday
d.getHours();     // local hours
d.getUTCHours();  // UTC hours — different if you're not in UTC

// Timestamps
Date.now();        // milliseconds since Unix epoch (Jan 1, 1970 UTC)
d.getTime();       // same, for an existing Date object
```

### What to actually use: Temporal (coming) and workarounds

```js
// For formatting — use Intl.DateTimeFormat (built-in, no library needed)
const fmt = new Intl.DateTimeFormat("en-US", {
  year: "numeric", month: "long", day: "numeric"
});
fmt.format(new Date()); // "January 15, 2024"

// Relative time (2 days ago, in 3 hours)
const rel = new Intl.RelativeTimeFormat("en", { numeric: "auto" });
rel.format(-2, "day");  // "2 days ago"
rel.format(1, "hour");  // "in 1 hour"

// Date arithmetic — always work in timestamps (milliseconds)
const oneDay = 24 * 60 * 60 * 1000;
const tomorrow = new Date(Date.now() + oneDay);

// The real fix: the Temporal API
// Currently in Stage 3 (proposal) — ships in modern browsers progressively
// Temporal.PlainDate, Temporal.ZonedDateTime — immutable, correct timezone support
// For production today: use date-fns or Luxon for complex date work
```

---

## 30 — Array Methods: The Ones You Might Be Misusing

Everyone knows `map`, `filter`, `reduce`. Here are the ones beyond those.

```js
// flat and flatMap
[1, [2, 3], [4, [5, 6]]].flat();     // [1, 2, 3, 4, [5, 6]] — one level
[1, [2, 3], [4, [5, 6]]].flat(2);    // [1, 2, 3, 4, 5, 6] — two levels
[1, [2, 3], [4, [5, 6]]].flat(Infinity); // fully flattened

// flatMap = map + flat(1) — more efficient than doing both separately
const sentences = ["hello world", "foo bar"];
sentences.flatMap(s => s.split(" ")); // ["hello", "world", "foo", "bar"]

// find and findIndex — first match
[1, 2, 3, 4].find(n => n > 2);       // 3 — the value
[1, 2, 3, 4].findIndex(n => n > 2);  // 2 — the index
[1, 2, 3, 4].findLast(n => n < 3);   // 2 — search from end
[1, 2, 3, 4].findLastIndex(n => n < 3); // 1 — index from end

// at — supports negative indices
const arr = [1, 2, 3, 4, 5];
arr.at(-1);  // 5 — last element (cleaner than arr[arr.length - 1])
arr.at(-2);  // 4 — second to last

// includes vs indexOf — when to use which
arr.includes(3);      // true — when you just need existence
arr.indexOf(3);       // 2 — when you need the position
arr.includes(NaN);    // true — includes handles NaN
arr.indexOf(NaN);     // -1 — indexOf uses ===, NaN !== NaN

// some and every
arr.some(n => n > 4);  // true — at least one matches
arr.every(n => n > 0); // true — all match
arr.every(n => n > 3); // false — not all match

// reduce vs reduceRight
[1, 2, 3, 4].reduce((acc, n) => acc + n, 0);       // 10, left to right
[[1,2],[3,4],[5,6]].reduceRight((acc, arr) => [...acc, ...arr], []);
// [5, 6, 3, 4, 1, 2] — right to left

// Array.from — create arrays from anything iterable
Array.from("hello");        // ["h","e","l","l","o"]
Array.from({length: 5}, (_, i) => i * 2); // [0, 2, 4, 6, 8]
Array.from(new Set([1,2,2,3])); // [1, 2, 3]
Array.from(document.querySelectorAll("div")); // array from NodeList

// toSorted, toReversed, toSpliced, with — non-mutating versions (ES2023)
const original = [3, 1, 2];
const sorted = original.toSorted();   // [1, 2, 3] — original unchanged
original.sort();                       // [1, 2, 3] — mutates original

const arr2 = [1, 2, 3];
arr2.with(1, 99);  // [1, 99, 3] — new array, original unchanged
```

---

## 31 — Object Methods: What's Available at the Top Level

`Object.*` is an underused toolbox. Most developers only know `Object.keys`.

```js
// Enumeration
Object.keys(obj)    // ["key1", "key2"] — own, enumerable, string keys
Object.values(obj)  // [val1, val2] — same
Object.entries(obj) // [["key1", val1], ["key2", val2]] — same

// All own keys including non-enumerable and Symbols
Object.getOwnPropertyNames(obj)   // all own string keys
Object.getOwnPropertySymbols(obj) // all own Symbol keys
Reflect.ownKeys(obj)              // all own keys (strings + Symbols)

// Creating objects
Object.create(proto)            // new object with proto as [[Prototype]]
Object.create(null)             // no prototype — truly empty object
                                // no .toString, .hasOwnProperty, nothing

// Merging
Object.assign(target, ...sources) // copies own enumerable properties to target
// WARNING: Object.assign mutates target and does shallow copy
const merged = Object.assign({}, defaults, overrides); // common pattern

// Better: spread (also shallow, but doesn't mutate)
const merged = { ...defaults, ...overrides };

// Freezing and sealing
Object.freeze(obj)   // no add, delete, or modify — throws in strict mode, silent otherwise
Object.seal(obj)     // no add or delete, but CAN modify existing
Object.isFrozen(obj)
Object.isSealed(obj)

// Property descriptors
Object.getOwnPropertyDescriptor(obj, "key");
// { value: ..., writable: true, enumerable: true, configurable: true }

Object.defineProperty(obj, "key", {
  value: 42,
  writable: false,   // cannot change the value
  enumerable: false, // won't show in for...in or Object.keys
  configurable: false // cannot redefine or delete this property
});

// fromEntries — inverse of entries
Object.fromEntries([["a", 1], ["b", 2]]); // { a: 1, b: 2 }
Object.fromEntries(new Map([["a", 1]]));  // { a: 1 }

// Transform object values (no built-in — use entries + fromEntries)
const doubled = Object.fromEntries(
  Object.entries(obj).map(([k, v]) => [k, v * 2])
);

// hasOwn — modern version of hasOwnProperty (works on Object.create(null))
Object.hasOwn(obj, "key"); // true if key is own property
// Don't use: obj.hasOwnProperty("key") — fails if prototype is null
```

---

## 32 — Performance Patterns: Writing Fast JavaScript

JavaScript performance is not about micro-optimizations. It's about understanding where the real costs are.

### The actual expensive operations

```js
// DOM manipulation is the #1 culprit
// BAD — causes a reflow for each append
items.forEach(item => {
  document.body.appendChild(createEl(item));
});

// GOOD — one DOM operation
const fragment = document.createDocumentFragment();
items.forEach(item => fragment.appendChild(createEl(item)));
document.body.appendChild(fragment);

// Forced synchronous layouts (layout thrashing)
// WRONG — reads layout, writes DOM, reads layout, writes DOM...
elements.forEach(el => {
  const height = el.offsetHeight; // read — triggers layout
  el.style.height = height + 10 + "px"; // write — invalidates layout
});

// RIGHT — batch reads, then batch writes
const heights = elements.map(el => el.offsetHeight); // all reads
elements.forEach((el, i) => {
  el.style.height = heights[i] + 10 + "px"; // all writes
});
```

### Hidden class optimization (V8 internals)

```js
// V8 assigns "hidden classes" to objects for property lookup optimization
// Changing the shape of an object causes it to be deoptimized

// SLOW — adds properties in different order, different hidden classes
function createUser(fast) {
  if (fast) return { name: "Alex", age: 30 };
  return { age: 30, name: "Alex" }; // different insertion order = different hidden class
}

// FAST — consistent shape
function createUser(name, age) {
  return { name, age }; // always same order = same hidden class
}

// SLOW — adding properties after creation
const user = {};
user.name = "Alex";
user.age = 30;

// FAST — all properties at creation
const user = { name: "Alex", age: 0 }; // define shape upfront
user.age = 30;                          // update, don't add
```

### Memoization — cache expensive results

```js
function memoize(fn) {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

const expensiveCalc = memoize(function(n) {
  // slow operation
  return fibonacci(n);
});

expensiveCalc(40); // computed
expensiveCalc(40); // returned from cache instantly
```

### Debounce and throttle — controlling call frequency

```js
// Debounce: wait until calls STOP for N ms, then fire once
function debounce(fn, delay) {
  let timer;
  return function(...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}

const search = debounce(fetchResults, 300);
// Typing "javascript" fires one request, not one per keystroke

// Throttle: fire at most once per N ms, regardless of call frequency
function throttle(fn, limit) {
  let inThrottle = false;
  return function(...args) {
    if (!inThrottle) {
      fn.apply(this, args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
}

const onScroll = throttle(updatePosition, 16); // ~60fps
```

### The cost of closures in hot paths

```js
// Closures in tight loops create allocation pressure
// SLOW — creates a new closure for every item in every render
function render(items) {
  return items.map(item => () => handleClick(item.id)); // new fn per item
}

// FAST — stable reference, event delegation, or useCallback
// Store handler outside, use event.target.dataset instead
```

---

<div align="center">

## The Language Rewards the Patient

Arrays, closures, and `fetch` will take you far.  
Symbols, generators, Proxy, and WeakMaps take you further.  

**The developers who know what `Symbol.toPrimitive` does, why `Map` beats an object in dynamic-key scenarios, and when a generator replaces a recursive function — they're not showing off. They're using the right tool.**

Go be one of them.

</div>