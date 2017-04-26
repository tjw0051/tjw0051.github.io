---
layout:     post
title:      "Putting projects where you want with Go"
subtitle:   Screw you gopher, and your workspace!
date:       2017-04-25 16:02:00
author:     "Tom"
header-img: "img/2017-04-25_cover.jpg"
header-mask: 0.5
catalog: true
tags:
    - Go
    - shell
    - Project Management
---

# Overview

Since picking up Go I've been a bit resistant to some of its ways, particularly the way it manages projects. As someone who obsessively organises their projects into nicely-defined directories, I was a bit taken-a-back by Go's unconventional workspace structure. I typically don't like to use the pre-defined workspace folders of IDEs like Visual Studio or Jetbrains products.

On a recent project I decided enough is enough, i'm taking back my filesystem and there's nothing Go can do to stop me! I wanted to see if I could symlink my project directory into Go's workspace path. And, it works! You can sym link your project's folder, e.g. `'Example/'` into your GOPATH src folder, e.g. `'$HOME/go/src/Example'`. 

If you know how to do this and like the idea, then great! Go forth and liberate your filesystem!

If not, I'm going to outline below how to set up a nice Go environment on your system that uses this method.

---

# Setting your Go Path

The GoPath is set to your home directory at $Home/go by default. For this example we're going to stick with this location, but you could move this to some deep dark hole where you never have to see it again. To see where the go path is set to, enter this into your terminal:

```
go env GOPATH
```

If it is set to your home directory, you should see:

```
/Users/[your-username]/go
```

If it is not set here, enter the following:

```
export GOPATH=$HOME/go
```

You can verify the gopath has been set correctly by again entering:

```
go env GOPATH
```

# Creating the Workspace

Next we create the following directory structure:
```
$HOME
- go
  - bin/
  - pkg/
  - src/
```

# Setting Up a New Project

We're going to set up a project called *MyProject* and save it to a projects folder in our home directory, or wherever you'd like to store your project.

```
mkdir ~/Projects
mkdir ~/Projects/MyProject
touch ~/Projects/MyProject/Main.go
```

We'll create a quick *Hello World* program to show that we can build our application. Open Main.go in `~/Projects/MyProject` and enter:

```go
package main

import (
  "log"
)

func main() {
  log.Printf("Hello World!")
}
```

Next we create a symbolic link to our projects folder in go's workspace:

```
ln -s ~/Projects/MyProject ~/go/src/MyProject
```

---

I hope some people find this useful. Its obviously not rocket science but it might not be obvious if you want to avoid Go's workspace.


—— Tom 2017.04
