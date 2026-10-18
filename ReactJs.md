<div align="center">

# React Is Not What You Think

### *The broken, beautiful, and deeply misunderstood parts of React nobody warns you about*

**Most developers think React is just JSX, props, and `useState`. This is for the ones who want to understand what actually happens behind the UI.**

</div>

## What's Inside

- **React's Origin** — Why React was created and what problem it solved
- **JSX** — HTML-like syntax that is actually JavaScript
- **Components** — Functions that describe user interfaces
- **Props** — One-way data flow from parent to child
- **State** — Values that survive renders and trigger updates
- **The Virtual DOM** — The in-memory UI description React compares
- **Reconciliation** — How React calculates and commits updates
- **State Batching** — Why multiple updates may produce one render
- **Functional Updates** — The correct way to update from previous state
- **Controlled Forms** — Making React the source of truth
- **Refs** — Accessing the DOM without triggering renders
- **Effects** — Synchronizing React with the outside world
- **Context** — Sharing data without prop drilling
- **React Router** — Navigation without full-page reloads
- **Memoization** — `React.memo`, `useMemo`, and `useCallback`
- **Code Splitting** — Loading route code only when needed
- **Real Patterns** — Authentication, carts, forms, API calls, and loaders

---

## 01 — React Is a UI Library, Not a Magic Framework

React is a **JavaScript library for building user interfaces**.

It was created at Facebook by **Jordan Walke**. The early internal prototype was called **FaxJS**, and React was open-sourced in 2013.

The original problem was not that HTML was difficult.

The problem was that large applications became difficult to keep synchronized.

```js
// Data changes
user.name = "Alex";

// Somewhere else, the UI still displays:
<h1>Old Name</h1>
```

In a traditional application, developers manually find the affected DOM nodes and update them.

React introduced a different mental model:

```txt
State changes
    ↓
React creates a new UI description
    ↓
React compares it with the previous description
    ↓
React updates only what changed
```

You do not normally tell React:

```js
document.querySelector("h1").textContent = user.name;
```

You describe what the UI should look like:

```jsx
<h1>{user.name}</h1>
```

React takes responsibility for synchronizing that description with the real DOM.

> React is not primarily a DOM manipulation library.  
> It is a system for describing UI as a function of state.

```txt
UI = f(state)
```

The same state should produce the same UI.

---

## 02 — JSX Is JavaScript Wearing HTML

JSX looks like HTML, but it is not HTML.

```jsx
const element = <h1>Hello World</h1>;
```

The browser cannot execute this directly. A compiler such as Babel, esbuild, or SWC transforms it into JavaScript.

Conceptually, this:

```jsx
const element = <h1>Hello World</h1>;
```

Becomes something similar to:

```js
const element = React.createElement(
  "h1",
  null,
  "Hello World"
);
```

`React.createElement()` receives:

```js
React.createElement(type, props, ...children)
```

| Argument | Meaning |
|---|---|
| `"h1"` | The element type |
| `null` | The props object |
| `"Hello World"` | The child content |

The result is not a real DOM node.

It is a plain JavaScript object describing what React should eventually render.

```js
{
  type: "h1",
  props: {
    children: "Hello World"
  }
}
```

React later uses this object to construct and update the real DOM.

### JSX Expressions

JavaScript expressions go inside curly braces:

```jsx
const name = "Alex";
const age = 22;

return (
  <div>
    <h1>Hello, {name}</h1>
    <p>You are {age} years old.</p>
  </div>
);
```

Statements do not go directly inside JSX:

```jsx
// Invalid JSX
{
  if (isLoggedIn) {
    return <Dashboard />;
  }
}
```

Use an expression instead:

```jsx
{isLoggedIn ? <Dashboard /> : <Login />}
```

Or:

```jsx
{isLoggedIn && <Dashboard />}
```

### JSX Must Have One Root

This is invalid:

```jsx
return (
  <h1>Hello</h1>
  <p>World</p>
);
```

Wrap the elements in a parent:

```jsx
return (
  <div>
    <h1>Hello</h1>
    <p>World</p>
  </div>
);
```

Or use a Fragment:

```jsx
return (
  <>
    <h1>Hello</h1>
    <p>World</p>
  </>
);
```

Fragments add grouping without adding an unnecessary DOM element.

---

## 03 — Components Are Functions With a Contract

A React component is usually a JavaScript function that returns JSX.

```jsx
const Greeting = () => {
  return <h1>Hello World</h1>;
};

export default Greeting;
```

It is used like an HTML element:

```jsx
<Greeting />
```

The difference between a normal function and a component is not the function syntax.

The difference is how React uses it.

| | Normal Function | React Component |
|---|---|---|
| Invocation | `calculate()` | `<Calculate />` |
| Return value | Any JavaScript value | JSX or `null` |
| Naming | Any valid name | Must begin with uppercase |
| Purpose | General logic | Describes UI |

### Component Naming

This is treated as a custom component:

```jsx
<About />
```

This is treated as a native HTML tag:

```jsx
<about />
```

React uses capitalization to distinguish custom components from built-in elements.

```jsx
const Button = () => <button>Click</button>;
```

Correct:

```jsx
<Button />
```

Incorrect:

```jsx
<button />
```

The second one renders a native HTML button rather than your custom component.

### Components Should Be Pure

A component should ideally behave like a pure function:

```jsx
const Greeting = ({ name }) => {
  return <h1>Hello {name}</h1>;
};
```

Given the same props and state, it should return the same UI.

Avoid performing side effects directly inside render:

```jsx
const About = ({ greet }) => {
  greet(); // Side effect during render
  return <h1>About</h1>;
};
```

