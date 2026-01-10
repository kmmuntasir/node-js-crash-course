# Module 3: React.js Fundamentals & Advanced Patterns

## React Core Concepts

### React Fundamentals

**JSX Syntax and Transformation**

JSX is a syntax extension for JavaScript that allows writing HTML-like code in JavaScript files. It's transformed into regular JavaScript function calls by tools like Babel before execution. JSX enables developers to write declarative UI code that closely resembles the final HTML structure, making React code more readable and maintainable.

```javascript
// JSX syntax
const element = <div className="greeting">Hello, World!</div>;

// Transformed to JavaScript
const element = React.createElement('div', { className: 'greeting' }, 'Hello, World!');

// Expressions in JSX
const name = 'John';
const greeting = <div>Hello, {name}!</div>;

// Conditional rendering
const isLoggedIn = true;
const element = <div>{isLoggedIn ? <Dashboard /> : <Login />}</div>;

// Lists and keys
const items = ['Apple', 'Banana', 'Orange'];
const list = <ul>{items.map((item, index) => <li key={index}>{item}</li>)}</ul>;
```

**Components: Functional vs Class Components**

React components can be defined as functional components (simple functions returning JSX) or class components (ES6 classes extending React.Component). Functional components are the modern standard, especially with hooks, while class components were the original way to manage state and lifecycle methods. Functional components are simpler, easier to test, and have better performance characteristics.

```javascript
// Functional component (modern approach)
function Welcome({ name }) {
  return <h1>Hello, {name}!</h1>;
}

// Arrow function component
const Welcome = ({ name }) => <h1>Hello, {name}!</h1>;

// Class component (legacy approach)
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}

// Functional component with hooks
function Counter() {
  const [count, setCount] = React.useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

**Props: Passing Data and Prop Drilling**

Props (properties) are read-only data passed from parent to child components. Props enable component composition and data flow in React applications. Prop drilling occurs when data is passed through multiple intermediate components that don't use the data, creating deep component trees and making updates difficult. Solutions to prop drilling include Context API and state management libraries.

```javascript
// Parent component passing props
function App() {
  const user = { name: 'John', age: 30 };
  return <UserProfile user={user} />;
}

// Child component receiving props
function UserProfile({ user }) {
  return (
    <div>
      <h2>{user.name}</h2>
      <p>Age: {user.age}</p>
    </div>
  );
}

// Prop drilling example (problematic)
function App() {
  const theme = 'dark';
  return (
    <div>
      <Header theme={theme} />
      <Main theme={theme} />
      <Footer theme={theme} />
    </div>
  );
}

// Each component passes theme down without using it
```

**State: useState Hook Basics**

State is data that changes over time and affects component rendering. The `useState` hook is the fundamental way to manage state in functional components. It returns an array with the current state value and a function to update it. State updates trigger re-renders of the component and its children. State should be kept minimal and only include data that affects rendering.

```javascript
import React, { useState } from 'react';

function Counter() {
  // useState returns [currentValue, updateFunction]
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');

  // Simple state update
  const increment = () => setCount(count + 1);

  // State update based on previous value
  const decrement = () => setCount(prev => prev - 1);

  // Object state
  const [user, setUser] = useState({ name: '', age: 0 });

  const updateName = (newName) => {
    setUser(prev => ({ ...prev, name: newName }));
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
    </div>
  );
}
```

**Component Lifecycle (Class Components)**

Class components have lifecycle methods that execute at specific points during a component's existence. Mounting methods (`componentDidMount`, `componentWillMount`) run when component is inserted into DOM. Updating methods (`componentDidUpdate`, `shouldComponentUpdate`) run when props or state change. Unmounting methods (`componentWillUnmount`, `componentDidUnmount`) run when component is removed. Understanding lifecycle is crucial for side effects like data fetching, subscriptions, and cleanup.

```javascript
class UserProfile extends React.Component {
  constructor(props) {
    super(props);
    this.state = { user: null, loading: true };
  }

  // Mounting phase
  componentDidMount() {
    this.fetchUserData();
  }

  // Updating phase
  shouldComponentUpdate(nextProps, nextState) {
    // Return false to prevent re-render
    return nextProps.id !== this.props.id;
  }

  componentDidUpdate(prevProps, prevState) {
    if (prevProps.id !== this.props.id) {
      this.fetchUserData();
    }
  }

  // Unmounting phase
  componentWillUnmount() {
    // Cleanup: cancel requests, remove listeners
    if (this.request) {
      this.request.abort();
    }
  }

  fetchUserData() {
    fetch(`/api/users/${this.props.id}`)
      .then(res => res.json())
      .then(user => this.setState({ user, loading: false }))
      .catch(error => this.setState({ error: error.message }));
  }

  render() {
    const { user, loading, error } = this.state;
    
    if (loading) return <div>Loading...</div>;
    if (error) return <div>Error: {error}</div>;
    
    return (
      <div>
        <h2>{user.name}</h2>
        <p>Email: {user.email}</p>
      </div>
    );
  }
}
```

---

### Rendering & Reconciliation

**Virtual DOM Concept**

The Virtual DOM is a lightweight JavaScript representation of the actual DOM. React maintains a virtual DOM tree in memory and compares it with the actual DOM to determine minimal changes needed. This approach, called reconciliation, reduces expensive DOM operations and improves performance. The virtual DOM enables React to batch updates and apply them efficiently, making applications faster and more responsive.

```
Actual DOM (expensive to manipulate)
    ↓
React Virtual DOM (fast, in-memory)
    ↓
Reconciliation (diff and calculate changes)
    ↓
Minimal DOM updates (only what changed)
```

**Reconciliation Algorithm**

React's reconciliation algorithm determines how to update the DOM efficiently when state or props change. It compares the new virtual DOM tree with the previous one, identifies differences (diffing), and applies only the necessary changes to the actual DOM. This process happens automatically and is optimized to minimize DOM mutations. React Fiber is the current implementation of this algorithm, enabling incremental rendering and better prioritization of updates.

```javascript
// Reconciliation example
function App() {
  const [items, setItems] = React.useState([1, 2, 3]);

  // React creates new virtual DOM
  const newItems = [...items, 4];
  setItems(newItems);

  // Reconciliation process:
  // 1. Compare old virtual DOM with new virtual DOM
  // 2. Identify differences (added 4)
  // 3. Update only the changed parts of actual DOM
  // 4. Re-render affected components
}
```

**Diffing Strategy**

React uses a heuristic diffing algorithm to compare virtual DOM trees efficiently. When comparing lists, React uses keys to identify which items changed, were added, or were removed. For elements, React compares element types and attributes. The algorithm is optimized for common cases and falls back to full re-renders when necessary. Understanding diffing helps write efficient components and avoid unnecessary re-renders.

```javascript
// Diffing with keys (efficient)
function TodoList({ items }) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>
          {item.text}
        </li>
      ))}
    </ul>
  );
}

// Diffing without keys (inefficient, causes issues)
function BadTodoList({ items }) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>  {/* Index keys are problematic */}
          {item.text}
        </li>
      ))}
    </ul>
  );
}
```

**Keys and Why Index Keys Are Problematic**

Keys help React identify which items have changed, been added, or removed in a list. Using array indices as keys is problematic because React can't distinguish items properly when the list order changes, leading to bugs like incorrect focus, state loss, or performance issues. Stable, unique identifiers (like IDs) should be used as keys instead.

```javascript
// Problematic: Using index as key
function BadList({ items }) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>  {/* Index changes when list changes */}
          <input value={item.text} />
        </li>
      ))}
    </ul>
  );
}

// Good: Using unique ID as key
function GoodList({ items }) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>  {/* Stable, unique identifier */}
          <input value={item.text} />
        </li>
      ))}
    </ul>
  );
}

// Issues with index keys:
// - Input focus lost when list reorders
// - State gets mixed up
// - Performance degradation
// - Unintended side effects
```

**React Fiber Architecture Overview**

React Fiber is the reimplementation of React's core algorithm, introduced in React 16. Fiber enables incremental rendering, allowing React to break rendering work into chunks and prioritize updates. This architecture improves responsiveness by returning control to the browser between rendering units, enabling smoother animations and interactions. Fiber also makes it easier to implement features like error boundaries and concurrent rendering.

```
React Fiber Architecture:
┌─────────────────────────────────┐
│     Work Scheduler           │ → Prioritizes and schedules updates
├─────────────────────────────────┤
│     Reconciler (Fiber)       │ → Incremental reconciliation
├─────────────────────────────────┤
│     Renderer                 │ → Updates actual DOM
└─────────────────────────────────┘

