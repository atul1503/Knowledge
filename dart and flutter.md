## How to create flex layout in flutter

Flex container is created by `Flex` widget. Flex child is created by `Flexible` widget.
`Flex` has `children:` argument to specify its children.
`Flexible` has two arguments that we need to set:
1. `fit:` has to be set to either `FlexFit.loose` or `FlexFit.tight`. Tight means that it will be forced to follow the given constraints. Loose means that its constraint is present but it has to adhere to it only if required.
2. `flex:` has to be an integer value. Its like a proportion value. For eg, if Flex container has x space and it has two children with their `flex` set to 4 and 3 respectively then space will be divided as (4x/7) and (3x/7). This is the constraint that is put on the children. If the fit is loose then children can take upto 4x/7 and 3x/7 space(in this case) but if fit is tight then children have to take exactly 4x/7 and 3x/7 space.

**Flex,like this, is best for layouts where you don't know the exact postion they have to be in.

## How to create absolute positioning.

In layouts, this is only required if you want to set the exact position of children in a parent.
Use `Stack` as parent and `Positioned` as child. 
`Positioned` has `top,left,bottom,right` properties to set its position inside the `Stack` parent. Any of its children cannot display itself out of the stack widget.
If the child's position is set in such a way that it is supposed to show outside the stack parent, then the child doesnot even render on the screen. But if the stack is unbounded then the child may display itself without any problem no matter what you set its position.

## How to create infinite scrolling effect.

Use `focus_detector` package to do that. With this package installed, you can wrap your widget as a child inside a `FocusDetector` widget. This widget also has a `onVisibilityGained:` argument, which accepts a callback with no arguments and this callback is called when your widget gains visibility. 

So in case of infinite scrolling, create a list view and in the item builder, return this focus detector widget. In the onVisibilityGained arg, pass a callback function which checks if the last widget is being refered by the index of list view then add more items to your data source. In the child of the focus detector widget gives the widget which displays a single item from the datasource.

Also, since there will be multiple elements inside a listview of the same type, you must provide a `key:` to the focus detector widget.

The datasource should be a state of the widget and you have to add items to it with api calls to the backend.

You can also do the same for the first widget and check if its visible then put items on the front instead of back.

**An example:**

install it with this: `flutter pub add focus_detector`

now do something like this:
```
ListView.builder(
          itemBuilder: (BuildContext ctx, int index) {
            return FocusDetector(
                key: Key(arr[index].toString()),
                onVisibilityGained: () {
                  if (index == arr.length - 1) {
                    addValue(arr[index]); // add more items to the end of the datasource using api calls. 
                  }
                  if (index == 0) {
                    subtractValue(arr[index]);  // add items to the beginning of the datasource.
                  }
                },
                child: Text(arr[index].toString()));
          },
          itemCount: arr.length,
        )
```


## Thumb rules

1. Using stack and postioned, you can create any layout you want, but for standard layouts, always use flex and flexible.
2. If there are multiple widgets of the same type inside a parent, make sure that for each child gets a unique `key:` value. This ensures that flutter can uniquely identify each widget.
3. Use `Padding` as parent of the child to which you are trying to give padding.
4. If you want to set the position and size of a `Container` to certain size and if the sizing of the container is taking up the full space of the container then replace container with a stack and use positioned as one of the children of the stack and then put that container as the child of that positioned. Something like this.

   ```
     Stack
        Positioned
           Container //Your Container that you are trying to have a certain size.
   ```
   With `Positioned` here, you can set both the size(heigth and width) and position(left,right,top,bottom) of the container.

## How to manage state in enterprise Flutter applications

In small apps, you can use `setState` to update the UI. But in enterprise apps, you need better ways to manage state because you have more screens, more data, and more people working on the code. Good state management makes your app easier to test, debug, and scale.

### Simple state management with setState
Use `setState` inside a `StatefulWidget` to update the UI. This is only good for small widgets or screens.

```
class CounterWidget extends StatefulWidget {
  @override
  State<CounterWidget> createState() => _CounterWidgetState();
}

class _CounterWidgetState extends State<CounterWidget> {
  int count = 0; // count is the state variable

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Count: ' + count.toString()),
        ElevatedButton(
          onPressed: () {
            setState(() {
              count++; // setState updates the UI
            });
          },
          child: Text('Increment'),
        ),
      ],
    );
  }
}
```

### State management for large apps
For bigger apps, use a state management package. The most common are:
- **Provider** (official, simple, good for most apps)
- **Riverpod** (more flexible, safer)
- **Bloc** (good for complex business logic)
- **GetX**, **MobX**, etc. (other options)

#### Example: Using Provider
First, add provider: `flutter pub add provider`

Create a class to hold your state:
```
class CounterModel extends ChangeNotifier {
  int count = 0;
  void increment() {
    count++;
    notifyListeners(); // tells widgets to update
  }
}
```

Wrap your app with `ChangeNotifierProvider`:
```
void main() {
  runApp(
    ChangeNotifierProvider(
      create: (_) => CounterModel(), // provides the state
      child: MyApp(),
    ),
  );
}
```

Use the state in your widget:
```
class CounterWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final counter = Provider.of<CounterModel>(context); // gets the state
    return Column(
      children: [
        Text('Count: ' + counter.count.toString()),
        ElevatedButton(
          onPressed: counter.increment, // calls increment
          child: Text('Increment'),
        ),
      ],
    );
  }
}
```
- `ChangeNotifier` is a class that can notify listeners when data changes.
- `ChangeNotifierProvider` makes the state available to all widgets below it.
- `Provider.of<T>(context)` gets the state object.

**Best practices for enterprise apps:**
- Split state into small, focused classes (one per feature or screen).
- Keep business logic out of UI widgets.
- Use dependency injection for testability.
- Use immutable data where possible.
- Write tests for your state classes.

## Common Flutter widgets and their usage

Here are some widgets you will use in almost every app:

- **Container**: A box to hold other widgets. You can set its size, color, padding, margin, etc.
  ```
  Container(
    width: 100, // width in logical pixels
    height: 50, // height in logical pixels
    color: Colors.blue, // background color
    child: Text('Hello'),
  )
  ```
- **Row**: Puts children in a horizontal line.
  ```
  Row(
    children: [Text('A'), Text('B')],
  )
  ```
- **Column**: Puts children in a vertical line.
  ```
  Column(
    children: [Text('A'), Text('B')],
  )
  ```
- **Text**: Shows a string of text.
  ```
  Text('Hello', style: TextStyle(fontSize: 20)), // fontSize sets text size
  ```
- **Image**: Shows an image from assets or network.
  ```
  Image.asset('assets/logo.png') // from local assets
  Image.network('https://example.com/image.png') // from the web
  ```
- **ListView**: Scrollable list of widgets.
  ```
  ListView(
    children: [Text('Item 1'), Text('Item 2')],
  )
  ```
- **Scaffold**: Basic page layout with app bar, body, etc.
  ```
  Scaffold(
    appBar: AppBar(title: Text('Title')),
    body: Center(child: Text('Body')),
  )
  ```
- **AppBar**: Top bar for titles and actions.
  ```
  AppBar(
    title: Text('My App'),
    actions: [IconButton(icon: Icon(Icons.search), onPressed: () {})],
  )
  ```
- **ElevatedButton**: A button with elevation (shadow).
  ```
  ElevatedButton(
    onPressed: () { print('Clicked'); }, // onPressed is called when tapped
    child: Text('Click me'),
  )
  ```
- **Padding**: Adds space around a widget.
  ```
  Padding(
    padding: EdgeInsets.all(8), // 8 pixels on all sides
    child: Text('Padded'),
  )
  ```
- **Center**: Centers its child.
  ```
  Center(child: Text('Centered'))
  ```

For each widget, set properties like `child`, `children`, `color`, `padding`, etc. the first time you use them. This helps you understand what each property does.

## How to use Riverpod for state management

Riverpod is a modern state management library for Flutter. It is safer and more flexible than Provider. You do not need to use `BuildContext` to read state, and it works well for both simple and complex apps.

First, add riverpod: `flutter pub add flutter_riverpod`

### Basic usage

1. Import Riverpod:
   ```
   import 'package:flutter_riverpod/flutter_riverpod.dart';
   ```
2. Create a provider for your state:
   ```
   final counterProvider = StateProvider<int>((ref) => 0); // holds an int, starts at 0
   ```
3. Use a `ConsumerWidget` to read and update the state:
   ```
   class CounterWidget extends ConsumerWidget {
     @override
     Widget build(BuildContext context, WidgetRef ref) {
       final count = ref.watch(counterProvider); // reads the state
       return Column(
         children: [
           Text('Count: ' + count.toString()),
           ElevatedButton(
             onPressed: () {
               ref.read(counterProvider.notifier).state++; // updates the state
             },
             child: Text('Increment'),
           ),
         ],
       );
     }
   }
   ```
   - `StateProvider` is a simple way to store a value and update it.
   - `ConsumerWidget` lets you read providers using `WidgetRef`.
   - `ref.watch` listens to changes in the provider.
   - `ref.read(...notifier).state` lets you update the value.

4. Wrap your app with `ProviderScope` at the top:
   ```
   void main() {
     runApp(ProviderScope(child: MyApp()));
   }
   ```
   - `ProviderScope` is required for Riverpod to work. It stores all providers.

### Best practices
- Use `StateNotifierProvider` for more complex state (like classes with multiple fields and methods).
- Keep your providers in a separate file for large apps.
- Use `ref.listen` for side effects (like showing dialogs or navigation).

