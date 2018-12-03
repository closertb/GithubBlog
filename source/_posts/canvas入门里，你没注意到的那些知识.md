---
title: canvas入门里，你没注意到的那些知识
date: 2017-07-20 15:19:03
tags:
---
## 来个气势如虹的开头 ##
与看各种文章相比，我更喜欢数学里的逻辑；与学习各种日新月异的框架相比，我更喜欢基础扎实带给人的那种踏实；与拼凑页面页面来回跳转相比，我更喜欢动画，图形在页面中表现的直观。  
也许你和我一样，冲着对H5的好奇，冲着对图形的热爱，学了一下canvas，没有熟练，只是简单入了个门，或许你在入门的门槛上就绊倒了，同学不哭，站起来继续撸。我把我入门两天的积累写下来，有些门槛，绊倒了数百次，有些应该也适合你。下文的ctx指的是canvas创建的2d图形化实例，代码：  
  
    var canvas = document.querySelector('#test');  //test是Html中一个canvas元素的ID,其宽为600，高为400；
    var ctx = canvas.getContext('2d');
## 说件重要的事 ##  

2d上下文（画布）坐标开始于canvas元素的左上角，其顶点为坐标原点，x轴正方向指向屏幕右侧，Y轴正方向指向屏幕下方。所以x值越大越靠右，Y值越大越靠下。这和我们传统的坐标轴是有很大区别的，下面一张图，也许能看出。Y轴反向也许很好理解，但顺时针正0度在坐标轴的那个位置，就值得我们好好记住了，使用arc，熟悉这个常识很重要。

![坐标系比较图][1]
    
