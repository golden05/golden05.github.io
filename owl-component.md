# Owl Component

## overview

表示一部分用户界面的class，他是组件树的一部分，有一个env环境，从parent到children

```python
const { Component, xml, useState } = owl;
class Counter extends Component {
    static template = xml`
      <button t-on-click="increment">
        Click Me! [<t esc="state.value"/>]
      </button>`;
    state = useState({ value: 0});
    increment() {
        this.state.value++;
    }
}
```

使用xml helper定义inline 模板，使用useState 钩子返回他的参数

## Properties 和 methods

- env(object): 组件的环境
- props(object): 对象从parent到child组件的props
- render(deep[=false])：直接调用将会引起rerender
  
## static properties

- template(string): 将要render组件的模板名字
- components(object, optional):模板需要的包含任何子组件的类
- props(object, optional):是描述组件的类型和维度属性的对象
- defaultProps(object, optional):定义最上层属性的默认值
  
## 生命周期

setup        none           设置  
willStart    onWillStart    异步，在首次render前  
willRender   onWillRender   在组件render前  
rendered     onRendered     在组件render后  
mounted      onMounted      在组件render和加到DOM后  
willUpdateProps onWillUpdateProps 异步，在属性更新前  
willPatch    onWillPatch    DOM被patch前  
patched      onPatched      DOM被patch后  
willUnmount  onWillUnmount  在从DOM移除前
willDestroy  onWillDestroy  在组件被去除前
error        onError        捕获或处理errors  

## sub components

使用其他组件定义一个组件，组合，注册子组件类在他的 static components 对象

```javascript
class Child extends Component {
  static template = xml`<div>child component <t t-esc="props.value"/></div>`;
}

class Parent extends Component {
  static template = xml`
    <div>
      <Child value="1"/>
      <Child value="2"/>
    </div>`;
  static components = {Child};
}
```

## 动态sub components

不是很常用，使用 t-component指令被用于接受一个动态值

```javascript
class A extends Component {
  static template = xml`<div>child a </div>`;
}
class B extends Component {
  static template = xml`<span>child b</span>`;
}
class Parent extends Component {
  static template = xml`<t t-component="myComponent"/>`;
  
  state = useState({child: "a"});

  get myComponent() {
    return this.state.child === "a" ? A : B;
  }
}
```

## status helper

找到当前组件的状态，使用 status helper
```javascript
const { status } = owl;

console.log(status(component));
```