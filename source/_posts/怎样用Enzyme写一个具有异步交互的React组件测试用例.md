---
title: 怎样用Enzyme写一个具有异步交互的React组件测试用例
date: 2018-09-26 23:24:52
tags:
  - jest
  - react
  - antd
  - 单元测试
  - enzyme
---
关于前端React组件测试（jest，Enzyme），网上有大量的入门文章，可以看看，但如果你确实想了解前端自动化测试，个人更推荐看官方的文档和一些比较官方的测试案列，这里推荐两个：
 1. [enzyme官方文档][1]，涵盖了各种说明和API；
 2. [jest官方文档][2]，涵盖了各种说明和API；
 3. [antd基础组件库][3]，每个组件都有较丰富的测试用例；  

本篇文章适合对前端组件测试有一定概念的同学，本文将包含以下几点：
 - shallow, mount, render三种方法渲染的区别；
 - 组件事件的模拟及事件回调的mock；
 - 异步事件的模拟；
### 三种方法渲染的区别 ###
组件测试最重要的前提，就是你需要知道怎么实例化自己的组件，然后才能去判断是否渲染正常，交互怎么样，功能是不是都OK，而这就和下面要说的渲染方法相关，了解一下三种方法的区别，有助于自己少掉坑，少抠脑壳，少掉头发，用一个关于antd Select组件的示例来说明。  

```
      function RenderTest() {
          return (
            <div className="render">
              <Select>
                  <Option key="1" value="1">test1</Option>
                  <Option key="2" value="2">test2</Option>
                  <Option key="3" value="3">test3</Option>
              </Select>
            </div>
          );
      }
      describe('base render test', () => {
          it('component mounted right', () => {
            const wrapper = mount(
              <RenderTest />
            );
            console.log('***mount***',
            'Select:', wrapper.find('Select').length,
            '  Option:', wrapper.find('Option').length,
              '   Div', wrapper.find('div').length,
              '   class', wrapper.find('.render').length);
          });
      });
```
在浏览器中，挂载这个RenderTest，渲染出来的DomTree是这样的：
 ![clipboard.png][7]
然后在组件测试中分别用了三种方法来渲染，得到是下面的结果：
![clipboard.png][8] 

 - Shallow：与语义一样，肤浅表面的，也是浅渲染, 大概就是组件长啥样，就渲染成啥样，子组件不会递归渲染；
 - Mount:  又称Full DOM rendering，组件层虚拟Dom与真实Dom的渲染，所以这个方法里你既能看到Select节点(为2与组件的定义有关)，Option为0，因为Antd的Option都是以绝对定位的方式挂载在body节点下的，而非wrapper节点里。你还可以打印wrapper.find('Trigger')的长度，结果为1,至于为什么，和Select为2一样，需要去看Select的源码；
 - Render: 又称静态渲染，真实dom节点的渲染，无虚拟Dom，不过有趣的是wrapper节点就是根节点，所以wrapper.find('.render')的length为0,而且很多前两者能用的方法在这里都没有，比如containsMatchingElement这样最基本的方法。

### 组件事件的模拟及事件回调的mock ###
下面两节都将以自己最近封装的一个Antd组件为例来做说明，如下图所示：
![组件实例演示][6]  

这个组件的大致实现如上面图所示，产品需求就是需要一个编辑框，这个框在用户点击输入时，需要弹出一个搜索框，根据用户的输入远程搜索获取数据形成一个下拉列表，供用户选择，选择完成后搜索框被收起。而从代码上，也很简单： 
 
```
    <div
      className="originSearch"
      style={style}
      ref={el => this.searchInputWrapper = el}
    >
      <Input
        ref={e => this.searchInput = e}
        readOnly
        placeholder={placeholder}
        value={valueFormat(value)}
        style={{ width: '100%' }}
        size="default"
        {...inputProps}
      />
      {
        isShowSearch &&
        <div className="js-origin-search origin-search">
          <Icon type="search" className="origin-search-icon" />
          <AutoComplete
            autoFocus
            ref={el => this.searchRealInput = el}
            className="certain-category-search"
            dropdownClassName="certain-category-search-dropdown"
            dropdownMatchSelectWidth
            onSearch={this.handleChange}
            onSelect={this.handleSelect}
            style={{ width: '100%' }}
            optionLabelProp="value"
          >
            {loading ? [<Option key="loading" disabled><Spin spinning={loading} style={{ paddingLeft: '45%', textAlign: 'center' }} /></Option>] : options}
          </AutoComplete>
        </div>
      }
    </div>
```
第一个事件，用户开始输入，Input被单击，click事件捕捉，isShowSearch由false变为true, AutoComplete组件渲染，并自动获得焦点。
对于这一个测试，这是需要有用户交互的，和测试点击浏览器，所以我们需要用到模拟事件simulate，看下面代码：  

```
      it('Input state change disable when click', () => {
        const wrapper = mount(
          <HInputSearch {...searchProps} />
        );
        wrapper.find('input').simulate('click');
        expect(wrapper.state('isShowSearch')).toEqual(true);
        const inputNodes = wrapper.find('input');
        expect(inputNodes.length).toEqual(2);
      });
```
除了模拟点击事件，还可以模拟输入框值的change事件，接着我们还可以检测这个变化是否触发了相应的方法，比如下面这段：  

