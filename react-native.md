# React Native Development Guide

## Basic Setup and Project Structure

1. Create a new React Native project using the CLI. This creates all the necessary files and folder structure.
   ```bash
   npx create-expo-app MyApp --template blank
   cd MyApp
   ```
   The `--template blank` flag creates a minimal project without extra features. Expo is the recommended way to start because it handles native dependencies automatically.

   We use `npx` instead of `npm` because:
   - `npx` executes packages directly without installing them globally
   - It ensures you always use the latest version of create-expo-app
   - Avoids cluttering your global npm packages
   - Prevents version conflicts between projects
   
   If we used `npm`, we would need to first install create-expo-app globally (`npm install -g create-expo-app`) and then run it. This could lead to using outdated versions if we don't regularly update the global package.

   # Install packages using npm
   ```bash
   npm install package-name    # Install a package and save to package.json
   npm install --save-dev package-name  # Install dev dependency
   ```
   We use `npm` (not `npx`) to install packages because:
   - `npm install` adds the package as a project dependency in package.json
   - It downloads the package files into node_modules
   - Allows version control through package.json
   - Enables other developers to install all dependencies with `npm install`

   Common packages you'll need:
   ```bash
   npm install @react-navigation/native  # Navigation between screens
   npm install @react-native-async-storage/async-storage  # Local storage
   npm install axios  # Making HTTP requests
   ```

   For Expo-specific packages, use the `expo install` command:
   ```bash 
   expo install expo-camera  # Camera access
   expo install expo-location  # GPS/location
   ```
   `expo install` ensures you get versions compatible with your Expo SDK version.

   Remember to restart your development server after installing new packages:
   ```bash
   # Stop current server with Ctrl+C, then
   npx expo start --clear
   ```

2. Your main entry point is `App.js`. This is where your entire app starts running.
   ```javascript
   import { StatusBar } from 'expo-status-bar';
   import { StyleSheet, Text, View } from 'react-native';

   export default function App() {
     return (
       <View style={styles.container}>
         <Text>Hello World!</Text>
         <StatusBar style="auto" />
       </View>
     );
   }

   const styles = StyleSheet.create({
     container: {
       flex: 1,  // flex: 1 makes the container take up all available space
       backgroundColor: '#fff',
       alignItems: 'center',     // centers children horizontally
       justifyContent: 'center', // centers children vertically
     },
   });
   ```
   - `View` is like a `div` in web development - it's a container for other components.
   - `Text` displays text. In React Native, you MUST use `Text` for any text, you can't just put text directly in a `View`.
   - `StyleSheet.create()` creates optimized styles. It's better than inline styles for performance.
   - `StatusBar` controls the app's status bar (the bar at the top of the phone showing time, battery, etc).
     - `style="auto"` automatically switches between light and dark text based on background color
     - Other options are:
       ```javascript
       <StatusBar 
         style="light" // Forces white text
         style="dark"  // Forces dark text
         hidden={true} // Hides the status bar completely
       />
       ```
     - This is important for making your app look polished and ensuring status bar content remains readable

3. To run your app, use these commands:
   ```bash
   npx expo start           # Starts the development server
   ```
   This opens a QR code you can scan with the Expo Go app on your phone, or you can press 'i' for iOS simulator or 'a' for Android emulator.

   # Run in web browser
   npx expo start --web        # Opens in your default browser
   ```
   - This requires the `@expo/webpack-config` package which is included by default in new Expo projects
   - Web support lets you develop and test your app in a browser which can be faster than using simulators
   - Not all React Native components work on web - you may need web-specific alternatives
   - Common web limitations:
     ```javascript
     // These work differently or not at all on web:
     - StatusBar component
     - Some touch gestures
     - Platform-specific APIs
     ```
   - Use Platform.select() for web-specific code:
     ```javascript
     import { Platform } from 'react-native';
     
     const styles = StyleSheet.create({
       container: {
         ...Platform.select({
           web: {
             // Web-specific styles
             cursor: 'pointer',
           },
           default: {
             // Native styles
           }
         })
       }
     });
     ```

   # Install web dependencies
   ```bash
   npx expo install react-dom react-native-web @expo/metro-runtime
   ```
   - `react-dom` is needed to render React components in the browser
   - `react-native-web` translates React Native components to web equivalents
   - `@expo/metro-runtime` provides the development runtime for web support
   
   If you don't plan to use web support, you can remove "web" from your project config:
   ```javascript
   // app.json or app.config.js
   {
     "expo": {
       "platforms": ["ios", "android"] // Remove "web" if not using
     }
   }
   ```
   # Web packages in Android builds
   When you build your app for Android, the web-specific packages (`react-dom`, `react-native-web`, `@expo/metro-runtime`) are not included in the final APK/AAB bundle. This is because:

   1. Metro bundler (React Native's bundler) only includes the code paths that are actually used
   2. Platform-specific code is eliminated during build time using:
      ```javascript
      Platform.select({
        android: { /* android code */ },
        ios: { /* ios code */ },
        web: { /* web code - excluded in native builds */ }
      })
      ```
   3. Tree shaking removes unused dependencies
   
   So having web packages installed won't increase your Android app size. They're only used when running in web mode.

## Core Components You'll Use Daily

### View - The Basic Container
`View` is your basic building block. It's like a `div` but for mobile.
```javascript
import { View } from 'react-native';

<View style={styles.container}>
  <Text>Content goes here</Text>
</View>
```

### Text - Displaying Text
All text must be wrapped in `Text` components. You cannot put raw text directly in a `View`.
```javascript
import { Text } from 'react-native';

<Text style={styles.title}>My App Title</Text>
<Text style={styles.body}>This is body text</Text>
```

### Image - Displaying Images
Use `Image` for both local and remote images.
```javascript
import { Image } from 'react-native';

// Local image from your project
<Image 
  source={require('./assets/logo.png')} 
  style={styles.logo} 
/>

// Remote image from internet
<Image 
  source={{uri: 'https://example.com/image.jpg'}} 
  style={styles.profilePic} 
/>
```
- `source` is the image location. Use `require()` for local files, `{uri: 'url'}` for remote images.
- Always provide a `style` with width and height, or the image might not show.

### TouchableOpacity - Making Things Clickable
This makes any component clickable. It's like a `button` but you can put anything inside it.
```javascript
import { TouchableOpacity, Text } from 'react-native';

<TouchableOpacity 
  style={styles.button} 
  onPress={() => console.log('Button pressed!')}
>
  <Text style={styles.buttonText}>Click Me</Text>
</TouchableOpacity>
```
- `onPress` is the function that runs when user taps.
- The component becomes slightly transparent when pressed (that's the "Opacity" part).

### TextInput - Getting User Input
For text input fields like username, password, etc.
```javascript
import { TextInput } from 'react-native';
import { useState } from 'react';

function MyComponent() {
  const [text, setText] = useState(''); // text holds the current input value
  
  return (
    <TextInput
      style={styles.input}
      placeholder="Enter your name"     // gray text shown when empty
      value={text}                      // current value of the input
      onChangeText={setText}            // function called when user types
      autoCapitalize="none"             // don't auto-capitalize first letter
      keyboardType="email-address"     // shows email keyboard on mobile
    />
  );
}
```
- `value` and `onChangeText` make this a "controlled component" - React controls the input value.
- `placeholder` is the gray hint text shown when the input is empty.
- `keyboardType` changes the mobile keyboard (email, numeric, phone, etc.).

## Layout with Flexbox

React Native uses Flexbox for layouts, similar to CSS but with some differences.

### Basic Flex Layout
```javascript
const styles = StyleSheet.create({
  container: {
    flex: 1,                    // takes all available space
    flexDirection: 'column',    // children stack vertically (default)
    justifyContent: 'center',   // centers children along main axis (vertical)
    alignItems: 'center',       // centers children along cross axis (horizontal)
  },
  
  row: {
    flexDirection: 'row',       // children line up horizontally
    justifyContent: 'space-between', // spreads children apart
  },
});
```
- `flex: 1` makes a component expand to fill available space.
- `flexDirection` controls if children stack vertically ('column') or horizontally ('row').
- `justifyContent` controls spacing along the main axis.
- `alignItems` controls alignment along the cross axis.

### Common Layout Examples
```javascript
// Header with title and button
<View style={styles.header}>
  <Text style={styles.title}>My App</Text>
  <TouchableOpacity style={styles.headerButton}>
    <Text>Settings</Text>
  </TouchableOpacity>
</View>

const styles = StyleSheet.create({
  header: {
    flexDirection: 'row',           // title and button side by side
    justifyContent: 'space-between', // title on left, button on right
    alignItems: 'center',           // vertically center both
    padding: 20,
    backgroundColor: '#f0f0f0',
  },
});

// Card layout
<View style={styles.card}>
  <Image source={{uri: 'image-url'}} style={styles.cardImage} />
  <View style={styles.cardContent}>
    <Text style={styles.cardTitle}>Card Title</Text>
    <Text style={styles.cardDescription}>Description text here</Text>
  </View>
</View>

const cardStyles = StyleSheet.create({
  card: {
    backgroundColor: 'white',
    borderRadius: 8,          // rounded corners
    padding: 16,
    margin: 10,
    shadowColor: '#000',      // drop shadow (iOS)
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,             // drop shadow (Android)
  },
  cardImage: {
    width: '100%',
    height: 200,
    borderRadius: 8,
  },
  cardContent: {
    marginTop: 12,
  },
});
```

## State Management with useState

State lets you store and update data that changes over time.

### Basic State Example
```javascript
import { useState } from 'react';
import { View, Text, TouchableOpacity } from 'react-native';

function Counter() {
  const [count, setCount] = useState(0); // count starts at 0
  
  return (
    <View>
      <Text>Count: {count}</Text>
      <TouchableOpacity onPress={() => setCount(count + 1)}>
        <Text>Increment</Text>
      </TouchableOpacity>
      <TouchableOpacity onPress={() => setCount(count - 1)}>
        <Text>Decrement</Text>
      </TouchableOpacity>
    </View>
  );
}
```
- `useState(0)` creates state variable `count` starting at 0 and `setCount` function to update it.
- When you call `setCount`, the component re-renders with the new value.
- Use curly braces `{count}` to display JavaScript variables inside JSX.

### State with Objects and Arrays
```javascript
function TodoList() {
  const [todos, setTodos] = useState([]);    // array of todo items
  const [newTodo, setNewTodo] = useState(''); // text for new todo
  
  const addTodo = () => {
    if (newTodo.trim()) {  // only add if text is not empty
      setTodos([...todos, { id: Date.now(), text: newTodo, done: false }]);
      setNewTodo(''); // clear input after adding
    }
  };
  
  const toggleTodo = (id) => {
    setTodos(todos.map(todo => 
      todo.id === id ? {...todo, done: !todo.done} : todo
    ));
  };
  
  return (
    <View>
      <TextInput
        value={newTodo}
        onChangeText={setNewTodo}
        placeholder="Add new todo"
      />
      <TouchableOpacity onPress={addTodo}>
        <Text>Add</Text>
      </TouchableOpacity>
      
      {todos.map(todo => (
        <TouchableOpacity key={todo.id} onPress={() => toggleTodo(todo.id)}>
          <Text style={todo.done ? styles.done : styles.pending}>
            {todo.text}
          </Text>
        </TouchableOpacity>
      ))}
    </View>
  );
}
```
- `[...todos, newItem]` spreads the existing array and adds a new item (don't mutate state directly).
- `map()` creates a new array by transforming each item.
- `{...todo, done: !todo.done}` spreads the object and updates one property.

## Navigation Between Screens

Use React Navigation to move between different screens in your app.

### Setup Navigation
First install the required packages:
```bash
npm install @react-navigation/native @react-navigation/native-stack
npx expo install react-native-screens react-native-safe-area-context
```

### Basic Stack Navigation
```javascript
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';

