## Variables And Data Types In Go

Variables are containers that hold some value. There are two types of variables in Go, one whose value can be changed and the other whose can not we call them `constants`.



We can define a variable using the `var` keyword followed by the variable name and then its data type (what type value it is expected to store). Here are some examples:

```go
var name string = "Go Language"
var number int = 23
var isTrue bool = true

fmt.Println(name)
fmt.Println(int)
fmt.Println(isTrue)
```

> Using just `var name string` and not setting it to any value, it is called variable definition, we are defining what a variable's name is and what kind of data it can store and if it can be reassigned (set to a new value) or not.

> When we assign a value to a variable for the first time it is called variable initialization.

Constants are defined the same way as variables instead of using var we use `const`.

```go
const name = "Vivek"
name = "Harsh" // Error: cannot assign to name 
```



### Data Types

Go is statically typed, meaning, we need to tell the compiler about what type of data we are expecting. 

We have multiple data types for different purposes. Here are some of them:

- String
- Integer
- Float
- Boolean
- Array
- Map

#### String

For textual data, we use the string type. We've been using the string data type from the start.

Anything within a double quote is considered a string. Here's an example:

```go
var text string = "Hello"
```

#### Integers

Integer represents whole numbers positive or negative, like 5, -5, 0.

```go
var positiveInt int = 5
var negativeInt int = -5
var zero int = 0
```

Signed integers, declared with int can store both positive and negative whole numbers. But there is a limit to how big of a number they can store.

| **Type** | **Size**                                       | **Range**                                   |
|----------|------------------------------------------------|---------------------------------------------|
| int      | 32 or 64 bits depending on 32 or 64 bit system | See the respective column                   |
| int8     | 8 bits                                         | -128 to 127                                 |
| int16    | 16 bits                                        | -32768 to 32767                             |
| int32    | 32 bits                                        | -2147483648 to 2147483647                   |
| int64    | 64 bits                                        | -9223372036854775808 to 9223372036854775807 |

> **Note:**
> There are also `Unsigned` types for integers.

#### Unsigned Integers

Unsigned integers are declared with the `uint` keyword, these can only store positive whole numbers (yes this includes zero).

Here are the different variations of `uint` and their limitations:

| **Type** | **Size**                                       | **Range**                 |
|----------|------------------------------------------------|---------------------------|
| uint     | 32 or 64 bits depending on 32 or 64 bit system | See the respective column |
| uint8    | 8 bits                                         | 0 to 255                  |
| uint16   | 16 bits                                        | 0 to 65535                |
| uint32   | 32 bits                                        | 0 to 4294967295           |
| uint64   | 64 bits                                        | 0 to 18446744073709551615 |

#### Float

The float data types can store positive or negative decimal numbers:

| **Type** | **Size**                                       | **Range**                 |
|----------|------------------------------------------------|---------------------------|
| float32  | 32 bits                                        | -3.4e+38 to 3.4e+38.      |
| float64  | 64 bits                                        | -1.7e+308 to +1.7e+308.   |


Here is an example of how we can use floats:

```go
package main
import ("fmt")

func main() {
    var x float64 = 234.43
    var y float32 = 3.4e+38
    fmt.Printf("Type: %T, value: %v\n", x, x)
    fmt.Printf("Type: %T, value: %v", y, y)
}
```

`fmt.Printf("text")` is another method from the "fmt" package it is used to format an insert text at some predefined placeholders. You can learn more about fmt [here](https://pkg.go.dev/fmt).

#### Boolean

A variable with `bool` data type can store either `true` or `false`. the default value for a bool variable is false.

```go
var isTrue bool = true
var isFalse bool = false
var defaultValue bool

fmt.Println("isTrue:", isTrue)
fmt.Println("isFalse:", isFalse)
fmt.Println("defaultValue:", defaultValue)
```

> **Note: **
> We'll talk about the arrays and maps separately.
