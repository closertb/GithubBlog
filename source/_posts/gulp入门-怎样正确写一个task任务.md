---
title: gulp入门:怎样正确写一个task任务
date: 2017-06-23 21:25:10
categories: '前端自动化'  
tags:
    - gulp
    - 前端自动化
---
[gulp官网][1]的API用了寥寥几个字，写完了四个基本方法的调用，但task占了二分之一,足以见得他的重要,所以得好好研究。从下面这张图片说起：

![图片描述][2]

当定义一个任务时，你可以定义它的前置任务，可以是一个列表，这个列表会以最大的并发数执行，待这个前置任务列表中所有任务执行完后，才会执行当前任务，但是这个列表中的所有任务都必须使用正确的异步执行方式，：使用一个 callback，或者返回一个 promise 或 stream。熟读API项中的注意项：

```
默认的，task 将以最大的并发数执行，也就是说，gulp 会一次性运行所有的 task 并且不做任何等待。如果你想要创建一个序列化的 task 队列，并以特定的顺序执行，你需要做两件事：

给出一个提示，来告知 task 什么时候执行完毕，
并且再给出一个提示，来告知一个 task 依赖另一个 task 的完成。

```

你可能或者见过下面这些task书写方式：
```
   //刚入门，多数人的写法
   gulp.task('revImg',function(){
    gulp.src('img/**/*')
    .pipe(gulp.dest('dist/img'))
    });

    //读过API，返回一个stream的写法
    gulp.task('revImg',function(){
    return gulp.src('img/**/*')
    .pipe(gulp.dest('dist/img'))
    });

    //读过API，接受一个 callback
    gulp.task('revImg',function(){
    return gulp.src('img/**/*')
    .pipe(gulp.dest('dist/img'))
    });
    //promise这种太高大上的，在gulp里还没用过；具体来看一段真实的演示代码；
        gulp.task('tC',function(){
            gulp.src('css/*.css')
            .pipe(contact('index.css'))
            .pipe(rev())
            .pipe(gulp.dest('dist/test'))
            .pipe(rev.manifest({merge:true}))
            .pipe(gulp.dest('dist/test'))
        });
        gulp.task('tJ',function(){
            gulp.src('js/main/require.js')
            .pipe(rev())
            .pipe(rev.manifest({merge:true}))
            .pipe(gulp.dest('dist/test'))
        });
        gulp.task('tt',['tC','tJ'],function(){
            gulp.src('img/**/*')
            .pipe(gulp.dest('dist/test'));
        })
```
![图片描述][3]
上面一幅图展示了不加return（上） 和加return（下） 两种方式console里面的打印信息，第一种都没感觉到最大并发，执行现象完全没法解释，后一种就是按照正确的方式，即先执行前面两个前置任务，再执行定义的任务。所以你的前置任务按正确的异步书写形式很重要。
煮饭去了，待续！！！！


  [1]: http://www.gulpjs.com.cn/docs/api/
  [2]: https://sfault-image.b0.upaiyun.com/342/197/342197810-5973f0d85c927_articlex



[3]:/myBlog/upload/170623201965.png