const Stack = createNativeStackNavigator();
// createNativeStackNavigator() creates a stack navigator object that manages screen transitions
// Stack.Navigator and Stack.Screen are components from this object
// This pattern is used because:
// 1. It creates a navigation stack that mimics how users expect mobile apps to work
// 2. Each screen is pushed/popped from the stack, maintaining navigation history
// 3. Provides built-in back button and swipe gestures on iOS
// 4. Handles screen transitions and animations automatically
// 5. Manages the navigation state internally

// Example of what happens in a stack:
// Initial: [Home]
// Navigate to Details: [Home, Details] 
// Go back: [Home]
// The last screen in the array is what the user sees


// Your screens
function HomeScreen({ navigation }) {
  return (
    <View style={styles.center}>
      <Text>Home Screen</Text>
      <TouchableOpacity 
        onPress={() => navigation.navigate('Details', { userName: 'John' })}
      >
        <Text>Go to Details</Text>
      </TouchableOpacity>
    </View>
  );
}

function DetailsScreen({ route, navigation }) {
  const { userName } = route.params; // get data passed from previous screen
  
  return (
    <View style={styles.center}>
      <Text>Details Screen</Text>
      <Text>Hello {userName}!</Text>
      <TouchableOpacity onPress={() => navigation.goBack()}>
        <Text>Go Back</Text>
      </TouchableOpacity>
    </View>
  );
}

// Main App with navigation
export default function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator initialRouteName="Home">
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Details" component={DetailsScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```
- `NavigationContainer` wraps your entire app navigation.
- `Stack.Navigator` creates a stack of screens (like pages in a book).
- `navigation.navigate('ScreenName', {data})` moves to another screen and passes data.
- `route.params` contains data passed from the previous screen.
- `navigation.goBack()` returns to the previous screen.


### Tab Navigation
For bottom tab navigation like Instagram or Twitter:
```bash
npm install @react-navigation/bottom-tabs
```

```javascript
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { Ionicons } from '@expo/vector-icons';

const Tab = createBottomTabNavigator();

function TabNavigation() {
  return (
    <Tab.Navigator
      screenOptions={({ route }) => ({
        tabBarIcon: ({ focused, color, size }) => {
          let iconName;
          if (route.name === 'Home') {
            iconName = focused ? 'home' : 'home-outline';
          } else if (route.name === 'Profile') {
            iconName = focused ? 'person' : 'person-outline';
          }
          return <Ionicons name={iconName} size={size} color={color} />;
        },
        tabBarActiveTintColor: 'tomato',   // color when tab is selected
        tabBarInactiveTintColor: 'gray',   // color when tab is not selected
      })}
    >
      <Tab.Screen name="Home" component={HomeScreen} />
      <Tab.Screen name="Profile" component={ProfileScreen} />
    </Tab.Navigator>
  );
}
```
- `tabBarIcon` function controls what icon shows for each tab.
- `focused` parameter tells you if the tab is currently selected.
- `tabBarActiveTintColor` and `tabBarInactiveTintColor` control tab colors.

### Why tabBarIcon exists?

The `tabBarIcon` function is a crucial part of tab navigation that:

1. **Customizes Icons:** Lets you define custom icons for each tab route, making navigation visually intuitive.
   ```javascript
   tabBarIcon: ({ focused, color, size }) => {
     return <Ionicons name={focused ? 'home' : 'home-outline'} size={size} color={color} />;
   }
   ```

2. **Handles Tab States:** Receives a `focused` parameter to show different icons/styles when a tab is selected vs unselected.

3. **Maintains Consistency:** Gets `color` and `size` parameters from the navigator to keep icons visually consistent with the navigation theme.

We can't directly provide icons in the Tab.Screen element because the tab navigation system needs dynamic control over the icons based on the tab's state. The tabBarIcon function allows us to:

1. Dynamically switch icons based on whether the tab is focused or not
2. Apply consistent sizing and coloring from the navigator's theme
3. Handle the icon's appearance across different states (focused/unfocused)

If we tried to put icons directly in Tab.Screen, we'd lose this dynamic control and couldn't respond to the tab's changing states. The function approach gives us the flexibility needed for a proper tab navigation experience.


## Styling Your App

React Native uses StyleSheet for styling, similar to CSS but with some differences.

### Basic Styling
```javascript
import { StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
    padding: 20,
  },
  
  title: {
    fontSize: 24,           // font size in logical pixels
    fontWeight: 'bold',     // can be 'normal', 'bold', '100'-'900'
    color: '#333',          // text color
    marginBottom: 20,       // space below the element
    textAlign: 'center',    // 'left', 'center', 'right'
  },
  
  button: {
    backgroundColor: '#007AFF', // background color
    paddingHorizontal: 20,      // padding on left and right
    paddingVertical: 12,        // padding on top and bottom
    borderRadius: 8,            // rounded corners
    alignItems: 'center',       // center the text inside
  },
  
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: '600',
  },
});
```

### Layout Properties
```javascript
const layoutStyles = StyleSheet.create({
  // Position and size
  positioned: {
    position: 'absolute',    // takes element out of normal flow
    top: 50,                 // distance from top
    left: 20,                // distance from left
    width: 100,              // fixed width
    height: 50,              // fixed height
  },
  
  // Margins (space outside the element)
  margins: {
    margin: 10,              // all sides
    marginTop: 5,            // just top
    marginHorizontal: 15,    // left and right
    marginVertical: 10,      // top and bottom
  },
  
  // Padding (space inside the element)
  paddings: {
    padding: 20,             // all sides
    paddingLeft: 10,         // just left
    paddingHorizontal: 15,   // left and right
    paddingVertical: 8,      // top and bottom
  },
  
  // Borders
  bordered: {
    borderWidth: 1,          // width of border
    borderColor: '#ddd',     // color of border
    borderRadius: 8,         // rounded corners
    borderTopWidth: 2,       // different width for top border
  },
});
```

