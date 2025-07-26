+++
title = 'React hooks - useEffect'
date = 2024-07-18T22:13:00+08:00
draft = false
categories = ['Frontend']
tags = ['react', 'hooks']
+++
## 前言

這篇是 useEffect，在這個 Hook 寫完之後就會直接進入實戰了，話不多說，請直接開始閱讀文章！

[官網](https://react.dev/reference/react/useEffect)

## 文章架構

如上篇，我會按照官方文件的架構，首先講解定義，再用一些簡單的範例講解應該怎麼使用這些 Hooks，最後再講解一些容易犯錯的用例。

## 定義

這個 Hook 的用途是當你需要與 React 外部的東西互動時，將這些外部互動包成一組 “Effect”，用來將切開元件與外部系統的互動，並且避免不穩定的多次渲染。

```tsx
useEffect(setup, dependencies?)
```

我們可以看到他的初始化分成兩個部分:

- 傳入
  - setup: 傳入一個 function，與外部系統互動的細節，包含當系統結束時的後處理，為一組完整的 Effect。
  - dependencies: 可忽略，當需要時傳入一組 Array，做為這組 Effect 的依賴，當依賴的值改變時，就會觸發後處理，並且重新初始化，互動 / 連結外部系統。

## 範例

以下是使用 useEffect 的範例。

```tsx
import { useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
   // Effect setup function
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    // 後處理 function
    return () => {
      connection.disconnect();
    };
    // dependencies
  }, [serverUrl, roomId]);
  // ...
}
```

官方給了幾個 use case，像是連接到聊天室 server、監聽全域事件(window: pointermove)、觸發動畫、控制對話框以及控制觀察者模式等等，以下以連接到聊天室 server 與監聽全域事件。

```tsx
// use case 1.
// connnect to chatroom server.
import { useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
 // 將 serverUrl 設為 state，就可以作為 dependencies 被 useEffect 監聽
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');
  
  useEffect(() => {
   // setup function: create connection
   const connection = createConnection(serverUrl, roomId);
    connection.connect();
    // clean up function: connection disconnect. 
   return () => {
      connection.disconnect();
   };
   // dependencies: when serverUrl or roomId changed, 
   // do clean up first then setup again
  }, [serverUrl, roomId]);
  // ...
}

// use case 2.
// Listening to a global browser event.
import { useState, useEffect } from 'react';

export default function App() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
   // setup function: combine a move cursor handler to listen to window event.
    function handleMove(e) {
      setPosition({ x: e.clientX, y: e.clientY });
    }
    window.addEventListener('pointermove', handleMove);
    // clean up function: remove window event listener.
    return () => {
      window.removeEventListener('pointermove', handleMove);
    };
    // no dependencies: so setup and clean up only happend on component render.
  }, []);
  // ...
}
```

## 注意事項

### 1. 依賴的使用方式 其一

根據傳入的依賴陣列，useEffect 會有不同的行為，這裡總共有三種，分別為有傳入依賴陣列、空依賴陣列和不傳入參數。

```jsx
// situation 1: with dependencies
useEffect(() => {
  // ...
}, [a, b]); // Runs again if a or b are different

// situation 2: empty dependencies array
useEffect(() => {
  // ...
}, []); // Does not run again (except once in development)

// situation 3: no dependencies
useEffect(() => {
  // ...
}); // Always runs again
```

### 2. 依賴的使用方式 其二

不要使用 Effect 中會使用的 State 作為依賴傳入 useEffect 中，Effect 更新 State，State 更新 Effect 不斷循環，也就造成不斷執行 useEffect 中的程式碼。

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const intervalId = setInterval(() => {
      setCount(count + 1); // You want to increment the counter every second...
    }, 1000)
    return () => clearInterval(intervalId);
  }, [count]); // 🚩 ... but specifying `count` as a dependency always resets the interval.
  // ...
}
```

### 3. 依賴的使用方式 其三

別傳入不需要的參數或是物件作為依賴，才不會意外觸發 useEffect。

像是在下方的這個範例中，就不需要將 serverUrl 和 roomId 包起來，serverUrl 不會變動，只需要傳入 roomId 作為依賴觸發 useEffect 就好了。

```jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  const options = { // 🚩 This object is created from scratch on every re-render
    serverUrl: serverUrl,
    roomId: roomId
  };

  useEffect(() => {
    const connection = createConnection(options); // It's used inside the Effect
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // 🚩 As a result, these dependencies are always different on a re-render
  // ...
```

### 4. 依賴的使用方式 其四

不要使用函式作為依賴項傳入，因為每次都會產生不一樣的值，會一直觸發 useEffect，所以請不要使用函式作為依賴項傳入。

```jsx
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  function createOptions() { // 🚩 This function is created from scratch on every re-render
    return {
      serverUrl: serverUrl,
      roomId: roomId
    };
  }

  useEffect(() => {
    const options = createOptions(); // It's used inside the Effect
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, [createOptions]); // 🚩 As a result, these dependencies are always different on a re-render
  // ...
```

### 5. 當元件掛載時 useEffect 都會執行兩次

React 會在使用 Strict Mode 開發的時候，多執行一次 useEffect 用於檢查你的 useEffect 有沒有正確定義，以凸顯特定錯誤。

### 6. 每次重新渲染 useEffect 都會執行 / 無窮迴圈

請參考依賴的使用方式之一及之二，正確設定依賴就可以避免這些問題。

## 結語

如果是碰過 Vue 的朋友們就可以感覺到，React 的官方說明文件提到的這些有的沒的錯誤，或是注意事項，在寫 Vue 的同時也有遇到過，只要再繼續深挖這些錯誤，就可以知道我們要做的事將 JS 的特性學好，就可以很直覺地避開這些詭異的錯誤了。

寫完了這些 Hooks 之後，再來我就會直接寫一個專案將這些 Hooks 運用在專案中了，期待 React 為我帶來與 Vue 不同的開發體驗！
