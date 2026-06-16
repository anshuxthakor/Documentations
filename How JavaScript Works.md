<div align="center">

# How JavaScript Really Works

### *The hidden machinery behind every line you write*

**Most developers think JavaScript is just `console.log()` and `addEventListener`. This is for the ones who want to go further.**

</div>

## What's Inside

- **Three Truths** — Single-threaded, synchronous, JIT-compiled. Burn these in.
- **The Engine** — What actually happens between you writing code and it running
- **Execution Context** — The sealed container every piece of JS lives inside
- **The Call Stack** — How JS tracks what's running, and what breaks it
- **Hoisting** — Why code "works" before it's defined (and when it doesn't)
- **Scope** — The regions where variables live, and the rules that contain them
- **The Scope Chain** — How JS hunts for variables it can't find
- **Closures** — The most important concept in the language. Full stop.

---

## 01 — Three Truths. Memorize Them Now.

Everything else in this document builds on these. If you misunderstand any one of them, every other concept will be slightly wrong.

**JavaScript is single-threaded.**

One call stack. One thread of execution. It can do exactly one thing at a time. There is no "run two functions in parallel" inside the language itself.

**JavaScript is synchronous by default.**

Code runs top to bottom, line by line. A line does not start until the previous one finishes.

**JavaScript is interpreted *and* JIT-compiled.**

Old explanation: "JS is an interpreted language." Modern reality: engines use **JIT (Just-In-Time) compilation** — they interpret your code to start fast, then compile the frequently-used parts into optimized machine code. It's a hybrid.

> **The myth to kill:** *"JS is multi-threaded because async works."*  
> JS itself is single-threaded. Async behaviour — timers, network calls — is handled by the **environment** (browser or Node), not by the JS engine. The language and the runtime are not the same thing.

---

## 02 — The Engine: Your Code's Translator

JS code can't run on its own. It needs an **engine** — a program (mostly written in C++) that reads your JavaScript and turns it into something the machine actually understands.

| Engine | Used In |
|---|---|
| **V8** | Chrome, Node.js, Edge |
| **SpiderMonkey** | Firefox |
| **JavaScriptCore** | Safari |

Every time you run a JS file, the engine does this — in order:

```
1. Tokenizing   →  "let x = 10;" becomes: [let] [x] [=] [10] [;]
2. Parsing      →  Tokens become an Abstract Syntax Tree (AST)
3. Compilation  →  AST becomes bytecode (Ignition), then machine code (TurboFan)
4. Execution    →  Code runs on the Call Stack
```

```js
// This one line
let x = 10;

// Goes through all four stages before "x" ever exists in memory
```

**Why it matters:** "My JS is slow" is almost never about your logic. It's about how the engine treats your patterns — how many times it deoptimizes, how many hidden class transitions happen, how much the GC has to clean up. Understanding the pipeline tells you *where* performance actually lives.

> The engine isn't just running your code. It's making bets on what your code *will do* — and recompiling when those bets are wrong.

---

## 03 — Execution Context: The Container Everything Runs Inside

An **Execution Context** is the environment in which a piece of JavaScript is evaluated and executed. Think of it as a sealed container that holds everything the code needs: its variables, its functions, and the value of `this`.

JS never runs code "loose." It always runs inside a context.

### The Global Execution Context (GEC)

Created first. Automatically. The moment your script starts. There is exactly **one**. In the browser, it creates the `window` object and sets `this` to point at it.

### Function Execution Contexts (FEC)

A **new** context is created every time a function is called. Each gets its own private variables. When the function finishes, its context is destroyed — *usually*. Closures are the exception. More on that soon.

### The Two Phases Every Context Goes Through

This is the real reason half of JS's "weird" behaviour exists.

**Phase 1 — Creation (Memory Allocation)**  
Before a single line executes, JS scans the code and sets up memory:
- All `var` variables are stored and set to `undefined`
- All function declarations are stored in full
- The value of `this` is determined

**Phase 2 — Execution (Code Runs)**  
Now JS runs the code line by line, assigning real values to what it reserved in Phase 1.

```js
var x = 10;
function greet() {
  console.log("Hello");
}
greet();
```

```
Creation phase:  x → undefined | greet → full function stored | this → set
Execution phase: x becomes 10  | greet() is called → new FEC created → "Hello" prints
```

> The two-phase model is the real reason hoisting works. Everything in Section 05 flows from this.

---

## 04 — The Call Stack: How JS Knows What's Running

The **Call Stack** is a data structure that tracks which function is currently running. It works on **LIFO — Last In, First Out**. The last function pushed in is the first one to come out.

