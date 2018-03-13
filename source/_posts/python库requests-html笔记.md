---
title: python库requests-html笔记
date: 2018-03-13 10:08:43
categories: 技术
tags:
- python
---

前段日子看到一个很火的 python 库。叫 requests-html ，作者就是 requests 库的作者，所以这是一个看作者名字就可以 star 的项目。话说作者太秀了，一个摄影师，写了一个大名鼎鼎的 requests 库，长得还帅。。赢家了。

之前写 web 请求和 html 解析用的都是 requests + BeautifulSoup 。requests 库完胜 urllib，然后 BeautifulSoup 说实话实在繁琐和有一些反人类。requests-html 在 requests 基础上直接做了 html 的解析，还加了一些 feature。

## feature
- 完全支持 JavaScript。这个很关键，以前抓取通过 JavaScript 异步加载的数据我是通过抓包来做的，现在可以 render() 直接渲染了。
- CSS 选择器。类 JQuery 语法，可以说是非常人性化和强大了。
- XPath 选择器。
- 伪造的请求头。（就像真的一样。。）
- 自动的重定向。
- 连接池和 cookie 的持久化。
- requests 的各种魔法般的语法都被保留。

## 安装
只支持 python3.6 。
```
pip install requests-html
```

## 请求
使用方法和 requests 一样。
```python
>>> from requests_html import HTMLSession
>>> session = HTMLSession()

>>> r = session.get('https://python.org/')
>>> r
<Response [200]>
```

## HTML 解析
获取所有链接
```python
>>> r.html.links
{'/',
 '//docs.python.org/3/tutorial/',
 '//docs.python.org/3/tutorial/controlflow.html',
 '//docs.python.org/3/tutorial/controlflow.html#defining-functions',
 '//docs.python.org/3/tutorial/introduction.html#lists',
 '//jobs.python.org',
 '/about/',
 '/about/apps',
 ...
}
```

获取网页代码
```python
>>> r.html.html
'<!doctype html>\n<!--[if lt IE 7]>   <html class="no-js ie6 lt-ie7 lt-ie8 lt-ie9">   <![endif]-->\n<!--[if IE 7]>      <html class="no-js ie7 lt-ie8 lt-ie9">          <![endif]-->\n<!--[if IE 8]>      <html class="no-js ie8 lt-ie9">                 <![endif]-->\n<!--[if gt IE 8]><!--><html class="no-js" lang="en" dir="ltr">  <!--<![endif]-->\n\n<head>\n    <meta charset="utf-8">\n    <meta http-equiv="X-UA-Compatible" content="IE=edge">\n\n    <link rel="prefetch" href="//ajax.googleapis.com/ajax/libs/jquery/1.8.2/jquery.min.js">\n\n    <meta name="application-name" content="Python.org">\n    <meta name="msapplication-tooltip" content="The official home of the Python Programming Language">\n    <meta name="apple-mobile-web-app-title" content="Python.org">\n    <meta name="apple-mobile-web-app-capable" content="yes">\n    <meta name="apple-mobile-web-app-status-bar-style" content="black">\n\n    <meta name="viewport" content="width=device-width, initial-scale=1.0">\n    <meta name="HandheldFriendly" content="True">\n    <meta name="format-detection" content="telephone=no">\n ...'
```

JQuery 语法选中元素
```python
>>> about = r.html.find('#about', first=True)
>>> about
<Element 'li' id='about' class=('tier-1', 'element-1') aria-haspopup='true'>
```

获取元素的文本
```python
>>> print(about.text)
About
Applications
Quotes
Getting Started
Help
Python Brochure
```

获取元素的属性
```python
>>> about.attrs
{'aria-haspopup': 'true', 'class': ('tier-1', 'element-1'), 'id': 'about'}
```

封装了常用正则，这个语法太直观了。。
```python
>>> r.html.search('Python is {} language')[0]
'a programming'
```

还有一个特别直观的元素层级选择器。。
```python
>>> r = session.get('https://github.com/')
>>> sel = 'body > div'
>>> print(r.html.find(sel, first=True).text)
Skip to content
Features
Business
Explore
Marketplace
Pricing
/dashboard
Sign in or Sign up
```

find() 还有一个特别实用的语法糖——containing 参数。
```python
>>> r = session.get('http://python-requests.org/')
>>> r.html.find('a', containing='kenneth')
[<Element 'a' href='http://kennethreitz.com/pages/open-projects.html'>,
 <Element 'a' href='http://kennethreitz.org/'>,
 <Element 'a' href='https://twitter.com/kennethreitz' class=('twitter-follow-button',) data-show-count='false'>,
 <Element 'a' class=('reference', 'internal') href='dev/contributing/#kenneth-reitz-s-code-style'>]
```

## 分页
文档里说 `There’s also intelligent pagination support (always improving)`， 有一个长期改进的智能分页功能。
```python
>>> r = session.get('https://reddit.com')
>>> for html in r.html:
...     print(html)
<HTML url='https://www.reddit.com/'>
<HTML url='https://www.reddit.com/?count=25&after=t3_83x8iq'>
<HTML url='https://www.reddit.com/?count=50&after=t3_83w1oo'>
<HTML url='https://www.reddit.com/?count=75&after=t3_83vkt2'>
<HTML url='https://www.reddit.com/?count=100&after=t3_83wm8g'>
...
```

当然也可以用 next()，一个类似迭代器的用法。
```python
>>> r = session.get('https://reddit.com')
>>> r.html.next()
'https://www.reddit.com/?count=25&after=t3_81pm82'
```

## JavaScript 支持
这个部分在今天，也就是 2018 年 3 月 13 日感觉还没有完全完善。因为我在 window10 用 render() 报了很诡异的错误，然后我在 issue 里看到好像在 window10 平台确实有点问题。然后我在 ubuntu 跑了一下就可以了。。

作者仍在更新和优化。

```python
>>> from requests_html import HTML
>>> doc = """<a href='https://httpbin.org'>"""

>>> html = HTML(html=doc)
>>> script = """
        () => {
            return {
                width: document.documentElement.clientWidth,
                height: document.documentElement.clientHeight,
                deviceScaleFactor: window.devicePixelRatio,
            }
        }
    """
>>> val = html.render(script=script, reload=False)

>>> print(val)
{'width': 800, 'height': 600, 'deviceScaleFactor': 1}
```

## 参考
- [GITHUB](https://github.com/kennethreitz/requests-html)
- [文档](http://html.python-requests.org/)

这个库感觉要一统江湖的节奏。。