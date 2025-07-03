# React Notes

## Basics

1. **React** is a JavaScript library for building user interfaces. It lets you create reusable UI components that manage their own state.

2. **JSX** is React's syntax that looks like HTML but is actually JavaScript. It gets compiled to regular JavaScript function calls.
   ```jsx
   const element = <h1>Hello, world!</h1>;
   ```
   This JSX gets compiled to: `React.createElement('h1', null, 'Hello, world!')`.

3. **Components** are the building blocks of React apps. They are JavaScript functions that return JSX. Component names must start with a capital letter.
   ```jsx
   function Welcome(props) {
     return <h1>Hello, {props.name}</h1>;
   }
   ```

4. **Props** (short for properties) are how you pass data from parent components to child components. Props are read-only - child components cannot modify them.
   ```jsx
   function Greeting(props) {
     return <h1>Hello, {props.name}!</h1>;
   }
   
   // Usage
   <Greeting name="Sarah" />
   ```

5. **State** is data that can change over time in a component. When state changes, React re-renders the component automatically.
   ```jsx
   import { useState } from 'react';
   
   function Counter() {
     const [count, setCount] = useState(0);  // useState returns [currentValue, setterFunction]
     
     return (
       <div>
         <p>You clicked {count} times</p>
         <button onClick={() => setCount(count + 1)}>Click me</button>
       </div>
     );
   }
   ```

6. **Event handlers** are functions that get called when users interact with your components. Always pass functions, not function calls.
   ```jsx
   function Button() {
     const handleClick = () => {
       alert('Button clicked!');
     };
     
     return <button onClick={handleClick}>Click me</button>;  // Correct: pass function
     // return <button onClick={handleClick()}>Click me</button>;  // Wrong: calls function immediately
   }
   ```

## Component Lifecycle & Effects

7. **useEffect** is a Hook that lets you perform side effects in functional components. Side effects include data fetching, subscriptions, or manually changing the DOM.
   ```jsx
   import { useEffect, useState } from 'react';
   
   function UserProfile({ userId }) {
     const [user, setUser] = useState(null);
     
     useEffect(() => {
       // This runs after every render
       fetchUser(userId).then(setUser);
     }, [userId]);  // Dependency array: only re-run when userId changes
     
     return <div>{user ? user.name : 'Loading...'}</div>;
   }
   ```

8. **Dependency array** in useEffect controls when the effect runs:
   - No array: runs after every render
   - Empty array `[]`: runs only once after initial render
   - With dependencies `[count, name]`: runs when any dependency changes

9. **Cleanup functions** in useEffect prevent memory leaks by cleaning up subscriptions, timers, etc.
   ```jsx
   useEffect(() => {
     const timer = setInterval(() => {
       console.log('Timer tick');
     }, 1000);
     
     // Cleanup function - runs when component unmounts or before next effect
     return () => {
       clearInterval(timer);
     };
   }, []);
   ```

## Lists and Keys

10. **Rendering lists** requires using the `map()` function to transform data into JSX elements.
    ```jsx
    function TodoList({ todos }) {
      return (
        <ul>
          {todos.map(todo => (
            <li key={todo.id}>{todo.text}</li>  // key prop is required for list items
          ))}
        </ul>
      );
    }
    ```

11. **Keys** help React identify which items have changed, been added, or removed. Always use unique, stable identifiers as keys. **Never use array indexes as keys** if the list can change order.
    ```jsx
    // Good: using unique ID
    {items.map(item => <Item key={item.id} data={item} />)}
    
    // Bad: using array index (only okay if list never changes)
    {items.map((item, index) => <Item key={index} data={item} />)}
    ```

## Conditional Rendering

12. **Conditional rendering** lets you show different content based on conditions. Use `&&` for simple conditions and ternary operator `? :` for if-else logic.
    ```jsx
    function Greeting({ isLoggedIn, user }) {
      return (
        <div>
          {isLoggedIn && <p>Welcome back, {user.name}!</p>}  // Shows only if isLoggedIn is true
          {isLoggedIn ? <LogoutButton /> : <LoginButton />}   // Shows different buttons
        </div>
      );
    }
    ```

## Forms and Controlled Components

13. **Controlled components** are form elements whose values are controlled by React state. This gives you full control over the form data.
    ```jsx
    function ContactForm() {
      const [email, setEmail] = useState('');
      const [message, setMessage] = useState('');
      
      const handleSubmit = (e) => {
        e.preventDefault();  // Prevents page refresh
        console.log('Email:', email, 'Message:', message);
      };
      
      return (
        <form onSubmit={handleSubmit}>
          <input 
            type="email" 
            value={email}                           // Controlled by state
            onChange={(e) => setEmail(e.target.value)}  // Updates state on change
            placeholder="Your email"
          />
          <textarea 
            value={message}
            onChange={(e) => setMessage(e.target.value)}
            placeholder="Your message"
          />
          <button type="submit">Send</button>
        </form>
      );
    }
    ```

14. **e.preventDefault()** stops the default browser behavior. For forms, it prevents the page from refreshing when submitted.

## State Management Patterns

15. **Lifting state up** means moving state to the closest common parent when multiple components need to share the same data.
    ```jsx
    function App() {
      const [user, setUser] = useState(null);  // State lifted to parent
      
      return (
        <div>
          <Header user={user} />      // Both components receive user as props
          <Profile user={user} />
        </div>
      );
    }
    ```

16. **State updates are asynchronous** and React batches them for performance. When updating state based on previous state, use the functional form.
    ```jsx
    // Wrong: may not work as expected
    setCount(count + 1);
    setCount(count + 1);  // Both might use the same 'count' value
    
    // Correct: uses the latest state value
    setCount(prevCount => prevCount + 1);
    setCount(prevCount => prevCount + 1);  // Each uses the updated value
    ```

## Component Communication

17. **Props drilling** happens when you pass props through many component levels. **Context** solves this by letting you share data without passing props manually at every level.
    ```jsx
    import { createContext, useContext } from 'react';
    
    // Create context
    const UserContext = createContext();
    
    // Provider component
    function App() {
      const [user, setUser] = useState(null);
      
      return (
        <UserContext.Provider value={{ user, setUser }}>  // Provides value to all children
          <Header />
          <Main />
        </UserContext.Provider>
      );
    }
    
    // Consumer component (can be deep in the tree)
    function UserProfile() {
      const { user } = useContext(UserContext);  // Accesses context value
      return <div>{user?.name}</div>;
    }
    ```

18. **Callback props** let child components communicate back to parents by calling functions passed as props.
    ```jsx
    function Parent() {
      const [message, setMessage] = useState('');
      
      return (
        <div>
          <Child onMessageChange={setMessage} />  // Pass callback as prop
          <p>Message: {message}</p>
        </div>
      );
    }
    
    function Child({ onMessageChange }) {
      return (
        <input 
          onChange={(e) => onMessageChange(e.target.value)}  // Call parent's function
          placeholder="Type a message"
        />
      );
    }
    ```

## Performance & Optimization

19. **React.memo** prevents unnecessary re-renders by memoizing component results. Component only re-renders if its props change.
    ```jsx
    const ExpensiveComponent = React.memo(function ExpensiveComponent({ data }) {
      console.log('Rendering ExpensiveComponent');  // Only logs when data changes
      return <div>{data.title}</div>;
    });
    ```

20. **useMemo** memoizes expensive calculations so they only run when dependencies change.
    ```jsx
    function ProductList({ products, filter }) {
      const filteredProducts = useMemo(() => {
        console.log('Filtering products...');  // Only runs when products or filter changes
        return products.filter(product => product.category === filter);
      }, [products, filter]);  // Dependencies
      
      return (
        <div>
          {filteredProducts.map(product => <Product key={product.id} {...product} />)}
        </div>
      );
    }
    ```

21. **useCallback** memoizes functions to prevent child components from re-rendering unnecessarily.
    ```jsx
    function Parent({ items }) {
      const [count, setCount] = useState(0);
      
      // Without useCallback, this function is recreated on every render
      const handleItemClick = useCallback((itemId) => {
        console.log('Item clicked:', itemId);
      }, []);  // No dependencies, so function never changes
      
      return (
        <div>
          <button onClick={() => setCount(count + 1)}>Count: {count}</button>
          {items.map(item => (
            <ExpensiveChild key={item.id} item={item} onClick={handleItemClick} />
          ))}
        </div>
      );
    }
    ```

## Custom Hooks

22. **Custom Hooks** are functions that start with "use" and let you reuse stateful logic between components. They're just regular functions that can call other Hooks.
    ```jsx
    // Custom Hook for handling localStorage
    function useLocalStorage(key, initialValue) {
      const [storedValue, setStoredValue] = useState(() => {
        try {
          const item = window.localStorage.getItem(key);
          return item ? JSON.parse(item) : initialValue;
        } catch (error) {
          return initialValue;
        }
      });
      
      const setValue = (value) => {
        try {
          setStoredValue(value);
          window.localStorage.setItem(key, JSON.stringify(value));
        } catch (error) {
          console.error('Error saving to localStorage:', error);
        }
      };
      
      return [storedValue, setValue];
    }
    
    // Usage in component
    function Settings() {
      const [theme, setTheme] = useLocalStorage('theme', 'light');
      
      return (
        <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
          Current theme: {theme}
        </button>
      );
    }
    ```

## Error Handling

23. **Error Boundaries** are React components that catch JavaScript errors anywhere in their child component tree and display a fallback UI instead of crashing.
    ```jsx
    class ErrorBoundary extends React.Component {
      constructor(props) {
        super(props);
        this.state = { hasError: false };
      }
      
      static getDerivedStateFromError(error) {
        return { hasError: true };  // Update state to show fallback UI
      }
      
      componentDidCatch(error, errorInfo) {
        console.error('Error caught by boundary:', error, errorInfo);
      }
      
      render() {
        if (this.state.hasError) {
          return <h1>Something went wrong.</h1>;  // Fallback UI
        }
        
        return this.props.children;
      }
    }
    
    // Usage
    function App() {
      return (
        <ErrorBoundary>
          <MyComponent />  // If this crashes, ErrorBoundary catches it
        </ErrorBoundary>
      );
    }
    ```

## Common Patterns & Best Practices

24. **Fragment** (`<React.Fragment>` or `<>`) lets you group multiple elements without adding extra DOM nodes.
    ```jsx
    function UserInfo() {
      return (
        <>  {/* Short syntax for React.Fragment */}
          <h1>John Doe</h1>
          <p>Software Developer</p>
        </>
      );
    }
    ```

25. **Prop destructuring** makes your components cleaner by extracting props directly in the parameter list.
    ```jsx
    // Instead of this
    function Button(props) {
      return <button className={props.className} onClick={props.onClick}>{props.children}</button>;
    }
    
    // Do this
    function Button({ className, onClick, children }) {
      return <button className={className} onClick={onClick}>{children}</button>;
    }
    ```

26. **Default props** can be set using default parameter values or the defaultProps property.
    ```jsx
    // Method 1: Default parameters (recommended)
    function Button({ type = 'button', children = 'Click me' }) {
      return <button type={type}>{children}</button>;
    }
    
    // Method 2: defaultProps (older approach)
    Button.defaultProps = {
      type: 'button',
      children: 'Click me'
    };
    ```

## HTTP Requests & Data Fetching

27. **Fetch API** is the modern way to make HTTP requests. Always handle both success and error cases.
    ```jsx
    function UserList() {
      const [users, setUsers] = useState([]);
      const [loading, setLoading] = useState(true);
      const [error, setError] = useState(null);
      
      useEffect(() => {
        async function fetchUsers() {
          try {
            const response = await fetch('/api/users');
            if (!response.ok) {
              throw new Error(`HTTP error! status: ${response.status}`);
            }
            const userData = await response.json();
            setUsers(userData);
          } catch (err) {
            setError(err.message);
          } finally {
            setLoading(false);
          }
        }
        
        fetchUsers();
      }, []);
      
      if (loading) return <div>Loading...</div>;
      if (error) return <div>Error: {error}</div>;
      
      return (
        <ul>
          {users.map(user => <li key={user.id}>{user.name}</li>)}
        </ul>
      );
    }
    ```

28. **POST requests** require setting headers and body. Always use `JSON.stringify()` for JSON data.
    ```jsx
    const createUser = async (userData) => {
      try {
        const response = await fetch('/api/users', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',  // Important: tells server you're sending JSON
          },
          body: JSON.stringify(userData)  // Convert object to JSON string
        });
        
        if (!response.ok) {
          throw new Error('Failed to create user');
        }
        
        return await response.json();
      } catch (error) {
        console.error('Error creating user:', error);
        throw error;
      }
    };
    ```

## React Router (Navigation)

29. **React Router** enables navigation in single-page applications. Install with `npm install react-router-dom`.
    ```jsx
    import { BrowserRouter, Routes, Route, Link, useNavigate } from 'react-router-dom';
    
    function App() {
      return (
        <BrowserRouter>  {/* Enables routing for the entire app */}
          <nav>
            <Link to="/">Home</Link>  {/* Link components create navigation links */}
            <Link to="/about">About</Link>
            <Link to="/users">Users</Link>
          </nav>
          
          <Routes>  {/* Container for all route definitions */}
            <Route path="/" element={<Home />} />  {/* Route matches URL path */}
            <Route path="/about" element={<About />} />
            <Route path="/users/:id" element={<UserProfile />} />  {/* :id is a URL parameter */}
          </Routes>
        </BrowserRouter>
      );
    }
    ```

30. **useNavigate** hook provides programmatic navigation (navigating from JavaScript code instead of user clicks).
    ```jsx
    function LoginForm() {
      const navigate = useNavigate();  // Hook for programmatic navigation
      
      const handleSubmit = async (formData) => {
        const success = await login(formData);
        if (success) {
          navigate('/dashboard');  // Redirect after successful login
        }
      };
      
      return <form onSubmit={handleSubmit}>...</form>;
    }
    ```

31. **useParams** hook extracts URL parameters from the current route.
    ```jsx
    function UserProfile() {
      const { id } = useParams();  // Extracts :id from URL like /users/123
      const [user, setUser] = useState(null);
      
      useEffect(() => {
        fetchUser(id).then(setUser);  // Use the ID to fetch user data
      }, [id]);
      
      return <div>{user ? user.name : 'Loading...'}</div>;
    }
    ```

