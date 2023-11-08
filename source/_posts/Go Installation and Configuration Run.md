---
title: Go Installation and Configuration Run
date: 2023-11-06 11:04:00
categories: 
  - Go
tags: 
  - Backend Technology Sharing
  - environment
  - executable
  - recognize
  - development
  - variables
  - network
  - environment
description: You may have noticed that when we configure the Go language environment, we don't use the traditional installation package, but directly download the .zip file and unzip it to configure the environment variables, which is different from the .msi installation in that it doesn't write some configuration information into our computer's registry, and it can't generate shortcuts. The Go program is compiled into an executable .exe file.，此…
cover: https://raw.githubusercontent.com/zqwuming/blogimage/img/img/go.webp
---
### Go Installation and Configuration Run

> A must-see guide to configuring and running Go environment variables for newbies.

Go download link:[studygolang.com/dl](https://studygolang.com/dl)

We can go directly to the Go language Chinese website to find the latest version of the current stable operation (currently 2020-11-08 is 1.15.4), and then find a Windows system to download (as shown in the figure):

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/qB5Drm4uRx7JYhZ.webp)

> There are two options for downloading the zip archive or the msi installer. To save time, we'll download a zip archive, the first one in the red box, go.15.4.windows-386.zip.

Once the download is complete, unzip it to a directory on your computer, this is the directory I unzipped locally, and then go to the bin directory after unzipping the package:

![](https://s2.loli.net/2023/11/07/kbqYEshRpou75cJ.webp)

![](https://s2.loli.net/2023/11/07/UJ5TPBENpFvQcK3.webp)

Then right click on this computer (for Win7 etc. open My Computer) and go to the properties page:

![](https://s2.loli.net/2023/11/07/98NxDTUns4LzQ13.webp)

Open Advanced System Settings, Environment Variables, and edit the value of Path in System Variables:

![](https://s2.loli.net/2023/11/07/wJuDaH5BX8IvKqQ.webp)

Then add the address of the bin directory:

![](https://s2.loli.net/2023/11/07/5t3ANlVDG7kphQH.webp)

And finally, to check if we've configured it successfully, first, call out the command line (shortcut Win + R) and type cmd:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/12d346963d37400da534cd6b7bbbaadf%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

在任意目录下输入 go version：

![](https://s2.loli.net/2023/11/07/I3lRJ8qUB75OWus.webp)

> The Go version 1.15.4 we just downloaded appears, la la la la ~~ Congratulations, the configuration is successful!

### 运行 Go 程序

Now we can run our Go program directly from the command line.

First, from the command line, go to the Go file directory. For example, my current helloworld.go is in the C:\Users\37595\Desktop\KnowledgeBase\code\1_helloworld directory:

![](https://s2.loli.net/2023/11/07/2a84neb3OM5NlDA.webp)

1. ##### There are two ways to run it:

   1. go build compiles helloworld, and after successful compilation, a new .exe file is added to the directory:

![](https://s2.loli.net/2023/11/07/hwlHPYmL7fvuMa1.webp)

> Execute the compiled file:

![](https://s2.loli.net/2023/11/07/VoeOrpyJRgn6PuM.webp)

1. The go run command is executed directly:

![](https://s2.loli.net/2023/11/07/AI36lUkyOFNEfnR.webp)

* ### Summary

  * You may have noticed that we didn't use the traditional installer method to configure the Go environment, but instead downloaded the .zip file and unzipped it to configure the environment variables. The difference between this and the .msi installation is that it doesn't write any configuration information into our computer's registry, and it can't generate shortcuts. Therefore, if you are already familiar with the installation process, or want to save time, you can choose the .zip installation method directly, and you can also run the Go program normally!
  * After compiling the Go program, it will be an executable .exe file, which can still be run on computers that do not have the Go compilation package installed and configured, just like Java and other programming languages, which can be compiled once and run everywhere. The difference is that the .class bytecode file generated after Java compilation needs to be run on the JVM (Java Virtual Machine), while the .exe file compiled by Go can be opened and run directly on a Windows machine.
> PS：Some operating system computers will flash back after opening the .exe file directly, in this case, you can add a line at the end of the code to get the input parameter code from the keyboard on it:

```arduino
 fmt.Scanf("a")
```
