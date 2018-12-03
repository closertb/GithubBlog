---
title: 数据可视化，个人经验总结（Echarts相关）
date: 2018-03-05 22:26:17
tags:
     - 可视化
     - echarts
---
## 数据可视化 ##

![个人能力分析][1]
数据可视化旨在借助于图形化手段，清晰有效地传达与沟通信息（来源于bd）.在我们生活中最常见的，就有各种统计数据做成图表、股票k线图、能力雷达图这些（上面那张个人能力分析图，图片数据纯属虚构）；而对于前端开发者来说，就是用一些大神开发好的可视化图表组件将后端传过来的数据用一种直观，清晰的方式呈现在浏览器中，常用的可视化图表图库包括（排名不分先后），后面文章中都是围绕Echarts库的运用：
 - D3
 - Echarts
 - three.js
 - HighCharts
 - Charts
 - G2
## 色彩的应用 ##
色彩的应用作为数据可视化重要的部分，同样的数据，同样的图表类型，如果不同的人或者不同的公司做出来，有可能呈现的效果会截然不同，这其中的重要区别可能就是色彩的应用。Echarts不同的图表，都提供了一套默认的主题色，所以尽管我们不设置颜色，其呈现的效果也还是不错的。Echarts图表中可进行颜色设置的地方很多，包括但不仅限于下图（[官方demo][2]）所示的内容：

![clipboard.png][3]

![clipboard.png][4]

关于上面图提到的每个部分在option怎么设置，Echarts的配置手册都有详细描述，这里主要说一些工作中不常用到但又很关键的部分。从实现层面上来讲，颜色的设置分两种，option属性设置和css样式设置，至于为什么，可以从上图的dom结构得到答案，每个Echarts实例大体都包含两个元素：canvas 和 div（方框标注部分），div负责图表tooltip的展示(黄色框圈起部分)，而canvas负责除黄色框以外的所有部分。如果是简单的颜色设置，如上面的展示的那张标注图，option属性设置color就足够了，但如果要做出下图所示的强调色，option属性设置color就显得捉襟见肘了，在标题和tooltip的数据显示上，应用了混合色用以加强数据的表现：

![clipboard.png][5]

对于tooltip中的强调色，由于其根本是dom元素的操作，所以要做出上图所示的效果很简单，控制div元素及其子元素的样式就可以了，like：

          option = {
            tooltip: {
              trigger: 'axis',
              backgroundColor: 'rgba(0,0,0,.8)',
              textStyle: {
                color: '#b4d1e6'
              },
              /*formatter属性的应用，直接行内css样式操作*/
              formatter: function (val) {
                return val[0].name + ':<span style="color:#ffbf00;font-weight: bold;font-size:14px;padding: 0 5px;" >' + addSeparator(val[0].data) + '个</span>'
              }
            }
            ...其他设置
         }
而对于标题或则图表其他部分要进行混合色的设置，就不是那么简单了，因为其不接受tooltip那种dom元素的直接样式操作，但Echart还是留了足够多的入口来解决这样的需求：富文本标签（rich），[官方讲解][6]，比如上面那一段混合色的标题，可以这样来实现，代码拷贝到[官方demo][7]，即可查看效果,更多用法可查看[官方示例][8]：

    title:{
              show:true,
              left:'center',
              top:15,
              text:'{a|2017年全省应聘人员总数统计:}{b|165,338}{c|人}',
              textStyle :{
                rich: {
                  a: {
                    color: '#8bb8e8',
                    fontSize: 14,
                    fontWeight: 'bold'
                  },
                  b: {
                    color: '#ffcf2a',
                    fontSize: 16,
                    fontWeight: 'bold'
                  },
                  c: {
                    color: '#8bb8e8',
                    fontSize: 14,
                    fontWeight: 'bold'
                  }
                }

             }
    }
## 自动轮播（AutoToolTip）的应用 ##
Echarts中的normal与emphasis,以及tooltip的加入，通过hover与unhover状态的切换，让图表多了一些交互。特别是上面提到的tooltip的自定义样式，让展示效果提升了一个档次。但作为前端可视化，很多时候显示在一个大屏幕上，用于参观展示用，所以参观展示的人是不大可能用鼠标一个一个hover来查看具体的数据，这就要求我们需要用自动轮播来代替hover触发tooltip。为此，官方提供了[dispatchAction方法][9]与[官方demo][10]，也可参考[网上一篇文章][11]提供的思路和源码，封装一个图表通用的自动轮播工具，我自己也封装了一个，[欢迎参阅][12],下面是map使用轮播时的效果图。
![clipboard.png][13]
使用思路（Vue框架下使用）,首先将autoShowTip对象添加为Echarts的一个方法;然后在Echarts实例实例化之后，调用this.$echarts.AutoShowTip方法，并传入实例对象，option对象，轮播时间等参数：

