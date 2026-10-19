<div align="center">

# TypeScript Is Not Just JavaScript With Types

### *The compiler, the safety net, and the parts nobody explains properly*

**Most developers learn TypeScript by adding `: string` everywhere. This is for the ones who want to understand what TypeScript actually protects — and what it cannot protect at runtime.**

</div>

## What's Inside

- **TypeScript's Core Idea** — Types exist mostly before the code runs
- **Inference vs Annotation** — When TypeScript knows the type automatically
- **Type Erasure** — Why most TypeScript disappears after compilation
- **Primitive Types** — Strings, numbers, booleans, symbols, and BigInts
- **`any` vs `unknown`** — Escape hatch versus safe uncertainty
- **Arrays and Tuples** — Flexible collections versus fixed structures
- **Unions and Literal Types** — Restricting values to valid possibilities
- **Enums** — The TypeScript feature that survives compilation
- **Functions** — Typing parameters, callbacks, return values, and rest parameters
- **Interfaces and Type Aliases** — Describing the shape of data
- **React + TypeScript** — Typing state, props, API data, and components
- **Configuration** — Understanding `tsconfig.json`
- **Destructuring** — Pattern matching with defaults and renaming
- **Spread and Rest** — Two uses of the same three dots
- **Optional Chaining** — Safe access through uncertain data
- **Nullish Coalescing** — Defaults that respect valid falsy values
- **Generics** — Types that work across many data shapes
- **Private Fields** — Real encapsulation in JavaScript classes
- **Symbols** — Invisible object keys and protocol hooks
- **Iterators and Generators** — Custom sequences and lazy data
- **Maps and Sets** — Data structures beyond plain objects
- **Proxies and Reflect** — Intercepting object operations
- **Tagged Templates** — Turning strings into structured data
- **Getters and Setters** — Properties that execute code
- **Performance** — What actually makes JavaScript slow

---

## 01 — TypeScript Is a Compiler-Time Safety Net

TypeScript is a programming language built on top of JavaScript.

It adds a static type system that can detect mistakes before the code runs.

```ts
let username: string = "Alex";

username = "Jordan"; // valid
username = 42;       // TypeScript error
```

JavaScript allows this:

```js
let username = "Alex";
username = 42;
```

TypeScript warns you before the program reaches production.

But there is an important detail:

> TypeScript types mostly disappear when the code is compiled to JavaScript.

This TypeScript:

```ts
let username: string = "Alex";
```

becomes JavaScript similar to this:

```js
let username = "Alex";
```

The `: string` annotation does not exist at runtime.

TypeScript helps during:

- Development
- Editor autocomplete
- Static analysis
- Compilation
- Refactoring
- Code review

TypeScript does not automatically validate:

- API responses
- User input
- Database data
- JSON files
- Runtime JavaScript values
- Data from an untyped library

```txt
TypeScript:
  checks your code before execution

JavaScript:
  executes the code at runtime
```

This is the most important mental model:

```txt
TypeScript prevents many mistakes while you write code.
It does not turn JavaScript into a runtime validation system.
```

---

## 02 — Type Inference vs. Type Annotation

There are two primary ways to tell TypeScript what a value should be.

### Type Annotation

You explicitly write the type:

```ts
let name: string = "Alex";
let age: number = 25;
let isAdmin: boolean = false;
```

The syntax is:

```ts
let variableName: Type = value;
```

### Type Inference

TypeScript can often understand the type automatically:

```ts
let name = "Alex";
let age = 25;
let isAdmin = false;
```

TypeScript infers:

```txt
name    → string
age     → number
isAdmin → boolean
```

You do not need to annotate every obvious variable.

```ts
const message = "Hello";
// TypeScript already knows message is a string
```

### Once Inferred, the Type Is Enforced

```ts
let score = 100;

score = 200;   // valid
score = "200"; // error
```

The initial value gives TypeScript enough information to determine the variable's type.

### When Annotation Is Useful

Annotations are especially useful when:

- There is no initial value
- The type is complex
- The value comes from an API
- A function needs a documented contract
- You want to restrict a value more precisely
- TypeScript cannot infer the intended type

```ts
let result: string;

result = "success";
```

Without an annotation, an uninitialized variable may not have the type you intended.

### Do Not Annotate Everything Blindly

This is unnecessary:

```ts
const username: string = "Alex";
```

This is often enough:

```ts
const username = "Alex";
```

Good TypeScript uses inference where the type is obvious and annotations where they add clarity or safety.

---

## 03 — Type Erasure: Most Types Disappear

TypeScript is not directly executed by the browser.

The TypeScript compiler or another tool transforms it into JavaScript.

```ts
const add = (a: number, b: number): number => {
  return a + b;
};
```

After compilation:

```js
const add = (a, b) => {
  return a + b;
};
```

These parts disappear:

```txt
: number
: string
: boolean
interface declarations
type aliases
union types
tuple restrictions
generic annotations
```

They existed for the compiler, not the JavaScript engine.

### The Runtime Does Not Know Your Types

```ts
let age: number = 25;
```

At runtime:

```js
let age = 25;
```

JavaScript does not know that `age` was originally declared as a number by TypeScript.

This means TypeScript will not protect you from external data automatically:

```ts
interface User {
  name: string;
  age: number;
}

const response: User = await fetchUser();
```

The annotation does not magically validate the response.

If the server returns:

```json
{
  "name": 42,
  "age": "unknown"
}
```

TypeScript will not stop the value at runtime unless you use a runtime validation library or manually validate it.

> Static types describe what you believe a value looks like.  
> Runtime validation proves whether that belief is true.

### Important Exception: Enums

Most type features disappear, but enums generate runtime JavaScript:

```ts
enum Role {
  ADMIN,
  USER
}
```

becomes an actual JavaScript object.

This distinction matters:

| TypeScript Feature | Runtime JavaScript |
|---|---|
| Type annotations | Removed |
| Interfaces | Removed |
| Type aliases | Removed |
| Union types | Removed |
| Tuples | Become normal arrays |
| Generics | Removed |
| `enum` | Generates runtime code |
| Classes | Remain as JavaScript classes |

---

## 04 — Primitive Types

TypeScript supports JavaScript's primitive values with type names.

```ts
let username: string = "Alex";
let age: number = 25;
let isOnline: boolean = true;
let empty: undefined = undefined;
let largeNumber: bigint = 79697678n;
let uniqueKey: symbol = Symbol("key");
```

| Type | Example | Meaning |
|---|---|---|
| `string` | `"Alex"` | Text |
| `number` | `25` | Integers and floating-point numbers |
| `boolean` | `true` | `true` or `false` |
| `undefined` | `undefined` | Missing value |
| `null` | `null` | Intentional absence of value |
| `bigint` | `100n` | Large integers |
| `symbol` | `Symbol("id")` | Unique identifiers |

JavaScript has one numeric type:

```ts
let integer: number = 10;
let decimal: number = 10.5;
let negative: number = -10;
```

There is no separate TypeScript type for:

- `int`
- `float`
- `double`

All of them are `number`.

### `null` and `undefined`

These are separate types:

```ts
let missing: undefined = undefined;
let empty: null = null;
```

With strict configuration enabled, you cannot assign `null` to a normal string:

```ts
let name: string = null; // error with strictNullChecks
```

You must explicitly allow it:

```ts
let name: string | null = null;
```

This forces you to handle the missing case.

---

## 05 — `any`, `unknown`, and `never`

These three types are often misunderstood.

### `any` Turns TypeScript Off

```ts
let value: any = "hello";

value.toUpperCase();
value.doesNotExist();
value();
value.foo.bar.baz;
```

TypeScript allows all of this because `any` disables checking for that value.