Benefits:
- Incremental rendering (return control to browser)
- Priority-based updates (user interactions first)
- Better error handling (error boundaries)
- Concurrent rendering capabilities
```

---

### Hooks Deep Dive

**useState: State Management Patterns**

The `useState` hook is the foundation of state management in functional components. It can hold primitive values, objects, or arrays. State updates can be functional (receiving previous state) or direct (replacing with new value). Understanding state update patterns helps avoid common bugs like stale closures and unnecessary re-renders.

```javascript
function Counter() {
  const [count, setCount] = React.useState(0);

  // Direct update (replaces entire state)
  const reset = () => setCount(0);

  // Functional update (receives previous state)
  const increment = () => setCount(prev => prev + 1);

  // Object state
  const [user, setUser] = React.useState({ name: '', age: 0 });

  const updateName = (name) => {
    setUser(prev => ({ ...prev, name })); // Merge with previous
  };

  // Multiple state variables
  const [name, setName] = React.useState('');
  const [email, setEmail] = React.useState('');
  const [age, setAge] = React.useState(0);

  return <div>{/* ... */}</div>;
}
```

**useEffect: Side Effects and Dependencies**

The `useEffect` hook handles side effects in functional components, replacing lifecycle methods like `componentDidMount`, `componentDidUpdate`, and `componentWillUnmount`. It runs after render commits to screen and can optionally return a cleanup function. The dependency array controls when the effect runs: empty array means run only on mount, omitted means run on every render, and specified dependencies mean run when those dependencies change.

```javascript
function UserProfile({ userId }) {
  const [user, setUser] = React.useState(null);
  const [loading, setLoading] = React.useState(true);

  // Run once on mount (empty dependency array)
  React.useEffect(() => {
    fetchUser(userId).then(data => {
      setUser(data);
      setLoading(false);
    });
  }, []); // Empty array = run only on mount

  // Run when userId changes
  React.useEffect(() => {
    setLoading(true);
    fetchUser(userId).then(data => {
      setUser(data);
      setLoading(false);
    });
  }, [userId]); // Run when userId changes

  // Run on every render (no dependency array)
  React.useEffect(() => {
    document.title = `User: ${user?.name || 'Loading'}`;
  }); // Omitted dependency array

  // With cleanup function
  React.useEffect(() => {
    const timer = setInterval(() => {
      console.log('Tick');
    }, 1000);

    return () => clearInterval(timer); // Cleanup on unmount
  }, []);
}
```

**useEffect Internals & Implementation**

The `useEffect` hook is implemented using React's Fiber architecture, with effects stored in the fiber node's `memoizedState`. Each effect has a tag indicating its type (Passive, Layout), a create function, destroy (cleanup) function, and dependencies array. Effects are scheduled during the render phase and executed during the commit phase. Understanding these internals helps debug performance issues and understand when effects run.

```javascript
// Effect object structure (internal representation)
const effect = {
  tag: 0b001, // Passive effect (useEffect)
  create: () => { /* effect function */ },
  destroy: () => { /* cleanup function */ },
  deps: [dependency1, dependency2], // Dependency array
  next: null // Pointer to next effect in list
};

// Effect execution phases:
// 1. Render phase: Create effect and schedule it
// 2. Commit phase - Before mutation: Layout effects run
// 3. Commit phase - Mutation: DOM updates applied
// 4. Commit phase - After mutation: Passive effects run after paint

// useEffect vs useLayoutEffect
// useEffect: Runs after paint (non-blocking, better for most effects)
// useLayoutEffect: Runs before paint (blocking, use for DOM measurements)
```

**useCallback: Memoizing Functions**

The `useCallback` hook memoizes functions, preventing unnecessary re-creations on every render. It returns a memoized version of the callback that only changes when its dependencies change. This is particularly useful when passing callbacks to child components that are memoized with `React.memo`, preventing child re-renders when the parent re-renders but the callback hasn't actually changed.

```javascript
function Parent() {
  const [count, setCount] = React.useState(0);

  // Memoized callback - only recreated when count changes
  const handleClick = React.useCallback(() => {
    setCount(count + 1);
  }, [count]); // Dependency: recreate only when count changes

  // Without useCallback (recreated on every render)
  const badHandleClick = () => {
    setCount(count + 1);
  };

  return <Child onClick={handleClick} />;
}

// Child component with React.memo
const Child = React.memo(function Child({ onClick }) {
  console.log('Child rendered');
  return <button onClick={onClick}>Click me</button>;
});

// With useCallback: Child renders only when onClick changes
// Without useCallback: Child renders on every Parent render
```

**useMemo: Memoizing Values**

The `useMemo` hook memoizes expensive calculations, preventing re-computation on every render. It takes a function and dependency array, returning the cached result when dependencies haven't changed. This is useful for derived state, filtered lists, sorted arrays, or any computation that's expensive relative to rendering. Overusing `useMemo` can actually hurt performance due to memoization overhead.

```javascript
function ExpensiveComponent({ items, filter }) {
  // Memoized expensive computation
  const filteredItems = React.useMemo(() => {
    console.log('Computing filtered items');
    return items.filter(item => 
      item.name.toLowerCase().includes(filter.toLowerCase())
    );
  }, [items, filter]); // Recompute only when items or filter changes

  // Without useMemo: recomputes on every render
  // const badFilteredItems = items.filter(item => ...);

  return (
    <ul>
      {filteredItems.map(item => <li key={item.id}>{item.name}</li>)}
    </ul>
  );
}

// Memoizing derived state
function UserProfile({ user }) {
  const fullName = React.useMemo(
    () => `${user.firstName} ${user.lastName}`,
    [user.firstName, user.lastName]
  );

  const initials = React.useMemo(
    () => `${user.firstName[0]}${user.lastName[0]}`,
    [user.firstName, user.lastName]
  );

  return <div>{fullName} ({initials})</div>;
}
```

**useRef: DOM Refs and Persistent Values**

The `useRef` hook returns a mutable ref object that persists across renders. It's commonly used to access DOM elements directly or to store values that don't trigger re-renders when updated. Unlike state, updating a ref's `.current` property doesn't cause a re-render, making it ideal for storing timers, previous values, or any mutable data that shouldn't affect rendering.

```javascript
function TextInput() {
  const inputRef = React.useRef(null);
  const [value, setValue] = React.useState('');

  // Access DOM element directly
  const focusInput = () => {
    inputRef.current.focus(); // Direct DOM access
  };

  // Store value without triggering re-render
  const timerRef = React.useRef(null);

  React.useEffect(() => {
    timerRef.current = setInterval(() => {
      console.log('Timer tick');
    }, 1000);

    return () => clearInterval(timerRef.current);
  }, []);

  // Store previous value (common pattern)
  const prevValueRef = React.useRef('');

  React.useEffect(() => {
    prevValueRef.current = value;
  }, [value]);

  const hasChanged = value !== prevValueRef.current;

  return (
    <div>
      <input
        ref={inputRef}
        value={value}
        onChange={e => setValue(e.target.value)}
      />
      <button onClick={focusInput}>Focus</button>
      {hasChanged && <span>Changed!</span>}
    </div>
  );
}
```

**useContext: Context Consumption**

The `useContext` hook subscribes to a React context and returns its current value. Context provides a way to pass data through the component tree without manually passing props at every level. It's useful for global state like themes, user authentication, or locale. The context value is determined by the nearest `Provider` component above in the tree.

```javascript
// Create context
const ThemeContext = React.createContext('light');

// Provider component
function ThemeProvider({ children, theme }) {
  return (
    <ThemeContext.Provider value={theme}>
      {children}
    </ThemeContext.Provider>
  );
}

// Consumer component using useContext
function ThemedButton() {
  const theme = React.useContext(ThemeContext);
  
  return (
    <button style={{
      backgroundColor: theme === 'dark' ? '#333' : '#fff',
      color: theme === 'dark' ? '#fff' : '#333'
    }}>
      Click me
    </button>
  );
}

// Usage in app
function App() {
  const [theme, setTheme] = React.useState('light');

  return (
    <ThemeProvider theme={theme}>
      <ThemedButton />
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Toggle Theme
      </button>
    </ThemeProvider>
  );
}
```

**useReducer: Complex State Logic**

The `useReducer` hook manages complex state logic using a reducer function, similar to Redux. It's useful when state depends on previous state, when there are multiple related state variables, or when state transitions follow specific patterns. The reducer receives current state and an action, returning the new state. This pattern makes state updates predictable and easier to test.

```javascript
// Reducer function
function counterReducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return { count: 0 };
    case 'set':
      return { count: action.payload };
    default:
      return state;
  }
}

// Component using useReducer
function Counter() {
  const [state, dispatch] = React.useReducer(counterReducer, { count: 0 });

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
      <button onClick={() => dispatch({ type: 'set', payload: 10 })}>Set to 10</button>
    </div>
  );
}

