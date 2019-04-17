---
title: 测试草稿文章
tags:
---

# 静态代理
代理模式的示例代码：http://www.cnblogs.com/puyangsky/p/6218925.html


# 动态代理

在讲JDK的动态代理方法之前，不妨先想想如果让你来实现一个可以任意类的任意方法的代理类，该怎么实现？有个很naive的做法，通过反射获得Class和Method，再调用该方法，并且实现一些代理的方法。我尝试了一下，很快就发现问题所在了。于是乎，还是使用JDK的动态代理接口吧。



java动态代理：http://www.cnblogs.com/jqyp/archive/2010/08/20/1805041.html