---
title: 详解网红前端经典面试题：setTimeout与循环闭包
date: 2017-06-08 21:25:37
categories: '前端面试'
tags:
    - 闭包
    - setTimeout
    - 异步
    - 前端面试
---
最近一道面试试题非常火热，堪称面试界网红：

    function test(){
         for (var i=0; i<5; i  ) {
            setTimeout( function timer() {
                console.log(new Date(),i);
            }, 1000 );
        }
        console.log('end',new Date(),i); //为方便后边演示，这里加了打印end标志
    }
不理解闭包，变量作用域和setTimeout函数的同学很多会给出答案A：0,1,2,3,4,5和答案B：5，0,1,2,3,4;不奇怪，但正确答案却是5,5,5,5,5，我以前也是。当然相比较，说出答案B至少比答案A多知道setTimeout函数的用法，重点不在那个延迟1000ms，重点在setTimeout函数，后面会细说。
首先三个概念：
setTimeout(code,millisec)函数：用于在指定的毫秒数后调用函数或计算表达式，接受两个参数，第一个参数为一个函数或计算表达式，我们通过该函数定义将要执行的操作。第二个参数为一个时间毫秒数，表示延迟执行的时间。至于什么异步调用，队列这些概念，这里不做详述，可阅读[好文][1]：
变量作用域：函数内部定义的变量与外部定义的变量，外部指包含这个函数的空间，是父子关系，二不是兄弟，编过程的应该都理解；
闭包：闭包（Closure）是词法闭包（Lexical Closure）的简称，是引用了自由变量的函数。这个被引用的自由变量将和这个函数一同存在，即使已经离开了创造它的环境也不例外。详细运用，[推荐读][2]，个人还是推荐红宝书上面的讲解。
等明白上面第一个settimeout概念后，最后一行为什么先打印最后一行的结果了；
明白变量作用域后，就会明白console.log('end',new Date(),i)中的i是for循环声明的那个i变量，因为var声明的变量不存在代码块（{}）作用域的概念，所以最后打印的值是5；
明白函数后，和变量作用域一起理解，我们可以得出类似如下所示的图（如果理解不正确，还请大神指正）

![图片描述][3]

在for循环声明的五个TimeOut Callback函数都有对变量i的引用，而不是拷贝。因为5个timeout函数都涉及到延迟执行的情况，所以当主线程执行完后（end被打印时），timeout这些回调依次执行（队列：FIFO），此时i的值已经为5了，知道以上这些，后面就简单多了。

开始回到正题：
其实写出这个函数期望输出5，0,1,2,3,4，要达到这个结果，方法有多种，这里列出典型的三种：

方法1：IIFE：

    function test(){
        for (var i = 0; i < 5; i  ) {
         (function() { // j = i
          var  j =i;
          setTimeout(function() {
           console.log(new Date, j);
          }, 1000);
         })();
        }
        console.log(new Date, i);
    }

方法2：函数调用按值传递：

    var output = function (i) {
         setTimeout(function() {
          console.log(new Date, i);
         }, 1000);
    };
    function test(){
        for (var i = 0; i < 5; i  ) {
         output(i); // 这里传过去的 i 值被复制，而不是引用
        }
        console.log(new Date, i);
    }

方法3： ES6 使用le指令声明：

    function test(){
         for (let i=0; i<5; i  ) {
            setTimeout( function timer() {
                console.log(new Date(),i);
            }, 1000 );
        }
     //   console.log('end',new Date(),i);  //因为变量作用域的问题，这里会报i 不存在，未声明
    }

细度上面的三种方法，其实他们相似度很高。首先方法1（声明即执行）和方法2（提前声明，调用时执行），其实他们的思路完全一致，都利用了JavaSrcipt中函数基本类型变量传值，都是值的拷贝，而不是值的引用，然后通过在for循环中执行一个闭包函数，建立一个闭包作用域，来保证引用的i值为注册该回调函数时的值。立即即执行，如果看着别扭，下面这样写也是可以的：

        function test(){
            for (var i = 0; i < 5; i  ) {
             (function() { // j = i
              var  j =i;
              setTimeout(function() {
               console.log(new Date, j);
              }, 1000);
             })();
            }
            console.log(new Date, i);
        }
然后方法3，是利用ES6 let命令声明变量块级作用域的概念，和前面for循环使用var声明i不同的是，var声明的i在整个test()函数作用域内有效，每一次循环， 新的i值都会覆盖旧值；而let声明的， 当前的i只在本轮循环有效， 所以每一次循环的i其实都是一个新的变量，所以也导致打印end时，报i 不存在，未声明的错误，这就是块级作用域的效果，所以5个timeout回调函数虽然都引用了变量i,但实际上这5个i是独立的，仅在自己的块级作用域内有效，其写法类似于：

       function test(){
        for (var i=0; i<5; i  ) {
          let j =i;
            setTimeout( function timer() {
                console.log(new Date(),j);
            }, 1000 );
        }
        console.log('end',new Date(),i);
    }
所以总体来看，上面的方法解决的思路都是从作用域这个概念上下手的，前两者利用function声明形成了自己的作用域，后者利用let命令形成的块级作用域，而来确保对i值的正确引用。
以上就是自己对这个网红面试题的深入理解，如果有说的有错或模棱两可的地方，还请不吝指教。


[1]:from-settimeout-said-the-event-loop-model/

[2]:http://www.ruanyifeng.com/blog/2009/08/learning_javascript_cl

[3]: https://sfault-image.b0.upaiyun.com/293/402/2934028905-59394db8888e2_articlex

osures.html