// Complex state example
function todoReducer(state, action) {
  switch (action.type) {
    case 'add':
      return [...state, { id: Date.now(), text: action.payload, completed: false }];
    case 'toggle':
      return state.map(todo =>
        todo.id === action.id ? { ...todo, completed: !todo.completed } : todo
      );
    case 'delete':
      return state.filter(todo => todo.id !== action.id);
    default:
      return state;
  }
}
```

**Custom Hooks: Creating Reusable Logic**

Custom hooks allow you to extract and reuse stateful logic between components. A custom hook is a function that uses other hooks and returns values or functions. By convention, custom hooks start with "use" and can call other hooks internally. This pattern promotes code reuse, reduces duplication, and makes components more focused on rendering.

```javascript
// Custom hook for fetching data
function useFetch(url) {
  const [data, setData] = React.useState(null);
  const [loading, setLoading] = React.useState(true);
  const [error, setError] = React.useState(null);

  React.useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(setData)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [url]);

  return { data, loading, error };
}

// Custom hook for local storage
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = React.useState(() => {
    return localStorage.getItem(key) || initialValue;
  });

  const setValue = (value) => {
    setStoredValue(value);
    localStorage.setItem(key, JSON.stringify(value));
  };

  return [storedValue, setValue];
}

// Custom hook for window size
function useWindowSize() {
  const [size, setSize] = React.useState({
    width: window.innerWidth,
    height: window.innerHeight
  });

  React.useEffect(() => {
    const handleResize = () => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    };

    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return size;
}

// Using custom hooks
function UserProfile({ userId }) {
  const { data: user, loading, error } = useFetch(`/api/users/${userId}`);
  const [theme, setTheme] = useLocalStorage('theme', 'light');

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return <div>{user?.name} (Theme: {theme})</div>;
}
```

**Rules of Hooks and Common Pitfalls**

Hooks must follow specific rules to work correctly: only call hooks at the top level of functional components, never inside loops, conditions, or nested functions. Always call hooks in the same order on every render. Common pitfalls include conditional hooks, hooks in callbacks, and missing dependencies in `useEffect` and `useCallback`.

```javascript
// ✅ Correct: Hooks at top level
function GoodComponent() {
  const [count, setCount] = React.useState(0);
  const [name, setName] = React.useState('');

  React.useEffect(() => {
    console.log('Effect runs');
  }, [count]);

  return <div>{count}</div>;
}

// ❌ Wrong: Conditional hooks
function BadComponent({ condition }) {
  const [count, setCount] = React.useState(0);

  if (condition) {
    // Hook inside condition - violates rules
    const [name, setName] = React.useState('');
    return <div>{name}</div>;
  }

  return <div>{count}</div>;
}

// ❌ Wrong: Hooks in loops
function VeryBadComponent({ items }) {
  const [counts, setCounts] = React.useState([]);

  items.forEach((item, index) => {
    // Hook inside loop - violates rules
    const [value, setValue] = React.useState(0);
    setCounts(prev => [...prev, value]);
  });

  return <div>{counts.join(', ')}</div>;
}

// ❌ Wrong: Hooks in callbacks
function AnotherBadComponent() {
  const [count, setCount] = React.useState(0);

  const handleClick = () => {
    // Hook inside callback - violates rules
    const [doubled, setDoubled] = React.useState(0);
    setDoubled(count * 2);
  };

  return <button onClick={handleClick}>Double</button>;
}

// Common pitfalls:
// 1. Missing dependencies in useEffect/useCallback
React.useEffect(() => {
  fetchData();
}); // Missing dependency - will cause stale closures

// 2. Including unstable dependencies
React.useEffect(() => {
  fetchData();
}, [props.user]); // user object changes on every render

// 3. Early returns before hooks
function BadComponent({ condition }) {
  if (condition) return <div>Early</div>;
  
  const [state, setState] = React.useState(0); // Hook after return - error!
  
  return <div>{state}</div>;
}
```

---

### Component Patterns

**Controlled vs Uncontrolled Components**

Controlled components have their form values controlled by React state, while uncontrolled components manage their own state internally. Controlled components give React full control over form data, making it easier to validate and manipulate. Uncontrolled components are simpler and can be easier to integrate with non-React code. The choice depends on use case: controlled for complex forms, uncontrolled for simple inputs.

```javascript
// Controlled component (React manages state)
function ControlledForm() {
  const [name, setName] = React.useState('');
  const [email, setEmail] = React.useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log({ name, email }); // Access current values from state
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={name}
        onChange={e => setName(e.target.value)}
        placeholder="Name"
      />
      <input
        value={email}
        onChange={e => setEmail(e.target.value)}
        placeholder="Email"
      />
      <button type="submit">Submit</button>
    </form>
  );
}

// Uncontrolled component (DOM manages state)
function UncontrolledForm() {
  const nameRef = React.useRef(null);
  const emailRef = React.useRef(null);

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log({
      name: nameRef.current.value,
      email: emailRef.current.value
    }); // Access values from DOM refs
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        ref={nameRef}
        defaultValue="Name"
        placeholder="Name"
      />
      <input
        ref={emailRef}
        defaultValue="Email"
        placeholder="Email"
      />
      <button type="submit">Submit</button>
    </form>
  );
}

// When to use each:
// Controlled: Complex forms, validation, multi-step forms
// Uncontrolled: Simple forms, integrating with non-React code
```

**Higher-Order Components (HOCs)**

Higher-Order Components are functions that take a component and return a new component with enhanced props or behavior. HOCs are used for code reuse, cross-cutting concerns like authentication, logging, or data fetching. While still useful, HOCs have been largely replaced by hooks for most use cases. Understanding HOCs helps maintain legacy code and understand certain React patterns.

```javascript
// HOC that adds loading state
function withLoading(WrappedComponent) {
  return function WithLoadingComponent(props) {
    const [loading, setLoading] = React.useState(false);

    if (loading) {
      return <div>Loading...</div>;
    }

    return <WrappedComponent {...props} />;
  };
}

// HOC that adds authentication check
function withAuth(WrappedComponent) {
  return function WithAuthComponent(props) {
    const [isAuthenticated, setIsAuthenticated] = React.useState(false);

    React.useEffect(() => {
      checkAuth().then(setIsAuthenticated);
    }, []);

    if (!isAuthenticated) {
      return <div>Please log in</div>;
    }

    return <WrappedComponent {...props} />;
  };
}

// Using HOCs
const UserProfileWithLoading = withLoading(UserProfile);
const UserProfileWithAuth = withAuth(UserProfile);

// In component
function App() {
  return <UserProfileWithAuth userId={123} />;
}

// HOC composition
function withLoadingAndAuth(WrappedComponent) {
  return withAuth(withLoading(WrappedComponent));
}
```

**Render Props Pattern**

The render props pattern involves passing a function as a prop that a component calls to determine what to render. This enables component logic reuse and data sharing between components. The component receiving the render prop is often called a "render prop component" or "headless component." This pattern has been largely replaced by hooks but is still useful in some scenarios.

```javascript
// Data provider component with render prop
function DataSource({ getData, render }) {
  const [data, setData] = React.useState(null);
  const [loading, setLoading] = React.useState(true);

  React.useEffect(() => {
    getData().then(setData).finally(() => setLoading(false));
  }, [getData]);

  return render({ data, loading });
}

// Using the component
function App() {
  return (
    <DataSource
      getData={() => fetch('/api/users')}
      render={({ data, loading }) => (
        <div>
          {loading ? (
            <p>Loading...</p>
          ) : (
            <ul>
              {data?.map(user => <li key={user.id}>{user.name}</li>)}
            </ul>
          )}
        </div>
      )}
    />
  );
}

// Multiple render props
function MultiRender({ renderHeader, renderContent, renderFooter }) {
  return (
    <div>
      {renderHeader()}
      {renderContent()}
      {renderFooter()}
    </div>
  );
}
```

**Compound Components**

Compound components are a pattern where multiple components work together to share implicit state through a shared context. The parent component manages state and provides sub-components that can be used together or separately. This pattern provides flexible APIs while maintaining control over internal state. Examples include UI libraries like Radix UI, Headless UI, and many form libraries.

```javascript
// Create context for compound component
const TabsContext = React.createContext();

// Parent compound component
function Tabs({ children, defaultIndex = 0 }) {
  const [activeIndex, setActiveIndex] = React.useState(defaultIndex);

  return (
    <TabsContext.Provider value={{ activeIndex, setActiveIndex }}>
      {children}
    </TabsContext.Provider>
  );
}

// Child components
function TabList({ children }) {
  const { activeIndex, setActiveIndex } = React.useContext(TabsContext);
  
  return (
    <div role="tablist">
      {React.Children.map(children, (child, index) => {
        if (React.isValidElement(child) && child.type === Tab) {
          return React.cloneElement(child, {
            isActive: index === activeIndex,
            onClick: () => setActiveIndex(index)
          });
        }
        return child;
      })}
    </div>
  );
}

