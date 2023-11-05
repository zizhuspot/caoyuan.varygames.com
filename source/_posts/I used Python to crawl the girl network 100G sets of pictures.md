---
title: I used Python to crawl the girl network 100G sets of pictures
date: 2023-11-05 00:06:00
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
description: Preface Recently, I have been working on monitoring related support facilities and found that many scripts are based on Python. A long time ago I heard of its great name, life is short, I learn Python, this is not a joke. With the rise of artificial intelligence, machine learning, deep learning, most of the artificial intelligence currently on the market
cover: https://s2.loli.net/2023/11/05/cxQ2T1yfdtn7ZHF.png
---
![](https://s2.loli.net/2023/11/05/cxQ2T1yfdtn7ZHF.png)

## **Preface**

Recently, I've been working on surveillance-related packages and found that many scripts are based on Python. I heard its great name a long time ago, life is short, I learn Python, this is not a joke. With the rise of Artificial Intelligence, Machine Learning, Deep Learning, most of the AI code on the market is written in Python. So in the age of artificial intelligence, it's time to learn some Python.

**Advancement Guide

For those who don't have any language development experience, it is recommended to learn it systematically from the beginning, whether it is a book, video or text tutorial.

For students with development experience in other languages, it is recommended to start with a case study, such as crawling a set of images from a certain website.

Because the language is figured out, grammar and so on, as long as you want to sense of language, the code can basically read a eight or nine.

So it is not recommended that experienced developers learn from scratch, whether it is a video or a book, for the beginning of learning a language is too much time.

Of course, when you go deeper into it, you still need to learn it systematically, which is an afterthought.

## **Software tools***

#### **Python3**

The latest version of Python 3.7.1 is chosen here.

Recommended installation tutorial:

http://www.runoob.com/python3/python3-install.html

Win Download Address:

https://www.python.org/downloads/windows

Linux download address:

https://www.python.org/downloads/source

#### **PyCharm**

Visualization development tools:

http://www.jetbrains.com/pycharm

## **Cases**

**Realization steps**

Take the girl picture as an example, it is actually very simple, divided into the following four steps:

* Get the number of pages on the home page, and create a folder corresponding to the page number
* Get the column address of the page
* into the column, get the column page number (each column has multiple pictures under the page display)
* get to the columns under the pair of tags in the picture and download

### **Note **

Crawling process, also need to pay attention to the following points, may be helpful to you:

1) guide library, in fact, it is similar to the framework or tools in Java, the bottom are encapsulated

Installation of third-party libraries

```python
# Win下直接装的 python3
pip install bs4、pip install requests
# Linux python2 python3 共存
pip3 install bs4、pip3 install requests
```

Importing third-party libraries

```python
# 导入requests库
import requests
# 导入文件操作库
import os
# bs4全名BeautifulSoup，是编写python爬虫常用库之一，主要用来解析html标签。
import bs4from bs4 
import BeautifulSoup
# 基础类库
import sys
# Python 3.x 解决中文编码问题
import importlibimportlib.reload(sys)

```

2）Define the method function, a crawler may be several hundred lines, so try not to write a bunch of

```python
def download(page_no, file_path):    # 这里写代码逻辑

```

3) Define global variables

```python
# 给请求指定一个请求头来模拟chrome浏览器
global headers 
# 告诉编译器这是全局变量 
headers headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.99 Safari/537.36'}
# 函数内使用之前需要
# 告诉编译器我在这个方法中使用的a是刚才定义的全局变量 headers ，而不是方法内部的局部变量。
global headers

```

4) Anti-theft chain

Some sites have anti-piracy links, the omnipotent python solution

```python
headers = {'Referer': href}img = requests.get(url, headers=headers)
```

5) Switching Versions

The Linux server is using AliCloud server, the default version python2, python3 install your own

```python
[root@AY140216131049Z mzitu]# python2 -VPython 2.7.5
[root@AY140216131049Z mzitu]# python3 -VPython 3.7.1
# 默认版本
[root@AY140216131049Z mzitu]# python -VPython 2.7.5
# 临时切换版本 <whereis python>
[root@AY140216131049Z mzitu]# alias python='/usr/local/bin/python3.7'
[root@AY140216131049Z mzitu]# python -VPython 3.7.1

```

6) Abnormal Capture

In the process of crawling there may be an exception page, here we capture, does not affect the subsequent operations

```python
try:    # 业务逻辑except Exception as e:   print(e)
```

### **Code implementation**

Edit script: vi mzitu.py