Render should describe the UI.

Side effects belong in:

- Event handlers
- `useEffect`
- Other controlled synchronization mechanisms

---

## 04 — Props Only Flow Downward

Props are how a parent sends data to a child.

```jsx
const App = () => {
  return <User name="Alex" age={22} />;
};
```

The child receives those values:

```jsx
const User = (props) => {
  return (
    <div>
      <h1>{props.name}</h1>
      <p>{props.age}</p>
    </div>
  );
};
```

A cleaner version uses destructuring:

```jsx
const User = ({ name, age }) => {
  return (
    <div>
      <h1>{name}</h1>
      <p>{age}</p>
    </div>
  );
};
```

Props are read-only.

```jsx
const User = ({ name }) => {
  // Do not do this
  name = "Changed";
};
```

A child should not modify its props.

### Props Are One-Way

React data normally flows like this:

```txt
Parent
  ↓ props
Child
```

It does not naturally flow sideways:

```txt
Sibling A  ✕  Sibling B
```

And a child cannot directly change a parent's local variable.

However, a parent can pass a function to the child:

```jsx
const App = () => {
  const handleMessage = (message) => {
    console.log(message);
  };

  return <Child onMessage={handleMessage} />;
};
```

The child calls that function:

```jsx
const Child = ({ onMessage }) => {
  return (
    <button onClick={() => onMessage("Hello from Child")}>
      Send Message
    </button>
  );
};
```

The child is not sending data upward through props.

It is calling a function owned by the parent.

### The `children` Prop

Anything placed between a component's opening and closing tags becomes `children`.

```jsx
<Card>
  <h1>Title</h1>
  <p>Description</p>
</Card>
```

The component receives it:

```jsx
const Card = ({ children }) => {
  return (
    <div className="card">
      {children}
    </div>
  );
};
```

This makes wrapper components possible:

```jsx
<ContextProvider>
  <App />
</ContextProvider>
```

The provider does not need to know the details of the application it wraps.

---

## 05 — State Is a Render Trigger, Not Just a Variable

`useState` allows a component to store a value that survives re-renders.

```jsx
import { useState } from "react";

const Counter = () => {
  const [count, setCount] = useState(0);

  return (
    <div>
      <h1>{count}</h1>

      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
};
```

`useState()` returns an array with two values:

```js
const [state, setState] = useState(initialValue);
```

| Part | Meaning |
|---|---|
| `state` | Current state value |
| `setState` | Function used to update the value |
| `initialValue` | Value used during the first render |

Calling the setter has two important effects:

1. It schedules a state update.
2. It tells React to render the component again.

```js
setCount(10);
```

React receives the new value and renders the component with `count = 10`.

### Same-Value Bailout

React compares the new state with the old state using `Object.is()`.

```jsx
const [flag, setFlag] = useState(true);

<button onClick={() => setFlag(false)}>
  Change
</button>
```

The first click changes:

```txt
true → false
```

React re-renders.

Every later click tries to set:

```txt
false → false
```

React can bail out because the value did not change.

```js
Object.is(false, false); // true
```

### Hooks Must Follow Rules

Hooks must be called:

- At the top level of a component
- In the same order on every render
- Inside functional components or custom hooks

Do not call hooks inside conditions:

```jsx
// Wrong
if (isLoggedIn) {
  const [user, setUser] = useState(null);
}
```

Do not call hooks inside loops:

```jsx
// Wrong
items.forEach(() => {
  useState(false);
});
```

Correct:

```jsx
const Component = () => {
  const [user, setUser] = useState(null);

  if (!user) {
    return <Login />;
  }

  return <Dashboard />;
};
```

The hook exists on every render. Only the returned JSX changes.

---

## 06 — Primitive State and Reference State Behave Differently

JavaScript values are broadly divided into primitives and reference values.

```jsx
const [count, setCount] = useState(0);
const [user, setUser] = useState({
  name: "John",
  age: 30
});
```

`count` is a primitive.

`user` is an object reference.

### Primitive State

Primitive values include:

- Numbers
- Strings
- Booleans
- `null`
- `undefined`
- BigInts
- Symbols

Updating a primitive creates a different value:

```js
setCount(count + 1);
```

React can easily detect the difference.

### Reference State

Objects and arrays are reference values.

```js
const user = {
  name: "John"
};
```

The variable does not contain the entire object directly. It points to the object's location in memory.

This is a mutation:

```js
user.name = "Doe";
```

The object changed, but React was not notified.

This is also problematic:

```js
user.name = "Doe";
setUser(user);
```

The object still has the same reference.

```txt
Old reference: abc123
New reference: abc123
```

React may consider the state unchanged.

### Create a New Reference

The correct pattern is to create a new object:

```jsx
setUser({
  ...user,
  name: "Doe"
});
```

Now the references differ:

```txt
Old reference: abc123
New reference: xyz789
```

React detects the change and re-renders.

For arrays:

```jsx
setItems([...items, newItem]);
```

For removing an item:

```jsx
setItems(items.filter((item) => item.id !== id));
```

For updating an item:

```jsx
setItems(
  items.map((item) =>
    item.id === id
      ? { ...item, completed: true }
      : item
  )
);
```

> Never mutate objects or arrays stored in React state.  
> Create a new object or array reference instead.

---

## 07 — State Updates Are Batched

Consider this code:

```jsx
const handleClick = () => {
  setCount(count + 1);
  setCount(count + 1);
  setCount(count + 1);
};
```

A beginner may expect the count to increase by three.

If `count` is `0`, all three expressions are calculated using the same render snapshot:

```js
setCount(1);
setCount(1);
setCount(1);
```

The final value becomes `1`.

React also batches these updates into one render.

```txt
Three state updates
      ↓
One queued update cycle
      ↓
One render
      ↓
One commit
```

### Functional Updates

When the next state depends on the previous state, use the functional updater form:

```jsx
const handleClick = () => {
  setCount((previousCount) => previousCount + 1);
  setCount((previousCount) => previousCount + 1);
  setCount((previousCount) => previousCount + 1);
};
```

Now React processes the update queue in order:

```txt
Initial state: 0

0 + 1 = 1
1 + 1 = 2
2 + 1 = 3

Final state: 3
```

The component still renders once, but the final result is correct.

### Direct Value vs Functional Update

| | Direct Value | Functional Update |
|---|---|---|
| Example | `setCount(count + 1)` | `setCount(prev => prev + 1)` |
| Uses | Current render snapshot | Latest queued state |
| Good for | Replacing state with a known value | Updates based on previous state |
| Example | `setName("Alex")` | Incrementing, toggling, appending |

Use this:

```jsx
setName("Alex");
```

Use this:

```jsx
setCount((prev) => prev + 1);
setOpen((prev) => !prev);
setItems((prev) => [...prev, item]);
```

Since React 18, batching also works across many asynchronous boundaries such as:

- React event handlers
- Promises
- Timeouts
- Native event handlers

---

## 08 — The Virtual DOM Is Not the Real DOM

The real DOM is the browser's actual document tree.

```html
<body>
  <h1>Hello</h1>
  <p>World</p>
</body>
```

The browser stores and renders this structure.

React creates a lightweight JavaScript representation of the UI.

```txt
Real DOM
- Browser-owned
- Expensive to update directly
- Connected to layout and painting

Virtual DOM
- JavaScript objects
- Cheap to create and compare
- Used by React to calculate changes
```

When state changes:

```txt
1. React creates a new element tree.
2. React compares it with the previous tree.
3. React identifies the differences.
4. React commits those differences to the real DOM.
5. The browser paints the updated result.
```

This process is called **reconciliation**.

### Render Phase and Commit Phase

React's update process has two major phases.

#### Render Phase

During the render phase, React:

1. Calls components.
2. Creates a new UI tree.
3. Compares the new tree with the previous tree.
4. Calculates what needs to change.

This phase can be paused, restarted, or abandoned by modern React features.

#### Commit Phase

During the commit phase, React:

1. Applies the calculated changes to the real DOM.
2. Runs the required DOM mutations.
3. Allows the browser to paint the result.

The commit phase must be applied consistently because it affects the visible interface.

```txt
State or props change
        ↓
Render phase
        ↓
New UI tree
        ↓
Diffing
        ↓
Commit phase
        ↓
Real DOM update
        ↓
Browser paint
```

### Stack Reconciler vs Fiber

React originally used a synchronous stack reconciler.

React 16 introduced Fiber.

| | Stack Reconciler | Fiber Reconciler |
|---|---|---|
| Execution | Synchronous | Incremental |
| Can pause work? | No | Yes |
| Can prioritize updates? | Limited | Yes |
| Enables concurrency? | No | Yes |
| Introduced | Early React | React 16 |

Fiber breaks rendering work into smaller units so React can decide which work matters most.

For example:

```txt
User typing → high priority
Large hidden list → lower priority
```

This allows React to remain responsive while processing large updates.

---

## 09 — Lists Need Stable Keys

Rendering an array:

```jsx
const products = [
  { id: 1, name: "Keyboard" },
  { id: 2, name: "Mouse" }
];

return (
  <ul>
    {products.map((product) => (
      <li key={product.id}>
        {product.name}
      </li>
    ))}
  </ul>
);
```

The `key` helps React identify which item is which between renders.

```txt
Previous:
[A, B, C]

New:
[A, C]

React uses keys to understand:
- A stayed
- B was removed
- C moved
```

Use a stable unique identifier:

```jsx
key={product.id}
```

Avoid using array indexes when items can be reordered, inserted, or removed:

```jsx
// Risky when the list changes
key={index}
```

An index key can cause React to associate the wrong state with the wrong item.

Use indexes only when:

- The list never changes order
- Items are never inserted or deleted
- Items do not contain independent state

---

## 10 — Controlled Forms Make React the Source of Truth

A form input can be controlled by React state.

```jsx
import { useState } from "react";

const Form = () => {
  const [name, setName] = useState("");

  return (
    <input
      value={name}
      onChange={(event) => setName(event.target.value)}
    />
  );
};
```

The data flow is:

```txt
User types
    ↓
onChange runs
    ↓
State updates
    ↓
Component re-renders
    ↓
value prop updates input
```

React controls what appears inside the input.

### One State Per Field

```jsx
const [name, setName] = useState("");
const [email, setEmail] = useState("");
const [password, setPassword] = useState("");
```

This works, but it becomes repetitive as the form grows.

### One Object for the Form

```jsx
const [formData, setFormData] = useState({
  name: "",
  email: "",
  password: ""
});
```

Use the spread operator when updating one field:

```jsx
setFormData({
  ...formData,
  name: event.target.value
});
```

Without the spread, the other fields disappear:

```jsx
// Wrong
setFormData({
  name: event.target.value
});
```

### A Reusable `handleChange`

Use the input's `name` attribute as the object key:

```jsx
const handleChange = (event) => {
  const { name, value } = event.target;

  setFormData((previous) => ({
    ...previous,
    [name]: value
  }));
};
```

```jsx
<input
  name="name"
  value={formData.name}
  onChange={handleChange}
/>

<input
  name="email"
  value={formData.email}
  onChange={handleChange}
/>

<input
  name="password"
  value={formData.password}
  onChange={handleChange}
/>
```

