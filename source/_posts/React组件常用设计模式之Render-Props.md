---
title: React组件常用设计模式之Render Props
date: 2018-12-03 21:11:17
tags:
---
自己在总结最近半年的[React开发最佳实践][5]时，提到了Render Props，想好好写写，但感觉篇幅又太长，所以就有了此文。愿你看完，能有所收获，如果有什么不足或错误之处还请指正。文中所提到的所有代码都可以在示例项目中找到，并npm i,npm start跑起来：
Github:[示例项目][6]

[react官方文档][1]中是这样描述Render Props的：术语 “render prop” 是指一种在 React 组件之间使用一个值为函数的 prop 在 React 组件间共享代码的简单技术。带有render prop 的组件带有一个返回一个***React元素的函数并调用该函数而不是实现自己的渲染逻辑***，并配上了这样的示例：  
```
<DataProvider render={data => (
  <h1>Hello {data.target}</h1>
)}/>

// DataProvider 内部的渲染逻辑类似于下面这样
<div>
    <p>这是一行伪代码</p>
    {this.props.render(data)}
</div>
```
看到这里，有可能你已经恍然大悟，原来所谓的Render Props设计模式不过如此，就是可以用react组件的render属性实现动态化的渲染逻辑。首先需要澄清两点：
 - 不是利用react组件的render属性，像我们class组件拥有的render函数，而是给他自定义一个render函数，属性名可以叫child，也可以叫what，可以是除react固有属性(key，className)以外的任何名字,比如用一个generate属性传递渲染逻辑，然后渲染this.props.generate(data)；
 ```
 <DataProvider
    render={data => (<h1>Hello {data.target}</h1>)}
 />
 ```
 - 上面这种把渲染逻辑写在prop属性中,如果子组件渲染代码多时，看着机会让人感觉有点凌乱，所以更多的时候我们是利用react组件固有的children属性来实现这种设计模式，所以render props更多的时候被称之为使用children prop。
  ```
 <DataProvider>
  {data => (
    <h1>Hello {data.target}</h1>
  )}
 </DataProvider>
 
 ```
Render Props设计模式与我们平常使用的通用组件（传递不同属性，渲染不同结果）相比，后者只是常规的React组件编写方式，用于同一个组件在不同的组件下调用，方便重用，拥有相同的渲染逻辑，更多用于展示型组件，而前者与高阶组件（Hoc）一样，是React组件的一种设计模式，用于方法的公用，渲染逻辑由调用者决定，更多用于容器型组件，当然强调点还是***方法的重用***  

### 初识render props ###
在公司刚用react不久，我自己封装了一款组件，是在antd的AutoComplete组件上进行封装的，在我的一篇文章[《antd组件使用进阶及踩过的坑》][2]提到过，动态效果如下：
![动态效果][3]
而在具体实现里我有这样的一段代码：
```
// 搜索过程代码处理
this.setState({ loading: true, options: null });
// 获取数据，并格式化数据
fetchData(params).then((list) => {
  let options;
  if (isEmpty(list)) {
    options = [DefaultOption];
  } else {
    // **用户自定义数据格式转换；**
    options = format(list).map(({ label, value }, key) => (
      <Option key={key} value={String(value)}>{label}</Option>));
  }
  !options.length && options.push(DefaultOption);
  this.setState({ options, loading: false, seachRes: list });
});
// render实现部分代码：
  <AutoComplete
    autoFocus
    className="certain-category-search"
    dropdownClassName="certain-category-search-dropdown"
    dropdownMatchSelectWidth
    size={size}
    onBlur={this.handleCloseSearch}
    onSearch={this.handleChange}
    onSelect={this.handleSelect}
    style={{ width: '100%' }}
    optionLabelProp="value"
  >
    {loading ?
      [<Option key="loading" disabled>
        <Spin spinning={loading} style={{ paddingLeft: '45%', textAlign: 'center' }} />
      </Option>] : options
    }
  </AutoComplete>


// 调用代码
  const originProps = {
    searchKey: 'keyword',
    fetchData: mockFetch,
    format: datas => datas.map(({ id, name }, index) => ({
    label: `${name}(${id})`,
    value: name,
    key: index
  }))
  };

  <OriginSearch {...originProps} />
```
当时自己对render props并不了解，现在分析一下代码，会发现格式化数据的format属性与render Props有一点像,只不过他调用的位置有点曲折（在远程搜索函数中，根据搜索后的数据执行了渲染逻辑，然后更新到state中，在render函数中再从state中取出来渲染），从性能上来讲也多了一丁点的消耗。这个format函数已经实现了我们想要的，但是为了对比，用children prop来实现了一下：
```
    // 设置loading状态，清空option
    this.setState({ loading: true, options: null });
    // 获取数据
    fetchData(params).then((list) => {
      if (fetchId !== this.lastFethId) { // 异步程序，保证回调是按顺序进行
        return;
      }
      this.setState({ loading: false, seachRes: list });
    });
// render实现部分代码： 
   <AutoComplete
    autoFocus
    className="certain-category-search"
    dropdownClassName="certain-category-search-dropdown"
    dropdownMatchSelectWidth
    size={size}
    onBlur={this.handleCloseSearch}
    onSearch={this.handleChange}
    onSelect={this.handleSelect}
    style={{ width: '100%' }}
    optionLabelProp="value"
  >
    {loading ?
      [<Option key="loading" disabled>
        <Spin spinning={loading} style={{ paddingLeft: '45%', textAlign: 'center' }} />
      </Option>] : this.props.children(seachRes)
    }
  </AutoComplete>

// 调用代码
  const originProps = {
    searchKey: 'keyword',
    fetchData: mockFetch 
  };
  <OriginSearchWithRenderProp {...originProps}>
    {(datas) => 
      datas.map(({id, name}, index) => (
        <Option key={index}>{`${name}(${id})`}</Option>
      ))
    }
  </OriginSearchWithRenderProp>
```
可以看到，当我用children prop来重写这个组件时也是可以的，而且内部逻辑看起来似乎变得更简单。但是不是就完美了，还是值得推敲的，接着往下看。  