```ts
let data: any = {
  name: "Alex"
};

data.age.toFixed();
```

This compiles, but it may crash at runtime because `age` does not exist.

Use `any` sparingly.

It is sometimes useful when:

- Migrating a large JavaScript project
- Working with an untyped library
- Temporarily bypassing an incomplete type
- Prototyping quickly

But `any` spreads uncertainty through the codebase.

### `unknown` Is the Safer Alternative

```ts
let value: unknown = "hello";
```

You cannot use it directly:

```ts
value.toUpperCase(); // error
```

You must narrow it first:

```ts
if (typeof value === "string") {
  console.log(value.toUpperCase());
}
```

`unknown` means:

> This value could be anything, and you must prove what it is before using it.

```ts
function printValue(value: unknown) {
  if (typeof value === "string") {
    console.log(value.toUpperCase());
  }

  if (typeof value === "number") {
    console.log(value.toFixed(2));
  }
}
```

| | `any` | `unknown` |
|---|---|---|
| Accepts any value | Yes | Yes |
| Allows direct property access | Yes | No |
| Requires narrowing | No | Yes |
| Safe by default | No | Yes |

### `never` Represents the Impossible

`never` represents a value that can never occur.

A function that always throws:

```ts
function fail(message: string): never {
  throw new Error(message);
}
```

A function with an infinite loop:

```ts
function runForever(): never {
  while (true) {
    console.log("running");
  }
}
```

It is also useful for exhaustive checks:

```ts
type Status = "loading" | "success" | "error";

function handleStatus(status: Status) {
  switch (status) {
    case "loading":
      return "Loading...";
    case "success":
      return "Success";
    case "error":
      return "Error";
    default:
      return assertNever(status);
  }
}

function assertNever(value: never): never {
  throw new Error(`Unexpected value: ${value}`);
}
```

If a new status is added but not handled, TypeScript can detect the missing case.

---

## 06 — Arrays Are Typed Collections

A typed array describes the type of every element.

```ts
let numbers: number[] = [1, 2, 3, 4];
let names: string[] = ["Alex", "Jordan"];
let flags: boolean[] = [true, false, true];
```

These are equivalent:

```ts
let numbers: number[] = [1, 2, 3];
let numbers2: Array<number> = [1, 2, 3];
```

The first form is more common.

### Array Type Enforcement

```ts
const numbers: number[] = [1, 2, 3];

numbers.push(4);     // valid
numbers.push("four"); // error
```

TypeScript knows what methods are available:

```ts
numbers.map((number) => number * 2);
numbers.reduce((sum, number) => sum + number, 0);
```

Inside the callback, `number` is automatically inferred as a `number`.

### Arrays of Objects

```ts
interface User {
  id: number;
  name: string;
}

const users: User[] = [
  {
    id: 1,
    name: "Alex"
  },
  {
    id: 2,
    name: "Jordan"
  }
];
```

Now every user must follow the `User` shape.

```ts
users.push({
  id: 3,
  name: "Taylor"
});
```

This is invalid:

```ts
users.push({
  id: "3",
  name: "Taylor"
});
```

The `id` must be a number.

### Empty Arrays and `never[]`

This can be a problem:

```ts
const values = [];
```

Depending on the context, TypeScript may infer the array too narrowly, sometimes as `never[]`.

Use an explicit type when the array starts empty:

```ts
const values: string[] = [];
const products: Product[] = [];
```

This is especially important with React state:

```tsx
const [products, setProducts] = useState<Product[]>([]);
```

---

## 07 — Tuples Are Fixed-Length Arrays

A tuple is an array with:

- A fixed number of elements
- A fixed order
- A known type for every position

```ts
let user: [number, string, boolean] = [
  1,
  "Alex",
  true
];
```

This means:

```txt
Index 0 → number
Index 1 → string
Index 2 → boolean
```

This is not the same as:

```ts
let values: (number | string | boolean)[] = [
  1,
  "Alex",
  true
];
```

The regular array can have any length and any order.

The tuple has a specific structure.

### Tuple Example

```ts
type Coordinate = [number, number];

const point: Coordinate = [10, 20];
```

This is invalid:

```ts
const point: Coordinate = [10];
const point2: Coordinate = ["10", "20"];
```

### Optional Tuple Elements

```ts
type Person = [string, number, boolean?];

const first: Person = ["Alex", 25];
const second: Person = ["Alex", 25, true];
```

### Readonly Tuples

```ts
const point: readonly [number, number] = [10, 20];
```

The tuple cannot be modified:

```ts
point[0] = 50; // error
```

### Tuples Are Normal Arrays at Runtime

This:

```ts
const user: [number, string] = [1, "Alex"];
```

becomes:

```js
const user = [1, "Alex"];
```

The fixed-length and positional rules exist only during type checking.

---

## 08 — Unions Mean “This or That”

A union type allows a value to have one of several possible types.

```ts
let value: string | number;

value = "hello";
value = 42;
```

This is invalid:

```ts
value = true;
```

The pipe symbol means:

```txt
string OR number
```

### Narrowing a Union

TypeScript does not allow operations that are unsafe for every member:

```ts
function printValue(value: string | number) {
  // value.toUpperCase(); // error
}
```

Narrow the value first:

```ts
function printValue(value: string | number) {
  if (typeof value === "string") {
    console.log(value.toUpperCase());
  } else {
    console.log(value.toFixed(2));
  }
}
```

### Discriminated Unions

A common pattern is to add a shared property that identifies the variant:

```ts
type Result =
  | {
      status: "success";
      data: string;
    }
  | {
      status: "error";
      message: string;
    };

function displayResult(result: Result) {
  if (result.status === "success") {
    return result.data;
  }

  return result.message;
}
```

Once `status` is checked, TypeScript knows which object shape is available.

### Literal Unions

A union can restrict a value to exact strings:

```ts
type Status = "pending" | "success" | "error";

let status: Status = "pending";

status = "success"; // valid
status = "failed";  // error
```

This is much safer than:

```ts
let status: string = "pending";
```

With `string`, a typo is accepted:

```ts
status = "sucess";
```

With a literal union, TypeScript catches it immediately.

---

## 09 — Enums Are Runtime Objects

Enums create a named group of constants.

```ts
enum Role {
  ADMIN,
  SUP_ADMIN,
  USER
}

let role: Role = Role.USER;
```

By default, numeric enums start at zero:

```txt
Role.ADMIN     → 0
Role.SUP_ADMIN → 1
Role.USER      → 2
```

Enums generate JavaScript at runtime.

Conceptually, TypeScript creates a mapping like:

```js
Role[Role["ADMIN"] = 0] = "ADMIN";
Role[Role["SUP_ADMIN"] = 1] = "SUP_ADMIN";
Role[Role["USER"] = 2] = "USER";
```

This allows both directions:

```ts
Role.ADMIN; // 0
Role[0];    // "ADMIN"
```

### String Enums

```ts
enum Direction {
  Up = "UP",
  Down = "DOWN",
  Left = "LEFT",
  Right = "RIGHT"
}
```

String enums are often easier to debug because the runtime values are meaningful strings.

### Enum vs Literal Union

These solve similar problems:

```ts
enum Role {
  ADMIN = "admin",
  USER = "user"
}
```

```ts
type Role = "admin" | "user";
```

| | Enum | Literal Union |
|---|---|---|
| Generates runtime code | Yes | No |
| Can be used as a runtime object | Yes | No |
| Lightweight | No | Yes |
| Good for fixed runtime constants | Yes | Sometimes |
| Disappears after compilation | No | Yes |

Use enums when you need a runtime object.

Use literal unions when you only need compile-time restrictions.

---

## 10 — Functions Need Typed Contracts

TypeScript can type:

