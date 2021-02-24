# Bevy+Rapier Tetris workshop
During this workshop, we'll try to end up with something resembling Tetris, with one twist: A physics
engine.

The goal of the workshop is to use simple Rust for some cheap laughs. The are no rights
and wrongs. A physics engine lives its own life, we can just feed it with any weird data and
something interesting will usually happen.

The idea for this workshop came after one evening of trying out Bevy. It is so surprisingly simple
to use, that I felt it must be ideal for an entry-level Rust workshop.

## Preparations
1. Install Rust: https://www.rust-lang.org/tools/install
2. Install Visual Studio Code
3. Install vscode Rust Analyzer plugin: https://marketplace.visualstudio.com/items?itemName=matklad.rust-analyzer
4. (optional): turn on vscode `[x] Format on save` for idiomatic Rust autoformatting
5. Build and run the game:

```
$ cargo run
```

# Introduction
This project is very small, it has only one source file: `src/main.rs`.

The file already contains skeleton code that sets up a basic Bevy+Rapier environment, with at
least something interesting on the screen. It also contains the `Game` struct, which contains
a bery basic description of a Tetris board, with some utility methods predefined, so
we can get going faster.

## Rust
The Rust syntax and concepts we'll use in this workshop is quite simple. We'll only
work with numerical data, not text. See the end of this README for a quick walkthrough of
basic Rust syntax we'll encounter.

## Bevy
Bevy (https://bevyengine.org) a simple data-driven game engine.
Bevy is designed around an [Entity Component System](https://en.wikipedia.org/wiki/Entity_component_system):

### Entity
All game objects are dumb _Entities_, which can be thought of as some kind of Object ID.

### Component
All properties and behaviour of entities are defined by _Components_. A component is just
some piece of _data_ associated with an entity. It can have any type. Some components are Bevy built-ins,
such as providing an entity with a graphical representation (it would typically need a `Mesh`, `Material`, etc), and in order to have a position and extent on the screen it must have a `Transform`.

In our code we'll use Bevy's `SpriteBundle`, a nifty helper that instantiates many useful components
at the same time, so that we can see graphics instantly.

Other componenent types can be supplied by the client, as one sees fit. To Bevy, there is no distinction between a built-in component and user-supplied component. Cool.

### System
Because components are just "plain old data", they also do absolutely nothing by themselves!
The final piece of the puzzle is _Systems_.
A system is just a _function_ (executed as part of the game loop), which operates on sets of entities matching a specific `Query`.

Queries filter different entities based on the type of the _Components_ associated with the query.

Examples of different query types:
```rust
// Every entity in the game:
Query<Entity>

// The Transform of all entities that have a Transform component:
Query<&Transform>

// Tuple queries match entities that have these two components at the same time:
Query<(&Transform, &mut MyComponent)>
```

Entities get _spawned_ (and maybe later _despawned_) by using Bevy's _command interface_:

```rust
fn my_system(commands: &mut Commands) {
    let my_entity = commands
        .spawn((Component1, Component2))
        .current_entity()
        .unwrap();

    commands.despawn(my_entity);
}
```

There are no rules for the argument structure of systems. A system does not need
to take a command parameter:

```rust
fn my_other_system(query: Query<(Entity, &MyComponent)>) {
    for (entity, my_component) in query.iter() {
        println!("my_component: {}", my_component);
    }
}
```

Several systems bundled together are called _Plugins_. We'll run Bevy with its built-in `DefaultPlugins` enabled, in order to see graphics on the screen.

### Resource
The last little concept to note about Bevy is `Resource`, which is a kind of "singleton object"
managed by Bevy. Inside a system, it may be accessed through an parameter of the `Res` type: `Res<MyResource>`, or `ResMut<MyResource>` if it needs to be mutated.

How everything fits together in the end will hopefully be apparent when we go through the source file!

## Rapier
Rapier is the physics engine. It is separate from Bevy, but thankfully they provide a plugin!

A very high level overview of what the Rapier system(s) do:
1. Compute mechanical interactions in the physics domain
2. Update every Bevy `Transform` associated with respective Entity

Now we can just start to attach physical components to our entities, and when we attach Bevy's graphical components as well, we end up with screen action!

### RigidBody
This component represents some _physical body_ that can be subjected (and reacts to) to physical forces.

The RigidBody stores the entity's mass, position, velocity, acceleration, orientation, angular momentum, etc in the physical domain. A RigidBody alone is shapeless.

### Collider
This component allows Rapier to detect collisions between bodies, and generate the resulting
forces. The collider needs to have a shape. In this simple game
we'll only work with _Cuboids_, rectangular shapes. Those are also the cheapest to detect
collisions for.

# Tasks
## Intro: Debug the deliberately buggy program :P
1. Make the single visible block appear elsewhere than in the boring (0, 0) board position. Read about the coordinate systems, find the bug and fix it!
2. Display all 4 tetromino blocks instead of just one.

## Extend the feature set! (no particular order)
* Make something interesting happen when the tetromino is "sleeping"/reaches physical equilibrium (`tetromino_sleep_detection`)
* _Somehow_ distinguish between the currently user-controlled blocks (tetromino) and board blocks?
* Add rotation/torque (e.g. `A+D`?)
* Make the tetromino blocks stick together (Joints!)
* Try to make it all more colorful and aesthetically compelling
* More tetromino shapes
* Add walls? Too boring?
* Remove "filled" rows? Too boring?

## Bonus?
* Death/gamification/scoring system?

## Crazy ideas?
* Randomize block/tetromino mass/linear damping?
* Let the floor "pivot" so we have to keep it balanced?
* Explosions?
* You name it

# Appendix: Rust syntax
Maybe you're new to Rust, or just need to freshen up on its basic syntax. Here's that
very basic reference.

(Most things in Rust are expressions, but variable declarations aren't)

## Fundamental types
```rust
u8    // unsigned byte
i8    // signed byte
u32   // unsigned 32-bit integer
i32   // signed 32-bit integer
usize // machine-word-sized unsigned integer
f32   // 32-bit floating point number
```

## Variable
```rust
let a = 10;
```

## _Mutable_ variable
```rust
let mut a = 10;
a += 1;
```

## Tuple
```rust
let tuple: (u8, f32) = (42, 42.0);
```

## Vec (the _collection_ one, "List of stuff")
```rust
let v: Vec<i32> = vec![-1, 0, 1];
```

## Function
```rust
fn foo(arg0: f32, arg1: f32) -> f32 {
    let tmp = arg0 + arg1;

    tmp + 1.0
}
```

## Conditional
Note the "missing" parenthesis! It takes some time to get used to.
```rust
let result = if some_bool_expr {
    42
} else {
    7
};
```

## Struct
```rust
struct Foo {
    bar: usize,
    baz: f32
}

impl Foo {
    fn compute_stuff(&self) -> f32 {
        self.bar as f32 + self.baz
    }
}

let foo = Foo {
    bar: 42,
    baz: 1.0
};
foo.compute_stuff();
```

## Enum
```rust
enum Foo {
    A,
    B
}
```

## Match
```rust
let value = match foo {
    Foo::A => 4,
    Foo::B => 5,
    _ => 42
}
```

## Option
```rust
let mut opt: Option<i32> = None;
opt = Some(42);

if let Some(n) = opt {
    println!("The number was defined as {}", n);
}

let squared = match opt {
    Some(n) => n * n,
    None => -1
};
```

## &Reference
Rust is pass-by-value. In order to instead pass (or receive) by reference, we use the `&` operator:

```rust
let value: i32 = 55;
let reference: &i32 = &value;
```

## Deref
```rust
let copied = *reference;
```
