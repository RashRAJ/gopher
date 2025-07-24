## Composite Types in Go

Besides the basic types, Go provides several **composite types**, which allow for more complex data structures. These include:

| Type      | Description                                                             | Example               |
|-----------|-------------------------------------------------------------------------|-----------------------|
| Pointer   | The memory address of a variable                                        | `*int`                |
| Array     | A container of elements of the same type with a fixed length            | `[2]int`              |
| Slice     | A contiguous segment of an array                                        | `[]int`               |
| Map       | A dictionary or associative array                                       | `map[int]int`         |
| Struct    | A collection of fields that can have different types                    | `struct { value int }`|
| Function  | A function type with specified parameters and return values             | `func(int, int) int`  |
| Channel   | Typed pipes used for communicating values of a specified type           | `chan int`            |
| Interface | A collection of method signatures that a type must implement            | `interface{}`         |

The empty interface, interface{}, is a generic type that can contain any value. Since this interface has no requirements (methods), it can be satisfied by any value.

Interfaces, pointers, slices, functions, channels, and maps can have a void value, which is represented in Go by nil:

Pointers are self-explanatory; they are not referring to any variable address.
The interface's underlying value can be empty.
Other pointer types, like slices or channels, can be empty.

Custom-defined types
A package can define its own types by using the type defined definition expression, where definition is a type that shares a defined memory representation. Custom types can be defined by basic types:
```
type Message string    // custom string
type Counter int       // custom integer
type Number float32    // custom float
type Success bool      // custom boolean
They can also be defined by composite types like slices, maps, or pointers:

type StringDuo [2]string            // custom array   
type News chan string               // custom channel
type Score map[string]int           // custom map
type IntPtr *int                    // custom pointer
type Transform func(string) string  // custom function
type Result struct {                // custom struct
    A, B int
}
```
They can also be used in combination with other custom types:
```
type Broadcast Message // custom Message
type Timer Counter     // custom Counter
type News chan Message // custom channel of custom type Message
```
The main uses of custom types are to define methods and to make the type specific for a scope, like defining a string type called Message.

Interface definitions work in a different way. They can be defined by specifying a series of different methods, for instance:
```
type Reader interface {
    Read(p []byte) (n int, err error)
}
They can also be a composition of other interfaces:

type ReadCloser interface {
    Reader 
    Closer 
}
Alternatively, they can be a combination of the two interfaces:

type ReadCloser interface {
    Reader        // composition
    Close() error // method
}
```


Some types need to use built-in functions so that they can be initialized correctly:

- `new` makes it possible to create a pointer of a certain type while already allocating some space for the underlying variable.
- make initializes slices, maps, and channels:
- Slices need an extra argument for the size, and an optional one for the capacity of the underlying array.
- Maps can have one argument for their initial capacity.
- Channels can also have one argument for its capacity. They are unlike maps, which cannot change.

```
a := new(int)                   // pointer of a new in variable
sliceEmpty := make([]int, 0)    // slice of int of size 0, and a capacity of 0
sliceCap := make([]int, 0, 10)  // slice of int of size 0, and a capacity of 10
map1 = make(map[string]int)     // map with default capacity
map2 = make(map[string]int, 10) // map with a capacity of 10
ch1 = make(chan int)            // channel with no capacity (unbuffered)
ch2 = make(chan int, 10)        // channel with capacity of 10 (buffered)
```


## Operators in Go

| Operator | Name                       | Description                                               | Example     |
|----------|----------------------------|-----------------------------------------------------------|-------------|
| `=`      | Assignment                 | Assigns the value to a variable                           | `a = 10`    |
| `:=`     | Declaration and assignment | Declares a variable and assigns a value to it             | `a := 0`    |
| `==`     | Equals                     | Compares two values, returns true if they are equal       | `a == b`    |
| `!=`     | Not equals                 | Compares two values, returns true if they are not equal   | `a != b`    |
| `+`      | Plus                       | Adds two numeric values                                   | `a + b`     |
| `-`      | Minus                      | Subtracts one numeric value from another                  | `a - b`     |
| `*`      | Times                      | Multiplies two numeric values                             | `a * b`     |
| `/`      | Divide                     | Divides one numeric value by another                      | `a / b`     |
| `%`      | Modulo                     | Returns the remainder after division                      | `a % b`     |
| `&`      | AND                        | Bitwise AND                                               | `a & b`     |
| `&^`     | Bit clear                  | Bitwise AND NOT (clears bits)                             | `a &^ b`    |
| `<<`     | Left shift                 | Shifts bits to the left                                   | `a << b`    |
| `>>`     | Right shift                | Shifts bits to the right                                  | `a >> b`    |
| `&&`     | Boolean AND                | Logical AND                                               | `a && b`    |
| `||`     | Boolean OR                 | Logical OR                                                | `a || b`    |
| `!`      | Boolean NOT                | Logical NOT                                               | `!a`        |
| `<-`     | Receive                    | Receives value from a channel                             | `<-a`       |
| `<-`     | Send                       | Sends value to a channel                                  | `a <- b`    |
| `&`      | Reference                  | Gets the address of a variable                            | `&a`        |
| `*`      | Dereference                | Gets the value from a pointer                             | `*a`        |


| Symbol | Name        | Think of it as...                | Used for...                               |
| ------ | ----------- | -------------------------------- | ----------------------------------------- |
| `&`    | Reference   | “Where is this stored?”          | Getting the address of a variable         |
| `*`    | Dereference | “What’s stored at this address?” | Accessing/modifying the value via pointer |

& — Reference Operator
Purpose: Gets the memory address of a variable.

Meaning: “Give me the address where this value is stored.”

Usage: Used to create a pointer to a variable.

Example:

go
```
a := 42
p := &a  // p now holds the address of variable a
fmt.Println(p)  // prints something like 0xc000018030
Now p is a pointer to a.
```

* — Dereference Operator
Purpose: Gets the value from a pointer.

Meaning: “Go to the address this pointer holds and get the value.”

Usage: Used to access or modify the value stored at the memory address.

Example:

go
```
a := 42
p := &a      // p is a pointer to a
fmt.Println(*p)  // prints 42 (dereferencing p to get the value of a)

*p = 100     // changes the value of a through the pointer
fmt.Println(a)   // prints 100
```

## Casting
Converting a type into another type is an operation called casting, which works slightly differently for interfaces and concrete types:

Interfaces can be casted to a concrete type that implements it. This conversion can return a second value (a Boolean) and show whether the conversion was successful or not. If the Boolean variable is omitted, the application will panic on a failed casting.
With concrete types, casting can happen between types that have the same memory structure, or it can happen between numerical types:




