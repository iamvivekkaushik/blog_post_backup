## Array & Slice in GoLang

Let's say we are working on a go app and we have to store the data of 2 users, so far we know about variables and how they can be used for data, let's say we create two variables and store the name of our 2 users. But then we receive a request to add another user well, we can create another variable for our new user. But you can see this is becoming a problem. Our application is not scalable and the code will also start to become a mess after a certain number of users. How do we get around this problem?

Meet Arrays and Slices.

### Array

An array is also like a variable it has an identifier and a data type. But unlike a variable we can store data many different predefined numbers of elements. 

Let's say for 100 users, we create an array:

```go
var users = [100]string{}   // an empty string array of 100 elements
```

When creating an array we need to know its size beforehand (how many elements it can hold).

First, we're defining a variable `users` which holds the memory address for the array, after the `=` sign we're defining the size of the array and then the type of data this array will hold, and finally the `{}` is the initial value of the array, which is currently empty. 

You can also add initial values by providing them in the `{}` separated by a comma.

```go
var users = [100]string{"Vivek", "Harsh", "Kartik"}   // an string array with 3 initial value 
```

How about accessing and storing the data?

```go
var users = [50]string{"Vivek", "Harsh", "Kartik"}

users[3] = "Sachin" // storing the user at 3rd index

fmt.Println(users[0]) // this will print "Vivek"
fmt.Println(users[1]) // this will print "Harsh"

users[0] = "Vivek Kaushik" // Reassign the value at index 0
fmt.Println(users[1]) // Will print "Vivek Kaushik"

fmt.Printf("length of the array: %v\n", len(users))  // Will print "length of the array: 100"
```

Arrays have something called as Index. Think of the array as a box with compartments, and each compartment has a number assigned to it (index). The number starts from 0 to the length of the array minus 1. So for our previous user example, we will have index from 0 to 99. We can use these indexes to access, assign or reassign values.

If we try to access a value from an index that is out of range, from our previous example `fmt.Println(users[100])` this will give us an `Index out of bound` error as we only had index up to 99.

> Note: 
> `len()` is another function, that gives us the size of the array. 


### Slices

Arrays are great but they are not always useful, as most of the time we do not know how many elements we will be adding. This is where `Slices` comes in.

For slices we don't have to define size, they are dynamic and can grow and shrink as needed.


```go
var users = []string{} // empty slice of 0 length

fmt.Printf("length of the slice: %v\n", len(users))  // Will print "length of the slice: 0"

users = append(users, "Vivek")

fmt.Printf("length of the slice: %v\n", len(users))  // Will print "length of the slice: 1"

fmt.Println(users[0]) // this will print "Vivek"
```

First, we initialize an empty slice which is very similar to an array except we don't specify the size of it. Next we print the size of our newly defined slice using the `len` function which gives us zero. Then we use another function called `append` which takes our slice and a value and appends it to the end of slice. After using the `len` function again we see that the size is now 1. Finally we print the value at index  0.
