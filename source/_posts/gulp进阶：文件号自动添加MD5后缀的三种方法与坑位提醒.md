---
title: gulp进阶：文件号自动添加MD5后缀的三种方法与坑位提醒
date: 2017-07-23 21:21:04
categories: '前端自动化'  
tags:
    - gulp
    - 前端自动化
---
自己[个人博客][1]上线一个月了，先庆祝一下。
前端时间自己写过一篇[gulp入门的流水账][2]，算是自己gulp用了一段时间后的自我总结，但上次文章最后也提到了还有一件要完成的事就是：怎样将生成后的JS及CSS文件带上如browser-sync-client.js?v=2.18.12版本号的形式。这两天也翻阅了大量博客，方法有很多，但还是先明白我们为什么需要这么做。文章很长，很细，望耐心，毕竟是40度高温下烤出来的，我们不熟，我不会让你看。
## 为什么需要给文件加版本号 ##
1.方便版本控制，比如A：app-12345.js或B：app.js?v=12345版本的样式，当文件更新后，我们能明显看到差异，但是A与B还是存在不同，A是在文件名上进行控制，B是在请求头上进行控制，不懂？不要急，后面会慢慢说。
2.强制浏览器更新请求文件，也许你遇到过，当你的同事告诉你他改的样式文件已经上传到服务器更新了，但是你刷新后就是一股鬼火直冒，更新个屁,还是老样子。其实是浏览器的错，也不正确，还有你们网站建设自己的原因。
 - 先说浏览器，具体看下图![图片][3]
   因为http请求时，如果访问的路径不变，而客户端缓存中又有该文件时，浏览器会直接调用缓存中的文件，这样的话，即使服务器的css内容变化了，但是客户端仍然有可能显示的是旧文件。
 - 你自己网站建设原因，没有谁会去把服务器配成每一次请求都调用本地内存，现在前端优化304都嫌慢，更别说重新发起一次完整请求。所以解决办法就是在你的文件上加上你更新后的版本号，当你更新服务器文件后，即使你未删除老的文件，当用户（回头客，首次访问不会有问题）访问你的网站时，就知道要请求的文件本地没有缓存，需要发起一次完整请求。
 - A:app-12345.js和B：app.js?v=12345那种控制方式更好，要第一眼看当然是B方式看着更符合我们的审美啊。但是，确实A确实比B更严谨。因为部分代理缓存服务器不会缓存网址中包含
   '?' 的资源，所以B方法可能会导致你网站的缓存功能失效，每次都需要发起新的完整请求。但我就觉得B方式更符合我的审美，所以就是要用。。。。

## 实现方式 ##
方法A在上一篇文章已经提过，这里直接说方法B，首先明白实现方法B我们需要干那些：

 1. 修改引用文件，让其带版本号，如在index.html中，需要将<link href='css/index.css' rel='stylesheet'>改成带版本号的<link href='css/index.css?v=6bcd23aa12' rel='stylesheet'>
 2. 将index.css文件名改成index.css?v=6bcd23aa12，真的吗？如果你这时没骂这傻逼，那么你和我曾经一样,还是too young to simple。不用改，你只需要将css/index.css的文件用最新的覆盖就好，文件名真不用改。为什么？因为文件类型后面的问号加版本号都起不到任何作用，它只是告诉浏览器这时一个新的请求。

