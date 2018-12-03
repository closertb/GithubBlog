---
title: 一个javaWEB伪全栈的VUE入坑随笔，从零点零五学起
date: 2017-04-01 21:28:46
categories: '框架学习'
tags:
---
开始时间：3.26日

接触Vue，先在官网十目一行学完了[基本特性][1]，作为一个JAVA WEB的伪全栈，用Myclipse感受了一把双向绑定，感觉比JQUERY的JSRENDER要强悍不少，但这开发环境吧，不能写个.vue，就总觉得自己不能零距离接触VUE。网上各种百度，逛知乎，起初我是抵触的，当年笔记本装了一个VS STUDIO，C盘瞬间变红的陈年往事还历历在目，但为了学的专业一点，还是装了一个VS CODE，什么vetur，auto close tag插件一装，简直炫酷，关于安装了vs code,你还需添点啥，[推荐这一篇][2]。
   后面接触一周了，感觉VUE最吸引自己的是根据需要定义模块化HTML，即Vue.component，就像JSRENDER一样，但VUE组件化更解耦，也更加随意。HTML页面引用VUE.JS这种方式感觉显得不是那么现代化，所以又开始捣腾，接触NODE.JS,WEBPACK。所以，事又来了，安装NodeJs,开始接触纯前端的开发。

## 1、WINDOWS下安装配置[NODEJS推荐][3] ##
重点：
A：系统环境里添加安装目录node_modules文件夹地址

B:添加设置prefix和cache文件node_global和node_cache

## 2、WINDOWS下构建vue WEBPACK的开发环境 ##
[推荐查看][4]，前端小妹的博客，讲的都特别实在
重点：A、不要总是npm，在天朝，想干活，还是多用cnpm,不然就是等。。。。。
B、跟着小妹大侠新建第一个VUE项目时，让你选择是否配置ESLINT检查支持时，千万不要选YES，不然你后面学习VUE，保准用不了一个小时，你就印了那句话“从入门到放弃”，重要的事情说三遍，别选YES，别选YES，别选YES，跟着小妹学，是没有错的。
C：小妹谁提到了WEBPACK，但是也就提提，标题扣五分；
## 开始开发吧 ##
别墨迹。开发除了自己的第一个基于VUE的[前端单页面应用][5]（基本实现，还需改进）：
这里讲一个学以致用的知识点：
*列表选中样式的切换:*
JQUERY中我们通常采用:
      $('li[class='active']').removeClass('active'); //将当前选中的项目解除被选中的样式

      $(selector).addClass('active');//为选中的条目添加被选中的样式；

VUE中没有选择器，但对于CSS属性支持状态关联操作（Class 与 Style 绑定）：
    eg：v-bind:class='{ active: isActive }'

解读：
当isActive为真时，active样式有效，Dom渲染结果是：class=“active”
当为false时，active样式无效，Dom渲染结果是：class=“”
因此我们可以利用这个属性做文章
标签HTML：<li v-for:'tagName of tagNames'    v-bind:class={active:activeName==tagName} v-on:click='selected(tagName)'>
这条语句我们生成了一个列表，并为其绑定了一个选中事件，为class动态绑定了一个判断事件
同样我们在选择这个事件中：

        function selected(seclctedName){
         this.activeName= seclctedName;
        }
数据属性：

        data(){
            return{
                tagNames:[
                    {name:'hello',tabLink:'/Hello'},
                    {name:'Login',tabLink:'/Login'},
                    {name:'Nav',tabLink:'/Nav'}
                ],
                activeName:'hello'  //当activeName初始值为空时，浏览器加载时默认没有选择的列表项，当为hello时，hello列表默认被选中
            }
        },
4、感觉差不多了吧，外面的世界还有一项技能，你需要掌握，使用GITHUB管理你自己的代码用啥软件：[git使用方法][6]，一步一步，讲的真的不能更好。

5、下一步，写LOGIN


  [1]: http://cn.vuejs.org/v2/guide/
  [2]: http://www.cnblogs.com/zzsdream/p/6592429.html
  [3]: http://blog.csdn.net/xxmeng2012/article/details/51492149
  [4]: http://www.cnblogs.com/jiajia123/p/6132265.html
  [5]: https://github.com/closertb/myVue
  [6]: http://www.jianshu.com/p/7fa6b2d81f19