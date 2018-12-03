---
title: Antd使用进阶及踩过的坑
date: 2018-06-09 15:33:19
tags:
---
### 扯点犊子###
一晃眼，两个月过去了，自己从一家不大不小的屌丝公司跳到一家被具有纯正互联网血液的公司。从以前的围绕jQuery、Echarts为主技术栈开展工作，到现在以React、Antd为主技术栈开发业务；但不是所有的业务antd都能支持，所以有时得自己动手，在antd上做一层浅封装。

### 自定义表单组件 ###
在[Antd][1]的Form表单介绍一节中，提到过自定义表单控件。其实例是关于货币价值转换的，如下图所示：
![clipboard.png][2]

当我们在我们的页面中需要频繁的用到某一个组合类型的组件，而Antd又不支持时，最好的做法就是对Antd组件做一层浅封装行成一个独立的组件，当然也可以使用html 自有的表单元素进行封装，只是这样做出来费事，且样式和整个页面没有那么容易统一。封装的注意事项在上面的截图中已经一一列出，接下来将以一个实例来操作说明。  
### 一个带远程搜索的下拉选择组件 ###

![clipboard.png][4]

这个组件的大致实现需求如上面动态图所示。产品需求就是需要一个编辑框，这个框在用户点击输入时，需要弹出一个搜索框，根据用户的输入远程搜索获取数据形成一个下拉列表，供用户选择。这在jquery时代，这是一个很常见的需求，也有很多的组件可选择，但在Antd的组件库中，没有完全匹配的，但有及其相似功能的，比如:

![clipboard.png][5]  

这个组件与产品的需求契合度已经达到了80%, 但是产品说了搜索输入框需要与编辑输入框分开，并且有明显的区别，ok，那就费点事，把Antd组件稍微做一下改变嘛。


![clipboard.png][6]

所以简单分解一下，需要用到Input，Icon, Select这三种组件，具体实现可以查看[SandBox上的源码及示例][3]。说一下自己遇到的难点：
#### 支持双向绑定 ####
          <FormItem type="inline" label="员工姓名">
            {getFieldDecorator('search',{
              initialValue: { name: 'Dom' }
            })(
              <OriginSearch {...modalProps} />
            )}
          </FormItem>
在Antd的Form表单组件中，如果需要做数据双向绑定，就需要用到其提供的getFieldDecorator方法来装饰组件，而我们自己封装的组件要支持这个特性的话，如最开始提到的，我们需要使用onChange方法来触发装饰器值的同步。 

![clipboard.png][7]  

在我们为一个组件添加了装饰器后，可以查看props明显发现多了id，onChange, value三个属性，value属性是用于获取我们在initialValue设定的值的，而onChange方法是用于同步值，实现双向绑定。所以在我们这个组件中，当用户从下拉框中选择一个选项后，我们需要调用onChange方法去同步值，代码如下所示：  

      handleSelect(value, option) {
        const { handleSelect } = this.props;
        const { seachRes } = this.state;
        // 初始化基础信息
        const selectValue = handleSelect(value, option, seachRes) || value;
        this.triggerChange(selectValue);
        this.setState({ value: selectValue });
        this.handleCloseSearch();
      }
      triggerChange(value) {
        const { onChange } = this.props;
        // 调用装饰器方法去同步选中后的值
        onChange && onChange(value);
      }
#### 点击组件以外的地方收起组件 ####
这看似是个很容易实现的需求，但因为Antd所有的弹框组件都用了同一套方法，其弹框Dom树并不是挂载在Select输入框的父节点上，而是直接挂载在Body节点上，所以想用冒泡的机制来实现就不可能了。所以就和投机用了点击事件的节点名称来判断，看具体实现：  

      componentDidUpdate(prevProps, prevState) {
        const { isShowSearch } = this.state;
        const bodyNode = document.querySelector("body");
        if (isShowSearch !== prevState.isShowSearch) {
          // 状态切换的时候才改变状态
          if (isShowSearch) {
            document
              .querySelector(".js-origin-search .ant-select-search__field")
              .focus();
            bodyNode.addEventListener("click", this.handleChangeVisible);
          } else {
            bodyNode.removeEventListener("click", this.handleChangeVisible);
          }
        }
      }
      handleChangeVisible(event) {
        const { isShowSearch } = this.state;
        event = event || window.event;
        const elem = event.target;
        let inComponentClick = false;
        // 当搜索框框被打开时，点击空白处搜索框收起；由于antd的下拉列表是挂载在body下，而非搜索框节点下的某一子节点，所以
        // 无法采用阻止冒泡的方式来避免body下的click事件被响应，所以只有靠判断被点击的节点类，来判断body的click事件是否响应
        if (
          (this.searchInputElement && this.searchInputElement.contains(elem)) ||
          elem.className.indexOf("ant-select-dropdown") !== -1
        ) {
          inComponentClick = true;
        }
        // 当点击事件为非下拉列表选中事件，切搜索框为展开时，触发搜索框收起方法；
        !inComponentClick && isShowSearch && this.handleCloseSearch();
      }
