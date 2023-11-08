---
title: gRPC Response to ChatGPT Streaming Q&A
date: 2023-11-06 13:04:00
categories: 
  - Technology
tags: 
  - RPC
  - ChatGPT
  - Remote
  - programmer
  - development
  - framework
  - network
  - communication
description: RPC (Remote Procedure Call) is a computer communication protocol that allows a program running on one computer to call a subroutine in another address space without the programmer having to pay attention to the details by calling it as if it were a local program.
cover: https://raw.githubusercontent.com/zqwuming/blogimage/img/img/e9a2f9a520cd41db834ce8167dabfe02%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp
---
目录

1. 初始RPC
2. RPC与HTTP
3. 流行的RPC框架
4. Protobuf与gRPC
5. gRPC响应ChatGPT问答
6. 小结

## 1. 初始RPC

**RPC 是什么？**

> RPC (Remote Procedure Call) is a computer communication protocol. The protocol allows a program running on one computer to call a subroutine in another address space (usually one computer on an open network), and the programmer calls it as if it were a local program, without having to additionally program for this interaction (no need to pay attention to the details). --Wikipedia

In layman's terms, suppose there are two servers A and B, and two programs (program 1 and program 2) are deployed on these two servers. Since they are two machines, their IP addresses, memory space, etc. are definitely not shared, so how does program 1 call the methods of program 2?

At this point we need to agree on a protocol to allow applications on the two machines to communicate, RPC is such a protocol, it is through the following steps to allow the two programs to recognize the identity of the other program:

1. two machines to send and receive data, so one acts as a server, one acts as a client, they need to ** establish a TCP connection ** (on-demand call, can be a short connection, can also be a long connection);
2. before connecting a TCP connection, the client needs to know **the IP address and port number** of the server, where the IP address is a unique identifier of the host in the network and the port number is a unique identifier of the application program (aka process) on the host;
3. Before communicating, the server runs the application and listens for the corresponding process port number;
4. The client initiates an RPC remote procedure call, passing the parameters of the program interaction to the server, which then transmits them back to the client after processing the received data, disconnects the TCP connection and ends the call.

The whole process is shown in the following figure:

Translated with www.DeepL.com/Translator (free version)

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/e9a2f9a520cd41db834ce8167dabfe02%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

* Client Stub (Client Stub): stores the server address message of the communication and packages the client's request ** into a message body that can be transmitted in the network;
* Server Stub (server-side stub): receives messages sent by the client and packages the return results into message bodies that can be transmitted across the network.

* Sockets (network sockets): a set of program interfaces for applications that can be used to exchange data between different hosts in a network.

Translated with www.DeepL.com/Translator (free version)

## 2. RPC vs. HTTP

**HTTP & RPC, how to choose? **

After learning about RPC, some people may still wonder: since they are both communication protocols, should we choose HTTP (HyperText Transfer Protocol) or RPC protocol for program interaction and application development?

This starts with the attributes of both of them. First, the transfer protocol:

* RPC is a communication protocol based on the TCP transport layer or the HTTP2 application layer;
* HTTP is based on HTTP protocols only, including HTTP1.x (i.e. HTTP1.0, 1.1) and HTTP2, and many browsers currently use 1.x by default to access server data.

Performance consumption (in terms of data type comparison):

* RPC, can be based on gRPC (an RPC framework) to achieve efficient binary transmission;
* HTTP, most of which is implemented through json, byte size and serialization are more performance consuming than gRPC.

On load balancing:

* RPC, basically comes with a load balancing strategy;
* HTTP, need to configure Nginx, HAProxy to realize.

Transfer efficiency:

* RPC, the use of custom TCP protocol, you can make the request message size smaller, or use the HTTP2 protocol, can also be very good to reduce the size of the message, improve the transmission efficiency;
* HTTP, if it is based on HTTP1.x protocol, the request will contain a lot of useless content; if it is based on HTTP2.0, then a simple encapsulation can be used as RPC, which is the standard RPC framework is more advantageous for service governance.

In summary, we can easily find that **RPC from the performance consumption and transmission efficiency, as well as load balancing and other aspects are stronger than HTTP **. At this point, careful friends may have found that, that why our common systems and websites are using the HTTP protocol, do not change to RPC communication?

To give a common example, HTTP is like Mandarin, RPC is like a local dialect, such as Cantonese, southwest of Yunnan, Guizhou, Sichuan.

