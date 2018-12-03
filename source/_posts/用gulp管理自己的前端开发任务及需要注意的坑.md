---
title: 用gulp管理自己的前端开发任务及需要注意的坑
date: 2017-7-03 21:22:15
categories: '前端自动化'  
tags:
    - gulp
    - 前端自动化
---
关于gulp，grunt,webpack，刚走前端模块化的我，真的是傻傻分不清楚，幸好有大神各种答疑解惑，使我略知一二，[你也想知道的][1]，也许还想知道点啥，资源罗列：
1. [中文官方文档][2]；
1. [阮老师的gulp入门][3]；
1. [我的参考][4];

本次开发,受尤大神知乎上的回答提示，没有采用vue-cli直接入手vue框架，而是采用vue全家桶 requireJs gulp着手自己的前端构建，gulp为自己第一次使用，所以写下此文，算是对自己成长的一次记录。关于gulp，首先你得知道npm,node这些常识，其次官方的API应该细度加实践一下，其四个基本操作：gulp.task、gulp.src、gulp.dest、gulp.watch四个基本方法，需用知道是干什么的，该怎么用。gulp有什么用？文章将基于以下4条逐一展开讲：
1. 搭建web服务器
1. 优化资源，比如压缩CSS、JavaScript、压缩图片；
1. 使用预处理器LESS，jade，JSX需要编译发布；
1. 文件保存时自动重载浏览器；

## 一：开启一个本地web服务 ##

通常开发时，我们不可能一直写静态页面，我们需要在其他设备查看效果或者与后台的动态交互，使前端开发变得更有意义。所以与后端交付之前，你得有一个本地服务器来发布你的内容。直白点说，没有服务器，我们是通过这样的链接（file:///D:/vueProject/myblog/dist/index.html）访问我们的页面的，而有了服务器依赖，我们是通过这样的链接（http://localhost/）访问我们的页面的，应用上线的感觉，有没有？其实以前基于JavaWeb开发（tomcat）网页时，根本就没这档子事。闲话少扯，进入正题。利用gulp资源，开启一个服务器，你需要下载安装gulp-webserver这个插件，然后这样配置,源码：

    var gulp = require('gulp');
    var webserver = require('gulp-webserver');
    gulp.task('Server',function(){
        gulp.src('dist')  //你web资源的根目录
        .pipe(webserver({
            port:80,
            host:'127.0.0.1',
            liveload:true,
            directoryListing:{
                path:'index.html', //你web资源的起始页，在dist目录下
                enable:true
            }
        }))
    });
基于以上，然后在命令行中输入gulp Server就可以开启一个本地端口为80（也可以为其他）的本地服务器，很简单有木有，但上面的服务器有一个问题，只有本机能访问，局域网内其他设备无法访问该站点，别信网上那些啥开防火墙，开端口胡扯的，根本没关系，因为gulp的web-server就只有这点功能，想要服务器局域网都能访问，在[gulp-webserver官方文档][5]的FAQ给出了解决方案：Set 0.0.0.0 as host option.
## 二：优化资源 ##