function Tab({ children, isActive, onClick }) {
  return (
    <button
      role="tab"
      aria-selected={isActive}
      onClick={onClick}
      style={{ fontWeight: isActive ? 'bold' : 'normal' }}
    >
      {children}
    </button>
  );
}

function TabPanel({ children, index }) {
  const { activeIndex } = React.useContext(TabsContext);
  
  if (index !== activeIndex) return null;
  
  return <div>{children}</div>;
}

// Usage
function App() {
  return (
    <Tabs defaultIndex={0}>
      <TabList>
        <Tab>Tab 1</Tab>
        <Tab>Tab 2</Tab>
        <Tab>Tab 3</Tab>
      </TabList>
      <TabPanel index={0}>Content 1</TabPanel>
      <TabPanel index={1}>Content 2</TabPanel>
      <TabPanel index={2}>Content 3</TabPanel>
    </Tabs>
  );
}
```

**Container vs Presentational Components**

Container components (also called "smart" components) handle logic and state management, while presentational components (also called "dumb" components) focus solely on rendering based on props. This separation of concerns makes components more reusable, easier to test, and clearer in purpose. Containers connect to data sources, while presentational components receive data through props.

```javascript
// Presentational component (dumb, pure UI)
function UserCard({ user, onEdit, onDelete }) {
  return (
    <div className="user-card">
      <h2>{user.name}</h2>
      <p>{user.email}</p>
      <button onClick={() => onEdit(user.id)}>Edit</button>
      <button onClick={() => onDelete(user.id)}>Delete</button>
    </div>
  );
}

// Container component (smart, handles logic)
function UserCardContainer({ userId }) {
  const [user, setUser] = React.useState(null);

  React.useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);

  const handleEdit = (id) => {
    // Edit logic
  };

  const handleDelete = (id) => {
    // Delete logic
  };

  if (!user) return <div>Loading...</div>;

  return <UserCard user={user} onEdit={handleEdit} onDelete={handleDelete} />;
}

// Benefits of separation:
// - Presentational: Reusable, easy to test, focused on UI
// - Container: Manages state, business logic, data fetching
// - Clear separation of concerns
// - Easier to maintain and refactor
```

---

## Advanced React Architecture

### State Management

**Context API: Provider/Consumer Pattern**

The Context API provides a way to pass data through the component tree without prop drilling. A context is created with `React.createContext()`, provided by a `Provider` component, and consumed with `useContext()` hook or `Consumer` component. Context is ideal for global or semi-global state like themes, user authentication, or application settings.

```javascript
// Create context
const UserContext = React.createContext(null);

// Provider component
function UserProvider({ children }) {
  const [user, setUser] = React.useState(null);

  const login = (userData) => setUser(userData);
  const logout = () => setUser(null);

  const value = { user, login, logout };

  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
}

// Custom hook for consuming context
function useUser() {
  const context = React.useContext(UserContext);
  if (!context) {
    throw new Error('useUser must be used within UserProvider');
  }
  return context;
}

// Using context in components
function UserProfile() {
  const { user, logout } = useUser();

  if (!user) return <div>Please log in</div>;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
      <button onClick={logout}>Logout</button>
    </div>
  );
}

// Nested contexts
function App() {
  return (
    <ThemeProvider>
      <UserProvider>
        <NotificationProvider>
          <MainApp />
        </NotificationProvider>
      </UserProvider>
    </ThemeProvider>
  );
}
```

**Context Performance Pitfalls and Optimization**

Context can cause performance issues when consumers re-render unnecessarily due to context value changes. Every context value change triggers all consumers to re-render, even if they only use part of the value. Optimization strategies include splitting contexts by concern, memoizing context values, and using selectors to prevent unnecessary re-renders.

```javascript
// Problem: Single large context causes all consumers to re-render
const AppContext = React.createContext({
  user: null,
  theme: 'light',
  notifications: [],
  settings: {}
});

// Solution: Split into smaller contexts
const UserContext = React.createContext(null);
const ThemeContext = React.createContext('light');
const NotificationContext = React.createContext([]);

// Memoized context value to prevent unnecessary re-renders
function ThemeProvider({ children, theme }) {
  const memoizedValue = React.useMemo(
    () => ({ theme, toggleTheme: () => {} }),
    [theme]
  );

  return (
    <ThemeContext.Provider value={memoizedValue}>
      {children}
    </ThemeContext.Provider>
  );
}

// Selector pattern for optimized context consumption
function useUserSelector(selector) {
  const context = React.useContext(UserContext);
  return React.useMemo(() => selector(context), [context, selector]);
}

// Usage
function UserName() {
  const name = useUserSelector(user => user?.name);
  return <h1>{name}</h1>;
}