## How to use multiple ChangeNotifierProviders (MultiProvider)

If you have more than one state class, use `MultiProvider` to provide them all at once. This is common in large apps.

First, add provider: `flutter pub add provider` (if not already added).

Example:
```
class CounterModel extends ChangeNotifier {
  int count = 0;
  void increment() {
    count++;
    notifyListeners();
  }
}

class ThemeModel extends ChangeNotifier {
  bool isDark = false;
  void toggleTheme() {
    isDark = !isDark;
    notifyListeners();
  }
}

void main() {
  runApp(
    MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (_) => CounterModel()),
        ChangeNotifierProvider(create: (_) => ThemeModel()),
      ],
      child: MyApp(),
    ),
  );
}
```
- `MultiProvider` takes a `providers:` list, where you put each `ChangeNotifierProvider`.
- Each provider is created with a `create:` function.
- All state objects are available to widgets below `MultiProvider`.

To use them in a widget:
```
final counter = Provider.of<CounterModel>(context);
final theme = Provider.of<ThemeModel>(context);
```
Now you can read and update both states in your widgets.

## How to give any shape to your widget in Flutter

In Flutter, you can give any shape to your widget, not just rectangles, circles, or triangles. You do this by using the `ClipPath` widget with a custom `CustomClipper<Path>`. The `Path` class lets you draw any shape you want by moving to points and drawing lines, curves, or arcs between them.

### Basic usage with ClipPath

1. Use `ClipPath` as the parent of your widget.
2. Pass a custom clipper to the `clipper:` property. This clipper defines the shape.

Example: Clipping a widget to a custom zig-zag shape
```
class ZigZagClipper extends CustomClipper<Path> {
  @override
  Path getClip(Size size) {
    Path path = Path();
    path.moveTo(0, 0);
    path.lineTo(size.width, 0);
    path.lineTo(size.width, size.height - 20);
    for (double i = size.width; i > 0; i -= 20) {
      path.lineTo(i - 10, size.height);
      path.lineTo(i - 20, size.height - 20);
    }
    path.lineTo(0, size.height - 20);
    path.close();
    return path;
  }

  @override
  bool shouldReclip(CustomClipper<Path> oldClipper) => false;
}

ClipPath(
  clipper: ZigZagClipper(), // sets the custom shape
  child: Container(
    color: Colors.orange,
    height: 120,
    width: 200,
    child: Center(child: Text('ZigZag Shape')),
  ),
)
```
- `ClipPath` clips its child to the shape defined by the `clipper`.
- `CustomClipper<Path>` lets you define any shape using the `Path` API.
- `getClip(Size size)` returns a `Path` for the given size.
- `moveTo(x, y)` moves the pen to a point without drawing.
- `lineTo(x, y)` draws a straight line from the current point.
- You can also use `arcTo`, `quadraticBezierTo`, `cubicTo`, etc. for curves.

### Drawing curves and custom paths

To draw curves, use the `Path` methods like `quadraticBezierTo`, `cubicTo`, and `arcTo`. These let you create smooth, flowing shapes beyond straight lines.

#### Example: Drawing a wave with a quadratic Bezier curve
```
class WaveClipper extends CustomClipper<Path> {
  @override
  Path getClip(Size size) {
    Path path = Path();
    path.lineTo(0, size.height - 40);
    // Draw a quadratic Bezier curve from (0, height-40) to (width, height-40)
    // with control point at (width/2, height)
    path.quadraticBezierTo(
      size.width / 2, size.height, // control point
      size.width, size.height - 40 // end point
    );
    path.lineTo(size.width, 0);
    path.close();
    return path;
  }

  @override
  bool shouldReclip(CustomClipper<Path> oldClipper) => false;
}

ClipPath(
  clipper: WaveClipper(),
  child: Container(
    color: Colors.blue,
    height: 120,
    width: 200,
    child: Center(child: Text('Wave Shape')),
  ),
)
```
- `quadraticBezierTo(controlX, controlY, endX, endY)` draws a smooth curve from the current point to (endX, endY), using (controlX, controlY) to control the curve's bend.

#### Example: Drawing a cubic Bezier curve
```
path.cubicTo(
  30, 0,      // first control point
  80, 120,    // second control point
  200, 100    // end point
);
```
- `cubicTo` uses two control points for more complex curves.

#### Example: Drawing an arc
```
Rect rect = Rect.fromCircle(center: Offset(100, 100), radius: 80);
path.arcTo(rect, 0, 3.14, false); // draws a half-circle arc
```
- `arcTo(rect, startAngle, sweepAngle, forceMoveTo)` draws an arc inside the given rectangle.
- Angles are in radians (3.14 is about 180 degrees).

**Summary:** Use `quadraticBezierTo`, `cubicTo`, and `arcTo` on a `Path` to draw any kind of curve or arc. Combine these with lines for complex, smooth shapes.