The computed property:

```js
[name]: value
```

means JavaScript uses the value inside `name` as the actual object key.

### Form Submission

```jsx
const handleSubmit = (event) => {
  event.preventDefault();

  console.log(formData);

  setFormData({
    name: "",
    email: "",
    password: ""
  });
};
```

The `preventDefault()` call prevents the browser from reloading the page.

---

## 11 — React Hook Form Uses Uncontrolled Inputs

Controlled inputs update React state on every keystroke.

For large forms, this can cause many re-renders.

React Hook Form uses an uncontrolled approach based on DOM refs.

```bash
npm install react-hook-form
```

```jsx
import { useForm } from "react-hook-form";

const Form = () => {
  const {
    register,
    handleSubmit,
    reset
  } = useForm();

  const onSubmit = (data) => {
    console.log(data);
    reset();
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register("name")} />
      <input {...register("email")} />
      <button type="submit">Submit</button>
    </form>
  );
};
```

### What `register()` Returns

Conceptually:

```js
register("name");
```

returns properties similar to:

```js
{
  name,
  ref,
  onChange,
  onBlur
}
```

The spread syntax attaches all of them:

```jsx
<input {...register("name")} />
```

### Validation Rules

```jsx
<input
  {...register("email", {
    required: "Email is required",
    pattern: {
      value: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
      message: "Enter a valid email"
    }
  })}
/>
```

Read validation errors:

```jsx
const {
  register,
  handleSubmit,
  formState: { errors }
} = useForm();
```

```jsx
{errors.email && (
  <p>{errors.email.message}</p>
)}
```

### Why React Hook Form Performs Well

| Controlled Form | React Hook Form |
|---|---|
| State changes every keystroke | Uses refs internally |
| Component often re-renders every keystroke | Fewer unnecessary renders |
| Manual validation | Built-in validation rules |
| Manual reset logic | `reset()` helper |
| More direct React model | More optimized for large forms |

---

## 12 — `useRef` Is a Persistent Mutable Box

`useRef` returns an object with a `.current` property.

```jsx
const inputRef = useRef(null);
```

The object remains the same across renders.

Changing `.current` does not trigger a re-render.

```jsx
inputRef.current = "new value";
```

### Accessing a DOM Element

```jsx
import { useRef } from "react";

const Input = () => {
  const inputRef = useRef(null);

  const handleClick = () => {
    console.log(inputRef.current.value);
  };

  return (
    <div>
      <input
        ref={inputRef}
        placeholder="Type something"
      />

      <button onClick={handleClick}>
        Read Value
      </button>
    </div>
  );
};
```

After mounting:

```txt
inputRef
  ↓
inputRef.current
  ↓
real DOM input element
  ↓
inputRef.current.value
  ↓
text typed by the user
```

### `useState` vs `useRef`

| | `useState` | `useRef` |
|---|---|---|
| Value persists across renders | Yes | Yes |
| Changing value causes render | Yes | No |
| Has a setter | Yes | No |
| Common use | Data shown in UI | DOM nodes and mutable values |

Use `useState` when the value affects what is rendered.

Use `useRef` when the value needs to persist but should not cause a visual update.

### Callback Refs

The `ref` attribute can receive a function:

```jsx
<input
  ref={(node) => {
    console.log(node);
  }}
/>
```

React calls the function with the DOM node when it mounts.

A callback ref can be used to store multiple DOM nodes:

```jsx
const formRef = useRef({});

<input
  ref={(node) => {
    formRef.current.name = node;
  }}
/>

<input
  ref={(node) => {
    formRef.current.email = node;
  }}
/>
```

Later:

```js
formRef.current.name.value;
formRef.current.email.value;
```

A callback ref is useful when a component needs to run setup logic as soon as a DOM node becomes available.

---

## 13 — `useEffect` Is for Synchronizing With the Outside World

An effect is code that interacts with something outside the normal render calculation.

Examples include:

- Fetching data
- Starting timers
- Adding event listeners
- Connecting to subscriptions
- Updating browser APIs
- Integrating third-party libraries

```jsx
useEffect(() => {
  // Setup code
}, []);
```

An effect can return a cleanup function:

```jsx
useEffect(() => {
  const interval = setInterval(() => {
    console.log("running");
  }, 1000);

  return () => {
    clearInterval(interval);
  };
}, []);
```

### Dependency Array Behavior

```jsx
useEffect(() => {
  // Runs after every render
});
```

```jsx
useEffect(() => {
  // Runs after the initial mount
}, []);
```

```jsx
useEffect(() => {
  // Runs on mount and whenever count changes
}, [count]);
```

| Dependency Array | Behavior |
|---|---|
| No array | Every render |
| `[]` | Initial mount |
| `[value]` | Mount and whenever `value` changes |

### Cleanup Runs Before Re-Run

```jsx
useEffect(() => {
  console.log("setup");

  return () => {
    console.log("cleanup");
  };
}, [value]);
```

When `value` changes:

```txt
Previous cleanup
      ↓
New setup
```

When the component unmounts:

```txt
Final cleanup
```

Cleanup should undo what setup did:

```txt
setInterval     → clearInterval
addEventListener → removeEventListener
subscribe       → unsubscribe
connect         → disconnect
```

### Fetching Data

```jsx
useEffect(() => {
  const fetchProducts = async () => {
    try {
      const response = await fetch("/api/products");
      const data = await response.json();
      setProducts(data);
    } catch (error) {
      console.error(error);
    }
  };

  fetchProducts();
}, []);
```

Do not make the effect callback itself `async`:

```jsx
// Avoid this pattern
useEffect(async () => {
  // ...
}, []);
```

