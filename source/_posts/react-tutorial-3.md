title: react入门教程-3
date: 2015-08-26 22:25:05
categories: react  
tags: ['react','javascript','前端']
description:
---
react入门教程-3  
<!--more-->
###组件的生命周期
** 挂载和移除 **
```javascript
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
                        return {count:0,}
                    },
                    componentWillMount: function () {
                        console.log("componentWillMount");
                        var self = this;
                        this.timer = setInterval(function () {
                            console.log(self.state.count);
                            self.setState({count: self.state.count+1,})
                        }, 1000);
                    },
                    componentDidMount: function () {
                        console.log("compoentDidMount");
                    },
                    componentWillUnmount: function () {
                        console.log("componentWillUnmount");
                        clearInterval(this.timer);
                    },
                    removeComp: function () {
                        React.unmountComponentAtNode(document.getElementById("app"));
                    },
                    render: function() {
                        console.log("render");
                        return (
                            <div>
                                <h1>Hello World {this.props.name}</h1>
                                <h3>count: {this.state.count}</h3>
                                <button type="button" name="button" onClick={this.removeComp}>click</button>
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
** 更新 **
```javascript
<script type="text/jsx">
                var MessageBox = React.createClass({
                    getInitialState: function () {
                        return {count:0,}
                    },
                    shouldComponentUpdate: function (nextProp, nextState) {
                        console.log("shouldComponentUpdate");
                        return true;
                    },
                    componentWillUpdate: function (nextProp, nextState) {
                        console.log("componentWillUpdate");
                    },
                    componentDidUpdate: function () {
                        console.log("componentDidUpdate");
                    },
                    doUpdate: function () {
                        this.setState({count: this.state.count+1});
                    },
                    render: function() {
                        console.log("render");
                        return (
                            <div>
                                <h1>Hello World {this.props.name}</h1>
                                <h3>count: {this.state.count}</h3>
                                <button type="button" name="button" onClick={this.doUpdate}>click</button>
                                <SubMessageBox count={this.state.count} />
                            </div>

                        );
                    }
                });

                var SubMessageBox = React.createClass({
                    shouldComponentUpdate: function (nextProp, nextState) {
                        console.log("shouldComponentUpdate");
                        return true;
                    },
                    componentWillReceiveProps: function (nextProp) {
                        console.log("componentWillReceiveProps");
                    },
                    render: function() {
                        console.log("SubMessageBox render");

                        return (
                            <div>
                                SubMessageBox {this.props.count}
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
```
- 挂载  
    1.*componentWillMount()*在挂载发生之前立即被调用。在这里调用*setState*不会触发二次渲染.  
    2.*componentDidMount()*在挂载结束之后马上被调用。需要*DOM*节点的初始化操作应该放在这里.在*render*方法之后被调用,在这个方法中可以获取*DOM*节点.
- 更新  
    1.*componentWillReceiveProps(object nextProps)*当一个挂载的组件接收到新的props的时候被调用。该方法应该用于比较*this.props*和*nextProps*，然后使用*this.setState()*来改变*state*。  
    2.*shouldComponentUpdate(object nextProps, object nextState): boolean*当组件做出是否要更新DOM的决定的时候被调用。实现该函数，优化*this.props*和*nextProps*，以及*this.state*和*nextState*的比较，如果不需要*React*更新*DOM*，则返回*false*。  
    3.*componentWillUpdate(object nextProps, object nextState)*在更新发生之前被调用。你可以在这里调用*this.setState()*。  
    4.*componentDidUpdate(object prevProps, object prevState)*在更新发生之后调用。
- 移除  
    1.*componentWillUnmount()*在组件移除和销毁之前被调用。清理工作应该放在这里。

###mixin
>组件是 *React* 里复用代码最佳方式，但是有时一些复杂的组件间也需要共用一些功能。有时会被称为跨切面关注点。*React* 使用*mixins*来解决这类问题。  
一个通用的场景是：一个组件需要定期更新。用*setInterval()*做很容易，但当不需要它的时候取消定时器来节省内存是非常重要的。*React* 提供生命周期方法来告知组件创建或销毁的时间。

``` javascript
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title>Hello World</title>
    </head>
    <body>
        <div id="app"></div>
        <script type="text/javascript" src="bower_components/react/react-with-addons.js"></script>
        <script type="text/javascript" src="bower_components/react/JSXTransformer.js"></script>
        <script type="text/jsx">
            var MessageBox = React.createClass({
                mixins: [ React.addons.LinkedStateMixin ],
                getInitialState: function () {
                    return {
                        value: "awesome!!",
                        isAwesome: true,
                    }
                },
                render: function() {
                    return (
                        <div>
                            <h3>{this.state.value}</h3>
                            <input type="text" name="txtName" valueLink={this.linkState("value")} />
                            <br />
                            <input type="checkbox" name="chbAwesome" checkedLink={this.linkState("isAwesome")} />
                            <SubMessageBox valueLink={this.linkState("value")} checkedLink={this.linkState("isAwesome")} />
                        </div>
                    );
                }
            });
            var SubMessageBox = React.createClass({
                render: function() {
                    return (
                        <div>
                            <h3>SubMessageBox</h3>
                            <input type="text" name="txtName" valueLink={this.props.valueLink} />
                            <br />
                            <input type="checkbox" name="chbAwesome" checkedLink={this.props.checkedLink} />
                            <SubSubMessageBox {...this.props} />
                        </div>
                    );
                }
            });
            var SubSubMessageBox = React.createClass({
                render: function() {
                    return (
                        <div>
                            <h4>SubSubMessageBox</h4>
                            <input type="text" name="txtName" valueLink={this.props.valueLink} />
                            <br />
                            <input type="checkbox" name="chbAwesome" checkedLink={this.props.checkedLink} />
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
> 注意需要更改引用 `<script type="text/javascript" src="bower_components/react/react-with-addons.js"></script>`  

###mixin进阶
**自定义mixin**
```javascript
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
            var stateRecordMixin = {
                componentWillMount: function () {
                    console.log("stateRecordMixin componentWillMount");
                    this.states = [];
                },
                componentWillUpdate: function (nextProp, nextState) {
                    console.log("stateRecordMixin componentWillUpdate");
                    this.states.push(nextState);
                },
                previousState: function () {
                    var index = this.states.length - 1;
                    var stateTmp = index==-1 ? {count:0} : this.states[index-1];
                    if(stateTmp == null){
                        return {count:0};
                    }
                    return stateTmp;
                },
            }
            var MessageBox = React.createClass({
                mixins: [ stateRecordMixin ],
                getInitialState: function () {
                    return {
                        count: 0,
                    }
                },
                doUpdate: function () {
                    this.setState({
                        count: this.state.count + 1,
                    });
                },
                render: function() {
                    console.log("render");
                    return (
                        <div>
                            <h3>{this.state.count} || {this.previousState().count}</h3>
                            <br />
                            <button type="button" name="button" onClick={this.doUpdate}>Add</button>
                            <SubMessageBox count={this.state.count} />
                        </div>
                    );
                }
            });
            var SubMessageBox = React.createClass({
                mixins: [ stateRecordMixin ],
                getInitialState: function () {
                    return {
                        count: 0,
                    }
                },
                componentWillReceiveProps: function (nextProp) {
                    this.setState({
                        count:nextProp.count * 10,
                    })
                },
                render: function() {
                    return (
                        <div>
                            <h3>SubMessageBox {this.state.count} || {this.previousState().count} </h3>
                            <SubSubMessageBox {...this.props} />
                        </div>
                    );
                }
            });
            var SubSubMessageBox = React.createClass({
                mixins: [ stateRecordMixin ],
                getInitialState: function () {
                    return {
                        count: 0,
                    }
                },
                componentWillReceiveProps: function (nextProp) {
                    this.setState({
                        count:nextProp.count * 100,
                    })
                },
                render: function() {
                    return (
                        <div>
                            <h4>SubSubMessageBox {this.state.count} || {this.previousState().count} </h4>
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
>关于 mixin 值得一提的优点是，如果一个组件使用了多个 mixin，并用有多个 mixin 定义了同样的生命周期方法（如：多个 mixin 都需要在组件销毁时做资源清理操作），所有这些生命周期方法都保证会被执行到。方法序是：首先按 mixin 引入顺序执行 mixin 里方法，最后执行组件内定义的方法。  

```javascript
<script type="text/jsx">
            var stateRecordMixin = {
                componentWillMount: function () {
                    console.log("stateRecordMixin componentWillMount");
                    this.states = [];
                },
                componentWillUpdate: function (nextProp, nextState) {
                    console.log("stateRecordMixin componentWillUpdate");
                    this.states.push(nextState);
                },
                previousState: function () {
                    var index = this.states.length - 1;
                    var stateTmp = index==-1 ? {count:0} : this.states[index-1];
                    if(stateTmp == null){
                        return {count:0};
                    }
                    return stateTmp;
                },
            }
            var MessageBox = React.createClass({
                mixins: [ stateRecordMixin ],
                getInitialState: function () {
                    return {
                        count: 0,
                    }
                },
                componentWillMount: function () {
                    console.log("componentWillMount");
                    this.states = [];
                },
                componentWillUpdate: function (nextProp, nextState) {
                    console.log("componentWillUpdate");
                    this.states.push(nextState);
                },
                doUpdate: function () {
                    this.setState({
                        count: this.state.count + 1,
                    });
                },
                render: function() {
                    console.log("render");
                    return (
                        <div>
                            <h3>{this.state.count} || {this.previousState().count}</h3>
                            <br />
                            <button type="button" name="button" onClick={this.doUpdate}>Add</button>
                        </div>
                    );
                }
            });
            var SubMessageBox = React.createClass({
                mixins: [ stateRecordMixin ],
                getInitialState: function () {
                    return {
                        count: 0,
                    }
                },
                componentWillReceiveProps: function (nextProp) {
                    this.setState({
                        count:nextProp.count * 10,
                    })
                },
                render: function() {
                    return (
                        <div>
                            <h3>SubMessageBox {this.state.count} || {this.previousState().count} </h3>
                            <SubSubMessageBox {...this.props} />
                        </div>
                    );
                }

            });

            var SubSubMessageBox = React.createClass({
                mixins: [ stateRecordMixin ],
                getInitialState: function () {
                    return {
                        count: 0,
                    }
                },
                componentWillReceiveProps: function (nextProp) {
                    this.setState({
                        count:nextProp.count * 100,
                    })
                },
                render: function() {
                    return (
                        <div>
                            <h4>SubSubMessageBox {this.state.count} || {this.previousState().count} </h4>
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
```
