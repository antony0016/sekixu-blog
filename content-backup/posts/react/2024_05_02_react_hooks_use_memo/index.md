+++
title = 'React hooks - useMemo'
date = 2024-05-02T13:16:32+08:00
draft = false
categories = ['Frontend']
tags = ['react', 'hooks']
+++
## 前言

接續前一篇，這篇是 useMemo，請直接開始閱讀文章！

[官網](https://react.dev/reference/react/useMemo)

## 文章架構

如上篇，我會按照官方文件的架構，首先講解定義，再用一些簡單的範例講解應該怎麼使用這些 Hooks，最後再講解一些容易犯錯的用例。

## 定義

用來快取常常需要重新計算的狀態，如果有用過 vue.js 的 computed，應該很容易上手這個 hook。

```tsx
const cachedValue = useMemo(calculateValue, dependencies)
```

我們可以看到他分成三個部分:

- 傳入
  - calculateValue: 傳入一個 function，用來計算出每次的結果並回傳。
  - dependencies: 用來關注那些變動了就應該重新計算狀態的變數。
- 傳出
  - cachedValue: 最新的計算好的 state，和其他 hook 提供的一樣都是 read only。

## 範例

以下是使用 useMemo 的範例。

```tsx
// without useMemo
// component need to recalculate todo list,
// even props are the same.
export default function TodoList({ todos, tab, theme }) {
  // ...
  return (
    <div className={theme}>
      <List items={filterTodos(todos, tab)} />
    </div>
  );
}

// with useMemo
// when props are the same,
// component will use cached todos from latest render. 
import { useMemo } from 'react';

function TodoList({ todos, tab }) {
  const visibleTodos = useMemo(
    () => filterTodos(todos, tab),
    [todos, tab]
  );
  // ...
}

export default function TodoList({ todos, tab, theme }) {
  // ...
  return (
    <div className={theme}>
      <List items={visibleTodos} />
    </div>
  );
}
```

由於父元件重新渲染時，也會同時將所有子元件重新渲染，如果想要避免大量重新渲染時，有兩種方式，用 useMemo 把元件先計算好之後存起來(block 1)，或是用 memo 做個別元件的快取，這樣可以更簡單寫出想要的效果(block 2)。

```tsx
// block 1
// 
export default function TodoList({ todos, tab, theme }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  const children = useMemo(() => <List items={visibleTodos} />, [visibleTodos]);
  return (
    <div className={theme}>
      {children}
    </div>
  );
}

// block 2
import { memo } from 'react';

export default function TodoList({ todos, tab, theme }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  return (
    <div className={theme}>
      <List items={visibleTodos} />
    </div>
  );
}

const List = memo(function List({ items }) {
  // ...
});
```

## 注意事項

### 1. 不要在 useMemo 裡面更新 dependencies

因為 useMemo 會關注 dependencies，所以如果在 useMemo 裡面更新 dependencies 的話，就會循環觸發 useMemo

```tsx
// wrong usage
const visibleTodos = useMemo(() => {
  // 🚩 Mistake: mutating a prop
  todos.push({ id: 'last', text: 'Go for a walk!' });
  const filtered = filterTodos(todos, tab);
  return filtered;
}, [todos, tab]);

// correct usage
const visibleTodos = useMemo(() => {
  const filtered = filterTodos(todos, tab);
  // ✅ Correct: mutating an object you created during the calculation
  filtered.push({ id: 'last', text: 'Go for a walk!' });
  return filtered;
}, [todos, tab]);
```

### 2. 在對的地方使用 useMemo

請不要在迴圈中使用 useMemo，如果要在這種場景下使用，請參考下面的修正方式。

```tsx
// use useMemo in a loop
function ReportList({ items }) {
  return (
    <article>
      {items.map(item => {
        // 🔴 You can't call useMemo in a loop like this:
        const data = useMemo(() => calculateReport(item), [item]);
        return (
          <figure key={item.id}>
            <Chart data={data} />
          </figure>
        );
      })}
    </article>
  );
}

// fix solution
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
  // ✅ Call useMemo at the top level:
  const data = useMemo(() => calculateReport(item), [item]);
  return (
    <figure>
      <Chart data={data} />
    </figure>
  );
}
```

## 結語

 這個 hook 挺簡單的，但是也是常常會用到的一種，需要好好記住使用場景。