- Parameters
- Return values
- Optional parameters
- Default parameters
- Callbacks
- Rest parameters
- Function variables

```ts
function add(a: number, b: number): number {
  return a + b;
}
```

The function promises:

```txt
Input:
  number
  number

Output:
  number
```

This is invalid:

```ts
add(10, "20");
```

### Return Type Inference

TypeScript can infer the return type:

```ts
function multiply(a: number, b: number) {
  return a * b;
}
```

The return type is inferred as `number`.

An explicit return annotation is useful when:

- The function is public
- The return type is complex
- You want to prevent accidental changes
- The function has multiple branches

```ts
function getMessage(isSuccess: boolean): string {
  return isSuccess ? "Success" : "Failed";
}
```

### Arrow Functions

```ts
const subtract = (a: number, b: number): number => {
  return a - b;
};
```

### Optional Parameters

Optional parameters use `?`.

```ts
function greet(name: string, title?: string) {
  if (title) {
    return `Hello ${title} ${name}`;
  }

  return `Hello ${name}`;
}
```

The optional parameter has the type:

```ts
string | undefined
```

### Default Parameters

```ts
function createUser(
  name: string = "Anonymous",
  role: string = "viewer"
) {
  return {
    name,
    role
  };
}
```

If no argument is provided, the default value is used.

### Function Types

```ts
let operation: (a: number, b: number) => number;

operation = (a, b) => a + b;
operation = (a, b) => a * b;
```

The variable can store any function matching that signature.

### Callback Types

```ts
function processUser(
  user: string,
  callback: (name: string) => void
) {
  callback(user);
}

processUser("Alex", (name) => {
  console.log(name);
});
```

The callback:

```ts
(name: string) => void
```

accepts a string and returns nothing meaningful.

---

## 11 — Rest Parameters Accept Any Number of Arguments

A normal function may expect a fixed number of arguments:

```ts
function sum(a: number, b: number): number {
  return a + b;
}
```

A rest parameter accepts any number of arguments:

```ts
function sum(...numbers: number[]): number {
  return numbers.reduce(
    (total, number) => total + number,
    0
  );
}
```

```ts
sum(1, 2);
sum(1, 2, 3, 4);
sum(10, 20, 30, 40, 50);
```

The arguments are collected into an array:

```txt
sum(1, 2, 3)

numbers = [1, 2, 3]
```

### Rest vs Spread

The same three dots have opposite meanings depending on location.

| | Rest | Spread |
|---|---|---|
| Location | Function parameters | Function calls, arrays, objects |
| Action | Collects values | Expands values |
| Example | `(...values)` | `sum(...numbers)` |

Rest collects:

```ts
function log(...messages: string[]) {
  console.log(messages);
}
```

Spread expands:

```ts
const numbers = [1, 2, 3];

sum(...numbers);
```

### `reduce` and Rest Parameters

```ts
const total = numbers.reduce(
  (accumulator, currentValue) =>
    accumulator + currentValue,
  0
);
```

The accumulator moves through the array:

```txt
initial: 0
0 + 1 = 1
1 + 2 = 3
3 + 3 = 6
```

The second argument, `0`, is the initial accumulator value.

---

## 12 — Interfaces Describe Object Shapes

An interface describes what properties an object should contain.

```ts
interface User {
  id: number;
  name: string;
  email: string;
}
```

Use it like this:

```ts
const user: User = {
  id: 1,
  name: "Alex",
  email: "alex@example.com"
};
```

This is invalid:

```ts
const user: User = {
  id: "1",
  name: "Alex"
};
```

The `id` has the wrong type, and `email` is missing.

### Optional Properties

Use `?` for properties that may not exist:

```ts
interface User {
  id: number;
  name: string;
  email?: string;
}
```

Both are valid:

```ts
const first: User = {
  id: 1,
  name: "Alex"
};

const second: User = {
  id: 2,
  name: "Jordan",
  email: "jordan@example.com"
};
```

### Readonly Properties

```ts
interface User {
  readonly id: number;
  name: string;
}
```

```ts
user.id = 10; // error
```

`readonly` prevents reassignment through that type.

It does not automatically make nested objects deeply immutable.

### Nested Interfaces

```ts
interface Address {
  city: string;
  zip: string;
}

interface User {
  name: string;
  address: Address;
}
```

```ts
const user: User = {
  name: "Alex",
  address: {
    city: "Berlin",
    zip: "10115"
  }
};
```

### Interfaces vs Type Aliases

This interface:

```ts
interface User {
  name: string;
}
```

is similar to:

```ts
type User = {
  name: string;
};
```

Both can describe object shapes.

Type aliases are especially useful for unions:

```ts
type Status = "pending" | "success" | "error";
```

Interfaces are often preferred for extendable object contracts.

---

## 13 — Type Aliases Create Reusable Types

A type alias gives a name to a type.

```ts
type UserId = number;
type Username = string;
```

```ts
const id: UserId = 10;
const name: Username = "Alex";
```

Type aliases become more useful for complex types:

```ts
type Status = "pending" | "success" | "error";

type Result =
  | {
      status: "success";
      data: string;
    }
  | {
      status: "error";
      message: string;
    };
```

### Object Type Alias

```ts
type Product = {
  id: number;
  title: string;
  price: number;
};
```

### Combining Types

```ts
type HasId = {
  id: number;
};

type HasName = {
  name: string;
};

type User = HasId & HasName;
```

`&` creates an intersection type.

A `User` must contain both:

```ts
const user: User = {
  id: 1,
  name: "Alex"
};
```

### Interfaces Can Extend Interfaces

```ts
interface Person {
  name: string;
}

interface Employee extends Person {
  employeeId: number;
}
```

```ts
const employee: Employee = {
  name: "Alex",
  employeeId: 100
};
```

---

## 14 — Destructuring Is Pattern Matching

Destructuring extracts values from arrays and objects.

### Array Destructuring

```ts
const [first, second, third] = [1, 2, 3];

console.log(first);  // 1
console.log(second); // 2
console.log(third);  // 3
```

Skip elements with empty positions:

```ts
const [, second, , fourth] = [1, 2, 3, 4];

console.log(second); // 2
console.log(fourth); // 4
```

### Default Values

```ts
const [x = 10, y = 20] = [5];

console.log(x); // 5
console.log(y); // 20
```

The default is used only when the value is `undefined`.

### Rest Destructuring

```ts
const [first, ...rest] = [1, 2, 3, 4];

console.log(first); // 1
console.log(rest);  // [2, 3, 4]
```

The rest element must be last.

### Swapping Variables

```ts
let a = 1;
let b = 2;

[a, b] = [b, a];

console.log(a); // 2
console.log(b); // 1
```

### Object Destructuring

```ts
const user = {
  name: "Alex",
  age: 30,
  role: "developer"
};

const { name, age } = user;
```

### Renaming Properties

```ts
const {
  name: userName,
  age: userAge
} = user;
```

The original object properties are still called `name` and `age`.

The local variables are called `userName` and `userAge`.

### Default Object Values

```ts
const {
  title: postTitle = "Untitled"
} = {};
```

```ts
console.log(postTitle); // "Untitled"
```

### Destructuring Function Parameters

```ts
function greet({
  name,
  age
}: {
  name: string;
  age: number;
}) {
  console.log(name, age);
}
```

This is cleaner than:

```ts
function greet(user: { name: string; age: number }) {
  console.log(user.name, user.age);
}
```

### Important Destructuring Rule

Defaults handle `undefined`, but not `null`.

```ts
const { name = "Anonymous" } = undefined as any;
// default is used
```

```ts
const { name = "Anonymous" } = null as any;
// runtime error
```

`null` cannot be destructured like an object.

---

## 15 — Spread and Rest Are Not the Same Thing

