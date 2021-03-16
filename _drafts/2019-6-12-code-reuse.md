---
layout: post
title: 前端组件代码复用技术汇总
date: 2019-6-12
categories: 前端
cover: https://cdn-images-1.medium.com/max/1600/1*STGt7geAHf9P8gektqNAjw.png
tags: 前端
---

* 单一职责
  > 单一职责要求将组件限制在一个'合适'的粒度. 这个粒度是比较主观的概念, 换句话说'单一'是一个相对的概念. 我个人觉得单一职责并不是追求职责粒度的'最小'化, 粒度最小化是一个极端, 可能会导致大量模块, 模块离散化也会让项目变得难以管理. 单一职责要求的是一个适合被复用的粒度. 避免一鱼多吃。
* 低耦合，高内聚
* DRY

> 单一职责原则告诉我们实现类要职责单一；里氏替换原则告诉我们不要破坏继承体系；依赖倒置原则告诉我们要面向接口编程；接口隔离原则告诉我们在设计接口的时候要精简单一；迪米特法则告诉我们要降低耦合。而开闭原则是总纲，他告诉我们要对扩展开放，对修改关闭。


1. Mixin

2. Render props

3. Context

4. HOC

5. Hooks
  - 最高境界
    * 有逻辑的组件没有视图
    * 有视图的组件没有逻辑

    `useFriendStatus` 组件是没有视图的逻辑组件，即自定义 Hook, 一种名字以 `use` 开头并调用其他 Hook 的函数. 此组件接受一个用户 id，通过 `useState` 和 `useEffect` 完成对在线状态的订阅，并对外返回一个变量。
    可以看到此组定义 hook 完全符合单一原则，它单纯的封装了 online 状态的订阅逻辑, 就像一个 js 工具库一样。神奇的是，作为自定义 hook，它可以和 react 组件完美结合。

    ```jsx
    import React, { useState, useEffect } from 'react';

    function useFriendStatus(friendID) {
      const [isOnline, setIsOnline] = useState(null);

      function handleStatusChange(status) {
        setIsOnline(status.isOnline);
      }

      useEffect(() => {
        ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
        return () => {
          ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
        };
      });

      return isOnline;
    }
    ```

    `FriendStatus` 和 `FriendListItem` 2 个是 UI 组件，他们都依赖于订阅状态这个逻辑。如果没有 hook，我们可能不得不把订阅逻辑在 2 个组件中都写一次。

    ```jsx
    function FriendStatus(props) {
      const isOnline = useFriendStatus(props.friend.id);

      if (isOnline === null) {
        return 'Loading...';
      }
      return isOnline ? 'Online' : 'Offline';
    }
    ```

    ```jsx
    function FriendListItem(props) {
      const isOnline = useFriendStatus(props.friend.id);

      return (
        <li style={{ color: isOnline ? 'green' : 'black' }}>
          {props.friend.name}
        </li>
      );
    }
    ```


class 组件
```jsx
class FriendStatusWithCounter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0, isOnline: null };
    this.handleStatusChange = this.handleStatusChange.bind(this);
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentDidUpdate(prevProps) {
    document.title = `You clicked ${this.state.count} times`;

    // 取消订阅之前的 friend.id
    ChatAPI.unsubscribeFromFriendStatus(
      prevProps.friend.id,
      this.handleStatusChange
    );
    // 订阅新的 friend.id
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  handleStatusChange(status) {
    this.setState({
      isOnline: status.isOnline
    });
  },

  render () {
    return <div>{ this.state.isOnline }</div>
  }
}
```

hook 组件，简洁，封装性好。

```jsx
  function FriendStatusWithCounter(props) {
    const [count, setCount] = useState(0);

    useEffect(() => {
      document.title = `You clicked ${count} times`;
    });

    // 逻辑封装在一个 hook 里，而不是分散在 class 组件的生命周期内。优雅，而且
    const [isOnline, setIsOnline] = useState(null);
    useEffect(() => {
      function handleStatusChange(status) {
        setIsOnline(status.isOnline);
      }

      ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
      return () => {
        ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
      };
    });

    return (
      <div>{ isOnline }</div>
    )
  }
```