```
      it('Input state change disable when click', () => {
        const inputValue = 'change';
        const change = jest.spyOn(HInputSearch.prototype, 'handleChange'); // handleChange是在定义组件时，定义的一个原型方法
        const wrapper = mount(
          <HInputSearch {...searchProps} />
        );
        wrapper.find('input').simulate('click');
        expect(wrapper.state('isShowSearch')).toEqual(true);
        const inputNode = wrapper.find('.origin-search input');
        inputNode.simulate('change', { target: { value: inputValue } });
        expect(wrapper.find('input').get(1).props.value).toEqual(inputValue);
        expect(change).toBeCalledWith(inputValue); // 这里可以用toBeCalled检测是否调用，而使用toBeCalledWith除了检测是否调用，还可以检测是否正确的传参；
      });  
```
除了在原型上直接mock响应的方法，也可以直接在实例上，查找出某个节点利用jest.spyOn来检测某个方法是否被调用。在下一节还会继续对jest的函数mock进行说明。
### 异步请求的模拟 ###
此次封装的案例组件我称之为远程搜索输入框，所以涉及到防抖与异步请求的发起，所以在Input框值变化时，首先是使用lodash的debounce函数防抖，然后发起请求。所以当我们进行触发后的流程测试时，比如异步请求是否被调用，返回值是否正常的被存入state，Option是否生成，这些统统没法立即执行测试，而是需要一段时间的等待再来判断，我们把这称之为异步测试。进行这个测试，先理一理思路：
 - 首先： 需要模拟一个异步请求；
 - 其次： 需要模拟获取数据后数据转换函数format；
 - 最后： 模拟一个异步任务，这个简单，用setTimeout就可以。
来看一下实现：  

```
      export const response = [{
        name: '李梅梅', id: 12,
      }, {
        name: '徐雷雷', id: 13,
      }, {
        name: 'james', id: 14,
      }];
      export default function fetch() {
        return new Promise(resolve =>
          setTimeout(() => {
            resolve(response);
          }, 5000)
        );
      }
      const mockFetch = jest.fn(val => fetch(val));
      const mockFormat = jest.fn(data => data.map(({ id, name }, index) => ({
        label: `${name}(${id})`,
        value: name,
        key: index
      })));
      const searchProps = {
        value: initValue,
        style: { width: '100%' },
        search: {
          keyword: undefined,
        },
        onSelect: mockSelect,
        format: mockFormat,  // 利用jest.fn() mock的
        fetchData: mockFetch, // 利用jest.fn() mock的
      };
      it('Input state change disable when click', (done) => { // 异步测试必备
          const inputValue = 'change';
          const change = jest.spyOn(HInputSearch.prototype, 'handleChange'); // handleChange是在定义组件时，定义的一个原型方法
          const wrapper = mount(
            <HInputSearch {...searchProps} />
          );
          wrapper.find('input').simulate('click');
          const inputNode = wrapper.find('.origin-search input');
          inputNode.simulate('change', { target: { value: inputValue } });
          expect(change).toBeCalledWith(inputValue); // 这里可以用toBeCalled检测是否调用，而使用toBeCalledWith除了检测是否调用，还可以检测是否正确的传参；
          setTimeout(() => { // 模拟异步任务
            expect(mockFetch).toHaveBeenCalledWith({ keyword: inputValue });
            expect(mockFormat).toBeCalled();
            done();  // 这个done()很重要，会告诉这个异步测试是否完成
          }, 1000);
      });  
```
结合上面的实例可以看出，好像异步测试也不是很麻烦，就在测试用例中多了个测试完的回调函数done；异步请求和mock函数其实质都是用jest.fn直接包一下；异步任务可以直接用setTimeout或者setImmediate来模拟，当然也有文艺一点的写法，比如写一个通用的：  
```
    // 异步任务模拟
    function mockPromises() {
      return new Promise(resolve => setTimeout(() => {
        resolve();
      }, yourTime));
    }
```
### 结语 ###
写完这一个组件，自己收获还是非常大的，并不是学了jest或者enzyme这么多API的使用，说实话，这个意义真不大。主要意义在于不自愿的去看了一些Antd组件以及Antd 底层组件的一些实现源码，它的高阶组件应用以及组件拆分的方式让我还是很有收获。另外，我其实一直在思考，写单元测试的意义，因为开始我以为这个很神奇，能够写一些逻辑什么的，就能找出自己组件的bug.其实不是，单测只是在你能想到的案列进行描述，然后看是否运行正常，有些小bug也许能找出来，但一些没想到的应用场景，bug还是在哪里，并没有被发现。所以我觉得意义不大，但后面一次优化，改变了我的想法。试想：
 - 当你歇了一段时间，也许你自己写的组件你都看不懂了，但你接到团队其他人的需求需要优化其中的一部分代码，你改完觉得没问题，就push了代码，但是一上线，发现动了不改动的逻辑，影响了最初的功能。这个时候，单元测试的意义就体现了。你第一次写了这个组件，并且写了相应的单元测试，并且代码测试覆盖率能达到80%,当你下一次优化时，优化完你只需要再跑一遍以前写的单元测试（如果是功能性的优化，有必要针对这个补充一个针对性的单元测试），如果测试跑通，你在push代码，这样发生bug的概率会显著下降。
以上就是本篇文章所有，谢谢浏览。有什么描述不严谨之处，还请指正。
源码及测试用例：[地址][4]



  [1]: http://airbnb.io/enzyme/
  [2]: https://jestjs.io/docs/en/getting-started
  [3]: https://github.com/react-component
  [4]: https://codesandbox.io/s/j4y3019y65
  [5]: https://github.com/react-component
  [6]: https://sfault-image.b0.upaiyun.com/693/125/693125077-5b1c813ccd518_articlex
  [7]: https://image-static.segmentfault.com/368/933/3689330350-5baa33bb24836_articlex
  [8]: https://image-static.segmentfault.com/217/867/2178679210-5baa3922e35b3_articlex