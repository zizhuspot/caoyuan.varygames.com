---
title: Swaggo Automated API Documentation (Developer Utility)
date: 2023-11-07 04:04:00
categories: 
  - Technology
tags: 
  - Backend Technology Sharing
  - Swagger
  - Go
  - implemen
  - development
  - framework
  - Automated
  - Documentation
description: Swaggo is the Go language automatically generates swagger document tool ,developers can use it to easily implement 0 document programming 
cover: https://raw.githubusercontent.com/zqwuming/blogimage/img/img/7eb75ca6cd3796.jpeg
---
* ## Table of Contents

  1. Introduction
  2. Introduction to Swagger
  3. Building a Swagger Project
  4. Introducing Swagger UI for rendering document pages
  5. The flag library controls whether or not the UI page is rendered.
  6. Summary

  ## 1. Introduction

  As a back-end developer in the Internet, the daily 996,007 development work is not the most difficult, the most difficult work is often day after day to hold some boring and inefficient meetings, as well as endless pull-through alignment.

  When each product demand comes, in addition to the coding work is relatively easy, other things such as system design, program review, development coordination and upstream and downstream communication (sib, shuaiguo) work is very time-consuming and laborious.

  If there is one thing that is indispensable in this chain of workflow - it is probably documentation.

  Documents archived for each version of a product include, but are not limited to: product description documents, **system design documents, detailed design documents, interface documents, test documents** and operation and maintenance documents.

  Generally speaking, programmers contact documents at least design documents and interface documents, in some high-performance or high reliability requirements of the business, in addition to the testers, may also need to developers in the test before the output of some of the business of the self-testing documents: including performance testing, stress test documents and so on.

  Thinking about it makes one's head spin!

  Then, as programmers, how do we solve this series of processes under the bombardment of documents?

  * If it is a design document, you may need a variety of UML diagrams to assist, with as few words as possible to express the advantages of clear design;
  * If it is a test document, you can combine the test background, test tools and data to show the effectiveness of the test, for example, what changes have been made to the interface, the response has been reduced from 300ms to less than 200ms, and the performance improvement rate of 50%;
  * If it is an interface document, then I'm going to introduce this swag tool, or can help you.

## 2. swagger introduction

## 2.1 Background

Years later, facing the code bouncing on the screen, the back-end developer Xiao_❤️ will recall the distant night when he first wrote the interface document. On that day, little ❤️ had just finished writing the code, looking at the evening sun in the sky, thinking that everything was fine and he could get off work soon. Front-end developers small A came over, "hello old brother, may I ask how is the interface development, tomorrow can be linked up?"

"Emmm although this piece of work is more difficult, but I have been working day and night these days to catch up with the work, tomorrow should be able to interconnect!"

As soon as Little A heard this, his eyes began to shine, thinking that he could get off work earlier once the intermodulation was over in a couple days. So he started to say excitedly, "Give me an interface document quickly, I'll add them to the page tonight".

At this time, the development manager also came over, "interface documents or very important, involving the consistency and stability of the front and back end interfaces, the need to write and maintain, small ❤️ son you add off work, write the document tonight, it does not take long, estimated half an hour to get it done! In the process of writing to pay attention to the systematization of thinking, think about what you do this thing where the value point? What you do, and the company's differentiation of other teams in the where? What you do, does it precipitate a set of reusable physical information and methodology? Why are you the one doing it and no one else can? Sink your thinking into a daily, weekly, and monthly report that reflects thinking about the interface documentation thing, not just progress. Okay, get busy!"

The leader has spoken, how can we say no? Even though 10,000 grass horses are running in my heart, I can only nod my head with a smile and say yes.

As a result, each unimpressive interface needs to be defined by in-parameters, out-parameters, and examples, so that it can be linked with the front-end and subsequently archived. After much deliberation over the format of the documentation, a final version was created. The documentation for a particular interface looks like this:

> _Interface description: add xxxx business_.
> _Interface address: /api/v1/resourcexx_.
> _Request method: POST_

_Request header parameters _
_Field name_ _Type_ _Required_ _Description_ _Content-Type_ _string_ _yes_ _application/json;charset=UTF-8_

_Request body parameters: _
_Field Name_ _Type_ _Required_ _Description_ _app_id_ _string_ _Yes_ _application_id_ _source_type_ _integer_ _Yes_ _data_source_, 0: local, 1: data-center_ _data_ids_ _string_ _No_ _data_set_id_ _No

............

_Response parameters: _
_field name_ _type_ _description_ _code_ _integer_ _response code, 200 for return success, 400 pass parameter error, 500 internal error_ _msg_ _string_ _error message, when the response code is 200, the value is "ok" _ _data_ _json_ _response body, according to the different business to decide the _

_Request example:_

```css
curl --location --request POST 'http://127.0.0.1:5187/nlp/addMarketBoard' \
--header 'Content-Type: application/json' \
--form 'file=@".../测试小组挂断率.xlsx"' \
--form 'file_name="002"' \
--form 'component_type="0"' \
--form 'app_id="1056"' \
--form 'board_name="minio测试"' \
--form 'source_type="0"' \
--form 'data_ids="67"'
```

Some people can not help but ask: this is something that should not take too much time, right ~ Indeed, as our leader said, an interface does not look too much text, are some fixed format, copy and paste the work only.

However, the end of development in addition to my own, there are dozens of interfaces developed together with colleagues ah ...... The leader's idea is that I'll do it together with the things that I do by hand anyway.

There is no CPU I'm not too sure, but when the last interface added to the document, I saw the bright moon in the night sky, and then fell into meditation: I do not know whether it was to stay in love with the beauty of the evening sun, or pondering Wang Jiefu "when the moon shines on me back" state of mind.