### More examples
- **Circle:** Use `ClipOval` (built-in for circles/ovals).
- **Rounded rectangle:** Use `ClipRRect` with a `borderRadius:`.
- **Any SVG shape:** Use a package like `flutter_custom_clippers` or draw the path yourself.

### Tips
- The `Path` class is very powerful. You can draw literally any shape by combining lines and curves.
- For complex shapes, you can use vector tools (like SVG editors) to get path data and convert it to Dart code.
- There are many open-source clippers available for common shapes (waves, arrows, stars, etc.).

**Summary:** To give any shape to your widget, use `ClipPath` with a custom `CustomClipper<Path>`. Define your shape in the `getClip` method using the `Path` API.

## Why do we have StatefulWidget when all the UI is defined in State<T>?

In Flutter, a `StatefulWidget` is a widget that can rebuild itself when its state changes. But you might notice that all the UI code is actually written in the `State<T>` class, not in the `StatefulWidget` itself. Why is that?

### The reason for this design
- The `StatefulWidget` class is just a configuration object. It holds the values (constructor arguments) that do not change during the widget's lifetime.
- The `State<T>` class holds the mutable state and the `build` method. This is where you put variables that can change and cause the UI to update.
- When Flutter needs to rebuild the widget (for example, after calling `setState`), it does not recreate the `StatefulWidget` object. Instead, it keeps the same `State` object and just calls its `build` method again.
- This separation allows Flutter to efficiently update the UI without losing the state, even if the widget is rebuilt with new configuration.

### Example
```
class MyCounter extends StatefulWidget {
  final String title; // configuration, does not change
  MyCounter({required this.title});

  @override
  State<MyCounter> createState() => _MyCounterState();
}

class _MyCounterState extends State<MyCounter> {
  int count = 0; // mutable state

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text(widget.title), // access configuration from widget
        Text('Count: ' + count.toString()),
        ElevatedButton(
          onPressed: () {
            setState(() {
              count++;
            });
          },
          child: Text('Increment'),
        ),
      ],
    );
  }
}
```
- `MyCounter` is the `StatefulWidget`. It holds the `title` property, which does not change.
- `_MyCounterState` is the state class. It holds the `count` variable, which can change.
- The `build` method is in the state class, so it can use both the configuration (`widget.title`) and the mutable state (`count`).

**Summary:**
- `StatefulWidget` holds configuration (unchanging values).
- `State<T>` holds mutable state and the UI code.
- This separation lets Flutter efficiently update the UI and keep state even when the widget is rebuilt with new configuration.

## Dart: Constructors, Lambdas, and Function Arguments

### Constructor syntaxes
Dart has several ways to define constructors:

- **Default constructor:**
  ```
  class Person {
    Person() {
      // code
    }
  }
  ```
- **Constructor with arguments:**
  ```
  class Person {
    String name;
    int age;
    Person(this.name, this.age); // assigns arguments to fields
  }
  ```
- **Named constructor:**
  ```
  class Person {
    String name;
    Person.guest() : name = 'Guest'; // named constructor
  }
  ```
- **Named parameters (with curly braces):**
  ```
  class Person {
    String name;
    int age;
    Person({required this.name, this.age = 0}); // named and default value
  }
  // Usage: Person(name: 'Atul')
  ```
- **Redirecting constructor:**
  ```
  class Person {
    String name;
    Person(this.name);
    Person.guest() : this('Guest'); // calls another constructor
  }
  ```
- **Factory constructor:**
  ```
  class Person {
    factory Person.fromJson(Map<String, dynamic> json) {
      return Person(json['name']);
    }
    Person(this.name);
    String name;
  }
  ```

### Lambda (function expression) syntaxes
- **Arrow syntax:**
  ```
  int add(int a, int b) => a + b; // single-expression function
  ```
- **Anonymous function (lambda):**
  ```
  var list = [1, 2, 3];
  list.forEach((item) {
    print(item);
  });
  // or with arrow:
  list.forEach((item) => print(item));
  ```
- **Assigning a function to a variable:**
  ```
  var multiply = (int a, int b) => a * b;
  print(multiply(2, 3)); // 6
  ```

### Ways to define function arguments
- **Positional arguments:**
  ```
  void greet(String name, int age) {
    print('Hello $name, age $age');
  }
  greet('Atul', 30);
  ```
- **Optional positional arguments (with square brackets):**
  ```
  void greet(String name, [int? age]) {
    print('Hello $name, age $age');
  }
  greet('Atul');
  ```
- **Named arguments (with curly braces):**
  ```
  void greet({required String name, int age = 0}) {
    print('Hello $name, age $age');
  }
  greet(name: 'Atul');
  ```
- **Mixing positional and named arguments:**
  ```
  void greet(String greeting, {required String name}) {
    print('$greeting $name');
  }
  greet('Hi', name: 'Atul');
  ```