- Function is **called** → it's **pushed** onto the stack
- Function **returns** → it's **popped** off
- The bottom of the stack is always the Global Execution Context

```js
function one() {
  two();
  console.log("one done");
}
function two() {
  three();
  console.log("two done");
}
function three() {
  console.log("three done");
}
one();
```

```
Step 1: one() called       →  [ one | GEC ]
Step 2: two() called       →  [ two | one | GEC ]
Step 3: three() called     →  [ three | two | one | GEC ]
Step 4: three() finishes   →  [ two | one | GEC ]       prints "three done"
Step 5: two() finishes     →  [ one | GEC ]             prints "two done"
Step 6: one() finishes     →  [ GEC ]                   prints "one done"
```

Functions *start* in the order `one → two → three`, but *finish* in reverse. That's LIFO in action.

### Stack Overflow

If functions keep getting pushed without ever returning — usually from **infinite recursion** — the stack runs out of space:

```js
function loop() {
  loop(); // never returns
}
loop(); // ❌ RangeError: Maximum call stack size exceeded
```

Every recursive function must have a base case. Without one, it's not recursion — it's a countdown to a crash.

---

## 05 — Hoisting: The Creation Phase in Disguise

Hoisting is JS's behaviour of *appearing* to move declarations to the top of their scope. Nothing physically moves. This is just the **Creation Phase** allocating memory before a single line of code runs.

### `var` — Hoisted and Initialized as `undefined`

```js
console.log(a); // undefined — not an error
var a = 5;
console.log(a); // 5
```

The variable exists before the line. It's just empty.

### `let` and `const` — Hoisted Into the Temporal Dead Zone

`let` and `const` are hoisted too — but not initialized. Between the start of the scope and the line they're declared on, they exist in the **Temporal Dead Zone (TDZ)**. Touch them there and you get an error.

```js
console.log(b); // ❌ ReferenceError: Cannot access 'b' before initialization
let b = 10;
```

| Keyword | Hoisted? | Value Before Declaration | Access Early |
|---|---|---|---|
| `var` | Yes | `undefined` | `undefined` |
| `let` | Yes | none (TDZ) | ReferenceError |
| `const` | Yes | none (TDZ) | ReferenceError |

### Function Declarations vs. Function Expressions

This one catches everyone.

```js
// Declaration — fully hoisted, callable before it's defined
sayHi(); // ✅ works
function sayHi() { console.log("Hi"); }

// Expression — only the variable is hoisted, not the function
sayBye(); // ❌ TypeError: sayBye is not a function
var sayBye = function() { console.log("Bye"); };
```

In the second case, `sayBye` is hoisted as `undefined`. Calling `undefined()` throws a TypeError. With `let`/`const` it would throw a ReferenceError instead — the TDZ gets you first.

> **The rule:** declarations are hoisted with their body. Expressions are hoisted only as a variable.

---

## 06 — Scope: Where Variables Live

**Scope** is the region of code where a variable can be accessed. Get this wrong and you'll spend hours debugging things that were never broken.

### Global Scope

Declared outside any function or block. Accessible from anywhere.

```js
let appName = "MyApp";
function show() {
  console.log(appName); // ✅
}
```

### Function Scope

Variables declared inside a function stay inside it. No exceptions.

```js
function test() {
  var secret = 42;
}
console.log(secret); // ❌ ReferenceError
```

### Block Scope

A block is anything inside `{ }`. `let` and `const` respect blocks. `var` ignores them completely.

```js
{
  let a = 1;
  var b = 2;
}
console.log(b); // 2  — var leaked out
console.log(a); // ❌ ReferenceError — let stayed in
```

### Lexical Scope

**Lexical** means "based on where the code is physically written" — not where or how it's called. A function's scope is decided by its location in the source code at write time, not at runtime.

```js
function outer() {
  let name = "Sara";
  function inner() {
    console.log(name); // sees outer's variable — because of where it's written
  }
  inner();
}
outer(); // "Sara"
```

`inner` is written *inside* `outer`. That physical nesting is the entire mechanism. Move `inner` outside and the rule changes immediately.

---

## 07 — The Scope Chain: How JS Hunts for Variables

When you use a variable, JS doesn't just look in the current scope and give up. It searches outward.

```
1. Current scope → found? Use it.
2. Parent scope  → found? Use it.
3. Parent's parent... all the way to global scope.
4. Still nothing? → ReferenceError.
```

This outward search is the **Scope Chain**. It only travels in one direction — outward. Never inward.

