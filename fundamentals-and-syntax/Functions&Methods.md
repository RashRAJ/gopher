
## Functions and methods

Functions in Go are identified by the func keyword, followed by an identifier, eventual arguments, and return values. Functions in Go can return more than one value at a time. The combination of arguments and returned types is referred to as a signature, as shown in the following code:

```
func simpleFunc()
func funcReturn() (a, b int)
func funcArgs(a, b int)
func funcArgsReturns(a, b int) error
```

The part between brackets is the function body, and the return statement can be used inside it for an early interruption of the function. If the function returns values, then the return statement must return values of the same type.

The return values can be named in the signature; they are zero value variables and if the return statement does not specify other values, these values are the ones that are returned:

```
func foo(a int) int {        // no variable for returned type
    if a > 100 {
        return 100
    }
    return a
}

func bar(a int) (b int) {    // variable for returned type
    if a > 100 {
        b = 100
        return               // same as return b
    }
    return a
}
```

Functions are first-class types in Go and they can also be assigned to variables, with each signature representing a different type. They can also be anonymous; in this case, they are called closures. Once a variable is initialized with a function, the same variable can be reassigned with another function with the same signature. Here's an example of assigning a closure to a variable:

```
var a = func(item string) error { 
    if item != "elixir" {
        return errors.New("Gimme elixir!")
    }
    return nil 
}
```
The functions that are declared by an interface are referred to as methods and they can be implemented by custom types. The method implementation looks like a function, with the exception being that the name is preceded by a single parameter of the implementing type. This is just syntactic sugar – the method definition creates a function under the hood, which takes an extra parameter, that is, the type that implements the method.

This syntax makes it possible to define the same method for different types, each of which will act as a namespace for the function declaration. In this way, it is possible to call a method in two different ways, as shown in the following code:

```
type A int

func (a A) Foo() {}

func main() {
    A{}.Foo()  // Call the method on an instance of the type
    A.Foo(A{}) // Call the method on the type and passing an instance as argument  
}
```

It's important to note that a type and its pointer share the same namespace, so the same method can be implemented just for one of them. The same method cannot be defined for both the type and for its pointer, since declaring the method twice (for a type and its pointer) will produce a compile error (method redeclared). Methods cannot be defined for interfaces, only for concrete types, but interfaces can be used in composite types, including function parameters and return values, as we can see in the following examples:
```
// use error interface with chan
type ErrChan chan error
// use error interface in a map
type Result map[string]error

type Printer interface{
    Print()
}
// use the interface as argument
func CallPrint(p Printer) {
    p.Print()
}
```
The built-in package already defines an interface that is used all over the standard library and in all the packages that are available online – the error interface:

```
type error interface {
    Error() string
}
```
This means that whatever type has an Error() string method can be used as an error, and each package can define its error types according to its needs. This can be used to concisely carry information about the error. In this example, we are defining ErrKey, which is specifying that a string key was not found. We don't need anything else besides the key to represent our error, as shown in the following code:
```
type ErrKey string

func (e Errkey) Error() string {
    returm fmt.Errorf("key %q not found", e)
}
```

## Values and pointers

In Go, everything is passed by a value, so when a function or method is invoked, a copy of the variable is made in the stack. This implies that changes that are made to the value are not reflected outside of the called function. Even slices, maps, and other reference types are passed by value, but since their internal structure contains pointers, they act as if they were passed by reference. If a method is defined for a type, it cannot be defined for its pointer and vice versa. The following example has been used to check that the value is updated only inside the method, and that the change does not reflect the main function:
```
package main

import (
    "fmt"
)

type A int

func (a A) Foo() {
    a++
    fmt.Println("foo", a)
}

func main() {
    var a A
    fmt.Println("before", a) // 0
    a.Foo() // 1
    fmt.Println("after", a) // 0
}
```
In order to change the original variable, the argument must be a pointer to the variable itself – the pointer will be copied, but it will reference the same memory area, making it possible to change its value. Note that assigning another value pointer, instead of its content, doesn't change what the original pointer refers to because it is a copy. 

If we use a method for the type instead of its pointer, we won't see the changes being propagated outside the method.

In the following example, we are using a value receiver. This makes the User value in the Birthday method a copy of the User value in main:
```
type User struct {
    Name string
    Age int
}

func (u User) Birthday() {
    u.Age++
    fmt.Println(u.Name, "turns", u.Age)
}

func main() {
    u := User{Name: "Pietro", Age: 30}
    fmt.Println(u.Name, "is now", u.Age)
    u.Birthday()
    fmt.Println(u.Name, "is now", u.Age)
}
```
The full example is available at https://play.golang.org/p/hnUldHLkFJY.

Since the change is applied to the copy, the original value is left intact, as we can see from the second print statement. If we want to change the value in the original object, we have to use the pointer receiver so that the one that's been copied will be the pointer and the changes will be made to the underlying value:
```
func (u *User) Birthday() {
    u.Age++
    fmt.Println(u.Name, "turns", u.Age)
}
```
The full example is available at https://play.golang.org/p/JvnaQL9R7U5.

We can see that using the pointer receiver allows us to change the underlying value, and that we can change one field of the struct or replace the whole struct itself, as shown in the following code:

```
func (u *User) Birthday() {
    *u = User{Name: u.Name, Age: u.Age + 1}
   fmt.Println(u.Name, "turns", u.Age)
}
```
The full example is available at https://play.golang.org/p/3ugBEZqAood.

If we try to change the value of the pointer instead of the underlying one, we will edit a new object that is not related to the one that was created in main, and the changes will not propagate:

```
func (u *User) Birthday() {
    u = &User{Name: u.Name, Age: u.Age + 1}
    fmt.Println(u.Name, "turns", u.Age)
}
```
The full example is available at https://play.golang.org/p/m8u2clKTqEU.

Some types in Go are automatically passed by reference. This happens because these types are defined internally as structs that contain a pointer. This creates a list of the types, along with their internal definition:

Types	Internal definitions
```
map	
struct {
    m *internalHashtable
}
slice	
struct {
    array *internalArray 
    len int
    cap int
}
channel	
struct {
    c *internalChannel
}
```
