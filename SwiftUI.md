# SwiftUI Documentation

## Basic Views and Layout

1. **View** is the basic building block. Everything you see on screen is a View. Views describe what the UI should look like, not how to change it.
   ```swift
   struct ContentView: View {  // Curly braces define the struct's implementation block
       // The 'body' property is required by the View protocol - it's the only required property
       // You must name it 'body' - no other name will work
       // This property defines what your view looks like and returns its content
       // SwiftUI calls this property to render your view on screen
       var body: some View {   // 'some View' means this returns a specific type that conforms to View protocol
           // Curly braces after View define the body property's implementation block
           // This block contains the code that gets executed whenever SwiftUI needs to render this view
           // It runs when the view first appears and re-runs whenever any @State or observed data changes
           // This block is a closure that returns a View
           // You can write logical code here, like:
           // - Variables and constants using let/var 
           // - Control flow (if/else, switch, loops)
           // - Function calls
           // - Calculations
           // You have access to:
           // - Properties and methods of the view struct
           // - @State and other property wrappers
           // - Environment values
           // - Any imported frameworks
           // The final expression must return a View
           // In this example, Text("Hello, World!") is the final expression,
           // so this view's body returns a Text view. You can return any view
           // type - Text, Button, VStack, etc. The last view in the body
           // becomes what this view displays
           Text("Hello, World!")
       }
   }
   ```
   
   Key syntax explained:
   - Curly braces `{}` after `struct` declaration define the struct's implementation block, containing all its properties and methods
   - `some View` uses Swift's "opaque return type" feature - it means this returns a specific view type (known to compiler but hidden from us)
   - Curly braces after `View` in `body` define a closure - the actual implementation of what this view looks like
   - SwiftUI uses trailing closure syntax extensively, where the last parameter (the view's content) is written in braces after the declaration

2. **VStack** arranges views vertically (top to bottom). **HStack** arranges views horizontally (left to right). **ZStack** layers views on top of each other.
   ```swift
   VStack {  // Vertical arrangement
       Text("Top")
       Text("Middle") 
       Text("Bottom")
   }
   
   HStack {  // Horizontal arrangement
       Text("Left")
       Text("Center")
       Text("Right")
   }
   
   ZStack {  // Layered on top of each other
       Circle().fill(Color.blue)
       Text("On top of circle")
   }
   ```

3. **Text** displays strings. You can style it with modifiers like `.font()`, `.foregroundColor()`, `.bold()`.
   ```swift
   Text("Hello")
       .font(.title)           // Makes text bigger (title size)
       .foregroundColor(.red)  // Changes text color to red
       .bold()                 // Makes text bold
   ```

4. **Image** displays pictures from your app bundle or SF Symbols (Apple's icon library).
   ```swift
   Image("photo-name")         // Shows image from your app's assets
   Image(systemName: "heart")  // Shows SF Symbol (Apple's icons)
       .foregroundColor(Color.red)  // Colors the icon red (Color.red makes it explicit that .red comes from the Color type in SwiftUI. Color has static properties like .red, .blue etc. that return Color instances)
       .font(.largeTitle)      // Makes icon bigger
   ```

5. **Button** responds to taps. The first parameter is the action (what happens when tapped), the second is the label (what the button looks like).
   ```swift
   Button(action: {
       print("Button tapped!")  // This runs when button is pressed
   }) {
       Text("Tap me")  // This is what the button looks like
   }
   
   // Shorter syntax when action is simple
   Button("Tap me") {
       print("Button tapped!")
   }
   ```

## State Management

6. **@State** is for data that can change within a single view. When @State data changes, SwiftUI automatically redraws the view. Always mark @State as `private` since it belongs only to this view.
   ```swift
   struct CounterView: View {
       @State private var count = 0  // @State makes this changeable
       
       var body: some View {
           VStack {
               Text("Count: \(count)")
               Button("Increment") {
                   count += 1  // When this changes, UI updates automatically
               }
           }
       }
   }
   ```

7. **@Binding** lets a child view modify data owned by its parent. Use `$` to pass a binding from parent to child.
   ```swift
   struct ParentView: View {
       @State private var name = ""  // Parent owns the data
       
       var body: some View {
           ChildView(name: $name)  // $ creates a binding to pass to child
       }
   }
   
   struct ChildView: View {
       @Binding var name: String  // Child can modify parent's data through binding
       
       var body: some View {
           TextField("Enter name", text: $name)  // Changes parent's @State
       }
   }
   ```

8. **@ObservedObject** is for data that lives outside your view, in a separate class. The class must conform to `ObservableObject` and use `@Published` for properties that should trigger UI updates.
   ```swift
   class UserData: ObservableObject {
       @Published var name = ""     // @Published means "update UI when this changes"
       @Published var isLoggedIn = false
   }
   
   struct ContentView: View {
       @ObservedObject var userData = UserData()  // Watches for changes
       
       var body: some View {
           Text("Welcome, \(userData.name)")  // Updates when userData.name changes
       }
   }
   ```

9. **@StateObject** is like @ObservedObject but creates and owns the object. Use @StateObject when the view should create the object, use @ObservedObject when the object comes from somewhere else.
   ```swift
   struct MainView: View {
       @StateObject private var userData = UserData()  // This view creates and owns it
       
       var body: some View {
           ProfileView(userData: userData)  // Pass to child as @ObservedObject
       }
   }
   ```

## Common UI Elements

10. **TextField** lets users type text. Always needs a binding to store the text and a placeholder string.
    ```swift
    struct LoginForm: View {
        @State private var username = ""
        @State private var password = ""
        
        var body: some View {
            VStack {
                TextField("Username", text: $username)  // Placeholder and binding
                SecureField("Password", text: $password) // SecureField hides text (for passwords)
            }
        }
    }
    ```

11. **List** creates scrollable lists. Can display static content or dynamic content from arrays.
    ```swift
    // Static list
    List {
        Text("Item 1")
        Text("Item 2") 
        Text("Item 3")
    }
    
    // Dynamic list from array
    struct TodoList: View {
        let todos = ["Buy milk", "Walk dog", "Code app"]
        
        var body: some View {
            List(todos, id: \.self) { todo in  // id: \.self means use the string itself as ID
                Text(todo)
            }
        }
    }
    ```

12. **NavigationView** and **NavigationLink** handle navigation between screens. NavigationView is the container, NavigationLink creates tappable items that navigate.
    ```swift
    NavigationView {
        VStack {
            NavigationLink("Go to Detail", destination: DetailView())  // Tapping navigates to DetailView
            NavigationLink("Go to Settings", destination: SettingsView())
        }
        .navigationTitle("Home")  // Sets the navigation bar title
    }
    ```

13. **Sheet** presents modal screens that slide up from bottom. Use `@State` to control when sheet is shown.
    ```swift
    struct MainView: View {
        @State private var showingSheet = false  // Controls if sheet is visible
        
        var body: some View {
            Button("Show Sheet") {
                showingSheet = true  // This makes sheet appear
            }
            .sheet(isPresented: $showingSheet) {  // Sheet appears when showingSheet is true
                ModalView()
            }
        }
    }
    ```

## Layout and Styling

14. **Padding** adds space around views. **Spacing** in stacks controls space between items.
    ```swift
    VStack(spacing: 20) {  // 20 points between each item in stack
        Text("First")
        Text("Second")
    }
    .padding()  // Adds default padding around entire stack
    .padding(.horizontal, 30)  // Adds 30 points padding only left and right
    ```

15. **Frame** controls view size. **Background** adds color or other views behind content.
    ```swift
    Text("Hello")
        .frame(width: 200, height: 100)  // Makes text take exact size
        .background(Color.blue)          // Blue background behind text
        .cornerRadius(10)                // Rounded corners
    ```

16. **Alignment** in stacks controls how items line up. Default is center, but you can specify leading, trailing, top, bottom, etc.
    ```swift
    VStack(alignment: .leading) {  // All items align to left edge
        Text("Short")
        Text("This is a longer text")
        Text("Medium")
    }
    
    HStack(alignment: .top) {  // All items align to top edge
        Text("Top aligned")
        VStack {
            Text("Line 1")
            Text("Line 2")
        }
    }
    ```

## Modifiers and Styling

17. **Modifiers change views and return new views.** Order matters! Each modifier creates a new view, so the order affects the final result.
    ```swift
    Text("Hello")
        .padding()        // First: add padding to text
        .background(.red) // Then: add red background behind the padding
        .padding()        // Finally: add more padding outside the red background
    
    // Different order = different result
    Text("Hello")
        .background(.red) // First: red background close to text
        .padding()        // Then: padding outside the red background
    ```

18. **Conditional modifiers** apply styling based on conditions. Use ternary operator or if-else inside modifier.
    ```swift
    struct ToggleButton: View {
        @State private var isSelected = false
        
        var body: some View {
            Button("Toggle") {
                isSelected.toggle()
            }
            .foregroundColor(isSelected ? .white : .black)  // White text if selected, black if not
            .background(isSelected ? .blue : .gray)         // Blue background if selected, gray if not
        }
    }
    ```

19. **Custom modifiers** let you reuse styling across multiple views. Create a struct that conforms to `ViewModifier`.
    ```swift
    struct CardStyle: ViewModifier {
        func body(content: Content) -> some View {  // Content is whatever view this modifier is applied to
            content
                .padding()
                .background(Color.white)
                .cornerRadius(10)
                .shadow(radius: 5)
        }
    }
    
    // Use the custom modifier
    Text("Card content")
        .modifier(CardStyle())
    
    // Or create an extension for easier use
    extension View {
        func cardStyle() -> some View {
            modifier(CardStyle())
        }
    }
    
    Text("Card content").cardStyle()  // Much cleaner!
    ```

## Advanced Layout

20. **GeometryReader** gives you information about the available space. Useful for creating views that adapt to screen size.
    ```swift
    GeometryReader { geometry in  // geometry tells you available width, height, etc.
        HStack {
            Rectangle()
                .fill(Color.red)
                .frame(width: geometry.size.width * 0.3)  // Takes 30% of available width
            
            Rectangle()
                .fill(Color.blue)
                .frame(width: geometry.size.width * 0.7)  // Takes 70% of available width
        }
    }
    ```

21. **LazyVStack** and **LazyHStack** only create views when they're needed. Use for long lists to improve performance.
    ```swift
    ScrollView {
        LazyVStack {  // Only creates visible items, not all 1000 at once
            ForEach(1...1000, id: \.self) { number in
                Text("Item \(number)")
                    .padding()
            }
        }
    }
    ```

22. **Grid layouts** arrange items in rows and columns. Use `LazyVGrid` for vertical scrolling grids, `LazyHGrid` for horizontal scrolling.
    ```swift
    let columns = [
        GridItem(.flexible()),  // Column that adjusts to fit content
        GridItem(.flexible()),
        GridItem(.flexible())
    ]
    
    ScrollView {
        LazyVGrid(columns: columns, spacing: 20) {  // 3 columns with 20 points between items
            ForEach(photos, id: \.id) { photo in
                Image(photo.name)
                    .aspectRatio(1, contentMode: .fit)  // Square images
            }
        }
    }
    ```

## Animation and Transitions

23. **Implicit animations** automatically animate when @State changes. Just add `.animation()` modifier.
    ```swift
    struct AnimatedButton: View {
        @State private var isExpanded = false
        
        var body: some View {
            Button("Tap me") {
                isExpanded.toggle()
            }
            .frame(width: isExpanded ? 200 : 100, height: 50)  // Size changes
            .background(isExpanded ? .blue : .red)             // Color changes
            .animation(.easeInOut(duration: 0.5), value: isExpanded)  // Animates all changes
        }
    }
    ```

24. **Explicit animations** give you more control. Use `withAnimation` to animate specific changes.
    ```swift
    struct ExplicitAnimation: View {
        @State private var offset = 0.0
        
        var body: some View {
            Circle()
                .offset(x: offset)
                .onTapGesture {
                    withAnimation(.spring()) {  // Only animates changes inside this block
                        offset = offset == 0 ? 100 : 0
                    }
                }
        }
    }
    ```

25. **Transitions** control how views appear and disappear. Common transitions: `.slide`, `.opacity`, `.scale`.
    ```swift
    struct TransitionDemo: View {
        @State private var showDetails = false
        
        var body: some View {
            VStack {
                Button("Toggle Details") {
                    withAnimation {
                        showDetails.toggle()
                    }
                }
                
                if showDetails {
                    Text("Here are the details!")
                        .transition(.slide)  // Slides in from side when appearing
                }
            }
        }
    }
    ```

## Data Flow Patterns

26. **Environment Objects** pass data deep into view hierarchies without manual passing. Perfect for app-wide data like user settings.
    ```swift
    class AppSettings: ObservableObject {
        @Published var isDarkMode = false
        @Published var userName = ""
    }
    
    @main
    struct MyApp: App {
        let settings = AppSettings()
        
        var body: some Scene {
            WindowGroup {
                ContentView()
                    .environmentObject(settings)  // Makes settings available to all child views
            }
        }
    }
    
    struct DeepChildView: View {
        @EnvironmentObject var settings: AppSettings  // Gets settings from environment
        
        var body: some View {
            Text("Welcome, \(settings.userName)")
        }
    }
    ```

27. **@AppStorage** automatically saves simple data to UserDefaults. Perfect for user preferences.
    ```swift
    struct SettingsView: View {
        @AppStorage("username") private var username = ""  // Automatically saved to UserDefaults
        @AppStorage("isDarkMode") private var isDarkMode = false
        
        var body: some View {
            VStack {
                TextField("Username", text: $username)  // Changes automatically save
                Toggle("Dark Mode", isOn: $isDarkMode)   // Changes automatically save
            }
        }
    }
    ```

## Common Patterns and Best Practices

28. **Extract subviews** when views get complex. Break large views into smaller, focused pieces.
    ```swift
    // Instead of one huge view, break it down:
    struct ProfileView: View {
        let user: User
        
        var body: some View {
            VStack {
                ProfileHeader(user: user)    // Separate view for header
                ProfileStats(user: user)     // Separate view for stats  
                ProfileActions(user: user)   // Separate view for action buttons
            }
        }
    }
    
    struct ProfileHeader: View {
        let user: User
        
        var body: some View {
            HStack {
                AsyncImage(url: user.avatarURL)  // AsyncImage loads images from URLs
                VStack(alignment: .leading) {
                    Text(user.name).font(.title)
                    Text(user.email).foregroundColor(.secondary)
                }
            }
        }
    }
    ```

29. **Use computed properties** for conditional content instead of complex if-else in body.
    ```swift
    struct WeatherView: View {
        let temperature: Double
        
        // Computed property makes body cleaner
        private var weatherDescription: String {
            if temperature > 80 {
                return "It's hot! 🔥"
            } else if temperature > 60 {
                return "Nice weather 😊"
            } else {
                return "It's cold 🥶"
            }
        }
        
        private var weatherColor: Color {
            temperature > 80 ? .red : temperature > 60 ? .green : .blue
        }
        
        var body: some View {
            Text(weatherDescription)
                .foregroundColor(weatherColor)
        }
    }
    ```

30. **Preview providers** let you see your views in Xcode's preview. Create multiple previews for different states.
    ```swift
    struct ContentView: View {
        var body: some View {
            Text("Hello, World!")
        }
    }
    
    struct ContentView_Previews: PreviewProvider {
        static var previews: some View {
            Group {
                ContentView()
                    .previewDisplayName("Light Mode")
                
                ContentView()
                    .preferredColorScheme(.dark)  // Dark mode preview
                    .previewDisplayName("Dark Mode")
                
                ContentView()
                    .previewDevice("iPhone SE (3rd generation)")  // Different device
                    .previewDisplayName("Small Screen")
            }
        }
    }
    ```

## Error Handling and Loading States

31. **Handle loading and error states** in your views. Users should always know what's happening.
    ```swift
    struct UserListView: View {
        @StateObject private var viewModel = UserListViewModel()
        
        var body: some View {
            NavigationView {
                Group {  // Group lets you switch between different views
                    if viewModel.isLoading {
                        ProgressView("Loading users...")  // Shows spinning indicator
                    } else if let errorMessage = viewModel.errorMessage {
                        VStack {
                            Text("Error: \(errorMessage)")
                                .foregroundColor(.red)
                            Button("Retry") {
                                viewModel.loadUsers()
                            }
                        }
                    } else {
                        List(viewModel.users) { user in
                            Text(user.name)
                        }
                    }
                }
                .navigationTitle("Users")
                .onAppear {
                    viewModel.loadUsers()  // Load data when view appears
                }
            }
        }
    }
    
    class UserListViewModel: ObservableObject {
        @Published var users: [User] = []
        @Published var isLoading = false
        @Published var errorMessage: String?
        
        func loadUsers() {
            isLoading = true
            errorMessage = nil
            
            // Simulate API call
            Task {
                do {
                    let fetchedUsers = try await UserAPI.fetchUsers()
                    await MainActor.run {  // Update UI on main thread
                        self.users = fetchedUsers
                        self.isLoading = false
                    }
                } catch {
                    await MainActor.run {
                        self.errorMessage = error.localizedDescription
                        self.isLoading = false
                    }
                }
            }
        }
    }
    ```

## Thumb Rules for SwiftUI

1. **Start with @State for simple data, move to ObservableObject for complex data.** If your data is just a string or number that changes, use @State. If it's complex data or shared between views, use ObservableObject.

2. **Modifier order matters.** `.padding().background(.red)` is different from `.background(.red).padding()`. Think of each modifier as wrapping the previous view.

3. **Extract views early.** If your `body` has more than 10 lines, consider breaking it into smaller views. It makes code easier to read and reuse.

4. **Use computed properties for conditional logic.** Instead of complex if-else statements in your view body, create computed properties that return the conditional values.

5. **Always handle loading and error states.** Users should never see a blank screen or wonder if something is broken. Show progress indicators and error messages.

6. **Use preview providers extensively.** They make development much faster. Create previews for different data states (empty, loading, error, success).

7. **Prefer @StateObject over @ObservedObject when creating objects.** Use @StateObject when your view creates the object, @ObservedObject when the object comes from somewhere else. 