The three dots have different behavior depending on context.

### Spread Arrays

```ts
const first = [1, 2, 3];
const second = [4, 5, 6];

const combined = [...first, ...second];

console.log(combined);
// [1, 2, 3, 4, 5, 6]
```

### Spread Objects

```ts
const defaults = {
  color: "blue",
  size: "medium"
};

const overrides = {
  size: "large",
  weight: "bold"
};

const result = {
  ...defaults,
  ...overrides
};
```

Later properties overwrite earlier properties:

```ts
{
  color: "blue",
  size: "large",
  weight: "bold"
}
```

### Object Spread Is Shallow

```ts
const original = {
  user: {
    name: "Alex"
  }
};

const copy = {
  ...original
};

copy.user.name = "Jordan";

console.log(original.user.name);
// "Jordan"
```

The outer object was copied, but the nested `user` object still has the same reference.

For a structured clone:

```ts
const deepCopy = structuredClone(original);
```

Or use a carefully chosen serialization or cloning strategy appropriate to the data.

### Rest in Destructuring

```ts
const user = {
  name: "Alex",
  age: 30,
  role: "developer"
};

const {
  name,
  ...rest
} = user;
```

```ts
console.log(name);
// "Alex"

console.log(rest);
// { age: 30, role: "developer" }
```

Rest collects the remaining values.

```txt
Spread → expands
Rest   → collects
```

---

## 16 — Optional Chaining Prevents Unsafe Access

Optional chaining uses `?.`.

It stops evaluation when the value before it is `null` or `undefined`.

Without optional chaining:

```ts
const city =
  user &&
  user.address &&
  user.address.city;
```

With optional chaining:

```ts
const city = user?.address?.city;
```

If `user` or `address` is missing, the result is `undefined` instead of an error.

### Optional Property Access

```ts
const name = user?.name;
```

### Optional Method Calls

```ts
const result = user?.getName?.();
```

The method is called only if it exists.

### Optional Array Access

```ts
const first = products?.[0];
```

### Dynamic Property Names

```ts
const value = object?.["dynamic-key"];
```

### Optional Chaining Does Not Handle Every Falsy Value

Optional chaining only stops for:

- `null`
- `undefined`

It does not stop for:

- `0`
- `false`
- `""`

```ts
const count = object?.count;
```

If `count` is `0`, the result is correctly `0`.

---

## 17 — Nullish Coalescing Preserves Valid Falsy Values

The nullish coalescing operator is `??`.

It uses the right side only when the left side is:

- `null`
- `undefined`

```ts
const port = config.port ?? 3000;
```

If:

```ts
config.port = 0;
```

the result is:

```ts
0
```

This is different from `||`:

```ts
const port = config.port || 3000;
```

With `||`, all falsy values trigger the fallback:

- `0`
- `false`
- `""`
- `null`
- `undefined`
- `NaN`

### Comparison

```ts
0 || 42;        // 42
0 ?? 42;        // 0

"" || "default";  // "default"
"" ?? "default";  // ""

false || true;   // true
false ?? true;   // false

null || "value"; // "value"
null ?? "value"; // "value"
```

Use `??` when only missing values should trigger the fallback.

### Logical Assignment

```ts
user.name ??= "Anonymous";
```

This assigns only when `user.name` is `null` or `undefined`.

Other logical assignment operators:

```ts
config.debug ||= false;
cache.value ??= calculateValue();
enabled &&= isAllowed;
```

| Operator | Assigns when |
|---|---|
| `||=` | Left side is falsy |
| `??=` | Left side is nullish |
| `&&=` | Left side is truthy |

---

## 18 — Symbols Are Invisible Unique Keys

A symbol is a primitive value that is always unique.

```ts
const id1 = Symbol("id");
const id2 = Symbol("id");

console.log(id1 === id2);
// false
```

Even with the same description, every symbol is different.

### Symbols as Object Keys

```ts
const id = Symbol("id");

const user = {
  name: "Alex",
  [id]: 12345
};
```

Access the symbol property:

```ts
user[id];
// 12345
```

This does not access the same property:

```ts
user.id;
// undefined
```

The string `"id"` and the symbol `Symbol("id")` are different keys.

### Symbols Do Not Appear in Normal Enumeration

```ts
Object.keys(user);
// ["name"]
```

```ts
JSON.stringify(user);
// {"name":"Alex"}
```

Find symbol keys with:

```ts
Object.getOwnPropertySymbols(user);
```

### Well-Known Symbols

JavaScript uses special symbols to define protocols:

```ts
Symbol.iterator
Symbol.toPrimitive
Symbol.hasInstance
Symbol.toStringTag
```

For example, `Symbol.iterator` makes an object work with `for...of`.

```ts
class Range {
  constructor(
    private start: number,
    private end: number
  ) {}

  *[Symbol.iterator]() {
    for (let current = this.start; current <= this.end; current++) {
      yield current;
    }
  }
}

for (const number of new Range(1, 5)) {
  console.log(number);
}
```

Symbols allow objects to participate in JavaScript language behavior without relying on ordinary string property names.

---

## 19 — Iterators and Generators Create Sequences

An iterator is an object with a `next()` method.

```ts
const iterator = {
  values: [10, 20, 30],
  index: 0,

  next() {
    if (this.index < this.values.length) {
      return {
        value: this.values[this.index++],
        done: false
      };
    }

    return {
      value: undefined,
      done: true
    };
  }
};
```

```ts
iterator.next();
// { value: 10, done: false }

iterator.next();
// { value: 20, done: false }

iterator.next();
// { value: 30, done: false }

iterator.next();
// { value: undefined, done: true }
```

### Iterable Objects

An iterable object defines `[Symbol.iterator]()`:

```ts
const iterable = {
  values: [1, 2, 3],

  *[Symbol.iterator]() {
    yield* this.values;
  }
};
```

Now it can be used with:

```ts
for (const value of iterable) {
  console.log(value);
}
```

### Generators

A generator function uses `function*` and `yield`.

```ts
function* range(start: number, end: number) {
  for (let value = start; value <= end; value++) {
    yield value;
  }
}
```

```ts
for (const value of range(1, 5)) {
  console.log(value);
}
```

Generators are lazy.

Nothing runs until the generator is pulled:

```ts
const generator = range(1, 3);

generator.next();
// { value: 1, done: false }

generator.next();
// { value: 2, done: false }

generator.next();
// { value: 3, done: false }

generator.next();
// { value: undefined, done: true }
```

### Generators Can Receive Values

```ts
function* conversation() {
  const name = yield "What is your name?";
  const age = yield `Hello ${name}. How old are you?`;

  return `${name} is ${age} years old.`;
}

const generator = conversation();

generator.next();
// { value: "What is your name?", done: false }

generator.next("Alex");
// { value: "Hello Alex. How old are you?", done: false }

generator.next(30);
// { value: "Alex is 30 years old.", done: true }
```

Generators are useful for:

- Lazy sequences
- Pagination
- Custom iteration
- Streaming data
- State machines
- Controlled execution

---

## 20 — Maps Store Values Under Any Key

A plain object normally uses strings or symbols as keys.

A `Map` can use almost anything:

```ts
const map = new Map();

const userKey = {
  id: 1
};

map.set(userKey, "User data");
map.set(42, "The answer");
map.set(true, "Boolean key");
```

Read values:

```ts
map.get(userKey);
map.get(42);
map.get(true);
```

Check for a key:

```ts
map.has(42);
```

Remove a key:

```ts
map.delete(42);
```

Read the size:

```ts
map.size;
```

### Map Iteration

```ts
for (const [key, value] of map) {
  console.log(key, value);
}
```

A `Map` preserves insertion order.

### Object vs Map

Use a plain object when:

