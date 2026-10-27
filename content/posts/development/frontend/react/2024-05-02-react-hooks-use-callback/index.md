+++
title = 'React hooks - useCallback'
date = 2024-05-02T16:12:12+08:00
draft = false
categories = ['Frontend']
tags = ['react', 'hooks']
aliases = ['/posts/react/2024_05_02_react_hooks_use_callback/', '/posts/react/2024_05_02_react_hooks_use_callback/index.html']
+++
## 前言

接續前一篇，這篇是 useCallback，請直接開始閱讀文章！

[官網](https://react.dev/reference/react/useCallback)

## 系列文章

React hooks 系列的其他文章在 [hooks 標籤](/tags/hooks/)。

## 定義

用來快取會根據輸入值有不同定義的 function，基本上可以想像成用來快取 function 的 useMemo。

```tsx
const cachedFn = useCallback(fn, dependencies)
```

我們可以看到他分成三個部分:

- 傳入
  - fn: 傳入一個 function 架構，裡面包含會變動的元素。
  - dependencies: 用來比較函式是否有變動了的變數陣列。
- 傳出
  - cachedFn: 最新的計算好的 function，和其他 hook 提供的一樣都是 read only。

## 範例

以下是使用 useCallback 的範例。

```tsx
// ProductPage.js
import { useCallback } from 'react';

function ProductPage({ productId, referrer, theme }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);
  // ... 
  return (
    <div className={theme}>
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}

// ShippingForm.js
import { memo } from 'react';

const ShippingForm = memo(function ShippingForm({ onSubmit }) {
  // ...
});
```

由於父元件重新渲染時，也會同時將所有子元件重新渲染，如果想要避免大量重新渲染時，可以多做一步，在會使用到快取起來的 function 的元件時，使用 memo 讓物件知道這是重用同樣 function 的物件不需要重新 render。

原理:

如果單用 memo 不使用 useCallback 的話，每次父元件只要重新渲染，即使 handleSubmit 的內容都還是一樣的，handleSubmit 都還是會被重新產生，讓子元件的 memo 判定是不同 function 觸發子元件的重新渲染，所以一定要搭配著使用。

```tsx
// 只用 memo、沒有 useCallback
function ProductPage({ productId, referrer, theme }) {
  // 每次 render 都會產生新的 handleSubmit
  function handleSubmit(orderDetails) {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }
  return (
    <div className={theme}>
      {/* 🔴 onSubmit 每次都是新的 function，memo 包起來的 ShippingForm 還是會重新 render */}
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}
```

## 注意事項

### 1. 應該大量使用 useCallback 嗎

不應該，以下是 React 給出的原因:

- 在不需要 cache function 時使用 useCallback 只會影響可讀性，對於性能沒有任何幫助。

於是 React 也提出了如何減少 / 正確使用 useCallback 的原則：

1. 當你的元件被包起來的時候，請使用[這個技巧](https://react.dev/learn/passing-props-to-a-component#passing-jsx-as-children)，來讓你的父元件知道只有他自己需要被更新。
2. 請優先以狀態本地化下去設計你的元件，如果是像表單這類的元件，就請不要將 state 或是 function 向上提升到父元件注入([像這樣](https://react.dev/learn/sharing-state-between-components))，甚至是 Dom tree root 或是全域性的狀態。
3. [保持簡潔的渲染邏輯](https://react.dev/learn/keeping-components-pure)，如果在渲染的過程中出現了問題，那請優先解決問題 === bug，而不是考慮使用 useCallback，useCallback 是用來增進效能而不是解決 bug <== 這句我加的XD。
4. [避免不必要的手續](https://react.dev/learn/you-might-not-need-an-effect)，大部分的效能問題都是因為多做了許多不需要的事情，因此多渲染了好幾次或是多計算了好幾次。
5. [將不需要的 Dependencies 移出你的 useMemo / useCallback](https://react.dev/learn/removing-effect-dependencies)，這兩個都是根據我們提供的 Dependencies 進行 state / function 的重新渲染，所以請好好思考更新時機並放入應該被比較的變數。

### 2. 每次 useCallback 都回傳了新的 function(re-render)

請檢查一下是否有好好將比較變數填入 Dependencies 中，否則函式中有沒被用來比叫就會傳回不同的 function(block 1)，或是你的變數本來就一直在變動，那麼 useCallback 也就當然每次都會產出不一樣的 function(block 2)。

```tsx
// block 1
// wrong case
function ProductPage({ productId, referrer }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }); // 🔴 Returns a new function every time: no dependency array
  // ...
}

// correct case
function ProductPage({ productId, referrer }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]); // ✅ Does not return a new function unnecessarily
  // ...
}

// block 2
// function component
function ProductPage({ productId, referrer }) {
 const handleSubmit = useCallback((orderDetails) => {
   // ..
 }, [productId, referrer]);
 
 console.log([productId, referrer]);
}

// console
Object.is(temp1[0], temp2[0]); // Is the first dependency the same between the arrays?
Object.is(temp1[1], temp2[1]); // Is the second dependency the same between the arrays?
Object.is(temp1[2], temp2[2]); // ... and so on for every dependency ...
```

### 3. 在正確的地方使用 useCallback

在這個範例中在 ReportList 包裝 function 注入到 chart 犯了元件的設計問題，應該將零散的元件包成 Report 元件，並且讓 Report 包裝好自己的狀態與函式，才是 useCallback 的正確用法。

```tsx
// wrong case
function ReportList({ items }) {
  return (
    <article>
      {items.map(item => {
        // 🔴 You can't call useCallback in a loop like this:
        const handleClick = useCallback(() => {
          sendReport(item)
        }, [item]);

        return (
          <figure key={item.id}>
            <Chart onClick={handleClick} />
          </figure>
        );
      })}
    </article>
  );
}

// correct case
function ReportList({ items }) {
  return (
    <article>
      {items.map(item =>
        <Report key={item.id} item={item} />
      )}
    </article>
  );
}

function Report({ item }) {
  // ✅ Call useCallback at the top level:
  const handleClick = useCallback(() => {
    sendReport(item)
  }, [item]);

  return (
    <figure>
      <Chart onClick={handleClick} />
    </figure>
  );
}

```

## 結語

useCallback 比想像中的更複雜，或者可以說 React 提醒了我們更多注意事項，由於在沒有好好理解這些 Hooks 的狀況下使用不只沒有好處，更可能造成奇怪的錯誤，所以要好好遵照 React 提供的方法使用，才可以兼顧可維護性與可讀性。
