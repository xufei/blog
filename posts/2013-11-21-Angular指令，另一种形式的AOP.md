Angular指令，另一种形式的AOP
====

AOP并不是一个新名词，它存在的意义是：把那些与主流程相关性较小的东西独立出去，用挂接的方式放在合适的位置。在类似Spring等后端框架里，这种设计手段已经被广泛运用了，在前端领域，用得还不算多，但也有些框架用了，也有相关文章进行了探讨。

比如说：

- @司徒正美的这篇：http://www.cnblogs.com/rubylouvre/archive/2009/08/08/1541578.html
- 腾讯Alloy团队的这篇：http://www.alloyteam.com/2013/08/yong-aop-gai-shan-javascript-dai-ma/
- 还有阿里Arale框架中：http://aralejs.org/base/docs/aspect.html

前端JavaScript中的AOP主要是为了做一些日志、统计等功能，原理是把真实的函数进行了一个包装，上面几篇都写得很清楚了，不再赘述。

除了JavaScript中使用AOP，在作为纯展示的HTML中也能有这种思维方式的影子，最典型的就是CSS。

考虑这么一个场景，我们有一个已经做完的系统，界面很多，突然出于某种需求，想要让所有文本框中输入的东西都过滤某些特殊字符，并且要尽可能少修改代码，应当怎么办呢？

早期IE中，有一种技术称为behaviour，使用一个HTC（HTML Component）文件，通过CSS附加到某元素上，从而改变它的行为。通过这个东西，我们可以改变input的行为，在其中监听各种事件，作相应的处理。

然后通过这样的样式代码去改变行为：

    input {
        behavior: url(mask.htc)
    }

现在