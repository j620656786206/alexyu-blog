---
title: Typescript 學習筆記 - Section 2
description: 關於Typescript The Complete Developer's Guide的學習筆記, Section 2
date: 2023-02-01
tags:
  - typescript
  - frontend
layout: layouts/post.njk
---

- [Course Overview](#course-overview)
- [Types](#types)
  - [Plain Definition + Overview](#plain-definition--overview)
  - [Why do we care?](#why-do-we-care)
  - [When to use this?](#when-to-use-this)

## Course Overview

整堂課程的前半段會著重在typescript的語法syntax跟feature介紹，後面才會介紹如何使用typescript，並且應用在design pattern

## Types
### Plain Definition + Overview

這邊作者給出的定義是
> Easy way to refer to the different properties + function that a value has

value可以是string, object, array, etc..., 比如說"red", 他的type屬於string, 所以他會有string的prototype所有的function可以使用, 比如`includes()`, `charAt()`, `indexOf()`, etc..., 當我們說一個值的type是string的時候, 很自然的便知道這個值有屬於string proptype的function可以使用。舉一反三, 假設我們定義好了一個type `Todo`, 與其在程式碼裡面說某個值有`Todo`的function, properties, 倒不如簡稱說這個值的type是`Todo`就好

### Why do we care?

- 有了定義好的type，當我們在寫程式碼時如果呼叫了不屬於這個type的function或者properties的時候，Typescript compiler可以幫我們找出這種錯誤
- 定義value的type可以幫助其他工程師了解整個資料的flow與codebase

### When to use this?

答案是everywhere😂