### children prop正确打开方式 ###
上一节那一个示例的改写，从render props定义去看，以前的写法对于一个展示型的组件来说，其实更合适，调用者可以编写更少的逻辑，而改写后对调用者就显得有点麻烦，因为虽然调用者可以自己定义渲染逻辑，但是AutoComplete这个组件能接收的子组件类型很有限，对于我这个功能来说，Option是唯一可以接收的子组件类型，所以意义不大。而render props这种模式定义的组件更着重于方法的重用，以及渲染逻辑可以由调用者自定义。来看一看下面这一种需求：

![clipboard.png][9]  

这个需求是我司的一个优惠券运营定制需求，产品想实现各种动态（可增减，数目不定）规则的定义。这种表单是最让人头疼的，不过写表单本来就是一件很让人头疼的事，最近看了篇文章[《再也不想写表单了》][4]图文并茂，让我深有感触。我在上一篇文章[《React文档，除了主要概念，高级指引其实更好用》][5]总结了怎样用配置的写antd表单。
回到正题，这个需求大致来看，第一： 需要自定义表单,常用的表单输入，为一个下拉框或一个输入框什么的，这里都是一个字段有多个输入，或则有多个选择。第二： 每个表单项要实现动态增减。所以如果不用设计模式的话，可能这四个表单项，你需要写4个自定义的组件，何况我那个需求这种表单有8个。但如果仔细观察，可以发现，这4个表单有着相似的行为，就是动态增减，每个表单每一项数据结构相似，只是四个表单的渲染不同。所以这就是最适合render props这种设计模式来定义这个容器，被这个容器包裹的组件他们拥有一个增加和一个减少的方法，然后他们相似的数据结构大概是这样：  

