---
layout: post
title: "react-router的Link实现refresh的效果"
tags: [react, 实战技巧, Web开发]
---

使用React实现SPA时，经常使用react-router进行路由。
正常情况下并不会有特殊的情况，但有时候我们会希望在菜单中设置多个只有QueryString不同的链接，例如：

```javascript
  <Menu>
    <Menu.Item key="sub-3-0"><Link to="/books?category=0">自然科学</Link></Menu.Item>
    <Menu.Item key="sub-3-1"><Link to="/books?category=1">社会科学</Link></Menu.Item>
  </Menu>
```

在routes中配置对应的路由:

```javascript
  <Route path="books" component={Book}></Route>
```

这时候，当前菜单是［自然科学］时，如果点击［社会科学］菜单，页面并不会刷新。

这不是我所期望的，我期望的是页面刷新并更改内容为［社会科学］的内容。

解决办法之一： 通过React的componentWillReceiveProps来捕获这种变化：

```javascript
  componentWillReceiveProps(nextProps) {
    if(nextProps.location.query.category !== this.props.location.query.category){
      ...
    }
  }
```


参考资源：

  * [https://github.com/reactjs/react-router/issues/1102](https://github.com/reactjs/react-router/issues/1102)
  * [https://github.com/reactjs/react-router/issues/1982](https://github.com/reactjs/react-router/issues/1982)