## Common Mistakes & Solutions

32. **Not wrapping multiple elements**: Components must return a single parent element or Fragment.
    ```jsx
    // Wrong: Multiple elements without wrapper
    function MyComponent() {
      return (
        <h1>Title</h1>
        <p>Content</p>  // Error: Adjacent JSX elements must be wrapped
      );
    }
    
    // Right: Wrapped in Fragment
    function MyComponent() {
      return (
        <>
          <h1>Title</h1>
          <p>Content</p>
        </>
      );
    }
    ```

33. **Forgetting `key` prop in lists**: React needs keys to efficiently update lists.
    ```jsx
    // Wrong: Missing key prop
    {items.map(item => <Item data={item} />)}
    
    // Right: With key prop
    {items.map(item => <Item key={item.id} data={item} />)}
    ```

34. **Mutating state directly**: Always create new objects/arrays instead of modifying existing ones.
    ```jsx
    // Wrong: Mutating state
    const addItem = () => {
      items.push(newItem);  // Modifies existing array
      setItems(items);
    };
    
    // Right: Creating new array
    const addItem = () => {
      setItems([...items, newItem]);  // Spread operator creates new array
    };
    
    // For objects
    const updateUser = () => {
      setUser({ ...user, name: 'New Name' });  // Spread creates new object
    };
    ```

35. **Using stale state in event handlers**: When state updates depend on previous state, use the functional form.
    ```jsx
    // Wrong: May use stale state
    const increment = () => {
      setCount(count + 1);
    };
    
    // Right: Always uses latest state
    const increment = () => {
      setCount(prevCount => prevCount + 1);
    };
    ```

36. **Not handling loading and error states**: Always show appropriate UI for different states.
    ```jsx
    function DataComponent() {
      const [data, setData] = useState(null);
      const [loading, setLoading] = useState(true);
      const [error, setError] = useState(null);
      
      // Always handle all three states: loading, error, success
      if (loading) return <div>Loading...</div>;
      if (error) return <div>Error: {error}</div>;
      if (!data) return <div>No data found</div>;
      
      return <div>{data.title}</div>;
    }
    ```

## Redux State Management

37. **Redux** is a predictable state container for JavaScript apps. It helps you manage application state in a single place, making it easier to debug and test. Think of it as a global state that any component can access.

38. **Store** is the single source of truth that holds your entire application state. You create it once and it contains all the data your app needs.
    ```jsx
    import { createStore } from 'redux';
    
    // Simple store creation
    const store = createStore(rootReducer);
    
    // Get current state
    const currentState = store.getState();
    ```

39. **Actions** are plain JavaScript objects that describe what happened. They must have a `type` property and can contain additional data (`payload`).
    ```jsx
    // Action types (constants to avoid typos)
    const ADD_TODO = 'ADD_TODO';
    const TOGGLE_TODO = 'TOGGLE_TODO';
    
    // Action creators (functions that return actions)
    const addTodo = (text) => ({
      type: ADD_TODO,
      payload: {
        id: Date.now(),
        text: text,
        completed: false
      }
    });
    
    const toggleTodo = (id) => ({
      type: TOGGLE_TODO,
      payload: { id }
    });
    ```

40. **Reducers** are pure functions that take the current state and an action, then return a new state. They specify how the application's state changes in response to actions.
    ```jsx
    const initialState = {
      todos: [],
      filter: 'ALL'
    };
    
    function todoReducer(state = initialState, action) {
      switch (action.type) {
        case ADD_TODO:
          return {
            ...state,  // Spread existing state
            todos: [...state.todos, action.payload]  // Add new todo without mutating
          };
        
        case TOGGLE_TODO:
          return {
            ...state,
            todos: state.todos.map(todo =>
              todo.id === action.payload.id
                ? { ...todo, completed: !todo.completed }  // Toggle this todo
                : todo  // Keep other todos unchanged
            )
          };
        
        default:
          return state;  // Always return current state for unknown actions
      }
    }
    ```

41. **Dispatching actions** is how you trigger state changes. You call `store.dispatch(action)` to send an action to the reducer.
    ```jsx
    // Dispatch actions to change state
    store.dispatch(addTodo('Learn Redux'));
    store.dispatch(addTodo('Build an app'));
    store.dispatch(toggleTodo(1));
    
    console.log(store.getState());  // See the updated state
    ```

42. **Connecting React to Redux** requires wrapping your app with a `Provider` component that makes the store available to all components.
    ```jsx
    import { Provider } from 'react-redux';
    import { createStore } from 'redux';
    
    const store = createStore(todoReducer);
    
    function App() {
      return (
        <Provider store={store}>  {/* Makes store available to all child components */}
          <TodoApp />
        </Provider>
      );
    }
    ```

43. **useSelector** hook lets you extract data from the Redux store state in functional components.
    ```jsx
    import { useSelector } from 'react-redux';
    
    function TodoList() {
      // useSelector takes a function that receives state and returns the data you need
      const todos = useSelector(state => state.todos);
      const filter = useSelector(state => state.filter);
      
      // Component re-renders automatically when selected state changes
      return (
        <ul>
          {todos.map(todo => (
            <li key={todo.id}>{todo.text}</li>
          ))}
        </ul>
      );
    }
    ```

44. **useDispatch** hook gives you access to the dispatch function so you can dispatch actions from components.
    ```jsx
    import { useDispatch } from 'react-redux';
    
    function AddTodo() {
      const dispatch = useDispatch();  // Get dispatch function
      const [text, setText] = useState('');
      
      const handleSubmit = (e) => {
        e.preventDefault();
        dispatch(addTodo(text));  // Dispatch action to Redux store
        setText('');
      };
      
      return (
        <form onSubmit={handleSubmit}>
          <input value={text} onChange={(e) => setText(e.target.value)} />
          <button type="submit">Add Todo</button>
        </form>
      );
    }
    ```

## Redux Toolkit (Modern Redux)

45. **Redux Toolkit (RTK)** is the official, recommended way to write Redux logic. It simplifies Redux code and includes useful utilities. Install with `npm install @reduxjs/toolkit react-redux`.

## Understanding NPM Package Names

**What does the @ symbol mean?**

The `@` symbol in npm package names indicates a **scoped package**. Scopes are like namespaces that group related packages together under an organization or user name.

```bash
npm install @reduxjs/toolkit    # Scoped package
npm install redux               # Regular package
npm install @material-ui/core   # Another scoped package
```

**Breaking down `@reduxjs/toolkit`:**
- `@reduxjs` = the scope (organization name)
- `/toolkit` = the specific package name within that scope

**Benefits of scoped packages:**
- **Organization**: All Redux-related packages are under `@reduxjs/`
- **Avoid naming conflicts**: Multiple organizations can have packages with similar names
- **Official packages**: Scoped packages often indicate official/trusted packages

**What does "toolkit" mean?**

A **toolkit** in programming is a collection of tools and utilities designed to work together for a specific purpose. It's like a toolbox with everything you need.

```jsx
// Redux Toolkit includes:
// - configureStore (store setup)
// - createSlice (actions + reducers)
// - createAsyncThunk (async operations)
// - Immer (immutable updates)
// - Redux DevTools integration
// - And more utilities

import { configureStore, createSlice, createAsyncThunk } from '@reduxjs/toolkit';
```

**Other common toolkit examples:**
- `@testing-library/react` - Testing utilities
- `@mui/material` - Material-UI component library
- `@angular/cli` - Angular command-line tools

46. **configureStore** replaces `createStore` and comes with good defaults including Redux DevTools support. It requires an **object** as parameter, not the reducer function directly.

    ```jsx
    import { configureStore } from '@reduxjs/toolkit';
    
    // Method 1: Single reducer (for simple apps)
    const store = configureStore({
      reducer: todoSlice.reducer  // Pass reducer function directly
    });
    
    // Method 2: Multiple reducers (most common)
    const store = configureStore({
      reducer: {
        todos: todoSlice.reducer,  // Each slice manages part of the state
        user: userSlice.reducer,   // State will be: { todos: {...}, user: {...} }
        posts: postSlice.reducer
      }
    });
    
    // configureStore expects an object with:
    // - reducer: (required) single reducer or object of reducers
    // - middleware: (optional) custom middleware
    // - devTools: (optional) enable/disable Redux DevTools
    // - preloadedState: (optional) initial state
    ```

**What does "slice" mean?**

A **slice** in Redux is a portion of your application state along with the logic to update it. Think of it like cutting a pizza into slices - each slice handles one specific part of your app's data.

```jsx
// This slice handles only todo-related state and actions
const todoSlice = createSlice({
  name: 'todos',           // This slice's name
  initialState: {          // This slice's initial state
    items: [],
    loading: false
  },
  reducers: {              // This slice's actions and how they update state
    addTodo: (state, action) => { /* ... */ },
    removeTodo: (state, action) => { /* ... */ }
  }
});

// Your complete app state might look like:
// {
//   todos: { items: [], loading: false },    // todoSlice manages this part
//   user: { name: '', email: '' },           // userSlice manages this part  
//   posts: { list: [], currentPost: null }   // postSlice manages this part
// }
```

**Why use slices?**
- **Organization**: Each slice handles one feature (todos, users, posts, etc.)
- **Maintainability**: Easier to find and update specific functionality
- **Reusability**: Slices can be used in different parts of your app
- **Auto-generation**: createSlice automatically creates action creators and action types

## Complete Redux Toolkit Implementation Example

**Where is the initial state structure defined?**

Yes, the **initial state structure is defined inside each slice**. Each slice declares its own piece of the state structure.

```jsx
// todoSlice.js - Define todo state structure
const todoSlice = createSlice({
  name: 'todos',
  initialState: {          // Initial state structure defined HERE
    items: [],             // Array of todo items
    loading: false,        // Loading indicator
    filter: 'all',         // Current filter
    error: null            // Error message
  },
  reducers: { /* ... */ }
});

// userSlice.js - Define user state structure  
const userSlice = createSlice({
  name: 'user',
  initialState: {          // User state structure defined HERE
    profile: null,         // User profile data
    isLoggedIn: false,     // Authentication status
    preferences: {         // Nested object
      theme: 'light',
      notifications: true
    }
  },
  reducers: { /* ... */ }
});

// Final combined state will be:
// {
//   todos: { items: [], loading: false, filter: 'all', error: null },
//   user: { profile: null, isLoggedIn: false, preferences: {...} }
// }
```

**Complete working example with slices, store, and components:**

```jsx
// store/todoSlice.js
import { createSlice } from '@reduxjs/toolkit';

const todoSlice = createSlice({
  name: 'todos',
  initialState: {
    items: [],
    loading: false,
    filter: 'all'  // 'all', 'completed', 'pending'
  },
  reducers: {
    addTodo: (state, action) => {
      state.items.push({
        id: Date.now(),
        text: action.payload,
        completed: false
      });
    },
    toggleTodo: (state, action) => {
      const todo = state.items.find(todo => todo.id === action.payload);
      if (todo) {
        todo.completed = !todo.completed;
      }
    },
    deleteTodo: (state, action) => {
      state.items = state.items.filter(todo => todo.id !== action.payload);
    },
    setFilter: (state, action) => {
      state.filter = action.payload;
    },
    setLoading: (state, action) => {
      state.loading = action.payload;
    }
  }
});

// Export actions (auto-generated by createSlice)
export const { addTodo, toggleTodo, deleteTodo, setFilter, setLoading } = todoSlice.actions;

// Export reducer
export default todoSlice.reducer;
```

```jsx
// store/store.js
import { configureStore } from '@reduxjs/toolkit';
import todoReducer from './todoSlice';

export const store = configureStore({
  reducer: {
    todos: todoReducer  // todos slice will be accessible as state.todos
  }
});
```

```jsx
// App.js - Setup Provider
import { Provider } from 'react-redux';
import { store } from './store/store';
import TodoApp from './TodoApp';

function App() {
  return (
    <Provider store={store}>
      <TodoApp />
    </Provider>
  );
}
```

**Using useSelector and useDispatch in components:**

```jsx
// components/TodoApp.js
import React from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { addTodo, setFilter } from '../store/todoSlice';

function TodoApp() {
  const dispatch = useDispatch();
  
  // useSelector - Component ONLY re-renders when these specific values change
  const { items, loading, filter } = useSelector(state => state.todos);
  const totalTodos = useSelector(state => state.todos.items.length);
  const completedCount = useSelector(state => 
    state.todos.items.filter(todo => todo.completed).length
  );
  
  const handleAddTodo = (text) => {
    dispatch(addTodo(text));  // Dispatch action to add todo
  };
  
  const handleFilterChange = (newFilter) => {
    dispatch(setFilter(newFilter));  // Dispatch action to change filter
  };
  
  // Filter todos based on current filter
  const filteredTodos = items.filter(todo => {
    if (filter === 'completed') return todo.completed;
    if (filter === 'pending') return !todo.completed;
    return true; // 'all'
  });
  
  return (
    <div>
      <h1>Todo App</h1>
      <p>Total: {totalTodos}, Completed: {completedCount}</p>
      
      <AddTodoForm onAddTodo={handleAddTodo} />
      
      <FilterButtons currentFilter={filter} onFilterChange={handleFilterChange} />
      
      {loading ? (
        <p>Loading...</p>
      ) : (
        <TodoList todos={filteredTodos} />
      )}
    </div>
  );
}
```

```jsx
// components/AddTodoForm.js
import React, { useState } from 'react';

function AddTodoForm({ onAddTodo }) {
  const [text, setText] = useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault();
    if (text.trim()) {
      onAddTodo(text.trim());  // Call parent's function which dispatches action
      setText('');
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={text}
        onChange={(e) => setText(e.target.value)}
        placeholder="Add new todo..."
      />
      <button type="submit">Add Todo</button>
    </form>
  );
}
```

```jsx
// components/TodoList.js
import React from 'react';
import { useDispatch } from 'react-redux';
import { toggleTodo, deleteTodo } from '../store/todoSlice';

function TodoList({ todos }) {
  const dispatch = useDispatch();
  
  return (
    <ul>
      {todos.map(todo => (
        <TodoItem 
          key={todo.id} 
          todo={todo}
          onToggle={() => dispatch(toggleTodo(todo.id))}    // Dispatch toggle action
          onDelete={() => dispatch(deleteTodo(todo.id))}    // Dispatch delete action
        />
      ))}
    </ul>
  );
}

function TodoItem({ todo, onToggle, onDelete }) {
  return (
    <li style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={onToggle}  // Calls parent's onToggle which dispatches action
      />
      <span>{todo.text}</span>
      <button onClick={onDelete}>Delete</button>
    </li>
  );
}
```