虽然这只是一次很简单的封装，但其包含的知识点还是非常多的。自己还封装过日期多选，日期选择增加至今，地址地区联合选择器这种，从实现上其实都是一个思路，在这一个SandBox项目中都能看到。
### 实践后的幡然醒悟 ###
用了两个多月，其实Antd自己本身没啥坑，只是由于我们组现在使用的版本是2.9，但自己习惯于看3.x的版本文档，所以多次在一个地方徘徊好久，总以为是自己代码实现有问题，其实是2.9版本还没有实现。
#### 总结一 ####
在Select组件上，2.9与3.x就有较大的差异： 

1. Select 的option必须带有不同的Key值，且value值也不能有相同的，比如在远程搜索加载员工列表时，就会出现同名的情况，所以这时的value就不能只用名字，得用value加工号或则其他值来代替。
2. Select 的 onchange事件在3.x版本以前回调函数只有value值，没有option回调参数。
3. Select 的notFundContent属性可配置结合Spin实现加载动画，但在版本3以下，该配置对于comobox模式无效（其文档未对这个特性（Bug）做说明）。。。
4. Select 的 onSelect事件在3.x以后也有较大改动，其option参数包含的内容作了很大调整，在2.9版本还可以通过option.props.index获取选择的索引,在3.x版本只能间接通过设置key为index，然后通过获取key值来获取index；
5. Select 组件渲染出来的下拉列表是没有挂载在Select组件父节点上的，其是采用绝对定位，挂载在body节点上的。。。所有用父节点做筛选是无法获取的。
![clipboard.png][8]
#### 总结二 ####
另外在表单组件自校验validator的使用上，有一个隐藏的少有人知的使用方法是：  

        <FormItem {...formItemLayout} label="确认密码">
            {
                getFieldDecorator('confirmPassword', {
                    rules: [{
                            required: true,
                            message: '请再次输入以确认新密码',
                        }, {
                            validator: this.handleConfirmPassword
                        }],
                })(<Input type="password" />)
            }
        </FormItem> 
        handleConfirmPassword(rule, value, callback) {
            if (value && value.length < 5 ) {
                callback('长度不足');
                return;
            }
            // Note: 必须总是返回一个 callback，否则 validateFieldsAndScroll 无法响应
            callback()
        }
#### 总结三 ####
当我们使用getFieldDecorator并用initialValue设定初始值时，当我们改变组件的值时，组件表现出的值也改变了，但这个值并不是initialValue设定的，其是组件内部的state值保持的，如果需要继续用initialValue来设定组件的值，我们需要调用resetFields方法使initialValue有效；
#### 总结四 ####
Antd组件个人觉得最好用的功能就是Table,其配合pagination可以直接实现前端分页，在有些使用场景可以大大提高使用体验。

  [1]: https://ant.design/components/form-cn/#this.props.form.getFieldDecorator%28id,-options%29
  [2]: https://sfault-image.b0.upaiyun.com/357/899/3578995350-5b1b822e903dc_articlex
  [3]: https://codesandbox.io/s/j4y3019y65
  [4]: https://sfault-image.b0.upaiyun.com/693/125/693125077-5b1c813ccd518_articlex
  [5]: https://sfault-image.b0.upaiyun.com/327/278/3272781358-5b1c832e33ed7_articlex
  [6]: https://sfault-image.b0.upaiyun.com/706/868/706868443-5b1c868611c53_articlex
  [7]: https://sfault-image.b0.upaiyun.com/330/836/3308362307-5b1c938a89854_articlex
  [8]: https://sfault-image.b0.upaiyun.com/304/283/3042832531-5b1ca659b243e_articlex