// Only re-renders when user.name changes, not entire context
```

**Redux Toolkit: Slices, Thunks, Selectors**

Redux Toolkit is the official recommended way to write Redux logic, providing simplified APIs and good defaults. It uses "slices" to organize state by feature, "thunks" for async actions, and "selectors" for deriving data. Redux Toolkit includes `configureStore`, `createSlice`, and `createAsyncThunk` utilities that reduce boilerplate and enforce best practices.

```javascript
import { configureStore, createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// Create a slice
const userSlice = createSlice({
  name: 'user',
  initialState: { user: null, loading: false, error: null },
  reducers: {
    setUser: (state, action) => {
      state.user = action.payload;
    },
    clearUser: (state) => {
      state.user = null;
    }
  },
  extraReducers: {
    // Async thunks handled separately
  }
});

// Create async thunk
const fetchUser = createAsyncThunk(
  'user/fetchUser',
  async (userId) => {
    const response = await fetch(`/api/users/${userId}`);
    return response.json();
  }
);

// Add extra reducers for async thunks
const userSliceWithThunks = userSlice;
userSliceWithThunks.extraReducers = {
  [fetchUser.pending]: (state) => {
    state.loading = true;
  },
  [fetchUser.fulfilled]: (state, action) => {
    state.loading = false;
    state.user = action.payload;
  },
  [fetchUser.rejected]: (state, action) => {
    state.loading = false;
    state.error = action.payload;
  }
};

// Configure store
const store = configureStore({
  reducer: {
    user: userSliceWithThunks
  }
});

// Selector
const selectUser = (state) => state.user.user;
const selectUserLoading = (state) => state.user.loading;

// Using in component
function UserProfile({ userId }) {
  const dispatch = useDispatch();
  const user = useSelector(selectUser);
  const loading = useSelector(selectUserLoading);

  React.useEffect(() => {
    dispatch(fetchUser(userId));
  }, [dispatch, userId]);

  if (loading) return <div>Loading...</div>;

  return <div>{user?.name}</div>;
}
```

**Zustand: Lightweight State Management**

Zustand is a minimal state management solution that uses a simple API without boilerplate. It creates a store with hooks for accessing state and actions, avoiding the complexity of Redux. Zustand is fast, has a small bundle size, and works well with TypeScript. It's ideal for small to medium applications where Redux would be overkill.

```javascript
import create from 'zustand';

// Create store
const useStore = create((set) => ({
  count: 0,
  increment: () => set(state => ({ count: state.count + 1 })),
  decrement: () => set(state => ({ count: state.count - 1 })),
  reset: () => set({ count: 0 }),
  user: null,
  setUser: (user) => set({ user })
}));

// Using the store in components
function Counter() {
  const { count, increment, decrement, reset } = useStore();

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}

function UserProfile() {
  const { user, setUser } = useStore();

  React.useEffect(() => {
    fetchUser().then(setUser);
  }, [setUser]);

  return user ? <div>{user.name}</div> : <div>Loading...</div>;
}

// Selectors with Zustand
const useCount = useStore(state => state.count);
const useUser = useStore(state => state.user);

// Async actions
const useStore = create((set) => ({
  user: null,
  fetchUser: async () => {
    const user = await fetch('/api/user');
    set({ user });
  }
}));
```

**State Management Selection Criteria**

Choosing the right state management solution depends on application complexity, team size, and specific requirements. Local state with `useState` is sufficient for simple, isolated component state. Context API works well for global state like themes or user authentication. Redux is suitable for complex, enterprise-scale applications with many interacting features. Zustand, Jotai, or Recoil offer simpler alternatives for medium complexity.

| Solution | Best For | Complexity | Bundle Size | Learning Curve |
|----------|-----------|------------|-------------|----------------|
| useState | Simple components | Low | Minimal | None |
| Context API | Global state, themes, auth | Low-Medium | Minimal | Low |
| Redux Toolkit | Large, complex apps | High | Medium | Medium |
| Zustand | Medium apps | Low-Medium | Small | Low |
| Recoil | Medium apps | Medium | Medium | Medium |
| Jotai | Medium apps | Medium | Small | Low |
```

### Performance Optimization

**React.memo() for Component Memoization**

`React.memo()` is a higher-order component that memoizes the result, preventing re-renders when props haven't changed. It does a shallow comparison of props by default, but can accept a custom comparison function. Memoization is useful for components that render frequently with expensive operations or are deep in the component tree.

```javascript
// Memoized component (only re-renders when props change)
const ExpensiveComponent = React.memo(function ExpensiveComponent({ data, onClick }) {
  console.log('ExpensiveComponent rendered');
  
  const processedData = processData(data); // Expensive computation
  
  return (
    <div onClick={onClick}>
      {processedData.map(item => <div key={item.id}>{item.name}</div>)}
    </div>
  );
});

// With custom comparison function
const UserCard = React.memo(function UserCard({ user }) {
  return <div>{user.name}</div>;
}, (prevProps, nextProps) => {
  // Only re-render if user.id changed
  return prevProps.user.id !== nextProps.user.id;
});

// Parent component
function App() {
  const [data, setData] = React.useState([]);
  const [count, setCount] = React.useState(0);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Re-render parent</button>
      <ExpensiveComponent data={data} />
      {/* ExpensiveComponent won't re-render if data hasn't changed */}
    </div>
  );
}
```

**useMemo() and useCallback() Anti-Patterns**

Overusing `useMemo()` and `useCallback()` can hurt performance due to memoization overhead and complexity. Memoization has a cost: creating and maintaining caches, comparing dependencies, and potential memory issues. Only memoize when there's a measurable performance problem, not preemptively. Avoid memoizing simple operations, small arrays, or values that change frequently.

```javascript
// ❌ Anti-pattern: Memoizing simple operations
function BadComponent({ items }) {
  const doubled = React.useMemo(
    () => items.map(item => item * 2),
    [items]
  );
  
  return <div>{doubled.join(', ')}</div>;
}

// ✅ Better: Direct computation
function GoodComponent({ items }) {
  const doubled = items.map(item => item * 2);
  
  return <div>{doubled.join(', ')}</div>;
}

// ❌ Anti-pattern: Memoizing frequently changing values
function BadCounter() {
  const [count, setCount] = React.useState(0);
  
  const doubled = React.useMemo(() => count * 2, [count]);
  // Memoization overhead on every state change
  
  return <div>{doubled}</div>;
}

// ✅ Better: Direct computation
function GoodCounter() {
  const [count, setCount] = React.useState(0);
  
  const doubled = count * 2; // Simple multiplication is fast
  
  return <div>{doubled}</div>;
}

// ❌ Anti-pattern: Unstable dependencies
function BadComponent({ items }) {
  const processed = React.useMemo(
    () => items.filter(item => item.active),
    [items] // items reference changes on every render
  );
  
  return <div>{/* ... */}</div>;
}

// ✅ Better: Stable dependencies
function GoodComponent({ items }) {
  const processed = React.useMemo(
    () => items.filter(item => item.active),
    [items.length] // Use length instead of array reference
  );
  
  return <div>{/* ... */}</div>;
}
```

**Code Splitting with React.lazy() and Suspense**

Code splitting divides application code into smaller chunks that are loaded on demand, reducing initial bundle size and improving load time. `React.lazy()` enables lazy loading of components with dynamic imports, while `Suspense` provides a fallback UI while the lazy component loads. This pattern is essential for large applications to improve perceived performance and user experience.

```javascript
// Lazy load a component
const LazyDashboard = React.lazy(() => import('./Dashboard'));
const LazySettings = React.lazy(() => import('./Settings'));

// Using Suspense for fallback UI
function App() {
  return (
    <div>
      <Header />
      <Suspense fallback={<div>Loading dashboard...</div>}>
        <LazyDashboard />
      </Suspense>
      <Suspense fallback={<div>Loading settings...</div>}>
        <LazySettings />
      </Suspense>
      <Footer />
    </div>
  );
}

// Lazy loading with error boundary
function AppWithErrorBoundary() {
  return (
    <ErrorBoundary fallback={<div>Something went wrong</div>}>
      <App />
    </ErrorBoundary>
  );
}

// Route-based code splitting
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Contact = lazy(() => import('./pages/Contact'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<div>Loading...</div>}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
          <Route path="/contact" element={<Contact />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

**Virtual Scrolling for Large Lists**

Virtual scrolling renders only visible items in a large list, dramatically improving performance for lists with thousands of items. Libraries like `react-window` and `react-virtualized` provide components that handle the complex logic of calculating visible items and recycling DOM elements. This technique reduces DOM nodes and memory usage while maintaining smooth scrolling performance.

```javascript
import { FixedSizeList } from 'react-window';

function VirtualList({ items }) {
  // Render only visible items (e.g., 20 out of 10,000)
  const Row = ({ index, style }) => (
    <div style={style}>
      Item {index}: {items[index].name}
    </div>
  );

  return (
    <FixedSizeList
      height={50}
      itemCount={items.length}
      itemSize={50}
      width={800}
    >
      {Row}
    </FixedSizeList>
  );
}

// Without virtualization (slow for large lists)
function SlowList({ items }) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}

// Benefits of virtual scrolling:
// - Constant DOM nodes regardless of list size
// - Smooth scrolling performance
// - Reduced memory usage
// - Better UX for large datasets
```

**Preventing Unnecessary Re-Renders**

Unnecessary re-renders occur when components update without actual data changes, wasting CPU cycles and degrading performance. Prevention strategies include memoizing components with `React.memo()`, using stable props and callbacks, optimizing context consumption, and structuring components to minimize propagation of updates. Profiling with React DevTools helps identify re-render issues.

```javascript
// ❌ Problem: Inline function creates new reference on each render
function BadParent() {
  const [count, setCount] = React.useState(0);

  return (
    <div>
      <Child onClick={() => setCount(count + 1)} />
      {/* Child re-renders on every Parent render */}
    </div>
  );
}

// ✅ Solution: useCallback for stable function reference
function GoodParent() {
  const [count, setCount] = React.useState(0);

  const handleClick = React.useCallback(() => {
    setCount(c => c + 1);
  }, []); // Empty deps = stable reference

  return (
    <div>
      <Child onClick={handleClick} />
      {/* Child only re-renders when handleClick changes */}
    </div>
  );
}

// ❌ Problem: New object reference on each render
function BadParent() {
  const [items, setItems] = React.useState([]);

  return <Child config={{ items, onToggle: () => {}} />;
}

// ✅ Solution: useMemo for stable object reference
function GoodParent() {
  const [items, setItems] = React.useState([]);

  const config = React.useMemo(
    () => ({ items, onToggle: () => {} }),
    [items]
  );

  return <Child config={config} />;
}

// Profile with React DevTools to identify re-renders
import { Profiler } from 'react';

function App() {
  return (
    <Profiler id="App" onRender={(id, phase, actualDuration) => {
      console.log(`${id} ${phase} took ${actualDuration}ms`);
    }}>
      <YourApp />
    </Profiler>
  );
}
```

**Profiling React Apps with DevTools**

React DevTools provides profiling capabilities to measure component render performance and identify bottlenecks. The profiler records why components rendered, how long they took, and what caused re-renders. This information helps optimize components by identifying expensive renders, unnecessary re-renders, and state update patterns.

```javascript
import { Profiler } from 'react';

// Profiling a component
function ProfiledComponent() {
  return (
    <Profiler
      id="ProfiledComponent"
      onRender={(id, phase, actualDuration, baseDuration, startTime, commitTime, interactions) => {
        console.log('Component:', id);
        console.log('Phase:', phase);
        console.log('Duration:', actualDuration);
        console.log('Base duration:', baseDuration);
        console.log('Interactions:', interactions);
      }}
    >
      <ExpensiveComponent />
    </Profiler>
  );
}

// Flame graph visualization in DevTools
// - Yellow bars: Longer render times
// - Gray bars: Idle time
// - Component hierarchy: See which components caused renders

// Recording profile
// 1. Open React DevTools
// 2. Go to Profiler tab
// 3. Click "Record"
// 4. Interact with your app
// 5. Stop recording
// 6. Analyze the flame graph
```

**React Memory Leak Patterns & Prevention**

Memory leaks in React occur when components retain references to resources after unmounting, preventing garbage collection. Common causes include event listeners not removed, timers not cleared, subscriptions not unsubscribed, and closures retaining large objects. Prevention involves always returning cleanup functions from `useEffect` and using tools like React DevTools and ESLint to detect leaks.

```javascript
// ❌ Memory leak: Event listener not removed
function LeakyComponent() {
  React.useEffect(() => {
    const handler = () => console.log('clicked');
    window.addEventListener('click', handler);
    // Missing: return () => window.removeEventListener('click', handler);
  }, []);

  return <div>Click anywhere</div>;
}

// ✅ Fixed: Cleanup event listener
function FixedComponent() {
  React.useEffect(() => {
    const handler = () => console.log('clicked');
    window.addEventListener('click', handler);

    // Return cleanup function
    return () => {
      window.removeEventListener('click', handler);
    };
  }, []);

  return <div>Click anywhere</div>;
}

// ❌ Memory leak: Timer not cleared
function LeakyTimer() {
  React.useEffect(() => {
    const timer = setInterval(() => {
      console.log('Tick');
    }, 1000);
    // Missing: return () => clearInterval(timer);
  }, []);

  return <div>Timer running</div>;
}

// ✅ Fixed: Cleanup timer
function FixedTimer() {
  React.useEffect(() => {
    const timer = setInterval(() => {
      console.log('Tick');
    }, 1000);

    return () => clearInterval(timer);
  }, []);

  return <div>Timer running</div>;
}

// ❌ Memory leak: Subscription not unsubscribed
function LeakySubscription() {
  React.useEffect(() => {
    const subscription = dataSource.subscribe(data => {
      console.log('Data:', data);
    });
    // Missing: return () => subscription.unsubscribe();
  }, []);

  return <div>Listening for data</div>;
}

// ✅ Fixed: Cleanup subscription
function FixedSubscription() {
  React.useEffect(() => {
    const subscription = dataSource.subscribe(data => {
      console.log('Data:', data);
    });

    return () => subscription.unsubscribe();
  }, []);

  return <div>Listening for data</div>;
}

// Detection with React DevTools
// - Use "Highlight updates when components render" in Profiler
// - Look for components that render when props haven't changed
// - Check for memory usage in Memory tab
// - Use "Component" tab to inspect hooks and state
```

---

### Routing

**React Router: BrowserRouter, Routes, Route**

React Router is the standard routing library for React applications, providing declarative routing with components like `BrowserRouter`, `Routes`, and `Route`. It manages browser history, handles URL changes, and renders components based on the current path. The library supports nested routes, route parameters, query strings, and programmatic navigation.

```javascript
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/users/:id" element={<UserProfile />} />
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}