```
const datas = [{
      key: 0,
      value1: '',
      value2: '',
    }, {
      key: 1,
      value1: '',
      value2: '',
    }]  
```
基于以上的总结我们可以这样实现组件（看代码）：
```
export default class DaynamicForm extends React.Component {
  constructor(props) {
    super(props);
    const { value = [{ key: 0 }] } = props;
    this.state = {
      rules: value.map((ele, key) => ({ ...ele, key })),
    };
    this.handlMinus = this.handlMinus.bind(this);
    this.handlAdd = this.handlAdd.bind(this);
    this.handleChange = this.handleChange.bind(this);
  }
  // 处理减项，逻辑删除
  handlMinus(index) {
    const { rules } = this.state;
    rules[index].deleteFlag = true;
    this.setState({
      rules: [...rules]
    });
    this.trigger(rules);
  }
// 处理增项
  handlAdd() {
    let { rules } = this.state;
    rules = rules.concat([{ value: undefined, key: rules.length }]);
    this.setState({ rules: [...rules] });
    this.trigger(rules);
  }
  // 处理表单值的变化
  handleChange(val, index, key) {
    const { rules } = this.state;
    rules[index][key] = val;
    this.setState({
      rules: [...rules]
    });
    this.trigger(rules);
  }
  // 触发外部订阅
  trigger(res) {
    const { onChange } = this.props;
    onChange && onChange(res.filter(e => !e.deleteFlag));
  }
  render() {
    const { children } = this.props;
    const { rules } = this.state;
    const actions = {
      handlAdd: this.handlAdd,
      handlMinus: this.handlMinus,
      handleChange: this.handleChange,
    };
    return (
      <div>
        {rules.filter(rule => !rule.deleteFlag).map(rule => (
          <div key={rule.key}>
            {children(rule, actions)}
            {rule.key === 0 ?
              <Button
                style={{ marginLeft: 10 }}
                onClick={this.handlAdd}
                type="primary"
                shape="circle"
                icon="plus"
              /> :
              <Button
                style={{ marginLeft: 10 }}
                type="primary"
                shape="circle"
                icon="minus"
                onClick={() => this.handlMinus(rule.key)}
              />
            }
          </div>)
        )}
      </div>
    );
  }
}

```
而调用的代码，以满减规则为例：
```
    <FormItem {...formItemLayout} label="满减规则">
      <DaynamicForm key="fullRules">
        {(rule, actions) => <span key={rule.key}>
          <span>满</span>
          <InputNumber
            style={{ margin: '0 5px', width: 100 }}
            value={rule.full}
            min={0.01}
            step={0.01}
            precision={2}
            placeholder=">0, 2位小数"
            onChange={e => actions.handleChange(e, rule.key, 'full')}
          />元，减
          <InputNumber
            style={{ margin: '0 5px', width: 100 }}
            value={rule.reduction}
            min={0.01}
            step={0.01}
            precision={2}
            placeholder=">0, 2位小数"
            onChange={e => actions.handleChange(e, rule.key, 'reduction')}
          />
          元
        </span>}
      </DaynamicForm>
    </FormItem>
```
从上面的组件实现代码和调用代码可以看出，render props很好的实现了这一切，并很好的诠释了方法公用，渲染逻辑自定义的概念。详细代码可参见[示例项目][6]
### 推荐 ###
render props这种设计模式，我在使用两种库时见过：react-motion与apollo-graphql。[react-motion][7]是一个react动画库, 常用于个性化动画的定义，比如实现一个平移动画,先看效果：
![实现效果][8]  

实现代码：
```
<Motion style={{x: spring(this.state.open ? 400 : 0)}}>
  {({x}) =>
    // children is a callback which should accept the current value of
    // `style`
    <div className="demo0">
      <div className="demo0-block" style={{
        WebkitTransform: `translate3d(${x}px, 0, 0)`,
        transform: `translate3d(${x}px, 0, 0)`,
      }} />
    </div>
  }
</Motion>
```
apollo-graphql同时兼容了高阶组件和render props的写法，render props模式调用时如下面所示：
```
  <Query query={GET_DOGS}>
    {({ loading, error, data }) => {
      if (loading) return "Loading...";
      if (error) return `Error! ${error.message}`;

      return (
        <select name="dog" onChange={onDogSelected}>
          {data.dogs.map(dog => (
            <option key={dog.id} value={dog.breed}>
              {dog.breed}
            </option>
          ))}
        </select>
      );
    }}
  </Query>
```
以上，就是Render Props，希望你看完能在自己的开发中用起来，如果你发现本文有什么错误或不足之处，欢迎指正。


  [1]: https://react.docschina.org/docs/render-props.html
  [2]: http://closertb.site/2018/06/Antd%E4%BD%BF%E7%94%A8%E8%BF%9B%E9%98%B6%E5%8F%8A%E8%B8%A9%E8%BF%87%E7%9A%84%E5%9D%91/
  [3]: https://sfault-image.b0.upaiyun.com/693/125/693125077-5b1c813ccd518_articlex
  [4]: https://zhuanlan.zhihu.com/p/48241645
  [5]: http://closertb.site/2018/11/React%E6%96%87%E6%A1%A3%EF%BC%8C%E9%99%A4%E4%BA%86%E4%B8%BB%E8%A6%81%E6%A6%82%E5%BF%B5%EF%BC%8C%E9%AB%98%E7%BA%A7%E6%8C%87%E5%BC%95%E5%85%B6%E5%AE%9E%E6%9B%B4%E5%A5%BD%E7%94%A8/
  [6]: https://github.com/closertb/PromoteMyReact
  [7]: https://github.com/chenglou/react-motion
  [8]: https://image-static.segmentfault.com/399/062/3990626986-5c03f5e43f5dd_articlex
  [9]: https://image-static.segmentfault.com/957/576/957576598-5c03e847cec67_articlex