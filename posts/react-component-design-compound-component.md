---
title: React Component Design - Compound Component
description: React Component Design - Compound Component
date: 2023-07-31
tags:
  - react
  - component design
  - design pattern
  - compound component
layout: layouts/post.njk
---

- [前言](#前言)
- [Compound Component](#compound-component)
- [More Flexible Compound Component](#more-flexible-compound-component)
  - [Check Children](#check-children)
  - [Using Context](#using-context)
- [結論](#結論)
- [Related articles](#related-articles)


# 前言

假設今天我們有一個React component `Switch`，它的props如下

```jsx
Switch.props = {
  on: PropTypes.boolean,
  onClick: PropTypes.func
}
```

請設計一個component叫`Toggle`，這個`Toggle`有一個`state`儲存`on`的狀態，`on`變化時要呼叫props傳下來的`onToggle`callback，相信大多數人會實作類似以下的程式碼

```jsx
import React from 'react'
import { Switch } from '../switch'

class Toggle extends React.Component {
  state = {on: false}
  toggle = () =>
    this.setState(
      ({on}) => ({on: !on}),
      () => {
        this.props.onToggle(this.state.on)
      },
    )

  render() {
    const {on} = this.state
    return <Switch on={on} onClick={this.toggle} />
  }
}
```

使用`Toggle`的方式則類似這樣

```jsx
function Usage({
  onToggle = (...args) => console.log('onToggle', ...args),
}) {
  return <Toggle onToggle={onToggle} />
}
```

這樣的設計看起來沒什麼問題，`on` state保留在`Toggle` 裡面，並不需要透露給外面知道。

如果今天我們接到需求需要在`on`state `true`跟`false`各顯示不同的字串，於是我們修改成

```jsx
import React from 'react'
import { Switch } from '../switch'

class Toggle extends React.Component {
  state = {on: false}
  toggle = () => (
    // same codes
  )

  render() {
    const { isOnText, isOffText } = this.props
    const { on } = this.state
    return (
      <>
        {on ? isOnText : isOffText}
        <Switch on={on} onClick={this.toggle} />
      </>
    )
  }
}
```

這樣的方式有三個缺點：

1. 顯示的順序寫死在component裡面，無法客製化順序或者樣式
2. props命名方式沒統一的情況下，無法讓人一眼就知道這props是什麼意思，要傳什麼內容進來，需要檢視`Toggle`的`propTypes`才可以了解
3. 如果今天在`on`是`true`的時候有額外的callback要呼叫，需要再新增一次props給`Toggle`

# Compound Component

這時候**Compound Component**的設計方式便派上用場，它的概念是

> [讓你的 UI 元件透過 `this.props.children` 的方式傳入給 `parent component`，利用 `React.Children.map()` 來 render 所有傳入的 `this.props.children`，並且透過 `React.cloneElement` 將 parent 的 `state` 傳入每個 children 的 `props`，讓 parent 與 children 之間會 **隱含著狀態的共享**，對於元件使用者來說，他只需要傳入想要的 `children component`，不用知道 parent 與 children 之間如何溝通，當然也能隨意調整順序，這樣的 API 設計，對於元件使用者就非常的友善。

在這樣的原則下，不難發現，Compound component 必須要 **同時結合使用 parent component 與 children component 才有意義**。](https://blog.techbridge.cc/2018/06/27/advanced-react-component-patterns-note/)
>

實作後的結果如下

```jsx
// Compound Components

import React from 'react'
import {Switch} from '../switch'

class Toggle extends React.Component {
  static On = ({on, children}) => (on ? children : null)
  static Off = ({on, children}) => (on ? null : children)
  static Button = ({on, toggle, ...props}) => (
    <Switch on={on} onClick={toggle} {...props} />
  )

  state = {on: false}
  toggle = () =>
    this.setState(
      ({on}) => ({on: !on}),
      () => this.props.onToggle(this.state.on),
    )

  render() {
    return React.Children.map(this.props.children, child =>
      React.cloneElement(child, {
        on: this.state.on,
        toggle: this.toggle,
      }),
    )
  }
}

function Usage({
  onToggle = (...args) => console.log('onToggle', ...args),
}) {
  return (
    <Toggle onToggle={onToggle}>
      <Toggle.On>The button is on</Toggle.On>
      <Toggle.Off>The button is off</Toggle.Off>
      <Toggle.Button />
    </Toggle>
  )
}
```

實作後的結果解決了上面提到的3個問題：

1. 順序現在可以依照`children`的排序做顯示，樣式也可以自定義
2. props維持在最小限度，只有`onToggle`需要傳入
3. `Toggle`內的`children`根據介面，`On`跟`Off`可以接收到`on` state，`Button`則可以收到`on` state跟`toggle` function。所以`On`跟`Off`可以根據接收到的`on`state自行決定要觸發的callback

# More Flexible Compound Component

## Check Children

在上面的實作中，我們的Compound Components只有辦法接受`Toggle`裡面定義的`On`,`Off`,`Button` children，這邊可以做檢查判斷children是否有在`Toggle`的定義內，如果children不在`Toggle`的children的定義內，則正常render children

```jsx
// Compound Components

import React from 'react'
import {Switch} from '../switch'

function componentHasChild(child) {
  for (const property in Toggle) {
    if (Toggle.hasOwnProperty(property)) {
      if (child.type === Toggle[property]) {
        return true
      }
    }
  }
  return false
}

class Toggle extends React.Component {
  static On = ({on, children}) => (on ? children : null)
  static Off = ({on, children}) => (on ? null : children)
  static Button = ({on, toggle, ...props}) => (
    <Switch on={on} onClick={toggle} {...props} />
  )
  state = {on: false}
  toggle = () =>
    this.setState(
      ({on}) => ({on: !on}),
      () => this.props.onToggle(this.state.on),
    )
  render() {
    return React.Children.map(this.props.children, child => {
      if (componentHasChild(child)) {
        return React.cloneElement(child, {
          on: this.state.on,
          toggle: this.toggle,
        })
      }
      return child
    })
  }
}

const Hi = () => <h4>Hi</h4>
function Usage({
  onToggle = (...args) => console.log('onToggle', ...args),
}) {
  return (
    <Toggle onToggle={onToggle}>
      <Toggle.On>The button is on</Toggle.On>
      <Toggle.Off>The button is off</Toggle.Off>
      <Toggle.Button />
      <Hi />
    </Toggle>
  )
}
```

## Using Context

現在我們的`Toggle`元件只有辦法render出第一層的children，當我們想要把`on`state跟`toggle`傳給children不管層數有幾層時，可以使用`React`的`context` API。

```jsx
// Flexible Compound Components with context

import React from 'react'
import {Switch} from '../switch'

const ToggleContext = React.createContext()

class Toggle extends React.Component {
  static On = ({children}) => (
    <ToggleContext.Consumer>
      {({on}) => (on ? children : null)}
    </ToggleContext.Consumer>
  )
  static Off = ({children}) => (
    <ToggleContext.Consumer>
      {({on}) => (on ? null : children)}
    </ToggleContext.Consumer>
  )
  static Button = props => (
    <ToggleContext.Consumer>
      {({on, toggle}) => (
        <Switch on={on} onClick={toggle} {...props} />
      )}
    </ToggleContext.Consumer>
  )
  state = {on: false}
  toggle = () =>
    this.setState(
      ({on}) => ({on: !on}),
      () => this.props.onToggle(this.state.on),
    )

  render() {
    // 由於不用傳遞 props 給 children，也就不用 React.Children.map 了，直接使用 this.props.children 即可
    return (
      <ToggleContext.Provider
        value={{on: this.state.on, toggle: this.toggle}}
      >
        {this.props.children}
      </ToggleContext.Provider>
    )
  }
}

function Usage({
  onToggle = (...args) => console.log('onToggle', ...args),
}) {
  return (
    <Toggle onToggle={onToggle}>
      <Toggle.On>The button is on</Toggle.On>
      <Toggle.Off>The button is off</Toggle.Off>
      <div>
        <Toggle.Button />
      </div>
    </Toggle>
  )
}
```

這邊我們可以增加`Toggle`的children validation，避免`On`, `Off`, `Button`在`Toggle`以外的地方被使用

```jsx
// Flexible Compound Components with context (extra credit 1)
// This adds validation to the consumer

import React from 'react'
import {Switch} from '../switch'

const ToggleContext = React.createContext()

function ToggleConsumer(props) {
  return (
    <ToggleContext.Consumer {...props}>
      {context => {
        if (!context) {
          throw new Error(
            `Toggle compound components cannot be rendered outside the Toggle component`,
          )
        }
        return props.children(context)
      }}
    </ToggleContext.Consumer>
  )
}

class Toggle extends React.Component {
  static On = ({children}) => (
    <ToggleConsumer>
      {({on}) => (on ? children : null)}
    </ToggleConsumer>
  )
  static Off = ({children}) => (
    <ToggleConsumer>
      {({on}) => (on ? null : children)}
    </ToggleConsumer>
  )
  static Button = props => (
    <ToggleConsumer>
      {({on, toggle}) => (
        <Switch on={on} onClick={toggle} {...props} />
      )}
    </ToggleConsumer>
  )
  state = {on: false}
  toggle = () =>
    this.setState(
      ({on}) => ({on: !on}),
      () => this.props.onToggle(this.state.on),
    )
  render() {
    return (
      <ToggleContext.Provider
        value={{on: this.state.on, toggle: this.toggle}}
      >
        {this.props.children}
      </ToggleContext.Provider>
    )
  }
}

function Usage({
  onToggle = (...args) => console.log('onToggle', ...args),
}) {
  return (
    <Toggle onToggle={onToggle}>
      <Toggle.On>The button is on</Toggle.On>
      <Toggle.Off>The button is off</Toggle.Off>
      <div>
        <Toggle.Button />
      </div>
    </Toggle>
  )
}
```

在使用Provider/Context的時候，要特別注意re-render的問題，這邊可以改良一下把Provider的`value`改以`state`傳入

```jsx
// Flexible Compound Components with context
// This allows you to avoid unecessary rerenders

import React from 'react'
import {Switch} from '../switch'

const ToggleContext = React.createContext()

function ToggleConsumer(props) {
  return (
    <ToggleContext.Consumer {...props}>
      {context => {
        if (!context) {
          throw new Error(
            `Toggle compound components cannot be rendered outside the Toggle component`,
          )
        }
        return props.children(context)
      }}
    </ToggleContext.Consumer>
  )
}

class Toggle extends React.Component {
  static On = ({children}) => (
    <ToggleConsumer>
      {({on}) => (on ? children : null)}
    </ToggleConsumer>
  )
  static Off = ({children}) => (
    <ToggleConsumer>
      {({on}) => (on ? null : children)}
    </ToggleConsumer>
  )
  static Button = props => (
    <ToggleConsumer>
      {({on, toggle}) => (
        <Switch on={on} onClick={toggle} {...props} />
      )}
    </ToggleConsumer>
  )
  // The reason we had to move `toggle` above `state` is because
  // in our `state` initialization we're _using_ `this.toggle`. So
  // if `this.toggle` is not defined before state is initialized, then
  // `state.toggle` will be undefined.
  toggle = () =>
    this.setState(
      ({on}) => ({on: !on}),
      () => this.props.onToggle(this.state.on),
    )
  state = {on: false, toggle: this.toggle}
  render() {
    return (
      <ToggleContext.Provider value={this.state}>
        {this.props.children}
      </ToggleContext.Provider>
    )
  }
}

function Usage({
  onToggle = (...args) => console.log('onToggle', ...args),
}) {
  return (
    <Toggle onToggle={onToggle}>
      <Toggle.On>The button is on</Toggle.On>
      <Toggle.Off>The button is off</Toggle.Off>
      <div>
        <Toggle.Button />
      </div>
    </Toggle>
  )
}
```

# 結論

最後來對`Compound Component`做總結，這樣的設計方式能讓parent component開放一定程度的介面給children component使用，使得parent component跟children component使用上有一定程度的語意關聯，render的順序也可以保持彈性。但相反的，同時也綁死children component只能在parent component做使用。

如果要對children component更有彈性，比如檢查children component是否有在parent component的介面內，或者讓children component有多級層數，都需要更加複雜的檢查以及props傳遞的方式(Provider/Context)。

# Related articles

- [https://youtu.be/hEGg-3pIHlE](https://youtu.be/hEGg-3pIHlE)
- [https://plainenglish.io/blog/5-advanced-react-patterns-a6b7624267a6](https://plainenglish.io/blog/5-advanced-react-patterns-a6b7624267a6)
- [https://frontendmasters.com/courses/advanced-react-patterns/](https://frontendmasters.com/courses/advanced-react-patterns/)
- [https://blog.techbridge.cc/2018/06/27/advanced-react-component-patterns-note/](https://blog.techbridge.cc/2018/06/27/advanced-react-component-patterns-note/)
- [https://blog.techbridge.cc/2018/07/21/advanced-react-component-patterns-note-II/](https://blog.techbridge.cc/2018/07/21/advanced-react-component-patterns-note-II/)
- [https://kentcdodds.com/blog/answers-to-common-questions-about-render-props](https://kentcdodds.com/blog/answers-to-common-questions-about-render-props)
- [https://youtu.be/BcVAq3YFiuc](https://youtu.be/BcVAq3YFiuc)
- [https://codesandbox.io/s/github/kentcdodds/advanced-react-patterns-v2](https://codesandbox.io/s/github/kentcdodds/advanced-react-patterns-v2)