Instead, define an async function inside the effect.

### State Updates Are Not Immediately Visible

```jsx
useEffect(() => {
  fetchData();
  console.log(apiData);
}, []);
```

If `fetchData()` later calls:

```js
setApiData(response.data);
```

the `console.log(apiData)` immediately after the call still sees the old value.

State setters schedule updates. They do not synchronously mutate the variable from the current render.

To observe updated data:

```jsx
useEffect(() => {
  console.log(apiData);
}, [apiData]);
```

---

## 14 — `useEffect` Cleanup Prevents Leaks

This component starts an interval:

```jsx
const About = () => {
  useEffect(() => {
    const interval = setInterval(() => {
      console.log("About is running");
    }, 1000);

    return () => {
      clearInterval(interval);
    };
  }, []);

  return <h1>About</h1>;
};
```

The lifecycle is:

```txt
About mounts
    ↓
Effect starts interval
    ↓
About remains mounted
    ↓
About unmounts
    ↓
Cleanup clears interval
```

Without cleanup, the interval can continue running after the component disappears.

Repeatedly mounting and unmounting the component could create multiple intervals:

```txt
Mount 1 → interval 1
Unmount → interval 1 still running

Mount 2 → interval 2
Unmount → interval 2 still running

Mount 3 → interval 3
```

This is a memory leak and unnecessary background work.

The same issue applies to event listeners:

```jsx
useEffect(() => {
  const handleScroll = () => {
    console.log(window.scrollY);
  };

  window.addEventListener("scroll", handleScroll);

  return () => {
    window.removeEventListener("scroll", handleScroll);
  };
}, []);
```

---

## 15 — Debouncing and Throttling Are Different

Debouncing and throttling both control how often code runs, but they solve different problems.

| | Debounce | Throttle |
|---|---|---|
| Mental model | Wait for quiet | At most once per interval |
| Common use | Search input | Scroll listener |
| First event | Usually delayed | Usually immediate |
| Repeated events | Resets timer | Ignored during cooldown |

### Debounced Search

```jsx
useEffect(() => {
  if (!searchData) {
    setProducts(allProducts);
    return;
  }

  const timeout = setTimeout(() => {
    const result = allProducts.filter((product) =>
      product.title
        .toLowerCase()
        .includes(searchData.toLowerCase())
    );

    setProducts(result);
  }, 700);

  return () => {
    clearTimeout(timeout);
  };
}, [searchData, allProducts]);
```

Typing:

```txt
s → schedule timer A
h → cancel A, schedule timer B
i → cancel B, schedule timer C
r → cancel C, schedule timer D
t → cancel D, schedule timer E
700ms silence → timer E runs
```

The search runs once after the user stops typing.

### Throttled Scroll

```jsx
useEffect(() => {
  let locked = false;

  const handleScroll = () => {
    if (locked) return;

    locked = true;
    setScrollY(window.scrollY);

    setTimeout(() => {
      locked = false;
    }, 200);
  };

  window.addEventListener("scroll", handleScroll);

  return () => {
    window.removeEventListener("scroll", handleScroll);
  };
}, []);
```

The first event runs immediately.

Events during the cooldown are ignored.

---

## 16 — Context API Solves Prop Drilling

Suppose data lives in `App`, but a deeply nested component needs it:

```txt
App
  └── Layer1
        └── Layer2
              └── Layer3
                    └── DeepComponent
```

Without Context, every layer may need to receive and forward the data:

```jsx
<App user={user} />
<Layer1 user={user} />
<Layer2 user={user} />
<Layer3 user={user} />
<DeepComponent user={user} />
```

This is prop drilling.

### Creating Context

```jsx
import { createContext } from "react";

export const AppContext = createContext();
```

The context object itself does not hold the data.

It provides a channel through which data can be shared.

### Creating a Provider

```jsx
import { useState } from "react";
import { AppContext } from "./AppContext";

export const AppProvider = ({ children }) => {
  const [user, setUser] = useState(null);

  return (
    <AppContext.Provider value={{ user, setUser }}>
      {children}
    </AppContext.Provider>
  );
};
```

The `value` prop can contain any JavaScript value. Usually, multiple values are placed inside one object.

### Wrapping the Application

```jsx
import { createRoot } from "react-dom/client";
import App from "./App";
import { AppProvider } from "./context/AppContext";

createRoot(document.getElementById("root")).render(
  <AppProvider>
    <App />
  </AppProvider>
);
```

Everything inside the provider can read the context.

### Reading Context

```jsx
import { useContext } from "react";
import { AppContext } from "../context/AppContext";

const Profile = () => {
  const { user } = useContext(AppContext);

  return <h1>{user?.name}</h1>;
};
```

The component does not need the data passed through every intermediate component.

### Context Re-Renders

When the provider's value changes, components using that context re-render.

Context is useful for data such as:

- Theme
- Authenticated user
- Locale
- Cart
- Shared configuration

Context is not automatically a complete global state management solution.

For large, frequently changing state, dedicated tools such as Redux or Zustand may provide better structure and tooling.

---

## 17 — Derived State Should Usually Stay Derived

Do not store values that can be calculated from existing state.

```jsx
const cartItem = cart.find((item) => item.id === id);
```

This is derived data.

Do not create unnecessary state:

```jsx
const [cartItem, setCartItem] = useState(null);
```

If `cartItem` can be calculated from `cart`, storing it separately creates synchronization problems.

Another example:

```jsx
const subtotal = cart.reduce(
  (sum, item) => sum + item.price * item.qty,
  0
);
```

Whenever `cart` changes, the subtotal is recalculated.

