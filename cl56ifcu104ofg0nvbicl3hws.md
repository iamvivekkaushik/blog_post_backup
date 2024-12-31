---
title: "Getting Started With Go"
seoTitle: "Getting Started With GoLang"
seoDescription: "In this post, we will set up the project and familiarize ourselves with some basic go commands."
datePublished: Mon Jul 04 2022 08:58:24 GMT+0000 (Coordinated Universal Time)
cuid: cl56ifcu104ofg0nvbicl3hws
slug: getting-started-with-go
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1656870470158/TILbiKESj.png
tags: go, golang, learning

---

In this post, I'll walk you through setting up your go project. As a prerequisite you must have go installed in your system, you can get it for your system from [https://go.dev/dl/](https://go.dev/dl/)


Now that that's out of the way, go ahead and create a directory called `my_project`, Inside the directory you need a `go.mod` at the root of the project, this file that keeps track of the modules that provide those package (this of it as package.json file if you're coming from Java Script). You can create the go.mod file using `go mod init module_name` command, you can use your own module name, typical practice is to use your version control repo path for ex `github.com/iamvivekkaushik/my_project`. That is if you want to publish your module for others, the module path must be a location from which Go tools can download your module.



Now create a `main.go` file that will contain our go code and paste the following code.

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
``` 


To run your program use:

```
$ go run main.app
Hello World!
```

When developing an executable program, we use the package `main`. The package “main” tells the Go compiler that the package should compile as an executable program instead of a shared library. 

We will separate our source code into separate packages, this is to enable re-usablity of the code. The naming convention for Go package is to use the name of the system directory where we're putting the source file, the package will be same for each file within a directory.

After defining the package we use the import statement to import a package called `fmt`, probably short for formatter. The package `fmt` comes from the Go standard library. When we import packages, the Go compiler will look on the locations specified by the environment variable `GOROOT` and `GOPATH`.

Then we define a `main` function using the func keyword and use the "fmt" to print the "Hello World!" text.

If you're coming from JS you might be wondering I never called the main function (we will learn about functions in the coming posts) how is it running. So the main function is actually automatically called when you run the program, this is called the entrypoint of your project. You must have a main function defined in your main package.

---

Here are some commands to help you get started.

To add  missing module requirements or remove unneeded requirements, use:

```
go mod tidy
```

To add a third party package:

```
go get <module_url>
```

