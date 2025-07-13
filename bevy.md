# Bevy Game Engine Documentation

## What is Bevy?

Bevy is a game engine written in Rust programming language. A game engine is a software framework that helps you create video games. Bevy makes it easier to build games by providing ready-made tools and systems.

## Setting Up Bevy

### Installing Bevy

First, you need to add Bevy to your Rust project. In your `Cargo.toml` file, add this:

```toml
[dependencies]
bevy = "0.11"
```

The `[dependencies]` section tells Rust which external libraries your project needs. The `bevy = "0.11"` line means we want to use version 0.11 of Bevy.

### Basic Bevy Program

Here's the simplest Bevy program:

```rust
use bevy::prelude::*;

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .run();
}
```

Let's break this down:
- `use bevy::prelude::*;` - This imports all the commonly used Bevy items into our program
- `App::new()` - This creates a new Bevy application. Think of it as starting a new game
- `.add_plugins(DefaultPlugins)` - This adds basic functionality like window management, input handling, and rendering to our app
- `.run()` - This starts the application and keeps it running until you close it

## Core Concepts

### 1. Entity Component System (ECS)

Bevy uses something called ECS. This is a way to organize game objects.

#### Entity
An entity is like a unique ID number for a game object. It doesn't contain any data itself, just represents "something" in your game.

#### Component
A component is data that describes one aspect of an entity. For example:
- Position component stores where something is located
- Velocity component stores how fast something is moving
- Health component stores how much health something has

#### System
A system is a function that does something with entities and their components.

### 2. Components

Let's create some basic components:

```rust
use bevy::prelude::*;

#[derive(Component)]
struct Position {
    x: f32,
    y: f32,
}

#[derive(Component)]
struct Velocity {
    x: f32,
    y: f32,
}

#[derive(Component)]
struct Health {
    value: i32,
}
```

Explanation:
- `#[derive(Component)]` - This tells Bevy that this struct is a component
- `struct Position` - This creates a new data type called Position
- `x: f32, y: f32` - These are fields that store decimal numbers (f32 means 32-bit floating point)
- `value: i32` - This stores a whole number (i32 means 32-bit integer)

### 3. Systems

Systems are functions that work with components. Here's a simple movement system:

```rust
fn movement_system(mut query: Query<(&mut Position, &Velocity)>) {
    for (mut position, velocity) in query.iter_mut() {
        position.x += velocity.x;
        position.y += velocity.y;
    }
}
```

Explanation:
- `mut query: Query<(&mut Position, &Velocity)>` - This queries (asks for) all entities that have both Position and Velocity components
- `&mut Position` - The `&mut` means we can change the Position
- `&Velocity` - The `&` means we can only read the Velocity
- `for (mut position, velocity) in query.iter_mut()` - This loops through all entities that match our query
- `position.x += velocity.x` - This adds the velocity to the position (making the object move)

### 4. Resources

Resources are global data that any system can access. They're like global variables but managed by Bevy.

```rust
#[derive(Resource)]
struct GameScore {
    value: i32,
}

fn setup_game(mut commands: Commands) {
    commands.insert_resource(GameScore { value: 0 });
}
```

Explanation:
- `#[derive(Resource)]` - This tells Bevy this is a resource
- `commands.insert_resource()` - This adds the resource to our game

### 5. Spawning Entities

To create game objects, you spawn entities with components:

```rust
fn spawn_player(mut commands: Commands) {
    commands.spawn((
        Position { x: 0.0, y: 0.0 },
        Velocity { x: 1.0, y: 0.0 },
        Health { value: 100 },
    ));
}
```

Explanation:
- `commands.spawn()` - This creates a new entity
- The tuple `(Position { ... }, Velocity { ... }, Health { ... })` - This adds multiple components to the entity at once

### 6. Events

Events are messages that systems can send to each other:

```rust
#[derive(Event)]
struct PlayerDiedEvent {
    player_id: u32,
}

fn health_system(
    mut query: Query<(Entity, &mut Health)>,
    mut death_events: EventWriter<PlayerDiedEvent>,
) {
    for (entity, mut health) in query.iter_mut() {
        if health.value <= 0 {
            death_events.send(PlayerDiedEvent {
                player_id: entity.index(),
            });
        }
    }
}

fn handle_death(mut death_events: EventReader<PlayerDiedEvent>) {
    for event in death_events.iter() {
        println!("Player {} died!", event.player_id);
    }
}
```

Explanation:
- `#[derive(Event)]` - This makes our struct an event
- `EventWriter<PlayerDiedEvent>` - This allows a system to send events
- `EventReader<PlayerDiedEvent>` - This allows a system to receive events
- `death_events.send()` - This sends an event
- `death_events.iter()` - This reads all events that were sent

## Basic Game Example

Here's a complete example that creates a simple moving square:

```rust
use bevy::prelude::*;

#[derive(Component)]
struct Position {
    x: f32,
    y: f32,
}

#[derive(Component)]
struct Velocity {
    x: f32,
    y: f32,
}

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_systems(Startup, setup)
        .add_systems(Update, (movement_system, render_system))
        .run();
}

fn setup(mut commands: Commands) {
    // Create a camera
    commands.spawn(Camera2dBundle::default());
    
    // Create a moving square
    commands.spawn((
        Position { x: 0.0, y: 0.0 },
        Velocity { x: 50.0, y: 0.0 },
        SpriteBundle {
            sprite: Sprite {
                color: Color::RED,
                custom_size: Some(Vec2::new(50.0, 50.0)),
                ..default()
            },
            ..default()
        },
    ));
}

fn movement_system(
    time: Res<Time>,
    mut query: Query<(&mut Position, &Velocity)>,
) {
    for (mut position, velocity) in query.iter_mut() {
        position.x += velocity.x * time.delta_seconds();
        position.y += velocity.y * time.delta_seconds();
    }
}

fn render_system(
    mut query: Query<(&Position, &mut Transform), Without<Camera>>,
) {
    for (position, mut transform) in query.iter_mut() {
        transform.translation.x = position.x;
        transform.translation.y = position.y;
    }
}
```

Explanation:
- `Camera2dBundle::default()` - This creates a 2D camera to view our game
- `SpriteBundle` - This is a bundle (group) of components for displaying 2D images
- `Sprite { color: Color::RED, ... }` - This creates a red square
- `Vec2::new(50.0, 50.0)` - This sets the size to 50x50 pixels
- `time.delta_seconds()` - This gives us the time since the last frame, making movement smooth
- `Transform` - This is Bevy's built-in component for position, rotation, and scale
- `Without<Camera>` - This excludes camera entities from our query

## Schedules

Schedules control when and in what order systems run. Bevy has built-in schedules for different phases of the game loop:

```rust
use bevy::prelude::*;

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_systems(Startup, setup_game)           // Runs once at startup
        .add_systems(Update, (move_player, update_ui))  // Runs every frame
        .add_systems(FixedUpdate, physics_system)   // Runs at fixed intervals
        .add_systems(PostUpdate, cleanup_system)    // Runs after Update
        .run();
}

fn setup_game() {
    println!("Game is starting up!");
}

fn move_player() {
    // Player movement logic
}

fn update_ui() {
    // UI update logic
}

fn physics_system() {
    // Physics calculations that need consistent timing
}

fn cleanup_system() {
    // Clean up temporary data after main updates
}
```

Explanation:
- `Startup` - This schedule runs once when the game starts
- `Update` - This schedule runs every frame (most game logic goes here)
- `FixedUpdate` - This schedule runs at regular intervals regardless of frame rate
- `PostUpdate` - This schedule runs after Update, useful for cleanup
- `(move_player, update_ui)` - This tuple makes both systems run in the same schedule

### System Ordering

You can control the order systems run within a schedule:

```rust
fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_systems(Update, (
            input_system,
            movement_system.after(input_system),
            collision_system.after(movement_system),
        ))
        .run();
}
```