```txt
cart changes
    ↓
component renders
    ↓
subtotal is calculated from current cart
```

Storing `subtotal` separately could produce stale values.

### When to Memoize Derived Data

For cheap calculations:

```jsx
const subtotal = cart.reduce(...);
```

is usually best.

For expensive calculations:

```jsx
const result = useMemo(() => {
  return expensiveCalculation(data);
}, [data]);
```

The question is not:

> Can this value be memoized?

The question is:

> Is recalculating this value expensive enough to justify memoization?

---

## 18 — React Router Connects URLs to UI

React Router allows a React application to change views without a full browser reload.

In declarative mode:

```jsx
import {
  BrowserRouter,
  Routes,
  Route
} from "react-router";

const App = () => {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </BrowserRouter>
  );
};
```

`BrowserRouter` uses the browser's History API.

Everything using router features must be inside it.

```txt
BrowserRouter
  └── App
        ├── NavLink
        ├── Routes
        └── useNavigate
```

### Navigation With `NavLink`

```jsx
import { NavLink } from "react-router";

const Navbar = () => {
  return (
    <nav>
      <NavLink to="/">Home</NavLink>
      <NavLink to="/about">About</NavLink>
    </nav>
  );
};
```

`NavLink` can receive a function for active styling:

```jsx
<NavLink
  to="/about"
  className={({ isActive }) =>
    isActive ? "active" : "inactive"
  }
>
  About
</NavLink>
```

### Nested Routes and `Outlet`

```jsx
<Routes>
  <Route path="/" element={<MainLayout />}>
    <Route index element={<Home />} />
    <Route path="about" element={<About />} />
    <Route path="contact" element={<Contact />} />
  </Route>
</Routes>
```

The layout renders the matching child through `<Outlet />`:

```jsx
import { Outlet } from "react-router";

const MainLayout = () => {
  return (
    <div>
      <Navbar />
      <main>
        <Outlet />
      </main>
    </div>
  );
};
```

The navbar stays mounted while the content inside the outlet changes.

### Dynamic Routes

```jsx
<Route
  path="/products/:id"
  element={<ProductDetail />}
/>
```

Read the parameter:

```jsx
import { useParams } from "react-router";

const ProductDetail = () => {
  const { id } = useParams();

  return <h1>Product ID: {id}</h1>;
};
```

URL parameters are strings:

```js
id === "42";
```

Convert them when numeric behavior is required:

```js
const numericId = Number(id);
```

### Programmatic Navigation

```jsx
import { useNavigate } from "react-router";

const ProductCard = ({ product }) => {
  const navigate = useNavigate();

  return (
    <button
      onClick={() => navigate(`/products/${product.id}`)}
    >
      View Product
    </button>
  );
};
```

### Protected Routes

A protected route can render children only when the user is authenticated:

```jsx
import { Navigate, Outlet } from "react-router";

const ProtectedRoute = () => {
  const isLoggedIn = Boolean(localStorage.getItem("user"));

  if (!isLoggedIn) {
    return <Navigate to="/" />;
  }

  return <Outlet />;
};
```

Route configuration:

```jsx
<Route element={<ProtectedRoute />}>
  <Route path="/dashboard" element={<Dashboard />} />
</Route>
```

### Data Router

React Router also supports a data-router style:

```jsx
import {
  createBrowserRouter,
  RouterProvider
} from "react-router";

const router = createBrowserRouter([
  {
    path: "/",
    element: <MainLayout />,
    children: [
      {
        index: true,
        element: <Home />
      },
      {
        path: "about",
        element: <About />
      }
    ]
  }
]);

const AppRoutes = () => {
  return <RouterProvider router={router} />;
};
```

Data routers support features such as:

- Loaders
- Actions
- Route-level lazy loading
- Fetchers
- Error elements

---

## 19 — Loaders Fetch Data Before a Route Renders

A route loader can fetch data before the route component renders.

```jsx
const router = createBrowserRouter([
  {
    path: "/users",
    element: <Users />,
    loader: async () => {
      const response = await fetch("/api/users");
      return response.json();
    }
  }
]);
```

Read loader data:

```jsx
import { useLoaderData } from "react-router";

const Users = () => {
  const users = useLoaderData();

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
};
```

For independent requests, use `Promise.all`:

```jsx
loader: async () => {
  const [users, products] = await Promise.all([
    getUsers(),
    getProducts()
  ]);

  return { users, products };
}
```

This runs both requests concurrently rather than waiting for one before starting the other.

### Loader and Component Loading

With a normal component-level `React.lazy()`:

```jsx
const About = lazy(() => import("./About"));
```

the lazy module may begin loading only when React first renders the component.

If a route loader must finish first, the data request and component chunk can happen sequentially.

Route-level lazy loading can co-locate the route module and improve this flow.

### Always Handle Route Errors

A data router should define an error element:

```jsx
{
  path: "/users",
  element: <Users />,
  loader: getUsers,
  errorElement: <ErrorPage />
}
```

If a loader throws, the router can render the dedicated error UI.

---

## 20 — Authentication Is Shared State Plus Routing

A simple authentication architecture may contain:

```txt
AuthProvider
  ├── registeredUser
  ├── loggedInUser
  └── setters

AuthLayout
  └── Keeps logged-in users away from login pages

ProtectedRoutes
  └── Keeps logged-out users away from private pages
```

### Auth Context

```jsx
import { createContext, useState } from "react";

export const AuthContext = createContext();

export const AuthProvider = ({ children }) => {
  const [loggedInUser, setLoggedInUser] = useState(
    JSON.parse(localStorage.getItem("loggedInUser")) || null
  );

  return (
    <AuthContext.Provider
      value={{ loggedInUser, setLoggedInUser }}
    >
      {children}
    </AuthContext.Provider>
  );
};
```