### Responsive Design
```javascript
import { Dimensions } from 'react-native';

const screenWidth = Dimensions.get('window').width;  // device screen width
const screenHeight = Dimensions.get('window').height; // device screen height

const responsiveStyles = StyleSheet.create({
  card: {
    width: screenWidth * 0.9,    // 90% of screen width
    maxWidth: 400,               // but never more than 400px
    minHeight: 200,              // at least 200px tall
  },
  
  // For tablets vs phones
  container: {
    padding: screenWidth > 768 ? 40 : 20, // more padding on tablets
  },
});
```

## Lists and Data Display

### FlatList - Efficient Lists
Use `FlatList` for displaying lists of data. It's optimized for performance with large lists.
```javascript
import { FlatList } from 'react-native';

function UserList() {
  const [users, setUsers] = useState([
    { id: '1', name: 'John Doe', email: 'john@example.com' },
    { id: '2', name: 'Jane Smith', email: 'jane@example.com' },
    // ... more users
  ]);
  
  const renderUser = ({ item }) => (
    <View style={styles.userCard}>
      <Text style={styles.userName}>{item.name}</Text>
      <Text style={styles.userEmail}>{item.email}</Text>
    </View>
  );
  
  return (
    <FlatList
      data={users}                    // array of data to display
      renderItem={renderUser}         // function to render each item
      keyExtractor={(item) => item.id} // unique key for each item
      ItemSeparatorComponent={() => <View style={styles.separator} />}
      refreshing={false}              // show refresh spinner
      onRefresh={() => {}}            // function called when user pulls to refresh
      onEndReached={() => {}}         // function called when user scrolls to bottom
      onEndReachedThreshold={0.1}     // how close to bottom before calling onEndReached
    />
  );
}
```
- `data` is the array of items to display.
- `renderItem` is a function that gets called for each item with `{item, index}`.
- `keyExtractor` returns a unique string for each item (important for performance).
- `onEndReached` is useful for infinite scrolling - load more data when user reaches the bottom.

### ScrollView vs FlatList
```javascript
// ScrollView - use for small lists or complex layouts
<ScrollView style={styles.container}>
  <Text>Header</Text>
  <Image source={...} />
  <Text>Some content</Text>
  {/* Mix of different components */}
</ScrollView>

// FlatList - use for large lists of similar items
<FlatList
  data={largeDataArray}
  renderItem={renderItem}
  keyExtractor={keyExtractor}
/>
```
- `ScrollView` renders all children at once. Good for small lists or mixed content.
- `FlatList` only renders visible items. Much better for large lists (hundreds/thousands of items).

## Making API Calls

Use fetch (built-in) or install axios for HTTP requests.

### Basic Fetch Examples
```javascript
import { useEffect, useState } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);     // holds user data
  const [loading, setLoading] = useState(true); // shows loading state
  const [error, setError] = useState(null);   // holds error message
  
  // Fetch data when component mounts
  useEffect(() => {
    fetchUser();
  }, [userId]); // re-run when userId changes
  
  const fetchUser = async () => {
    try {
      setLoading(true);
      setError(null);
      
      const response = await fetch(`https://api.example.com/users/${userId}`);
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      const userData = await response.json();
      setUser(userData);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };
  
  if (loading) {
    return <Text>Loading...</Text>;
  }
  
  if (error) {
    return (
      <View>
        <Text>Error: {error}</Text>
        <TouchableOpacity onPress={fetchUser}>
          <Text>Retry</Text>
        </TouchableOpacity>
      </View>
    );
  }
  
  return (
    <View>
      <Text>{user.name}</Text>
      <Text>{user.email}</Text>
    </View>
  );
}
```

### POST Request Example
```javascript
const createUser = async (userData) => {
  try {
    const response = await fetch('https://api.example.com/users', {
      method: 'POST',                    // HTTP method
      headers: {
        'Content-Type': 'application/json', // tells server we're sending JSON
        'Authorization': 'Bearer ' + token, // if you need authentication
      },
      body: JSON.stringify(userData),    // convert object to JSON string
    });
    
    if (!response.ok) {
      throw new Error('Failed to create user');
    }
    
    const newUser = await response.json();
    return newUser;
  } catch (error) {
    console.error('Error creating user:', error);
    throw error;
  }
};

// Usage in a component
function CreateUserForm() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [loading, setLoading] = useState(false);
  
  const handleSubmit = async () => {
    setLoading(true);
    try {
      await createUser({ name, email });
      // Success - clear form or navigate away
      setName('');
      setEmail('');
      alert('User created successfully!');
    } catch (error) {
      alert('Failed to create user');
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <View>
      <TextInput
        value={name}
        onChangeText={setName}
        placeholder="Name"
      />
      <TextInput
        value={email}
        onChangeText={setEmail}
        placeholder="Email"
        keyboardType="email-address"
      />
      <TouchableOpacity 
        onPress={handleSubmit} 
        disabled={loading}      // disable button while loading
      >
        <Text>{loading ? 'Creating...' : 'Create User'}</Text>
      </TouchableOpacity>
    </View>
  );
}
```

## Common Patterns and Best Practices

### Loading States
Always show loading states for better user experience:
```javascript
function DataComponent() {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  
  if (loading) {
    return (
      <View style={styles.center}>
        <ActivityIndicator size="large" color="#0000ff" />
        <Text>Loading...</Text>
      </View>
    );
  }
  
  return (
    <View>
      {/* Your data display */}
    </View>
  );
}
```
- `ActivityIndicator` shows a spinning loader (like a spinner).
- Always provide feedback when something is loading.

### Error Boundaries and Error Handling
```javascript
function SafeComponent({ children }) {
  const [hasError, setHasError] = useState(false);
  
  if (hasError) {
    return (
      <View style={styles.errorContainer}>
        <Text>Something went wrong!</Text>
        <TouchableOpacity onPress={() => setHasError(false)}>
          <Text>Try Again</Text>
        </TouchableOpacity>
      </View>
    );
  }
  
  return children;
}
```

### Custom Hooks for Reusable Logic
```javascript
// Custom hook for API calls
function useApi(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetch(url)
      .then(response => response.json())
      .then(setData)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [url]);
  
  return { data, loading, error };
}

