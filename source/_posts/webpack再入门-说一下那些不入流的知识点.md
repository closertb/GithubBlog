---
title: webpack再入门,说一下那些不入流的知识点
date: 2017-08-14 21:19:27
categories: '前端自动化'
tags:
    -webpack
    -vue-cli
---
### 先说说Vue-Cli（Vue项目脚手架）###
关于它能干什么，就不再赘述了，我们主要谈谈生成的与webpack相关的项目结构：
![图片描述][1]

大体上就分三层（当然随着你在生成项目配置的参数不同，项目结构可能会有不同），首先package.json里面的devDependencies属性里，包含了构建这个项目webpack所需要的各种依赖node包和执行项目的快捷指令配置，build文件夹是一些和webpack相关的配置，而config是一些和项目相关的配置，关于这两个文件下每个文件具体是干啥的，可以看下[这篇文章][2]，我只简单说明一下，在执行命令时，这些文件是怎么组合在一起使用的，也可以理解成执行顺序，可以粗略看看：
![图片描述][3]
所以我们知道，Vue-Cli之所以便捷，因为他启用了可配置参数来初始化一个项目，至于为什么要将我们通常的一个webpack.config.js能完成的事情写到2个文件夹12个文件来完成，大概就是传说中的模块化吧，鬼知道呢？
### 关于打包时资源路径的配置 ###
assetsSubDirectory：资源子目录，指js,css，img存放的目录，其路径相对于index.html
比如我将其配置成：assetsSubDirectory：''和assetsSubDirectory：'static',从下面的图，你应该就可以直观感受配置assetsSubDirectory这个路径的区别了。多说一句，此时index.html中js,css的资源路径是这样的：script type=text/javascript src='/static/js/manifest.js?v=8cca728d'>，注意/static/
![clipboard.png][6]

assetsPublicPath：资源目录，默认是这样配置的assetsPublicPath: '/'，指assetsSubDirectory目录也就是js,css,img等资源相对于服务器host地址，传说中的绝对路径，host是什么,ip地址加端口号，比如http://192.168.0.102:8089/newB/beaty.img，其host指的就是http://192.168.0.102:8089，所以如果你如果和我一样，用的是tomcat服务器，那就会遇到当我们要访问我们的网页时,访问的地址是http://ip:port/projectName/index.html,一般没有项目会直接用http://ip:port/index.html这个地址。所以像上面提到的js路径地址，这时就会被解析成http://ip:port/static/js/manifest.js?v=8cca728d，而正确的的url路径应该是http://ip:port/projectName/static/js/manifest.js?v=8cca728d，所以我们需要将assetsPublicPath: '/'改成assetsPublicPath: '/projectName/'，这样才能保证资源的正确发布。
*坑位提示：*自己当时是这样配置的assetsPublicPath: '/projectName'，也就是相对于正确的少了一个'/'，但神奇的是js，css都能打包出正确的路径，自己自动添加了一个'/'，但在一个.vue的template下有一个图片路径被解析成http://ip:port/projectNamestatic/img/flower.jpg，rojectName与static之间少了一个'/'，个人估计是针对于index.html中路径的替换和其他位置的，多了一些容错的处理。
### 关于项目，文件，内容hash值 ###
在webpack打包中，有三类hash值，还是一篇[好文推荐][4]，分别是：
[hash]:[hash] is replaced by the hash of the compilation.
整个项目编译，产生的hash值，官方js打包也是默认使用这个值,所以你所有的静态文件都用这个打包的话，就会看到打包出的文件含有的hash值一样，见下图
[chunkhash]:[chunkhash] is replaced by the hash of the chunk.
模块文件编译，产生的hash值，所以不同的模块产生的hash值就不一样，见下图
[contenthash]:模块中某个内容，也可以理解成模块中的模块，编译时，产生的值，常见的就是.vue单文件组件中的scoped css，这个在编译时，vue-cli打包css用的就是这个contenthash，话说多了无益，直接看图。
![图片描述][5]


  [1]: https://sfault-image.b0.upaiyun.com/153/249/1532490388-598e5a65879e2
  [2]: http://blog.csdn.net/hongchh/article/details/55113751
  [3]: https://sfault-image.b0.upaiyun.com/271/299/2712991453-598e6ba103afd
  [4]: http://www.cnblogs.com/ihardcoder/p/5623411.html
  [5]:https://sfault-image.b0.upaiyun.com/233/588/2335881263-598e8de741c13_articlex
  [6]:https://sfault-image.b0.upaiyun.com/170/006/1700067063-598e743411d50