+++
title = 'React hooks - useState'
date = 2024-05-02T06:45:00+08:00
draft = false
categories = ['Frontend']
tags = ['react', 'hooks']
aliases = ['/posts/react/2024_05_02_react_hooks_use_state/', '/posts/react/2024_05_02_react_hooks_use_state/index.html']
+++

## 前言

最近正在學習 React，在我學這套框架之前，早就聽過很多人被 Hooks 搞得一個頭兩個大了，於是決定一邊學習一邊把重點寫下來，避免我之後又突然忘記了 XDD

[官網](https://react.dev/reference/react/useState)

## 文章架構

這篇會按照官方文件的架構，首先講解定義，再用一些簡單的範例講解應該怎麼使用這些 Hooks，最後再講解一些容易犯錯的用例。

定義

…

範例

…

易錯用例

…

## 定義

第一個是 useState，用於簡單儲存元件中的狀態使用。

```tsx
const [state, setState] = useState(initialState)
```

我們可以看到他分成三個部分:

- initState: 預設狀態，就是這個 State 的預設值。
- state: state 就是你儲存的值，一開始會和你的 initState 是一樣的值，注意所有 State 都是唯讀的，要改變你的 state 就使用 setState。
- setState: 用來改變 state，由於 React 採用的是單向資料流，不像 vue 是雙向資料流，可以直接更改 state，所以要透過 setState，告訴 react 你改了這個 state，請幫我更新到畫面上。

## 範例

以下是一些使用 useState 的範例。

```tsx
import { useState } from 'react';

function MyComponent() {
 // init
  const [age, setAge] = useState(28);
  const [name, setName] = useState('Taylor');
  const [todos, setTodos] = useState(() => createTodos());
  
  // usage
  // age: 28
  // handleAgeChange(30)
  // age: 30
  const handleAgeChange = (newAge) => {
   setAge(newAge)
  }
  // name: 'Taylor'
  // handleNameChange('John')
  // age: 'John'
  const handleNameChange = (newName) => {
   setName(newName)
  }
  // todos: [{todo1}, {todo2}]
  // handleTodosChange({todo3})
  // todos: [{todo1}, {todo2}, {todo3}]
  const handleTodosChange = (newTodo) => {
   setTodos([...todos, newTodo])
  }
}
```

useState 也有另一個用法，如果像是計數用的 state 也可以直接傳入一個函式， useState 會傳入一個參數(state)讓你使用，就可以透過使用這個參數計算後回傳就可以達到更新的效果，以下是範例。

```tsx
import { useState } from 'react';

function MyComponent() {
 // init
  const [counter, setCounter] = useState(0);
  
  // usage
  // counter: 0
  // counterIncrease()
  // counter: 1
  const counterIncrease = () => {
   setCounter(prevCounter => prevCounter + 1)
  }
}
```

## 注意事項

### 1. state 更新問題

剛剛在範例的第二個用法中，講到了可以傳入一個函式更新 state 的方式，這是最安全的使用方式，因為如果使用第一個用法的話，會遇到下面這個問題。

```tsx
import { useState } from 'react';

function MyComponent() {
 // init
  const [counter, setCounter] = useState(0);
  
  // usage
  const counterIncrease2 = () => {
   setCounter(counter + 1)
   setCounter(counter + 1)
  }
  // counter: 0
  // counterIncrease2()
  // counter: 1
  const showCounter = () => {
   console.log(counter)
   counterIncrease2()
   console.log(counter)
  }
}
```

### 2. 將函數儲存於 state 中

在 useState 的範例中沒有提到 function as state 的用法，但是如果你需要這樣用的話請多注意，由於有將 function 傳入 useState 與 setState 作為初始函式與更新函式的用法，所以按照直覺的寫法會出問題的，請參考以下範例XDD

```tsx
// wrong case
const [fn, setFn] = useState(someFunction);

function handleClick() {
  setFn(someOtherFunction);
}

// correct case
const [fn, setFn] = useState(() => someFunction);

function handleClick() {
  setFn(() => someOtherFunction);
}
```

## 結語

 React 的官方文件寫得有夠清楚的XD，不是說 Vue 的官方文件不清楚，但是 React 提供了更多 use case 甚至 trouble shooting，真的很好理解用法，並且避免了未來的潛在錯誤，但是目前簡單看了一下，至少還有 7 個 hook 要寫…，加油吧…