- Keys are strings
- The shape is known
- JSON serialization is needed
- You are representing a record

Use a `Map` when:

- Keys are not only strings
- Frequent additions and removals occur
- Insertion order matters
- You need `.size`
- You need direct key-based operations

### Converting Between Objects and Maps

```ts
const map = new Map(
  Object.entries({
    a: 1,
    b: 2
  })
);
```

```ts
const object = Object.fromEntries(map);
```

---

## 21 — Sets Store Unique Values

A `Set` stores unique values.

```ts
const values = new Set([1, 2, 3, 2, 1]);

console.log(values);
// Set { 1, 2, 3 }
```

Add a value:

```ts
values.add(4);
```

Check for a value:

```ts
values.has(2);
```

Delete a value:

```ts
values.delete(2);
```

Read the size:

```ts
values.size;
```

### Removing Duplicates

```ts
const numbers = [1, 2, 2, 3, 3, 4];

const uniqueNumbers = [
  ...new Set(numbers)
];

console.log(uniqueNumbers);
// [1, 2, 3, 4]
```

### Set Operations

```ts
const a = new Set([1, 2, 3, 4]);
const b = new Set([3, 4, 5, 6]);

const union = new Set([
  ...a,
  ...b
]);

const intersection = new Set(
  [...a].filter((value) => b.has(value))
);

const difference = new Set(
  [...a].filter((value) => !b.has(value))
);
```

```txt
union        → {1, 2, 3, 4, 5, 6}
intersection → {3, 4}
difference   → {1, 2}
```

### WeakMap and WeakSet

`WeakMap` keys and `WeakSet` values must be objects.

They hold references weakly, allowing garbage collection when no other references exist.

```ts
const cache = new WeakMap<object, unknown>();

function processUser(user: object) {
  if (cache.has(user)) {
    return cache.get(user);
  }

  const result = expensiveComputation(user);
  cache.set(user, result);
  return result;
}
```

When `user` is no longer referenced elsewhere, the cache entry can be removed by garbage collection.

A regular `Map` would keep the object alive.

---

## 22 — Proxies Intercept Object Operations

A `Proxy` wraps an object and intercepts operations such as:

- Reading a property
- Writing a property
- Deleting a property
- Calling a function
- Constructing an object

```ts
const handler = {
  get(target: object, key: string | symbol) {
    console.log(`Reading ${String(key)}`);
    return Reflect.get(target, key);
  },

  set(
    target: object,
    key: string | symbol,
    value: unknown
  ) {
    console.log(`Writing ${String(key)}`);
    return Reflect.set(target, key, value);
  }
};

const user = new Proxy(
  {
    name: "Alex"
  },
  handler
);
```

```ts
user.name;
// logs "Reading name"

user.name = "Jordan";
// logs "Writing name"
```

### Use `Reflect` Inside Proxy Traps

This is the correct forwarding pattern:

```ts
get(target, key) {
  return Reflect.get(target, key);
}
```

Avoid directly bypassing the proxy mechanism:

```ts
target[key];
```

`Reflect` provides the standard operation API and correctly forwards:

- The target
- The property key
- The receiver
- The return behavior

### Validation Proxy

```ts
function createTypedObject(
  schema: Record<string, string>
) {
  return new Proxy({} as Record<string, unknown>, {
    set(target, key, value) {
      const expectedType = schema[String(key)];

      if (
        expectedType &&
        typeof value !== expectedType
      ) {
        throw new TypeError(
          `${String(key)} must be ${expectedType}`
        );
      }

      target[key] = value;
      return true;
    }
  });
}

const user = createTypedObject({
  name: "string",
  age: "number"
});

user.name = "Alex";
user.age = 30;

user.age = "thirty";
// TypeError
```

Proxies are powerful, but they add invisible behavior.

Use them carefully because they can make debugging and performance analysis harder.

---

## 23 — Tagged Template Literals Transform Strings

A normal template literal interpolates values:

```ts
const name = "Alex";

const message = `Hello, ${name}!`;
```

A tagged template passes the template to a function.

```ts
function tag(
  strings: TemplateStringsArray,
  ...values: unknown[]
) {
  console.log(strings);
  console.log(values);

  return strings[0] + String(values[0]) + strings[1];
}

tag`Hello, ${name}!`;
```

The tag function receives:

```txt
strings → raw string sections
values  → interpolated values
```

For:

```ts
tag`Hello, ${name}!`;
```

the arrays are conceptually:

```ts
strings = ["Hello, ", "!"];
values = ["Alex"];
```

### SQL Tag

A tagged template can separate query text from values:

```ts
function sql(
  strings: TemplateStringsArray,
  ...values: unknown[]
) {
  const query = strings.reduce(
    (result, string, index) =>
      result +
      string +
      (values[index] !== undefined
        ? `$${index + 1}`
        : ""),
    ""
  );

  return {
    query,
    params: values
  };
}
```

```ts
const id = userInput;

const result = sql`
  SELECT * FROM users
  WHERE id = ${id}
`;
```

Conceptually:

```ts
{
  query: "SELECT * FROM users WHERE id = $1",
  params: [userInput]
}
```

The value is passed as a parameter instead of being directly concatenated into SQL.

### HTML Sanitization

A tag can escape dangerous content:

```ts
function html(
  strings: TemplateStringsArray,
  ...values: unknown[]
) {
  const escape = (value: unknown) =>
    String(value)
      .replace(/&/g, "&amp;")
      .replace(/</g, "&lt;")
      .replace(/>/g, "&gt;");

  return strings.reduce(
    (result, string, index) =>
      result +
      string +
      (values[index] !== undefined
        ? escape(values[index])
        : ""),
    ""
  );
}
```

```ts
const comment =
  "<script>alert('xss')</script>";

const safe = html`<p>${comment}</p>`;
```

Tagged templates are not just string formatting.

They are a way to define a small language inside JavaScript.

---

## 24 — Getters and Setters Create Computed Properties

A getter looks like a normal property but runs a function when accessed.

```ts
const circle = {
  radius: 5,

  get area() {
    return Math.PI * this.radius ** 2;
  },

  get circumference() {
    return 2 * Math.PI * this.radius;
  }
};
```

Use it like a property:

```ts
circle.area;
circle.circumference;
```

No parentheses are required.

The values are calculated every time they are accessed.

```ts
circle.radius = 10;
circle.area;
```

The area changes automatically because it is derived from the current radius.

### Setter Validation

```ts
class Temperature {
  #celsius = 0;

  get celsius() {
    return this.#celsius;
  }

  set celsius(value: number) {
    if (value < -273.15) {
      throw new RangeError(
        "Below absolute zero"
      );
    }

    this.#celsius = value;
  }

  get fahrenheit() {
    return this.#celsius * 9 / 5 + 32;
  }

  set fahrenheit(value: number) {
    this.celsius = (value - 32) * 5 / 9;
  }
}
```

```ts
const temperature = new Temperature();

temperature.celsius = 100;
temperature.fahrenheit;
// 212

temperature.celsius = -300;
// RangeError
```

Getters and setters allow an object to control how properties are read and written.

### Defining Accessors on Existing Objects

```ts
const person = {
  first: "Alex",
  last: "Thakor"
};

Object.defineProperty(person, "fullName", {
  get() {
    return `${this.first} ${this.last}`;
  },

  set(value: string) {
    [this.first, this.last] = value.split(" ");
  },

  enumerable: true,
  configurable: true
});
```

```ts
person.fullName;
// "Alex Thakor"

person.fullName = "Jordan Smith";

person.first;
// "Jordan"
```

---

## 25 — Private Class Fields Provide Real Encapsulation

Before private class fields, developers often used naming conventions:

```ts
class BankAccount {
  _balance = 0;
}
```