那这样的话，不就很简单了吗，只需要gulp rev()生成版本号之后，再将版本号改成带问号的形式存到rev-manifest.json，然后根据此对照替换index.html页面的相关引用。思路是对的，当不局限于此，这里说两种。
### 第一种： ###
[参考文章][4]，但是如果你和我一样，gulp-revCollector版本是1.2以后的，第一、二步可以，第三步，完了，什么鬼，源码不一样啊，当时就懵逼了，怎么办，要想学，还能咋办，看源码呗。gulp-revCollector部分源码：

        var defaults = {
            revSuffix: '-[0-9a-f]{8,10}-?',  //这个正则表达式下面会用到
            extMap: {
                '.scss': '.css',
                '.less': '.css',
                '.jsx': '.js'
            }
        };
        function revCollector(opts) { //函数入口
        opts = _.defaults((opts || {}), defaults);

        var manifest  = {};
        var mutables = [];
        return through.obj(function (file, enc, cb) {
            if (!file.isNull()) {
                var mData = _getManifestData.call(this, file, opts); //判断文件不为空后，干的第一件事就是调用 _getManifestData函数，似乎会返回一个值下面详解
                ....//中间略，不重要
         }
        function _getManifestData(file, opts) {
        var data;
        var ext = path.extname(file.path);
        if (ext === '.json') {  //判断是不是json文件,也就是判断是不是rev-manifest.json文件
            var json = {};
            try {
                var content = file.contents.toString('utf8');
                if (content) {
                    json = JSON.parse(content); //将json字符串转JSON对象
                }
            } catch (x) {
                this.emit('error', new PluginError(PLUGIN_NAME,  x));
                return;
            }
            if (_.isObject(json)) {  //判断是不是对象，肯定是啦，往下走
                var isRev = 1;
                Object.keys(json).forEach(function (key) {

                    if (!_.isString(json[key])) {
                        isRev = 0;
                        return;
                    }
                    //下面这句很重要，原生语句的意图就是从index-7ef5d9ee29.css这样的字符串中替换掉前面加入的-revHash这一截字符串，就是得到index.css。
                  //  let cleanReplacement =  path.basename(json[key]).replace(new RegExp( opts.revSuffix ), '' );
                  let cleanReplacement =  path.basename(json[key]).split('?')[0];//最暴力的解决办法
                  //下面这条语句很好理解啦，看两个人是不是都有上面替换出的字符串，必须两个都有，前面做的所有工作，就是为了满足他
                    if (!~[
                            path.basename(key),
                            _mapExtnames(path.basename(key), opts)
                        ].indexOf(cleanReplacement)
                    ) {
                        isRev = 0;
                    }
                });

                        if (isRev) {
                            data = json;
                        }
                    }

                }
                return data;
            }
所以这次修改的重点就是将let cleanReplacement =  path.basename(json[key]).replace(new RegExp( opts.revSuffix ), '' );替换为let cleanReplacement =  path.basename(json[key]).split('?')[0];，当然，如果你读懂了，你会发现revCollector这个代码作者给你提供了配置revSuffix这个替换正则表达式的接口，所以你还可以这样就能达到第三步的效果：

```
  gulp.task('update',['revC'],function (cb) {
        gulp.src(['rev-manifest.json', 'index.html'])
            .pipe(revCollector({
                revSuffix: '\?v=[0-9a-f]{8,10}-?'
            }))
            .pipe(rename('index.html'))
            .pipe(gulp.dest('dist'))
            .on('end', cb);
    });

```

其实第一种方法个人不太赞同使用，因为涉及到修改插件包源码，所以无论通用性或移植性还是后期维护，都有不便。
### 第二种 ###
[参考文章][5]，这篇其实与上篇相比就是有很大进步，不用修改源码，只用正则表达式和字符替换就干完了所有事，还是值得推荐，唯一不足的就是好像我们又要学更多的gulp插件，好烦。
### 第三种(自研) ###
其实第一种方法前面两条插件源码修改，只是为了获得一个结果：将index-7ef5d9ee29.css变换为index.css?v=7ef5d9ee29,然后再结合我们第一种方法的改进，就可以达到目的了。为完成第一步，我们可以直接对rev.manifest()中的字符流进行正则匹配替换得到类似index.css?v=7ef5d9ee29的样式，当然这涉及到一定的through2.obj方法调用（这个方法几乎在所有的gulp插件都有用，目的就是将文件转化成流，其他不需要懂，模仿就行），有点自己写gulp插件的意思，整个过程实现如下：

    function replaceSuffix() {
        const pattern =/-[0-9a-f]{8,10}-?.[^']*/gmi; //匹配出-7ef5d9ee29.css，用于后面做文章
        return through2.obj(function(file, encoding, done) {
            let content =String(file.contents).replace(pattern,function(match,pos,origin){
            const pat=/[0-9a-f]{8,10}-?/g;  //匹配出7ef5d9ee29，用于后面拼接
            if(pat.test(match)){
                return  RegExp['$''].concat('?v=', RegExp['$&']);//如果$'和$&这句话看不懂，红宝书第五章正则表达式部分该复习了；
            }else{
                return match;
            }
            });
            file.contents = new Buffer(content);
            this.push(file);
            done();
        });
    }
    gulp.task('cls',function () {
        return gulp.src([
            'dist'
        ]).pipe(clean());
    });

    gulp.task('revC',['cls'],function(){
        return gulp.src('css/*.css')
        .pipe(contact('index.css'))
        .pipe(minifyCss())
        .pipe(gulp.dest('dist'))
        .pipe(rev())
        .pipe(rev.manifest({merge:true}))
        .pipe(replaceSuffix())  //利用上面写的方法替换得到类似index.css?v=7ef5d9ee29的样式
        .pipe(gulp.dest('dist'));
    });
    gulp.task('update',['revC'],function (cb) {
        gulp.src(['dist/rev-manifest.json', 'index.html'])
            .pipe(revCollector({
                revSuffix: '\?v=[0-9a-f]{8,10}-?'  //利用revCollector的可配置，去满足我们需要的模式；
            }))
            .pipe(rename('index.html'))
            .pipe(gulp.dest('dist'))
            .on('end', cb);
    });
相比于第一种和第二种，是不是有一种超脱的感觉，不过记得在写上面那个方法前，需要先引入    through2 这个插件；三种方法就这多么多分析，想选哪一款，自己琢磨。
## 坑位提醒 ##
### js,css生成md5文件号到rev-manifest.json同一个相互覆盖 ###
我们常常是分别压缩所有的js和css然后生成md5码，然后保存在rev-manifest.json文件中，但有时错误的配置rev.manifest({option})中的option,就有可能导致各种错误的发生；[作品RD][6]中有配置的详细说明，提醒一句善用merge，慎用base。有时虽然你option已经配置了{merge:true}，但生成的rev-manifest.json还是缺少某一项，特别你是要用这个文件去更新你的HTNL的时候，这时你要怀疑的是你写task的方式，请看[个人博客中的文章][7]。


### gulp-uglify压缩js报错:  ###

GulpUglifyError: unable to minify JavaScript，造成这个的原因有很多种，但我遇到的是uglify暂且不支持对es6编写的代码的压缩，就算你只用let或者const声明了变量，不行，反正就是不行。
解决办法：babel这时就是救世主，不然它的存在就没有意义，就是说使用gulp-babel转化成ES5再压缩，具体看附录源码。


  [1]: http://closertb.gohost.xzbiz.cn/myBlog
  [2]: https://segmentfault.com/a/1190000010017382
  [3]:http://closertb.gohost.xzbiz.cn/myBlog/upload/170622322868.png
  [4]: http://www.cnblogs.com/givebest/p/4707432.html
  [5]: http://blog.csdn.net/itpinpai/article/details/53011860
  [6]: https://github.com/sindresorhus/gulp-rev
  [7]: http://closertb.site/myBlog/#/coninfo?arcindex=12