### Public Auth Layout

```jsx
const AuthLayout = () => {
  const { loggedInUser } = useContext(AuthContext);

  if (loggedInUser) {
    return <Navigate to="/dashboard" />;
  }

  return <Outlet />;
};
```

### Private Route Layout

```jsx
const ProtectedRoutes = () => {
  const { loggedInUser } = useContext(AuthContext);

  if (!loggedInUser) {
    return <Navigate to="/" />;
  }

  return <Outlet />;
};
```

These layouts guard opposite sides of the application:

| Layout | Allows | Redirects |
|---|---|---|
| `AuthLayout` | Logged-out users | Logged-in users to dashboard |
| `ProtectedRoutes` | Logged-in users | Logged-out users to login |

### Important Security Warning

Storing passwords directly in `localStorage` is not real authentication.

```jsx
localStorage.setItem(
  "registeredUser",
  JSON.stringify({
    username,
    password
  })
);
```

This may be acceptable for a local learning demo, but production authentication requires:

- A backend
- Secure password hashing
- Sessions or secure tokens
- HTTPS
- Server-side authorization
- Proper cookie configuration

---

## 21 — `React.memo` Prevents Some Child Renders

By default, when a parent re-renders, its children may also render.

```jsx
const Home = () => {
  console.log("Home rendered");
  return <h1>Home</h1>;
};
```

Wrap a component in `React.memo`:

```jsx
export default React.memo(Home);
```

React compares the component's props.

If the props are shallowly equal, React can skip rendering the child.

### What `React.memo` Does Not Do

It does not stop the parent from rendering.

It does not stop the child from rendering when:

- Its props change
- Its own state changes
- A context it reads changes

It only helps with parent renders caused by unrelated changes.

### The Function Reference Problem

This creates a new function on every render:

```jsx
const greet = () => {
  console.log("Hello");
};
```

Even if the code looks identical, the function reference changes.

```txt
Render 1 → greet reference A
Render 2 → greet reference B
Render 3 → greet reference C
```

A memoized child receiving `greet` sees a changed prop every time.

Use `useCallback`:

```jsx
const greet = useCallback(() => {
  console.log("Hello");
}, []);
```

Now the function reference remains stable until a dependency changes.

---

## 22 — `useMemo` Caches a Value

`useMemo` caches the result of a calculation:

```jsx
const calculation = useMemo(() => {
  return expensiveCalculation(data);
}, [data]);
```

The calculation runs:

- On the first render
- Again when `data` changes

It does not run on unrelated renders.

### `useMemo` Does Not Make Code Asynchronous

This code still blocks the main thread when it runs:

```jsx
const result = useMemo(() => {
  let sum = 0;

  for (let i = 0; i < 100000000; i++) {
    sum += i;
  }

  return sum;
}, [data]);
```

`useMemo` controls frequency.

It does not move the work to another thread.

### `useMemo` vs Derived State

For cheap values:

```jsx
const subtotal = cart.reduce(
  (sum, item) => sum + item.price * item.qty,
  0
);
```

For genuinely expensive values:

```jsx
const result = useMemo(
  () => expensiveCalculation(data),
  [data]
);
```

Do not use memoization everywhere automatically.

Memoization itself has complexity and dependency-management costs.

---

## 23 — `useCallback` Caches a Function Reference

`useCallback` returns a stable function reference:

```jsx
const handleClick = useCallback(() => {
  console.log("clicked");
}, []);
```

This is useful when passing a function to a memoized child:

```jsx
const App = () => {
  const handleClick = useCallback(() => {
    console.log("clicked");
  }, []);

  return <Child onClick={handleClick} />;
};
```

```jsx
const Child = React.memo(({ onClick }) => {
  console.log("Child rendered");

  return (
    <button onClick={onClick}>
      Click
    </button>
  );
});
```

Without `useCallback`, the child may see a new function prop on every parent render.

### The Three Memoization Tools

| Tool | Caches | Re-runs when |
|---|---|---|
| `React.memo` | Component rendering | Props change |
| `useMemo` | Calculation result | Dependencies change |
| `useCallback` | Function reference | Dependencies change |

```txt
useMemo       → value
useCallback   → function
React.memo    → component render
```

They solve related but different problems.

---

## 24 — Code Splitting Loads Code on Demand

A normal import is eager:

```jsx
import Home from "./Home";
```

The module is included in the initial dependency graph.

A dynamic import is lazy:

```jsx
const About = lazy(() => import("./About"));
```

The module can be placed into a separate bundle and loaded when needed.

Wrap lazy components in `Suspense`:

```jsx
import { lazy, Suspense } from "react";

const About = lazy(() => import("./About"));

const App = () => {
  return (
    <Suspense fallback={<p>Loading...</p>}>
      <About />
    </Suspense>
  );
};
```

When the chunk is downloading:

```txt
About cannot render yet
        ↓
Suspense shows fallback
        ↓
Chunk arrives
        ↓
About renders
```

A persistent layout outside the boundary can remain visible:

```jsx
<MainLayout>
  <Navbar />

  <Suspense fallback={<PageLoader />}>
    <About />
  </Suspense>
</MainLayout>
```

The navbar stays mounted while the route content loads.

### Code Splitting and Loaders

A route with both a loader and a component-level lazy import can experience sequential work:

```txt
Navigation
    ↓
Loader request
    ↓
Loader resolves
    ↓
Lazy component begins loading
    ↓
Component renders
```

Route-level lazy APIs can coordinate route module loading more effectively.

---