**useSelector Re-rendering Optimization:**

**Yes, you are absolutely correct!** Components using `useSelector` only re-render when the **specific part of state they select actually changes**, not on every state change.

```jsx
function TodoStats() {
  // This component ONLY re-renders when the todos array changes
  const todoCount = useSelector(state => state.todos.items.length);
  
  // This component will NOT re-render if:
  // - User profile changes (state.user.profile)
  // - Todo loading state changes (state.todos.loading)
  // - Todo filter changes (state.todos.filter)
  // 
  // It ONLY re-renders when state.todos.items.length actually changes
  
  console.log('TodoStats component re-rendered'); // Only logs when todoCount changes
  
  return <p>Total todos: {todoCount}</p>;
}

function UserProfile() {
  // This component ONLY re-renders when user data changes
  const user = useSelector(state => state.user.profile);
  
  // This will NOT re-render when todos are added/removed/toggled
  // because it doesn't select any todo-related state
  
  console.log('UserProfile component re-rendered'); // Only logs when user changes
  
  return <div>{user?.name || 'Not logged in'}</div>;
}
```

**Key takeaways:**
- Initial state structure is defined in each slice's `initialState`
- useSelector provides automatic optimization - only re-renders when selected data changes
- Use useDispatch to trigger actions from components
- Each slice manages its own piece of state independently

## Spread Operator for Complex Nested State Updates

When working with complex nested objects in React state, you must create new objects instead of mutating existing ones. The **spread operator (`...`)** helps you do this correctly.

**Simple object update:**
```jsx
function UserProfile() {
  const [user, setUser] = useState({
    name: 'John',
    email: 'john@example.com',
    age: 30
  });
  
  const updateName = (newName) => {
    setUser({
      ...user,        // Spread existing properties
      name: newName   // Override specific property
    });
  };
}
```

**Nested object update (2 levels deep):**
```jsx
function UserSettings() {
  const [user, setUser] = useState({
    profile: {
      name: 'John',
      email: 'john@example.com'
    },
    preferences: {
      theme: 'light',
      notifications: true
    }
  });
  
  const updateEmail = (newEmail) => {
    setUser({
      ...user,              // Spread top level
      profile: {
        ...user.profile,    // Spread nested level
        email: newEmail     // Update specific nested property
      }
    });
  };
  
  const updateTheme = (newTheme) => {
    setUser({
      ...user,                    // Spread top level
      preferences: {
        ...user.preferences,      // Spread nested level
        theme: newTheme           // Update specific nested property
      }
    });
  };
}
```

**Deeply nested object update (3+ levels deep):**
```jsx
function CompanyData() {
  const [company, setCompany] = useState({
    info: {
      name: 'Tech Corp',
      address: {
        street: '123 Main St',
        city: 'New York',
        country: {
          name: 'USA',
          code: 'US'
        }
      }
    },
    employees: [
      { id: 1, name: 'John', department: { name: 'Engineering', budget: 50000 } }
    ]
  });
  
  // Update deeply nested country name
  const updateCountryName = (newCountryName) => {
    setCompany({
      ...company,                           // Level 1: Spread company
      info: {
        ...company.info,                    // Level 2: Spread info
        address: {
          ...company.info.address,          // Level 3: Spread address
          country: {
            ...company.info.address.country, // Level 4: Spread country
            name: newCountryName             // Finally update the property
          }
        }
      }
    });
  };
  
  // Update employee department budget
  const updateEmployeeBudget = (employeeId, newBudget) => {
    setCompany({
      ...company,
      employees: company.employees.map(employee =>
        employee.id === employeeId
          ? {
              ...employee,                    // Spread employee
              department: {
                ...employee.department,       // Spread department
                budget: newBudget            // Update budget
              }
            }
          : employee                         // Keep other employees unchanged
      )
    });
  };
}
```

**Arrays with nested objects:**
```jsx
function TodoApp() {
  const [state, setState] = useState({
    todos: [
      {
        id: 1,
        text: 'Learn React',
        completed: false,
        tags: ['programming', 'frontend'],
        metadata: {
          priority: 'high',
          createdBy: { name: 'John', id: 123 }
        }
      }
    ],
    filter: 'all'
  });
  
  // Add new todo
  const addTodo = (newTodo) => {
    setState({
      ...state,
      todos: [...state.todos, newTodo]  // Spread array to create new array
    });
  };
  
  // Update specific todo's nested property
  const updateTodoPriority = (todoId, newPriority) => {
    setState({
      ...state,
      todos: state.todos.map(todo =>
        todo.id === todoId
          ? {
              ...todo,                        // Spread todo
              metadata: {
                ...todo.metadata,             // Spread metadata
                priority: newPriority         // Update priority
              }
            }
          : todo                             // Keep other todos unchanged
      )
    });
  };
  
  // Add tag to specific todo
  const addTagToTodo = (todoId, newTag) => {
    setState({
      ...state,
      todos: state.todos.map(todo =>
        todo.id === todoId
          ? {
              ...todo,
              tags: [...todo.tags, newTag]   // Spread array to add new tag
            }
          : todo
      )
    });
  };
  
  // Update creator name for specific todo
  const updateTodoCreator = (todoId, newCreatorName) => {
    setState({
      ...state,
      todos: state.todos.map(todo =>
        todo.id === todoId
          ? {
              ...todo,
              metadata: {
                ...todo.metadata,
                createdBy: {
                  ...todo.metadata.createdBy,  // Spread nested object
                  name: newCreatorName          // Update name
                }
              }
            }
          : todo
      )
    });
  };
}
```

**Helper function for deep updates:**
```jsx
// Utility function to make deep updates easier
function updateNestedProperty(obj, path, value) {
  const keys = path.split('.');
  const lastKey = keys.pop();
  
  let current = { ...obj };
  let pointer = current;
  
  // Create nested structure
  for (const key of keys) {
    pointer[key] = { ...pointer[key] };
    pointer = pointer[key];
  }
  
  // Set the final value
  pointer[lastKey] = value;
  
  return current;
}

// Usage example
function UserProfile() {
  const [user, setUser] = useState({
    profile: {
      personal: {
        name: 'John',
        address: {
          city: 'New York'
        }
      }
    }
  });
  
  const updateCity = (newCity) => {
    const updatedUser = updateNestedProperty(user, 'profile.personal.address.city', newCity);
    setUser(updatedUser);
  };
}
```

**Common patterns for complex updates:**

1. **Multiple updates at once:**
```jsx
const updateMultipleFields = () => {
  setUser({
    ...user,
    profile: {
      ...user.profile,
      name: 'New Name',        // Update multiple fields
      email: 'new@email.com'   // in the same nested object
    },
    preferences: {
      ...user.preferences,
      theme: 'dark'           // Also update different nested object
    }
  });
};
```

2. **Conditional nested updates:**
```jsx
const conditionalUpdate = (shouldUpdateEmail) => {
  setUser({
    ...user,
    profile: {
      ...user.profile,
      name: 'Updated Name',
      // Conditionally include email update
      ...(shouldUpdateEmail && { email: 'new@example.com' })
    }
  });
};
```

3. **Removing nested properties:**
```jsx
const removeNestedProperty = () => {
  const { propertyToRemove, ...restOfProfile } = user.profile;
  
  setUser({
    ...user,
    profile: restOfProfile  // Profile without the removed property
  });
};
```

**Why Redux Toolkit is easier:**
```jsx
// With Redux Toolkit + Immer, you can write "mutative" logic
const userSlice = createSlice({
  name: 'user',
  initialState: { /* complex nested state */ },
  reducers: {
    updateNestedProperty: (state, action) => {
      // This looks like mutation but is actually safe
      state.profile.personal.address.city = action.payload;
      // Immer handles creating new objects behind the scenes
    }
  }
});

// Instead of complex spread operations in regular React:
setUser({
  ...user,
  profile: {
    ...user.profile,
    personal: {
      ...user.profile.personal,
      address: {
        ...user.profile.personal.address,
        city: newCity
      }
    }
  }
});
```

**Best practices for nested updates:**
- **Keep state shallow when possible** - avoid deep nesting
- **Use helper functions** for complex updates
- **Consider using libraries** like Immer for complex state
- **Break down large objects** into smaller pieces
- **Use Redux Toolkit** for complex state management

## React Performance Optimization

55. **React.memo** prevents functional components from re-rendering if their props haven't changed. Use it for components that receive the same props frequently.
    ```jsx
    // Without memo - re-renders every time parent renders
    function ExpensiveChild({ name, count }) {
      console.log('ExpensiveChild rendered');
      return <div>{name}: {count}</div>;
    }
    
    // With memo - only re-renders when name or count changes
    const OptimizedChild = React.memo(function ExpensiveChild({ name, count }) {
      console.log('OptimizedChild rendered');
      return <div>{name}: {count}</div>;
    });
    
    // Custom comparison function (optional)
    const SmartChild = React.memo(function SmartChild({ user }) {
      return <div>{user.name}</div>;
    }, (prevProps, nextProps) => {
      // Return true if props are equal (skip re-render)
      return prevProps.user.name === nextProps.user.name;
    });
    ```

56. **useMemo** memoizes expensive calculations so they only run when dependencies change.
    ```jsx
    function ProductList({ products, searchTerm, sortBy }) {
      // Expensive filtering and sorting - only runs when dependencies change
      const processedProducts = useMemo(() => {
        console.log('Processing products...');
        return products
          .filter(product => product.name.toLowerCase().includes(searchTerm.toLowerCase()))
          .sort((a, b) => a[sortBy] > b[sortBy] ? 1 : -1);
      }, [products, searchTerm, sortBy]);  // Dependencies
      
      return (
        <div>
          {processedProducts.map(product => (
            <ProductItem key={product.id} product={product} />
          ))}
        </div>
      );
    }
    ```

57. **useCallback** memoizes functions to prevent child components from re-rendering unnecessarily.
    ```jsx
    function TodoApp({ todos }) {
      const [filter, setFilter] = useState('all');
      
      // Without useCallback - new function created on every render
      const handleDelete = (id) => {
        // delete logic
      };
      
      // With useCallback - same function reference when dependencies don't change
      const handleDeleteOptimized = useCallback((id) => {
        // delete logic
      }, []);  // No dependencies, function never changes
      
      const handleToggle = useCallback((id) => {
        // toggle logic that depends on current todos
      }, [todos]);  // Function changes only when todos change
      
      return (
        <div>
          {todos.map(todo => (
            <TodoItem 
              key={todo.id} 
              todo={todo}
              onDelete={handleDeleteOptimized}  // Stable function reference
              onToggle={handleToggle}
            />
          ))}
        </div>
      );
    }
    ```

58. **Lazy loading components** with `React.lazy()` and `Suspense` to split code and load components only when needed.
    ```jsx
    import { lazy, Suspense } from 'react';
    
    // Lazy load heavy components
    const HeavyChart = lazy(() => import('./components/HeavyChart'));
    const UserDashboard = lazy(() => import('./components/UserDashboard'));
    
    function App() {
      const [showChart, setShowChart] = useState(false);
      
      return (
        <div>
          <h1>My App</h1>
          
          {showChart && (
            <Suspense fallback={<div>Loading chart...</div>}>
              <HeavyChart />  {/* Only loads when showChart is true */}
            </Suspense>
          )}
          
          <Suspense fallback={<div>Loading dashboard...</div>}>
            <UserDashboard />
          </Suspense>
        </div>
      );
    }
    ```

59. **Virtualization** for large lists to render only visible items instead of all items.
    ```jsx
    import { FixedSizeList as List } from 'react-window';
    
    function VirtualizedList({ items }) {
      // Row component for each item
      const Row = ({ index, style }) => (
        <div style={style}>
          <div>Item {items[index].name}</div>
        </div>
      );
      
      return (
        <List
          height={600}        // Container height
          itemCount={items.length}  // Total number of items
          itemSize={50}       // Height of each item
          width="100%"
        >
          {Row}
        </List>
      );
    }
    
    // For 10,000 items, only ~12 DOM elements are rendered (visible ones)
    // Instead of 10,000 DOM elements
    ```

60. **Debouncing expensive operations** like API calls or complex calculations.
    ```jsx
    import { useState, useEffect, useMemo } from 'react';
    
    function useDebounce(value, delay) {
      const [debouncedValue, setDebouncedValue] = useState(value);
      
      useEffect(() => {
        const handler = setTimeout(() => {
          setDebouncedValue(value);
        }, delay);
        
        return () => clearTimeout(handler);  // Cleanup timeout
      }, [value, delay]);
      
      return debouncedValue;
    }
    
    function SearchBox() {
      const [searchTerm, setSearchTerm] = useState('');
      const debouncedSearchTerm = useDebounce(searchTerm, 300);  // 300ms delay
      
      // Expensive search only runs after user stops typing for 300ms
      const searchResults = useMemo(() => {
        if (!debouncedSearchTerm) return [];
        
        console.log('Performing expensive search...');
        return performExpensiveSearch(debouncedSearchTerm);
      }, [debouncedSearchTerm]);
      
      return (
        <div>
          <input
            value={searchTerm}
            onChange={(e) => setSearchTerm(e.target.value)}
            placeholder="Search..."
          />
          {searchResults.map(result => (
            <div key={result.id}>{result.title}</div>
          ))}
        </div>
      );
    }
    ```

61. **Optimizing state structure** to minimize re-renders by keeping related data together and separating unrelated data.
    ```jsx
    // Bad: Causes unnecessary re-renders
    function BadExample() {
      const [user, setUser] = useState({ name: '', email: '' });
      const [todos, setTodos] = useState([]);
      const [uiState, setUiState] = useState({ loading: false, modal: false });
      
      // Changing modal state re-renders entire component
      const toggleModal = () => setUiState({ ...uiState, modal: !uiState.modal });
    }
    
    // Good: Separate concerns, minimal re-renders
    function GoodExample() {
      const [user, setUser] = useState({ name: '', email: '' });
      const [todos, setTodos] = useState([]);
      const [loading, setLoading] = useState(false);
      const [showModal, setShowModal] = useState(false);
      
      // Only modal-related components re-render
      const toggleModal = () => setShowModal(!showModal);
    }
    ```