// Navigation with Link component
function Navigation() {
  return (
    <nav>
      <Link to="/">Home</Link>
      <Link to="/about">About</Link>
      <Link to="/users/123">User 123</Link>
    </nav>
  );
}
```

**Route Parameters and Query Strings**

Route parameters capture dynamic segments of the URL path (e.g., `/users/:id`), while query strings contain optional data after `?` (e.g., `/users?page=2&sort=name`). React Router provides `useParams()` hook for route parameters and `useSearchParams()` or `useLocation()` for query strings. Understanding the difference is crucial for building dynamic applications.

```javascript
import { useParams, useSearchParams } from 'react-router-dom';

function UserProfile() {
  // Route parameter: /users/:id
  const { id } = useParams();
  
  // Query string: /users?id=123&page=2
  const [searchParams] = useSearchParams();
  const page = searchParams.get('page');
  const sort = searchParams.get('sort');

  return (
    <div>
      <h2>User ID: {id}</h2>
      <p>Page: {page}</p>
      <p>Sort: {sort}</p>
    </div>
  );
}

// Programmatic navigation with parameters
function UserList() {
  const navigate = useNavigate();

  const goToUser = (userId) => {
    navigate(`/users/${userId}?tab=profile`);
  };

  return (
    <div>
      {users.map(user => (
        <div key={user.id} onClick={() => goToUser(user.id)}>
          {user.name}
        </div>
      ))}
    </div>
  );
}
```

**Nested Routes and Outlet**

Nested routes allow creating hierarchical route structures where child routes render within parent layouts. The `Outlet` component renders child route content at the appropriate location in the parent component. This pattern is useful for layouts with shared headers, sidebars, or navigation that persist across multiple pages.

```javascript
import { Routes, Route, Outlet, Link } from 'react-router-dom';

// Parent layout component
function Layout() {
  return (
    <div className="layout">
      <header>
        <Link to="/">Home</Link>
        <Link to="/dashboard">Dashboard</Link>
      </header>
      <main>
        <Outlet /> {/* Child routes render here */}
      </main>
      <footer>© 2024</footer>
    </div>
  );
}

// Nested route configuration
function App() {
  return (
    <Routes>
      <Route path="/" element={<Layout />}>
        <Route index element={<Home />} />
        <Route path="dashboard" element={<Dashboard />} />
        <Route path="settings" element={<Settings />} />
      </Route>
    </Routes>
  );
}

// Dashboard with its own nested routes
function Dashboard() {
  return (
    <Routes>
      <Route index element={<DashboardHome />} />
      <Route path="overview" element={<Overview />} />
      <Route path="reports" element={<Reports />} />
    </Routes>
  );
}
```

**Programmatic Navigation**

Programmatic navigation allows navigating between routes without user interaction, useful for redirects after login, form submissions, or conditional navigation. React Router provides `useNavigate()` hook for navigation, which can navigate to absolute paths, relative paths, or replace the current history. Navigation can be triggered programmatically or through the `Navigate` component.

```javascript
import { useNavigate, Navigate } from 'react-router-dom';

function LoginForm() {
  const navigate = useNavigate();

  const handleLogin = async (credentials) => {
    const success = await login(credentials);
    
    if (success) {
      // Navigate to dashboard after successful login
      navigate('/dashboard');
    }
  };

  return <form onSubmit={handleLogin}>...</form>;
}

function ProtectedRoute({ children }) {
  const isAuthenticated = useAuth();

  // Redirect if not authenticated
  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }

  return children;
}

function LogoutButton() {
  const navigate = useNavigate();

  const handleLogout = () => {
    logout();
    // Replace history (can't go back to protected page)
    navigate('/login', { replace: true });
  };

  return <button onClick={handleLogout}>Logout</button>;
}

// Navigation with state
function UserProfile() {
  const navigate = useNavigate();

  const handleEdit = () => {
    navigate('/users/123/edit', { state: { fromProfile: true } });
  };

  return <button onClick={handleEdit}>Edit Profile</button>;
}
```

**Route Guards and Protected Routes**

Route guards protect routes from unauthorized access by checking authentication or permissions before rendering. They typically redirect unauthenticated users to login pages or show access denied messages. Guards can be implemented as wrapper components, using the `Navigate` component, or with custom routing logic. This pattern is essential for applications with authentication and authorization requirements.

```javascript
import { Navigate, Outlet, useLocation } from 'react-router-dom';

// Auth guard component
function AuthGuard({ children }) {
  const { isAuthenticated, user } = useAuth();
  const location = useLocation();

  // Save intended destination for redirect after login
  React.useEffect(() => {
    if (!isAuthenticated) {
      // Store the location user was trying to access
      sessionStorage.setItem('redirectAfterLogin', location.pathname);
    }
  }, [isAuthenticated, location]);

  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }

  return children;
}

// Permission guard component
function PermissionGuard({ children, requiredPermission }) {
  const { user } = useAuth();

  if (!user?.permissions?.includes(requiredPermission)) {
    return <Navigate to="/unauthorized" replace />;
  }

  return children;
}

// Usage in routes
function App() {
  return (
    <Routes>
      <Route
        path="/dashboard"
        element={
          <AuthGuard>
            <Dashboard />
          </AuthGuard>
        }
      />
      <Route
        path="/admin"
        element={
          <PermissionGuard requiredPermission="admin">
            <AdminPanel />
          </PermissionGuard>
        }
      />
    </Routes>
  );
}

// Custom route guard with logic
function RoleGuard({ children, allowedRoles }) {
  const { user } = useAuth();

  const hasAccess = allowedRoles.includes(user?.role);

  if (!hasAccess) {
    return <Navigate to="/forbidden" replace />;
  }

  return children;
}
```

**Lazy Loading Route Components**

Lazy loading route components reduces initial bundle size by loading route code only when needed. Combined with `Suspense`, it provides fallback UI while the lazy component loads. This improves perceived performance, especially for large applications with many routes. Each route can be code-split independently based on usage patterns.

```javascript
import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';

// Lazy load route components
const Home = lazy(() => import('./pages/Home'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));
const Admin = lazy(() => import('./pages/Admin'));

// Loading fallback component
function PageLoader() {
  return <div className="loader">Loading page...</div>;
}

function App() {
  return (
    <Suspense fallback={<PageLoader />}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
        <Route path="/admin" element={<Admin />} />
      </Routes>
    </Suspense>
  );
}

// Grouping lazy loaded routes
const routes = [
  {
    path: '/reports',
    component: lazy(() => import('./pages/Reports'))
  },
  {
    path: '/analytics',
    component: lazy(() => import('./pages/Analytics'))
  }
];