The underscore did not actually make the field private.

Anyone could still access it:

```ts
account._balance;
```

Modern JavaScript supports private fields using `#`.

```ts
class BankAccount {
  #balance: number;
  #owner: string;

  constructor(
    owner: string,
    initialBalance: number
  ) {
    this.#owner = owner;
    this.#balance = initialBalance;
  }

  deposit(amount: number) {
    if (amount <= 0) {
      throw new Error(
        "Amount must be positive"
      );
    }

    this.#balance += amount;
    return this;
  }

  withdraw(amount: number) {
    if (amount > this.#balance) {
      throw new Error(
        "Insufficient funds"
      );
    }

    this.#balance -= amount;
    return this;
  }

  get balance() {
    return this.#balance;
  }

  toString() {
    return `${this.#owner}: $${this.#balance}`;
  }
}
```

```ts
const account = new BankAccount(
  "Alex",
  1000
);

account.deposit(500);
account.withdraw(200);

account.balance;
// 1300
```

This is invalid:

```ts
account.#balance;
```

Private fields are enforced by the JavaScript engine, not only by TypeScript.

### Private Static Fields

```ts
class IdGenerator {
  static #nextId = 1;

  static generate() {
    return IdGenerator.#nextId++;
  }
}

IdGenerator.generate();
// 1

IdGenerator.generate();
// 2
```

The field belongs to the class rather than to each instance.

---

## 26 — Regular Expressions Are a Language Inside JavaScript

Regular expressions describe text patterns.

```ts
const pattern = /^\d+$/;
```

This matches a string containing only digits.

```ts
pattern.test("123");
// true

pattern.test("12x");
// false
```

### Regex Literal vs Constructor

```ts
const literal = /hello/i;
```

The pattern is compiled as part of the source.

```ts
const dynamic = new RegExp("hello", "i");
```

The constructor is useful when the pattern is dynamic.

### Common Flags

| Flag | Meaning |
|---|---|
| `i` | Case-insensitive |
| `g` | Global matching |
| `m` | Multiline mode |
| `s` | Dot matches newline |
| `u` | Unicode mode |
| `d` | Match indices |

### `test`

```ts
/^\d+$/.test("123");
// true
```

Returns a boolean.

### `match`

```ts
"hello world".match(/\w+/);
```

Returns the first match.

With the global flag:

```ts
"hello world".match(/\w+/g);
// ["hello", "world"]
```

### `matchAll`

```ts
const text = "2024-01-15";

for (const match of text.matchAll(
  /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/g
)) {
  console.log(match.groups);
}
```

Named capture groups make the result easier to understand.

### Replace

```ts
"hello".replace(/l/, "r");
// "herlo"
```

With a global flag:

```ts
"hello".replace(/l/g, "r");
// "herro"
```

With a function:

```ts
"hello world".replace(
  /\b\w/g,
  (character) => character.toUpperCase()
);
// "Hello World"
```

### Lookahead

A positive lookahead checks what follows without consuming it:

```ts
/\d+(?= dollars)/.exec(
  "100 dollars and 50 cents"
);
```

A negative lookahead checks what does not follow:

```ts
/\d+(?! dollars)/.exec(
  "100 dollars and 50 cents"
);
```

Regular expressions are powerful, but they should remain readable.

For complex validation, named functions or dedicated parsers may be easier to maintain.

---

## 27 — Dates Are Full of Historical Problems

JavaScript's `Date` API has several awkward behaviors.

### Zero-Based Months

```ts
new Date(2024, 0, 1);
// January 1, 2024

new Date(2024, 11, 31);
// December 31, 2024
```

Months start at `0`:

```txt
0  → January
1  → February
11 → December
```

Days start at `1`.

This inconsistency is a common source of bugs.

### Date Parsing Can Be Environment-Dependent

```ts
new Date("2024-01-15");
new Date("2024/01/15");
new Date("January 15, 2024");
```

Different formats can be interpreted differently depending on the environment and timezone.

Prefer standardized formats when possible:

```ts
new Date("2024-01-15T00:00:00.000Z");
```

### Extracting Date Parts

```ts
const date = new Date();

date.getFullYear();
date.getMonth();
date.getDate();
date.getDay();
date.getHours();
date.getMinutes();
date.getSeconds();
```

Remember:

```txt
getMonth() → 0 to 11
getDate()  → 1 to 31
getDay()   → 0 to 6
```

`getDay()` represents the weekday:

```txt
0 → Sunday
1 → Monday
...
6 → Saturday
```

### Timestamps

```ts
Date.now();
date.getTime();
```

These return milliseconds since the Unix epoch:

```txt
January 1, 1970 UTC
```

### Formatting Dates

Use `Intl.DateTimeFormat` for display:

```ts
const formatter = new Intl.DateTimeFormat(
  "en-US",
  {
    year: "numeric",
    month: "long",
    day: "numeric"
  }
);

formatter.format(new Date());
// "January 15, 2024"
```

### Relative Time

```ts
const relative = new Intl.RelativeTimeFormat(
  "en",
  {
    numeric: "auto"
  }
);

relative.format(-2, "day");
// "2 days ago"
```

For complex date and timezone operations, use a dedicated modern library or the Temporal API when available.

---

## 28 — Useful Array Methods Beyond `map`

Most developers know:

```ts
map
filter
reduce
```

But many other array methods are useful.

### `flat`

```ts
const nested = [
  1,
  [2, 3],
  [4, [5, 6]]
];

nested.flat();
// [1, 2, 3, 4, [5, 6]]
```

Specify a depth:

```ts
nested.flat(2);
// [1, 2, 3, 4, 5, 6]
```

### `flatMap`

```ts
const sentences = [
  "hello world",
  "foo bar"
];

const words = sentences.flatMap(
  (sentence) => sentence.split(" ")
);

// ["hello", "world", "foo", "bar"]
```

This combines:

```txt
map + flat(1)
```

### `find`

```ts
const product = products.find(
  (product) => product.id === 3
);
```

Returns the first matching value.

### `findIndex`

```ts
const index = products.findIndex(
  (product) => product.id === 3
);
```

Returns the matching index.

### `findLast`

```ts
const last = numbers.findLast(
  (number) => number < 3
);
```

Searches from the end.

### `at`

```ts
const last = numbers.at(-1);
const secondLast = numbers.at(-2);
```

This is cleaner than:

```ts
numbers[numbers.length - 1];
```

### `includes`

```ts
numbers.includes(3);
```

Checks whether a value exists.

Unlike `indexOf`, `includes` correctly handles `NaN`:

```ts
[NaN].includes(NaN);
// true
```

### `some` and `every`

```ts
numbers.some((number) => number > 10);
numbers.every((number) => number > 0);
```

```txt
some   → at least one matches
every  → all must match
```

### `reduceRight`

```ts
const result = arrays.reduceRight(
  (accumulator, array) => [
    ...accumulator,
    ...array
  ],
  []
);
```

It reduces from the right side instead of the left.

### Non-Mutating Methods

Modern JavaScript provides methods that return a new array:

```ts
const sorted = numbers.toSorted();
const reversed = numbers.toReversed();
const updated = numbers.with(1, 99);
```

The original array remains unchanged.

This is useful when working with immutable state.

---

## 29 — Object Methods Give You Low-Level Control

The `Object` API is larger than most developers realize.

### Enumerating Properties

```ts
Object.keys(object);
Object.values(object);
Object.entries(object);
```

```ts
const user = {
  name: "Alex",
  age: 30
};

Object.keys(user);
// ["name", "age"]

Object.values(user);
// ["Alex", 30]