- **Default values:**
  ```
  void greet({String name = 'Guest'}) {
    print('Hello $name');
  }
  greet();
  ```

**Summary:**
- Dart supports multiple constructor styles, arrow/lambda functions, and flexible ways to define function arguments (positional, optional, named, default values). Use curly braces `{}` for named arguments, square brackets `[]` for optional positional arguments, and `=>` for single-expression functions.

## Dart: Major Array (List) and Map Methods for DSA

### List (Array) Methods
- **add(element):** Adds an element to the end of the list.
  ```
  var arr = [1, 2];
  arr.add(3); // [1, 2, 3]
  ```
- **addAll(iterable):** Adds all elements from another list or iterable.
  ```
  arr.addAll([4, 5]); // [1, 2, 3, 4, 5]
  ```
- **insert(index, element):** Inserts an element at a specific index.
  ```
  arr.insert(1, 10); // [1, 10, 2, 3, 4, 5]
  ```
- **remove(element):** Removes the first occurrence of the element.
  ```
  arr.remove(10); // [1, 2, 3, 4, 5]
  ```
- **removeAt(index):** Removes the element at the given index.
  ```
  arr.removeAt(2); // [1, 2, 4, 5]
  ```
- **removeLast():** Removes the last element.
  ```
  arr.removeLast(); // [1, 2, 4]
  ```
- **contains(element):** Checks if the list contains the element.
  ```
  arr.contains(2); // true
  ```
- **indexOf(element):** Returns the first index of the element, or -1 if not found.
  ```
  arr.indexOf(4); // 2
  ```
- **sort([compare]):** Sorts the list in place. Optionally takes a compare function.
  ```
  arr.sort(); // ascending
  arr.sort((a, b) => b - a); // descending
  ```
- **reversed:** Returns an iterable of the list in reverse order.
  ```
  arr.reversed.toList(); // [4, 2, 1]
  ```
- **map(function):** Returns a new iterable with each element transformed by the function.
  ```
  var doubled = arr.map((x) => x * 2).toList(); // [2, 4, 8]
  ```
- **where(function):** Returns a new iterable with elements that satisfy the function.
  ```
  var evens = arr.where((x) => x % 2 == 0).toList(); // [2, 4]
  ```
- **reduce(function):** Combines all elements into a single value using the function.
  ```
  var sum = arr.reduce((a, b) => a + b); // 7
  ```
- **fold(initial, function):** Like reduce, but starts with an initial value.
  ```
  var product = arr.fold(1, (a, b) => a * b); // 8
  ```
- **any(function):** Returns true if any element satisfies the function.
  ```
  arr.any((x) => x > 3); // true
  ```
- **every(function):** Returns true if all elements satisfy the function.
  ```
  arr.every((x) => x > 0); // true
  ```
- **sublist(start, [end]):** Returns a new list from start (inclusive) to end (exclusive).
  ```
  arr.sublist(1, 3); // [2, 4]
  ```
- **clear():** Removes all elements from the list.
  ```
  arr.clear(); // []
  ```

### Map Methods
- **putIfAbsent(key, function):** Adds a key with a value if it does not exist.
  ```
  var map = {'a': 1};
  map.putIfAbsent('b', () => 2); // {'a': 1, 'b': 2}
  ```
- **update(key, function, {ifAbsent}):** Updates the value for a key.
  ```
  map.update('a', (v) => v + 10); // {'a': 11, 'b': 2}
  map.update('c', (v) => 0, ifAbsent: () => 0); // {'a': 11, 'b': 2, 'c': 0}
  ```
- **remove(key):** Removes a key and its value.
  ```
  map.remove('b'); // {'a': 11, 'c': 0}
  ```
- **containsKey(key):** Checks if the map contains the key.
  ```
  map.containsKey('a'); // true
  ```
- **containsValue(value):** Checks if the map contains the value.
  ```
  map.containsValue(11); // true
  ```
- **keys / values:** Returns all keys or values as iterables.
  ```
  map.keys.toList(); // ['a', 'c']
  map.values.toList(); // [11, 0]
  ```
- **forEach(function):** Runs a function for each key-value pair.
  ```
  map.forEach((k, v) => print('$k: $v'));
  ```
- **map(function):** Returns a new map with each entry transformed by the function.
  ```
  var newMap = map.map((k, v) => MapEntry(k, v * 2)); // {'a': 22, 'c': 0}
  ```
- **addAll(otherMap):** Adds all key-value pairs from another map.
  ```
  map.addAll({'d': 5}); // {'a': 11, 'c': 0, 'd': 5}
  ```
- **clear():** Removes all entries from the map.
  ```
  map.clear(); // {}
  ```

**Summary:**
- Use these List and Map methods for common DSA tasks: adding, removing, searching, transforming, and aggregating data.

