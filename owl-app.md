# Owl App

## overview

每一个Owl应用有一个root元素、一套模板、一个环境和一些设置。App class是一个表达所有元素的类
```python
const {Component, App} = owl;
class MyComponent extends Component {...}
const app = new App(MyComponent, { props: {}, templates: "..."});
app.mount(document.body);
```

## API 

- 构造器 constrcutor(Root[, config])
- mount(target, options) 第一个参数是html元素，mount app到DOM指定的目标
- destroy() 删除app


