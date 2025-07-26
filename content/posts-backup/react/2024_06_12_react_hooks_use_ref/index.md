+++
title = 'React hooks - useRef'
date = 2024-06-12T20:04:00+08:00
draft = false
categories = ['Frontend']
tags = ['react', 'hooks']
+++
## 前言

這篇是 useRef，請直接開始閱讀文章！

[官網](https://react.dev/reference/react/useRef)

## 文章架構

如上篇，我會按照官方文件的架構，首先講解定義，再用一些簡單的範例講解應該怎麼使用這些 Hooks，最後再講解一些容易犯錯的用例。

## 定義

這個 Hook 的用途是避免元件中不需要重新 render 的部分被重新 render。

```tsx
const ref = useRef(initialValue)
```

我們可以看到他的初始化分成兩個部分:

- 傳入
  - initialValue: 初始值，會傳入 useRef 的 current 給元件使用。
- 傳出
  - ref: useRef 會回傳只有一個屬性(current)的物件，current 的預設值就是我們剛剛傳入的 initialValue。
    - ref 就是一個單純的 javascript 的物件，這個變數並不會被 React 單向資料流所監看，所以也不會因為這個物件內中的變動而觸發 re-render。
    - 使用：要使用 ref 就直接讀取 / 修改 ref.current 就可以了，不需要像 useState 那樣使用 get / set 的模式。

## 範例

以下是使用 useRef 的範例。

```tsx
import { useRef } from 'react';

function MyComponent() {
  const intervalRef = useRef(0);
  
  const handleStartClick = () => {
   const intervalId = setInterval(()=>{
    // ...
   }, 1000)
   intervalRef.current = intervalId
  }
  
  const handleStopClick = () => {
   const intervalId = intervalRef.current
   clearInterval(intervalId)
  }
  // ...
}
```

官方給了幾個 use case，像是紀錄 setInterval 的 Interval ID(如上)、綁定 DOM 進行操作以及避免特定物件的 recreating。

```tsx
// Use case 1. Record interval id, 
// after re-render will not loss interval within using useRef.
import { useRef } from 'react';

function MyComponent() {
  const intervalRef = useRef(0);
  
  const handleStartClick = () => {
   const intervalId = setInterval(()=>{
    // ...
   }, 1000);
   intervalRef.current = intervalId;
  }
  
  const handleStopClick = () => {
   const intervalId = intervalRef.current;
   clearInterval(intervalId);
  }
  // ...
}

// Use case 2. Manipulating the DOM with a ref, 
// ref.current will combine the DOM you chose,
// so you can manipulate the DOM easily.
import { useRef } from 'react';

function MyComponent() {
  const inputRef = useRef(null);
  
  const handleClick = () => {
   // inputRef.current equals the input DOM below.
   inputRef.current.focus();
  }
  
  return (
   <input ref={inputRef} />
  );
}

// Use case 3. Avoiding recreating the ref contents.
// ref will return the same object after re-render,
// This can avoid to recreating multiple times.
import { useRef } from 'react';

function MyComponent() {
  // new VideoPlayer() only called for first initial render,
  // but on another render still call this function to create new Object,
  // this way may cause memory leak.
  const playerRef = useRef(new VideoPlayer());
  
  // ...
}

function MyComponent() {
  const playerRef = useRef(null);
  // To prevent multiple object creation,
  // you can use singleton pattern to instead.
  if (playerRef.current === null) {
    playerRef.current = new VideoPlayer();
  }
  // ...
}
```

## 注意事項

### 1. 父元件綁定子元件內的 DOM

```jsx
const inputRef = useRef(null);

return <MyInput ref={inputRef} />;
```

在上面的情景，不做任何處理直接將 ref 綁上這個子元件，React 會警告你存取這個 ref 的時候會失敗，原因是在正常狀況下，元件是不會將內部的 DOM 暴露給外面存取的，只能透過 props，所以我們應該使用警告中會提示你使用的 React.forwardRef()。

透過下方的 Code 我們就可以直接在父元件透過 ref 的 props 來綁定到子元件的 input 了，可喜可賀，可喜可賀 XD

```jsx
// Original MyInput.jsx
export default function MyInput({ value, onChange }) {
 // This input will not exposed to other components.
 // so you can't bind ref by this way.
  return (
    <input
      value={value}
      onChange={onChange}
    />
  );
}

// Fixed MyInput.jsx
import { forwardRef } from 'react';

// Create a new called ref
const MyInput = forwardRef(({ value, onChange }, ref) => {
 // and use ref prop to bind your DOM,
 // so you can bind this DOM out of this component.
  return (
    <input
      value={value}
      onChange={onChange}
      ref={ref}
    />
  );
});

export default MyInput;
```

## 結語

useRef 也是一個相當重要的 hook，在當初寫 Vue 的時候也有透過類似的方式操作 DOM，但是在這裡釐清了更多使用上的細節，真的是相當好的官方文件，想學好 React 的各位請好好閱讀官方文件。
