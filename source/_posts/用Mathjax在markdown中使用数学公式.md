---
title: 用Mathjax在markdown中使用数学公式
date: 2018-06-03 18:11:17
categories: 备忘
tags:
- markdown
---

<script src="https://cdn.bootcss.com/mathjax/2.7.4/MathJax.js?config=default">
   MathJax.Hub.Config({
    tex2jax: {
    inlineMath: [['$','$'],["\\(","\\)"]],
    displayMath: [['\\[','\\]'], ['$$','$$']],
    },
 });
</script>

用 [Mathjax](https://docs.mathjax.org/en/latest/start.html) 来显示数学公式的。其实有一个 [hexo-math](https://github.com/hexojs/hexo-math) 的插件，是集成了 Mathjax 的，但是不知道为什么我这里安装总是出问题。。下次再试试好了。。所以我直接引用 Mathjax 了。

## Import

直接用 cdn 来导入 Mathjax。

```html
<script src="https://cdn.bootcss.com/mathjax/2.7.4/MathJax.js?config=default">
   MathJax.Hub.Config({
    tex2jax: {
    inlineMath: [['$','$'],["\\(","\\)"]],
    displayMath: [['\\[','\\]'], ['$$','$$']],
    },
 });
</script>
```

inlineMath 表示内置的数学表达式， displayMath 表示行间的数学表达式。

## inline math

行内置的数学表达式 $J(\theta_0, \theta_1) = \frac{1}{2m}\sum_{i=1}^n (h_\theta(x_i)-y_i)$ 。写法如下：

```
$J(\theta_0, \theta_1) = \frac{1}{2m}\sum_{i=1}^n (h_\theta(x_i)-y_i)^2$
```

## display math

行间的数学表达式。

$$\theta_j := \theta_j - \alpha\frac{\partial}{\partial\theta_j}J(\theta_0,\theta_1)$$

写法如下：
```
$$\theta_j := \theta_j - \alpha\frac{\partial}{\partial\theta_j}J(\theta_0,\theta_1)$$
```

## References
- [MathJax basic tutorial and quick reference](https://math.meta.stackexchange.com/questions/5020/mathjax-basic-tutorial-and-quick-reference/5044)
- [MathJax基本的使用方式](https://blog.csdn.net/u010945683/article/details/46757757)
- [Hexo下mathjax的转义问题](https://segmentfault.com/a/1190000007261752)
- [Getting Started —— Mathjax](https://docs.mathjax.org/en/latest/start.html)




