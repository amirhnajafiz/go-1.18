# Go 1.18
<img src="./assets/logo.avif" />

New features of Golang 1.18 with tutorial and examples.
In this repository I introduce new features of Golang version 1.18 with their examples and resources.

## Contents
- Introduction
    - Generics
    - Fuzzing
    - Workspace
    - Performance Improvements
- Examples
    - Implement **Stack** with Generics
    - Implement **Linkedlist** with Generics
    - Implement **BinaryTree** with Generics
- Resources
- Examples and Tutorials

## Introduction
The latest Go release, version 1.18, is a significant release, including changes to the language, implementation of the toolchain, runtime, and libraries. Go 1.18 arrives seven months after Go 1.17. As always, the release maintains the Go 1 promise of compatibility. We expect almost all Go programs to continue to compile and run as before.

### Getting started with generics
To support values of either type, that single function will need a way to declare what types it supports. To support this, you’ll write a function that declares type parameters in addition to its ordinary function parameters. These type parameters make the function generic, enabling it to work with arguments of different types. You’ll call the function with type arguments and ordinary function arguments. While a type parameter’s constraint typically represents a set of types, at compile time the type parameter stands for a single type – the type provided as a type argument by the calling code. If the type argument’s type isn’t allowed by the type parameter’s constraint, the code won’t compile.

Example:
```go
// SumIntsOrFloats sums the values of map m. It supports both int64 and float64
// as types for map values.
func SumIntsOrFloats[K comparable, V int64 | float64](m map[K]V) V {
    var s V
    for _, v := range m {
        s += v
    }
    return s
}
```

```go
SumIntsOrFloats[string, int64](ints)
SumIntsOrFloats[string, float64](floats))
```

You can omit type arguments in calling code when the Go compiler can infer the types you want to use. The compiler infers type arguments from the types of function arguments.

Note that this isn’t always possible. For example, if you needed to call a generic function that had no arguments, you would need to include the type arguments in the function call.
```go
SumIntsOrFloats(ints)
SumIntsOrFloats(floats))
```

#### Declare a type constraint
You declare a type constraint as an interface. The constraint allows any type implementing the interface. For example, if you declare a type constraint interface with three methods, then use it with a type parameter in a generic function, type arguments used to call the function must have all of those methods.
```go
type Number interface {
    int64 | float64
}

func SumNumbers[K comparable, V Number](m map[K]V) V {
    var s V
    for _, v := range m {
        s += v
    }
    return s
}
```

### Getting started with fuzzing
The unit test has limitations, namely that each input must be added to the test by the developer. One benefit of fuzzing is that it comes up with inputs for your code, and may identify edge cases that the test cases you came up with didn’t reach.

Example:
```go
func FuzzReverse(f *testing.F) {
    testcases := []string{"Hello, world", " ", "!12345"}
    for _, tc := range testcases {
        f.Add(tc)  // Use f.Add to provide a seed corpus
    }
    f.Fuzz(func(t *testing.T, orig string) {
        rev := Reverse(orig)
        doubleRev := Reverse(rev)
        if orig != doubleRev {
            t.Errorf("Before: %q, after: %q", orig, doubleRev)
        }
        if utf8.ValidString(orig) && !utf8.ValidString(rev) {
            t.Errorf("Reverse produced invalid UTF-8 string %q", rev)
        }
    })
}
```

When fuzzing, you can’t predict the expected output, since you don’t have control over the inputs.<br />
Note the syntax differences between the unit test and the fuzz test:
- The function begins with FuzzXxx instead of TestXxx, and takes \*testing.F instead of \*testing.T
- Where you would expect to a see a t.Run execution, you instead see f.Fuzz which takes a fuzz target function whose parameters are \*testing.T and the types to be fuzzed. The inputs from your unit test are provided as seed corpus inputs using f.Add.

### Getting started with multi-module workspaces
A workspace is a collection of modules on disk that are used as the root modules when running minimal version selection (MVS).

A workspace can be declared in a go.work file that specifies relative paths to the module directories of each of the modules in the workspace. When no go.work file exists, the workspace consists of the single module containing the current directory.

Most go subcommands that work with modules operate on the set of modules determined by the current workspace. go mod init, go mod why, go mod edit, go mod tidy, go mod vendor, and go get always operate on a single main module.

With multi-module workspaces, you can tell the Go command that you’re writing code in multiple modules at the same time and easily build and run code in those modules.

Initialize the workspace:
```shell
go work init [Directory name]
```
The go.work file has similar syntax to **go.mod**, the go directive tells Go which version of Go the file should be interpreted with. It’s similar to the go directive in the go.mod file. The use directive tells Go that the module in the given directory should be main modules when doing a build.

The go work init command tells go to create a go.work file for a workspace containing the modules in the given directory.<br />
The go command produces a go.work file that looks like this:
```go
go 1.18

use ./hello
```

The go command has a couple of subcommands for working with workspaces in addition to go work init:
- **go work use [-r] [dir]** adds a use directive to the go.work file for dir, if it exists, and removes the use directory if the argument directory doesn’t exist. The -r flag examines subdirectories of dir recursively.
- **go work edit** edits the go.work file similarly to go mod edit
- **go work sync** syncs dependencies from the workspace’s build list into each of the workspace modules.

### 20% Performance Improvements
Apple M1, ARM64, and PowerPC64 users rejoice! Go 1.18 includes CPU performance improvements of up to 20% due to the expansion of Go 1.17’s register ABI calling convention to these architectures. Just to underscore how big this release is, a 20% performance improvement is the fourth most important headline!

## Examples
In this section I implemented some data structures with Golang generics.

### Stack
Defining the data types:
```go
type Data interface {
	int64 | float64 | string
}

type Node[T Data] struct {
	Next  *Node[T]
	Value T
}

type Stack[T Data] struct {
	Head *Node[T]
}
```

Now we can create our stack in any type we want:
```go
    s := Stack[int64]{}
	s.Push(12)
	s.Push(129)
	s.Push(160)
```

## Resources
- [Beta installation of Go 1.18](https://go.dev/blog/go1.18beta2)
- [Whats new in Go 1.18](https://go.dev/blog/go1.18)
- [All new in Go 1.18](https://tip.golang.org/doc/go1.18)
- [Go workspace](https://go.dev/ref/mod#workspaces)
- [Go 1.18 overview](https://www.youtube.com/watch?v=-wpISpghaB8)

## Examples and Tutorials
- [Go generics tutorial](https://go.dev/doc/tutorial/generics)
- [Go fuzzings tutorial](https://go.dev/doc/tutorial/fuzz)
- [Go workspace tutorial](https://go.dev/doc/tutorial/workspaces)
- [Go examples for generics](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwjagPy1is_2AhXP8rsIHUWABXQQFnoECAgQAQ&url=https%3A%2F%2Fbignerdranch.com%2Fblog%2Fexploring-go-v1-18s-generics%2F&usg=AOvVaw0p24Y94Q3VshO1kUaKY_p7)
- [Type parameters in Go 1.18](https://www.youtube.com/watch?v=Rvq__lVVmQc)
- [First look at Go generics](https://www.youtube.com/watch?v=lw4X6takiRA)
