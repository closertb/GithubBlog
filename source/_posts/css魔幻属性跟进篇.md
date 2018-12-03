---
title: css魔幻属性跟进篇
date: 2018-03-26 17:32:56
categories: '基础知识'
tags:
    - css
    - float
---

白话：即上一篇我脑中飘来飘去的css魔幻属性自己的文章推出之后，这是自己写的第四篇CSS相关的文章，文章绝大部分是自己工作总结得来，另一部分是平日sf回答的与面试中向面试官交流学到的，都是一些比较基础，刨根问底的知识分享。
- [我脑中飘来飘去的css魔幻属性][1]
- [我所不注意的那些CSS冷知识，却阻止了我做项目的速度][2]
- [关于CSS样式动态注入，你不知道的那些冷知识][3]

## 清除浮动的原理 ##
在上一篇我脑中飘来飘去的css魔幻属性提到浮动不按想要的方式浮，里面提到清除浮动其实按原理来讲，就两个：
- clear：both(不准确，后面会讲)
- 触发BFC
前面因为没咋搞明白，就没有说为什么，最近因为偶然在sf上有人提问，就顺着这个问题去搜了相关资料，找了点答案。
### clear 清除浮动 ###
clear清除浮动的操作，基本思路是这样。首先为要清除浮动的盒子引入清除元素，现实表现里一般为一个空元素或者伪元素（before,after）。设置了clear属性后，盒子渲染时，会将这个元素的top border（上边缘）与浮动元素的底部对齐，来达到将盒子撑开的目的。但是这个与浮动元素底部对齐的元素与clear设置的属性（both,left,top）有关,具体可以看[W3C标准][4]。还是很简单易懂的。如果说的不是很明白，可以拷贝这段代码，试一下，然后切换clear的值，就会有种恍然大悟的感觉。 如果你觉得还不够直观，你可以将content的“”里写两个字，或则加个margin,border什么的。
** html 代码 **

        <body>
            <div class="float-left">
                这是左边浮动元素
            </div>
            <div class="clear-box">
                <div class="float-right">这是右侧浮动元素</div>
                <div>这是正常布局文档流元素。</div>
            </div>
        </body>

** css代码 **

        .float-left{
          float: left;
          width: 100px;
          height: 100px;
          background-color: lightpink;
        }
        .clear-box{
          margin-top: 50px; //这个没有用
          background-color: lightgreen;
          font-size: 16px;
        }
        .clear-box:after{
          content: '';
          display: block;
          clear: right; //both left
        }
        .float-right{
            width:100px;
            height:75px;
            float:left;
            background-color:red;
            border:1px solid black;
        }

![clipboard.png][10]

![clipboard.png][11]
### BFC 清除浮动 ###
说BFC清除浮动之前，得知道BFC的概念：
块格式化上下文（Block Formatting Context，BFC） 是Web页面的可视化CSS渲染的一部分，是布局过程中生成块级盒子的区域，也是**浮动元素**与其他元素的交互限定区域。其作用简单来讲就是，保证整个文档流中盒子与盒子之间的布局不相互干扰，这里其实已经很显示的说明BFC一大功效就是清除浮动，[触发BFC的条件详见MDN][5]。
至于BFC为什么可以清除浮动，就是形成BFC的盒子，其边框会去查询盒子里所有的正常布局文档流，和浮动的文档流，然后将盒子的底部边缘与盒子里最底部元素的盒子边缘对齐（这么讲会不会被警察关禁闭）。与clear区别的，这种清除浮动由于是盒子自己触发BFC，所以只能清除盒子里面的元素，而前面可以清除同一行所有左右两边的浮动。

![clipboard.png][12]

自此，清除浮动的原理就讲完了。但在这次参加面试前，一个问题自己也一直想搞懂，浮动是否脱离了文档流。MDN中是这样定义的：float CSS属性指定一个元素应沿其容器的左侧或右侧放置，允许文本和内联元素环绕它。该元素从网页的正常流动中移除，尽管仍然保持部分的流动性（与绝对定位相反）。这相反指的是什么，想知道。

**重点强调：**设置了float属性的元素，其display是什么属性。答案：block。inline元素设置为float后，其width和height都变成了可设置的属性。其原因就是设置float触发了BFC。是inline元素变成了一个块级盒子。其同样适用于设置position属性为绝对定位或固定定位的内联元素。