Explanation:
- `.after(input_system)` - This makes movement_system run after input_system
- Systems in the same tuple run in parallel unless you specify ordering

## World

World is the central storage for all game data. It holds all entities, components, and resources. Usually you don't work with World directly, but it's important to understand.

```rust
use bevy::prelude::*;

fn direct_world_access(world: &mut World) {
    // Spawn an entity directly in the world
    let entity = world.spawn((
        Position { x: 0.0, y: 0.0 },
        Velocity { x: 10.0, y: 0.0 },
    )).id();
    
    // Get a component from an entity
    if let Some(position) = world.get::<Position>(entity) {
        println!("Entity position: ({}, {})", position.x, position.y);
    }
    
    // Insert a resource into the world
    world.insert_resource(GameScore { value: 100 });
    
    // Get a resource from the world
    if let Some(score) = world.get_resource::<GameScore>() {
        println!("Current score: {}", score.value);
    }
}
```

Explanation:
- `world: &mut World` - This parameter gives you direct access to the world
- `world.spawn()` - This creates a new entity directly in the world
- `world.get::<Position>(entity)` - This gets a component from a specific entity
- `world.insert_resource()` - This adds a resource to the world
- `world.get_resource::<GameScore>()` - This gets a resource from the world

### World in Systems

Most of the time, you work with World indirectly through system parameters:

```rust
fn system_with_world_access(
    mut commands: Commands,        // Indirect access to modify world
    query: Query<&Position>,       // Indirect access to read components
    mut score: ResMut<GameScore>,  // Indirect access to modify resources
) {
    // Commands queue changes to be applied to the world later
    commands.spawn(Position { x: 0.0, y: 0.0 });
    
    // Query gives you access to components in the world
    for position in query.iter() {
        println!("Found position: ({}, {})", position.x, position.y);
    }
    
    // Resources give you access to global data in the world
    score.value += 10;
}
```

Explanation:
- `Commands` - This queues changes to be applied to the world later
- `Query` - This gives you access to entities and components in the world
- `ResMut` - This gives you mutable access to resources in the world
- The world applies all command changes at specific points in the frame

### Creating Your Own Schedule

You can create custom schedules for special timing needs:

```rust
#[derive(ScheduleLabel, Clone, Debug, PartialEq, Eq, Hash)]
struct MyCustomSchedule;

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_schedule(Schedule::new(MyCustomSchedule))
        .add_systems(MyCustomSchedule, my_special_system)
        .add_systems(Update, run_custom_schedule)
        .run();
}

fn my_special_system() {
    println!("Running in custom schedule!");
}

fn run_custom_schedule(world: &mut World) {
    // Manually run our custom schedule
    world.run_schedule(MyCustomSchedule);
}
```

Explanation:
- `#[derive(ScheduleLabel, ...)]` - This makes our struct work as a schedule identifier
- `Schedule::new(MyCustomSchedule)` - This creates a new empty schedule
- `world.run_schedule()` - This manually runs a schedule

## Plugins

Plugins are collections of systems and resources that add functionality to your game:

```rust
struct MyGamePlugin;

impl Plugin for MyGamePlugin {
    fn build(&self, app: &mut App) {
        app.add_systems(Startup, setup_my_game)
           .add_systems(Update, my_game_system);
    }
}

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_plugins(MyGamePlugin)
        .run();
}

fn setup_my_game() {
    println!("Setting up my game!");
}

fn my_game_system() {
    // Your game logic here
}
```

Explanation:
- `impl Plugin for MyGamePlugin` - This tells Bevy how to use our plugin
- `fn build(&self, app: &mut App)` - This function is called when the plugin is added
- `app.add_systems()` - This adds our systems to the app

## Asset Loading

Assets are external files like images, sounds, and models:

```rust
#[derive(Resource)]
struct GameAssets {
    player_texture: Handle<Image>,
}

fn load_assets(mut commands: Commands, asset_server: Res<AssetServer>) {
    let game_assets = GameAssets {
        player_texture: asset_server.load("player.png"),
    };
    commands.insert_resource(game_assets);
}

fn spawn_player_with_texture(
    mut commands: Commands,
    game_assets: Res<GameAssets>,
) {
    commands.spawn(SpriteBundle {
        texture: game_assets.player_texture.clone(),
        ..default()
    });
}
```

Explanation:
- `Handle<Image>` - This is a reference to an image asset
- `asset_server.load("player.png")` - This loads an image file from the assets folder
- `texture: game_assets.player_texture.clone()` - This uses the loaded image as a texture

## Input Handling

Handle keyboard and mouse input:

```rust
fn input_system(
    keyboard_input: Res<Input<KeyCode>>,
    mut query: Query<&mut Velocity>,
) {
    for mut velocity in query.iter_mut() {
        velocity.x = 0.0;
        velocity.y = 0.0;
        
        if keyboard_input.pressed(KeyCode::W) {
            velocity.y = 100.0;
        }
        if keyboard_input.pressed(KeyCode::S) {
            velocity.y = -100.0;
        }
        if keyboard_input.pressed(KeyCode::A) {
            velocity.x = -100.0;
        }
        if keyboard_input.pressed(KeyCode::D) {
            velocity.x = 100.0;
        }
    }
}
```

Explanation:
- `Input<KeyCode>` - This resource tracks keyboard input
- `keyboard_input.pressed(KeyCode::W)` - This checks if the W key is being held down
- `KeyCode::W` - This represents the W key on the keyboard

## States

States help manage different parts of your game (like menu, gameplay, pause):

```rust
#[derive(States, Debug, Clone, PartialEq, Eq, Hash, Default)]
enum GameState {
    #[default]
    Menu,
    Playing,
    Paused,
}

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_state::<GameState>()
        .add_systems(Update, menu_system.run_if(in_state(GameState::Menu)))
        .add_systems(Update, game_system.run_if(in_state(GameState::Playing)))
        .run();
}

fn menu_system(
    mut next_state: ResMut<NextState<GameState>>,
    keyboard_input: Res<Input<KeyCode>>,
) {
    if keyboard_input.just_pressed(KeyCode::Space) {
        next_state.set(GameState::Playing);
    }
}

fn game_system() {
    // Your game logic here
}
```

Explanation:
- `#[derive(States, ...)]` - This makes our enum work as a state
- `#[default]` - This sets the starting state
- `add_state::<GameState>()` - This adds state management to our app
- `run_if(in_state(GameState::Menu))` - This system only runs when in the Menu state
- `next_state.set(GameState::Playing)` - This changes the current state

## Timers

Timers help you do things at specific intervals:

```rust
#[derive(Resource)]
struct SpawnTimer {
    timer: Timer,
}

fn setup_timer(mut commands: Commands) {
    commands.insert_resource(SpawnTimer {
        timer: Timer::from_seconds(2.0, TimerMode::Repeating),
    });
}

fn spawn_enemy_system(
    time: Res<Time>,
    mut spawn_timer: ResMut<SpawnTimer>,
    mut commands: Commands,
) {
    if spawn_timer.timer.tick(time.delta()).just_finished() {
        commands.spawn((
            Position { x: 200.0, y: 0.0 },
            Velocity { x: -50.0, y: 0.0 },
            // Enemy components here
        ));
    }
}
```

Explanation:
- `Timer::from_seconds(2.0, TimerMode::Repeating)` - This creates a timer that triggers every 2 seconds
- `TimerMode::Repeating` - This makes the timer restart after it finishes
- `spawn_timer.timer.tick(time.delta())` - This updates the timer with the current frame time
- `just_finished()` - This returns true only on the frame when the timer finishes

## Sound

Playing sounds in Bevy:

