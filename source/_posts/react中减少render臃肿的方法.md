---
title: react中减少render臃肿的方法
date: 2019-03-07 23:25:07
tags:
  - React
---

最近看了mobx的相关知识，对computed属性十分喜欢。

想起在日常的项目开发中，react经常需要在render函数的开始部分进行一些props属性的计算，再将计算值用于渲染。一般会有以下画面：

```js
class User extends Component {
  render() {
    const { firstName, secondName } = this.props;
    const fullName = firstName + secondName;
    return <div>{fullName}</div>
  }
}
```
<!--more-->
当业务逻辑复杂时可能会造成render函数十分的臃肿。也许我们会进行这样的优化：

```js
class User extends Component {
  getFullName = () => {
    const { firstName, secondName } = this.props;
    return firstName + secondName;
  }
  render() {
    return <div>{this.getFullName()}</div>
  }
}
```

但是我们实质上寻求的是一种computed的属性，将其封装成函数只是一种曲线救国的手段。
直到我在看frontend master的过程中，才发现原来可以使用class的get属性：

```js
class User extends Component {
  get fullName() {
    const { firstName, secondName } = this.props;
    return firstName + secondName;
  }
  render() {
    return <div>{this.fullName}</div>
  }
}
```

### 总结：
使用get属性实现computed的功能可以减少render函数的臃肿。