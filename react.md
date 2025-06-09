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

46. **configureStore** replaces `createStore` and comes with good defaults including Redux DevTools support.
    ```jsx
    import { configureStore } from '@reduxjs/toolkit';
    
    const store = configureStore({
      reducer: {
        todos: todoSlice.reducer,  // Each slice manages part of the state
        user: userSlice.reducer
      }
    });
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