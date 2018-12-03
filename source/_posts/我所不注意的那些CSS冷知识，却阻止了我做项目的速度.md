---
title: 我所不注意的那些CSS冷知识，却阻止了我做项目的速度
date: 2017-06-20 21:24:31
categories: '基础知识'
tags:
    - 伪元素
    - flex
    - css3
---
## 有序无序列表序列号不显示 ##
我们常常要遇到的是，不显示序列号（设置list-style-type:none），但有时因为项目过大，设置了一些通用属性后，想显示序号却怎么也捣腾不出来,关于list-style-type：none属性，具体去看[W3C标准][1]，下面罗列那些情况，会阻止序列号显示的问题；
1. list-style-type:none,规范标准，没啥讨论的；
1. 当li标签border与ul外边框距离小于1em时，也就是：
width =ul_margin   ul_border   ul_padding   li_margin <1em(为啥是1em，因为序号字体和文字字体大小设置相关)；
1. li标签设置了overflow属性且属性值不为visible;

结论：所以写这块时，要注意自己css初始化样式的代码如margin:0这样的代码；
## 块状元素默认宽度 ##
display属性为block的快照元素，width不一定为父元素宽带的100%

1. position属性为absolute或fixed；和其上一级元素position是否为非static无关，就是不继承其宽度；
1. 和第一条类似，当设置了float属性；
1. 当父元素display为flex布局，display为inline-block布局时，虽然父元素的宽度受子元素撑开，当其子元素的宽度还是为父元素宽度的100%；

另外：可以关注一下CSS3width的四个新特性：fill-available, max-content, min-content, 以及fit-content，[张大神的讲解][2]奉上
## inline-block间距问题 ##
学会inline-block这个属性后，float这个神奇的怪物，出镜率就越来越低,但偶然发现,当margin和padding都设置为0时，块与块之间还是存在间距（大于4px）；百度了一下，原来这里面学问还真不少；推荐一篇讲透了的[文章][3]

## flex伸缩项目的margin：auto(奇淫技巧) ##
当学会了flex时，inline-block的出镜率又越来越低，前端真的是无论你跑多快，你永远跟不上。闲话少扯，当我们用flex实现下列这种布局转换时,
![图片描述][4]
如果我们css这样写：

    .headbox{display:flex;justify-content:space-between;margin 0 1rem；}
上面未登陆时，由于是两个元素，所以子元素不需要做任何动作，就可以实现如图所示的布局，但是当登陆后，子元素又两个变为三个（菜单按钮、欢迎信息、注销按钮）,我们首先想到的是不是将后面两个再装到另一个box中，但是flex的伸缩项目被设置margin：auto时，就有意想不到的效果,其设置了为'auto' 的 margin 会合并剩余的空间。它可以用来把伸缩项目挤到其他位置；粘贴下面的源码可以自己感受一下：

    <style>
         .flex{display: flex;justify-content: space-between;}
         .right {margin-right: auto;}
    </style>
    <div class='flex'>
      <button class='right'>菜单</button>
       <button>登录</button>
       <button>注销</button>
    </div>

## 伪元素的灵活使用 ##
伪元素有很多：其中以hover、first-child、after、before这些最常用，用好了hover可以增强交互的感觉，而用好了before，after，则可以少些好多重复的代码，特别这些代码是动态生成的时候，比如，当我要生成这样一个list清单样式的时候，如下图
![图片描述][5]
这个清单有很多相同之处，其格式可以这样概括： 图标 - 标题 - ‘-’ - 日期 ,所以我们以标题为准，利用伪元素，为其插入图标和 ‘-’ ，这在写代码时，就减少很多重复工作； 但有一个要重点提及的，就是**当用伪元素插入一个背景图时**，有两点需要注意:
1：content：''，虽然只设背景，但这句话还得有；
2:设背景图片，最好不要直接在content：URL()这样设置，除非你图片切的刚好，但还是不推荐，还是该采用background设置，利于图片缩放，如：

    .jsimg::before {content:'';background: url('../img/js.png') no-repeat;background-size: 3rem;width:3rem;height:3rem;}
3: 设置背景图片时width一定要指定；

## 后续再补充 ##
  [1]: http://www.w3school.com.cn/cssref/pr_list-style-type.asp
  [2]: http://www.zhangxinxu.com/wordpress/2016/05/css3-width-max-contnet-min-content-fit-content/
  [3]: http://www.w3cplus.com/css/fighting-the-space-between-inline-block-elements
  [4]: https://sfault-image.b0.upaiyun.com/118/004/1180040012-59707e3a23bb0
  [5]: https://sfault-image.b0.upaiyun.com/413/935/4139356578-5970864c1e278