62. **Using keys effectively** in lists to help React identify which items have changed.
    ```jsx
    function TodoList({ todos }) {
      return (
        <div>
          {/* Bad: Using array index as key */}
          {todos.map((todo, index) => (
            <TodoItem key={index} todo={todo} />  // Problems when list order changes
          ))}
          
          {/* Good: Using unique, stable ID */}
          {todos.map(todo => (
            <TodoItem key={todo.id} todo={todo} />  // React can track items correctly
          ))}
          
          {/* Good: Composite key when no ID available */}
          {todos.map(todo => (
            <TodoItem key={`${todo.text}-${todo.createdAt}`} todo={todo} />
          ))}
        </div>
      );
    }
    ```

63. **Code splitting at route level** to load only the code needed for each page.
    ```jsx
    import { lazy, Suspense } from 'react';
    import { BrowserRouter, Routes, Route } from 'react-router-dom';
    
    // Lazy load route components
    const Home = lazy(() => import('./pages/Home'));
    const About = lazy(() => import('./pages/About'));
    const Dashboard = lazy(() => import('./pages/Dashboard'));
    
    function App() {
      return (
        <BrowserRouter>
          <Suspense fallback={<div>Loading page...</div>}>
            <Routes>
              <Route path="/" element={<Home />} />
              <Route path="/about" element={<About />} />
              <Route path="/dashboard" element={<Dashboard />} />
            </Routes>
          </Suspense>
        </BrowserRouter>
      );
    }
    ```

64. **Avoiding object and function creation in render** to prevent unnecessary re-renders.
    ```jsx
    function ProductList({ products }) {
      // Bad: Creates new object/function on every render
      const badStyle = { marginTop: 10 };  // New object every time
      const badHandler = () => console.log('clicked');  // New function every time
      
      return (
        <div>
          {products.map(product => (
            <div 
              key={product.id}
              style={badStyle}           // Causes re-render
              onClick={badHandler}       // Causes re-render
            >
              {product.name}
            </div>
          ))}
        </div>
      );
    }
    
    // Good: Define outside component or use constants
    const PRODUCT_STYLE = { marginTop: 10 };  // Stable reference
    
    function OptimizedProductList({ products }) {
      const handleClick = useCallback(() => {
        console.log('clicked');
      }, []);  // Stable function reference
      
      return (
        <div>
          {products.map(product => (
            <div 
              key={product.id}
              style={PRODUCT_STYLE}      // Stable reference
              onClick={handleClick}      // Stable reference
            >
              {product.name}
            </div>
          ))}
        </div>
      );
    }
    ```

65. **Bundle optimization** techniques to reduce JavaScript bundle size.

## Understanding JavaScript Imports

**Named Imports vs Default Imports:**

```jsx
// Named imports - use { } to import specific exports
import { useState, useEffect } from 'react';        // Import specific functions
import { debounce, throttle } from 'lodash';        // Import multiple named exports

// Default imports - no { } for the default export  
import React from 'react';                          // Import default export
import debounce from 'lodash/debounce';            // Import default from specific file
import MyComponent from './MyComponent';            // Import default from your file
```

**How Package Imports Work:**

```jsx
// Method 1: Import from main package
import { debounce } from 'lodash';
// What happens:
// 1. Imports from lodash's main entry point (usually index.js)
// 2. Lodash's index.js exports ALL functions: { debounce, throttle, map, filter, ... }
// 3. Your bundler gets the ENTIRE lodash library (even if you only use debounce)
// 4. Result: Larger bundle size

// Method 2: Import from specific file
import debounce from 'lodash/debounce';
// What happens:
// 1. Imports directly from lodash/debounce.js file
// 2. This file ONLY contains the debounce function
// 3. Your bundler gets ONLY the debounce code
// 4. Result: Smaller bundle size
```

**Package Structure Example:**
```
lodash/
├── index.js          // Main entry point - exports everything
├── debounce.js       // Individual function file
├── throttle.js       // Individual function file
├── map.js           // Individual function file
└── package.json     // Defines "main": "index.js"
```

**Real Bundle Size Comparison:**
```jsx
// This imports ~70KB of lodash code
import { debounce } from 'lodash';

// This imports ~2KB of just debounce code  
import debounce from 'lodash/debounce';
```

**Other Import Patterns:**

```jsx
// 1. Mixed imports
import React, { useState, useEffect } from 'react';
//     ↑            ↑
//   default      named imports

// 2. Namespace imports
import * as lodash from 'lodash';         // Import everything as an object
lodash.debounce(fn, 300);                // Use with dot notation

// 3. Renaming imports
import { useState as useStateHook } from 'react';   // Rename named import
import myDebounce from 'lodash/debounce';          // Default imports can be renamed

// 4. Import with side effects only
import './styles.css';                    // Just execute the file, no imports
import 'some-polyfill';                  // Load polyfill code
```

**How Files Export Things:**

```jsx
// MyUtils.js - Different export types
export const add = (a, b) => a + b;           // Named export
export const subtract = (a, b) => a - b;      // Named export
export default function multiply(a, b) {       // Default export
  return a * b;
}

// Usage:
import multiply from './MyUtils';              // Import default (any name works)
import { add, subtract } from './MyUtils';    // Import named exports (exact names)
import times, { add, subtract } from './MyUtils'; // Both default and named
```

**Tree Shaking Explanation:**

```jsx
// Without tree shaking (old bundlers):
import { debounce } from 'lodash';
// Bundle includes: debounce + throttle + map + filter + 200+ other functions

// With tree shaking (modern bundlers):
import { debounce } from 'lodash';
// Bundle includes: only debounce (if lodash supports tree shaking)

// But lodash doesn't fully support tree shaking, so:
import debounce from 'lodash/debounce';  // This is more reliable
```

**Best Practices:**

```jsx
// ✅ Good - specific imports
import debounce from 'lodash/debounce';          // Small bundle
import { format } from 'date-fns';              // Tree-shakable library
import { Button, TextField } from '@mui/material'; // Modern libraries support this

// ❌ Bad - imports entire libraries  
import _ from 'lodash';                          // 70KB+ bundle
import * as dateFns from 'date-fns';           // Larger than needed
import MaterialUI from '@mui/material';         // Entire component library

// ✅ Good - your own modules
export const utils = { add, subtract };         // Named exports
export default function Calculator() {...}      // Default export

// Usage:
import Calculator, { utils } from './Calculator';
```

**Modern Library Examples:**

```jsx
// React (tree-shakable)
import { useState, useEffect } from 'react';    // ✅ Only imports used hooks

// Material-UI (tree-shakable)  
import { Button, TextField } from '@mui/material'; // ✅ Only imports used components

// Lodash (not tree-shakable)
import { debounce } from 'lodash';              // ❌ Imports entire library
import debounce from 'lodash/debounce';        // ✅ Imports only debounce

// Date-fns (tree-shakable)
import { format, addDays } from 'date-fns';    // ✅ Only imports used functions
```

**Dynamic Imports for Code Splitting:**

```jsx
// Static imports - loaded immediately
import HeavyComponent from './HeavyComponent';

// Dynamic imports - loaded when needed
async function loadComponent() {
  const { HeavyComponent } = await import('./HeavyComponent');  // Returns promise
  return HeavyComponent;
}

// React.lazy uses dynamic imports
const LazyComponent = lazy(() => import('./HeavyComponent'));
```

## Loading vs Importing - Key Differences

**Importing** = The syntax/declaration of what code you want to use
**Loading** = The actual process of fetching and executing that code

```jsx
// IMPORTING - This is just a declaration (happens at build time)
import React from 'react';                    // Declares dependency
import { useState } from 'react';             // Declares what you need
import MyComponent from './MyComponent';       // Declares local dependency

// LOADING - This is when code actually gets fetched/executed
```

**Static Loading (Traditional Imports):**

```jsx
// These imports load IMMEDIATELY when the module loads
import React from 'react';                    // ✅ Loaded at module startup
import lodash from 'lodash';                  // ✅ Loaded at module startup  
import './styles.css';                        // ✅ CSS loaded immediately

function App() {
  // By the time this function runs, ALL imports above are already loaded
  return <div>Hello</div>;
}

// Timeline:
// 1. Browser downloads your bundle.js
// 2. ALL imported modules load immediately 
// 3. Your component can run because dependencies are ready
```

**Dynamic Loading (Lazy Loading):**

```jsx
// These DON'T load until actually needed
const LazyComponent = lazy(() => import('./HeavyComponent'));  // ❌ Not loaded yet

function App() {
  const [showHeavy, setShowHeavy] = useState(false);
  
  return (
    <div>
      {showHeavy && (
        <Suspense fallback={<div>Loading...</div>}>
          {/* LOADING happens here when component first renders */}
          <LazyComponent />  
        </Suspense>
      )}
    </div>
  );
}

// Timeline:
// 1. Browser downloads main bundle.js (HeavyComponent NOT included)
// 2. User triggers showHeavy = true
// 3. React starts loading HeavyComponent.js (separate network request)
// 4. Suspense shows fallback while loading
// 5. Once loaded, HeavyComponent renders
```

**Build Time vs Runtime Loading:**

```jsx
// BUILD TIME LOADING (bundler processes these)
import React from 'react';                    // Bundler includes React in main bundle
import utils from './utils';                  // Bundler includes utils in main bundle

// RUNTIME LOADING (browser processes these)
const loadFeature = async () => {
  const module = await import('./feature');   // Browser makes HTTP request at runtime
  return module.default;
};

// With React.lazy
const Feature = lazy(() => import('./feature')); // Browser loads when needed
```

**Loading Strategies Comparison:**

```jsx
// 1. EAGER LOADING - Everything loads upfront
import Dashboard from './Dashboard';          // Main bundle: 500KB
import Analytics from './Analytics';          // All features included
import Reports from './Reports';              

function App() {
  return (
    <Routes>
      <Route path="/dashboard" element={<Dashboard />} />
      <Route path="/analytics" element={<Analytics />} />  
      <Route path="/reports" element={<Reports />} />
    </Routes>
  );
}
// Result: Slow initial load, fast navigation

// 2. LAZY LOADING - Load features when needed  
const Dashboard = lazy(() => import('./Dashboard'));   // Separate chunks
const Analytics = lazy(() => import('./Analytics'));   // dashboard.js: 150KB
const Reports = lazy(() => import('./Reports'));       // analytics.js: 200KB

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/analytics" element={<Analytics />} />
        <Route path="/reports" element={<Reports />} />
      </Routes>
    </Suspense>
  );
}
// Result: Fast initial load, loading states on navigation
```

**Module Loading Lifecycle:**

```jsx
// 1. MODULE PARSING - Import statements processed
import { createStore } from 'redux';          // Bundler sees this dependency
import UserService from './UserService';      // Bundler sees this dependency

// 2. DEPENDENCY RESOLUTION - Bundler finds all files
// bundler includes: redux.js + UserService.js + all their dependencies

// 3. BUNDLE CREATION - Code combined into chunks
// main.js: App + UserService + redux
// OR
// main.js: App + UserService
// vendor.js: redux (separate chunk)

// 4. RUNTIME LOADING - Browser executes
// Browser downloads main.js
// All imported modules execute immediately
// Your component code runs
```

**Conditional Loading Examples:**

```jsx
function App() {
  const [userType, setUserType] = useState(null);
  
  // Load different modules based on user type
  const loadUserInterface = async (type) => {
    if (type === 'admin') {
      const { AdminPanel } = await import('./AdminPanel');    // Load only for admins
      return AdminPanel;
    } else {
      const { UserPanel } = await import('./UserPanel');      // Load only for users  
      return UserPanel;
    }
  };
  
  // Load feature based on device
  const loadMobileFeatures = async () => {
    if (window.innerWidth < 768) {
      const { MobileNav } = await import('./MobileNav');      // Load only on mobile
      return MobileNav;
    }
    return null;
  };
}
```

**Loading Performance Patterns:**

```jsx
// PRELOADING - Load before needed
const preloadDashboard = () => {
  // Start loading but don't use yet
  import('./Dashboard');  // Loads in background
};

// Usage
function HomePage() {
  useEffect(() => {
    // Preload dashboard when user hovers on dashboard link
    const dashboardLink = document.getElementById('dashboard-link');
    dashboardLink.addEventListener('mouseenter', preloadDashboard);
  }, []);
}

// PREFETCHING - Load during idle time
// In your bundler config:
// import(/* webpackPrefetch: true */ './Dashboard')
// Browser loads this during idle time automatically
```

**Loading States in Practice:**

```jsx
function FeatureLoader() {
  const [loading, setLoading] = useState(false);
  const [Component, setComponent] = useState(null);
  const [error, setError] = useState(null);
  
  const loadFeature = async () => {
    setLoading(true);
    setError(null);
    
    try {
      // LOADING happens here
      const module = await import('./ExpensiveFeature');
      setComponent(() => module.default);  // Store loaded component
    } catch (err) {
      setError('Failed to load feature');  // Handle loading errors
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div>
      {!Component && (
        <button onClick={loadFeature}>
          {loading ? 'Loading...' : 'Load Feature'}
        </button>
      )}
      {error && <div>Error: {error}</div>}
      {Component && <Component />}
    </div>
  );
}
```

**Summary:**
- **Importing** = Declaring dependencies (syntax)
- **Loading** = Actually fetching/executing code (runtime process)
- **Static imports** = Load immediately at startup
- **Dynamic imports** = Load on-demand when needed
- **Bundle size** affects loading time
- **Code splitting** enables selective loading for better performance

66. **Image optimization** to reduce loading times and improve performance.
    ```jsx
    function OptimizedImage({ src, alt, ...props }) {
      return (
        <img
          src={src}
          alt={alt}
          loading="lazy"          // Native lazy loading
          decoding="async"        // Decode image asynchronously
          {...props}
        />
      );
    }
    
    // Using modern image formats
    function ResponsiveImage({ src, alt }) {
      return (
        <picture>
          <source srcSet={`${src}.webp`} type="image/webp" />  {/* Modern format */}
          <source srcSet={`${src}.jpg`} type="image/jpeg" />   {/* Fallback */}
          <img src={`${src}.jpg`} alt={alt} loading="lazy" />
        </picture>
      );
    }
    ```