## 四个基本方法 ##
### 描边:stroke与strokeStyle ###

     ctx.strokeStyle = 'red';
     ctx.arc(200,200,50,0,PI*2,false);
     ctx.stroke();   
     
 lineTo,moveTo方法，很基础，这里特别说一下arc(x,y,r,startAngle,endAngle,direction)画圆的方法，这个方法接受6个参数，前五个是必须的，分别是圆心位置（x坐标,y坐标），然后是圆的半径r，再然后是起始位置角和终止位置角，这两个很重要，结合前面那张图理解更，我们可知，上面这段代码绘制的第一个点是(0,50）,终止位置也是(0,50）。后面direction是一个可选参数，默认为false，表示顺时针绘制，为true时，为逆时针绘制。
 特别说明：   

 - 红宝书上说绘制路径，都应该以beginPath()方法开始，但实践证明，这并不需要，你只需要在stroke这个方法来结束这段路径的绘制，但加了也不坏事，所以还是加上吧;
 - 只调用一次lineTo(0,50)方法，你是看不到一条直线绘制出来的，因为（0,0）并不是默认的起点，除非你先用moveTo()方法设置一个起点，这个简单的道理叫两点确定一条直线；
 - closePath（）并不是一个与beginPath对照使用的方法，其作用是绘制闭合路径，比如下图所示。可以看到我们只调用了两次lineTo方法，但最后绘制除了三条线，最后这一条线就是closePath作用出来的。但需要注意的是，closePath需在stroke使用前调用。

![闭合路径][2]

    
### 填充:fillStyle与fill ###     

    ctx.fillStyle = 'red'
    ctx.arc(200,200,50,0,PI/3,false);
    ctx.fill();   
与描边对应的就是填充，但这个方法很不讲究，通常来讲，只有封闭区间才有资格填充，但fill这个方法不需要，只要你的路径不是一条直线，那它就能被填充，也就是不在同一条直线上的三点，能确定一个面，这个方法真的很牛逼，我是必须服气的。
### 绘制文本：stokeText与fillText ###  
以上两个方法都能写出一个字，但推荐的用法是fillText，因为其写出来的是实心字，而stokeText写出来的是描边空心字，绝大多数艺术字会采用这种写法。当然，你愿意的话也可以两者结合着用。  

    ctx.fillStyle = 'red'
    ctx.font= 'bold 12px Arial';
    ctx.fillText('Test',300,200);
    
    ctx.strokeStyle = 'red'
    ctx.font= 'bold 12px Arial';
    ctx.strokeText('Test',400,200);
    
### 绘制图像：drawImage  ###  
相信很多入门的，都看不到这个地方，canvas不就是绘制图像的嘛，啊,不准确，canvas是绘制图形的。具体说来就是drawImage，它不只能把图片绘制到画布上，他还能绘制canvas图形，这个在星空，雨滴等案列中应用最多，也是canvas离屏优化用的最多的，看下面这个示例。   
 
    var cache = document.createElement('canvas');
    cache.width = 50;
    cache.height = 50;
    var cacheCtx = cache.getContext('2d'); //这是一块虚拟画布，因为未添加到dom节点中；

    cacheCtx.beginPath();
    cacheCtx.strokeStyle = 'red';
    cacheCtx.moveTo(5,5);
    cacheCtx.lineTo(20,40);
    cacheCtx.lineTo(40,20);
    cacheCtx.closePath();
    cacheCtx.stroke();
    
    ctx.drawImage(cache,50,50);
    ctx.drawImage(cache,50,100);
    ctx.drawImage(cache,100,50);
    ctx.drawImage(cache,100,100);

![6][6]  
通过上例，我们就能看出drawImage在离屏优化中的应用。为什么这样可以，因为，像上面一样，我们要在一块画布上，画四个相同的三角形，这在浏览器表现上，你看不出任何卡顿，那如果你是要在屏幕上绘制一千个或万个这样的图形，且每一帧还要有规律的变换他们的位置，这时你就能明显感觉出浏览器的卡顿，因为你一直在操作dom中的元素，使浏览器一直在不停的重新绘制，渲染网页。但如果你在每一帧绘制下一个状态前，在js中，通过在虚拟的canvas中先绘制好下一帧的背景状态，然后通过drawImage这个方法，来更新浏览器中的画布状态，这样网页在一帧中就只需要渲染一次，而不是时时在渲染，这就达到了离屏优化的目的，所以这也是drawImage用的最多的场景。

过完以上知识，就算是复习一下入门知识了，至于为什么没提fillRect与strokeRect,其实那就是stroke与fill提供给绘制矩形的一个语法糖。
## 奇妙的变换 ##
也许你和我一样，不习惯自己的坐标系是从左上角开始的，我们更习惯坐标原点在左下角或者正中心。
### 坐标原点变换 translate方法###
原点变换到画布中心：ctx.translate(WIDTH/2,HEIGHT/2)，WIDTH与HEIGHT分别指画布的宽与高，为了将坐标变换表现得更明显，我在同一块区域，画了两块大小相似的画布，然后让一块使用原始坐标系，一块采用translate变换后的，具体看代码，和结果图;   
 
    <div class="red">
        <canvas id="test" width="600px" height="400px"></canvas>
        <canvas id="testa" width="598px" height="398px"></canvas>
    </div>
    
    var ctx = document.querySelector('#test').getContext('2d');
    var crx = document.querySelector('#testa').getContext('2d');
    
    ctx.beginPath();
    ctx.strokeStyle = 'blue';
    ctx.arc(0,0,5,0,PI*2,false);
    ctx.moveTo(0,0);
    ctx.lineTo(0,150);
    ctx.moveTo(0,0);
    ctx.lineTo(150,0);
    ctx.stroke();
    
    crx.beginPath();
    crx.translate(300,200);
    crx.arc(0,0,5,0,PI*2,false);
    crx.strokeStyle = 'red';
    crx.moveTo(0,0);
    crx.lineTo(0,150);
    crx.moveTo(0,0);
    crx.lineTo(150,0);
    crx.stroke();

![变化坐标系对比][3]
### 坐标系旋转变换 rotate方法###  
也许你接触canvas很久了，但rotate方法始终没有机会用，只记得红宝书上在绘制钟表时针，分针时，提到了rotate的用法，但只要你熟悉，用法就能千奇百怪。API明确的说rotate(angle)，是指围绕原点图像旋转angle弧度。所以，你需要记住两点。第一：图像是围绕原点旋转；第二：与其说是图像旋转，不如说是整个坐标系旋转了。看示例,还是和上面一样，有对比，才有说服力： 
 
    ctx.beginPath();
    ctx.strokeStyle = 'blue';
    ctx.translate(300,200);
    ctx.arc(0,0,5,0,PI*2,false);
    ctx.moveTo(0,0);
    ctx.lineTo(0,150);
    ctx.moveTo(0,0);
    ctx.lineTo(150,0);
    ctx.stroke();
    ctx.moveTo(0,0);
    ctx.lineTo(0,-150);
    ctx.stroke();
    
    crx.beginPath();
    crx.strokeStyle = 'red';
    crx.translate(300,200);
    crx.rotate(PI/3); //旋转60*
    crx.arc(0,0,5,0,PI*2,false);
    crx.moveTo(0,0);
    crx.lineTo(0,150);
    crx.moveTo(0,0);
    crx.lineTo(150,0);
    crx.stroke();
    crx.moveTo(0,0);
    crx.lineTo(0,-150);
    crx.stroke();

![旋转坐标系][4]
与上个图相比，我将两块画布的中心都先变换到了画布的中心，作为对比，红色画笔的做了旋转变换，在分别绘制了正x，正y轴以后，在stoke方法结束一段绘制以后，又绘制了负y轴，可以看到，上一段绘制路径的rotate仍然起作用,所以rotate是对画布坐标系做了变换，而不是狭义上的绘制图像，这个区别很重要。后面在上一段代码，不用sin，cos计算，就能绘制一个正多边形，这里以6边形为例。 
 
    ctx.beginPath();
    ctx.translate(300,200);
    ctx.strokeStyle = 'red';
    var a=0;
    for(var i=1;i<7;i++){
        ctx.lineTo(0,150);
        ctx.rotate(PI/3);
        a =a + PI/3 ;
    }
    ctx.closePath();
    ctx.stroke();
    ctx.lineTo(0,0);
    ctx.stroke();
    
    
![5][5]
## 不重要，但很点题的API ##
绘制一个普通的图形，也许上面的API也已足够，但要让你的图形有亮点，你可能还需要多了解一些，我个人觉得非常有用的两个就是阴影的绘制（shadow）与渐变色的添加（createLinearGradient）。在用canvas做星星背景时，或者飞行的流星周围的闪光，这些都是用shadow做出来的特效。而消逝的流星，头部亮，尾部暗的效果就是用渐变色做的，在以后的系列中，会结合实例多讲一些。

[1]:https://sfault-image.b0.upaiyun.com/336/552/3365522170-5a3919671ca84_articlex
[2]:https://sfault-image.b0.upaiyun.com/287/414/2874149570-5a392282d6332_articlex
[3]:https://sfault-image.b0.upaiyun.com/351/299/3512999298-5a3a6baf03813_articlex
[4]:https://sfault-image.b0.upaiyun.com/194/568/1945685394-5a3a744063ec5_articlex
[5]:https://sfault-image.b0.upaiyun.com/791/174/791174139-5a3a77a9200cd_articlex
[6]:https://sfault-image.b0.upaiyun.com/297/878/2978785898-5a3a80ee89c10_articlex

