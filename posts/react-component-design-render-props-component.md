---
title: React Component Design - Render Props Component
description: React Component Design - Render Props Component
date: 2023-08-08
tags:
  - react
  - component design
  - design pattern
  - render props component
layout: layouts/post.njk
---

讓我們一樣以`Toggle`作為例子，但這次使用`Toggle`的方式要類似於下面

```jsx
function Usage({
  onToggle = (...args) => console.log('onToggle', ...args),
}) {
  return (
    <Toggle onToggle={onToggle}>
      {({on, toggle}) => (
        <div>
          {on ? 'The button is on' : 'The button is off'}
          <Switch on={on} onClick={toggle} />
          <hr />
          <button aria-label="custom-button" onClick={toggle}>
            {on ? 'on' : 'off'}
          </button>
        </div>
      )}
    </Toggle>
  )
}
```

要符合這樣的使用方式其實不難，程式碼如下

```jsx
import React from 'react'
import {Switch} from '../switch'

class Toggle extends React.Component {
  state = {on: false}
  toggle = () =>
    this.setState(
      ({on}) => ({on: !on}),
      () => this.props.onToggle(this.state.on),
    )
  getStateAndHelpers() {
    return {
      on: this.state.on,
      toggle: this.toggle,
    }
  }
  render() {
    return this.props.children(this.getStateAndHelpers())
  }
}
```

可以看到`render function`把`Toggle`的`on` state跟`toggle`傳給`children`，由`children`自行決定要`render`什麼東西，這樣的方式更有彈性可以決定要顯示什麼樣子的component，也可以明確知道`on`跟`toggle`是來自於`Toggle`。跟`Compound component`的設計方式比起來，個人覺得這樣的方式有幾個優點：

- 少了複雜的Provider/Context state傳遞關係
- 不用validate children是不是有在`Toggle`的定義介面裡

但缺點也很明顯，雖然這樣的設計方式保持最大的彈性讓children自行決定要render什麼樣的東西，但這既是優點也是缺點，因為這樣的component所持有的邏輯過少，children還是需要自行決定顯示的內容。另外一個缺點則是如果component都採用這樣的方式，則會讓component的層級過於多層，底層的children component最終會很難排查props是從何而來。

雖然還沒介紹到`HOC component`的設計方式，但這邊[引述](https://youtu.be/BcVAq3YFiuc)一下`react-router`的作者Michael Jackson提到這樣做比`HOC component`好的原因還有：

- 解決了naming collision的問題
- 跟`HOC component`比起來提供更多的彈性可以決定`render`什麼樣的東西，換句話說，這樣的方式直接可以取代`HOC component`，但是，`HOC component`卻沒辦法取代`Render props`

之後等介紹到`HOC component`的時候會再更詳細說明優缺點。