**Performance monitoring and debugging:**
- Use **React DevTools Profiler** to identify slow components
- Use **Chrome DevTools Performance** tab to analyze rendering performance  
- Monitor **Lighthouse scores** for overall performance metrics
- Use **React.StrictMode** in development to catch performance issues early
- Consider **Web Vitals** metrics (LCP, FID, CLS) for user experience

## React Styling Techniques

67. **Centering divs** - Multiple approaches for different scenarios.

**Flexbox centering (most common):**
```jsx
function CenterWithFlex() {
  return (
    <div style={{
      display: 'flex',
      justifyContent: 'center',
      alignItems: 'center',
      height: '100vh'
    }}>
      <div>Perfectly centered content</div>
    </div>
  );
}

// CSS class version
.flex-center {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}
```

**CSS Grid centering:**
```jsx
function CenterWithGrid() {
  return (
    <div style={{
      display: 'grid',
      placeItems: 'center',
      height: '100vh'
    }}>
      <div>Grid centered content</div>
    </div>
  );
}

// Or with separate properties
.grid-center {
  display: grid;
  justify-content: center;
  align-content: center;
  height: 100vh;
}
```

**Absolute positioning centering:**
```jsx
function CenterAbsolute() {
  return (
    <div style={{ position: 'relative', height: '100vh' }}>
      <div style={{
        position: 'absolute',
        top: '50%',
        left: '50%',
        transform: 'translate(-50%, -50%)'  // Move back by half width/height
      }}>
        Absolutely centered
      </div>
    </div>
  );
}
```

**Text centering:**
```jsx
function CenterText() {
  return (
    <div style={{
      textAlign: 'center',
      lineHeight: '100vh'
    }}>
      Centered text
    </div>
  );
}
```

68. **CSS Animations** - Creating smooth animations and transitions.

**CSS Transitions (for hover effects):**
```jsx
function ButtonWithTransition() {
  const buttonStyle = {
    padding: '10px 20px',
    backgroundColor: '#007bff',
    color: 'white',
    border: 'none',
    borderRadius: '4px',
    cursor: 'pointer',
    transition: 'all 0.3s ease',
    transform: 'scale(1)'
  };
  
  const hoverStyle = {
    backgroundColor: '#0056b3',
    transform: 'scale(1.05)'
  };
  
  return (
    <button 
      style={buttonStyle}
      onMouseEnter={(e) => Object.assign(e.target.style, hoverStyle)}
      onMouseLeave={(e) => Object.assign(e.target.style, buttonStyle)}
    >
      Hover me!
    </button>
  );
}

// CSS version (recommended)
.animated-button {
  transition: all 0.3s ease;
  transform: scale(1);
}

.animated-button:hover {
  background-color: #0056b3;
  transform: scale(1.05);
}
```

**CSS Keyframe Animations:**
```jsx
function AnimatedLoader() {
  const spinnerStyle = {
    width: '50px',
    height: '50px',
    border: '5px solid #f3f3f3',
    borderTop: '5px solid #007bff',
    borderRadius: '50%',
    animation: 'spin 1s linear infinite'
  };
  
  return <div style={spinnerStyle}></div>;
}

// CSS keyframes (put in your CSS file)
/*
@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes slideIn {
  from { transform: translateX(-100%); }
  to { transform: translateX(0); }
}

@keyframes bounce {
  0%, 20%, 50%, 80%, 100% { transform: translateY(0); }
  40% { transform: translateY(-30px); }
  60% { transform: translateY(-15px); }
}
*/
```

**React state-based animations:**
```jsx
function FadeInComponent() {
  const [isVisible, setIsVisible] = useState(false);
  
  useEffect(() => {
    setIsVisible(true);  // Trigger animation on mount
  }, []);
  
  return (
    <div style={{
      opacity: isVisible ? 1 : 0,
      transform: isVisible ? 'translateY(0)' : 'translateY(20px)',
      transition: 'all 0.5s ease-in-out'
    }}>
      This fades in when component mounts
    </div>
  );
}
```

**Animated card flip:**
```jsx
function FlipCard() {
  const [isFlipped, setIsFlipped] = useState(false);
  
  const cardStyle = {
    width: '200px',
    height: '200px',
    perspective: '1000px'
  };
  
  const cardInnerStyle = {
    position: 'relative',
    width: '100%',
    height: '100%',
    textAlign: 'center',
    transition: 'transform 0.6s',
    transformStyle: 'preserve-3d',
    transform: isFlipped ? 'rotateY(180deg)' : 'rotateY(0deg)'
  };
  
  const sideStyle = {
    position: 'absolute',
    width: '100%',
    height: '100%',
    backfaceVisibility: 'hidden',
    display: 'flex',
    alignItems: 'center',
    justifyContent: 'center',
    borderRadius: '10px'
  };
  
  return (
    <div style={cardStyle} onClick={() => setIsFlipped(!isFlipped)}>
      <div style={cardInnerStyle}>
        <div style={{...sideStyle, backgroundColor: '#007bff', color: 'white'}}>
          Front Side
        </div>
        <div style={{
          ...sideStyle, 
          backgroundColor: '#28a745', 
          color: 'white',
          transform: 'rotateY(180deg)'
        }}>
          Back Side
        </div>
      </div>
    </div>
  );
}
```

69. **Creating shapes with CSS** - Triangles, circles, and creative shapes.

**Basic shapes:**
```jsx
function BasicShapes() {
  // Circle
  const circleStyle = {
    width: '100px',
    height: '100px',
    borderRadius: '50%',
    backgroundColor: '#007bff'
  };
  
  // Triangle (using borders)
  const triangleStyle = {
    width: 0,
    height: 0,
    borderLeft: '50px solid transparent',
    borderRight: '50px solid transparent',
    borderBottom: '100px solid #28a745'
  };
  
  // Diamond
  const diamondStyle = {
    width: '50px',
    height: '50px',
    backgroundColor: '#ffc107',
    transform: 'rotate(45deg)'
  };
  
  return (
    <div style={{ display: 'flex', gap: '20px', alignItems: 'center' }}>
      <div style={circleStyle}></div>
      <div style={triangleStyle}></div>
      <div style={diamondStyle}></div>
    </div>
  );
}
```

**Advanced shapes:**
```jsx
function AdvancedShapes() {
  // Heart shape
  const heartStyle = {
    position: 'relative',
    width: '100px',
    height: '90px',
    transform: 'rotate(-45deg)'
  };
  
  const heartBeforeAfter = {
    content: '""',
    width: '52px',
    height: '80px',
    position: 'absolute',
    left: '50px',
    transform: 'rotate(-45deg)',
    backgroundColor: '#e74c3c',
    borderRadius: '50px 50px 0 0'
  };
  
  // Star shape (using clip-path)
  const starStyle = {
    width: '100px',
    height: '100px',
    backgroundColor: '#f1c40f',
    clipPath: 'polygon(50% 0%, 61% 35%, 98% 35%, 68% 57%, 79% 91%, 50% 70%, 21% 91%, 32% 57%, 2% 35%, 39% 35%)'
  };
  
  // Wave shape
  const waveStyle = {
    width: '200px',
    height: '100px',
    backgroundColor: '#3498db',
    borderRadius: '0 0 50% 50%'
  };
  
  // Hexagon
  const hexagonStyle = {
    width: '100px',
    height: '57.735px',
    backgroundColor: '#9b59b6',
    clipPath: 'polygon(50% 0%, 100% 25%, 100% 75%, 50% 100%, 0% 75%, 0% 25%)'
  };
  
  return (
    <div style={{ display: 'flex', gap: '20px', flexWrap: 'wrap' }}>
      <div style={starStyle}></div>
      <div style={waveStyle}></div>
      <div style={hexagonStyle}></div>
    </div>
  );
}
```

**CSS-only icons:**
```jsx
function CSSIcons() {
  // Hamburger menu icon
  const hamburgerStyle = {
    width: '30px',
    height: '4px',
    backgroundColor: '#333',
    margin: '6px 0',
    transition: '0.4s'
  };
  
  // Plus icon
  const plusStyle = {
    width: '30px',
    height: '30px',
    position: 'relative'
  };
  
  const plusLine = {
    position: 'absolute',
    backgroundColor: '#333'
  };
  
  const plusHorizontal = {
    ...plusLine,
    width: '30px',
    height: '4px',
    top: '50%',
    transform: 'translateY(-50%)'
  };
  
  const plusVertical = {
    ...plusLine,
    width: '4px',
    height: '30px',
    left: '50%',
    transform: 'translateX(-50%)'
  };
  
  return (
    <div style={{ display: 'flex', gap: '40px' }}>
      {/* Hamburger */}
      <div>
        <div style={hamburgerStyle}></div>
        <div style={hamburgerStyle}></div>
        <div style={hamburgerStyle}></div>
      </div>
      
      {/* Plus */}
      <div style={plusStyle}>
        <div style={plusHorizontal}></div>
        <div style={plusVertical}></div>
      </div>
    </div>
  );
}
```

70. **React styling approaches** - Different ways to style React components.

**Inline styles:**
```jsx
function InlineStyled() {
  const containerStyle = {
    padding: '20px',
    backgroundColor: '#f8f9fa',
    borderRadius: '8px',
    boxShadow: '0 2px 4px rgba(0,0,0,0.1)'
  };
  
  return (
    <div style={containerStyle}>
      <h2 style={{ color: '#007bff', marginBottom: '10px' }}>
        Inline Styled Component
      </h2>
    </div>
  );
}
```

**CSS Modules:**
```jsx
// styles.module.css
/*
.container {
  padding: 20px;
  background-color: #f8f9fa;
  border-radius: 8px;
}

.title {
  color: #007bff;
  margin-bottom: 10px;
}
*/

import styles from './styles.module.css';

function CSSModuleStyled() {
  return (
    <div className={styles.container}>
      <h2 className={styles.title}>CSS Module Styled</h2>
    </div>
  );
}
```

**Styled Components (external library):**
```jsx
// npm install styled-components
import styled from 'styled-components';

const Container = styled.div`
  padding: 20px;
  background-color: #f8f9fa;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
`;

const Title = styled.h2`
  color: #007bff;
  margin-bottom: 10px;
  
  &:hover {
    color: #0056b3;
  }
`;

function StyledComponentExample() {
  return (
    <Container>
      <Title>Styled Component</Title>
    </Container>
  );
}
```

**Dynamic styling with state:**
```jsx
function DynamicStyled() {
  const [isActive, setIsActive] = useState(false);
  const [theme, setTheme] = useState('light');
  
  const buttonStyle = {
    padding: '10px 20px',
    border: 'none',
    borderRadius: '4px',
    cursor: 'pointer',
    backgroundColor: isActive ? '#28a745' : '#007bff',
    color: 'white',
    transform: isActive ? 'scale(1.1)' : 'scale(1)',
    transition: 'all 0.3s ease'
  };
  
  const containerStyle = {
    padding: '20px',
    backgroundColor: theme === 'dark' ? '#333' : '#fff',
    color: theme === 'dark' ? '#fff' : '#333',
    minHeight: '200px'
  };
  
  return (
    <div style={containerStyle}>
      <button
        style={buttonStyle}
        onClick={() => setIsActive(!isActive)}
      >
        {isActive ? 'Active' : 'Inactive'}
      </button>
      
      <button
        style={{ marginLeft: '10px' }}
        onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}
      >
        Toggle Theme
      </button>
    </div>
  );
}
```

**Responsive design patterns:**
```jsx
function ResponsiveComponent() {
  const [windowWidth, setWindowWidth] = useState(window.innerWidth);
  
  useEffect(() => {
    const handleResize = () => setWindowWidth(window.innerWidth);
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  const containerStyle = {
    display: 'grid',
    gridTemplateColumns: windowWidth < 768 
      ? '1fr'                    // Mobile: single column
      : 'repeat(auto-fit, minmax(300px, 1fr))',  // Desktop: responsive grid
    gap: '20px',
    padding: '20px'
  };
  
  return (
    <div style={containerStyle}>
      <div style={{ backgroundColor: '#007bff', padding: '20px', color: 'white' }}>
        Item 1
      </div>
      <div style={{ backgroundColor: '#28a745', padding: '20px', color: 'white' }}>
        Item 2
      </div>
      <div style={{ backgroundColor: '#ffc107', padding: '20px', color: 'white' }}>
        Item 3
      </div>
    </div>
  );
}

// CSS media queries approach
/*
.responsive-container {
  display: grid;
  gap: 20px;
  padding: 20px;
}

@media (max-width: 768px) {
  .responsive-container {
    grid-template-columns: 1fr;
  }
}

@media (min-width: 769px) {
  .responsive-container {
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  }
}
*/
```

47. **createSlice** combines actions and reducers in one place. It automatically generates action creators and action types.
    ```jsx
    import { createSlice } from '@reduxjs/toolkit';
    
    const todoSlice = createSlice({
      name: 'todos',  // Used to generate action types
      initialState: {
        items: [],
        loading: false
      },
      reducers: {
        // Redux Toolkit uses Immer under the hood, so you can "mutate" state directly
        addTodo: (state, action) => {
          state.items.push({
            id: Date.now(),
            text: action.payload,
            completed: false
          });
        },
        toggleTodo: (state, action) => {
          const todo = state.items.find(todo => todo.id === action.payload);
          if (todo) {
            todo.completed = !todo.completed;  // This looks like mutation but it's safe
          }
        },
        setLoading: (state, action) => {
          state.loading = action.payload;
        }
      }
    });
    
    // Export actions (automatically generated)
    export const { addTodo, toggleTodo, setLoading } = todoSlice.actions;
    
    // Export reducer
    export default todoSlice.reducer;
    ```