gulp的主要功能就是文件的合并，压缩，MD5，由于我的前端JS是基于requireJS构建的，为了在页面加载时提高响应速度，就需要减少文件请求数量并压缩文件的大小，为了做这些操作，需要下载gulp-requirejs-optimize（requireJS序列化工具），gulp-rename，gulp-concat，gulp-minify-css，gulp-rev，gulp-rev-collector，through2，gulp-clean，run-sequence等插件包。我要达到的目如下图所示，重新生成发布目录、CSS文件的合并压缩，JS文件的优化及压缩及重命名带上MD5序列号。上源码：
![图片描述][6]，

    var gulp = require('gulp'),
    reqOptimize =require('gulp-requirejs-optimize'),
    rename = require('gulp-rename'),
    changed = require('gulp-changed'),
    contact =require('gulp-concat'),
    rev =require('gulp-rev'),
    through2 = require('through2'),
    revCollector = require('gulp-rev-collector'),
    clean = require('gulp-clean'),
    runSequence = require('run-sequence'),
    minifyCss = require('gulp-minify-css');
    /*将requireJs文件转移到发布目录*/
    gulp.task('revJs',function(){
        gulp.src('js/main/*.js')
        .pipe(gulp.dest('dist/js/main'))
    });
    /*将所有的图片转移到发布目录*/
    gulp.task('revImg',function(){
        gulp.src('img/**/*')
        .pipe(gulp.dest('dist/img'))
    });
    /*将css文件合并压缩转移到发布目录*/
    gulp.task('revCss',function(){
         gulp.src('css/*.css')
        .pipe(contact('index.css'))
        .pipe(minifyCss())//{compatibility: 'ie8'}
        .pipe(gulp.dest('dist/css'))
    });
    /*将主文件依赖管理合并、压缩、重命名、并去掉.js操作，然后转移到发布目录，操作后文件名如app-1da68b69e1.js*/
    function modify(modifier) {
        return through2.obj(function(file, encoding, done) {
            var content = modifier(String(file.contents));
            file.contents = new Buffer(content);
            this.push(file);
            done();
        });
    }
    function replaceSuffix(data) {
        return data.replace(/.js/gmi, '');
    }
    gulp.task('optimizeJS', function (cb) {
        gulp.src('js/app.js')
        .pipe(reqOptimize({
        optimize:'none',
        paths:{
            vue:'lib/vue',
            vueRouter:'lib/vue-router',
            vueResource:'lib/vue-resource',
            temp:'component/template',
            resize:'component/resizeWindow'
        }
        }))
        .pipe(rev())  //- 文件名加MD5后缀
        .pipe(gulp.dest('dist/js'))   //- 生成MD5后的文件
        .pipe(rev.manifest({merge:true}))  //- 生成一个rev-manifest.json，记录版本映射
        .pipe(gulp.dest(''))
        .pipe(modify(replaceSuffix))            //- 对去掉rev-manifest问件中的文件去掉.js后缀，这主要是考虑requireJs的操作规范
        .pipe(gulp.dest(''))
        .on('end',cb);
    });
    /*由于对app.js重命名加入了md5序列号值，所以需要替换原始index.html中关于app.js的引用*/
    /*这里需要注意，revCollector()相当于一个全文件查找替换的过程，以我的为例*/
    /*我的rev-manifest.json文件中对应的映射是：'app': 'app-1da68b69e1'，所以这个函数在操作时，会对index.html全文模糊搜索‘app’这三个关键字，然后替换为app-1da68b69e1，*/
    /*这其中容易出错就在于他是模糊搜索，所以如果你的文件中有个css的class名或自定义的标签名会内容带有app三个字母，它都会进行替换，所以，在转换过程中，要避开这个坑*/
    gulp.task('updateHtml',function (cb) {
        gulp.src(['rev-manifest.json', 'index.html'])
            .pipe(revCollector())                   //- 替换为MD5后的文件名
            .pipe(rename('index.html'))
            .pipe(gulp.dest('dist'))
            .on('end', cb);
    });

基于以上源码，在命令行中依次输入gulp revJs,gulp revImg,gulp revCss, gulp optimizeJS,gulp updateHtml,就可以达到上述目的，这样是不是有点繁琐，能不能一步到位？当然可以，我们可以帮上述任务写到一个命令中，当然你也可以写到default任务中，然后执行  gulp alltask
```
 gulp.task('alltask',['revJs','revImg','revCss','optimizeJS','updateHtml'])

```