Object.entries(user);
// [["name", "Alex"], ["age", 30]]
```

### Including Non-Enumerable and Symbol Keys

```ts
Object.getOwnPropertyNames(object);
Object.getOwnPropertySymbols(object);
Reflect.ownKeys(object);
```

`Reflect.ownKeys()` includes both string and symbol keys.

### Creating Objects With a Prototype

```ts
const object = Object.create(null);
```

This object has no prototype.

It does not inherit:

```ts
object.toString;
object.hasOwnProperty;
```

This can be useful for dictionary-like objects, but it also means you lose normal object methods.

### Merging Objects

```ts
const merged = Object.assign(
  {},
  defaults,
  overrides
);
```

This performs a shallow copy.

Spread syntax is often clearer:

```ts
const merged = {
  ...defaults,
  ...overrides
};
```

### Freeze, Seal, and Prevent Extensions

```ts
Object.freeze(object);
```

Prevents adding, deleting, or modifying properties.

```ts
Object.seal(object);
```

Prevents adding or deleting properties, but existing values may still be changed.

```ts
Object.preventExtensions(object);
```

Prevents adding new properties.

### Property Descriptors

```ts
Object.getOwnPropertyDescriptor(
  object,
  "name"
);
```

A property descriptor can contain:

```ts
{
  value: "Alex",
  writable: true,
  enumerable: true,
  configurable: true
}
```

Define a property manually:

```ts
Object.defineProperty(object, "id", {
  value: 42,
  writable: false,
  enumerable: true,
  configurable: false
});
```

Object descriptors expose the lower-level rules behind JavaScript properties.

---

## 30 — TypeScript With React

TypeScript becomes especially useful in React because components exchange data through props, state, and API responses.

### Typing Component Props

```tsx
interface Product {
  id: number;
  title: string;
  price: number;
}

interface ProductCardProps {
  product: Product;
}

const ProductCard = ({
  product
}: ProductCardProps) => {
  return (
    <article>
      <h2>{product.title}</h2>
      <p>${product.price}</p>
    </article>
  );
};
```

A shorter inline version:

```tsx
const ProductCard = ({
  product
}: {
  product: Product;
}) => {
  return <h2>{product.title}</h2>;
};
```

Use a named interface when the props are reused or complex.

### Typing State

```tsx
const [count, setCount] = useState<number>(0);
const [name, setName] = useState<string>("");
const [products, setProducts] = useState<Product[]>([]);
```

TypeScript can often infer primitive state:

```tsx
const [count, setCount] = useState(0);
```

But an empty array needs an explicit type:

```tsx
const [products, setProducts] =
  useState<Product[]>([]);
```

### Nullable State

```tsx
const [user, setUser] =
  useState<User | null>(null);
```

When using it:

```tsx
return (
  <h1>
    {user ? user.name : "Not logged in"}
  </h1>
);
```

Or:

```tsx
<h1>{user?.name}</h1>
```

### Typing API Data

```tsx
interface Product {
  id: number;
  title: string;
  category: string;
  image: string;
  price: number;
  description: string;
  rating: {
    rate: number;
    count: number;
  };
}
```

```tsx
const [products, setProducts] =
  useState<Product[]>([]);

const getProducts = async () => {
  const response =
    await axios.get<Product[]>(
      "https://api.example.com/products"
    );

  setProducts(response.data);
};
```

Providing the generic to Axios is better than allowing `response.data` to become `any`.

### Typing Events

```tsx
const handleChange = (
  event: React.ChangeEvent<HTMLInputElement>
) => {
  console.log(event.target.value);
};
```

```tsx
const handleSubmit = (
  event: React.FormEvent<HTMLFormElement>
) => {
  event.preventDefault();
};
```

For a button:

```tsx
const handleClick = (
  event: React.MouseEvent<HTMLButtonElement>
) => {
  console.log(event.currentTarget);
};
```

### Typing `children`

```tsx
interface LayoutProps {
  children: React.ReactNode;
}

const Layout = ({
  children
}: LayoutProps) => {
  return <main>{children}</main>;
};
```

### Typing `useRef`

```tsx
const inputRef =
  useRef<HTMLInputElement | null>(null);
```

```tsx
inputRef.current?.focus();
```

### Typing `useReducer`

```tsx
type State = {
  count: number;
};

type Action =
  | { type: "increment" }
  | { type: "decrement" };

function reducer(
  state: State,
  action: Action
): State {
  switch (action.type) {
    case "increment":
      return {
        count: state.count + 1
      };

    case "decrement":
      return {
        count: state.count - 1
      };

    default:
      return assertNever(action);
  }
}
```

Discriminated unions make reducer actions safe and exhaustive.

---

## 31 — TypeScript With Vite and React

A Vite React TypeScript project commonly includes:

```txt
tsconfig.json
tsconfig.app.json
tsconfig.node.json
vite.config.ts
src/
  main.tsx
  App.tsx
```

### Root `tsconfig.json`

```json
{
  "files": [],
  "references": [
    {
      "path": "./tsconfig.app.json"
    },
    {
      "path": "./tsconfig.node.json"
    }
  ]
}
```

The root config delegates to smaller configurations.

### Application Configuration

The application config governs browser code:

```json
{
  "compilerOptions": {
    "lib": ["ES2023", "DOM"],
    "types": ["vite/client"],
    "module": "ESNext",
    "jsx": "react-jsx",
    "moduleResolution": "Bundler",
    "noEmit": true,
    "strict": true
  },
  "include": ["src"]
}
```

The DOM library is needed for:

```ts
document;
window;
HTMLElement;
fetch;
```

The Vite client types are needed for:

```ts
import.meta.env;
```

`jsx: "react-jsx"` enables JSX in `.tsx` files.

`noEmit: true` means TypeScript checks the code but does not generate JavaScript.

Vite performs the actual transformation and bundling.

### Node Configuration

The Node configuration applies to build tooling such as:

```txt
vite.config.ts
```

It may use:

```json
{
  "compilerOptions": {
    "lib": ["ES2023"],
    "types": ["node"],
    "module": "NodeNext"
  },
  "include": ["vite.config.ts"]
}
```

The browser and Node environments have different globals.

Separating their configurations prevents incorrect assumptions about which APIs exist.

### Useful Compiler Flags

```json
{
  "compilerOptions": {
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "noEmit": true
  }
}
```

| Option | Purpose |
|---|---|
| `strict` | Enables stronger type checking |
| `noUnusedLocals` | Reports unused local variables |
| `noUnusedParameters` | Reports unused parameters |
| `noFallthroughCasesInSwitch` | Detects accidental switch fallthrough |
| `noEmit` | Checks without generating JavaScript |

---

## 32 — `import type` Separates Types From Runtime Values

When a symbol exists only as a type, use a type-only import:

```ts
import type { Product } from "./types";
```

This is especially important with:

```json
{
  "verbatimModuleSyntax": true
}
```

An interface does not exist at runtime:

```ts
interface Product {
  id: number;
}
```

Therefore, importing it as a runtime value would be incorrect.

```ts
import { Product } from "./types";
```

Use:

```ts
import type { Product } from "./types";
```

For a file containing both values and types:

```ts
import { createProduct } from "./products";
import type { Product } from "./types";
```

This makes the boundary explicit:

```txt
Runtime import → exists in JavaScript
Type import    → erased during compilation
```

---

## 33 — Non-Null Assertions Trust You

DOM APIs often return nullable values:

```ts
const root =
  document.getElementById("root");
```

The inferred type is:

```ts
HTMLElement | null
```

TypeScript requires you to handle the possibility of `null`:

```ts
if (!root) {
  throw new Error("Root element not found");
}

createRoot(root).render(<App />);
```

You can also use the non-null assertion operator:

```ts
createRoot(
  document.getElementById("root")!
).render(<App />);
```

The `!` tells TypeScript:

> Trust me. This value will not be null.

But it does not add a runtime check.

If the element is missing, the program can still crash.

```txt
Non-null assertion:
  silences the compiler