48. **createAsyncThunk** handles async operations like API calls. It automatically dispatches pending, fulfilled, and rejected actions.
    ```jsx
    import { createAsyncThunk, createSlice } from '@reduxjs/toolkit';
    
    // Async thunk for fetching todos
    export const fetchTodos = createAsyncThunk(
      'todos/fetchTodos',  // Action type prefix
      async (userId) => {
        const response = await fetch(`/api/todos?userId=${userId}`);
        if (!response.ok) {
          throw new Error('Failed to fetch todos');
        }
        return response.json();  // This becomes action.payload in fulfilled case
      }
    );
    
    const todoSlice = createSlice({
      name: 'todos',
      initialState: {
        items: [],
        loading: false,
        error: null
      },
      reducers: {
        // Regular synchronous reducers
        addTodo: (state, action) => {
          state.items.push(action.payload);
        }
      },
      extraReducers: (builder) => {
        // Handle async thunk actions
        builder
          .addCase(fetchTodos.pending, (state) => {
            state.loading = true;
            state.error = null;
          })
          .addCase(fetchTodos.fulfilled, (state, action) => {
            state.loading = false;
            state.items = action.payload;  // API response data
          })
          .addCase(fetchTodos.rejected, (state, action) => {
            state.loading = false;
            state.error = action.error.message;  // Error message
          });
      }
    });
    ```

49. **Using async thunks in components** is the same as dispatching regular actions.
    ```jsx
    function TodoList() {
      const dispatch = useDispatch();
      const { items: todos, loading, error } = useSelector(state => state.todos);
      
      useEffect(() => {
        dispatch(fetchTodos(123));  // Dispatch async thunk
      }, [dispatch]);
      
      if (loading) return <div>Loading todos...</div>;
      if (error) return <div>Error: {error}</div>;
      
      return (
        <ul>
          {todos.map(todo => (
            <li key={todo.id}>{todo.text}</li>
          ))}
        </ul>
      );
    }
    ```

## Redux Best Practices

50. **Normalize state shape** means storing data in a flat structure with IDs as keys, similar to database tables. This makes updates easier and prevents data duplication.
    ```jsx
    // Bad: Nested structure
    const badState = {
      posts: [
        { id: 1, title: 'Post 1', author: { id: 1, name: 'John' } },
        { id: 2, title: 'Post 2', author: { id: 1, name: 'John' } }  // Duplicated author
      ]
    };
    
    // Good: Normalized structure
    const goodState = {
      posts: {
        byId: {
          1: { id: 1, title: 'Post 1', authorId: 1 },
          2: { id: 2, title: 'Post 2', authorId: 1 }
        },
        allIds: [1, 2]
      },
      authors: {
        byId: {
          1: { id: 1, name: 'John' }  // Single source of truth for author
        },
        allIds: [1]
      }
    };
    ```

51. **Use selector functions** to encapsulate state access logic and make it reusable across components.
    ```jsx
    // Selector functions
    const selectAllTodos = (state) => state.todos.items;
    const selectTodoById = (state, todoId) => state.todos.items.find(todo => todo.id === todoId);
    const selectCompletedTodos = (state) => state.todos.items.filter(todo => todo.completed);
    
    // Using in components
    function TodoList() {
      const todos = useSelector(selectAllTodos);
      const completedTodos = useSelector(selectCompletedTodos);
      
      return (
        <div>
          <p>Total: {todos.length}, Completed: {completedTodos.length}</p>
          {todos.map(todo => <TodoItem key={todo.id} todo={todo} />)}
        </div>
      );
    }
    ```

52. **Keep reducers pure** means they should not have side effects, API calls, or mutations. They should only transform state based on the action.
    ```jsx
    // Bad: Side effects in reducer
    const badReducer = (state, action) => {
      switch (action.type) {
        case 'ADD_TODO':
          localStorage.setItem('todos', JSON.stringify([...state.todos, action.payload]));  // Side effect!
          return { ...state, todos: [...state.todos, action.payload] };
      }
    };
    
    // Good: Pure reducer, side effects handled elsewhere
    const goodReducer = (state, action) => {
      switch (action.type) {
        case 'ADD_TODO':
          return { ...state, todos: [...state.todos, action.payload] };  // Pure transformation
      }
    };
    
    // Handle side effects in middleware or components
    useEffect(() => {
      localStorage.setItem('todos', JSON.stringify(todos));
    }, [todos]);
    ```

53. **Use Redux DevTools** for debugging. Redux Toolkit enables it by default, or you can add it manually.
    ```jsx
    // With vanilla Redux
    const store = createStore(
      rootReducer,
      window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__()
    );
    
    // With Redux Toolkit (automatic)
    const store = configureStore({
      reducer: rootReducer  // DevTools enabled by default
    });
    ```

54. **When to use Redux vs local state**: Use Redux for state that needs to be shared across many components or persisted across page navigation. Use local state for component-specific data.
    ```jsx
    // Use local state for: form inputs, UI state (modals, dropdowns), temporary data
    function LoginForm() {
      const [email, setEmail] = useState('');  // Local state for form
      const [showPassword, setShowPassword] = useState(false);  // Local UI state
    }
    
    // Use Redux for: user authentication, app-wide settings, shared data
    function App() {
      const user = useSelector(state => state.auth.user);  // Global user state
      const theme = useSelector(state => state.settings.theme);  // Global settings
    }
    ```

## Essential CSS Properties Deep Dive

71. **Most common CSS properties** and their values you'll use daily.

**Layout Properties:**
```css
/* Display - How element is rendered */
display: block;           /* Takes full width, stacks vertically */
display: inline;          /* Only takes needed width, flows horizontally */
display: inline-block;    /* Inline but can have width/height */
display: flex;            /* Flexbox container */
display: grid;            /* Grid container */
display: none;            /* Element disappears completely */

/* Position - How element is positioned */
position: static;         /* Default - normal document flow */
position: relative;       /* Relative to its normal position */
position: absolute;       /* Relative to nearest positioned parent */
position: fixed;          /* Relative to viewport, stays on scroll */
position: sticky;         /* Switches between relative and fixed */

/* Box model */
width: 100px;            /* Fixed width */
width: 50%;              /* Percentage of parent */
width: 100vw;            /* Viewport width */
width: auto;             /* Browser calculates */
width: max-content;      /* Width of content */
width: min-content;      /* Minimum width needed */

height: 100px;           /* Fixed height */
height: 100vh;           /* Viewport height */
height: auto;            /* Browser calculates */

/* Spacing */
margin: 10px;            /* All sides */
margin: 10px 20px;       /* Top/bottom, left/right */
margin: 10px 20px 30px 40px;  /* Top, right, bottom, left */
margin: auto;            /* Center horizontally */

padding: 10px;           /* Internal spacing - same syntax as margin */

/* Borders */
border: 1px solid black;        /* Width, style, color */
border-radius: 5px;             /* Rounded corners */
border-radius: 50%;             /* Perfect circle */
```

**Typography Properties:**
```css
/* Font properties */
font-family: Arial, sans-serif;     /* Font stack */
font-size: 16px;                    /* Size in pixels */
font-size: 1rem;                    /* Relative to root font size */
font-weight: normal;                /* normal, bold, 100-900 */
font-style: italic;                 /* normal, italic, oblique */

/* Text properties */
color: #333;                        /* Text color */
text-align: center;                 /* left, center, right, justify */
text-decoration: underline;         /* none, underline, line-through */
line-height: 1.5;                   /* Spacing between lines */
letter-spacing: 2px;                /* Space between characters */
text-transform: uppercase;          /* uppercase, lowercase, capitalize */
```

**Background Properties:**
```css
background-color: #f0f0f0;
background-image: url('image.jpg');
background-size: cover;             /* cover, contain, 100px, 50% */
background-position: center;        /* center, top, bottom, left, right */
background-repeat: no-repeat;       /* repeat, repeat-x, repeat-y, no-repeat */
```

**Flexbox Properties:**
```css
/* Container properties */
display: flex;
flex-direction: row;                /* row, column, row-reverse, column-reverse */
justify-content: center;            /* flex-start, center, flex-end, space-between, space-around */
align-items: center;                /* flex-start, center, flex-end, stretch, baseline */
flex-wrap: wrap;                    /* nowrap, wrap, wrap-reverse */
gap: 20px;                          /* Space between flex items */

/* Item properties */
flex: 1;                           /* Grow, shrink, basis combined */
flex-grow: 1;                      /* How much item should grow */
flex-shrink: 0;                    /* How much item should shrink */
flex-basis: 200px;                 /* Initial size before growing/shrinking */
align-self: center;                /* Override container's align-items for this item */
```

72. **The Border Triangle Trick** - How zero width/height creates shapes.

**How it works:**
```jsx
function TriangleExplanation() {
  // Step 1: Normal div with borders
  const normalDiv = {
    width: '100px',
    height: '100px',
    borderTop: '50px solid red',
    borderRight: '50px solid blue',
    borderBottom: '50px solid green',
    borderLeft: '50px solid yellow'
  };
  
  // Step 2: Make width smaller
  const smallerDiv = {
    width: '20px',
    height: '20px',
    borderTop: '50px solid red',
    borderRight: '50px solid blue', 
    borderBottom: '50px solid green',
    borderLeft: '50px solid yellow'
  };
  
  // Step 3: Zero width and height - only borders remain!
  const triangle = {
    width: 0,
    height: 0,
    borderTop: '50px solid transparent',    // Make unwanted sides transparent
    borderRight: '50px solid transparent',
    borderBottom: '50px solid green',       // This becomes the triangle
    borderLeft: '50px solid transparent'
  };
  
  return (
    <div style={{ display: 'flex', gap: '50px', alignItems: 'center' }}>
      <div style={normalDiv}></div>
      <div style={smallerDiv}></div>
      <div style={triangle}></div>
    </div>
  );
}
```

**Why this works:**
```
When width = 100px, height = 100px:
┌─────────────────┐ ← Top border (red)
│                 │
│    Content      │ ← Actual content area  
│     Area        │
│                 │
└─────────────────┘ ← Bottom border (green)

When width = 0, height = 0:
   /\     ← Top border becomes triangle pointing up
  /  \    
 /____\   ← Bottom border becomes triangle pointing down
 
Since content area is zero, only the border "triangles" remain!
```

**Different triangle directions:**
```jsx
function AllTriangles() {
  // Triangle pointing up
  const triangleUp = {
    width: 0,
    height: 0,
    borderLeft: '50px solid transparent',
    borderRight: '50px solid transparent',
    borderBottom: '50px solid blue'    // Bottom border creates upward triangle
  };
  
  // Triangle pointing down  
  const triangleDown = {
    width: 0,
    height: 0,
    borderLeft: '50px solid transparent',
    borderRight: '50px solid transparent', 
    borderTop: '50px solid red'        // Top border creates downward triangle
  };
  
  // Triangle pointing right
  const triangleRight = {
    width: 0,
    height: 0,
    borderTop: '50px solid transparent',
    borderBottom: '50px solid transparent',
    borderLeft: '50px solid green'     // Left border creates rightward triangle
  };
  
  // Triangle pointing left
  const triangleLeft = {
    width: 0,
    height: 0,
    borderTop: '50px solid transparent',
    borderBottom: '50px solid transparent',
    borderRight: '50px solid orange'   // Right border creates leftward triangle
  };
  
  return (
    <div style={{ display: 'flex', gap: '30px', alignItems: 'center' }}>
      <div style={triangleUp}></div>
      <div style={triangleDown}></div>
      <div style={triangleRight}></div>
      <div style={triangleLeft}></div>
    </div>
  );
}
```

73. **Transform & Transition** - The mini-languages of CSS animation.

**Transform - Spatial manipulation (like a graphics program):**
```jsx
function TransformExamples() {
  const examples = {
    // TRANSLATE - Move element
    translate: { transform: 'translate(50px, 100px)' },        // Move right 50px, down 100px
    translateX: { transform: 'translateX(50px)' },             // Move only horizontally
    translateY: { transform: 'translateY(-20px)' },            // Move only vertically
    translate3d: { transform: 'translate3d(10px, 20px, 30px)' }, // 3D movement
    
    // SCALE - Resize element
    scale: { transform: 'scale(1.5)' },                        // 150% of original size
    scaleX: { transform: 'scaleX(2)' },                        // Double width only
    scaleY: { transform: 'scaleY(0.5)' },                      // Half height only
    scaleXY: { transform: 'scale(2, 0.5)' },                   // Double width, half height
    
    // ROTATE - Spin element
    rotate: { transform: 'rotate(45deg)' },                    // 45 degree rotation
    rotateX: { transform: 'rotateX(45deg)' },                  // 3D rotation around X axis
    rotateY: { transform: 'rotateY(45deg)' },                  // 3D rotation around Y axis
    rotateZ: { transform: 'rotateZ(45deg)' },                  // Same as rotate()
    
    // SKEW - Distort element
    skew: { transform: 'skew(20deg, 10deg)' },                 // Skew X and Y
    skewX: { transform: 'skewX(20deg)' },                      // Skew only horizontally
    skewY: { transform: 'skewY(10deg)' },                      // Skew only vertically
    
    // COMBINED - Multiple transforms
    combined: { 
      transform: 'translate(50px, 20px) rotate(45deg) scale(1.2)' 
    }
  };
  
  return (
    <div style={{ display: 'grid', gridTemplateColumns: 'repeat(3, 1fr)', gap: '20px' }}>
      {Object.entries(examples).map(([key, style]) => (
        <div key={key} style={{ 
          ...style, 
          width: '50px', 
          height: '50px', 
          backgroundColor: '#007bff',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
          color: 'white',
          fontSize: '12px'
        }}>
          {key}
        </div>
      ))}
    </div>
  );
}
```

**Transition - How changes happen over time:**
```jsx
function TransitionExamples() {
  const [hover, setHover] = useState(false);
  
  const baseStyle = {
    width: '100px',
    height: '100px',
    backgroundColor: '#007bff',
    margin: '10px'
  };
  
  // Different transition properties
  const examples = [
    {
      name: 'All properties',
      style: {
        ...baseStyle,
        transition: 'all 0.3s ease',           // Property, duration, timing function
        ...(hover && { 
          backgroundColor: 'red', 
          transform: 'scale(1.2)' 
        })
      }
    },
    {
      name: 'Specific property',
      style: {
        ...baseStyle,
        transition: 'background-color 0.5s linear',  // Only animate background
        ...(hover && { backgroundColor: 'green' })
      }
    },
    {
      name: 'Multiple properties',
      style: {
        ...baseStyle,
        transition: 'transform 0.3s ease, opacity 0.5s ease-in-out',
        ...(hover && { 
          transform: 'rotate(180deg)', 
          opacity: 0.5 
        })
      }
    },
    {
      name: 'Timing functions',
      style: {
        ...baseStyle,
        transition: 'transform 1s cubic-bezier(0.68, -0.55, 0.265, 1.55)', // Bounce effect
        ...(hover && { transform: 'translateY(-50px)' })
      }
    }
  ];
  
  return (
    <div>
      <button onClick={() => setHover(!hover)}>
        {hover ? 'Reset' : 'Animate'}
      </button>
      <div style={{ display: 'flex', flexWrap: 'wrap' }}>
        {examples.map((example, index) => (
          <div key={index}>
            <p>{example.name}</p>
            <div style={example.style}></div>
          </div>
        ))}
      </div>
    </div>
  );
}

// Timing function options:
/*
ease          - Slow start, fast middle, slow end (default)
linear        - Constant speed
ease-in       - Slow start, fast end
ease-out      - Fast start, slow end  
ease-in-out   - Slow start and end
cubic-bezier(n,n,n,n) - Custom curve
*/
```