![clipboard.png][29]
这里需要提醒两点：
 - Echarts的dispatchAction方法对常用图表中的Radar图还不支持，个人知道支持的图表类型有：pie,bar,line,map,scatter系列；为解决Radar图的单轴hover与自动轮播，自己写了个方法，并将其也封装到autoShowTip对象中，具体的实现可参考个人博客的文章：[从0开始撸一个支持单轴轮播的雷达图之末篇][14]；
 - toolTip使用时，其显示的位置也是大有学问，比如下图左边所示，会弄巧成拙，所以控制tooltip的位置也很重要，[Echarts也为此提供了相应的方法][15]，比如加入下面这段代码，就可以达到右图所示的效果：
              position: function (pos, params, dom, rect, size) {
                var obj = {top: '10%'}; //y轴方向，其位置固定
               // obj[['left', 'right'][+(pos[0] < size.viewSize[0] - 20)]] = 5;
                if (pos[0] > (size.viewSize[0] - 100)) {
                  obj['right'] = 0;
                } else {
                  obj['left'] = pos[0];
                }
                return obj;
              }

![clipboard.png][28]
 - 另外，自动轮播还可以更好的展示数据，当我们数据过长，而展示空间有限时，我们可以把数据切为两端甚至多段，通过自动轮播切换，这样就可以在有限的空间里，展现最好的效果，下面是我做的一个Demo，源码及效果图:
        myChart = this.$echarts.init(target);
        let step =0;
        option.xAxis.data = labelData.slice(0,length);
        option.series[0].data = realData.slice(0,length);
        option.series[1].data = symbolData.slice(0,length);
        myChart.setOption(option);
        this.$echarts.AutoShowTip(myChart, option, 3000,{
          refreshOption: function () {
            step = ((step + length)>labelData.length)?0:(step+length);
            let endStep = ((step + length)>labelData.length)?8:(step+length);
            option.xAxis.data = labelData.slice(step,endStep);
            option.series[0].data = realData.slice(step,endStep);
            option.series[1].data = symbolData.slice(step,endStep);
          },
          isRefresh: labelData.length>length });
![clipboard.png][27]
## 坐标系的优化 ##
在做bar或则line，或则基于这两者的扩展系列时，除了上面提到的，其实设置或数值（value）轴的刻度，也是一件需要注意的事情。比如下面两张图片这样，如果单看，感觉没啥，但如果在一个大屏里有多个这种柱状或曲线图，有些间隔线三四根，有些八九根，还有些没有均分，这种可视化展示，就会给人一种七零八落的感觉，根据个人经验，将间隔线控制在6根以下，体验较好。
![clipboard.png][26]
除了上面提到的这一种，还有就是坐标轴范围过大，数值过大，造成不美观及可读性差。比如下面这种，简直就是失败的炫富，除去y轴数字重叠的问题，还有就是根本无法一眼知道他总收入究竟在那个段位，只知道很多，需要专心的数，才能知道，哦，，，嗦嘎，572.67亿元，抢了他，立刻，马上，现在就去：

![clipboard.png][25]

上面分享了两种表达不友好的数据展示图表，那怎么才能更好的优化呢。下面我提供一下自己在做公司一个项目时的思路，关于具体实现可以自己摸索,但如果你们家的后端都为你计算好了单位，处理了前两步，那你就只需要做最后一步了。关于为什么不直接设置yaxis的min，max，spiktLine来控制间隔线及间隔，Echarts官方文档有这样的回答：[坐标轴的分割段数，需要注意的是这个分割段数只是个预估值，最后实际显示的段数会在这个基础上根据分割后坐标轴刻度显示的易读程度作调整][16]，关于操作interval来控制间隔，也有这样一句提示：[因为 splitNumber 是预估的值，实际根据策略计算出来的刻度可能无法达到想要的效果，这时候可以使用 interval 配合 min、max 强制设定刻度划分，一般不建议使用][17]：


![clipboard.png][24]


## 地图的应用 ##
Echarts地图，其可以设置为geo地图引擎或百度地图引擎，好像其他地图也支持，只要你知道坐标系的转换关系。geo地图由于部分数据不符合国家《测绘法》规定，目前暂时已经停止下载服务，不过你想找，还是能找到，比如Echarts github账号下。地图表面上在充当一个图表的背景，实际上其更多的作用是作为一个坐标系-经纬度坐标系。关于Echarts geo地图的使用，个人有几点经验分享一下：
- 不同版本的js或则json地图数据，呈现出来的效果差别很大，大到测试给你提bug的地步，比如下面两幅图所示，左边图的甘孜州名称已经把自己的爪牙完全伸到雅安去了，而绵阳则像已经吞并了德阳,自己对比了一下数据，左图的地图json数据是压缩过的，右图是未压缩的，但讲道理的话，应该是这两个json不是同一个版本：