上述代码有一个问题，特别是针对我的项目，因为我的updateHtml是基于optimizeJS执行完后生成的rev-manifest.json执行的，但gulp.task中的任务组，默认是并行执行的，但我希望的是前三个转移并行执行，后面两个串行执行，经过资料查找得知，在gulp 4之前，需要依赖run-sequence来管理任务的执行顺序，而gulp 4引入了连个新的API：gulp.series（串行）和gulp.parallel（并行）来保证任务按指定的顺序执行。在这里我采用了run-sequence的解决方案：

    gulp.task('default', function(callback) {
        runSequence(
            'clean',                //- 上一次构建的结果清空
            'revImg',
            'revCss',
            'revJs',
            'optimizeJS',        //- - 文件合并与md5并去.js后缀
            'updateHtml',      //- 首页路径替换为md5后的路径
            'Server',   //- 服务器开启
            callback);
    });
下一步重点研究，怎样将生成后的JS及CSS文件带上如browser-sync-client.js?v=2.18.12版本号的形式。

## 四：文件更改保存时浏览器的自动重载（先跳过三） ##
在我们的开发过程中，我们经常会修改html，CSS，js文件，由于我们的生产目录与开发目录不一致，所以需要执行GULP命令来发布文件到生产目录。但是频繁的关闭服务与重启服务，这样就造成了很多时间浪费，所以我们需要利用gulp.watch来监视文件的改动，并将这些改动重新发布到生产目录，并重启服务（非手动）。由于个人觉得gulp-webServer与gulp-livereload的局限性，所以讲服务采用browser-sync来代替gulp-webServer，后者自身支持热更新，无需在浏览器安装任何插件（gulp-livereload需要安装livereload插件），直接上源码：

    var browserSync = require('browser-sync').create(); //引入模块
    /*每次发布生产文件前，先将dist目录下的文件情况*/
    gulp.task('clean',function () {
        return gulp.src([
            'rev-manifest.json',
            'dist/js/*.js',
            'dist/index.html'
        ]).pipe(clean());
    });
    /*每次app.js变动时，需要清除rev-manifest.json文件中的映射，使其保证只有唯一的一个映射*/
    gulp.task('JSreload',function(){
        return gulp.src(['rev-manifest.json', 'dist/js/*.js','dist/index.html']).pipe(clean());
    })
    /*每次任务发起前，先清空温江，然后再依次发布文件，启动服务器，监听变动*/
    gulp.task('server', ['clean'], function() {
        runSequence(
            'revImg',
            'revCss',
            'revJs',
            'optimizeJS',        //- - 文件合并与md5并去.js后缀
            'updateHtml'      //- 首页路径替换为md5后的路径
        );
        browserSync.init({
            port: 80,
            server: {
                baseDir: ['dist']
            }
        });
      //监控文件变化，自动更新
        gulp.watch('js/app.js', function(){
                runSequence(
                'JSreload',
                'optimizeJS',
                'updateHtml',
                browserSync.reload
            );
        });
        gulp.watch('css/*.css',  function(){
                runSequence(
                'revCss',
                browserSync.reload
            );
        });
        gulp.watch('index.html',  function(){
                runSequence(
                'updateHtml',
                browserSync.reload
            );
        });
    });
    //将server事务，注册为默认任务
     gulp.task('default',['server']);
基于以上操作，命令行运行gulp ,我们就开启了一个基于browserSync的本地服务器如下图所示，并且局域网内的设备都可以通过主机IP port访问应用。
![图片描述][7]

## 三：预处理器文件编译 ##
暂时没用到，后面用到再增加，可以参考[其他人的][8]


  [1]: http://liaokeyu.com/技术/2017/03/20/Gulp和Webpack是一回事吗.html
  [2]: http://www.gulpjs.com.cn/docs/
  [3]: http://javascript.ruanyifeng.com/tool/gulp.html
  [4]: http://muchstudy.com/2016/12/11/Gulp合并requirejs并MD5文件/?utm_source=tuicool&utm_medium=referral
  [5]: https://www.npmjs.com/package/gulp-webserver
  [6]: https://sfault-image.b0.upaiyun.com/201/210/2012100170-5958b42917a25_articlex
  [7]: https://sfault-image.b0.upaiyun.com/260/171/2601716298-5958db8f77a28_articlex
  [8]: https://www.w3ctrain.com/2015/12/22/gulp-for-beginners/