**Why they seem like whole languages:**
- **Transform** has 16+ different functions (translate, rotate, scale, skew, matrix, perspective)
- **Transition** has timing functions, delays, multiple property support
- **They combine** with each other and other CSS properties
- **3D capabilities** add complexity (perspective, transform-style, backface-visibility)

74. **Flexbox Gap Property** - Modern spacing solution.

**Yes, gap IS part of flexbox (and grid):**
```jsx
function FlexboxGapExamples() {
  // Without gap - old way (messy)
  const oldWay = {
    display: 'flex',
    // Have to manually add margins to each item ❌
  };
  
  const oldItemStyle = {
    marginRight: '20px',  // Every item needs margin
    marginBottom: '10px'  // For wrapping
  };
  
  // With gap - new way (clean)
  const newWay = {
    display: 'flex',
    gap: '20px',          // ✅ Automatic spacing between ALL items
    flexWrap: 'wrap'
  };
  
  const newItemStyle = {
    // No margins needed! ✅
  };
  
  return (
    <div>
      <h3>Old way (with margins):</h3>
      <div style={oldWay}>
        <div style={{...oldItemStyle, backgroundColor: '#007bff', padding: '10px'}}>Item 1</div>
        <div style={{...oldItemStyle, backgroundColor: '#28a745', padding: '10px'}}>Item 2</div>
        <div style={{...oldItemStyle, backgroundColor: '#ffc107', padding: '10px'}}>Item 3</div>
      </div>
      
      <h3>New way (with gap):</h3>
      <div style={newWay}>
        <div style={{...newItemStyle, backgroundColor: '#007bff', padding: '10px'}}>Item 1</div>
        <div style={{...newItemStyle, backgroundColor: '#28a745', padding: '10px'}}>Item 2</div>
        <div style={{...newItemStyle, backgroundColor: '#ffc107', padding: '10px'}}>Item 3</div>
      </div>
    </div>
  );
}
```

**Gap syntax options:**
```css
/* Single value - both row and column gap */
gap: 20px;

/* Two values - row gap, column gap */
gap: 20px 30px;

/* Individual properties */
row-gap: 20px;        /* Vertical spacing */
column-gap: 30px;     /* Horizontal spacing */

/* Different units */
gap: 1rem;            /* Relative to font size */
gap: 2%;              /* Percentage of container */
gap: 10px 5%;         /* Mixed units */
```

**Gap works in both Flexbox AND Grid:**
```jsx
function GapInGridAndFlex() {
  const gridStyle = {
    display: 'grid',
    gridTemplateColumns: 'repeat(3, 1fr)',
    gap: '20px'           // Works in grid
  };
  
  const flexStyle = {
    display: 'flex',
    flexWrap: 'wrap',
    gap: '15px'           // Also works in flex
  };
  
  return (
    <div>
      <h3>Grid with gap:</h3>
      <div style={gridStyle}>
        <div style={{backgroundColor: '#007bff', padding: '20px'}}>1</div>
        <div style={{backgroundColor: '#28a745', padding: '20px'}}>2</div>
        <div style={{backgroundColor: '#ffc107', padding: '20px'}}>3</div>
      </div>
      
      <h3>Flex with gap:</h3>
      <div style={flexStyle}>
        <div style={{backgroundColor: '#007bff', padding: '20px'}}>1</div>
        <div style={{backgroundColor: '#28a745', padding: '20px'}}>2</div>
        <div style={{backgroundColor: '#ffc107', padding: '20px'}}>3</div>
      </div>
    </div>
  );
}
```

75. **Clip-path and Polygon** - Advanced shape creation.

**Clip-path** creates complex shapes by "clipping" (hiding) parts of an element:
```jsx
function ClipPathExamples() {
  const baseStyle = {
    width: '150px',
    height: '150px',
    backgroundColor: '#007bff',
    margin: '20px'
  };
  
  const shapes = [
    {
      name: 'Triangle',
      style: {
        ...baseStyle,
        clipPath: 'polygon(50% 0%, 0% 100%, 100% 100%)'
        // Points: top-center, bottom-left, bottom-right
      }
    },
    {
      name: 'Hexagon', 
      style: {
        ...baseStyle,
        clipPath: 'polygon(50% 0%, 100% 25%, 100% 75%, 50% 100%, 0% 75%, 0% 25%)'
        // 6 points creating hexagon shape
      }
    },
    {
      name: 'Star',
      style: {
        ...baseStyle,
        clipPath: 'polygon(50% 0%, 61% 35%, 98% 35%, 68% 57%, 79% 91%, 50% 70%, 21% 91%, 32% 57%, 2% 35%, 39% 35%)'
        // 10 points creating star shape
      }
    },
    {
      name: 'Arrow',
      style: {
        ...baseStyle,
        clipPath: 'polygon(0% 20%, 60% 20%, 60% 0%, 100% 50%, 60% 100%, 60% 80%, 0% 80%)'
        // Arrow pointing right
      }
    },
    {
      name: 'Circle',
      style: {
        ...baseStyle,
        clipPath: 'circle(50% at 50% 50%)'
        // Radius 50%, centered at 50% 50%
      }
    },
    {
      name: 'Ellipse',
      style: {
        ...baseStyle,
        clipPath: 'ellipse(70% 50% at 50% 50%)'
        // Width 70%, height 50%, centered
      }
    }
  ];
  
  return (
    <div style={{ display: 'grid', gridTemplateColumns: 'repeat(3, 1fr)', gap: '20px' }}>
      {shapes.map((shape, index) => (
        <div key={index} style={{ textAlign: 'center' }}>
          <div style={shape.style}></div>
          <p>{shape.name}</p>
        </div>
      ))}
    </div>
  );
}
```

**Understanding Polygon coordinates:**
```
Polygon uses percentage coordinates:
0% 0%     = top-left corner
50% 0%    = top-center  
100% 0%   = top-right corner
0% 50%    = middle-left
50% 50%   = center
100% 50%  = middle-right
0% 100%   = bottom-left
50% 100%  = bottom-center
100% 100% = bottom-right

Triangle example:
polygon(50% 0%, 0% 100%, 100% 100%)
        ↑       ↑         ↑
   top-center, bottom-left, bottom-right
```

**Interactive clip-path generator:**
```jsx
function InteractiveClipPath() {
  const [points, setPoints] = useState([
    [50, 0],    // Top center
    [100, 50],  // Right center  
    [50, 100],  // Bottom center
    [0, 50]     // Left center
  ]);
  
  const polygonPath = `polygon(${points.map(([x, y]) => `${x}% ${y}%`).join(', ')})`;
  
  const shapeStyle = {
    width: '200px',
    height: '200px',
    backgroundColor: '#007bff',
    clipPath: polygonPath,
    margin: '20px auto'
  };
  
  return (
    <div>
      <div style={shapeStyle}></div>
      <p>Clip-path: {polygonPath}</p>
      <div>
        {points.map(([x, y], index) => (
          <div key={index}>
            Point {index + 1}: 
            <input 
              type="range" 
              min="0" 
              max="100" 
              value={x}
              onChange={(e) => {
                const newPoints = [...points];
                newPoints[index][0] = Number(e.target.value);
                setPoints(newPoints);
              }}
            /> X: {x}%
            <input 
              type="range" 
              min="0" 
              max="100" 
              value={y}
              onChange={(e) => {
                const newPoints = [...points];
                newPoints[index][1] = Number(e.target.value);
                setPoints(newPoints);
              }}
            /> Y: {y}%
          </div>
        ))}
      </div>
    </div>
  );
}
```

**Other clip-path functions:**
```css
/* Circle - radius at center position */
clip-path: circle(50px at 50% 50%);

/* Ellipse - width height at center position */  
clip-path: ellipse(50px 30px at 50% 50%);

/* Inset - rectangle with rounded corners */
clip-path: inset(10px 20px 30px 40px round 15px);
/*         top right bottom left   corner-radius */

/* Path - SVG path syntax (complex) */
clip-path: path('M 0 0 L 100 0 L 50 100 Z');
```

**Why these seem complex:**
- **Coordinate system** uses percentages and various units
- **Multiple syntaxes** for different shape types
- **SVG integration** adds path complexity  
- **Browser compatibility** varies for advanced features
- **Visual tools** are almost necessary for complex shapes

**Useful tools:**
- Clippy CSS clip-path maker: https://bennettfeely.com/clippy/
- CSS clip-path generator: https://www.cssportal.com/css-clip-path-generator/

## Bundle Size Optimization - Make Your App Load Lightning Fast ⚡

Bundle size is **THE #1 factor** that affects your app's loading speed. A smaller bundle = faster loading = happier users. Here's how to shrink your JavaScript bundles from bloated to blazing fast.

76. **First, measure your bundle size** - You can't optimize what you can't measure! Use these tools to see exactly what's making your bundle fat.

**Webpack Bundle Analyzer (the gold standard):**
```bash
# Install the analyzer
npm install --save-dev webpack-bundle-analyzer

# For Create React App
npm install --save-dev source-map-explorer
npm run build
npm run analyze  # or npx source-map-explorer build/static/js/*.js
```

**What you'll see:**
```
Your bundle breakdown:
├── node_modules (85% - THIS IS THE PROBLEM!)
│   ├── lodash: 70KB 😱
│   ├── moment: 67KB 😱  
│   ├── react: 42KB ✅
│   └── axios: 15KB ✅
├── src (15% - your actual code)
│   ├── components: 25KB
│   └── utils: 10KB
```

**Bundle analyzer shows you:**
- Which libraries are biggest (usually the culprits)
- Duplicate dependencies 
- Code that's imported but never used
- Opportunities for code splitting

77. **Import optimization** - The fastest way to cut bundle size in half. Most bloated bundles come from **importing entire libraries when you only need one function**.

**❌ BAD: Importing entire libraries**
```jsx
// These imports are BUNDLE KILLERS
import _ from 'lodash';                    // 70KB for one function!
import moment from 'moment';              // 67KB for date formatting!
import * as MaterialUI from '@mui/material';  // 1MB+ for a few components!
import { Icon } from '@iconify/react';    // 200KB+ for icons!

function MyComponent() {
  const debouncedSearch = _.debounce(search, 300);  // Used 1KB, imported 70KB
  const formattedDate = moment().format('YYYY-MM-DD');  // Used 2KB, imported 67KB
  
  return (
    <MaterialUI.Button>          // Used 15KB, imported 1MB+
      <Icon icon="mdi:home" />   // Used 1KB, imported 200KB+
    </MaterialUI.Button>
  );
}
```

**✅ GOOD: Specific imports**
```jsx
// These imports are BUNDLE FRIENDLY
import debounce from 'lodash/debounce';        // 2KB instead of 70KB!
import { format } from 'date-fns';            // 5KB instead of 67KB!
import { Button } from '@mui/material';       // 15KB instead of 1MB+
import HomeIcon from '@mui/icons-material/Home';  // 1KB instead of 200KB+

function MyComponent() {
  const debouncedSearch = debounce(search, 300);     // Only what you need
  const formattedDate = format(new Date(), 'yyyy-MM-dd');  // Tree-shakable!
  
  return (
    <Button startIcon={<HomeIcon />}>              // Specific imports only
      Click me
    </Button>
  );
}
```

**Bundle size comparison:**
```
Bad imports:  70KB + 67KB + 1MB + 200KB = 1.34MB 😱
Good imports: 2KB + 5KB + 15KB + 1KB = 23KB ✅

That's a 98% reduction! 🎉
```

78. **Tree shaking** - Let your bundler automatically remove unused code. But you need to help it by using ES6 imports and tree-shake-friendly libraries.

**Libraries that support tree shaking (✅ GOOD):**
```jsx
// These will only include what you import
import { debounce, throttle } from 'lodash-es';  // Note: lodash-es, not lodash
import { format, addDays } from 'date-fns';
import { Button, TextField } from '@mui/material';
import { selectUser, selectPosts } from './selectors';

// Your bundle will ONLY include debounce, throttle, format, addDays, Button, TextField
```

**Libraries that DON'T support tree shaking (❌ BAD):**
```jsx
// These will include the entire library
import { debounce } from 'lodash';        // Includes ALL of lodash
import { Icon } from '@iconify/react';    // Includes ALL icons
import firebase from 'firebase';         // Includes ALL Firebase services

// Solution: Use specific imports
import debounce from 'lodash/debounce';   // ✅ Only debounce
import { getAuth } from 'firebase/auth';  // ✅ Only auth service
```

**How to check if tree shaking is working:**
```jsx
// Create a test file
export const usedFunction = () => console.log('This is used');
export const unusedFunction = () => console.log('This is never used');

// In your component
import { usedFunction } from './test';
// Don't import unusedFunction

// Build and check bundle - unusedFunction should NOT be in the final bundle
```

79. **Code splitting** - Load code only when needed instead of everything upfront. This is the **most powerful** optimization technique.

**Route-based code splitting (easiest wins):**
```jsx
// ❌ BAD: All pages load immediately
import Home from './pages/Home';
import Dashboard from './pages/Dashboard';
import Profile from './pages/Profile';
import AdminPanel from './pages/AdminPanel';  // 500KB admin code loaded for everyone!

function App() {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/dashboard" element={<Dashboard />} />
      <Route path="/profile" element={<Profile />} />
      <Route path="/admin" element={<AdminPanel />} />  {/* Heavy admin code */}
    </Routes>
  );
}
// Result: 800KB bundle loaded immediately for everyone

// ✅ GOOD: Pages load only when visited
import { lazy, Suspense } from 'react';

const Home = lazy(() => import('./pages/Home'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Profile = lazy(() => import('./pages/Profile'));
const AdminPanel = lazy(() => import('./pages/AdminPanel'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}> 
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/profile" element={<Profile />} />
        <Route path="/admin" element={<AdminPanel />} />
      </Routes>
    </Suspense>
  );
}
// Result: 300KB initial bundle, other pages load when needed
```

