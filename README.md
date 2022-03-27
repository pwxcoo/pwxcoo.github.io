# Introduction
a personal [blog](http://blog.pwxcoo.com/) for me. 用的是 [hexo-theme-cactus](https://github.com/probberechts/hexo-theme-cactus) 主题。

# RECORD

## 20180304

_config.yml 中 `skip_render:` 添加 README.md。防止 README 被污染。

## 20180603

使用 mathjax 添加数学公式。

在 `themes/cactus/layout/_partial/scripts.ejs` 下

```javascript
<!-- mathjax -->
<script src="https://cdn.bootcss.com/mathjax/2.7.4/MathJax.js?config=default">
    MathJax.Hub.Config({
     tex2jax: {
     inlineMath: [['$','$'],["\\(","\\)"]],
     displayMath: [['\\[','\\]'], ['$$','$$']],
     },
  });
 </script>
```

## 20190508

fix Sina weibo image host temporarily in `/themes/cactus/layout/_partial/head.ejs`.

```
<meta name="referrer" content="no-referrer" />
```