Anyway, I don't want to do this job of writing interface documents for the second time in my life. So, before it was 11 o'clock (lights out at 11 o'clock in the company), I started to search for tools that can automatically generate interface documents, and I came across it - swagger.

### 2.2 What is swagger?

Swagger is the development tool framework for the world's largest OpenAPI specification (OAS), and one of the world's most popular restful API documentation generation tools, its advantages are:

* support for cross-platform , cross-language
* community open source, and very active
* has a very complete ecosystem (Swagger Editor, Swagger Codegen, Swagger UI ...)

> restful API: a kind of industry common routing specification design, it is the style of the Internet all the data are regarded as resources, the request URL naming is to locate the resource, the specific operation of the resource by the request method [GET/POST/DELETE/PUT] to decide. For example:
> GET [http://localhost:8080/api/v1/resource](https://link.juejin.cn?target=http%3A%2F%2Flocalhost%3A8080%2Fapi%2Fv1%2Fresource "http ://localhost:8080/api/v1/resource") // Get a certain resource on a certain local server.

### 2.3 Introduction to the swagger tool

This time, using Go as an example, we use swaggo as the tool to automatically generate the swagger API documentation and gin-swagger as the rendering package implementation of the Swagger UI.

The three toolkits to be introduced are as follows:

> go get -u github.com/swaggo/swag/cmd/swag or go install github.com/swaggo/swag/cmd/swag@latest (after go1.70)
> go get github.com/swaggo/gin-swagger
> go get github.com/swaggo/gin-swagger/swaggerFiles

More on when to use them next.

## 3. Building the Swagger Sample Project

## 3.1 Build a new project

Operating environment:

1. Windows 10 (OS optional)
2. Goland
3. Git
4. Go 1.17.11 (optional)

The whole project operation is realized in Goland, we default readers have installed the Goland/Go SDK, and downloaded the Git tool to execute the command operation.

#### 3.1.1 Creating a New Project and Introducing gin-swagger

First, let's create a new project in Goland and name it swagger-test:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/7c830f0d5b3742db933ad5b872ce9937%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

Then, go ahead and introduce the go mod into the project to manage the dependencies we'll be downloading later:

> go mod init swagger
go mod tidy

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/9644225884ec4b3aba69ab0df155cf90%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

Next, introduce the gin-swagger middleware and the built-in files for swagger file management into the project

> go get github.com/swaggo/gin-swagger
go get github.com/swaggo/gin-swagger/swaggerFiles

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/72580aa4396945eb83f1f1d7643b2853%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

#### 3.1.2 Writing Server Listening Code and Comments

Since gin-swagger has been introduced for this purpose, we will use the gin framework to handle browser requests in the following.

First, create a new api package and add the Hello method and annotations, which are the interface methods we will use later:

```go
package api

import (
    "net/http"

    "github.com/gin-gonic/gin"
)

type Response struct {
    Code    uint32      `json:"code"`
    Message string      `json:"message"`
    Data    interface{} `json:"data"`
}

func Hello(c *gin.Context) {
    res := Response{Code: 1001, Message: "first interface, hello", Data: "connect success!"}
    c.JSON(http.StatusOK, res)
}
```

Among them, the interface annotations in lines 11~25 are manually added according to the definition of swagger, the swaggo tool automatically generates API documentation, these annotations will be used, the basic definition of the annotations are:

* @Summary, interface summary
* @Description, the interface description
* @Tags, the interface tags, used to group the API
* @Accept, the interface accepts the type of input parameter, support mpfd (form), json, etc.
* @Produce, the type of output parameter returned by the interface, supports mpfd (form), json, etc. * @Param, the type of input parameter.
* @Param, the definition of the input parameter, from front to back are:

> As shown in the code @Param user_id query string true "User ID" minlength(1) maxlength(100), @Param format is:
> 1. parameter name 2. parameter type 3. data type 4. whether field is required 5. parameter description 6. other attributes

Path and response comments about the interface have:

* @Success, specify the data of the success response, in the format of 1.HTTP response code 2.response parameter type 3.response data type 4.other description
* @Failure, specifies the data after a failure response, same as Success

* @Router, specifying the route and HTTP method.

More fields can be found in the Swaggo documentation: [[gitcode.net/mirrors/swa...](https:// gitcode.net/mirrors/swaggo/swag/-/blob/master/README_zh-cn.md)] 

Next, create a new main.go in the project's home directory, specifying the api/v1/hello interface to point to the Hello method in the api package:

```go
package main

import (
    "swagger/api"

    "github.com/gin-gonic/gin"
    swaggerFiles "github.com/swaggo/files"
    ginSwagger "github.com/swaggo/gin-swagger"
)

func main() {
    r := gin.Default()
    r.GET("/swagger/*any", ginSwagger.WrapHandler(swaggerFiles.Handler))

    v1 := r.Group("/api/v1")
    {
       v1.GET("/hello", api.Hello)
   }
    r.Run(":8080")
}
```

Since we are going to generate the swagger UI page through the main method, we need to add comments to the page definition above the main method, common fields are:

* title, the title of the swagger UI.
* version, the version of the interface
* description, the description of the swagger document
* host, the host address of the record
* BasePath, the base path, which will be displayed in the swagger UI with host automatically spliced in.

After defining the swagger UI page with the above comments, we need to specify a route to access it:

```css
r := gin.Default()
r.GET("/swagger/*any", ginSwagger.WrapHandler(swaggerFiles.Handler))
```

gin.Default() defines an initialization engine for gin, and /swagger/*any serves as the access route. After generating the swagger file and starting the service, you can access the swagger UI page in a browser via 127.0.0.1:8080/swagger/index.html, with /swagger being the path prefix.

Next, we'll add a new business access interface. We'll add a new route group /api/vi as a prefix, and under this route group we'll add a /hello interface, which will point to the Hello method under the api package. Finally, we specify an 8080 port to listen on.

### 3.2 Generating API Documentation with the swaggo Tool

#### 3.2.1 Installing the swaggo tool

Once you have defined all of this, you can install the swaggo tool to automatically generate the swagger files. There are many ways to do this, but we'll use the Git method here.

We need to download and install git on our computer first, here we have git installed by default and configure the environment, then install swaggo:

(2.0) Linux installation:

> go get -u github.com/swaggo/swag/cmd/swag (before go1.70)
> go install github.com/swaggo/swag/cmd/swag@latest (after go1.70)

If the installation is unsuccessful in Linux environment, it may be caused by the inconsistency between GOPATH path and GOROOT, you can use `vi /etc/profile` command to modify it:

> export GOROOT=/usr/local/go ##GoLang installation directory.
> export PATH=$GOROOT/bin:$PATH
> export GOPATH=/usr/local/go ##GoLang project directory

Then refresh GOPATH: `source /etc/profile`

2.1）windows Installation method (this method is selected by default below)：

> go get -u github.com/swaggo/swag/cmd/swag

2.2）macOS Mounting method:

> mv $GOPATH/bin/swag /usr/local/go/bin

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/95b4dfe1196444bf951dfa5b4748219f%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

Test if the installation was successful:

> swag -v

#### 3.2.2 Generating API Documentation with swaggo

After the installation of swag is complete, we go to the home directory of the swagger-test project [that is, the root directory where the project was created, in the example, D:\runSpace\swagger-test], and use the command to generate the swagger file:

> swag init

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/50cb87d2a7b34db0ba52bad1c14600f1%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

At this point, we can see that the project has automatically generated a docs directory with the newly generated swagger files:

> ./docs
├── docs.go
├── swagger.json
└── swagger.yaml

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/ed9544c3f6dd4e4b8cc7e817076a46ad%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

#### 3.2.3 Importing API Documentation from the docs Package

Next, we add the path to the just-generated docs package in the main.go file, and import this path so that the API documentation can be accessed after the service is started:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/c452779fe4e14a529d1bd5cf22a3a17d%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

Then, we start the project with go run main.go:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/dc37555102d049bebbc58989cfbe10d7%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

You can see that port 8080 is up.

## 4. Swagger Document Rendering Demonstration and Testing

### 4.1 Swagger UI Page Access

#### 4.1.1 Server Connectivity Testing

First, we access the hello interface in the browser:

> http://[127.0.0.1:8080/api/v1/hello](https://link.juejin.cn?target=http%3A%2F%2F127.0.0.1%3A8080%2Fapi%2Fv1%2Fhello "http://127.0.0.1:8080/api/v1/hello")

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/f6f6f5161b784174976180d4ce8d5480%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

Successful access indicates that the port is up.

#### 4.1.2 Swagger UI Page Access

Next, we open the swagger UI page to view the API documentation for the interface.

> Path：http://[127.0.0.1:8080/](https://link.juejin.cn?target=http%3A%2F%2F127.0.0.1%3A8080%2Fapi%2Fv1%2Fhello "http://127.0.0.1:8080/api/v1/hello")swagger/index.html

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/297e43d85eff45f0b36ed87247a5c1af%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

As you can see, the swagger UI page is now accessible. It contains a basic description of swagger and API interface information, and you can also view the doc.json interface definition online:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/73ae485e727849ad9335760ffd7c4a8b%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

Here, we can see the power of swagger, just through a few comments and route definitions, you can write what we need to define in the interface document.

Moreover, the swagger API documentation is universal, almost all front-end, back-end and testing staff can understand. If there is a modification of the interface in the development process, we only need to change the comments, and then use the swag init command to update the docs, and then restart the service. Then we can dump the updated docs [http://ip+port/swagger/index.html](http://ip+port/swagger/index.html) to the front-end or testers.

### 4.2 API Testing

In addition to API documentation, swagger UI also provides interface testing, we can see a Try it out button on the right side of the /hello interface "Parameters" on the homepage:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/166f8aa35e7b466489d6157f622ea3e8%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

点击之后可以输入测试用例，输入参数后点击 Execute 执行：

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/317d9c44a47e45e791085cd2a577d5d5%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

The execution steps are:

1. Click Try it out button to start the test. 2.
2. Input the interface input parameter, there is only one input parameter in the example, and it is not business related, so you can fill in the parameter as you like.
3. Click the Execute button to execute the test.
4. Swagger UI generates a curl command to call the server-side interface. 5.
5. Return the response result: success/fail.

See here, you must have understood the basic usage of swaggo tool. However, the API documentation is generally only needed when we do testing in the development environment, after the service is online, most of the interface documentation is to be invisible to the user. Moreover, updating the swagger documentation every time a service goes live will also affect the speed of our service compilation.

Therefore, we can use the flag library's command parsing to control whether or not to hide the swagger UI interfaces at runtime.

## 5. The flag library controls whether to render the Swagger UI.

### 5.1 Adding flag parameter control

To control whether or not to render the UI page, we use a bool type field called swagger. The code for the main function above reads:

```go
func main() {
    swaggerTag := flag.Bool("swagger", false, "Whether to generate swagger document at build time")
    flag.Parse()

    r := gin.Default()
    if swaggerTag != nil && *swaggerTag {
       r.GET("/swagger/*any", ginSwagger.WrapHandler(swaggerFiles.Handler))
   }

    v1 := r.Group("/api/v1")
    {
       v1.GET("/hello", api.Hello)
   }
    r.Run(":8080")
}
```

Using the flag library's command parsing, you can pass in swagger commands to control whether or not to expose the swagger UI interface each time you run the service.

First, compile the main function into an executable called main.exe, and then run the main.exe executable (do the following in git):

> go build main.go
./main.exe

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/e287ef995c0440f290d5d0eb7f21fa25%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

The service started successfully.

### 5.2 Accessing a flag-controlled server

#### 5.2.1 Without adding swagger parameters

Next, we access the Swagger UI page by entering the address http://ip+port/swagger/index.html into our browser. We find that the server returns the response code 404: Interface does not exist.

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/fbf696c0dd0f4f64a04aa34133265ab1%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

It seems that the swagger UI page cannot be accessed without specifying the swagger parameter at runtime.

#### 5.2.2 Adding the swagger parameter

Next, let's add the swagger parameter at runtime to turn on access control for the UI page:

. /main.exe -swagger=true![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/1cb806212588432597bea8ee54eddea3~tplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

As you can see, the swagger UI access was successful.

## 6. Summary

Summary of Swagger auto-generated docs:

* When using the swag init command to generate documentation, the docs directory must be referenced in the service main, otherwise the API information cannot be accessed after running;
* Swagger UI access is controlled by ginSwagger.WrapHandler, which can be turned on or off by using the flag library parameter command;
* After modifying the interface definition, you need to use swag init to update the swagger file immediately, and then restart the service to take effect.

Swagger automatically generates documentation from the interface annotations, remember a few commonly used can be used in conjunction with the official documentation to use the best. This idea of annotative programming makes our code more readable, and can be seen in widely used frameworks in the industry, such as Spring Boot, Mybatis, and so on.

At the end of the day, development is a mental job, but more than that, it's a physical job. Therefore, the lazy programming, automated programming approach I personally respect. Extra time to brush the blog, look at the technical articles do not smell it ~ and then do not help, go to the forum to see the black talk discourse, the future to fool the leadership or CPU interns may be useful (🐕)

3. https://link.juejin.cn?target=https%3A%2F%2Fblog.csdn.net%2Fcsdnno11%2Farticle%2Fdetails%2F119634795 "https://blog.csdn.net/csdnno11/article/details/119634795")
