---
title: Microsoft open source the strongest Python automation gods Playwright!
date: 2023-11-05 00:07:00
categories: 
  - Technology
tags: 
  - Python
  - monitoring
  - facilities
  - improve
  - intelligence
  - machine
  - based
  - learning
description: Hello everyone, I am brother boy. I believe that friends who have played the crawler know selenium, an automated testing tool. Write a Python automation script to free your hands is basically a routine operation, crawlers can not crawl, use automated testing to get together. Although selenium has a complete documentation, but also requires a certain learning cost, for a pure white in terms of some threshold.…
cover: https://s2.loli.net/2023/11/05/h2aYw7jCZk5BdRn.webp
---
![](https://s2.loli.net/2023/11/05/h2aYw7jCZk5BdRn.webp)

Hello everyone, I am brother boy.

I believe that friends who have played the crawler know `selenium`, an automated testing tool. Writing a `Python` automation script to free your hands is basically a routine operation, and if you can't crawl with a crawler, you can use automated tests to make up for it.

Although `selenium` has a complete documentation, but also requires a certain learning cost, for a pure white still some threshold.

Recently, Microsoft open-sourced a project called `playwright-python`, which is simply awesome! This project is for the ` Python` language automation tools , even without writing code , you can realize the automation function .

![](https://s2.loli.net/2023/11/05/pQZyunt543r8kAO.webp)

It may seem a bit unbelievable to you, but it's that awesome. Let's take a look at this magic tool together.

## 1. Introduction to Playwright

`Playwright` is a powerful Python library that automates major browser automations such as `Chromium`, `Firefox`, `WebKit`, etc. with just one API, and supports running in headless mode, header mode at the same time.

The automation technology provided by Playwright is green, powerful, reliable and fast, and supports `Linux`, `Mac` and `Windows` operating systems.

![](https://s2.loli.net/2023/11/05/yuKScwrk3YpdPQE.webp)

## 2. Using Playwright

### Installation

Installation of `Playwright` is very simple, two steps.

```bash

pip install playwright

python -m playwright install
```

The two pip operations above are installed separately:

* Install the Playwright dependency library, which requires Python 3.7+.
* Install driver files for Chromium, Firefox, WebKit and other browsers.

### Recording

To use `Playwright` you don't need to write a single line of code, we just need to operate the browser manually, it will record our actions and then generate the code script automatically.

Here is the recorded command `codegen`, just one line.

```python

python -m playwright codegen
```

The usage of "codegen" can be viewed with "--help", the simple usage is to add the url link directly after the command, or add "options" if otherwise needed.

```python
python -m playwright codegen --help
Usage: index codegen [options] [url]

open page and generate code for user actions

Options:
  -o, --output   saves the generated script to a file
  --target        language to use, one of javascript, python, python-async, csharp (default: "python")
  -h, --help                display help for command

Examples:

  $ codegen
  $ codegen --target=python
  $ -b webkit codegen https://example.com
```

options meaning:

* -o: save the recorded script to a file.
* --target: specify the language to generate the script, there are two kinds, `JS` and `Python`, the default is Python.
* --b: specify the browser driver

For example, I want to search on `baidu.com`, use the `chromium` driver, and save the result as a `my.py` file in `python`.

```arduino
python -m playwright codegen --target python -o 'my.py' -b chromium https:
```

Command line input will automatically open the browser, and then you can see that every move on the browser will be automatically translated into code, as shown below.

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/91da3a38f52f4c598fd34017b7139b1d~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.webp)

Automatically close the browser when finished and save the generated automation script to a py file.

```scss
from playwright import sync_playwright

def run(playwright):
    browser = playwright.chromium.launch(headless=False)
    context = browser.newContext()

    # Open new page
    page = context.newPage()

    page.goto("https://www.baidu.com/")

    page.click("input[name=\"wd\"]")

    page.fill("input[name=\"wd\"]", "jingdong")

    page.click("text=\"京东\"")

    # Click //a[normalize-space(.)='京东JD.COM官网 多快好省 只为品质生活']
    with page.expect_navigation():
        with page.expect_popup() as popup_info:
            page.click("//a[normalize-space(.)='京东JD.COM官网 多快好省 只为品质生活']")
        page1 = popup_info.value
    # ---------------------
    context.close()
    browser.close()

with sync_playwright() as playwright:
    run(playwright)
```

In addition, `playwright` provides both synchronous and asynchronous API interfaces, documented below.

> 链接：[microsoft.github.io/playwright-...](https://link.juejin.cn?target=https%3A%2F%2Fmicrosoft.github.io%2Fplaywright-python%2Findex.html "https://microsoft.github.io/playwright-python/index.html")

### Synchronization

The following sample code: open three browsers in turn, go to baidu search, screenshot and exit.

```python
from playwright import sync_playwright

with sync_playwright() as p:
    for browser_type in [p.chromium, p.firefox, p.webkit]:
        browser = browser_type.launch()
        page = browser.newPage()
        page.goto('https://baidu.com/')
        page.screenshot(path=f'example-{browser_type.name}.png')
        browser.close()
```

### Asynchronous

Asynchronous operations can be combined with `asyncio` to perform three browser operations simultaneously.

```python
import asyncio
from playwright import async_playwright

async def main():
    async with async_playwright() as p:
        for browser_type in [p.chromium, p.firefox, p.webkit]:
            browser = await browser_type.launch()
            page = await browser.newPage()
            await page.goto('http://baidu.com/')
            await page.screenshot(path=f'example-{browser_type.name}.png')
            await browser.close()

asyncio.get_event_loop().run_until_complete(main())
```

### Mobile

What's more, `playwright` also supports browser emulation on mobile. Here's a snippet of code from the official documentation that simulates the Safari browser on an iphone 11 pro at a given geographic location, first navigating to `maps.google.com`, then executing the location and taking a screenshot.

```python
from playwright import sync_playwright

with sync_playwright() as p:
    iphone_11 = p.devices['iPhone 11 Pro']
    browser = p.webkit.launch(headless=False)
    context = browser.newContext(
        **iphone_11,
        locale='en-US',
        geolocation={ 'longitude': 12.492507, 'latitude': 41.889938 },
        permissions=['geolocation']
    )
    page = context.newPage()
    page.goto('https://maps.google.com')
    page.click('text="Your location"')
    page.screenshot(path='colosseum-iphone.png')
    browser.close()
```

It can also be used with the `pytest` plugin, so you can try it yourself.

## 3. Summary

`playwright` has many advantages over existing automated testing tools, such as:

* Cross-browser, supports Chromium, Firefox, WebKit.
* Cross-operating system, supports Linux, Mac, Windows.
* can provide recording code generation function, free hands
* Can be used for mobile

Currently there are shortcomings is that the ecology and documentation is not very complete, such as no API Chinese documentation, no better tutorials and examples for learning. However, I believe that as more and more people know, the future will be better and better.

> GitHub Link：[github.com/microsoft/p...](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fmicrosoft%2Fplaywright-python "https://github.com/microsoft/playwright-python")
open source organization：Microsoft

