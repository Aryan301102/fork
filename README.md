# The Jakt programming language

**Jakt** is a memory-safe systems programming language.

It currently transpiles to C++.

**NOTE:** The language is under heavy development.

## Usage

```
jakt file.jakt
clang++ -std=c++20 -Iruntime -Wno-user-defined-literals output.cpp
```

## Goals

1. Memory safety
2. Code readability
3. Developer productivity
4. Executable performance
5. Fun!

## Memory safety

The following strategies are employed to achieve memory safety:
- Automatic reference counting
- Strong typing
- Bounds checking
- No raw pointers in safe mode

In **Jakt**, there are three pointer types:

- [x] **T** (Strong pointer to reference-counted class `T`.)
- [ ] **weak T?** (Weak pointer to reference-counted class `T`. Becomes empty on pointee destruction.)
- [x] **raw T** (Raw pointer to arbitrary type `T`. Only usable in `unsafe` blocks.)

Null pointers are not possible in safe mode, but pointers can be wrapped in `Optional`, i.e `Optional<T>` or `T?` for short.

Note that **weak** pointers must always be wrapped in `Optional`. There is no **weak T**, only **weak T?**.

## Math safety

- [x] Integer overflow (both signed and unsigned) is a runtime error.
- [x] Numeric values are not automatically coerced to `int`. All casts must be explicit.

For cases where silent integer overflow is desired, there are explicit functions that provide this functionality.

## Code readability

Far more time is spent reading code than writing it. For that reason, **Jakt** puts a high emphasis on readability.

Some of the features that encourage more readable programs:

- [x] Immutable by default.
- [x] Argument labels in call expressions (`object.function(width: 10, height: 5)`)
- [ ] Inferred `enum` scope. (You can say `Foo` instead of `MyEnum::Foo`).
- [x] Pattern matching with `match`.
- [ ] Optional chaining (`foo?.bar?.baz` (fallible) and `foo!.bar!.baz` (infallible))
- [x] None coalescing for optionals (`foo ?? bar` yields `foo` if `foo` has a value, otherwise `bar`)
- [x] `defer` statements.
- [x] Pointers are always dereferenced with `.` (never `->`)
- [ ] Trailing closure parameters can be passed outside the call parentheses.
- [ ] Error propagation with `ErrorOr<T>` return type and dedicated `try` / `must` keywords.

## Function calls

When calling a function, you must specify the name of each argument as you're passing it:

```jakt
rect.set_size(width: 640, height: 480)
```

There are two exceptions to this:

- [x] If the parameter in the function declaration is declared as `anonymous`, omitting the argument label is allowed.
- [x] When passing a variable with the same name as the parameter.

## Structures and classes

There are two main ways to declare a structure in Jakt: `struct` and `class`.

### `struct`

Basic syntax:

```jakt
struct Point {
    x: i64
    y: i64
}
```

Structs in Jakt have *value semantics*:
- Variables that contain a struct always have a unique instance of the struct.
- Copying a `struct` instance always makes a deep copy.

```jakt
let a = Point(x: 10, y: 5)
let b = a
// "b" is a deep copy of "a", they do not refer to the same Point
```

**Jakt** generates a default constructor for structs. It takes all fields by name. For the `Point` struct above, it looks like this:

```jakt
Point(x: i64, y: i64)
```

Struct members are *public* by default.

### `class`

- [x] basic class support
- [ ] private-by-default members
- [ ] inheritance
- [ ] class-based polymorphism (assign child instance to things requiring the parent type)
- [ ] `Super` type
- [ ] `Self` type

Same basic syntax as `struct`:
```
class Size {
    width: i64
    height: i64

    public function area(this) => this.width * this.height
}
```

Classes in Jakt have *reference semantics*:
- Copying a `class` instance (aka an "object") copies a reference to the object.
- All objects are reference-counted by default. This ensures that objects don't get accessed after being deleted.

Class members are *private* by default.

### Member functions

Both structs and classes can have member functions.

There are three kinds of member functions:

**Static member functions** don't require an object to call. They have no `this` parameter.

```jakt
class Foo {
    function func() => println("Hello!")
}

// Foo::func() can be called without an object.
Foo::func()
```

**Non-mutating member functions** require an object to be called, but cannot mutate the object. The first parameter is `this`.

```jakt
class Foo {
    function func(this) => println("Hello!")
}

// Foo::func() can only be called on an instance of Foo.
let x = Foo()
x.func()
```

**Mutating member functions** require an object to be called, and may modify the object. The first parameter is `mutable this`.
```jakt
class Foo {
    x: i64

    function set(mutable this, anonymous x: i64) {
        this.x = x
    }
}

// Foo::set() can only be called on a mutable Foo:
let mutable foo = Foo(x: 3)
foo.set(9)
```

## Arrays

Dynamic arrays are provided via a built-in `Array<T>` type. They can grow and shrink at runtime.

`Array` is memory safe:
- Out-of-bounds will panic the program with a runtime error.
- Slices of an `Array` keep the underlying data alive via automatic reference counting.

### Declaring arrays

```jakt
// Function that takes an Array<i64> and returns an Array<String>
function foo(numbers: [i64]) -> [String] {
    ...
}
```

### Shorthand for creating arrays

```jakt
// Array<i64> with 256 elements, all initialized to 0.
let values = [0; 256]

// Array<String> with 3 elements: "foo", "bar" and "baz".
let values = ["foo", "bar", "baz"]
```

## Dictionaries

