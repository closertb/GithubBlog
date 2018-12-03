---
title: 我脑中飘来飘去的css魔幻属性
date: 2017-12-13 23:42:19
tags:
    - bfc
    - 浮动
---
最近看到一篇20 个CSS高级技巧汇总的汇总，感触很深，不过我想，与技巧相比，有些常见css布局难题，有时候更加让我们的日常开发变得踌躇沮丧吧。
在写这一篇文章之前，自己还写过一篇：[我所不注意的那些CSS冷知识，但却阻止了我做项目的速度][1],如果你看了，我相信你也会受益的。  

### 为什么此处li标签内的p元素看起来独自撑开了一行 ###
这是我在segmentfault上看到的一个问题，以前自己遇到过，所以就很热情洋溢的去回答了一下，难道遇到个自己会的，示例代码是这样的：    
CSS:  

    li{
        display: inline-block;
        text-align: center;
    }
    .left,.center,.right{
        width:300px;
        height:300px;
    }
    .left{
        background-color: #999;
    }
    .center{
        background-color: #ccc;
    }
    .right{
        background-color: #eee;
    }  
    HTML:
    <ul>
        <li class="left">
            <p style="display: inline-block;">1</p>  
        </li>
        <li class="center"></li>
        <li class="right"></li>
    </ul>  
    
![测试图片][2]
 大概就是这样子，其实文和图有点不对应，代码中第一个模块他只写了一个“1”,我为了现象更加明显，且好说明为什么，就打了一大段文字，现在我们来说说为什么。先来一张图，看懂vertical-align的几个属性,顺便带上[图片出处][3]，文章讲得还可以，理解这张图片，后面就好理解了。
 
![vertical-align属性][4]
 
 inline-block的vertical-align 属性默认是baseline对齐（深入理解的[送福利][5]），也就是英文文字小写字母a,b,c这类字母底部的那条线，因为这些是外国人发明的，所以以英文字母才有针对性。inline-block拥有vertical-align属性，其默认是基线对齐的，所以这三个inline-box需要基线对齐，而其基准线就是正常流中最后一个line box的基线,如果这个元素是空的，没有内容，那么这个基线就是最后这个元素的margin-bottom线；如果这个元素不为空，那么这个元素的基线就是元素里面内容最后一行文字的基线；所以我们一个一个来套，发现这三个li元素在一行，第一个有文字，其基线为文字底部；最后一个没有文字，***其基线为margin-bottom线,考试要考，划重点，可以自己为元素设置margin-bottom试试***,这就会造成第一个和二，三个错行的感觉，其实他两是为了基线对齐，所以多敲几十个文字就能明显看出其差别。所以最简单的解决方案就是为li添加vertical-align: 属性不为baseline，气不气，改变其纵向的对齐方式的默认属性;为啥非弄个折腾人勒。关于vertical-align,如果还想做这方面的深入了解，[可以看看张大侠的分析][6]
 
### img图片撑不满整个div，有空隙 ###
直接上图更直观(箭头所指)：
![撑不满的div][7]
相关css和html:  

    <style>
        body,div{margin: 0;padding: 0;}
        .test{
            background-color: yellowgreen;
        }
        img{
            width:260px;
            height:260px;
        }
    </style>
    <body>
    <div class="test">
        <img width="130" height="130" src="https://user-gold-cdn.xitu.io/2017/12/10/160409cc0f090c6f">
    </div>
</body>
其实这个问题，如果你单单这样看，和我一样涉世未深的话，是一眼看不出答案的，但是如果你在图片后面多敲两个文字，你就会发现，和上个问题，这又是一个有关于vertical-align属性相关的问题。  

    <div class="test">
        <img width="130" height="130" src="https://user-gold-cdn.xitu.io/2017/12/10/160409cc0f090c6f"><span>abcd看文字</span>
    </div>  
让人恍然大悟的效果图：
![恍然大悟的效果图][8]   
这下你应该就懂了，下面的空隙的距离实际上等与1个line-height的底边与baseline之间的间距。仔细观察，图片的底边是和a的下边缘是在一条水平线上的，而不是和‘看’字下边缘一条水平线上的。所以为什么上面说这又是一个和vertical-align属性相关的问题。先说解决方案
**针对于父元素div：**  
- 设置行高足够小,比如.test{line-height:0}，至于这么小吗，其实高度小于top线和baseline线之间的距离的距离就行了，至于到底多小，这和font-size是相关的，其目的就是没有多余的高度拿来给baseline下面的空间用(个人理解);
- 上面说了设置line-height最小和font-size相关，所以，还有的方法，就是直接设置字体大小为0，.test{font-size:0;}，道理你应该懂；  
 
**针对于图片div：** 
- 上面说了这是一个和vertical-align属性相关的问题，所以设置vertical-align属性不为baseline也可以解决，比如img{vertical-align:top;}，当然也可以是数字，比如img{vertical-align:-10px;}，这个数值绝对不是正值，其数值应该是大于bottom线和line-height的底边距离的；
- 最后一种，就是vertical-align是一个对块状元素无效的属性，仅针对于内联元素有效的，当然inline-block也有效.所以img{display:block;}也可以解决问题。
也许到这里，你和我一样，有疑惑，为什么vertical-align是一个对块状元素无效的属性，设置img为块级元素，其和div就可以完美在一起，而一个内联元素放在块状元素里，就非得有隔阂。开始，我也是有这个疑问的，个人理解就是块状元素里面装了一个内联元素，如果块状没有显示的设置高度，其高度是由里面的最高的lineboxes组成的，这个div其实就是有两个lineboxes组成，图片linebox和<span>，其实还有一个linebox就是div自身的innerText('')，这不过这里内容为空，如果你把span去掉，你就更能理解这个隐身的linebox，所以就像是两个内联元素在一起，需要baseline对齐。所以网上有人说设置img{font-size:0;},是非常错误的,img元素很特殊，他不但是内联元素，他还是一个置换元素（下面会讲），它的高度不是文字内容撑开的，是其置换的图片高度撑开的，所以设置font-size是无效的。
### 浮动不按想要的方式浮 ###
![浮动常出现的形式][9]
像上图那样的形式，盒子由导航栏和右侧一个搜索框或者登录名什么的一起组成，这也是我们常用浮动的方式来解决这样的布局。  
说浮动前，先说三点概念：  
1.浮动最初出现的意义是为了解决文字环绕图片这种在杂志报纸中常会出现的布局样式； （看下图）
2.浮动与绝对定位能实现相同的效果，但的区别是，浮动未脱离正常文档流，但绝对定位脱离了正常文档流；  
3.浮动能带来灵活的布局，但同时也带来了父元素高度塌陷的缺点（看下图），所以清除浮动是使用浮动前的必修课，后面会说到； 
![文字环绕][10]
![高度塌陷带来的布局混乱][11]
现在看一下高度塌陷相关的代码：  
  
      
        <div class="test">
        <img width="130" height="130" src="https://user-gold-cdn.xitu.io/2017/12/10/160409cc0f090c6f">
        1.浮动最初出现的意义是为了解决文字环绕图片这种在杂志报纸中常会出现的布局样式；<br>
        2.浮动与绝对定位能实现相同的效果，但的区别是，浮动未脱离正常文档流，但绝对定位脱离了正常文档流；<br>
        3.浮动能带来灵活的布局，但同时也带来了父元素高度塌陷的缺点，所以清除浮动是使用浮动前的必修课，后面会说到；<br>
        <br>
        </div>
        <div class="blank"></div>
        <div>
        <div class="box">
            <span class="dot"></span>
            我是下面一个div的文字。
        </div>
            <div class="blank"></div>
        <div class="box">
            <span class="dot"></span>
            我是再下面一个div的文字。。
        </div>
            <input  width="260" value="输入一段文字"/>
        </div>
          
        .test {
        background-color: yellowgreen;
        font-size: 18px;
        vertical-align: top;
        }
        .test span {
            background-color: bisque;
        }
        .blank {
            line-height: 20px;
            height: 20px;
        }
        img {
            width: 260px;
            height: 260px;
            float: left;
        }
        input {
            border: 1px solid red;
            height: 24px;
            margin-left: 30px;
        }
        .box {
            background: black;
            color: white;
            padding-left: 20px;
            line-height: 10px;
        }
        .box .dot {
            display: inline-block;
            width: 4px;
            height: 4px;
            background: white;
            vertical-align: bottom;
        }
图片一中，实现了文字环绕图片那种想要的效果，并且后面的元素没有上移错位,其原因是上面说过的，如果块状元素没有显示的设置高度，其高度由其元素内的最高的linebox决定,所以第一张图片div的高度是比img高度高的，因为我重要的事情说了三遍，文字够多。而第二张图片，div高度只有144px，因为img是浮动的，他的linebox被浮动属性破坏了,而文字又不够多，所以就造成了所谓的高度塌陷，致使最后两个div陷进了图片所在的div中，要知道，这种情况在正常块状元素布局中是根本不会出现的。至于解决浮动引起的高度塌陷，我总结了两条，分别是：  
1. 使用clear:both，常见的什么clearfix；  
2. 触发浮动元素父元素的BFC（块状格式上下文，为解决盒子与盒子之间，内容不相符影响而生的概念）；
清除浮动，相信大家都懂，而触发bfc。  