**Component-based code splitting:**
```jsx
// ❌ BAD: Heavy components loaded immediately
import DataVisualization from './DataVisualization';  // 200KB chart library
import VideoPlayer from './VideoPlayer';              // 300KB video library
import RichTextEditor from './RichTextEditor';        // 400KB editor library

function Dashboard() {
  const [showCharts, setShowCharts] = useState(false);
  const [showVideo, setShowVideo] = useState(false);
  
  return (
    <div>
      <h1>Dashboard</h1>
      {showCharts && <DataVisualization />}      {/* 200KB loaded upfront */}
      {showVideo && <VideoPlayer />}            {/* 300KB loaded upfront */}
      <RichTextEditor />                        {/* 400KB loaded upfront */}
    </div>
  );
}
// Result: 900KB loaded immediately, even if user never uses charts/video

// ✅ GOOD: Heavy components loaded on demand
import { lazy, Suspense } from 'react';

const DataVisualization = lazy(() => import('./DataVisualization'));
const VideoPlayer = lazy(() => import('./VideoPlayer'));
const RichTextEditor = lazy(() => import('./RichTextEditor'));

function Dashboard() {
  const [showCharts, setShowCharts] = useState(false);
  const [showVideo, setShowVideo] = useState(false);
  
  return (
    <div>
      <h1>Dashboard</h1>
      <Suspense fallback={<div>Loading charts...</div>}>
        {showCharts && <DataVisualization />}    {/* 200KB loaded only when needed */}
      </Suspense>
      <Suspense fallback={<div>Loading video...</div>}>
        {showVideo && <VideoPlayer />}          {/* 300KB loaded only when needed */}
      </Suspense>
      <Suspense fallback={<div>Loading editor...</div>}>
        <RichTextEditor />                      {/* 400KB loaded only when needed */}
      </Suspense>
    </div>
  );
}
// Result: Small initial bundle, features load when used
```

80. **Dynamic imports for libraries** - Load heavy libraries only when needed, not upfront.

**Heavy library splitting:**
```jsx
// ❌ BAD: Chart library loaded immediately
import Chart from 'react-chartjs-2';  // 150KB chart library

function Dashboard() {
  const [showChart, setShowChart] = useState(false);
  
  return (
    <div>
      <button onClick={() => setShowChart(true)}>Show Chart</button>
      {showChart && <Chart data={chartData} />}  {/* Library already loaded */}
    </div>
  );
}

// ✅ GOOD: Chart library loaded on demand
function Dashboard() {
  const [showChart, setShowChart] = useState(false);
  const [ChartComponent, setChartComponent] = useState(null);
  
  const loadChart = async () => {
    const { default: Chart } = await import('react-chartjs-2');  // Load when needed
    setChartComponent(() => Chart);
    setShowChart(true);
  };
  
  return (
    <div>
      <button onClick={loadChart}>Show Chart</button>
      {showChart && ChartComponent && <ChartComponent data={chartData} />}
    </div>
  );
}
```

**Conditional feature loading:**
```jsx
// Load different code based on user permissions
function App() {
  const user = useSelector(state => state.user);
  
  const loadUserInterface = async () => {
    if (user.isAdmin) {
      const { AdminDashboard } = await import('./AdminDashboard');    // 500KB admin code
      return AdminDashboard;
    } else {
      const { UserDashboard } = await import('./UserDashboard');      // 100KB user code
      return UserDashboard;
    }
  };
  
  // Only load the code that user actually needs
}

// Load based on device capabilities
function VideoPlayer() {
  const [VideoComponent, setVideoComponent] = useState(null);
  
  useEffect(() => {
    const loadVideo = async () => {
      if (window.innerWidth > 768) {
        // Load full-featured video player for desktop
        const { AdvancedVideoPlayer } = await import('./AdvancedVideoPlayer');
        setVideoComponent(() => AdvancedVideoPlayer);
      } else {
        // Load lightweight player for mobile
        const { SimpleVideoPlayer } = await import('./SimpleVideoPlayer');
        setVideoComponent(() => SimpleVideoPlayer);
      }
    };
    
    loadVideo();
  }, []);
  
  return VideoComponent ? <VideoComponent /> : <div>Loading...</div>;
}
```

81. **Preloading optimization** - Load code before users need it, but don't block initial render.

**Smart preloading strategies:**
```jsx
// Preload on hover (users about to click)
function Navigation() {
  const preloadDashboard = () => {
    // Start loading dashboard code when user hovers
    import('./pages/Dashboard');  // Loads in background
  };
  
  const preloadProfile = () => {
    import('./pages/Profile');    // Loads in background
  };
  
  return (
    <nav>
      <Link 
        to="/dashboard" 
        onMouseEnter={preloadDashboard}  // Preload on hover
      >
        Dashboard
      </Link>
      <Link 
        to="/profile" 
        onMouseEnter={preloadProfile}   // Preload on hover
      >
        Profile
      </Link>
    </nav>
  );
}

// Preload during idle time
function App() {
  useEffect(() => {
    // Preload secondary features when browser is idle
    if ('requestIdleCallback' in window) {
      requestIdleCallback(() => {
        import('./pages/Settings');     // Load when browser is idle
        import('./pages/Help');         // Load when browser is idle
      });
    }
  }, []);
}

// Preload based on user behavior
function HomePage() {
  const [preloadedAdmin, setPreloadedAdmin] = useState(false);
  
  useEffect(() => {
    // If user scrolls down, they're engaged - preload more features
    const handleScroll = () => {
      if (window.scrollY > 500 && !preloadedAdmin) {
        import('./AdminPanel');       // Preload heavy admin code
        setPreloadedAdmin(true);
      }
    };
    
    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, [preloadedAdmin]);
}
```

82. **Library alternatives** - Replace heavy libraries with lighter alternatives. This is often the **biggest bang for your buck**.

**Date libraries:**
```jsx
// ❌ HEAVY: Moment.js (67KB)
import moment from 'moment';
const formatted = moment().format('YYYY-MM-DD');

// ✅ LIGHT: date-fns (5-15KB, tree-shakable)
import { format } from 'date-fns';
const formatted = format(new Date(), 'yyyy-MM-dd');

// ✅ ULTRA-LIGHT: Native JavaScript (0KB)
const formatted = new Date().toLocaleDateString('en-CA');  // YYYY-MM-DD format
```

**Utility libraries:**
```jsx
// ❌ HEAVY: Lodash (70KB)
import _ from 'lodash';

// ✅ LIGHT: Specific lodash functions (2-5KB each)
import debounce from 'lodash/debounce';
import throttle from 'lodash/throttle';

// ✅ ULTRA-LIGHT: Native JavaScript (0KB)
const debounce = (func, delay) => {
  let timeoutId;
  return (...args) => {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => func(...args), delay);
  };
};
```

**HTTP libraries:**
```jsx
// ❌ HEAVY: Axios (15KB)
import axios from 'axios';
const response = await axios.get('/api/data');

// ✅ LIGHT: Native fetch (0KB)
const response = await fetch('/api/data');
const data = await response.json();

// ✅ TINY: Custom fetch wrapper (1KB)
const api = {
  get: (url) => fetch(url).then(r => r.json()),
  post: (url, data) => fetch(url, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data)
  }).then(r => r.json())
};
```

**CSS-in-JS libraries:**
```jsx
// ❌ HEAVY: Styled-components (50KB)
import styled from 'styled-components';
const Button = styled.button`color: blue;`;

// ✅ LIGHT: Emotion (25KB)
import { css } from '@emotion/react';
const buttonStyle = css`color: blue;`;

// ✅ ULTRA-LIGHT: CSS Modules (0KB runtime)
import styles from './Button.module.css';
const Button = () => <button className={styles.button}>Click</button>;
```

**Icon libraries:**
```jsx
// ❌ HEAVY: FontAwesome (200KB+)
import { FontAwesomeIcon } from '@fortawesome/react-fontawesome';
import { faHome, faUser, faSettings } from '@fortawesome/free-solid-svg-icons';

// ✅ LIGHT: Individual icon components (1-2KB each)
import { Home, User, Settings } from 'lucide-react';

// ✅ ULTRA-LIGHT: SVG icons (0.5KB each)
const HomeIcon = () => (
  <svg width="24" height="24" viewBox="0 0 24 24">
    <path d="M10 20v-6h4v6h5v-8h3L12 3 2 12h3v8z"/>
  </svg>
);
```

83. **Asset optimization** - Images, fonts, and other assets can be bundle size killers too.

**Image optimization:**
```jsx
// ❌ BAD: Large images loaded immediately
function ProductGallery() {
  return (
    <div>
      <img src="/images/product1-4k.jpg" alt="Product 1" />  {/* 2MB image */}
      <img src="/images/product2-4k.jpg" alt="Product 2" />  {/* 2MB image */}
      <img src="/images/product3-4k.jpg" alt="Product 3" />  {/* 2MB image */}
    </div>
  );
}

// ✅ GOOD: Optimized images with lazy loading
function ProductGallery() {
  return (
    <div>
      <img 
        src="/images/product1-thumbnail.webp"     // 50KB thumbnail
        loading="lazy"                            // Load when in viewport
        onClick={() => loadFullImage('product1')} // Load full size on click
        alt="Product 1"
      />
      <img 
        src="/images/product2-thumbnail.webp"     // 50KB thumbnail
        loading="lazy"
        onClick={() => loadFullImage('product2')}
        alt="Product 2"
      />
    </div>
  );
}
```

**Font optimization:**
```jsx
// ❌ BAD: Loading entire font families
/*
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@100;300;400;500;700;900&display=swap');
// Loads ALL font weights (500KB+)
*/

// ✅ GOOD: Load only needed font weights
/*
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap');
// Loads only 400 and 700 weights (150KB)
*/

// ✅ BETTER: Use system fonts (0KB)
const systemFontStack = {
  fontFamily: '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif'
};
```

84. **Bundle analysis workflow** - Your step-by-step process to identify and fix bundle bloat.

**The 5-step bundle diet:**
```bash
# Step 1: Measure current bundle size
npm run build
npm run analyze  # or source-map-explorer

# Step 2: Identify the biggest culprits
# Look for:
# - Libraries > 50KB
# - Duplicate dependencies
# - Unused code

# Step 3: Optimize imports
# Replace heavy libraries with lighter alternatives
# Use specific imports instead of full libraries
# Remove unused dependencies

# Step 4: Add code splitting
# Split routes with React.lazy
# Split heavy components
# Use dynamic imports for conditionally loaded code

# Step 5: Measure again
npm run build
npm run analyze
# Celebrate your smaller bundle! 🎉
```

**Bundle size targets:**
```
🎯 BUNDLE SIZE TARGETS:
- Initial bundle: < 200KB (gzipped)
- Total JavaScript: < 500KB (gzipped)
- Time to Interactive: < 3.5 seconds
- First Contentful Paint: < 1.5 seconds

📊 REAL-WORLD COMPARISON:
Small app:     50KB initial,  150KB total
Medium app:   150KB initial,  400KB total  
Large app:    250KB initial,  800KB total
Bloated app:  500KB initial, 2MB+ total 😱
```

**Monitoring bundle size:**
```jsx
// Add bundle size checks to your CI/CD
// package.json
{
  "scripts": {
    "build": "react-scripts build",
    "analyze": "npm run build && npx source-map-explorer build/static/js/*.js",
    "size-check": "npm run build && bundlesize"
  },
  "bundlesize": [
    {
      "path": "./build/static/js/*.js",
      "maxSize": "200kb"
    }
  ]
}

// CI will fail if bundle exceeds 200KB
// This prevents bundle bloat from creeping in
```

85. **Advanced bundle optimization** - Pro techniques for maximum performance.

**Webpack bundle splitting:**
```js
// webpack.config.js (if ejected from CRA)
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
        },
        common: {
          minChunks: 2,
          priority: -10,
          reuseExistingChunk: true,
        },
      },
    },
  },
};

// Results in:
// main.js - Your app code
// vendor.js - Third-party libraries  
// common.js - Shared code between routes
```

**Module concatenation (scope hoisting):**
```js
// webpack.config.js
module.exports = {
  optimization: {
    concatenateModules: true,  // Enables scope hoisting
    // Reduces function call overhead
    // Smaller bundle size
    // Better minification
  }
};
```

**Dead code elimination:**
```jsx
// Webpack removes unreachable code
function MyComponent() {
  if (false) {
    // This code will be removed in production build
    console.log('This will never run');
    const unusedVariable = 'This is eliminated';
  }
  
  return <div>Hello</div>;
}

// Use environment variables for feature flags
function App() {
  return (
    <div>
      <Header />
      {process.env.NODE_ENV === 'development' && <DebugPanel />}
      {/* DebugPanel is completely removed in production */}
      <MainContent />
    </div>
  );
}
```

**Bundle optimization checklist:**
```
✅ BUNDLE OPTIMIZATION CHECKLIST:

📦 IMPORTS:
□ No full library imports (lodash, moment, etc.)
□ Use tree-shakable alternatives
□ Remove unused dependencies
□ Check for duplicate packages

🔗 CODE SPLITTING:
□ Route-based splitting implemented
□ Heavy components split
□ Conditional features split
□ Preloading on user interaction

🎯 ASSETS:
□ Images optimized and lazy loaded
□ Fonts limited to needed weights
□ SVG icons instead of icon fonts
□ Compress static assets

⚡ PERFORMANCE:
□ Bundle < 200KB initial load
□ TTI < 3.5 seconds
□ Bundle analysis in CI/CD
□ Regular bundle size monitoring

🔧 ADVANCED:
□ Service worker for caching
□ HTTP/2 server push
□ Brotli compression
□ Resource hints (preload, prefetch)
```

**The bottom line:** Bundle optimization is **the highest-impact performance improvement** you can make. A 100KB bundle loads in 0.5 seconds on 3G, while a 1MB bundle takes 5+ seconds. Your users will thank you! 🚀
    