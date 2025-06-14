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

This covers the fundamentals you'll need for most React Native apps. The key is to start simple and gradually add complexity as you get comfortable with these basics. 