// Usage in components
function UserList() {
  const { data: users, loading, error } = useApi('/api/users');
  
  if (loading) return <Text>Loading...</Text>;
  if (error) return <Text>Error: {error.message}</Text>;
  
  return (
    <FlatList
      data={users}
      renderItem={({ item }) => <Text>{item.name}</Text>}
      keyExtractor={item => item.id}
    />
  );
}
```

### Performance Tips
1. **Use FlatList for large lists** instead of ScrollView with map.
2. **Optimize images** - specify width and height.
3. **Use useMemo and useCallback** for expensive calculations.
4. **Avoid inline functions** in render methods.

```javascript
// Bad - creates new function on every render
<TouchableOpacity onPress={() => handlePress(item.id)}>

// Good - use useCallback
const handlePress = useCallback((id) => {
  // handle press
}, []);

<TouchableOpacity onPress={() => handlePress(item.id)}>
```

## App Lifecycle Management

Managing your app's lifecycle is crucial for providing a good user experience and handling background tasks properly.

### App States and Transitions

React Native apps have three main states:
- **active**: App is running in the foreground and receiving events
- **background**: App is running in the background (user switched to another app)
- **inactive**: App is transitioning between foreground and background (like when receiving a phone call)

### Using AppState to Track Lifecycle

The `AppState` API lets you listen to app state changes:

```javascript
import { AppState } from 'react-native';
import { useEffect, useRef } from 'react';

function App() {
  const appState = useRef(AppState.currentState);
  const [appStateVisible, setAppStateVisible] = useState(appState.current);

  useEffect(() => {
    const subscription = AppState.addEventListener('change', nextAppState => {
      if (
        appState.current.match(/inactive|background/) &&
        nextAppState === 'active'
      ) {
        console.log('App has come to the foreground!');
        // App resumed from background
        onAppResume();
      } else if (
        appState.current === 'active' &&
        nextAppState.match(/inactive|background/)
      ) {
        console.log('App has gone to the background!');
        // App went to background
        onAppPause();
      }

      appState.current = nextAppState;
      setAppStateVisible(appState.current);
      console.log('AppState', appState.current);
    });

    return () => {
      subscription?.remove();
    };
  }, []);

  const onAppResume = () => {
    // Actions to perform when app comes to foreground
    // - Refresh data
    // - Resume timers
    // - Re-authenticate user if needed
    // - Resume audio/video playback
    console.log('App resumed - refreshing data...');
    fetchLatestData();
  };

  const onAppPause = () => {
    // Actions to perform when app goes to background
    // - Save current state
    // - Pause timers
    // - Pause audio/video
    // - Clear sensitive data from memory
    console.log('App paused - saving state...');
    saveCurrentState();
  };

  return (
    <View>
      <Text>Current state is: {appStateVisible}</Text>
    </View>
  );
}
```
- `AppState.addEventListener('change', callback)` listens for state changes.
- `AppState.currentState` gives you the current state immediately.
- Always clean up listeners in the `useEffect` cleanup function to prevent memory leaks.

### Custom Hook for App Lifecycle

Create a reusable hook for app lifecycle management:

```javascript
import { useEffect, useRef } from 'react';
import { AppState } from 'react-native';

function useAppState(onForeground, onBackground) {
  const appState = useRef(AppState.currentState);

  useEffect(() => {
    const subscription = AppState.addEventListener('change', nextAppState => {
      if (
        appState.current.match(/inactive|background/) &&
        nextAppState === 'active'
      ) {
        // App came to foreground
        onForeground?.();
      } else if (
        appState.current === 'active' &&
        nextAppState.match(/inactive|background/)
      ) {
        // App went to background
        onBackground?.();
      }

      appState.current = nextAppState;
    });

    return () => subscription?.remove();
  }, [onForeground, onBackground]);

  return appState.current;
}

// Usage in components
function MyComponent() {
  const [data, setData] = useState(null);

  useAppState(
    () => {
      // App resumed - refresh data
      console.log('App resumed, refreshing data...');
      fetchData().then(setData);
    },
    () => {
      // App paused - save state
      console.log('App paused, saving state...');
      saveToStorage(data);
    }
  );

  return <Text>Data: {data}</Text>;
}
```

### Background Tasks

#### Background Task Registration
For tasks that need to complete even when the app goes to background (like uploading a file):

```javascript
import { AppState } from 'react-native';
import * as TaskManager from 'expo-task-manager';
import * as BackgroundFetch from 'expo-background-fetch';

// Register a background task
const BACKGROUND_FETCH_TASK = 'background-fetch';

TaskManager.defineTask(BACKGROUND_FETCH_TASK, async () => {
  const now = Date.now();
  
  console.log(`Got background fetch call at date: ${new Date(now).toISOString()}`);
  
  // Perform your background work here
  try {
    // Example: Sync data with server
    await syncDataWithServer();
    
    // Example: Check for new messages
    const newMessages = await checkForNewMessages();
    if (newMessages.length > 0) {
      // Show local notification
      await showNotification('You have new messages!');
    }
    
    return BackgroundFetch.BackgroundFetchResult.NewData;
  } catch (error) {
    console.error('Background fetch failed:', error);
    return BackgroundFetch.BackgroundFetchResult.Failed;
  }
});

