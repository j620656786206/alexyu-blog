---
title: Typescript 學習筆記 - Section 5
description: 關於Typescript The Complete Developer's Guide的學習筆記, Section 5
date: 2023-05-25
tags:
  - typescript
  - frontend
layout: layouts/post.njk
---

- [Section Overview](#section-overview)
- [Arrays in Typescript](#arrays-in-typescript)
  - [Examples](#examples)
- [Why Typed Array?](#why-typed-array)
  - [Examples](#examples-1)
- [Multiple Types in Arrays](#multiple-types-in-arrays)
- [When to Use Typed Arrays](#when-to-use-typed-arrays)

## Section Overview

這章節主要會介紹如何使用 Typed Arrays

## Arrays in Typescript

定義: 在Typescript的世界裡, array裡面的element應該都維持一定程度上的相同型別

### Examples

```ts
// typescript inference const carMakers: string[]
const carMakers = ['ford', 'toyata', 'chevy']

// or typescript annotation
const carMakers: string[] = ['ford', 'toyata', 'chevy']
const carMakers: string[] = []

// typescript inference const carMakers: any[]
const carMakers = []

// const dates: Date[]
const dates = [new Date(), new Date()]

// const carsByMake: string[][]
const carsByMake = [
  ['f150'],
  ['corolla'],
  ['camaro'],
]
```

## Why Typed Array?
- Typescript(TS)可以幫助我們判斷從array取出來的值是什麼型別
- TS可以幫助我們將錯誤型別的值加到array裡
- 當在使用array.map, 'forEach', 'reduce'的時候TS可以幫助我們判斷型別
- array仍有一定的彈性加入不同型別的值

### Examples
```ts
// Help with inference when extracting values
// const car: string
const car = carMakers[0]
// const myCar: string
const myCar = carMakers.pop()

// Prevent incompatible values
// get ts lint error
carMakes.push(100)

// Help with 'map'
carMakes.map((car: string): string => {
  // get methods belongs to string
  return car
})
```


## Multiple Types in Arrays

```ts
// Flexible types
const importantDates: (Date | string)[] = [new Date()]
importantDates.push('2030-10-10')
importantDates.push(new Date())
// get ts lint error
importantDates.push(100)
```

## When to Use Typed Arrays
Anytime~
