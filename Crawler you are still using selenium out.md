---
title: Crawler you are still using selenium out
date: 2023-11-05 00:04:00
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
description: Crawler you are still using selenium, out! Take you to recognize a new domestic crawler framework, absolutely nice, absolutely simple and lightweight!
cover: https://s2.loli.net/2023/11/05/IYKOjTgmr6WFvSw.webp
---

### Synopsis

Recently encountered a thing: my chrome browser upgraded, but the corresponding webdriver has not been upgraded, I can only be forced to accept the use of safari browser to implement the crawler.

![](https://s2.loli.net/2023/11/05/ZU3PYOIykNp4gMn.webp)

Although it's the browser that comes with mac, I'm so used to chrome that I can't change my habits. But lately posting news is still forced to use safari as the browser.

I have also been from slenium as the crawler framework, it is the main webdriver, so there are a number of problems:

1. configuration is more troublesome, for newbies may not be very friendly
2. the version must match the version of the browser. I have a period of time is because of chrome upgrade, but the driver did not upgrade resulting in the use of scripts can not be operated server
3. selenium new version of the api and the old version of the big difference. When I was solving the problem, I found that many of the code examples given in the old documentation are no longer available in the new version.

Well, now the savior is here, selenium as a crawler tool has become history.

DrissionPage is a python based web automation tool. It can control the browser as well as send and receive packets, and it can combine the two into one. It combines the convenience of browser automation with the efficiency of requests. It's powerful, with tons of built-in user-friendliness and convenience. Its syntax is clean and elegant, and its code is small and newbie-friendly.

This is quoted from the [official DrissionPage documentation](https://link.juejin.cn?target=https%3A%2F%2Fg1879.gitee.io%2Fdrissionpagedocs%2F "https://g1879.gitee.io/ drissionpagedocs/"), but you'll have to check it out to see how it works. Come explore with `shigen`.

### Installation

```
&#xA0;pip install DrissionPage
```

![](https://s2.loli.net/2023/11/05/3V1MQGNCFnEpKJu.webp)

### Code test

According to the official case: [collect cat eye movie top100 list](https://link.juejin.cn?target=https%3A%2F%2Fg1879.gitee.io%2Fdrissionpagedocs%2Fdemos%2Fmaoyan_TOP100%2F " https://g1879.gitee.io/drissionpagedocs/demos/maoyan_TOP100/"), we directly copy and paste the code.

![](https://s2.loli.net/2023/11/05/tgsihvzoP7uDe2U.webp)

I waited a few seconds for it to open a new web tab page, and it was paging like crazy, and then the data was all in the `data.csv`.![](https://s2.loli.net/2023/11/05/5UJBEtmY2HkVzGu.webp)

This is much easier than using the `requests` library before!

The `shigen` is straight up fun, and I'm going to have to build my own code for the next one! Crawling [minimalist wallpaper](https://link.juejin.cn?target=https%3A%2F%2Fbz.zzzmh.cn%2Findex "https://bz.zzzmh.cn/index"). But it's also a bit heartbreaking for the author to have to endure such a traffic attack on a free site. Surely that proves the saying: **Free is the most expensive! **

![](https://s2.loli.net/2023/11/05/qVnU6CHabKL3WZ4.webp)

![](https://s2.loli.net/2023/11/05/JwazWVv1HsR7Ci6.webp)

The code is so simple, but in the end it did not download successfully, the front-end dealt with the address of the file.

![](https://s2.loli.net/2023/11/05/BZI2paNYlVksDWP.webp)

I'm sure we'll get a good solution later, and `shigen` will be updated continuously. Anyway, `drissionPage` is a great framework!

For more information on how to use it, you can also check out the documentation.