It does not:
  validate the value
  create the missing element
  prevent runtime errors
```

Use it only when the assumption is guaranteed by the application structure.

---

## 34 — Generics Let Types Work Across Many Values

A generic is a type parameter.

It allows one function or structure to work with multiple types while preserving type information.

```ts
function identity<T>(value: T): T {
  return value;
}
```

```ts
const numberValue = identity(10);
// number

const stringValue = identity("hello");
// string
```

The same function works for different types.

### Generic Arrays

```ts
function first<T>(items: T[]): T | undefined {
  return items[0];
}
```

```ts
const number = first([1, 2, 3]);
// number | undefined

const name = first(["Alex", "Jordan"]);
// string | undefined
```

### Generic API Response

```ts
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}
```

```ts
type UserResponse = ApiResponse<User>;
type ProductResponse = ApiResponse<Product[]>;
```

### Generic Constraints

```ts
function getId<T extends { id: number }>(
  value: T
) {
  return value.id;
}
```

The function accepts any object containing a numeric `id`.

```ts
getId({ id: 1, name: "Alex" });
```

This is invalid:

```ts
getId({ name: "Alex" });
```

### Generic React Hook

```tsx
function useFetch<T>(url: string) {
  const [data, setData] =
    useState<T | null>(null);

  // fetch implementation

  return data;
}
```

```tsx
const products =
  useFetch<Product[]>("/api/products");
```

Generics provide reusable code without falling back to `any`.

---

## 35 — TypeScript Cannot Replace Runtime Validation

This looks safe:

```ts
interface User {
  name: string;
  age: number;
}

const user =
  JSON.parse(input) as User;
```

But `as User` does not validate anything.

It only tells TypeScript to trust you.

If the input is:

```json
{
  "name": 123,
  "age": "unknown"
}
```

the cast still succeeds at compile time.

### Type Assertions Are Not Validation

```ts
const value = unknownData as Product;
```

This changes how TypeScript treats the value.

It does not transform or inspect the value.

### Manual Validation

```ts
function isUser(value: unknown): value is User {
  if (typeof value !== "object" || value === null) {
    return false;
  }

  const user = value as Record<string, unknown>;

  return (
    typeof user.name === "string" &&
    typeof user.age === "number"
  );
}
```

```ts
if (isUser(data)) {
  console.log(data.name);
}
```

The function is a type guard:

```ts
value is User
```

It tells TypeScript that the value can be treated as a `User` when the function returns `true`.

For larger applications, use a runtime schema validation library.

---

## 36 — Performance Is Usually About the Wrong Things

JavaScript performance is rarely improved by tiny syntax changes.

The expensive operations are usually:

- Excessive DOM manipulation
- Forced synchronous layout
- Repeated expensive calculations
- Unnecessary allocations
- Uncontrolled event frequency
- Unstable object shapes
- Large synchronous loops

### Avoid Repeated DOM Layout

This repeatedly reads and writes layout:

```js
elements.forEach((element) => {
  const height = element.offsetHeight;
  element.style.height = `${height + 10}px`;
});
```

A read can force the browser to calculate layout.

A better approach batches reads first:

```js
const heights = elements.map(
  (element) => element.offsetHeight
);

elements.forEach((element, index) => {
  element.style.height =
    `${heights[index] + 10}px`;
});
```

```txt
Bad:
  read → write → read → write

Better:
  read → read → read
  write → write → write
```

### Keep Object Shapes Consistent

JavaScript engines optimize objects based on their property structure.

Prefer:

```js
function createUser(name, age) {
  return {
    name,
    age
  };
}
```

Avoid creating the same conceptual object with different property orders:

```js
{
  name: "Alex",
  age: 30
}

{
  age: 30,
  name: "Alex"
}
```

Consistent shapes give engines better optimization opportunities.

### Debounce Expensive Input

```ts
function debounce<T extends (...args: any[]) => void>(
  callback: T,
  delay: number
) {
  let timer: ReturnType<typeof setTimeout>;

  return (...args: Parameters<T>) => {
    clearTimeout(timer);

    timer = setTimeout(() => {
      callback(...args);
    }, delay);
  };
}
```

```ts
const search = debounce(
  fetchSearchResults,
  300
);
```

Typing a full search query produces one request after the user pauses instead of one request per character.

### Throttle Frequent Events

```ts
function throttle<T extends (...args: any[]) => void>(
  callback: T,
  limit: number
) {
  let waiting = false;

  return (...args: Parameters<T>) => {
    if (waiting) return;

    callback(...args);
    waiting = true;

    setTimeout(() => {
      waiting = false;
    }, limit);
  };
}
```

Throttle is useful for:

- Scroll
- Resize
- Mouse movement
- Pointer events

---

## 37 — JavaScript's Weird Parts Are Historical

Many JavaScript behaviors are strange because the language was designed quickly and then had to preserve compatibility forever.

Examples include:

```js
typeof null;
// "object"
```

```js
0.1 + 0.2 === 0.3;
// false
```

```js
[] == false;
// true
```

```js
NaN === NaN;
// false
```

```js
0 === -0;
// true
```

```js
Object.is(0, -0);
// false
```

TypeScript does not remove JavaScript's runtime behavior.

It helps you describe and restrict values before execution, but the compiled program still follows JavaScript semantics.

This is why understanding JavaScript remains necessary even when writing TypeScript.

---

## Quick Summary

```txt
TypeScript:
  JavaScript plus a static type system.

Core principle:
  Most TypeScript types disappear after compilation.
  They protect development time, not runtime data automatically.

Inference:
  let name = "Alex"
  → TypeScript understands name is a string.

Annotation:
  let name: string = "Alex"
  → you explicitly declare the type.

any:
  disables type checking for a value.
  Use carefully.

unknown:
  accepts anything, but requires narrowing before use.

never:
  represents an impossible value or a function that never returns.

Arrays:
  number[] → any number of numbers.

Tuples:
  [number, string] → fixed length and fixed positions.

Unions:
  string | number → one of these types.

Literal unions:
  "pending" | "success" | "error"
  → only these exact values are valid.

Enums:
  generate runtime JavaScript.
  They are different from erased type aliases.

Interfaces:
  describe object shapes.

Type aliases:
  create reusable names for types, unions, intersections, and objects.

Generics:
  reusable type parameters that preserve type information.

React:
  useState<Product[]>([])
  → explicitly types an empty state array.

import type:
  separates compile-time imports from runtime imports.

Non-null assertion:
  value!
  → silences the compiler but does not add runtime safety.

Optional chaining:
  user?.address?.city
  → stops when a value is null or undefined.

Nullish coalescing:
  value ?? fallback
  → falls back only for null or undefined.

Spread:
  expands arrays or objects.

Rest:
  collects remaining values.

Map:
  keys can be any value.

Set:
  stores unique values.

Proxy:
  intercepts object operations.

Generator:
  produces lazy values through yield.

Getter:
  property access that runs code.

Private field:
  #value
  → runtime-enforced class encapsulation.
```

---

<div align="center">

## The Type System Is Not the Program

TypeScript will tell you that your code is structurally valid.

It will not tell you that your API response is lying.

It will not stop JavaScript from behaving strangely.

It will not automatically validate data from the outside world.

The developers who truly understand TypeScript know the difference between:

- A compile-time guarantee
- A runtime fact
- A type assertion
- A validation check
- A type annotation
- A real transformation
- A safe value
- A value that merely looks safe

**TypeScript is not about writing more types.**

**It is about making incorrect states difficult to represent while keeping JavaScript's flexibility.**

**Go be one of those.**

</div>