```rust
fn play_sound_system(
    mut commands: Commands,
    asset_server: Res<AssetServer>,
    keyboard_input: Res<Input<KeyCode>>,
) {
    if keyboard_input.just_pressed(KeyCode::Space) {
        commands.spawn(AudioBundle {
            source: asset_server.load("jump.ogg"),
            settings: PlaybackSettings::DESPAWN,
        });
    }
}
```

Explanation:
- `AudioBundle` - This is a bundle for playing sounds
- `source: asset_server.load("jump.ogg")` - This loads a sound file
- `PlaybackSettings::DESPAWN` - This removes the audio entity when the sound finishes

## Cameras

Different types of cameras:

```rust
// 2D Camera
fn setup_2d_camera(mut commands: Commands) {
    commands.spawn(Camera2dBundle::default());
}

// 3D Camera
fn setup_3d_camera(mut commands: Commands) {
    commands.spawn(Camera3dBundle {
        transform: Transform::from_xyz(0.0, 0.0, 10.0)
            .looking_at(Vec3::ZERO, Vec3::Y),
        ..default()
    });
}
```

Explanation:
- `Camera2dBundle` - This creates a 2D camera for 2D games
- `Camera3dBundle` - This creates a 3D camera for 3D games
- `Transform::from_xyz(0.0, 0.0, 10.0)` - This positions the camera at coordinates (0, 0, 10)
- `looking_at(Vec3::ZERO, Vec3::Y)` - This makes the camera look at the origin (0, 0, 0)

## Collision Detection

Basic collision detection:

```rust
#[derive(Component)]
struct Collider {
    size: Vec2,
}

fn collision_system(
    mut query: Query<(Entity, &Position, &Collider)>,
    mut commands: Commands,
) {
    let mut combinations = query.iter_combinations_mut();
    
    while let Some([(entity_a, pos_a, collider_a), (entity_b, pos_b, collider_b)]) = 
        combinations.fetch_next() {
        
        let distance_x = (pos_a.x - pos_b.x).abs();
        let distance_y = (pos_a.y - pos_b.y).abs();
        
        let collision_x = distance_x < (collider_a.size.x + collider_b.size.x) / 2.0;
        let collision_y = distance_y < (collider_a.size.y + collider_b.size.y) / 2.0;
        
        if collision_x && collision_y {
            println!("Collision detected!");
            // Handle collision here
        }
    }
}
```

Explanation:
- `Collider` - This component stores the size of an object's collision box
- `query.iter_combinations_mut()` - This gives us all unique pairs of entities
- `(pos_a.x - pos_b.x).abs()` - This calculates the absolute distance between two positions
- `collision_x && collision_y` - This checks if objects overlap in both x and y directions

## Common Patterns

### Update Position from Transform

```rust
fn sync_position_with_transform(
    mut query: Query<(&mut Position, &Transform)>,
) {
    for (mut position, transform) in query.iter_mut() {
        position.x = transform.translation.x;
        position.y = transform.translation.y;
    }
}
```

### Remove Entities

```rust
fn cleanup_system(
    query: Query<Entity, With<SomeComponent>>,
    mut commands: Commands,
) {
    for entity in query.iter() {
        commands.entity(entity).despawn();
    }
}
```

Explanation:
- `commands.entity(entity).despawn()` - This removes an entity from the game

## Best Practices

1. **Use Systems for Logic**: Put your game logic in systems, not in components
2. **Keep Components Simple**: Components should only store data, not behavior
3. **Use Resources for Global Data**: Things like game settings, scores, etc.
4. **Organize with Plugins**: Group related systems and resources into plugins
5. **Use States for Game Flow**: Manage different parts of your game with states

## Debugging Tips

1. **Use println! for Debug Output**: Add print statements to see what's happening
2. **Check Component Queries**: Make sure your queries match existing entities
3. **Use Bevy Inspector**: Add the `bevy-inspector-egui` crate to see components in real-time
4. **Start Simple**: Begin with basic functionality before adding complexity

This documentation covers the core concepts of Bevy game engine. Each concept builds on the previous ones, so read through them in order to understand how they work together to create games. 