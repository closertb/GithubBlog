---
title: 关于CSS样式动态注入，你不知道的那些冷知识
date: 2017-11-07 23:29:37
categories: '基础知识'
tags:
    - 动态加载
    - 样式注入
    - css
---

## 前言 ##
作为一个前端，我们都听过结构，样式，行为分离；关于样式，我们都听过外联样式，内联样式和行内样式；关于这三者，什么权重啊，啊，对了，这些都不会出现在这篇文章里，这篇文章就说一些那些我们不怎么使用的，动态引入css样式的方法；

## 静态样式引入 ##
前面说过外联样式，内联样式和行内样式，所谓外联样式，即样式文件是一个单独的css文件，通过link标签引入；而内联样式，是一种存在于html文件中，但与页面结构元素分离的，他们都是以存在于style标签中；而行内样式，即存在于某一个标签中，他们只对当前元素有效；说那么多，一张图胜过千言万语；
![样式引入][1]
无图说鬼话，有图说人话。是不是一下全看懂了，快夸我。样式引入方式的不同，也注定了他们作用的范围不同，外联能作用域多个html文件的**多个**htm页面的**多个**dom节点，两个多个；内联只能作用于**单个**html页面的**多个**dom节点；而行内嘛,就没多个了,就只能作用单个页面的样式属性所在的dom节点。
## 动态态样式引入 ##
其实，HTML文件静态样式引入，只要是一个前端，应该都明白，所以这篇文章，重点是要说动态样式的引入，说一些不常见当可能很适用的方法；  
### 行内样式 ###
看下面一段代码：  

        var triangle = document.createElement('label');
        triangle.style.width = '0';
        triangle.style.height= '0';
        triangle.style.position='absolute';
        triangle.style.left ='50%';
        triangle.style.top ='99%';
        triangle.style.marginLeft = '-5px';
        triangle.style.borderLeft = '5px solid transparent';
        triangle.style.borderRight = '5px solid transparent';
        triangle.style.borderTop= '5px solid white';
        triangle.style.borderTopColor = style.backgroundColor;
        label.appendChild(triangle);
这样的写法应该很常见吧，创建一个元素（当然你也可以获取一个元素），然后使用js代码为其动态添加样式,有可能你会问，这属性一个一个写，为啥不能直接对象，比如下面这样;  

        triangle.style ={
            width:'0',
            height:'0',
            position:'absolute'
        }
**注意哈，不行哈，这是绝对不行的，重要的事情重点标注**，那如果我想以对象的方式为元素添加样式呢？有，方法还不止一种（[操作HTML的样式类属性方法][2]）：

 1. triangle.style ="width:0;height:0;position:absolute;"(不推荐)
 2. triangle.style.cssText ="width:0;height:0;position:absolute;"(推荐)
 3. 首先将上面的样式属性事先写在一个样式class里，比如
    .triangle{width:0;height:0;position:absolute;},然后在js操作中，只需一句triangle.classList.add('.triangle'),动态为元素添加一个样式类
    （极力推荐）  
   
**这里说一个重点，易错点,使用dom.style为元素设置其浮动样式时，不可用dom.style.float = 'left'，为什么，因为float在css中是关键词，要设置其浮动属性，非IE浏览器得使用cssFloat（），而IE使用styleFloat,我走过的坑，但愿你不要再跳下去；**  
### 内联样式 ###
虽然上面我们极力推荐第3种来添加类样式为元素添加样式，但在一些插件的引入的时候，我们在引入其js的时候，还得相应的引入其css，比如下面这样：  
![图片描述][3]
是不是觉得有点烦，我个人写插件比较喜欢别人使用时，只需要一个文件就达到目的，而无需多在页面增加一次请求，所以这怎么做呢？
那就是样式的动态引入，如果你所写的插件只涉及到少许的样式操作，像我写的解决Echarts单轴雷达轮播那个插件，那用上面提到的直接操作行内样式就够了；但是如果涉及到大片的样式和插件样式动态变换，那么还是引入样式类比较简便，与上面截图不一样的是，我们是将样式写在插件的JS中，然后插件被调用时，动态注入我们的样式类，具体操作如下:
![图片描述][4]
仔细看看，可以发现，sytleStr其实就是我们通常css文件中定义的那些样式字符串，然后动态创建了一个sytle标签（设置其type很重要），并将样式字符串通过字符串节点的形式注入到标签中，最后将这个标签添加到被引用js所关联的html文件head头部，所形成的效果就是下面这样:
![图片描述][5]
这样写的好处就是，别人在使用你的插件时，无需多去引用你的css文件，这样看起来比较简洁，当然有些利弊也需要你权衡，比如维护你插件样式时，同直接在css样式文件中修改，这样的形式会显得稍微麻烦一些；
### 动态样式 ###
其实与上面的内联样式动态引入相比，外联样式的动态引入，相信被更多的人熟知。具体步骤就是，创建link标签，设置type属性，设置其href,然后添加到html文件当中；像下面这样：
![图片描述][6]

![图片描述][7]
可以看到html文件中有一个id为dynamicCreation的Link标签，而其关联的就是我们想为其添加的css文件。

以上三种动态样式注入，不同的使用场景，各有利弊，至于你想用哪一种，需要你自己权衡，睡觉去啦。。。。

  [1]: https://sfault-image.b0.upaiyun.com/117/327/1173275322-59ff2ab4b8df3_articlex
  [2]: http://www.runoob.com/jsref/prop-element-classlist.html
  [3]: https://sfault-image.b0.upaiyun.com/419/879/4198796031-5a01c5494d3d1_articlex
  [4]: https://sfault-image.b0.upaiyun.com/233/953/2339537391-5a01c95230ac4_articlex
  [5]: https://sfault-image.b0.upaiyun.com/824/267/824267981-5a01ca5d63628_articlex
  [6]: https://sfault-image.b0.upaiyun.com/305/744/3057443407-5a01cde22c94e_articlex
  [7]: https://sfault-image.b0.upaiyun.com/229/092/2290920069-5a01cdef58ab9_articlex