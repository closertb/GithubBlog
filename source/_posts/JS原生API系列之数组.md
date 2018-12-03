---
title: 用好js原生数组API
date: 2017-09-08 20:52:38
categories: '基础知识'
tags:
    - js
    - array
---

## 前言 ##
最近工作做数据交互展示，常和数据打交道，而随之而来的就是遇见后端传来的各种各样的数组，我需要用各式各样的方法来变换这些数据，来最好的展示这些数据；很多东西久了没用就容易忘，自己也是边查边用，这篇文章算是自己这一周学习的知识的总结。当然你也可以打看[MSDN查看更标准的叙述][1]
## 数组原生的API ###
那些需要知道的特性：
1：数组在JS中是对象的一种，所以数组也是引用类型，所以操作时要小心，时刻记住你操作的不是一个普通类型；
2：每个数组都自带一个length属性，这个属性很特别，可读也可写。JS函数的arguments因为也拥有length属性，所以其被称为类数组对象。
这里不会提到所有的API，因为真的太多了。我只提我最近常用的，还有被人们常说的增删查改。
### API之增删查改###
我以前听说增删查改（CRUD），是sql语言。但有一次参加面试，面试官问：说说你知道的JS数组增删查改，那数组的增删查改是那些呢？说真的，当时我一脸懵逼，我还以为数组还能用sql语句操作
**增加：**push，unshift,还有通常我们常用的arr[arr.length] = newChild;
首先我们有一个数组arr= [1,2,3,4,5,6]
push被称为栈操作，即栈顶入，栈顶出，所以当我们采用arr.push(7)，得到的结果是arr= [1,2,3,4,5,6,7],其相对于arr[arr.length] = 7;
unshift和push相反，即栈底入，所以当我们做和上面相似的操作，即arr.push（7）,得到的结果是arr= [7,1,2,3,4,5,6]
至于arr.push(3.5)和arr.unshift(3,5)答案是多少，你可以自己亲自尝试一下；
**删除：**pop（对应push）,arr.pop()操作后,arr是arr= [1,2,3,4,5];shift(对应unshift)，arr.shift()操作后，arr= [2,3,4,5,6];操作length，当我们arr.length = 4操作后，得到 arr =[1,2,3,4]；
这里提一句，使用delete arr[arr.length-1]，并不能删除最后一个元素，而只是将最后一个元素的赋值去除，值为undefined,且其length未变.
**查：**很多时候我们都采用for循环遍历加比较，来查找某个元素在数组中的索引，但其实js是支持indexOf方法的，当我们进行arr.indexOf(3),其会返回结果2,进行arr.indexOf(9)，其返回的结果就是-1，和字符串的indexOf方法何其相似,所以也有相对应的lastIndexOf()方法；

**改：**改在数组操作中很常用，也很直接，这个就不长述了。
这里说三个重要的方法concat,slice和splice，concat主要做数组拼接（我经常用它做数组的深拷贝）；slice主要做数组截取，而splice几乎能完成上述所有的CUD操作，之所以要把他们分开提，是因为这两个方法操作较复杂，其cancat与slice并不是对数组本身的操作，而是会产生一个新的Array数组，被操作的数组并没有改变；而splice方法，是直接对数组进行操作，仅当参数删除或替换元素操作时，会返回一个新的数组，其包含的元素就是返回的元素。
**concat方法**：数组的拼接，array.concat([item1[, item2[, . . . [, itemN]]]]) ，array指被拼接的对象数组，item为数组。但我用的最多的就是数组的深拷贝，看实例：

    var hege = ['Cecilie', 'Lone'];
    var stale = ['Emil', 'Tobias', 'Linus'];
    var kai = ['Robin'];
    var children = hege.concat(stale,kai);
    console.log(children)  //打印['Cecilie', 'Lone'，'Emil', 'Tobias', 'Linus'，'Robin']
    var deepArr = stale.concat([]);//数组深拷贝；
    deepArr.push('Denzel');
    console.log(deepArr); //打印['Emil','Tobias', 'Linus','Denzel']
    console.log(stale); //打印['Emil', 'Tobias', 'Linus'];
