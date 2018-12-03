---
title: 基于iScroll的三级联动选择组件的实现引发的知识点整理（个人踩坑心得）
date: 2017-05-24 21:27:29
categories: '前端实践'
tags:
    - 三级联动
    - iScroll
    - jQuery插件
---
## 引子 ##
五月开始出来找工作，简历被一家创业公司相中，就毫无准备的去面试了。面试过程自然也是惨不忍睹，但面试官给机会，就给出了一道题，完成类似于携程用车预约页面的时间选择组件。离开时面试官提示：一、这个其实是一个组件，只要改变组件绑定的数据，能实现很多选择；二、只针对移动端做。自己拿着面试题，其实挺彷徨的，除了知道用自己最擅长的jQuery.fn来实现这个组件，其他的一无所知，后面不断查资料、捣腾，倒也做出来了，但最后和技术Leader深入聊时，问及这个插件CSS部分的优化，自己答的确实不咋地，所以被无情的拒绝，后面自己闲下来，再看那个CSS时，写的真的是狗猫儿不是，然后就再一次捣腾，学习到了很多。这里做以记录，以免自己再次入坑。

 - [组件演示地址][1]：
 - 组件用到的框架：jQuery.js,iScroll.js Scroll入门[Demo][2]
 - 关于实现：大概就是实例化三个iScroll，并为每个实例绑定的数据，如果数据间有父子关系，需要联动，则可以使用一个定时器（iScroll有相应的触发事件，但当时没时间仔细学，所以自己写的定时器来实现，iScroll内部也是通过定时器实现的），来检测父原件的值是否变化，从而改变子组件的绑定数据，看源码比较好。

## 1、	iScroll实例化 ##
滚动组件HTML主要代码：

     <div id='ui-scroll'>
      <div id='grandwrapper'>
       <ul></ul>
     </div>
      <div id='parentwrapper'>
       <ul></ul>
     </div>
      <div id='childwrapper'>
       <ul></ul>
     </div>
     </div>

CSS代码：

    #ui-scroll{position: absolute;overflow: hidden;background: #F8F8F8;width:94%; margin:10px 3%;border: 1px solid #E0E0E0;
    border-radius: 4px;height:120px;text-align: center;line-height: 40px;top:50px;left:0;}
    #ui-scroll div{display:inline-block;height:100%; width:32%;margin-left:1%;text-align: center;vertical-align:top;}
    #ui-scroll li{color: #898989;font-size: 1.6rem;}

JS代码：（这里用到的是iScroll 4****版本5和4实例化参数设置很多不一样）

     grand = new iScroll('grandwrapper',{snap:'li',vScrollbar:false,
      onScrollEnd:function () {
       indexgrand= (this.y/40)*(-1) (rows-1)/2;
     }});
    //标准实例化参数 target =new iScroll('targetId',options)（iScroll 4）
    //标准实例化参数 target =new iScroll('#targetId',options)（iScroll 5）
    //粗略看，4和5好像差不多，都是传入一个el和一个options，但传入el时，区别在#号，也许你猜到了，4获取Dom节点用的是getElementById(el),而5采用了querySelector(el)
    //options：设置snap：'li'，用于自动捕获目标点，vScrollbar设置为false用来设置是纵向滚动条不显示，onScrollEnd设置了一个事件监听，每一次滑动结束后，获取当前被选中的值的Index,40是CSS设置的line-height值，而(rows-1)/2是为了插件可以设置显示行数，所以需要offset这个空值。
## 2、iScroll实现滑动，但元素滑动后，会回弹 ##
原因：wrapper元素height设置不正确。（这也是自己给自己挖的最大的坑）