## 重新认识盒子之padding ##
学前端的，出去面试，浮动和盒模型是必问。这里也不再说正常盒模型（W3C）与怪异盒模型（IE）的区别，重点谈padding。
### 设置相对单位时，其参照值是谁 ###
首先是说说单位。一般来说设定盒子某一属性，有如下几种单位可以设置：
 - padding ：20%；
 - padding ：2em；
 - padding ：5px;(最常见的)
 - padding ：2vh；
 - padding ：2rem;
px,vh,vw,rem这些绝对单位很好理解，但如果是em和%,你是否足够留意，其是根据谁来作为参照来计算的。首先%，直接看代码和效果图：

         <div class="big">
             <div class="small">这是子元素</div>
         </div>


        .big{
          height:200px;
          width:1000px;
          background-color: yellowgreen;
        }
        .big .small{
          width:50%;
          height:50%;
          margin-top:5%;
          margin-left:5%;
          border-top: 5px solid red;
          border-left: 5px solid red;
          padding-top: 5%;
          padding-left: 5%;
          background-color: #0e8cf6;
        }


![clipboard.png][13]
![clipboard.png][14]

从上面的图可以看出，margin,padding无论是top还是left设置为5%都是50,算下来就是以父级元素的宽度来作为参照的。不要问为啥border不能设置百分比，我不知道，我也没这种需求。
再看一下以em为单位是以谁做参照，HTML与上面一致，上css代码：

    .big{
      height:200px;
      width:1000px;
      font-size: 14px;
      background-color: yellowgreen;
    }

    .big .small{
      width:50%;
      height:50%;
      margin-top:2em;
      margin-left:2em;
      border-top: 5px solid red;
      border-left: 5px solid red;
      padding-top: 2em;
      padding-left: 2em;
      font-size: 20px;
      background-color: #0e8cf6;
    }

![clipboard.png][15]

从计算样式盒子可以看出，margin,padding无论是top还是left设置为2em都是以元素自身的font-size来计算的，所以和百分比又不一样。

如果知道这些，UI需求是让你在一个盒子里画一个正方形盒子，你就很自然的会想到。用padding的百分比特性来做。
** 如果再多想一些，当我们用css3特性来做位移比如：transform：translate（50%，50%）,其又是相对谁来计算呢？这里直接给出答案：其参照值是元素本身的长和宽，和前面padding又不一样。**
### 盒子里面的绝对定位，其零点在哪里 ###
自己写了一年css,其实一直只关注了设置border-box与content-box盒子模型时padding表现的差别。但这个盒子的零点，及子元素固定定位相对的零点在哪里呢？还是先看代码和效果图：

    <div class="big">
        <div class="small">
            <!--<div class="normal">这是正常文档流子元素</div>-->
            <div class="position">这是绝对定位子元素</div>
        </div>
    </div>


    .small .normal{
      height:40px;
      background-color: #999;
    }
    .small .position{
      position: absolute;
      width: 90%;
      height:30px;
      //top:0;
      //left:0;
      background-color: white;
    }

![clipboard.png][16]
![clipboard.png][17]

上面四张图片，分别展示了绝对定位时，设置top，left与不设置的差别。不设置时，其文档流开始的起点是**正常文档流**的位置，而设置了top，left的地方，其起点是父元素（padding+content）区域零点的位置。以上效果和父元素设置不设置box-sizing: border-box属性无关，表现一致。

### 奇葩的内联元素padding ###
sf上面有这样[一个提问][6]：为什么设置display:inline后，padding-bottom仍然起作用？如果一般看过css基础知识的人，都知道内联元素设置margin-top、bottom,padding-top、bottom是不起作用的，所以日常开发，我们一般不会用这两个属性,要用时更多也是把内联元素转换为inline-box。重现问题：

      <div id="fu">
        <p>1505</p>
        <p>计科</p>
    </div>

    #fu{
      //  margin-top: 20px;
      //  background-color: yellowgreen;
    }
    #fu p{
      display: inline;
      margin: 20px;
      padding: 20px;
      border: 5px solid transparent;
      background-color: #0e8cf6;
    }