## 25 — API Calls Need Loading, Error, and Empty States

A reliable data-fetching component should represent more than just the successful state.

```jsx
const Products = () => {
  const [products, setProducts] = useState([]);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const getProducts = async () => {
      try {
        setIsLoading(true);
        setError(null);

        const response = await fetch("/api/products");
        const data = await response.json();

        setProducts(data);
      } catch (error) {
        setError(error);
      } finally {
        setIsLoading(false);
      }
    };

    getProducts();
  }, []);

  if (isLoading) {
    return <p>Loading products...</p>;
  }

  if (error) {
    return <p>Something went wrong.</p>;
  }

  if (products.length === 0) {
    return <p>No products found.</p>;
  }

  return (
    <div>
      {products.map((product) => (
        <ProductCard
          key={product.id}
          product={product}
        />
      ))}
    </div>
  );
};
```

The UI has four possible states:

```txt
Loading
    ↓
Error OR Empty OR Loaded
```

### Axios Instances

For repeated API calls, create a configured instance:

```jsx
import axios from "axios";

export const axiosInstance = axios.create({
  baseURL: "https://api.example.com"
});
```

Use it with relative URLs:

```jsx
const response = await axiosInstance.get("/products");
```

### Axios Interceptors

Interceptors can run for every request and response:

```jsx
axiosInstance.interceptors.request.use(
  (config) => {
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

axiosInstance.interceptors.response.use(
  (response) => {
    return response;
  },
  (error) => {
    return Promise.reject(error);
  }
);
```

They are useful for:

- Adding authorization headers
- Logging
- Centralized error handling
- Refreshing expired tokens
- Normalizing responses

---

## 26 — A Cart Is Shared State Plus Derived State

A shopping cart is a good example of Context and immutable updates.

```jsx
const [cart, setCart] = useState([]);
```

Add an item:

```jsx
const addToCart = (product) => {
  setCart((previous) => [
    ...previous,
    {
      ...product,
      qty: 1
    }
  ]);
};
```

Update quantity:

```jsx
const updateQty = (id, delta) => {
  setCart((previous) =>
    previous
      .map((item) =>
        item.id === id
          ? {
              ...item,
              qty: item.qty + delta
            }
          : item
      )
      .filter((item) => item.qty > 0)
  );
};
```

The function:

```jsx
updateQty(id, 1);
```

increments.

```jsx
updateQty(id, -1);
```

decrements.

```jsx
updateQty(id, -item.qty);
```

removes the item completely.

### Derived Cart Item

```jsx
const cartItem = cart.find(
  (item) => item.id === product.id
);
```

This should usually remain derived rather than being duplicated in separate state.

### Derived Subtotal

```jsx
const subtotal = cart.reduce(
  (sum, item) => sum + item.price * item.qty,
  0
);
```

The subtotal automatically reflects the latest cart state.

### Checkout

```jsx
const handleCheckout = () => {
  setOrderPlaced(true);
  setCart([]);
};
```

Because the cart is shared, clearing it updates:

- The cart page
- The navbar badge
- Product quantity controls
- Any other component reading the cart context

---

## 27 — React Is a State Synchronization System

React becomes easier to understand when the pieces are connected.

```txt
Props
  → parent-to-child communication

State
  → persistent values that trigger renders

Virtual DOM
  → JavaScript description of the UI

Reconciliation
  → comparison between old and new descriptions

Commit
  → changes applied to the real DOM

Effects
  → synchronization with external systems

Context
  → shared values across deep component trees

Router
  → URL state connected to component trees
```

Most React bugs are caused by one of these misunderstandings:

- Treating state like a normal mutable variable
- Mutating objects instead of creating new references
- Reading stale values from a render closure
- Forgetting functional updates
- Running side effects during render
- Forgetting effect cleanup
- Storing derived data as independent state
- Passing unstable function references unnecessarily
- Using context for every possible value
- Forgetting that route loaders and lazy components have separate timing

---

## Quick Revision

```txt
React:
  A JavaScript library for describing user interfaces.

JSX:
  HTML-like syntax compiled into JavaScript calls.

Component:
  A function that returns JSX.

Props:
  Read-only data flowing from parent to child.

State:
  Data owned by a component that persists across renders.

setState:
  Schedules a state update and usually triggers a re-render.

Primitive state:
  Compare by value.

Object/array state:
  Compare by reference.
  Create a new reference when updating.

Functional updater:
  setCount(prev => prev + 1)
  Use whenever new state depends on previous state.

Batching:
  Multiple updates can be processed in one render.

Virtual DOM:
  In-memory JavaScript representation of UI.

Reconciliation:
  React's process of calculating what changed.

Commit:
  The phase where React updates the real DOM.

useEffect:
  Synchronizes React with external systems.

Cleanup:
  Undoes timers, listeners, subscriptions, and connections.

useRef:
  Persistent mutable value that does not trigger rendering.

Context:
  Shares data deeply without prop drilling.

React Router:
  Connects URLs to rendered components.

useMemo:
  Caches a calculated value.

useCallback:
  Caches a function reference.

React.memo:
  Skips child renders when props are shallowly equal.

lazy + Suspense:
  Loads component code only when needed.
```

---

<div align="center">

## The Gap Is Where Craft Lives

Most developers learn React by memorizing hooks and copying component patterns.

The developers who truly understand React know:

- What causes a render
- What causes a commit
- Which values belong in state
- Which values should be derived
- When references change
- When closures become stale
- When effects need cleanup
- When context is useful
- When memoization is unnecessary
- When the browser already solved the problem

**React is not about writing more components.**

**It is about building a clear relationship between state, rendering, and the outside world.**

**Go be one of those.**

</div>