```python
# coding=utf-8
# !/usr/bin/python
# 导入requests库
import requests
# 导入文件操作库
import os
import BeautifulSoup from bs4
import sys
import importlib
importlib.reload(sys)
# 给请求指定一个请求头来模拟chrome浏览器
global headersheaders = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64)AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.99 Safari/537.36'}
# 爬图地址
mziTu = 'http://www.mzitu.com/'
# 定义存储位置
global save_pathsave_path = '/mnt/data/mzitu'


# 创建文件夹
def createFile(file_path):
    if os.path.exists(file_path)
    is False:
        os.makedirs(file_path)
        # 切换路径至上面创建的文件夹    
        os.chdir(file_path)

# 下载文件
def download(page_no, file_path):
    global headers
    res_sub = requests.get(page_no, headers=headers) 
    # 解析html    
    soup_sub = BeautifulSoup(res_sub.text, 'html.parser')
    # 获取页面的栏目地址    
    all_a = soup_sub.find('div', class_='postlist').find_all('a', target='_blank')
    count = 0
    for a in all_a:
        count = count + 1
        if (count % 2) == 0:
            print("内页第几页：" + str(count))
            # 提取href            
            href = a.attrs['href']
            print("套图地址：" + href)
            res_sub_1 = requests.get(href, headers=headers)
            soup_sub_1 = BeautifulSoup(res_sub_1.text, 'html.parser')
            # ------ 这里最好使用异常处理 ------            
            try: 
                # 获取套图的最大数量                
                pic_max = soup_sub_1.find('div', class_='pagenavi').find_all('span')[6].text
                print("套图数量：" + pic_max)
                for j in range(1, int(pic_max) + 1):
                    # print("子内页第几页：" + str(j))
                    # j int类型需要转字符串
                    href_sub = href + "/" + str(j)
                    print(href_sub)
                    res_sub_2 = requests.get(href_sub, headers=headers)
                    soup_sub_2 = BeautifulSoup(res_sub_2.text, "html.parser")
                    img = soup_sub_2.find('div', class_='main-image').find('img')
                    if isinstance(img, bs4.element.Tag):
                        # 提取src                       
                        url = img.attrs['src']
                        array = url.split('/')
                        file_name = array[len(array) - 1]
                        # print(file_name)                        
                        # 防盗链加入Referer                        
                        headers = {'Referer': href}
                        img = requests.get(url, headers=headers)
                        # print('开始保存图片')                       
                        f = open(file_name, 'ab')
                        f.write(img.content) 
                        # print(file_name, '图片保存成功！')
                        f.close()
            except Exception as e:
                print(e)

# 主方法
def main():
    res = requests.get(mziTu, headers=headers)
    # 使用自带的html.parser解析    
    soup = BeautifulSoup(res.text, 'html.parser')
    # 创建文件夹    
    createFile(save_path)
    # 获取首页总页数    
    img_max = soup.find('div', class_='nav-links').find_all('a')[3].text
    # print("总页数:"+img_max)    
    for i in range(1, int(img_max) + 1):
        # 获取每页的URL地址        
        if i == 1:
            page = mziTu
        else:
        page = mziTu + 'page/' + str(i)
        file = save_path + '/' + str(i)
        createFile(file)
        # 下载每页的图片        
        print("套图页码：" + page)
        download(page, file)


if __name__ == '__main__':
    main() 
```

The script runs under the Linux server by executing the following command

```python
python 3 mzitu.py 
# 或者后台执行
nohup python3 -u mzitu.py > mzitu.log 2>&1 &
```

Currently only crawled a column of sets of pictures, a total of 17G, 5332 pictures.

```python
[root@itstyle mzitu]# du -sh 17G     .
[root@itstyle mzitu]# ll -stotal 5332
```

Below, please keep your eyes open for the cockamamie set moment.

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/11/12/16707ed15707ecaa~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png)

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/11/12/16707ed15710de0b~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png)

## **Summary***

As a beginner, the script must have more or less some problems or places to be optimized, such as the encounter Python aunt, but also please more guidance.

In fact, the script is very simple, from the configuration of the environment, the installation of the integrated development environment, write the script to the smooth execution of the entire script, almost spent four or five hours, and ultimately the script a sinewy execution. Limited to the server bandwidth and configuration of the impact of the 17G figure almost downloaded three or four hours, as for the rest of the 83G, partners download it on their own.
![](https://s2.loli.net/2023/11/04/vby3RNFwuTiUzr6.png)

—END—