个人理解：iSroll实现滑动，其实就是在你的屏幕中，把你要滑动的区域实例化成屏幕中的屏幕，所以当你的内容不足以撑开你实例化的这块屏幕时，当然就没法滑动。和在浏览器的滑动条一样，当我们web的内容高度还没有屏幕高度高时，浏览器的滑动条是不会出现的。
所以顺着这个思路，自己查看了iScroll的相关源码：

    if (dir == 'h') {  //横向滑动
       that.hScrollbarSize = that.hScrollbarWrapper.clientWidth;
       that.hScrollbarIndicatorSize = m.max(mround(that.hScrollbarSize * that.hScrollbarSize / that.scrollerW), 8);
       that.hScrollbarIndicator.style.width = that.hScrollbarIndicatorSize   'px';
       that.hScrollbarMaxScroll = that.hScrollbarSize - that.hScrollbarIndicatorSize;
       that.hScrollbarProp = that.hScrollbarMaxScroll / that.maxScrollX;
    } else {   //纵向滑动
       that.vScrollbarSize = that.vScrollbarWrapper.clientHeight; //获取元素高度
       that.vScrollbarIndicatorSize = m.max(mround(that.vScrollbarSize * that.vScrollbarSize / that.scrollerH), 8);
       that.vScrollbarIndicator.style.height = that.vScrollbarIndicatorSize   'px';
       that.vScrollbarMaxScroll = that.vScrollbarSize - that.vScrollbarIndicatorSize;
       that.vScrollbarProp = that.vScrollbarMaxScroll / that.maxScrollY;
    }

然后又在devtools里$('#grandwrapper').css('height')打印了自己写的wrapper的高度920px，见鬼了，仔细一想920/40 =23,好像有点眉目了，自己写CSS时，没有限定其高度，所以高度为auto，随子元素个数变化而变化。所以马上，立马去CSS中设置了一个高度160px:其实该设置成120px，所以这又引出了另一个问题：
纵向虽然可以滑动选择了，但是没法选择最后一个元素，原因，原因就在于这个160px,实例化的滑动区域比可见区域高，所以滑动不能选择最后一个元素。
## 3、	更新绑定数据后，区域个别部分出现无法选中的现象 ##

原因：当更新数据后，虽然实例化的wrapper区域是一定的，但滑动内容是针对wrapper的子元素，子元素个数变化，就会引起高度变化，所以每一次更新数据后，都应执行一次refresh()方法更新滑动内容高度，这样才能保证每个选项都能被选中。
## 4、	CSS 元素高度设置 ##

     #ui-scroll{position: absolute;overflow: hidden;height:120px;line-height: 40px;top:50px;}
     #ui-scroll div{position:absolute; width:32%;}
首先块级元素，width默认为100%，但inline-block则不是，默认为auto
但块级元素,height默认为auto,而不是父级元素的100%。但如果这样写
    #ui-scroll div{position:absolute; width:32%;top: 0;bottom: 0;}
div高度是和ui-scroll相关的,divHeight = ui-scrollHeight-top-bottom（但有个前提，ui-scroll元素定位不能是static）。所以这里height设置等价于height：100%
## 5、	清除浮动 ##
最近面试，技术面基本都要问关于float的相关知识，以及怎么解决。当时和技术Leader聊这个组件时，自己也是实在了float这个问题上，大概意思是如果不用inline-block,就用浮动你怎么实现，自己叭叭叭说了一堆，后来想想，真是蠢，根本就没有get到题点。
关于浮动的定义，百度

怎么清除浮动，从思路上来讲，有两种方式：
A、clear:both

方法有很多种：

1. 在浮动元素后加空元素，<div 'style:clear:both'></div>
1. 伪元素 parent：after{}

B、BFC(块级格式上下文)
关于块级格式上下文的定义，推荐看[这一篇][3]，
其实就是通过设置一个属性来触发BFC，然后达到清除浮动的目的。
触发BFC方式：
1. 根元素
1. float属性不为none
1. position为absolute或fixed
1. display为inline-block, table-cell, table-caption, flex, inline-flex
1. overflow不为visible

## 6、待续，还有个别地方还需要进一步研究。 ##
[1]:https://closertb.github.io/Denzel/selectScroll/date.html
[2]:http://iscrolljs.com/
[3]:http://www.cnblogs.com/lhb25/p/inside-block-formatting-ontext.html