// Register the background fetch task
async function registerBackgroundFetchAsync() {
  return BackgroundFetch.registerTaskAsync(BACKGROUND_FETCH_TASK, {
    minimumInterval: 15 * 60, // 15 minutes (minimum allowed)
    stopOnTerminate: false,   // Continue even if app is terminated
    startOnBoot: true,        // Start when device boots
  });
}

// Unregister background fetch
async function unregisterBackgroundFetchAsync() {
  return BackgroundFetch.unregisterTaskAsync(BACKGROUND_FETCH_TASK);
}

// Use in your app
function App() {
  useEffect(() => {
    registerBackgroundFetchAsync();
    
    return () => {
      unregisterBackgroundFetchAsync();
    };
  }, []);
  
  return <YourAppContent />;
}
```
- `TaskManager.defineTask()` defines what should happen during background execution.
- `BackgroundFetch.registerTaskAsync()` registers the task with the system.
- `minimumInterval` sets how often the task should run (minimum 15 minutes).
- `stopOnTerminate: false` keeps the task running even when app is closed.

#### Background App Refresh and Permissions

Before using background tasks, you need to request permission:

```javascript
import * as BackgroundFetch from 'expo-background-fetch';

async function requestBackgroundPermissions() {
  const { status } = await BackgroundFetch.requestPermissionsAsync();
  
  if (status !== 'granted') {
    alert('Background app refresh permission is required for this feature');
    return false;
  }
  
  return true;
}

// Check if background fetch is available
async function checkBackgroundFetchStatus() {
  const status = await BackgroundFetch.getStatusAsync();
  
  switch (status) {
    case BackgroundFetch.BackgroundFetchStatus.Restricted:
      console.log('Background execution is disabled');
      break;
    case BackgroundFetch.BackgroundFetchStatus.Denied:
      console.log('Background execution is denied');
      break;
    case BackgroundFetch.BackgroundFetchStatus.Available:
      console.log('Background execution is available');
      break;
  }
  
  return status;
}
```

### Long-Running Background Services

For services that need to run continuously in the background (like music players, location tracking):

#### Location Tracking Service
```javascript
import * as Location from 'expo-location';
import * as TaskManager from 'expo-task-manager';

const LOCATION_TASK_NAME = 'background-location-task';

// Define the background location task
TaskManager.defineTask(LOCATION_TASK_NAME, ({ data, error }) => {
  if (error) {
    console.error('Location task error:', error);
    return;
  }
  
  if (data) {
    const { locations } = data;
    console.log('Received new locations', locations);
    
    // Process location data
    locations.forEach(location => {
      // Send to server, save locally, etc.
      sendLocationToServer(location);
    });
  }
});

// Start background location tracking
async function startLocationTracking() {
  // Request permissions
  const { status: foregroundStatus } = await Location.requestForegroundPermissionsAsync();
  if (foregroundStatus !== 'granted') {
    console.log('Foreground location permission denied');
    return;
  }

  const { status: backgroundStatus } = await Location.requestBackgroundPermissionsAsync();
  if (backgroundStatus !== 'granted') {
    console.log('Background location permission denied');
    return;
  }

  // Start tracking
  await Location.startLocationUpdatesAsync(LOCATION_TASK_NAME, {
    accuracy: Location.Accuracy.Balanced,
    timeInterval: 15000,        // Update every 15 seconds
    distanceInterval: 0,        // Update for any distance change
    foregroundService: {
      notificationTitle: 'Using your location',
      notificationBody: 'To provide better experience, we track your location in background.',
    },
  });
}

// Stop location tracking
async function stopLocationTracking() {
  await Location.stopLocationUpdatesAsync(LOCATION_TASK_NAME);
}
```
- `foregroundService` creates a persistent notification on Android so the service can run.
- Background location requires special permissions and user consent.
- iOS has stricter background location policies than Android.

#### Music Player Background Service
```javascript
import { Audio } from 'expo-av';
import * as MediaLibrary from 'expo-media-library';

function MusicPlayer() {
  const [sound, setSound] = useState(null);
  const [isPlaying, setIsPlaying] = useState(false);

  useEffect(() => {
    // Configure audio session for background playback
    Audio.setAudioModeAsync({
      allowsRecordingIOS: false,
      staysActiveInBackground: true,    // Keep playing in background
      interruptionModeIOS: Audio.INTERRUPTION_MODE_IOS_DO_NOT_MIX,
      playsInSilentModeIOS: true,      // Play even when phone is on silent
      shouldDuckAndroid: true,         // Lower volume when other apps need audio
      interruptionModeAndroid: Audio.INTERRUPTION_MODE_ANDROID_DO_NOT_MIX,
      playThroughEarpieceAndroid: false,
    });

    return () => {
      // Cleanup when component unmounts
      if (sound) {
        sound.unloadAsync();
      }
    };
  }, []);

  const playSound = async (uri) => {
    try {
      const { sound: newSound } = await Audio.Sound.createAsync(
        { uri },
        { shouldPlay: true, isLooping: false },
        onPlaybackStatusUpdate
      );
      
      setSound(newSound);
      setIsPlaying(true);
    } catch (error) {
      console.error('Error playing sound:', error);
    }
  };

  const onPlaybackStatusUpdate = (status) => {
    if (status.isLoaded) {
      setIsPlaying(status.isPlaying);
      
      if (status.didJustFinish) {
        // Song finished, play next song
        playNextSong();
      }
    }
  };

  const pauseSound = async () => {
    if (sound) {
      await sound.pauseAsync();
      setIsPlaying(false);
    }
  };

  const resumeSound = async () => {
    if (sound) {
      await sound.playAsync();
      setIsPlaying(true);
    }
  };

  return (
    <View>
      <TouchableOpacity onPress={isPlaying ? pauseSound : resumeSound}>
        <Text>{isPlaying ? 'Pause' : 'Play'}</Text>
      </TouchableOpacity>
    </View>
  );
}
```
- `staysActiveInBackground: true` allows audio to continue playing when app is backgrounded.
- `playsInSilentModeIOS: true` ensures music plays even when phone is on silent mode.
- Handle audio interruptions (like phone calls) properly.

### Push Notifications for Background Communication

For communicating with users when your app is in background:

```javascript
import * as Notifications from 'expo-notifications';