// Render routes dynamically
function App() {
  return (
    <Suspense fallback={<PageLoader />}>
      <Routes>
        {routes.map((route, index) => (
          <Route
            key={index}
            path={route.path}
            element={<route.component />}
          />
        ))}
      </Routes>
    </Suspense>
  );
}
```

---

### Forms & Validation

**Controlled Form Inputs**

Controlled form inputs have their values controlled by React state, with changes handled through `onChange` handlers. This pattern gives React full control over form data, enabling validation, transformation, and conditional rendering based on input. Controlled inputs are the standard approach in React applications and work well with form validation libraries.

```javascript
function ContactForm() {
  const [formData, setFormData] = React.useState({
    name: '',
    email: '',
    message: ''
  });

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Submitted:', formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="name"
        value={formData.name}
        onChange={handleChange}
        placeholder="Name"
      />
      <input
        name="email"
        value={formData.email}
        onChange={handleChange}
        placeholder="Email"
      />
      <textarea
        name="message"
        value={formData.message}
        onChange={handleChange}
        placeholder="Message"
      />
      <button type="submit">Send</button>
    </form>
  );
}
```

**Form Validation Strategies**

Form validation ensures user input meets requirements before submission. Strategies include real-time validation (as user types), on-blur validation (when leaving field), and on-submit validation (before form submission). Validation can be implemented manually with state or using libraries like React Hook Form, Formik, or Zod. Proper validation provides immediate feedback and prevents invalid data from reaching the server.

```javascript
// Manual validation with state
function LoginForm() {
  const [email, setEmail] = React.useState('');
  const [password, setPassword] = React.useState('');
  const [errors, setErrors] = React.useState({});

  const validate = () => {
    const newErrors = {};
    
    if (!email) newErrors.email = 'Email is required';
    else if (!/\S+@\S+\.\S+/.test(email)) {
      newErrors.email = 'Invalid email format';
    }
    
    if (!password) newErrors.password = 'Password is required';
    else if (password.length < 8) {
      newErrors.password = 'Password must be at least 8 characters';
    }
    
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  return (
    <form onSubmit={(e) => { e.preventDefault(); if (validate()) console.log('Valid'); }}>
      <input
        value={email}
        onChange={e => setEmail(e.target.value)}
        placeholder="Email"
      />
      {errors.email && <span className="error">{errors.email}</span>}
      
      <input
        type="password"
        value={password}
        onChange={e => setPassword(e.target.value)}
        placeholder="Password"
      />
      {errors.password && <span className="error">{errors.password}</span>}
      
      <button type="submit">Login</button>
    </form>
  );
}

// Validation with Zod
import { z } from 'zod';

const loginSchema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(8, 'Password must be at least 8 characters')
});

function ZodLoginForm() {
  const [errors, setErrors] = React.useState({});

  const handleSubmit = (e) => {
    e.preventDefault();
    
    const formData = {
      email: e.target.email.value,
      password: e.target.password.value
    };

    const result = loginSchema.safeParse(formData);
    
    if (!result.success) {
      setErrors(result.error.flatten().fieldErrors);
      return;
    }

    console.log('Valid:', result.data);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="email" />
      {errors.email && <span className="error">{errors.email}</span>}
      
      <input name="password" type="password" />
      {errors.password && <span className="error">{errors.password}</span>}
      
      <button type="submit">Login</button>
    </form>
  );
}
```

**Form Libraries: React Hook Form, Formik**

React Hook Form and Formik are popular form libraries that reduce boilerplate and provide built-in validation. React Hook Form uses refs for performance and has a smaller bundle size. Formik provides a more feature-rich API with better TypeScript support. Both libraries integrate with validation schemas (Zod, Yup) and handle form state, submission, and error management automatically.

```javascript
// Using React Hook Form
import { useForm } from 'react-hook-form';

function RHFLoginForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting }
  } = useForm({
    defaultValues: { email: '', password: '' },
    resolver: zodResolver(loginSchema)
  });

  const onSubmit = async (data) => {
    try {
      await login(data);
      console.log('Logged in:', data);
    } catch (error) {
      console.error('Login failed:', error);
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        {...register('email')}
        placeholder="Email"
      />
      {errors.email && <span className="error">{errors.email.message}</span>}
      
      <input
        {...register('password')}
        type="password"
        placeholder="Password"
      />
      {errors.password && <span className="error">{errors.password.message}</span>}
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
}

// Using Formik
import { Formik, Form, Field, ErrorMessage } from 'formik';

function FormikLoginForm() {
  return (
    <Formik
      initialValues={{ email: '', password: '' }}
      validationSchema={loginSchema}
      onSubmit={async (values, { setSubmitting }) => {
        await login(values);
        setSubmitting(false);
      }}
    >
      {({ isSubmitting }) => (
        <Form>
          <Field name="email" type="email" placeholder="Email" />
          <ErrorMessage name="email" component="span" className="error" />
          
          <Field name="password" type="password" placeholder="Password" />
          <ErrorMessage name="password" component="span" className="error" />
          
          <button type="submit" disabled={isSubmitting}>
            {isSubmitting ? 'Logging in...' : 'Login'}
          </button>
        </Form>
      )}
    </Formik>
  );
}
```

**Error Handling and User Feedback**

Form error handling involves displaying validation errors, submission errors, and loading states to users. Errors should be clear, specific, and positioned near the related field. Loading states provide feedback during async operations. Success messages confirm completed actions. Good error handling improves user experience and reduces support requests.

```javascript
function RegistrationForm() {
  const [status, setStatus] = React.useState('idle'); // idle, loading, success, error
  const [error, setError] = React.useState(null);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setStatus('loading');
    setError(null);

    try {
      await register(formData);
      setStatus('success');
      setTimeout(() => setStatus('idle'), 3000);
    } catch (err) {
      setError(err.message);
      setStatus('error');
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      {/* Form fields */}
      
      {status === 'loading' && <div className="loading">Registering...</div>}
      
      {status === 'success' && (
        <div className="success">
          Registration successful! Redirecting...
        </div>
      )}
      
      {status === 'error' && (
        <div className="error">
          Error: {error}
          <button onClick={() => setStatus('idle')}>Try again</button>
        </div>
      )}
    </form>
  );
}
```

---

## React Ecosystem & SSR

### Server-Side Rendering (SSR)

**SSR vs Client-Side Rendering (CSR)**

Server-Side Rendering generates HTML on the server for each request, improving initial load time and SEO. Client-Side Rendering renders HTML in the browser using JavaScript, providing more interactivity after initial load. SSR is better for SEO, social media sharing, and first contentful paint, while CSR is better for complex applications requiring significant interactivity. Next.js supports both approaches with flexible configuration.

```
SSR Flow:
Request → Server renders HTML → Browser displays → Hydration → Interactive

CSR Flow:
Request → Browser loads minimal HTML → JavaScript executes → React renders → Interactive

SSR Benefits:
- Better SEO (search engines can crawl content)
- Faster first contentful paint
- Social media preview (Open Graph tags)
- Better performance on slow devices

CSR Benefits:
- Better interactivity after initial load
- Easier client-side navigation
- Reduced server load
```

**Next.js Fundamentals**

Next.js is a React framework that provides server-side rendering, static site generation, and API routes. It handles routing, code splitting, and optimization automatically. Next.js uses file-based routing, where files in the `pages/` or `app/` directory become routes automatically. The framework includes built-in image optimization, font optimization, and production build optimizations.

```javascript
// File-based routing (pages directory)
// pages/index.js → /
// pages/about.js → /about
// pages/users/[id].js → /users/123

// Dynamic routes
export default function UserProfile({ params }) {
  return <div>User ID: {params.id}</div>;
}

// getServerSideProps for SSR
export async function getServerSideProps(context) {
  const res = await fetch('/api/user');
  const user = await res.json();
  
  return {
    props: { user },
    // Optional: revalidate: 60 // Revalidate every 60 seconds
  };
}

// getStaticProps for SSG
export async function getStaticProps() {
  const posts = await fetch('/api/posts');
  return {
    props: { posts },
    revalidate: 3600 // Revalidate every hour
  };
}
```

**Static Site Generation (SSG)**

Static Site Generation pre-renders pages at build time, creating static HTML files that can be served by a CDN. This approach provides the best performance and lowest cost for content that doesn't change frequently. SSG is ideal for marketing pages, documentation, blogs, and other content that can be generated ahead of time.

```javascript
// Generate static pages at build time
export async function getStaticProps() {
  const posts = await fetch('/api/posts');
  
  return {
    props: { posts },
    revalidate: 3600 // Revalidate every hour (ISR)
  };
}

// Fallback pages for dynamic routes
export async function getStaticPaths() {
  const posts = await fetch('/api/posts');
  
  return {
    paths: posts.map(post => ({ params: { id: post.id.toString() } })),
    fallback: 'fallback' // Show fallback page for ungenerated paths
  };
}

// Incremental Static Regeneration (ISR)
// Combines SSG performance with SSR flexibility
// Pages are regenerated on-demand when revalidate time expires
// Good for frequently changing content (blogs, products)
```

**getServerSideProps and getStaticProps**

`getServerSideProps` fetches data on each request for server-side rendering, while `getStaticProps` fetches data at build time for static generation. `getServerSideProps` is used for dynamic data that changes frequently or requires request context, while `getStaticProps` is used for static content that can be pre-built. Choosing the right method impacts performance, caching, and build times.

```javascript
// getServerSideProps (runs on every request)
export async function getServerSideProps(context) {
  const { req, res, query } = context;
  const userId = query.id;
  
  // Access cookies, headers, etc.
  const token = req.cookies.token;
  
  const user = await fetchUser(userId, token);
  
  if (!user) {
    return {
      notFound: true // Render 404 page
    };
  }
  
  return {
    props: { user },
    // Optional: redirect: { destination: '/login', permanent: false }
  };
}

// getStaticProps (runs at build time)
export async function getStaticProps() {
  const posts = await fetch('/api/posts');
  
  return {
    props: { posts },
    revalidate: 3600 // Revalidate every hour
  };
}

// getStaticPaths (for dynamic SSG routes)
export async function getStaticPaths() {
  const posts = await fetch('/api/posts');
  
  return {
    paths: [
      { params: { id: '1' } },
      { params: { id: '2' } },
      { params: { id: '3' } }
    ],
    fallback: 'fallback'
  };
}
```

---

### Build Tools & Configuration

**Create React App vs Vite vs Next.js**

Create React App (CRA) is the official zero-config build tool for React, providing a pre-configured setup with webpack. Vite is a modern build tool that uses esbuild for faster builds and better development experience. Next.js is a full-featured framework with SSR, routing, and optimization built-in. The choice depends on project requirements: CRA for simple SPAs, Vite for faster development, Next.js for SSR/SSG needs.

| Feature | Create React App | Vite | Next.js |
|---------|------------------|-------|----------|
| Build Speed | Medium | Fast | Medium |
| Dev Server | Medium | Very Fast | Fast |
| HMR | Slow | Instant | Fast |
| Bundle Size | Larger | Smaller | Optimized |
| SSR | No | No | Yes |
| Routing | React Router | React Router | Built-in |
| Config | Low | Low | Medium |
| Learning Curve | Low | Low | Medium |

**Webpack Configuration Basics**

Webpack is a module bundler that takes dependencies and assets, transforming them into optimized bundles for the browser. React applications often use webpack through Create React App or custom configurations. Key concepts include entry points, loaders for different file types, plugins for optimization, and output configuration.

```javascript
// Basic webpack configuration
module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
    publicPath: '/'
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: 'babel-loader'
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './public/index.html'
    }),
    new MiniCssExtractPlugin({
      filename: 'styles.css'
    })
  ],
  optimization: {
    splitChunks: 'all',
    minimize: true
  }
};
```

**Environment Variables in React**

Environment variables in React applications are accessed through `process.env` and are typically defined in `.env` files. Create React App and Vite automatically load `.env` files, prefixing variables with `REACT_APP_` for security. Next.js provides `process.env` access and supports both server and client environment variables. Environment-specific configuration enables different behavior across development, staging, and production.

```javascript
// .env file
REACT_APP_API_URL=https://api.example.com
REACT_APP_ENVIRONMENT=development