## How to create a mutable list with a generic type in Dart

In Dart, you can create a mutable list (array) that holds any type you want by using generics. Use `List<T>` where `T` is the type you want to store.

### Example: Mutable list of integers
```
List<int> numbers = []; // empty, can add/remove items
numbers.add(1);
numbers.add(2);
print(numbers); // [1, 2]
```

### Example: Mutable list of strings
```
List<String> names = ['Alice', 'Bob'];
names.add('Charlie');
print(names); // [Alice, Bob, Charlie]
```

### Example: Mutable list of any custom type
```
class Person {
  String name;
  Person(this.name);
}

List<Person> people = [];
people.add(Person('Atul'));
people.add(Person('Sam'));
print(people.map((p) => p.name).toList()); // [Atul, Sam]
```

- `List<T>` is mutable by default. You can add, remove, or change elements.
- You can use any type for `T`: built-in types, custom classes, or even other generics (like `List<List<int>>`).

## Can I run a Dart program like Python without project creation?

Yes, you can run a Dart program directly from a single file, just like you do with Python scripts. You do not need to create a full project.

### How to do it
1. Save your Dart code in a file, for example `hello.dart`:
   ```dart
   void main() {
     print('Hello, Dart!');
   }
   ```
2. Run it from the terminal using the Dart SDK:
   ```bash
   dart run hello.dart
   ```
   Or (older syntax):
   ```bash
   dart hello.dart
   ```

- You only need to have the Dart SDK installed and available in your PATH.
- This works for any Dart file, not just `main.dart`.
- No need to create a `pubspec.yaml` or use `dart create` unless you want to use packages or build a bigger app.

**Summary:**
- You can run Dart files directly from the terminal, just like Python, without creating a project.

### How to use a library (package) in a Dart script

If you want to use a library (package) in your Dart script, you need to create a `pubspec.yaml` file in the same directory as your script. This tells Dart which packages to download.

#### Steps:
1. Create your Dart file, e.g. `example.dart`:
   ```dart
   import 'package:http/http.dart' as http;

   void main() async {
     var response = await http.get(Uri.parse('https://example.com'));
     print(response.statusCode);
   }
   ```
2. In the same directory, create a `pubspec.yaml` file:
   ```yaml
   name: example_script
   dependencies:
     http: ^1.0.0
   ```
3. Run `dart pub get` in the terminal to download the package:
   ```bash
   dart pub get
   ```
4. Now run your script as before:
   ```bash
   dart run example.dart
   ```

- You do **not** need to create a full project with `dart create`.
- Just make sure your Dart file and `pubspec.yaml` are in the same folder.
- You can add any number of dependencies in `pubspec.yaml`.

**Summary:**
- To use packages in a Dart script, add a `pubspec.yaml` in the same directory, run `dart pub get`, and then run your script.

## How to send network requests, handle JSON, and work with large files in Dart

### Sending network requests and parsing JSON

Use the `http` package for simple HTTP requests.

#### Example: GET request and mapping JSON to an object
1. Add to your `pubspec.yaml`:
   ```yaml
   dependencies:
     http: ^1.0.0
   ```
2. Example Dart code:
   ```dart
   import 'dart:convert';
   import 'package:http/http.dart' as http;

   class User {
     final String name;
     final int age;
     User({required this.name, required this.age});

     // Factory constructor to create a User from JSON
     factory User.fromJson(Map<String, dynamic> json) {
       return User(
         name: json['name'],
         age: json['age'],
       );
     }
   }

   void main() async {
     var response = await http.get(Uri.parse('https://example.com/user.json'));
     if (response.statusCode == 200) {
       var data = jsonDecode(response.body); // parse JSON string to Map
       var user = User.fromJson(data); // create object from JSON
       print(user.name);
     }
   }
   ```
- Use `http.get` for GET requests, `http.post` for POST, etc.
- Use `jsonDecode` to convert JSON string to a Dart map.
- Use a factory constructor to convert a map to your object.

### Downloading or uploading large files

For large files, use the `http` package with streams, or the `dio` package for more advanced features.

#### Downloading a large file with streams
```dart
import 'dart:io';
import 'package:http/http.dart' as http;

void main() async {
  var url = Uri.parse('https://example.com/largefile.zip');
  var request = http.Request('GET', url);
  var response = await request.send(); // returns a StreamedResponse

  var file = File('largefile.zip').openWrite(); // open file for writing
  await response.stream.pipe(file); // pipe the response stream to the file
  await file.close();
  print('Download complete');
}
```
- `http.Request(...).send()` gives you a `StreamedResponse`.
- Use `response.stream.pipe(file)` to write the stream to a file.