![clipboard.png][18]
上面是三幅图，分别代表三种状态。通过不断递进，我们就可以回答上面那个问题了。其实不是padding-bottom仍然起作用，准确来说是padding-bottom与padding-top都会起作用，只是起作用只是从表现上起作用，但并不占据文档流。怎么理解？第一：父元素黄绿色背景区域的高度和子元素内容高度一致，说明padding高度并没有被计算在内；第二：父元素没有加margin-top来占位时，padding-top那块区域是不可见的，所以内联元素padding是没有在正常文档流的。至于为什么，可以理解为内联元素没有盒模型，其高度由内容决定。由于其没有盒模型，所以没法控制padding-top和padding-bottom。
## 纵向上的margin:auto 用于垂直居中 ##
这一波面试，谈css的技术面试官，基本都会提怎么垂直水平居中。这确实是一个老生常谈的问题，以致于我越往后，回答的越含糊，如果你还不知道，可以看看[这篇文章][7]。基本就四种：table,flex,translate,定位加margin：auto。最后这一种很少人听说，但在居中盒子长宽值确定时，适用性确实很高。具体怎么操作呢：

     <div class="item">
            <div class="items-center">
                这是一个居中
            </div>
     </div>

    .item{
      width: 500px;
      height: 500px;
      position: relative;/*关键设置*/
      background-color: #999;
    }
    .item-center{
        position: absolute; /*关键设置，也可fixed*/
        top:0; /*关键设置*/
        bottom: 0;/*关键设置*/
        left:0;/*关键设置*/
        right:0;/*关键设置*/
        height:300px;/*关键设置，也可其他单位*/
        width: 300px;/*关键设置，也可其他单位*/
        margin: auto;/*关键设置*/
        background-color: yellowgreen;
        border: 5px solid darkgray;
    }

![clipboard.png][19]

上面的效果图，可以看到这种水平垂直居中方案也是666啊，前提是width,height必须显示设定，只兼容IE8+，其同样也适用于position：fixed的情况，具体视UI需求。我们通常只知道针对于块级元素，如果其定宽，可以使用margin：0 auto；来水平居中的，那这里又用auto实现了垂直居中，怎么实现的？本来想好好写的，可又看见我张老师已经做了一次剖析，自己只能仰望，[献上地址][8]。基本上从两个方面解释，能稍微解释同：
1：left,right,top,bottom设置为0，那么就说明item-center这个盒子，是会填满整个父级容器item的；
2：margin：auto 默认只会计算左右边距，所以上下如果设置为auto时默认是0；但对于脱离了正常文档流的定位元素，这个auto对于上下也是有效的，会自动均分左右两边的距离。所以这个盒子已经显示设定宽高，那么margin就会自动计算均分，达到居中的效果。

## 一些零碎的知识 ##
下面是一些零碎的经验分享，写出来共勉。
### 字体图标的使用 ###
![clipboard.png][20]

字体图标出现以后，其实精灵图的很多实用场景就被取代了，前端切图仔又可以好好安心写代码了。但使用字体图标图标还是有需要注意的地方，比如上方那张图，从正常到不正常（字体边框模糊），其实也就是font-weight设置的问题，由于font的继承性关系，所以很容易出现问题,所以字体图标样式初始化的时候将font-style与weight置为important还是很有必要的。

      .ued-components{
        font-family:"fe-components" !important;  //引用字体图标库
        font-size:16px;
        font-style:normal !important;  //设置字体样式
        font-weight:normal !important;  //设置字体加粗程度
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
      }

        <div class="ued-components send-img">&#xe6ae;</div>
### CSS动画丢帧 ###
![clipboard.png][23]

以上是一个用css3 animation 做的一个演示动画，如果仔细看，可以感觉到那种车速和档位不匹配的那种感觉，就是抖、抖、抖，看一下css代码的实现：

    .logo-animate {
        position: relative;
        line-height: 0;
        width: 240px;
        animation: move-float 8s linear 0s infinite;
    }

        @keyframes move-float {
            0% {
                margin-top:0;
            }
            50% {
                margin-top:-22px;
            }
            100% {
                margin-top:0;
            }
        }
在过往依赖jQuery的animate做图片轮播和列表轮播时，习惯于用margin来做位移。但是用纯css来做得时候，发现实现有明显的卡顿。后面一查看了[一篇文章][9]，发现css的animation实现最好依赖于transform来做，避免使用height,width,margin,padding等，具体原因在前面文章中有提到。所以代码优化一下，就是下面这样：

      @keyframes move-float {
        0% {
            transform: translate(95px, 0);
        }
        50% {
            transform: translate(95px, -22px);
        }
        100% {
            transform: translate(95px, 0);
        }
    }
### 字体溢出省略号的使用 ###

![clipboard.png][21]

如上图展示的那样，当我文字过多时，需要截断文字，使用省略号来保证正常的显示效果。用css的实现基本都是下面这段代码：

    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;

