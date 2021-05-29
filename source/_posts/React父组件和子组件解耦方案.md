---
title: React父组件和子组件解耦方案
date: 2019-03-24 22:01:12
categories:
  - 前端
  - 设计
tags:
  - React
  - 组件设计
cover: /2019/03/24/React父组件和子组件解耦方案/index.png
---

## 背景

业务中经常会遇到抽离组件的情况，那么父组件和子组件之间该怎么解耦呢？

## 案例

比如实现一个 List 组件：

```jsx
const Item = ({ text }) => (<div>{text}</div>);

class List extends React.Component {
  state = {
    list: [1, 2, 3, 4]
  };

  render() {
    return this.state.list.map(x => <Item text={x} />);
  }
}
```

### 思考

我们封装父组件经常会直接用到这种方法，但是如果该父组件是一个公共组件，那么当我们在外部引用它时，它对于我们而言其实是一个黑盒。父组件和子组件通过 list 牢牢的耦和在了一起。

<!--more-->

## 解耦方案

### 一、使用 render props

```jsx
class List extends React.Component {
  state = {
    list: [1, 2, 3, 4]
  };

  render() {
    return this.props.children(this.state.list);
  }
}

-- jsx

<List>
  {(list) => list.map(x => <Item text={x} />)}
</List>

```

可以看到，通过 render props 的设计实现了父组件和子组件之间的解耦。

### 二、Compound Component

```jsx
class List extends React.Component {
  state = {
    list: [1, 2, 3, 4]
  };

  render() {
    return (
      React.Children.map(
        this.props.children,
        (child, i) => React.cloneElement(child, {
          text: state.list[i]
        })
      );
    )
  }
}

--jsx

<List>
  <Item />
  <Item />
  <Item />
  <Item />
</List>
```

可以看到 compound component 也能实现父子组件解耦，只是在这里可能不是那么完美。其实 compound component 的应用远不止如此，读者可以自行挖掘。😃

## 总结

compound component 和 render props 均实现了父子组件的解耦，并且父组件对于使用者来说也不再是一个黑盒。

### 引申

将父子组件解耦后，父子组件之间只需要接口达成一致便能完成使用。此时若将单元测试覆盖于测试用例，具体又有什么便利之处呢？👾