![clipboard.png][23]
- geo的设置的必要性，前面说过，geo其实存在的重要的作用是作为图表坐标系。所以当你的series存在多个系列需要在同一个坐标系能设定数据，那设置geo是非常有必要的。但值得注意的是，一旦系列中设置了全局geo为参考坐标系，即指定了geoIndex,那么series-map.map 属性，以及 series-map.itemStyle 等样式配置不再起作用，而是采用 geo 中的相应属性；另外要强调的就是谨慎使用roam：true,最好设置一个缩放区间；
- 如果涉及到政府项目，对边界区域很敏感，则最好的选择就是使用Bmap作为地图坐标系；
## 自定义系列 ###
当时学了Echarts不久，看到[Gallery上面][18]一些炫酷的实例，自己也是想动手做了一个，可一看Echarts源码，一脸懵逼，后面又看了一下zRender及羡辙老师的水球图，感觉入了点门，但离做出炫酷效果还是有些差距。直到最近发现4.0版本介绍了自定义系列,捣鼓半天，自己做了个伪3d填充，2d坐标系的柱状图效果（加载动画化花了点时间），感兴趣的可以自己去研究一下：


![clipboard.png][22]


## 打个总结 ##
感觉这个经验分享帖，写的越来越像Echarts宣传贴了。掐指一算，自己做数据可视化已经快一年了，但感觉就像入了个门，作为前端的一个分支，水一样深。如果你对可视化还没有概念，推荐看一下蚂蚁金服去年推出的G2，其作者也是前Echarts的作者，里面讲了更多可视化图表理论性的东西，[G2可视化基础文档][19]。为了有一个直观的表现，自己用Vue搭了一个[可视化Demo][20],感兴趣的可以自己下载,如果感觉不错，请不要吝啬你的赞赏，我是个经得起表扬的人。
**顺便打个卖身广告：本人现在处于离职，如果哪位大神的公司需要找个搬砖的，请收下我的膝盖，[在线简历][21]，坐标成都。**
本文首发于：http://closertb.site ，转载请注明


  [1]:https://sfault-image.b0.upaiyun.com/998/578/998578698-5a9ac005b2d34_articlex
  [2]: http://echarts.baidu.com/examples/editor.html?c=bar-tick-align
  [3]:https://sfault-image.b0.upaiyun.com/128/216/1282165877-5a9b655ad6d61_articlex
  [4]:https://sfault-image.b0.upaiyun.com/941/399/941399198-5a9b67dc2269d_articlex
  [5]:https://sfault-image.b0.upaiyun.com/267/302/2673022103-5a9b6eab40960_articlex
  [6]: http://echarts.baidu.com/option.html#title.textStyle.rich
  [7]: http://echarts.baidu.com/examples/editor.html?c=bar-tick-align
  [8]: http://echarts.baidu.com/tutorial.html#%E5%AF%8C%E6%96%87%E6%9C%AC%E6%A0%87%E7%AD%BE
  [9]: http://echarts.baidu.com/api.html#echartsInstance.dispatchAction
  [10]: http://www.echartsjs.com/gallery/editor.html?c=doc-example/pie-highlight
  [11]: https://www.cnblogs.com/chengwb/p/5833454.html
  [12]: https://github.com/closertb/myAccumulation/tree/master/specialWidgets/EchartsRadarAutoTip
  [13]:https://sfault-image.b0.upaiyun.com/216/041/2160410295-5a9bbe9a2e9a0_articlex
  [14]: http://closertb.site/2017/09/%E4%BB%8E0%E5%BC%80%E5%A7%8B%E6%92%B8%E4%B8%80%E4%B8%AA%E6%94%AF%E6%8C%81%E5%8D%95%E8%BD%B4%E8%BD%AE%E6%92%AD%E7%9A%84%E9%9B%B7%E8%BE%BE%E5%9B%BE%E4%B9%8B%E6%9C%AA%E7%AF%87/
  [15]: http://echarts.baidu.com/option.html#tooltip.position
  [16]: http://echarts.baidu.com/option.html#xAxis.splitNumber
  [17]: http://echarts.baidu.com/option.html#xAxis.interval
  [18]: http://gallery.echartsjs.com/explore.html#sort=rank~timeframe=all~author=all
  [19]: http://antvis.github.io/vis/doc/chart/classify/compare.html
  [20]: https://github.com/closertb/simpleEchartsDemo
  [21]: http://closertb.site/resume
[22]:https://sfault-image.b0.upaiyun.com/495/242/495242313-5a9d5300140e7_articlex
[23]:https://sfault-image.b0.upaiyun.com/170/963/1709636159-5a9c01a5c95f7_articlex
[24]:https://sfault-image.b0.upaiyun.com/258/402/2584029376-5a9d423256102_articlex
[25]:https://sfault-image.b0.upaiyun.com/541/723/541723887-5a9d3b4ba8924_articlex
[26]:https://sfault-image.b0.upaiyun.com/251/806/2518066763-5a9d38f7a6138_articlex
[27]:https://sfault-image.b0.upaiyun.com/310/169/3101694142-5a9d35925128e_articlex
[28]:https://sfault-image.b0.upaiyun.com/227/685/227685420-5a9bdca053fe4_articlex
[29]:https://sfault-image.b0.upaiyun.com/143/977/1439776851-5a9bd386e54c7_articlex
