---
title: JS中的事件绑定，造成的程序重复执行
date: 2017-11-24 09:31:51
categories: '基础知识'
tags:
    - 事件绑定
    - 重复执行
    - 事件解绑
---
## 那些莫名奇妙的bug ##
歇了两周没写点什么了，感觉最近有点知识慌，没啥新知识，分享一下前段时间遇到的bug难题,这个Bug是关于jquery 的on方法绑交互事件，类似于$('#point').on('click','.read-more',function () {})这样的代码造成程序重复执行，很多人在文章里写到了，也说了用off方法来解绑，但他们很少能点出问题的本质，他们忽略了问题的本质其实是事件委托造成的。话不多说，上点天天看到的代码：
第一种：  

        $(document).on('click', function (e) {
            consol.log('jquery事件绑定')
        });
第二种：  
  
       document.addEventListener('click',function (e) {
            consol.log('原生事件绑定')            
       })；
第三种：  

       var id = setInterval(function () {
            console.log('定时器循环事件绑定')
       },1000)；
上面的代码，相信不少同盟，天天都会写到，看似简单的事件绑定，却经常能给我们带来意想不到的结果，特别是在这个SPA，应用AJAX页面局部刷新如此盛行的时代。那什么是事件绑定，造成的程序重复执行呢？这个事情要说清除，好像不是那么简单,还是用一段测试代码来说明吧。你可以拷贝到本地，自己试试：  

    <!DOCTYPE html>
    <html>
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
    </head>
    <body>
    <button class="add_but">点击</button>
    <div id="point">fdfsdf
    </div>
    <script src="https://cdn.bootcss.com/jquery/1.8.3/jquery.js"></script>    
    <script>
        var count=1;
        var example = {
            getData:function () {
                var data ={
                    content:'df'+count++,
                    href:''
                };
                this.renderData(data);
            },
            renderData:function (data) {
                document.getElementById('point').innerHTML='<div>this is a '+data.content+'点此<a class="read-more" href="javasript:;">查看更多</a></div>';
               $('#point').on('click','.read-more',function () {
                alert('事故发生点');
            })
    /*            setInterval(function () {
                    console.log('fdfdfg');
                },2000);*/
                /*用冒泡来绑定事件，类似于Jquery的on绑定事件*/
            /*  document.querySelector('body').addEventListener('click',function (e) {
                    if(e.target.classList.contains('read-more')){
                        alert('事故发生点');
                    }
                })*/
    
            }
        }  ;
        document.querySelector('.add_but').addEventListener('click',function (e) {
            example.getData();
            e.stopImmediatePropagation();
        });
    </script>
    </body>
    </html>
以上是我为说清这个事情写的一段测试代码,可以拷贝下来试试。当我们点击页面的按钮，触发调用example.getData()这个函数，模拟ajax获取数据成功后，就会根据局部刷新页面内元素类名为point的内容，同时会为加载这个内容中的read-more A标签绑定一个事件，就这样我们想要的效果出现啦，当元素第一次加载时，页面正常，‘事故发生点’弹出一次，当二次刷新触发后，你会发现其弹出了两次，当第三次时，你会发现，其弹三次，以此类推。。。。OMG，这个程序到底怎么了，我明明每次事件绑定前,前面绑定的元素都删除了，为什么，被删除的尸体感觉还在动作，好吧，上面就是我第一次遇到这个情况发出的感叹。
最后是问身边的大神，才突然领悟，原来绑定一直都在，而这个绑定被保存在一个叫做异步事件队列的地方，画了一张需要默契才能看懂的图，勉强看一看。
![图片描述][1]
## 还原真相 ##
其实上面那一段代码是为了测试而特意写的代码，除了定时器外，其他两个点击事件换个正常的写法，重复执行的情况是不会出现的，正常的代码：  

            // jquery 事件直接绑定的写法；
            $('#point .read-more').on('click',function () {
                alert('事故发生点');
            })
            // 原生JS 事件直接绑定的写法；
            document.querySelector('.read-more').addEventListener('click',function (e) {
                alert('事故发生点');
            })
看出差别了吗?其实就是不用冒泡来事件委托，而是直接给添加的元素绑定事件。所以**Dom事件是讲道理的，动态添加的元素，再动态为此绑定事件，待元素被删除后，与其绑定的相应事件其实是会从事件绑定队列中删除的**，而非如上面测试代码，给人的感觉是元素移除后，但其绑定的事件还在内存中。**但请记住**，这是个误会，上面测试的代码之所以给人这种错觉，是因为我们并没有为动态添加的元素绑定事件，而仅仅是用了事件委托的形式，实际上事件是绑定在#point元素上的，其一直存在，利用事件冒泡来让程序知道我们点击了动态添加的链接元素。测试中特意用原生js去重现了这次事件委托，jquery的on绑定事件其实原理基本相同。   
         
    document.querySelector('body').addEventListener('click',function (e) {
         if(e.target.classList.contains('read-more')){
              alert('事故发生点');
          }
    })
## 解除bug的那些方法 ##
### 定时器 ###
这个是最易犯的错误，当然也是最易解的错误，因为设定定时器时，其会返回一个数值，这个数值应该是事件队列此定时器中的一个编号吧，类似于9527；步骤就是设定一个全局变量来保持这个返回值id，在每次设定定时器时，先通过id清除已经设定过的定时器    

         clearInterval(intervalId); //粗暴的写法
         intervalId&&clearInterval(intervalId); //严谨的写法
         intervalId=setInterval(function () {
                    console.log('fdfdfg');
                },2000);  
                
### Dom事件 ###  
其实上面我们已经说过，最直接的办法就是不采用事件委托，而是采用直接绑定；除了这种方式，其实还有其他的方式，绑定的反义词嘛，就是解绑。在jquery中提供了unbind函数来解绑事件，不过在jquery 1.8 版本以后，这个方法已经不推荐了，而是推荐off方法。比如上面的on事件委托的方式，要解绑，可采用语句$('#point').off('click','.read-more')。

### 有缺陷的解决方案，添加flag ###
很好理解，第一次绑定后，flag置位，下一次在执行这个绑定时,程序就知道在这个节点上已经有了绑定，无需再添加，具体操作就是：  

         var flag = false;
         var example = {
            getData: function () {
                var data = {
                    content: 'df' + count++,
                    href: ''
                };
                this.renderData(data);
            },
            renderData: function (data) {
                document.getElementById('point').innerHTML = '<div>this is a ' + data.content + '点此<a class="read-more" href="javasript:;">查看更多</a></div>';
                !flag && $('#point').on('click', '.read-more', function () {
                    alert('事故发生点'+data.content);
                });
                flag = true;
            }
        };
从逻辑上，看起来没有问题，但仔细观察，发现这是有问题的。当我们第二次，第三次刷新时，弹出框的内容还是和第一次模拟刷新后点击后弹出的内容一致，还是'事故发生点df1'，而非和内容一样递增，为什么呢，事件队列里面的回调函数被单独保存起来了，data.content被深拷贝出来了，而不再是一个引用。确实有点难理解，我也不知道到底是为什么，如果哪位能说清楚，还请一定告知。
## 结个尾 ##
写在最后，其实平常写一些程序时，这些情况很少发生，事件绑定，造成程序重复执行这种情况通常会出现在我们写插件的时候，插件需要适应多种调用环境，所以在插件内部做到防止事件重复绑定的情况非常重要。           
  [1]: https://sfault-image.b0.upaiyun.com/183/479/1834797943-5a15903d99bfe_articlex