---
title: Let's Go
date: 2023-11-06 23:04:00
categories: 
  - Technology
tags: 
  - Backend Technology Sharing
  - Crawler
  - absolutely
  - recognize
  - development
  - framework
  - network
  - selenium
description: In September 2007, Rob Pike, 61, and Ken Thompson, 74, were in Google Labs. Pike, 61, Ken Thompson, 74, and Robert Griesemer, 64. Rob Pike (61), Ken Thompson (74) and Robert Griesemer (64) got together at Google Labs. The three engineers got together and gave me the simple name of Go. They were father and mother, and they had a son, so I was always the "set" of the family.
cover: https://raw.githubusercontent.com/zqwuming/blogimage/img/img/2942348d26214e2587eda93eba3311c6%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp
---
### The past and present of the Go language

> Hello, everyone, thank you for coming, I am the main character today - Go!

##### The Birth of Go

In September, 2007, in Google Labs, 61-year-old Rob Pike, 74-year-old Ken Thompson, and 74-year-old Rob Pike were working on the Go language. Rob Pike (Rob Pike), 61, Ken Thompson (Ken Thompson), 74, and 64-year-old Rob Thompson (Ken Thompson) in Google Labs. Ken Thompson, 74, and Robert Griesemer, 64. Ken Thompson, 74, and Robert Griesemer, 64, three engineers who got together and gave me the simple name of Go.

As they were both parents and had a son at an older age, I was always "loved by all". And because all three of them are very powerful, there is Ken, the father of B, C, and Unix, who won the Turing Award and the U.S. National Technology Award, Rob, the creator of UTF-8 character encoding, and the main contributor to Google V8 and HotSpot JVM. As a result, I'm also known to many as "Bull II".

November 10, 2009 is my birthday, my previous works were hosted in Google's codehub, and then more people used it, so the code was slowly migrated to GitHub. It's also slowly catching on in China, so let's give you a look at the scope of business I'm involved in:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/2023-11-07_223323.png)

> In addition to large-scale projects such as Docker containers and kubernetes (known in the industry as k8s), which are implemented entirely in Go, more and more major Internet vendors are converting to Go. Today, the Go language has achieved a crucial position in the field of cloud computing and in the development of microservices for many large projects.

##### Why Go language is popular all over the world

* First, I'm a compiled language; I can execute the same program 30% to 70% faster than an interpreted language like Java;
* Secondly, as the C language of the 21st century, my development speed is not far behind, and my concise and efficient syntax leaves C and C++ developers in the dust;
* Concurrency support is friendly, and the advantage of being able to achieve concurrency in one line of code will make me more and more popular in high concurrency scenarios;
* Finally, the arrival of the cloud computing era and the massive demand for Internet servers will allow me to make a big splash in the future.

> Don't ask what the big picture is, ask is to surpass Java, python and other languages, and be among the top of the popularity.

##### Go, object-oriented?

Many of my partners who are new to Go are puzzled by the fact that I support encapsulation, but not inheritance or polymorphism, so strictly speaking I am not an object-oriented language. But I support interfaces to apply any kind of data type, and allow object-oriented programming style, from this level I am also a kind of object-oriented language.

So, am I an object-oriented language or not? Let's take a look at how the official documentation defines me:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/342e1feb277f421c8e166978c0cb8675%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

> Answer: Yes and no. On the one hand Go has an object-oriented programming style, on the other hand it doesn't have object inheritance, so Go is both an object-oriented language and not quite.
And it's the lack of object inheritance that makes Go lighter and simpler than C++/Java. Think about base classes, derived classes, single inheritance, multiple inheritance, permission control, etc.

### Quick start

#### 1. hello, world First Go program

Whether you're following me into the world of the Go language based on a job requirement, a desire to learn a highly promising development language, or your first foray into computers and programming.

After hearing so many stories about the king's wife, I'm sure you're familiar with this pushy routine. So, without further ado. Life is short, Let's Go !

> The first program in the life of the vast majority of computer professionals:

```go

package main
import "fmt"
func main() {
    fmt.Println("hello world!")
}
```

As in many languages, the main function is the first function to be executed, but the main package is special in Go and is used to define a separate executable program. Simply put, the main package must be introduced wherever there is a main function and you need to run that main function:

```go
package main
```

The fmt package contains a number of standard output and input functions, such as the print function Print, Println in the program means that immediately after printing a new line.

> Don't know how to run a Go program yet? Check out this article on installing and configuring Go to run

The Go language provides two common ways to run programs:* Build a .exe file with the go build command, and then run

> go build hello.go This creates an .exe compiled file in the directory, which can be opened on a Windows machine, or executed from the command line or git bash.
> /hello.exe /hello.exe

* Run the file directly with the go run command

> go run hello.go

The result

> hello world!

#### 2. Find duplicate lines in a list of strings.

> This program will teach you about for loops in Go, and the definition and use of string slices (slices: understood as variable length arrays):

```go
package main
import "fmt"
func main() {
    arr := []string{"qq", "wechat", "dnf", "lol", "wechat"}
    counts := make(map[string]int)
    for i := 0; i < len(arr); i++ {
    	counts[arr[i]]++
	}
    for key, value := range counts {
    if n > 1 {
    	fmt.Printf("值为 %s 的字符串重复了%d次 \n", key, value)
    	}
    }
}
```

Define a string slice: arr := []string{}, with the variable name on the left, and := is a common assignment statement in Go, e.g.: i := 0, which is the equivalent of initializing the i variable before assigning it:

```css
 var i int i = 0
```

In my Go world, defining a map is usually done by initializing the map with make to prevent null pointer exceptions. Or you can define a map with initialized key-value pair elements when you create it:

```go
 counts := map[string]int {
     "qq" : 1,
     "wechat" : 2,
 }
```

> map [string] int, indicating a single string string as the key and the number of strings int type as the value.

Then, count the number of each string in the list of strings, here we have used the self-incrementing counts[arr[i]]++ to count the number of strings, equivalent:

```css
 counts[arr[i]] = counts[arr[i]] + 1
```

> The advantage of this self-incrementation is that it eliminates the need for recurring variables and simplifies the code.

Finally, we iterate to print out the strings that have a number of occurrences greater than 1.

Run the result:

> String with value wechat repeated 2 times

for is the only way to loop inside go, and there are two common forms:

```go
 for init; condition; post {

 }
```

> The for loop is formed without parentheses, but the curly braces after the for statement are required, and the left curly brace must be on the same line as the post statement.

```go
 for i, v := range array {

 }
```

> for range is used to quickly traverse data structures (arrays, slices, maps, etc.) with multiple elements, i is the index of the element (ranging from 0 to len(array)-1), and v is the value of the element.
That is, v == array[i]

### Summary

This article introduces the history of the Go language and its general application scenarios. It's worth mentioning that distributed development and microservices architecture are still at the cutting edge, and many programmers are still in traditional companies and small computer enterprises, and can't get in touch with new technologies and projects.

So I wrote Go this series, but also want to let more want to learn but no way and direction of the computer direction of the school personnel, or has embarked on the work of the practitioners can really understand the charm of Go. In many large Internet companies, such as Microsoft, Byte, Tencent, Kingsoft, Xunlei, including Huawei such as communications companies, in many business teams have begun to Go transition, Go will also become the hottest development language in the era of cloud computing!

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/2942348d26214e2587eda93eba3311c6%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

While the dawn is still uncertain, while the moon is still hanging in the sky, take your backpack and set out with me!
