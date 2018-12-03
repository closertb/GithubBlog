---
layout:
title: 关于vue导航列表样式切换（router-link-active）
date: 2017-04-17 21:26:44
categories: '框架学习'
tags:
---
![导航样式图][1]
当我们新建一个网站时，总是要做一个导航列表，在传统的WEB开发中这已经是一种很成熟的技术，自己学VUE，看了官方文档，加上自己摸索，也学到不少，现在拿来分享一下。在自己VUE入门学习的笔记中也有提及
第一种：JQUERY中我们通常采用:

    $('li[class='active']').removeClass('active'); //将当前选中的项目解除被选中的样式；

    $(selector).addClass('active');//为选中的条目添加被选中的样式；
非常简便，需要npm install jquery，并在baseConfig中配置。但学VUE，还是用其本身的Class 与 Style 绑定最好。
第二种：VUE中没有选择器，但对于CSS属性支持状态关联操作（Class 与 Style 绑定）：

    eg：v-bind:class='{ active: isActive }'

解读：当isActive值为真时，active样式有效，Dom渲染结果是：class=“active”
当为false时，active样式无效，Dom渲染结果是：class=“”
因此我们可以利用这个属性做文章
标签HTML：

      <li v-for:'tagName of tagNames' v-bind:class={active:activeName==tagName} v-on:click='selected(tagName)'>

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
                {name:'MyBlog',tabLink:'/MyBlog'}
            ],
            activeName:'hello' //当activeName初始值为空时，浏览器加载时默认没有选择的列表项，当为hello时，hello列表默认被选中
        }
    },
初看运行起来还可以，切换也正常，但当我们停留在非HELLO页面时，刷新页面，hello被选中了，而刷新前的选中样式被解除了，这是因为我们为activeName:'hello' 赋了初值。所以有BUG。
第三种：思路同二，但activeName,我新建导航样式列表组件时，我为其定义了一个TITLE属性


    props: {
      title: {
          type: String,
           default: 'any'
            }
      }


并在列表中使用:class='{active:title== tabbarName.name}来绑定active CSS，
当其他页面调用这个组件时，指定TITLE，比如：

    <v-header title='MyBlog'>
    </v-header>
这是当切换到MyBlog时，他就会被选中，随便刷新，都没有方法二的情况出现。

第四种：也是最官方，最简单的。自己当时看VUEROUTER，因为看着面熟，看的比较快，所以错过了这个知识点，开始页的最下面有这样一句话：当 <router-link> 对应的路由匹配成功，将自动设置 class 属性值 .router-link-active，所以你只需要在自己的STYLE文件中，写了.router-link-active的样式，列表选中后，系统就会自动去绑定这个样式。*此处应该有很多个锤头掩面的表情*。
 [1]: https://sfault-image.b0.upaiyun.com/198/578/1985780166-58f1817c7826a_articlex