**slice方法**：arrayObj.slice(start, [end])，从方法的描述可知，其可接受两个参数，start即截取开始的位置，end截取结束的位置，参数可选，如果没有，截取的位置是直到数组末尾，但需要注意的是，start位置的元素是被截取的，而end位置的元素是不包含的，只截取该元素前一个元素。

    var arr= [1,2,3,4,5,6]
    var newArr = arr.slice(0,arr.length);
    console.log(newArr)  //打印 [1,2,3,4,5,6]
    newArr = arr.slice(3,5);
    console.log(newArr)  //打印 [4,5]
其实前面都不复杂，复杂的是参数为负的时候，规则是这样的（来源于MSDN）：如果 start 为负，则将其视为 length   start，其中 length 为数组的长度。如果 end 为负，则将其视为 length   end，其中 length 为数组的长度。如果省略 end，则将一直提取到 arrayObj 的结尾。如果 end 出现在 start 之前，则不会将任何元素复制到新数组中。

    var arr= [1,2,3,4,5,6]
    var newArr = arr.slice(0,-1);
    console.log(newArr)  //打印 [1,2,3,4,5]
    newArr = arr.slice(-3,-1);
    console.log(newArr)  //打印 [4,5]

**splice方法**：arrayObj.splice(start, deleteCount, [item1[, item2[, . . . [,itemN]]]]),相比slice，其复杂太多，所以几乎能完成所有的cud操作，但与其不同的是，上面的方法都只能在数组的头和尾上进行操作，splice能完成任意位置的cud操作；其arrayObj参数为必需，且必须是一个 Array 对象；start参数必需，指数组中移除元素操作的起点，从 0 开始，deleteCount参数必需，指要移除的元素的个数，上面提到的返回数组，其数组元素的个数与deleteCount相等，当deleteCount=0时，其返回一个空数组；item1, item2,. . ., itemN可选，指插入数组中代替已移除元素的元素。直接看实例吧：
    let arr =[1,2,3,4,5,6]  //以下三步是独立操作，非连续操作。偷了个小懒
    arr.splice(2, 2, '11','12'); //这个操作删除了3,4,并在其位置上添加了11,12,相当于改，其结果[1, 2, '11', '12', 5, 6]

    arr.splice(2,0, '11','12');//这个操作删除了0个元素,添加了11,12,相当于增，其结果[1, 2, '11', '12',3,4,5,6]；

    arr.splice(2,2, );//这个操作删除了2个元素,添加了0个元素,相当于删，其结果[1, 2, '11', '12',3,4,5,6],相当于增加；
splice方法的start参数也支持负数，其会自动类加length，直到为正。
### API之重排序###
说道数据处理，也许你利马会想到排序，什么冒泡，差值，希尔，快速排序算法。而JS提供了reverse()数组反转和sort()排序来对数组进行重排序。
**数组反转**reverse():和其名字描述的一致，用于数组反转，需要注意的是，其是对数组本省的操作，并不会产生新数组；很简单，看个示例就明白了：
    var color =['a','b','e','d','c','f'];
    color.reverse();
    console.log(color);//打印[f,c,d,e,b,a]
**数组排序sort()**：这里所说的排序，并不是狭义的有序排列，你可以利用这个方法把有序的数组进行无序排列，为啥？应为sort（）方法支持你自己写比较函数。另外，在没有比较函数的情况下，sort（）方法是根据每个数组项的toString()后根据字典顺序进行排序的；

    var color =['a','b','e','d','c','f'];
    color.sort();
    console.log(color);//打印[a,b,c,d,e,f]
    var num =[1,3,2,12,24,5,7,19];
    num.sort(); //这将证明上面提到的根据每个数组项的toString()后根据字典顺序进行排序
    console.log(num);//打印[1, 12, 19, 2, 24, 3, 5, 7]