// Configure notification behavior
Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
    shouldPlaySound: false,
    shouldSetBadge: false,
  }),
});

function NotificationManager() {
  const [expoPushToken, setExpoPushToken] = useState('');

  useEffect(() => {
    registerForPushNotificationsAsync().then(token => setExpoPushToken(token));

    // Listen for notifications when app is in foreground
    const subscription = Notifications.addNotificationReceivedListener(notification => {
      console.log('Notification received:', notification);
      // Handle notification while app is open
    });

    // Listen for notification interactions (when user taps notification)
    const responseSubscription = Notifications.addNotificationResponseReceivedListener(response => {
      console.log('Notification tapped:', response);
      // Navigate to specific screen, update state, etc.
      handleNotificationResponse(response);
    });

    return () => {
      subscription.remove();
      responseSubscription.remove();
    };
  }, []);

  return null; // This component doesn't render anything
}

async function registerForPushNotificationsAsync() {
  let token;
  
  const { status: existingStatus } = await Notifications.getPermissionsAsync();
  let finalStatus = existingStatus;
  
  if (existingStatus !== 'granted') {
    const { status } = await Notifications.requestPermissionsAsync();
    finalStatus = status;
  }
  
  if (finalStatus !== 'granted') {
    alert('Failed to get push token for push notification!');
    return;
  }
  
  token = (await Notifications.getExpoPushTokenAsync()).data;
  console.log('Push token:', token);
  
  return token;
}

// Schedule local notifications (doesn't require server)
async function scheduleLocalNotification() {
  await Notifications.scheduleNotificationAsync({
    content: {
      title: "You've got mail! 📬",
      body: 'Here is the notification body',
      data: { someData: 'goes here' },
    },
    trigger: { seconds: 2 }, // Show after 2 seconds
  });
}

