---
title: Redis founder open source the smallest chat server, only 200 lines of code, a few days work has been 2.8K Star!
date: 2023-11-05 00:08:00
categories: 
  - Technology
tags: 
  - lunchtime
  - chatting
  - facilities
  - fictio
  - intelligence
  - technical
  - source
  - Redis
description: At lunchtime, we were chatting about some interesting things about the founder of Redis in a technical exchange group, such as writing science fiction novels after leaving Redis.
cover: https://s2.loli.net/2023/11/05/6sZPXHhpjzSDyF1.jpg
---
At lunchtime, we were chatting about some interesting things about the founder of `Redis` in a technical exchange group, such as writing science fiction novels after leaving `Redis`.

Because I was curious about science fiction, TJ searched for it. And I found that the `Redis` author has actually done a new job recently!

## The world's smallest chat server

This time the Redis author's new open source project is called:[**SmallChat**](https://link.juejin.cn?target=https%3A%2F%2Fwww.didispace.com%2Ftj%2Ftj-smallchat.html "https://www.didispace.com/tj/tj-smallchat.html")。 As you can tell from the about content, this open source project is to build the smallest chat server.

From the content of the open source project, it is true, just the following:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c71361924e60435eae111c5026fc4524~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

The code section is a staggering 200+ lines of code after removing a lot of comments, so it's really the ultimate in streamlining.

## Origins and Future

In the README of this project, there are no more instructions on how to use this project, but more on the background and future outlook of this project.

The content is also very worthwhile for everyone to savor, and TJ feels the mindset of a good developer, which is very worthwhile for everyone to learn. We can also learn more from this way of thinking to create something more interesting.

Here's a look at his story:

Yesterday I was talking with a few friends, mostly front-end developers, a bit far from systems programming. We were reminiscing about the old days of IRC. I said: writing a very simple IRC server is an experience that everyone should do (I showed them my implementation written in TCL; I was shocked that I wrote it 18 years ago: time flies). There are some very interesting parts of such a program. A single process performing multiplexing, getting client state and trying to quickly access such state after the client has new data, and so on.

But then the discussion changed, and I thought I would show you a very simple example written in C. What is the smallest chat server you can write? To be really minimal, we shouldn't need any proper clients. Even if it's not very good, it should work with telnet or `netcat`. The main operation of the server is just to receive some chat lines and send them to all other clients, sometimes called a fanout operation. However, this requires proper functionality, and then buffering and so on. We'd like it to be simpler: let's use kernel buffers for spoofing and pretend that we receive the full line from the client every time (this assumption is usually correct in practice, so things `kinda` work).

Well, with these tricks, we can implement a chat that even enables users to set their nicknames (removing spaces and comments, of course) in just 200 lines of code. Since I wrote this little program as an example for my friends, I decided to push it to GitHub as well.

About future work:

In the next few days, I will continue to modify this program in order to evolve it. The different evolutionary steps will be labeled according to the YouTube episodes of my Writing System software series (which covers such changes). Here's my plan (it may change, but more or less this is what I want to cover):

* Implement buffering of reads and writes
* Avoid linear arrays and use a dictionary data structure to hold client state
* Write a proper client: line editing that can handle asynchronous events
* Switch from select(2) to a more advanced API
* Simple symmetric encryption for chat

How about it? An interesting open source project was born. Well, today's sharing is here. Finally, the old rules , open source address:[github.com/antirez/sma...](https://github.com/antirez/smallchat) ， Go around the code if you're interested.