如果你想让上面的数据进行升序或者降序进行排序，你需要自己写一个比较函数，即这样：

    var num =[1,3,2,12,24,5,7,19];
    num.sort(function(a,b){
        return a-b;
    });
    console.log(num);//打印[1, 2, 3, 5, 7, 12, 19, 24]
    num.sort(function ()  //让有序变成乱序
    {
        return Math.random()<0.5?1:-1;
    });
    console.log(num);//打印[3, 5, 2, 1, 7, 19, 12, 24]

### API之循环遍历###
也许你已习惯了for循环，或者你对jquery的each方法已经产生了依赖，或许你应该接触点新知识了，毕竟ES6已经不算新了；ES7已经开始被支持了；而你还不知道用ES5的map,some,every,filter来循环遍历你的数组，甚至是forEach。
every():对数组中的每一项运行给定函数，如果该函数对每一项都返回true，则返回true；
some():对数组中的每一项运行给定函数，如果该函数任一项都返回true，则返回true；
所以，纵向看，其实every是所有项的&&操作，而some是||操作；
filter():对数组中的每一项运行给定函数，返回该函数该返回true的项组成新的数组；
map():对数组中的每一项运行给定函数，返回该函数该返回true的项组成新的数组；
forEach():功能类似于for循环；
对于上面5个方法，都有类似的回调参数（item,index,array）,试着从一个例子来了解他们，一个简单的例子显得有些苍白。假如现在我们有这样一个需求，已知某个四川省某个景区今日接待旅客总人数10000人，然后从购票信息获取到前十名的省份和人数，我们想计算这些省份每个所占比例，并把他们的人数用一个数组单独保存下来，用来找最大值，最小值，我们试着不用for循环来解决这个问题；

    var totle = 10000;
    var data = [{name:'四川省',num:3000},
        {name:'重庆市',num:500},
        {name:'江西省',num:900},
        {name:'湖南省',num:600},
        {name:'陕西省',num:800},
        {name:'河北省',num:300},
        {name:'湖北省',num:400},
        {name:'北京市',num:600},
        {name:'云南省',num:400},
        {name:'湖南省',num:300}];
    var tempArr = [];
    data = data.map(function (item) {
        tempArr.push(item.num);
        item.percent = item.num/totle;
        return item;
    })
    console.log(tempArr); //[3000, 500, 900, 600, 800, 300, 400, 600, 400, 300]
    console.log(data); //0:{name: '四川省', num: 3000, percent: 0.3} 1:{name: '重庆市', num: 500, percent: 0.05}......
短短四行代码就完成了这两个需求，如果我们只想要第2名到第9名的数据（即去掉一个最高分，去掉一个最低分）拿来做分析，所以根绝前面抽离出来的数组，我们再借助sort（）和filter（）来完成这个需求：

    tempArr.sort(function (v1,v2) { // 降序排列
        return v2-v1;
    });
    var max = tempArr.shift(),min = tempArr.pop();
    data=data.filter(function (item) {  //这个方法并没有完全达到需求，这里只是演示filter的用法，你可以试着优化这个函数，来完成这个需求
        return (item.num!==max)&&(item.num!==min)
    })
    console.log(tempArr);  //体会一下shift和pop[900, 800, 600, 600, 500, 400, 400, 300]
    console.log(data);   //0:{name: '重庆市', num: 500, percent: 0.05} 1:{name: '江西省', num: 900, percent: 0.09}打印出来，结果只剩下7个了，因为最小值出现了两次。
通过上面两个例子，也许你应该已经体会到了这些原生数组API的作用了，他们在数据处理中，优势非常大，但也不能说，以后就可以完全不依赖ofr循环了，还是很难，上面五个方法有一个通病，就是无法中止遍历，即在循环中break，break一些遍历查找中，还是相当省时，这也是为啥有时我们还是需要for循环来做一些操作. 至于some,every，foreach，你可以自己动手感受一下。
别总是沉溺于已会的那点知识，用好API，你的代码将显得干净，有趣。

  [1]: https://msdn.microsoft.com/zh-cn/library/tkcsy6fe%28v=vs.94%29.aspx