```js
let a = 10;
function outer() {
  let b = 20;
  function inner() {
    let c = 30;
    console.log(a, b, c); // 10  20  30
  }
  inner();
}
outer();
```

`inner` finds `c` in itself. Finds `b` in `outer`. Finds `a` in global. All three work.

But `outer` can never see `c`. It lives deeper, and the chain doesn't go inward. **One-directional. Always.**

---

## 08 — Closures: The Star of the Show

### What a closure actually is

A **closure** is a function bundled together with its surrounding **lexical environment**. In plain terms: a function **remembers the variables from the scope where it was created**, even after that outer scope has finished executing.

Think of it like a backpack. When a function is created, it packs the variables around it into a backpack and carries them wherever it goes. The outer scope can finish and disappear — the backpack stays.

```js
function counter() {
  let count = 0;          // local variable
  return function () {    // inner function is returned
    count++;
    return count;
  };
}

const inc = counter();    // counter() runs and finishes
console.log(inc()); // 1
console.log(inc()); // 2
console.log(inc()); // 3
```

Normally, when `counter()` finishes, `count` is destroyed. But the returned inner function still references it — so JS keeps `count` alive. That preserved bundle is the closure. And `count` is now completely **private**. Nothing outside can touch it directly.

### What closures are actually used for

**Data privacy:**

```js
function createBankAccount() {
  let balance = 0;
  return {
    deposit(amount) { balance += amount; return balance; },
    getBalance() { return balance; }
  };
}
const acc = createBankAccount();
acc.deposit(100);
console.log(acc.getBalance()); // 100
console.log(acc.balance);      // undefined — truly private
```

**Function factories:**

```js
function multiplier(factor) {
  return function(number) {
    return number * factor;
  };
}
const double = multiplier(2);
const triple = multiplier(3);
console.log(double(5)); // 10
console.log(triple(5)); // 15
```

**Currying:**

```js
function add(a) {
  return function(b) {
    return function(c) {
      return a + b + c;
    };
  };
}
console.log(add(1)(2)(3)); // 6
```

### The classic trap — closures in loops

This is the most famous closure interview question, and it trips up everyone the first time.

```js
for (var i = 0; i < 3; i++) {
  setTimeout(function() {
    console.log(i);
  }, 1000);
}
// Output: 3  3  3  — not 0 1 2
```

**Why?** `var` is function-scoped. There is only **one** `i`. By the time the callbacks run (after the loop finishes), `i` is already `3`. All three closures closed over the *same variable*.

**Fix with `let`:**

```js
for (let i = 0; i < 3; i++) {
  setTimeout(function() {
    console.log(i);
  }, 1000);
}
// Output: 0  1  2  ✅
```

`let` is block-scoped. Each iteration creates a **new** `i`. Each closure captures its own copy.

**Fix with IIFE (the pre-`let` way):**

```js
for (var i = 0; i < 3; i++) {
  (function(j) {
    setTimeout(function() {
      console.log(j);
    }, 1000);
  })(i);
}
// Output: 0  1  2  ✅
```

> Closures capture *variables*, not *values*. The loop problem is the proof. `var` vs `let` controls how many variables actually exist — and that changes everything.

---

## Quick Revision

| Concept | One-line summary |
|---|---|
| Single-threaded | One call stack, one thing at a time |
| JIT-compiled | Interpreted first, hot code compiled for speed |
| Execution Context | The sealed environment where code runs |
| Two phases | Creation (memory) → Execution (code) |
| Call Stack | LIFO structure tracking running functions |
| Stack overflow | Too many calls without returning |
| `var` hoisting | Hoisted as `undefined` |
| `let`/`const` hoisting | Hoisted into TDZ — error if accessed early |
| Lexical scope | Scope decided by where code is *written* |
| Scope chain | Look outward/up until the variable is found |
| Closure | Function + the variables it carries from its birthplace |

---

## Practice — Predict Before You Run

```js
// 1
console.log(x);
var x = 5;

// 2
foo();
function foo() { console.log("called"); }

// 3
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}

// 4
function make() {
  let secret = "hidden";
  return () => secret;
}
const reveal = make();
console.log(reveal());
```

Write your answer for each one before running it. The gap between what you predicted and what happened — that's exactly where the learning is.

---

<div align="center">

## The Gap Is Where Craft Lives

Most developers treat JavaScript as a scripting language you pick up as you go.  
The ones who understand the engine, the stack, and the scope chain write code that's debuggable, predictable, and fast.

**Go be one of those.**

</div>