#### Uploading a large file with streams
```dart
import 'dart:io';
import 'package:http/http.dart' as http;

void main() async {
  var url = Uri.parse('https://example.com/upload');
  var request = http.MultipartRequest('POST', url);
  request.files.add(await http.MultipartFile.fromPath('file', 'largefile.zip'));
  var response = await request.send();
  print('Upload status: ${response.statusCode}');
}
```
- Use `http.MultipartRequest` for file uploads.
- `MultipartFile.fromPath` reads the file as a stream and uploads it efficiently.

### Using streams for network data
- Streams let you process data as it arrives, instead of waiting for the whole response.
- Useful for large files, real-time data, or chunked responses.

#### Example: Reading a response stream line by line
```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

void main() async {
  var url = Uri.parse('https://example.com/streaming-data');
  var request = http.Request('GET', url);
  var response = await request.send();

  // Decode the stream as UTF-8 and split by lines
  await for (var line in response.stream.transform(utf8.decoder).transform(LineSplitter())) {
    print('Received: $line');
  }
}
```

**Summary:**
- Use `http` for simple requests and JSON.
- Use streams for large files or real-time data.
- Use factory constructors to convert JSON maps to Dart objects.
- For advanced needs, check out the `dio` package (supports progress, cancellation, etc.).

## How to do threading and work with asynchronous code in Dart

Dart uses an event loop and single-threaded model by default, but provides powerful tools for asynchronous programming and parallelism.

### Futures and async/await
- **Future:** Represents a value that will be available later (like a promise in JS).
- **async/await:** Lets you write asynchronous code that looks like synchronous code.

#### Example: Using Future and async/await
```dart
Future<String> fetchData() async {
  await Future.delayed(Duration(seconds: 2)); // simulate delay
  return 'Data loaded';
}

void main() async {
  print('Start');
  var data = await fetchData(); // waits for fetchData to complete
  print(data);
  print('End');
}
```
- Use `async` on a function to allow `await` inside it.
- `await` pauses execution until the Future completes.

#### Example: Handling errors with async/await
```dart
void main() async {
  try {
    var data = await fetchData();
    print(data);
  } catch (e) {
    print('Error: $e');
  }
}
```

### Microtasks and event queue
- **Microtasks:** Small tasks scheduled to run before the next event loop iteration.
- Use `scheduleMicrotask(() { ... })` to schedule a microtask.

#### Example:
```dart
import 'dart:async';

void main() {
  scheduleMicrotask(() => print('microtask'));
  print('event');
}
// Output: event\nmicrotask
```

### Isolates (true parallelism)
- Dart is single-threaded by default, but you can run code in parallel using Isolates.
- Each Isolate has its own memory and event loop.
- Use Isolates for CPU-intensive work (parsing, computation, etc.).

#### Example: Running code in a separate Isolate
```dart
import 'dart:isolate';

void heavyTask(SendPort sendPort) {
  int sum = 0;
  for (int i = 0; i < 100000000; i++) {
    sum += i;
  }
  sendPort.send(sum);
}

void main() async {
  ReceivePort receivePort = ReceivePort();
  await Isolate.spawn(heavyTask, receivePort.sendPort);
  var result = await receivePort.first;
  print('Sum: $result');
}
```
- Use `Isolate.spawn` to start a new isolate.
- Use `ReceivePort` and `SendPort` to communicate between isolates.

**Summary:**
- Use `Future` and `async/await` for most asynchronous code (network, file, timers).
- Use microtasks for small, high-priority tasks.
- Use Isolates for true parallelism and heavy computation.

## How to use images in a Flutter application

There are two common scenarios for showing images in Flutter:

### 1. Download and show an image from the internet
Use the `Image.network` widget to display an image from a URL.
```dart
Image.network(
  'https://example.com/image.png',
  width: 200,
  height: 200,
  fit: BoxFit.cover, // scales/crops the image to fit
)
```
- `fit` controls how the image is resized (e.g., `BoxFit.cover`, `BoxFit.contain`).
- You can use a package like `cached_network_image` for caching and placeholders.

#### Example with placeholder and error handling (using `cached_network_image`):
1. Add to `pubspec.yaml`:
   ```yaml
   dependencies:
     cached_network_image: ^3.2.3
   ```
2. Use in your widget:
   ```dart
   import 'package:cached_network_image/cached_network_image.dart';

   CachedNetworkImage(
     imageUrl: 'https://example.com/image.png',
     placeholder: (context, url) => CircularProgressIndicator(),
     errorWidget: (context, url, error) => Icon(Icons.error),
   )
   ```

### 2. Show an image that is part of the project (assets)
1. Put your image file (e.g., `logo.png`) in an `assets` folder in your project root.
2. Add the asset to your `pubspec.yaml`:
   ```yaml
   flutter:
     assets:
       - assets/logo.png
   ```
3. Use `Image.asset` to display it:
   ```dart
   Image.asset(
     'assets/logo.png',
     width: 100,
     height: 100,
   )
   ```
