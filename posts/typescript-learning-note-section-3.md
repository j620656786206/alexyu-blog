---
title: Typescript 學習筆記 - Section 3
description: 關於Typescript The Complete Developer's Guide的學習筆記, Section 3
date: 2023-03-14
tags:
  - typescript
  - frontend
layout: layouts/post.njk
---

- [Section Overview](#section-overview)
- [Type Annotation and inference](#type-annotation-and-inference)
  - [Plain Definition + Overview](#plain-definition--overview)
  - [Examples](#examples)
    - [The 'Any' Type](#the-any-type)
    - [Delayed Initialization](#delayed-initialization)
    - [When Inference Doesn't Work](#when-inference-doesnt-work)

## Section Overview

這章節主要會介紹Type Annotation跟Type Inference

## Type Annotation and inference
### Plain Definition + Overview

Type Annotation - 指的是我們告訴Typescript我們宣告的變數type是什麼
Type Inference - Typescript主動去判斷變數的type是什麼

### Examples

```ts
// apple's type is number
const apple: number = 5

// typescript will throw error
let apple: number = 5
apple = 'asdfdsfds'

let speed: string = 'fast'
let hasName: boolean = true

let nothingMuch: null = null
let nothing: undefined = undefined

// build in object
let now: Date = new Date()


// object literal annotations
let colors: string[] = ['red', 'green', 'blue']
let myNubmers: number[] = [1, 2, 3]
let truths: boolean[] = [true, true, false]

// Classes
class Car {}

const car: Car = new Car()

// object literal
let point: { x: number, y: number } = {
  x: 10,
  y: 20
}

// Function
const logNumber: (i: number) => void = (i: number) => {
  console.log(i)
}


// type annotation
// typescript will determine apple's type should be number
let apple = 5
```

```js
// const color is variable declaration
// 'red' is variable Initialization
const color = 'red'
```
如果宣告變數(declaration)以及初始化(initialization)都是在同一行程式碼，Typescript會自動判別color的tpye應該要是什麼，
所以在一般情況下我們可以使用type inference在任何地方，Type annotations只會用在三種情境
- 當我們宣告變數跟初始化變數不在同一行程式碼
- 當我們宣告的變數沒辦法被inferred
- 當function回傳'any' type，並且我們需要明確定義type

#### The 'Any' Type

```js
// When to use annotations
// 1) Functions that return the 'any' type
const json = '{"x": 10, "y": 20}'
const coordinates = JSON.parse(json)
console.log(coordinates) // {x: 10, y: 20}

// Typescript will not recognize the error
coordinates.dfgfdsgdfg
```

因為JSON.parse會回傳的type可能是number, string, object, etc..., 所以JSON.pars回傳的type是any
any在Typescript也是一種型別，Typescript並沒有辦法去檢查正確的property references
在任何情況下應該避免使用'any'

```js
// fix the any type
const json = '{"x": 10, "y": 20}'
const coordinates: { x: number, y: number } = JSON.parse(json)
console.log(coordinates) // {x: 10, y: 20}

// Typescript will throw the error
coordinates.dfgfdsgdfg
```

#### Delayed Initialization

```js
// 2) When we declare a variable on one line
// and initializate it later
let words = ['red', 'green', 'blue']
// change to foundWord: boolean
let foundWord

for(let i = 0; i < words.length; i++) {
  if(word[i] === 'green') {
    foundWord = true
  }
}
```

#### When Inference Doesn't Work

```js
// 3) Variable whose type cannot be inferred correctly
let numbers = [-10, -1, 12]
let numberAboveZero = false

for(let i = 0; i < numbers.length; i++) {
  if (number[i] > 0) {
    // typescript will throw error
    numberAboveZero = numbers[i]
  }
}
```

這邊可以修正為

```js
let numberAboveZero: boolean | number = false
```
