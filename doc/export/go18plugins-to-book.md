% Go 1.8 Release Party
% New feature: Plugins
% By: Daniel Toebe

# What is the Plugin feature?

## The Plugin feature creates .so files

- The new `Plugin` feature is a way to create .so (shared object) file for dynamically linking libs in Go

## Ok.. What is a shared object file?

- A .so file is a precompiled binary that allows you to call functions inside it dynamically at your executable's runtime

## What does that really mean?

- Now instead of only having `go get` to download a package source on your build machine and compiling that package in your executable
    - You now can compile your package as a plugin binary distribute it separately and call it in your executable, and it will be loaded at runtime.

# That sounds neat, but what is the benefit?

## First

- Your executable may be smaller when you distribute it. 

## Second

- You can version the .so file. Making sure your executable calls the right version which can prevent runtime errors
    - Now you can create a `plugin.so.0.1` instead of a regular import and have many executables depend on that. What is you have added more features? Instead of updating all the dependent executables, you can just create a new `plugin.so.0.2`, call that in all the new executables you create and v0.1, and v0.2 can live side by side in your file system.

## Finally

- Finally your .so files don't need to live in your `$GOPATH` at all! It can live anywhere in your file system and you can dynamically link in your executable.

## What do I mean by dynamically linking?

- Basically you can place your .so file where ever you want, but if you have to make sure that it is places in the same place in the file system from server to server.
    - Examples: `/opt/libs` or something like `/usr/libs`
    - Or you can just have an environment variable created when you distribute your .so file and use that in calling your shared object in your executable using `os.Getenv()`
    - Another option is passing the path to the .so file through os.Args, or as a flag

# How do we create a shared object file?

## Fist we need to have the prerequisite import

```Go
import "C"
```
While we are not calling any C functions directly this is needed on all Plugins

## Now we write our functions

```Go
// plugin.go
package main

import "C"

func Greet() string {
    return "Hello Phx Gophers"
}
```

## Now we compile it

```Shell
$ go build --buildmode=plugin plugin.go
```
- This produces a `plugin.so` file
- Note the use of `--buildmode=plugin`
    - The buildmode flag was introduced in Go 1.5, but the plugin option was introduced in Go 1.8
    - [buildmode description](https://golang.org/cmd/go/#hdr-Description_of_build_modes)

## if you want to see what the file looks like...

```Shell
$ file plugin.so
```
The output should look like:
```Shell
plugin.so: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), 
dynamically linked, BuildID[sha1]=fc2ec03a4fca7e2b78aa1cb8bd82a9e1c99a0554, 
not stripped
```
Here you can see it is an ELF type binary used by Linux, and importantly not stripped. The not stripped means it keeps all symbols in your .so file.  To dive a little deep this is how your executable can call the functions in the .so file, by referencing the symbols.

# Now lets write our executable

## Fist we need to import the plugin package from the 1.8 standard lib

```Go
import "plugin"
```
This is needed with any .go file that will call a .so file

## Now for the whole file

```Go
package main

import (
    "fmt"
    "plugin"
)

func main() {
    p, err := plugin.Open("plugin.so")
    if err != nil {//Do something}
    g, err := p.Lookup("Greet")
    if err != nil {//Do something}
    greet := g.(func() string)
    fmt.Println(greet())
}
```

## Notice the p.Lookup()

- "Greet" is the name of the function we created in `plugin.so` file, and called with a string because that is the name of the symbol. An error returned on the function is because since the .so file is loaded at runtime there is no guarantee that the symbol exists

## Notice the assertion when creating greet

- Since the .so file is loaded at runtime the compiler can't know what type you are trying to use

## The output

We run...
```Shell 
$ go run main.go
```

... and we get

```Shell
Hello Phx Gophers
```

## Thats the basics

# That sounds awesome but...

## Some current limitations

- Right now you can only use .so files compiled from the go compiler. Meaning you can’t currently use .so files created with C
- You can only create and use the plugin system on Linux, so not on OSX or Windows. This is because the output of the `buildmode=plugin` creates an ELF binary and not compatible on OSX or Windows.
- Your executable must be compiled with the same version toolchain as the .so file. Right now that isn't an issue since all of this is in the 1.8 release, but may be an issue when 1.8.x is released
