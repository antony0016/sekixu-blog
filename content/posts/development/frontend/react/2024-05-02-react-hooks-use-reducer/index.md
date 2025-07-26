+++
title = 'React hooks - useReducer'
date = 2024-05-02T10:45:03+08:00
draft = false
categories = ['Frontend']
tags = ['react', 'hooks']
aliases = ['/posts/react/2024_05_02_react_hooks_use_reducer/', '/posts/react/2024_05_02_react_hooks_use_reducer/index.html']
+++

## 前言

接續前一篇，這篇是 useReducer，請直接開始閱讀文章！

[官網](https://react.dev/reference/react/useReducer)

## 文章架構

如上篇，我會按照官方文件的架構，首先講解定義，再用一些簡單的範例講解應該怎麼使用這些 Hooks，最後再講解一些容易犯錯的用例。

## 定義

當元件中具有相當複雜的邏輯，用 useState 會影響你的元件的可維護性，就可以使用 useReducer 來提升元件的可維護性。

```tsx
const [state, dispatch] = useReducer(reducer, initialArg, init?)
```

我們可以看到他分成五個部分:

- 傳入
  - reducer: 是一個函式，用來定義如何更新 state，有兩個參數分別是 state 和 action，reducer 會透過我們傳入的 action 來更新 state。
  - initArg: 和 useState 中的 initState 一樣，用來設定一個或多個 state。
  - init?: 這是用來產生預設 state 的函式，這個函式會被傳入 initArg，用來給 init 函式使用，這個函式回傳才會是最終的 initArg。(可能有點難懂，等等用範例解釋!**)**
- 傳出
  - state: state 就是你儲存的值，一開始會和你的 initArg 是一樣的值，注意這裡和 useState 的 state 一樣都是唯讀的，要改變你的 state 就 dispatch 一個 action。
  - dispatch: 用 action 來觸發 reducer 更新 state。

## 範例

以下是一些使用 useReducer 的範例。

```tsx
// reducer function
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      return {
        name: state.name,
        age: state.age + 1
      };
    }
    case 'changed_name': {
      return {
        name: action.nextName,
        age: state.age
      };
    }
  }
  throw Error('Unknown action: ' + action.type);
}

// function component
function Form() {
  const [state, dispatch] = useReducer(reducer, { name: 'Taylor', age: 42 });
  
  function handleButtonClick() {
    dispatch({ type: 'incremented_age' });
  }

  function handleInputChange(e) {
    dispatch({
      type: 'changed_name',
      nextName: e.target.value
    });
  }
  // ...
}
```

## 注意事項

### 1. initArg with init function

這裡講解 initArg 和 init function 的關係，以下範例提供參考。

```tsx
// without init function
// function component
function Form() {
  const [state, dispatch] = useReducer(reducer, [
   {id: 0, name: 'John1'}, 
   {id: 1, name: 'John2'}, 
   {id: 2, name: 'John3'},
   // ... and other 47 elements
 ]);
  
  // ...
}

// with init function
// init function
function createInitState(name) {
 let initList = []
 for(let i = 0;i < 50;i++){
  initList.push({
   id: i,
   name: name + i.toString(),
  })
 }
 return initList;
}

// function component
function Form() {
  const [state, dispatch] = useReducer(reducer, 'John', createInitState);
  // init state will be a list with 50 elements 
  // ...
}
```

### 2. 注意 state 的更新時機

和 useState 一樣，在同一個函式中 dispatch 並不會馬上更新 state，會拿到一樣的結果(請參考  block 1)，如果真的需要馬上得到更新後的結果，請參考 block 2。

```tsx
// block 1
function handleClick() {
  console.log(state.age);  // 42

  dispatch({ type: 'incremented_age' }); // Request a re-render with 43
  console.log(state.age);  // Still 42!

  setTimeout(() => {
    console.log(state.age); // Also 42!
  }, 5000);
}

// block 2
function handleClickWithGuessNextState() {
 const action = { type: 'incremented_age' };
 dispatch(action);
 
 const nextState = reducer(state, action);
 console.log(state);     // { age: 42 }
 console.log(nextState); // { age: 43 }
}
```

### 3. 注意 reducer 中的回傳值

請注意 reducer 中的每個 return 都要回傳完整的 state，如果只有回傳部分的 state，那麼在該次的 reducer 執行完畢後，就會只剩下當次回傳的 state 內容。

```tsx
// reducer function
// state: {
//   name: 'John',
//   age: 18
// }
// 
// reducer with wrong return
// 1. dispatch({type: 'incremented_age'})
// state: {
//   age: 19
// }
// 
// 2. dispatch({type: 'changed_name', nextName: 'annie'})
// state: {
//   name: 'annie'
// }
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      return {
        age: state.age + 1
        // missing name
      };
    }
    case 'changed_name': {
      return {
       // missing age
        name: action.nextName,
      };
    }
  }
  throw Error('Unknown action: ' + action.type);
}
```

## 結語

 一開始看稍稍有點看不懂，但官網一樣給了很多範例參考，又多掌握了一個 hook，希望趕快寫完剩下的開始嘗試運用這些 hooks 到我的專案裡！
