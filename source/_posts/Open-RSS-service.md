---
title: Open RSS service
date: 2018-06-18 16:37:22
categories: record
tags:
- RSS
---

You may not believe it that I have never used RSS service. Before now if I wanted to browse others' blog, I will open anyone's blog directly... In other words, I need to find someone first and then browse his blog... Sounds silly...

Some time ago, I find a repository in Github about RSS —— [DIYgod/RSSHub](https://github.com/DIYgod/RSSHub). The repository works on using RSS to connect the world. On account of it, I started to acquaint RSS.

## XML
I often used XML, because of his simple grammar and similarity to HTML, I haven't deepened understanding to XML. Because XML is the understructure to HTML, I just like to take this opportunity to look at the specification of XML.

XML(e**X**tensible **M**arkup **L**anguage) is a markup language that defines a set fo rules for encoding document in a format. XML's grammar resemble HTML's grammar. 

### simple demo

A simple demo of XML:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<note>
  <to>Tove</to>
  <from>Jani</from>
  <heading>Reminder</heading>
  <body>Don't forget me this weekend!</body>
</note>
```

First line is called the XML prolog.
```xml
<?xml version="1.0" encoding="UTF-8"?>
```

### entity references
Some characters have a special meaning in XML. So there are 5 pre-defined entity references in XML:
- `&lt;` == `<`
- `&gt;` == `>`
- `&amp;` == `&`
- `&apos;` == `'`
- `&quot;` == `"`

### stores new line as LF
XML stores a new line as LF.

### More details
- Validator using DTD
- CDATA
- XSLT
- ...

A lot of advanced usage I will not launch a discussion.

## RSS
RSS(often called **Really Simple Syndication**) is a type of web feed. In a simple way to say, RSS is used to subscribe website you interested.

There is no things to say. Just download a RSS reader, and input the website you interested, you can subscribe it if it open RSS service.

And I want to recommend a repository [DIYgod/RSSHub](https://github.com/DIYgod/RSSHub). It can help us subscribe much information that traditional RSS cannot via its filter function.

And I open RSS service today...

```yml
feed:
  type: atom
  path: atom.xml
  limit: 20
  hub:
  content: true
  content_limit: 140
  content_limit_delim: ' '
```


## Others
I cannot buy the high-speed train ticket to go back Suzhou... So I bought bus ticket... Otherwise I cannot go back...

## References
- [XML —— Wikipedia](https://en.wikipedia.org/wiki/XML)
- [XML —— w3schools](https://www.w3schools.com/xml/xml_syntax.asp)
- [XML —— Runoob.com](http://www.runoob.com/xml/xml-summary.html)
- [RSS —— Wikipedia](https://en.wikipedia.org/wiki/RSS)
- [RSS —— Runoob.com](http://www.runoob.com/rss/rss-tutorial.html)
- [hexojs/hexo-generator-feed](https://github.com/hexojs/hexo-generator-feed)