- You can use subfolders (e.g., `assets/images/logo.png`) and list the folder in `pubspec.yaml` (e.g., `- assets/images/`).
- Asset images are bundled with your app and do not require internet access.

**Summary:**
- Use `Image.network` for images from the web.
- Use `Image.asset` for images included in your project.
- For caching and placeholders, use the `cached_network_image` package.

## How to write tests for a Flutter application

Flutter supports three main types of tests:
- **Unit tests:** Test a single function, method, or class.
- **Widget tests:** Test a single widget (UI component) in isolation.
- **Integration tests:** Test the full app or a large part of it (not covered here for brevity).

### 1. Unit test example
Suppose you have a simple function:
```dart
double add(double a, double b) => a + b;
```
Create a test file (e.g., `test/add_test.dart`):
```dart
import 'package:test/test.dart';
import '../lib/my_functions.dart'; // adjust path as needed

void main() {
  test('adds two numbers', () {
    expect(add(2, 3), 5);
    expect(add(-1, 1), 0);
  });
}
```
- Use `test` from the `test` package (included by default).
- Use `expect(actual, matcher)` to check results.

### 2. Widget test example
Suppose you have a widget:
```dart
import 'package:flutter/material.dart';

class MyButton extends StatelessWidget {
  final VoidCallback onPressed;
  final String label;
  MyButton({required this.onPressed, required this.label});

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: onPressed,
      child: Text(label),
    );
  }
}
```
Create a test file (e.g., `test/my_button_test.dart`):
```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:flutter/material.dart';
import '../lib/my_button.dart'; // adjust path as needed

void main() {
  testWidgets('MyButton displays label and reacts to tap', (WidgetTester tester) async {
    var pressed = false;
    await tester.pumpWidget(
      MaterialApp(
        home: MyButton(
          label: 'Click me',
          onPressed: () { pressed = true; },
        ),
      ),
    );

    expect(find.text('Click me'), findsOneWidget);
    await tester.tap(find.byType(ElevatedButton));
    expect(pressed, true);
  });
}
```
- Use `flutter_test` package (included by default in Flutter projects).
- Use `testWidgets` for widget tests.
- Use `WidgetTester` to build and interact with widgets.
- Use `find` to locate widgets in the widget tree.

### How to run tests
- Run all tests:
  ```bash
  flutter test
  ```
- Run a specific test file:
  ```bash
  flutter test test/my_button_test.dart
  ```

**Summary:**
- Use the `test` package for unit tests and `flutter_test` for widget tests.
- Place test files in the `test/` directory.
- Use `flutter test` to run your tests.

## Most common and enterprise way to add navigation in Flutter

Flutter provides two main ways to handle navigation:
- **Navigator 1.0 (classic push/pop):** Simple, good for small apps.
- **Navigator 2.0 (Router API) and packages like go_router:** Recommended for enterprise apps, deep linking, and complex navigation.

### 1. Simple navigation with Navigator 1.0
```dart
Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => SecondPage()),
);
// To go back:
Navigator.pop(context);
```
- Good for simple, stack-based navigation.

### 2. Enterprise navigation with go_router (recommended)
`go_router` is the most popular and enterprise-ready navigation package for Flutter. It supports deep linking, nested navigation, guards, and more.

#### How to use go_router
1. Add to your `pubspec.yaml`:
   ```yaml
   dependencies:
     go_router: ^13.0.0
   ```
2. Define your routes and use `GoRouter`:
   ```dart
   import 'package:flutter/material.dart';
   import 'package:go_router/go_router.dart';

   void main() {
     final GoRouter _router = GoRouter(
       routes: [
         GoRoute(
           path: '/',
           builder: (context, state) => HomePage(),
         ),
         GoRoute(
           path: '/details/:id',
           builder: (context, state) {
             final id = state.params['id'];
             return DetailsPage(id: id!);
           },
         ),
       ],
     );

     runApp(MyApp(router: _router));
   }

   class MyApp extends StatelessWidget {
     final GoRouter router;
     MyApp({required this.router});
     @override
     Widget build(BuildContext context) {
       return MaterialApp.router(
         routerConfig: router,
       );
     }
   }
   ```
3. Navigate between pages:
   ```dart
   // Go to details page with id=42
   context.go('/details/42');
   // Or use context.push('/details/42') to push onto the stack
   ```
- `go_router` supports named routes, query parameters, guards (for auth), nested navigation, and more.
- Use `context.go()` for navigation and `context.pop()` to go back.

### Why use go_router or Navigator 2.0 for enterprise apps?
- Handles deep links and browser URLs (for web).
- Supports complex navigation flows (auth, onboarding, tabs, etc.).
- Easier to test and maintain.
- Recommended by the Flutter team for new apps.

**Summary:**
- Use `Navigator.push`/`pop` for simple apps.
- Use `go_router` (or Navigator 2.0) for enterprise apps and complex navigation.