我说说我常用的几条，网上讲bfc的很多：  
- float属性不为none的元素  
- position（absolute，fixed）  
- display (table-cell,inline-block，flex等)
- overflow属性不为visible
除了上面讲的这些，我还遇到过有人问，为什么我用了浮动，但元素没有浮在这一行，却换了行，像下图这样
![浮动不按想要的方式浮][12]  

       <div>
        <div class="gr">我是导航栏的一些文字</div>
        <div class="fr">我想浮在右边</div>
       </div>
       .gr{
          background-color: yellowgreen;
          margin:5px;
        }
        .fr{
          float:right;
          background-color: green;
        }
上面这种没按想要的方式浮，是因为块状元素会不敢其内容长度有没有一行的长度，其都会占据一行的长度，后面的元素会自动换行。解决这个其最简单的方式就是将fr元素放在gr元素前，为什么这样就可以，因为float破坏了div元素的块状属性，但其未撑开父元素的高度，其浮动属性为right，默认从右侧开始布局，所以后面的div仍按正常的文档流从最左端开始布局。
### 有一种行内元素，又叫置换元素 ###
如果你看上面一题代码的时足够细心，你会发现我给img设置了width和height两个属性值为130，但由于又在css属性里定义了宽高260,但最终表现出的宽高为260。如果css不定义宽高呢?答案是多少,要不你试试，你慢慢试，我还是先公布答案：130.这里我们将会说一个css中的一个鲜为人知的术语:**置换元素**，那什么又是置换元素呢？  

置换元素是指：浏览器根据元素的标签和属性，来决定元素的具体显示内容。  

例如：浏览器根据<img>标签的src属性显示图片。input元素根据标签的type属性决定显示输入框还是按钮。还有，还有近来很火的canvas。  

置换元素有如下共同点：  
1. 置换元素一般内置宽高属性，因此可以设置其宽高；  
2. 置换元素与一般的行内元素相比，其可以设置margin,padding，height,width等css属性；  

感觉要写的还有很多，事件根本不够用，先睡了，未完待续  
如果文中有任何不足和错误之处，还请及时指正。



  [1]:http://closertb.site/2017/06/%E6%88%91%E6%89%80%E4%B8%8D%E6%B3%A8%E6%84%8F%E7%9A%84%E9%82%A3%E4%BA%9BCSS%E5%86%B7%E7%9F%A5%E8%AF%86%EF%BC%8C%E5%8D%B4%E9%98%BB%E6%AD%A2%E4%BA%86%E6%88%91%E5%81%9A%E9%A1%B9%E7%9B%AE%E7%9A%84%E9%80%9F%E5%BA%A6/
  [2]:https://user-gold-cdn.xitu.io/2017/12/5/160271d2448ebb35?w=800&h=466&f=png&s=20165
  [3]:https://www.cnblogs.com/QingFlye/p/3876191.html
  [4]:https://user-gold-cdn.xitu.io/2017/12/12/1604b330d0484d10?w=725&h=406&f=png&s=66415
  [5]:http://blog.csdn.net/q121516340/article/details/51483439
  [6]:http://www.zhangxinxu.com/wordpress/2010/05/%E6%88%91%E5%AF%B9css-vertical-align%E7%9A%84%E4%B8%80%E4%BA%9B%E7%90%86%E8%A7%A3%E4%B8%8E%E8%AE%A4%E8%AF%86%EF%BC%88%E4%B8%80%EF%BC%89/
  [7]:https://user-gold-cdn.xitu.io/2017/12/10/160409aa3e570aaf?w=509&h=301&f=png&s=26797
  [8]:https://user-gold-cdn.xitu.io/2017/12/12/1604b3f6e27e60cd?w=399&h=276&f=png&s=23720
  [9]:https://user-gold-cdn.xitu.io/2017/11/28/16002fe74ac2a178?w=1346&h=72&f=png&s=14689
  [10]: https://user-gold-cdn.xitu.io/2017/12/13/1605056f47c7b942?w=800&h=214&f=png&s=180473
  [11]:https://user-gold-cdn.xitu.io/2017/12/13/16050574a293142c?w=800&h=240&f=png&s=96387
  [12]:https://user-gold-cdn.xitu.io/2017/12/13/160507a1e9a84f9d?w=764&h=79&f=png&s=2303