The advantage of speaking Mandarin is that everyone understands it, and most people speak it, so **HTTP has a certain universality**. Speaking dialect, the advantage is that it can be more concise, more confidential, more customizable, the disadvantage is that the other party who "speaks" the dialect (especially the client side) must also understand, and once everyone speaks a dialect, it is difficult to change the dialect. So **RPC is generally used for internal service calls**, such as between service A and service B in the Ali Taobao system.

## 3. Popular RPC frameworks

> There are many popular RPC frameworks, here are three common ones.

1. `gRPC`: gRPC is an open source project announced by Google in 2015, based on the HTTP2.0 protocol, and supports many common programming languages. The HTTP 2.0 protocol is an upgraded version of the binary-based HTTP protocol, which supports features such as multiple concurrent data transfers.
2. `Thrift`: Thrift is an internal system cross-language RPC framework developed by Facebook, which was contributed to the Apache Foundation in 2007 and has become one of the many open source projects of Apache.
3. `Dubbo`: Dubbo is Alibaba in 2011, an open source RPC framework , in many Internet companies and enterprise applications are widely used , provides a series of protocols and serialization framework , pluggable , but only supports the Java language.

Foreign RPC evaluation, based on the comparison of the test situation of each RPC framework, from the ** throughput rate, response time and stability **, gRPC comprehensive performance is better, but also a lot of domestic companies in the use of the RPC framework. Moreover, gRPC is realized by the go language, with the popularity of microservices, cloud computing, go language companies and projects are increasing, so gRPC has also become the go language internal system communication choice.

gRPC based on **ProtoBuf (Protocol Buffer) serialization protocol ** development, its principle is through the IDL (Interface Definition Language, Interface Description Language) file to define the parameters of the service interface and the type of return value, and then through the code generation tool to generate the template of the server and client code. In this way, we only need to implement an IDL file and business interaction code, and then we can use gRPC to communicate.

A diagram shows the difference between gRPC and HTTP:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/83fe824f83ee4fdea8e7c3446728510f%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

## 4. Protobuf and gRPC

## 4.1 Introduction to Protobuf

Proto Buffer protocol (protobuf for short, same as below), like json and xml, is a kind of data serialization (serialization, that is, converting a piece of data in memory into binary form, and then transferring or storing it over the network).

* protobuf is a cross-language, cross-platform serialization protocol;
* Not only limited to gRPC, protobuf can also be used for data transfer and storage in other scenarios;

Unlike json and xml, protobuf needs to define IDL (Data Definition Rules) when using it, which has the advantage of smaller data size and faster transfer speed.

The transport protocol of gRPC uses protobuf, so we need to learn the rules of writing protobuf files first.

### 4.2 Protobuf Defining Data Structures

Similar to yaml and xml files, protobuf files need to be written in a specific format, and the following is a generalized way to write a protobuf file [gpt.proto]:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/14e2b35a72394ec8b6e25d5cb085a08c%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

In addition to the description in the image above, protobuf files also have some key fields, such as message, which is the most basic type in the protobuf protocol, equivalent to a class object in Java or a struct in Go. As you can see in the figure above, each message has one or more fields and field types, which are equivalent to the parameters and parameter types of an object.

Once we've written the protobuf file, we can start writing the communication logic for gRPC.

### 4.3 gRPC implementation

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/b46af0e783084e20ab6934731e9c14fa%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

As we said above, gRPC is a framework for cross-language communication, so the server and client can be different languages. Next, let's demonstrate the process of implementing gRPC communication in go language.

Steps:

1. write protobuf file
2. generate Go code
3. write the client side to listen on the port
4. write the server side, request data

First, let's create a new project, the directory structure is as follows [wecom project, is used to do GPT interaction, you can imitate, the important folders and file names have been circled in red]:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/0a377672b75149d6a51620a4d6d2b97a%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

#### 1) Writing the protobuf file

According to the above protobuf rules, we first write the protobuf file used in this project [protos/gpt/gpt.proto].

```ini
syntax = "proto3"
​
option go_package = "./;gpt"
​
package gpt
​
service Greeter {
  rpc GetGPTMessage (GPTRequest) returns (GPTReply) {}
}
​
message GPTRequest {
  string content = 1
}
​
message GPTReply {
  string message = 1
}
```

#### 2) Generate Go code

We need to use protoc tool to generate Go language code, first download the toolkit [according to different computer systems to install proto package]:

* [Windows 64位 点这里下载](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fprotocolbuffers%2FProtoBuf%2Freleases%2Fdownload%2Fv3.20.2%2Fprotoc-3.20.2-win64.zip)
* [Mac Intel 64位 点这里下载](https%3A%2F%2Fgithub.com%2Fprotocolbuffers%2FProtoBuf%2Freleases%2Fdownload%2Fv3.20.2%2Fprotoc-3.20.2-osx-x86_64.zip)
* [Mac ARM 64位 点这里下载](https%3A%2F%2Fgithub.com%2Fprotocolbuffers%2FProtoBuf%2Freleases%2Fdownload%2Fv3.20.2%2Fprotoc-3.20.2-osx-aarch_64.zip")
* [Linux 64位 点这里下载](https%3A%2F%2Fgithub.com%2Fprotocolbuffers%2FProtoBuf%2Freleases%2Fdownload%2Fv3.20.2%2Fprotoc-3.20.2-linux-x86_64.zip")

Then install the golang plugin [generating plugins for other languages such as Java and Python is different, see the official gRPC documentation for details]. [https://doc.oschina.net/grpc?t=58008)】

> go install google.golang.org/protobuf/cmd/protoc-gen-go@latest

This completes the installation of our toolkit.

After the installation is complete, go to the directory where the proto files are located:

> cd protos/gpt/

Generate code in Go [if protoc does not exist, the protoc toolkit is not installed or there is a problem with the environment variable settings].

> protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative gpt.proto

At this point, there are 3 files in the current directory [protos/gpt]:

> gpt.pb.go
gpt.proto
gpt_grpc.pb.go

#### 3) Add dependencies

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/85233e41d70247159ee0d4c5c509fb2d%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

If, like me, there is code marked in red after generating the code, you can add the dependency in the main directory [/wecom]:

> go mod tidy

If there is still red after go mod tidy, it may be caused by a lower version of golang's dependencies, so you need to change golang's dependencies in go.mod:

> go 1.18

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/4f538c58aa5e43f4bc04f5da71e4366a%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

#### 4) Write business code to implement the server side

First, import the grpc plugin package in your project

> go get google.golang.org/grpc

Then write the server-side business logic [gpt_server/main.go

```go
package main
​
import (
    "context"
    "flag"
    "fmt"
    "google.golang.org/grpc"
    "log"
    "net"
    pb "wecom/protos/gpt"
)
​
var (
    port = flag.Int("port", 50051, "port")
)
​
type server struct{
    pb.UnimplementedGreeterServer
}
​
func (s *server) GetGPTMessage(ctx context.Context, in *pb.GPTRequest) (*pb.GPTReply, error) {
    return &pb.GPTReply{Message: "gpt response"}, nil
}
​
func main() {
    flag.Parse()
    list, err := net.Listen("tcp", fmt.Sprintf(":%d", *port))
    if err != nil {
        log.Fatalf("listen failed, %v", err)
    }
    s := grpc.NewServer()

    pb.RegisterGreeterServer(s, &server{})
    log.Printf("listen success, %v", list.Addr())
    if err := s.Serve(list); err != nil {
        log.Fatalf("server failed, %v", err)
    }
}
```

Run the main function:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/069a32e234904313bd4211b84f4618c7%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

Next, we implement another client binding to a port to request messages from the server.

#### 5) Client-side Logic

From the server-side implementation above, the gRPC implementation is very simple, just follow the template generated by protobuf to fill in the business code! This process, we only need to focus on the server-side and client-side connection communication, and their connection is not very deep, and our HTTP listening and binding is the same principle.

Client-side business code [gpt_client/main.go

```go
package main
​
import (
   "context"
   pb "dm-lite/resource/proto/gpt"
   "flag"
   "google.golang.org/grpc"
   "google.golang.org/grpc/credentials/insecure"
   "log"
)
​
const defaultName = "world"
​
var (
   addr = flag.String("addr", "localhost:50051", "")
   name = flag.String("name", defaultName, "")
)
​
func main() {
   flag.Parse()
   conn, err := grpc.Dial(*addr, grpc.WithTransportCredentials(insecure.NewCredentials()))
   if err != nil {
      log.Fatalf("Dial failed, %v", err)
   }
   defer conn.Close()
   c := pb.NewGreeterClient(conn)
   ctx := context.Background()
   r, err := c.GetGPTMessage(ctx, &pb.GPTRequest{
      Content: "hello",
   })
   if err != nil {
      log.Fatalf("GetGPTMessage failed, %v", err)
   }
​
   log.Printf("get reply: %v", r.GetMessage())
}
```

When the server is listening on port 50051, we can run the client to call the `GetGPTMessage` method of gRPC. Run the client main function to get the result:![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/1a7878cf680c40d9ae64a8da71edd358%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

POTUS, the gRPC interface call was successful!

## 5. gRPC response to ChatGPT Q&A

**Streaming RPC**

The above implements the gRPC interface for real-time response, i.e.: a simple pattern of one question and one answer. If compared to a scenario in an interview, the simple pattern looks like this:

> (Interviewer) Q: Do you know gRPC?
> (Candidate) A: Yes, gRPC is an RPC framework initiated by Google;
> (Interviewer) Q: What else?
> (Candidate) A: gRPC is based on HTTP/2 protocol transport;
> (Interviewer) Q: What else?
> (Candidate) A: It uses Protocol Buffers as the interface description language;
> (Interviewer) Q: Can we finish this at once?
> (Candidate) A: ......

Then the interviewer asked us to do it, so we can't afford not to do it, so the streaming pattern RPC appears:

> (Interviewer) Q: Do you know gRPC?
> (Candidate) A: Yes, gRPC is an RPC framework initiated by Google... It is based on HTTP/2 protocol transport... And it uses Protocol Buffers as the interface description language.
> (Interviewer) Thinks to himself: not bad! I'm more of a stick-in-the-mud kind of guy than a toothpaste-squeezing kind of guy!

Next, we add a client-side streaming RPC interface to the proto file:

>

```scss
rpc GetGPTStreamData (GPTRequest) returns (stream GPTReply) {}
```

### 5.1 添加流式接口

Improvement of protobuf files [gpt/gpt.proto

```ini
syntax = "proto3"
​
option go_package = "./;gpt"
​
package gpt
​
service Greeter {
  rpc GetGPTMessage (GPTRequest) returns (GPTReply) {}
  rpc GetGPTStreamData (GPTRequest) returns (stream GPTReply) {}
}
​
message GPTRequest {
  string content = 1
}
​
message GPTReply {
  string message = 1
}
```

Following the three-step strategy for gRPC development, we'll start by generating template Go code from a proto file:

> protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative gpt.proto

### 5.2 Server side

Add streaming server-side logic [gpt_server/main.go], note that the following code is a new addition, not an override:

```go
func (s *server) GetGPTStreamData(in *pb.GPTRequest, gptStream pb.Greeter_GetGPTStreamDataServer) error {
   log.Printf("GetGPTStreamData Request: %v", in.GetContent())
   messages := []string{
      "春眠不觉晓",
      "处处闻啼鸟",
      "夜来风雨声",
      "花落知多少",
   }
​
   log.Println("Send reply:")
   for _, msg := range messages {

      if err := gptStream.Send(&pb.GPTReply{
         Message: msg,
      }); err != nil {
         log.Printf("Send error, %v", err)
         return err
      }
      time.Sleep(1 * time.Second)
   }
   return nil
}
```

First, start listening on the server side:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/e6aa9aa617a140bda4f4e1b7a03e52ff%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

### 5.3 Client

Streaming message reception, client code [gpt_client/main.go

```go
package main
​
import (
   "context"
   "flag"
   "fmt"
   "io"
   "log"
   "time"
   pb "wecom/protos/gpt"
​
   "google.golang.org/grpc"
   "google.golang.org/grpc/credentials/insecure"
)
​
const defaultName = "world"
​
var (
   addr = flag.String("addr", "localhost:50051", "")
   name = flag.String("name", defaultName, "")
)
​
func main() {
   flag.Parse()
   conn, err := grpc.Dial(*addr, grpc.WithTransportCredentials(insecure.NewCredentials()))
   if err != nil {
      log.Fatalf("Dial failed, %v", err)
   }
   defer conn.Close()
   c := pb.NewGreeterClient(conn)
   ctx, cancel := context.WithTimeout(context.Background(), 60*time.Second)
   defer cancel()
   steam, err := c.GetGPTStreamData(ctx, &pb.GPTRequest{
      Content: "背一下古诗《春眠》",
   })
   if err != nil {
      log.Fatalf("GetGPTMessage failed, %v", err)
   }
   log.Println("Get reply:")
   for {
      res, err := steam.Recv()
      if err == io.EOF {
         break
      }
      if err != nil {
         log.Fatalf("Recv failed, %v", err)
      }
      fmt.Printf("%v", res.GetMessage())
   }
}
```

Start the main function of the server and client, the code runs as follows:

/ _Video not supported_/

OK, the streaming gRPC responds successfully.

Since the project is doing ChatGPT recently, some users will use the streaming response Q&A, so we call the streaming Q&A interface of ChatGPT next to show the daily use scenario of the streaming interface.

### 5.4 GPT Streaming Q&A Demonstration

#### 1) Server-side logic

```go
func (s *RPCServe) GetGPTStreamData(in *pb.GPTRequest, gptStream pb.Greeter_GetGPTStreamDataServer) error {
   log.Printf("GetGPTStreamData Request: %v", in.GetContent())
   client := openai.NewClient(OPENAI_API_KEY)
   ctx := context.Background()
​

   req := openai.ChatCompletionRequest{
      Model:     openai.GPT3Dot5Turbo,
      MaxTokens: 2048,
      Messages: []openai.ChatCompletionMessage{
         {
            Role:    openai.ChatMessageRoleUser,
            Content: in.GetContent(),
         },
      },
      Stream: true,
   }

   stream, err := client.CreateChatCompletionStream(ctx, req)
   if err != nil {
      log.Fatalf("ChatCompletion failed, %v", err)
      return err
   }
   defer stream.Close()
​
   log.Println("Send reply:")
   for {
      response, err := stream.Recv()

      if errors.Is(err, io.EOF) {
         log.Printf("Stream finished")
         break
      }
​
      if err != nil {
         log.Fatalf("Stream error, %v", err)
         return err
      }
​

      data := &pb.GPTReply{
         Message: response.Choices[0].Delta.Content,
      }

      if err := gptStream.Send(data); err != nil {
         log.Printf("Send error, %v", err)
         return err
      }
   }
   return nil
}
```



#### 2) Client Logic

Streaming message reception, client code [gpt_client/main.go

```go
package main
​
import (
   "context"
   pb "dm-lite/resource/proto/gpt"
   "flag"
   "fmt"
   "google.golang.org/grpc"
   "google.golang.org/grpc/credentials/insecure"
   "io"
   "log"
   "time"
)
​
const defaultName = "world"
​
var (
   addr = flag.String("addr", "localhost:50051", "")
   name = flag.String("name", defaultName, "")
)
​
func main() {
   flag.Parse()
   conn, err := grpc.Dial(*addr, grpc.WithTransportCredentials(insecure.NewCredentials()))
   if err != nil {
      log.Fatalf("Dial failed, %v", err)
   }
   defer conn.Close()
   c := pb.NewGreeterClient(conn)
   ctx, cancel := context.WithTimeout(context.Background(), 60*time.Second)
   defer cancel()
   steam, err := c.GetGPTStreamData(ctx, &pb.GPTRequest{
      Content: "写一篇500字的作文，题目为"梦想"",
   })
   if err != nil {
      log.Fatalf("GetGPTMessage failed, %v", err)
   }
   log.Println("Get reply:")
   for {
      res, err := steam.Recv()
      if err == io.EOF {
         break
      }
      if err != nil {
         log.Fatalf("Recv failed, %v", err)
      }
      fmt.Printf("%v", res.GetMessage())
   }
}
```

Start the main function of the server and client, the code runs as follows:

/ _Video not supported_/

It is not difficult to find that in the above Q&A scenarios if there is no streaming response, the interface in the case of slow return, it will greatly affect the user experience. That's why some people say that love is a fine line, have you realized it?

> The above code address: [github.com/yangfx15/rp...] (https://github.com/yangfx15/rpc_test)

## 6. Summary

In this paper, we talked about RPC, mentioned the basic concepts of RPC and the difference between HTTP communication protocols, and the commonly used RPC communication frameworks. Then, we wrote a protobuf file according to the characteristics of gRPC and ran a simple gRPC communication program. Finally, since the project uses ChatGPT, we used the characteristics of gRPC streaming response and ChatGPT to make a simple streaming Q&A demo.

In this process, it is easy to realize that the interaction between gRPC and HTTP is very similar, but the advantage of gRPC is that the packet size is smaller and the communication is faster.

Therefore, gRPC is a very efficient way to interact with internal systems that communicate frequently. And it is also open-sourced by Google, just like go, so the community is very active, and there are perfect solutions to the common problems you usually encounter.