// Send push notification from your server
async function sendPushNotification(expoPushToken, title, body) {
  const message = {
    to: expoPushToken,
    sound: 'default',
    title: title,
    body: body,
    data: { someData: 'goes here' },
  };

  await fetch('https://exp.host/--/api/v2/push/send', {
    method: 'POST',
    headers: {
      Accept: 'application/json',
      'Accept-encoding': 'gzip, deflate',
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(message),
  });
}
```

### Platform-Specific Considerations

#### iOS Background App Refresh
```javascript
// iOS requires Background App Refresh to be enabled in Settings
// Check if it's available:
import { AppState } from 'react-native';

const checkiOSBackgroundRefresh = () => {
  if (Platform.OS === 'ios') {
    // iOS users need to enable Background App Refresh in Settings
    // You can guide them there but can't enable it programmatically
    console.log('For iOS: Ensure Background App Refresh is enabled in Settings > General > Background App Refresh');
  }
};
```

#### Android Background Services
```javascript
// Android has stricter background execution limits since API 26
// For long-running services, you need foreground services with notifications

import { Platform } from 'react-native';

const createAndroidForegroundService = () => {
  if (Platform.OS === 'android') {
    // This creates a persistent notification that allows background execution
    const foregroundService = {
      taskName: 'MyBackgroundTask',
      taskTitle: 'My App is running',
      taskDesc: 'Keeping your data synchronized',
      taskIcon: {
        name: 'ic_launcher',
        type: 'mipmap',
      }
    };
    
    // Implementation depends on your specific background task library
    return foregroundService;
  }
};
```

### Best Practices for App Lifecycle

1. **Always clean up resources** when app goes to background:
   ```javascript
   const onAppBackground = () => {
     // Stop timers
     clearInterval(myTimer);
     
     // Pause animations
     pauseAnimations();
     
     // Save current state
     saveAppState();
     
     // Clear sensitive data
     clearSensitiveData();
   };
   ```

2. **Refresh data when app resumes**:
   ```javascript
   const onAppForeground = () => {
     // Check if data is stale
     const lastUpdate = getLastUpdateTime();
     const now = Date.now();
     
     if (now - lastUpdate > 5 * 60 * 1000) { // 5 minutes
       refreshData();
     }
     
     // Resume animations
     resumeAnimations();
   };
   ```

3. **Handle network changes**:
   ```javascript
   import NetInfo from '@react-native-async-storage/async-storage';
   
   useEffect(() => {
     const unsubscribe = NetInfo.addEventListener(state => {
       if (state.isConnected && AppState.currentState === 'active') {
         // App is active and connected - sync data
         syncOfflineData();
       }
     });
     
     return unsubscribe;
   }, []);
   ```

4. **Test background behavior thoroughly**:
   - Test what happens when user switches apps
   - Test what happens when phone receives calls
   - Test what happens when device goes to sleep
   - Test on both iOS and Android (they behave differently)

Background services are powerful but come with platform restrictions and user privacy concerns. Always request proper permissions and be transparent about what your app does in the background.

This covers the fundamentals you'll need for most React Native apps. The key is to start simple and gradually add complexity as you get comfortable with these basics. 


## File Handling Across Platforms

Here's how to handle files in React Native for all platforms including web:

1. **Basic file picker using `expo-document-picker`**:
   ```javascript
   import * as DocumentPicker from 'expo-document-picker';

   const pickDocument = async () => {
     try {
       const result = await DocumentPicker.getDocumentAsync({
         // type can be "image/*", "application/pdf", etc
         type: "*/*",  // allows all file types
         multiple: false  // set to true to allow multiple files
       });
       
       if (result.type === 'success') {
         console.log('File URI:', result.uri);
         console.log('File name:', result.name);
         console.log('File size:', result.size);
       }
     } catch (err) {
       console.error('Error picking document:', err);
     }
   };
   ```
   - `type` specifies allowed file types using MIME types
   - `multiple` controls whether user can select multiple files
   - `result.uri` gives you the file location you can use for uploads

2. **Image picker with preview (works on all platforms)**:
   ```javascript
   import * as ImagePicker from 'expo-image-picker';
   import { Platform } from 'react-native';

   const pickImage = async () => {
     // Request permissions first on iOS
     if (Platform.OS !== 'web') {
       const { status } = await ImagePicker.requestMediaLibraryPermissionsAsync();
       if (status !== 'granted') {
         alert('Sorry, we need camera roll permissions!');
         return;
       }
     }

     const result = await ImagePicker.launchImageLibraryAsync({
       mediaTypes: ImagePicker.MediaTypeOptions.Images,
       allowsEditing: true,
       aspect: [4, 3],
       quality: 1,
     });

     if (!result.canceled) {
       // Handle the image
       const imageUri = result.assets[0].uri;
       setImage(imageUri);  // Update state with image URI
     }
   };
   ```

3. **Web-specific file input (for better web support)**:
   ```javascript
   import { Platform } from 'react-native';

   const FileInput = () => {
     if (Platform.OS === 'web') {
       return (
         <input
           type="file"
           onChange={(e) => {
             const file = e.target.files[0];
             console.log('Selected file:', file);
           }}
           accept="image/*,application/pdf"  // specify accepted file types
         />
       );
     }
     
     // Use DocumentPicker for mobile platforms
     return (
       <Button 
         title="Pick File" 
         onPress={pickDocument}  // from example 1
       />
     );
   };
   ```

4. **Handle file uploads consistently**:
   ```javascript
   const uploadFile = async (fileUri) => {
     const formData = new FormData();
     
     // Handle both web and mobile file paths
     if (Platform.OS === 'web') {
       // Web files are already in the right format
       formData.append('file', fileUri);
     } else {
       // Mobile needs more metadata
       formData.append('file', {
         uri: fileUri,
         type: 'application/octet-stream',  // or appropriate MIME type
         name: 'filename.ext'
       });
     }

     try {
       const response = await fetch('YOUR_UPLOAD_URL', {
         method: 'POST',
         body: formData,
         headers: {
           'Content-Type': 'multipart/form-data',
         },
       });
       const data = await response.json();
       console.log('Upload successful:', data);
     } catch (error) {
       console.error('Upload failed:', error);
     }
   };
   ```

Remember to:
- Add necessary permissions in `app.json` for Expo apps
- Handle different file types appropriately
- Consider file size limits on different platforms
- Test thoroughly on all target platforms
- Provide fallback options for unsupported features

5. **Playing audio files using `expo-av`**:
   ```javascript
   import { Audio } from 'expo-av';

   const playAudio = async (fileUri) => {
     try {
       // Create a new Sound object
       const { sound } = await Audio.Sound.createAsync(
         // Can be a require('./path/to/file.mp3') for local files
         // Or { uri: fileUri } for remote/picked files
         { uri: fileUri },
         // Initial playback status
         { 
           shouldPlay: true,     // Start playing immediately
           volume: 1.0,          // Full volume
           isLooping: false      // Don't loop the audio
         }
       );

       // Play the sound
       await sound.playAsync();

       // Don't forget to unload when done
       return () => {
         sound.unloadAsync();
       };
     } catch (error) {
       console.error('Error playing audio:', error);
     }
   };

   // Usage example:
   const AudioPlayer = () => {
     const [sound, setSound] = useState(null);

     useEffect(() => {
       // Cleanup function runs when component unmounts
       return sound ? () => sound.unloadAsync() : undefined;
     }, [sound]);

     const handlePlayback = async (fileUri) => {
       // Stop current sound if any
       if (sound) {
         await sound.unloadAsync();
       }

       // Load and play new sound
       const cleanup = await playAudio(fileUri);
       setSound(cleanup);
     };

     return (
       <Button 
         title="Play Audio"
         onPress={() => handlePlayback('your-file-uri-here')}
       />
     );
   };
   ```

   Key points about audio playback:
   - Install `expo-av` using `expo install expo-av`
   - Always unload sounds when done to free up resources
   - Handle the audio session properly for background playback
   - Consider platform-specific audio behaviors
   - Test with different audio formats (MP3, WAV, etc.)
   - Remember to request audio permissions if needed

   For background audio (iOS):
   ```javascript
   // In app.json
   {
     "expo": {
       "ios": {
         "infoPlist": {
           "UIBackgroundModes": ["audio"]
         }
       }
     }
   }
   ```


   ### Dynamic Styles with StyleSheet
   Yes, the style property accepts an array! This is how you can combine static StyleSheet styles with dynamic values:

   ```javascript
   import { useState } from 'react';

   function DynamicStyleExample() {
     const [padding, setPadding] = useState(10);
     
     return (
       <View style={[
         styles.container,    // Static styles from StyleSheet
         { padding: padding } // Dynamic styles as object
       ]}>
         <Text>Content with dynamic padding</Text>
       </View>
     );
   }

   // You can add multiple style objects to the array:
   <View style={[
     styles.container,
     styles.border,
     { padding: 20 },
     { backgroundColor: 'red' }
   ]}>
     <Text>Multiple styles</Text>
   </View>

   const styles = StyleSheet.create({
     container: {
       flex: 1,
       backgroundColor: '#fff',
     },
     border: {
       borderWidth: 1,
       borderColor: 'black',
     }
   });
   ```

   The style array is evaluated from left to right, so later styles will override earlier ones if they set the same properties. This is useful for:
   - Applying base styles from StyleSheet
   - Overriding specific properties dynamically
   - Conditionally adding styles
   
   Example with conditional styles:
   ```javascript
   <View style={[
     styles.base,
     isActive && styles.active,    // Only applied if isActive is true
     { padding: dynamicPadding }   // Dynamic value
   ]}>
     <Text>Conditional Styling</Text>
   </View>
   ```




