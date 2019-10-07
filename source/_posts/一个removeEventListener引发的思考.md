---
title: 一个removeEventListener引发的思考
date: 2019-07-31 22:41:00
tags:
---
### 起因
最近在看以前的代码时，发现年初在熟悉react hooks新特性时写下了这样一段代码：
```javascript
let i = 0;
function Test(props) {
  const { loading, error, startFetch } = props;
  useEffect(() => {
    const $btn = document.querySelector('.btn'); // .info-con 存在于外层的dom中
    $btn.addEventListener('click', () => {
      const action =`action data${i++}`;
      console.log('resbtn', action);
    }, false);
    return $btn.removeEventListener('click', () => {});
  });
  if (error) {
    return <div>{error.msg}</div>;
  }
  return (
    <div>
      {loading && <h3>loading</h3>}
      <h3 className="mt-10 mb-10">finished</h3>
      <button onClick={startFetch}>获取数据</button>
      <h4 className="mt-10"><span className="btn">Saga模拟测试</span></h4>
    </div>
  );
}
```
我用了addEventListener和removeEventListener来尝试useEffect的挂载和清除功能，细心的你，发现这段代码有几个错误呢？  
自我观察，自认为是有如下几个的：
 1. 清除函数使用方式错误，应该返回的是一个函数，like：() => { $btn.removeEventListener('click', () => {}); }，而我这里是一个语句；
 2. removeEventListener使用语法错误；
 3. useEffect未使用第二个参数，导致addEventListener挂载会多次执行（可以优化）  

第1个和第3个错误，是可以原谅的，当时自己对hooks还不熟悉。但第二个错误，是不可原谅的，这是需要检讨的。本文后面不会对useEffect做深入讲解，[官方文档][1]已经足够清楚，后面围绕removeEventListener来剖析。
### 认清removeEventListener 
第二个错误，拆开看，又可分为两个方面：removeEventListener使用方式多余与语法错误。  
 - 使用方式多余：虽然$btn添加了click监听事件回调，但由于这个节点属于当前Test组件，所以组件销毁时，其相关节点的监听事件也会一并销毁，这个在自己刚接触前端时就做过这一方面的[解析][2]，所以这里的removeEventListener使用是多余的。但如果换成一个组件外的节点，比如我后面替换的.info-con节点，这是一直存在于Layout组件中的，使用removeEventListener是必要的。
 - 语法错误：指使用removeEventListener，前面两个参数是必传的，事件类型（type:click），事件回调函数（listener: callback），由于使用addEventListener是为事件添加的一个队列（即同一个事件，可添加多个监听回调），所以事件回调函数（listener）是必传，且其引用与添加的事件监听回调函数指向相同。详细描述见[官方文档][3]  

关于语法错误，官方文档中有这样一段描述：  

![clipboard.png][5]

由于我在添加监听时，使用的是箭头函数，所以删除时无法找到相同引用的监听事件，所以第一件事就是改变监听函数的写法。完善后，写法是下面这样的：
```javascript
let i = 0;
function Test(props) {
  const { loading, error, startFetch } = props;
  useEffect(() => {
    const $btn = document.querySelector('.info-con');
    const eventAction = () => {
      const action =`action data${i++}`;
      console.log('resbtn:', action);
    };
    $btn.addEventListener('click', eventAction);
    return () => {
      $btn.removeEventListener('click', eventAction);
    };
  }, []);
  if (error) {
    return <div>{error.msg}</div>;
  }
  return (
    <div>
      {loading && <h3>loading</h3>}
      <h3 className="mt-10 mb-10">finished</h3>
      <button onClick={startFetch}>获取数据</button>
      <h4 className="mt-10"><span className="btn">Saga模拟测试</span></h4>
    </div>
  );
}
```
到此，看似已经结束了。但既然已经打开了，就深入的学习一下这个api吧，常用的前两个参数，我们都很熟悉，第三个参数写成布尔值也偶尔会有，但是否已足够了解呢？
### 第三个参数
addEventListener和removeEventListener传参是一样的，第三个可选参数都是一个对象或者一个布尔值：
```javascript
target.addEventListener(type, listener[, options]);
target.addEventListener(type, listener[, useCapture]);
target.removeEventListener(type, listener[, options]);
target.removeEventListener(type, listener[, useCapture]);
```
当参数为布尔值时，意指useCapture，是否在捕获阶段触发监听函数。而为对象时，可用选项如下：  

![clipboard.png][6]
之所以第三个参数有两种形态，是在旧版本中只存在一个布尔值，即useCapture属性；但随着时间推移以及发展的需要，需要支持设置更多的特性设置，所以有了options选项这个对象传参，又为了兼容以前的老程序，所以对两者进行了兼容。once和passive属性非常有趣，但我还没想到合适的使用它们的场景。查看官方文档，发现里面确实有好多以前没有关注到的东西，值得细细品味。
其实在js很多api中，我们都只用了一些常用的用法，而忽略了一些存在且也很适用的不常用传参，比如下面这些：
 - setTimout(callback, time, ...params): 这个有多有用呢，有一道网红面试题，关于闭包的，就可以利用这个参数来解决，在[详解网红前端经典面试题：setTimeout与循环闭包][4]提到过
```javascript
    function test(){
         for (var i=0; i<5; i  ) {
            setTimeout( function timer() {
                console.log(new Date(),i);
            }, 1000);
        }
        console.log('end',new Date(),i);
    }
```
 - bind(thisArg, ...params): 为函数预先传参，在react中用的比较多；
 - replace(reg, str | function): replace第二参数也可以为一个函数，用于自定义替换
 - split(reg, length): split其实也是有第二个参数的，用以指定返回数组的长度不超过指定大小；
 - 暂时想不起来了...  

### 思考
这一次关于对removeEventListener的使用错误，让我不得不意识到，对于自己下一步技术提升的着重点还是该多关注基础，不要学的太宽泛，而成为泛而不精的人，最后一无是处。最近这一年确实太痴迷（yilai）于框架（React），基础关注的太少。框架与基础，框架层出不穷，但基础只是持续在演进。


  [1]: https://react.docschina.org/docs/hooks-effect.html
  [2]: http://closertb.site/2017/11/JS%E4%B8%AD%E7%9A%84%E4%BA%8B%E4%BB%B6%E7%BB%91%E5%AE%9A%EF%BC%8C%E9%80%A0%E6%88%90%E7%9A%84%E7%A8%8B%E5%BA%8F%E9%87%8D%E5%A4%8D%E6%89%A7%E8%A1%8C/
  [3]: https://developer.mozilla.org/zh-CN/docs/Web/API/EventTarget/removeEventListener
  [4]: http://closertb.site/2017/06/%E8%AF%A6%E8%A7%A3%E7%BD%91%E7%BA%A2%E5%89%8D%E7%AB%AF%E7%BB%8F%E5%85%B8%E9%9D%A2%E8%AF%95%E9%A2%98%EF%BC%9AsetTimeout%E4%B8%8E%E5%BE%AA%E7%8E%AF%E9%97%AD%E5%8C%85/
 [5]: https://image-static.segmentfault.com/272/691/2726919146-5d46418e50c5d_articlex
 [6]:https://image-static.segmentfault.com/237/964/2379647791-5d468e6c02e82_articlex