// Accessing in React code
const apiUrl = process.env.REACT_APP_API_URL;
const environment = process.env.REACT_APP_ENVIRONMENT;

// Using in components
function App() {
  const [data, setData] = React.useState(null);

  React.useEffect(() => {
    if (environment === 'development') {
      console.log('Development mode');
    }
    
    fetch(`${apiUrl}/data`).then(setData);
  }, [apiUrl, environment]);

  return <div>{/* ... */}</div>;
}

// Next.js environment variables
// .env.local (not committed)
NEXT_PUBLIC_API_URL=https://api.example.com
NEXT_PUBLIC_ENVIRONMENT=development

// Accessing in Next.js
const apiUrl = process.env.NEXT_PUBLIC_API_URL;
const environment = process.env.NEXT_PUBLIC_ENVIRONMENT;

// Server-side only (not exposed to browser)
const databaseUrl = process.env.DATABASE_URL;
```

**Production Build Optimization**

Production build optimization reduces bundle size, improves runtime performance, and enhances user experience. Techniques include code splitting, tree shaking, minification, compression, and asset optimization. React's production build automatically applies many optimizations, but additional configuration can further improve performance.

```javascript
// Code splitting with React.lazy
const Dashboard = React.lazy(() => import('./Dashboard'));
const Settings = React.lazy(() => import('./Settings'));

// Tree shaking (ES6 modules)
import { useState, useEffect } from 'react'; // Only imports what's used

// Compression with webpack
const CompressionPlugin = require('compression-webpack-plugin');

module.exports = {
  plugins: [
    new CompressionPlugin({
      algorithm: 'gzip',
      test: /\.(js|css|html|svg)$/
    })
  ]
};

// Image optimization
// Next.js: Automatic with next/image
import Image from 'next/image';

<Image
  src="/profile.jpg"
  width={500}
  height={500}
  loading="lazy"
/>

// Asset optimization
// Minify CSS, optimize images, use modern formats (WebP)
```

---

## Quick Reference

**React Hooks Quick Reference**

| Hook | Purpose | Dependencies | Return Value |
|-------|-----------|--------------|--------------|
| useState | State management | Initial value | [state, setState] |
| useEffect | Side effects | Dependencies, cleanup function | void |
| useCallback | Memoize functions | Dependencies | Memoized callback |
| useMemo | Memoize values | Dependencies | Memoized value |
| useRef | Persistent refs | None | { current: value } |
| useContext | Consume context | Context object | Context value |
| useReducer | Complex state | Reducer, initial state | [state, dispatch] |
| useLayoutEffect | Synchronous effects | Dependencies, cleanup | void |

**Component Patterns Quick Reference**

| Pattern | Use Case | Example |
|---------|-----------|---------|
| Controlled Components | Forms, validation | `<input value={value} onChange={handleChange} />` |
| Uncontrolled Components | Simple inputs, legacy integration | `<input defaultValue="value" ref={inputRef} />` |
| HOC | Code reuse, cross-cutting concerns | `withAuth(Component)` |
| Render Props | Component flexibility, data sharing | `<DataList render={item => <Item item={item} />} />` |
| Compound Components | Related components, shared state | `<Tabs><Tab /><TabPanel /></Tabs>` |
| Container/Presentational | Separation of concerns | Container: logic, Presentational: UI |

**Performance Optimization Checklist**

- [ ] Profile components with React DevTools
- [ ] Use `React.memo()` for expensive components
- [ ] Memoize callbacks with `useCallback()`
- [ ] Memoize values with `useMemo()` (only when necessary)
- [ ] Implement code splitting with `React.lazy()`
- [ ] Use virtual scrolling for large lists
- [ ] Optimize images and assets
- [ ] Enable production builds with minification
- [ ] Monitor bundle size with tools like webpack-bundle-analyzer

**State Management Decision Tree**

```
Need State Management?
    ↓
Is it simple, local state?
    ↓ YES → Use useState
    ↓ NO
Is it global state (theme, auth)?
    ↓ YES → Use Context API
    ↓ NO
Is it complex, interconnected state?
    ↓ YES → Use Redux Toolkit / Zustand
    ↓ NO → Consider if you actually need it
```

**Common React Mistakes to Avoid**

- ❌ Using index as keys in lists
- ❌ Mutating state directly (`state.push()` instead of `setState([...state, item])`)
- ❌ Missing dependencies in `useEffect`/`useCallback`
- ❌ Calling hooks conditionally or in loops
- ❌ Not cleaning up effects (timers, listeners, subscriptions)
- ❌ Over-memoizing with `useMemo`/`useCallback`
- ❌ Using `useEffect` for state that should be in render
- ❌ Prop drilling instead of using Context
- ❌ Inline object/array creation in dependencies

**React vs Other Frameworks Comparison**

| Feature | React | Vue | Svelte |
|---------|-------|-----|--------|
| Virtual DOM | Yes | Yes | No (compiles to DOM) |
| Learning Curve | Medium | Easy | Very Easy |
| Bundle Size | Medium (42KB) | Small (34KB) | Tiny (1.6KB) |
| Ecosystem | Large | Medium | Growing |
| TypeScript Support | Excellent | Excellent | Good |
| Performance | Fast | Very Fast | Extremely Fast |
| SSR Support | Yes (Next.js) | Yes (Nuxt.js) | Yes (SvelteKit) |