- [x] Creating dictionaries
- [x] Indexing dictionaries
- [ ] Assigning into indexes (aka lvalue)

```jakt
function main() {
    let dict = ["a": 1, "b": 2]

    println("{}", dict["a"]!)
}
```

### Declaring dictionaries

```jakt
// Function that takes a Dictionary<i64, String> and returns an Dictionary<String, bool>
function foo(numbers: [i64:String]) -> [String:bool] {
    ...
}
```

### Shorthand for creating dictionaries

```jakt
// Dictionary<String, i64> with 3 entries.
let values = ["foo": 500, "bar": 600, "baz": 700]
```

## Sets

- [x] Creating sets
- [ ] Reference semantics

```jakt
function main() {
    let set = {1, 2, 3}

    println("{}", set.contains(1))
    println("{}", set.contains(5))
}
```

## Tuples

- [x] Creating tuples
- [x] Index tuples
- [ ] Tuple types

```
function main() {
    let x = ("a", 2, true)

    println("{}", x.1)
}
```

## Enums and Pattern Matching

- [x] Enums as sum-types
- [x] Generic enums
- [x] Enums as names for values of an underlying type
- [x] `match` expressions
- [x] Enum scope inference in `match` arms
- [ ] Nested `match` patterns
- [ ] Traits as `match` patterns
- [ ] Support for interop with the `?`, `??` and `!` operators

```jakt
enum MyOptional<T> {
    Some: T
    None
}

function value_or_default<T>(anonymous x: MyOptional<T>, default: T) -> T {
    return match x {
        Some(value) => value
        None => default
    }
}

enum Foo {
    StructLikeThingy {
        field_a: i32
        field_b: i32
    }
}

function look_at_foo(anonymous x: Foo) -> i32 {
    match x {
        StructLikeThingy(field_a: a, field_b: b) => {
            return a + b
        }
    }
}

enum AlertDescription: i8 {
    CloseNotify = 0
    UnexpectedMessage = 10
    BadRecordMAC = 20
    // etc
}

// Use in match:
function do_nothing_in_particular() => match AlertDescription::CloseNotify {
    CloseNotify => { ... }
    UnexpectedMessage => { ... }
    BadRecordMAC => { ... }
}
```

## Generics

- [x] Generic types
- [x] Generic type inference
- [ ] Traits

Jakt supports both generic structures and generic functions. 

```jakt
function id<T>(anonymous x: T) -> T {
    return x
}

function main() {
    let y = id(3)

    println("{}", y + 1000)
}
```

```jakt
struct Foo<T> {
    x: T
}

function main() {
    let f = Foo(x: 100)

    println("{}", f.x)
}
```

## Namespaces

- [x] Namespace support for functions and struct/class/enum
- [ ] Deep namespace support

```
namespace Greeters {
    function greet() {
        println("Well, hello friends")
    }
}

function main() {
    Greeters::greet()
}
```

## Type casts

There are four built-in casting operators in **Jakt**.

### Casts for all types

- `as? T`: Returns an `Optional<T>`, empty if the source value isn't convertible to `T`.
- `as! T`: Returns a `T`, aborts the program if the source value isn't convertible to `T`.

### Casts specific to numeric types

- `as truncated T`: Returns a `T` with out-of-range values truncated in a manner specific to each type.
- `as saturated T`: Returns a `T` with the out-of-range values saturated to the minimum or maximum value possible for `T`.

## Traits

**(Not yet implemented)**

To make generics a bit more powerful and expressive, you can add additional information to them:

```jakt
trait Hashable {
    function hash(self) -> i128
}

class Foo implements Hashable {
    function hash(self) => 42
}

type i64 implements Hashable {
    function hash(self) => 100
}
```

The intention is that generics use traits to limit what is passed into a generic parameter, and also to grant that variable more capabilities in the body. It's not really intended to do vtable types of things (for that, just use a subclass)

## Safety analysis

**(Not yet implemented)**

To keep things safe, there are a few kinds of analysis we'd like to do (non-exhaustive):

* Preventing overlapping of method calls that would collide with each other. For example, creating an iterator over a container, and while that's live, resizing the container
* Using and manipulating raw pointers
* Calling out to C code that may have side effects

## Error handling

Functions that can fail with an error instead of returning normally are marked with the `throws` keyword:

```jakt
function task_that_might_fail() throws -> usize {
    if problem {
        throw Error::from_errno(EPROBLEM)
    }
    ...
    return result
}

function task_that_cannot_fail() -> usize {
    ...
    return result
}
```

Unlike languages like C++ and Java, errors don't unwind the call stack automatically. Instead, they bubble up to the nearest caller.

If nothing else is specified, calling a function that `throws` from within a function that `throws` will implicitly bubble errors.

### Syntax for catching errors

If you want to catch errors locally instead of letting them bubble up to the caller, use a `try`/`catch` construct like this:

```jakt
try {
    task_that_might_fail()
} catch error {
    println("Caught error: {}", error)
}
```

There's also a shorter form:

```jakt
try task_that_might_fail() catch error {
    println("Caught error: {}", error)
}
```

### Rethrowing errors

**(Not yet implemented)**

## Inline C++

For better interoperability with existing C++ code, as well as situations where the capabilities of **Jakt** within `unsafe` blocks are not powerful enough, the possibility of embedding inline C++ code into the program exists in the form of `cpp` blocks:

```jakt
let mutable x = 0
unsafe {
    cpp {
        "x = (i64)&x;"
    }
}
println("{}", x)
```
