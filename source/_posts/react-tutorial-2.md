title: react入门教程-2  
date: 2015-08-23 10:17:42  
categories: react  
tags: ['react','javascript','前端']  
description:
---
react入门教程-2
<!--more-->
<br />
<br />
<br />
### 组件之间的参数传递(props)
``` javascript
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title>Hello World</title>
    </head>
    <body>
        <div id="app"></div>
        <script type="text/javascript" src="bower_components/react/react.js"></script>
        <script type="text/javascript" src="bower_components/react/JSXTransformer.js"></script>
        <script type="text/jsx">
                var MessageBox = React.createClass({
                    getInitialState: function () {
                        return {
                            subMessages:[
                                "test message 1",
                                "test message 2",
                                "test message 3",
                            ],
                        }
                    },
                    render: function() {
                        return (
                            <div>
                                <h1>Hello World {this.props.name}</h1>
                                <SubMessageBox  messages={this.state.subMessages}/>
                            </div>

                        );
                    }
                });
                var SubMessageBox  = React.createClass({
                    getDefaultProps: function () {
                        return {messages:["default message"],}
                    },
                    render: function() {
                        var msgs = [];
                        this.props.messages.forEach(function (msg, index) {
                            msgs.push(
                                <p key={"key_" + index}>"message:" {msg} {index}</p>
                            );
                        });
                        return (
                            <div>
                                {msgs}
                            </div>
                        );
                    }
                });
                React.render(
                    <MessageBox name={"shan"} />,
                    document.getElementById("app"),
                    function () {
                        console.log("render finished.");
                    }
                );
        </script>
    </body>
</html>
```
### Spread Attributes
``` javascript
var props = {};
props.foo = x;
props.bar = y;
var component = <Component {...props} />;
```
能被多次使用，也可以和其它属性一起用。注意顺序很重要，后面的会覆盖掉前面的。
``` javascript
var props = { foo: 'default' };
var component = <Component {...props} foo={'override'} />;
console.log(component.props.foo); // 'override'
```
接收props
``` javascript
var FancyCheckbox = React.createClass({
  render: function() {
    var { checked, ...other } = this.props;
    var fancyClass = checked ? 'FancyChecked' : 'FancyUnchecked';
    // `other` 包含 { onClick: console.log } 但 checked 属性除外
    return (
      <div {...other} className={fancyClass} />
    );
  }
});
React.render(
  <FancyCheckbox checked={true} onClick={console.log.bind(console)}>
    Hello world!
  </FancyCheckbox>,
  document.getElementById('example')
);
```
> ** tips **  
> *props*应该禁止修改, 修改*props*对象可能会导致预料之外的结果.  

### refs和getDOMNode()的使用
``` javascript
var FormApp = React.createClass({
                    getInitialState: function () {
                        return {
                            inputValue: "input value",
                            selectedValue: "A",
                            textareaValue: "some text...",
                            checkedValues:[],
                            radioCheckedValue: "B",
                        }
                    },
                    changeCheckBox: function (e) {
                        var checkedValues = this.state.checkedValues.slice();
                        var index = checkedValues.indexOf(e.target.value);
                        if(index == -1){
                            checkedValues.push(e.target.value);
                        }else{
                            checkedValues.splice(index, 1);
                        }
                        this.setState({
                            checkedValues:checkedValues,
                        });
                    },
                    changeRadio: function (e) {
                        this.setState({
                            radioCheckedValue: e.target.value,
                        });
                    },
                    handleSubmit: function (e) {
                        e.preventDefault();
                        var formData = {
                            input: this.refs.goodInput.getDOMNode().value,
                            select: this.refs.goodSelect.getDOMNode().value,
                            textarea: this.refs.goodTextarea.getDOMNode().value,
                            checkBox: this.state.checkedValues,
                            radio: this.state.radioCheckedValue,
                        }
                        console.log(formData);
                        this.refs.radioButtons.say();
                    },
                    render: function() {
                        return (
                            <div>
                                <form onSubmit={this.handleSubmit}>
                                    <input ref="goodInput" type="text" name="name" defaultValue={this.state.inputValue} /><br/>
                                    <select ref="goodSelect" defaultValue={this.state.selectedValue}>
                                        <option value="A">1</option>
                                        <option value="B">2</option>
                                        <option value="C">3</option>
                                    </select>
                                    <br/>
                                    <RadioButtons ref="radioButtons" changeRadio={this.changeRadio} />
                                    <br/>
                                    <CheckBoxButtons changeCheckBox={this.changeCheckBox} />

                                    <br />
                                    <textarea ref="goodTextarea" name="name" rows="8" cols="40" defaultValue={this.state.textareaValue}></textarea>

                                    <input type="submit" name="name" value="OK" />
                                </form>
                            </div>
                        );
                    }
                });
                var RadioButtons = React.createClass({
                    say: function () {
                        alert(" something... ");
                    },
                    render: function() {
                        return (
                            <span>
                                <input type="radio" name="goodRadio" value="A" onChange={this.props.changeRadio}/>
                                <input type="radio" name="goodRadio" value="B" onChange={this.props.changeRadio} defaultChecked />
                                <input type="radio" name="goodRadio" value="C" onChange={this.props.changeRadio}  />
                            </span>
                        );
                    }
                });
                var CheckBoxButtons = React.createClass({
                    render: function() {
                        return (
                            <span>
                                <input type="checkBox" name="goodChx" value="A" onChange={this.props.changeCheckBox} />
                                <input type="checkBox" name="goodChx" value="B"
                                onChange={this.props.changeCheckBox} />
                                <input type="checkBox" name="goodChx" value="C"
                                onChange={this.props.changeCheckBox} />
                            </span>
                        );
                    }
                });
                React.render(
                    <FormApp name={"shan"} />,
                    document.getElementById("app"),
                    function () {
                        console.log("render finished.");
                    }
                );
```
引用*DOM*节点. *getDOMNode()*仅在挂载在组件上有效(也就是说, 组件已经被放进了*DOM*中).如果尝试在一个未被挂载的组件上调用这个函数(例如在*render*中调用*getDOMNode()*),将会抛出异常.
``` javascript
var MyComponent = React.createClass({
  handleClick: function() {
    // Explicitly focus the text input using the raw DOM API.
    this.refs.myTextInput.getDOMNode().focus();
  },
  render: function() {
    // The ref attribute adds a reference to the component to
    // this.refs when the component is mounted.
    return (
      <div>
        <input type="text" ref="myTextInput" />
        <input
          type="button"
          value="Focus the text input"
          onClick={this.handleClick}
        />
      </div>
    );
  }
});
React.render(
  <MyComponent />,
  document.getElementById('example')
);
```
> **tips**  
> **表单**,受限组件和不受限组件.如果设置了*value*就是受限组件,只能通过*onChange*事件来改变*value*. 如果*value*没有设置或者是*NULL*,则和普通的*HTML*控件一致.
>*select*组件
>`<select>`使用*value*属性来代替*seleted*.
``` javascript
<select value="B">
    <option value="A">Apple</option>  
    <option value="B">Banana</option>  
    <option value="C">Cranberry</option>  
</select>
```
给*value*属性传递一个数组，可以选中多个选项：`<select multiple={true} value={['B', 'C']}>`。