但是遇到table的情况，尽管你设置了td的width属性，但还是不起作用。这是因为table布局的流体属性，其会根据内容再重新分配空间，所以还需要加上一个table-layout：fixed 这样的属性设置。其实除了单行可以用css做文字截取，多行也可以，只是在兼容性上和效果上还不足以在线上环境来使用。但是实现思路还是可以看一看：

    display: -webkit-box;
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 2;
    overflow: hidden;

![clipboard.png][22]

因使用了WebKit的CSS扩展属性，该方法适用于WebKit浏览器及移动端，但是要想做到兼容及显示效果完美，还是用css配合js来做，单行css来做已足够，但记得设置title属性，保证hover能读到完整的信息。
作为一个写CSS不到两年的前端，在工作中吃了很多基础不扎实的亏。学CSS也不如JS那样简单，知识成体系，所以除了看完一本CSS基础知识的书以外，更多的还是写、写、写，然后思考，尝试用不同的思路来解决。


  [1]: http://closertb.site/2017/12/%E6%88%91%E8%84%91%E4%B8%AD%E9%A3%98%E6%9D%A5%E9%A3%98%E5%8E%BB%E7%9A%84css%E9%AD%94%E5%B9%BB%E5%B1%9E%E6%80%A7/
  [2]: http://closertb.site/2017/06/%E6%88%91%E6%89%80%E4%B8%8D%E6%B3%A8%E6%84%8F%E7%9A%84%E9%82%A3%E4%BA%9BCSS%E5%86%B7%E7%9F%A5%E8%AF%86%EF%BC%8C%E5%8D%B4%E9%98%BB%E6%AD%A2%E4%BA%86%E6%88%91%E5%81%9A%E9%A1%B9%E7%9B%AE%E7%9A%84%E9%80%9F%E5%BA%A6/
  [3]: http://closertb.site/2017/11/%E5%85%B3%E4%BA%8ECSS%E6%A0%B7%E5%BC%8F%E5%8A%A8%E6%80%81%E6%B3%A8%E5%85%A5%EF%BC%8C%E4%BD%A0%E4%B8%8D%E7%9F%A5%E9%81%93%E7%9A%84%E9%82%A3%E4%BA%9B%E5%86%B7%E7%9F%A5%E8%AF%86/
  [4]: https://www.w3.org/wiki/CSS/Properties/clear
  [5]: https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Block_formatting_context
  [6]: https://segmentfault.com/q/1010000013651675/a-1020000013652427
  [7]: https://juejin.im/post/5854e137128fe100698e6271
  [8]: http://www.zhangxinxu.com/wordpress/2013/11/margin-auto-absolute-%E7%BB%9D%E5%AF%B9%E5%AE%9A%E4%BD%8D-%E6%B0%B4%E5%B9%B3%E5%9E%82%E7%9B%B4%E5%B1%85%E4%B8%AD/
  [9]: https://segmentfault.com/a/1190000006708777
  [10]: https://sfault-image.b0.upaiyun.com/745/978/745978850-5ab21c2f0fd8b_articlex
  [11]: https://sfault-image.b0.upaiyun.com/405/395/4053951900-5ab21b52a20b0_articlex
  [12]: https://sfault-image.b0.upaiyun.com/378/420/3784209841-5ab2226976a34_articlex
  [13]:https://sfault-image.b0.upaiyun.com/365/037/3650372559-5ab26c5fe6623_articlex
  [14]: https://sfault-image.b0.upaiyun.com/244/847/2448479491-5ab26bab3e122_articlex
  [15]: https://sfault-image.b0.upaiyun.com/137/590/1375908612-5ab26dc5dc8a0_articlex
  [16]: https://sfault-image.b0.upaiyun.com/140/836/1408365083-5ab2fb1de8ad6_articlex
  [17]: https://sfault-image.b0.upaiyun.com/305/913/3059133048-5ab2fbfff29ad_articlex
  [18]: https://sfault-image.b0.upaiyun.com/263/892/2638925197-5ab3080339029_articlex
  [19]: https://sfault-image.b0.upaiyun.com/205/609/2056099390-5ab31bad7ad0d_articlex
  [20]: https://sfault-image.b0.upaiyun.com/957/959/95795980-5ab856000afa8_articlex
  [21]: https://sfault-image.b0.upaiyun.com/357/526/3575264451-5ab8698404a29_articlex
  [22]:https://sfault-image.b0.upaiyun.com/200/787/2007875320-5ab8712920854_articlex
  [23]:https://sfault-image.b0.upaiyun.com/647/415/647415641-5ab85a4e4bc75_articlex
