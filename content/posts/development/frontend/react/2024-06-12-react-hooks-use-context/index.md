+++
title = 'React hooks - useContext'
date = 2024-06-12T03:30:00+08:00
draft = false
categories = ['Frontend']
tags = ['react', 'hooks']
aliases = ['/posts/react/2024_06_12_react_hooks_use_context/', '/posts/react/2024_06_12_react_hooks_use_context/index.html']
+++

## 前言

痾…不小心跳過了整整 4 個 Hooks…~~幹~~

這篇是 useContext，請直接開始閱讀文章！

[官網](https://react.dev/reference/react/useContext)

## 文章架構

如上篇，我會按照官方文件的架構，首先講解定義，再用一些簡單的範例講解應該怎麼使用這些 Hooks，最後再講解一些容易犯錯的用例。

## 定義

這個 Hook 比較特別，是用來定義全域變數用的，像是網頁的暗黑/白色主題切換需要讀取一個統一的變數，就會使用到這個 Hook，在父元件中注入後就可以在子元件中取得 Context 的值。

```tsx
const SomeContext = createContext(initValue);
```

我們可以看到他的初始化分成兩個部分:

- 傳入
  - initValue: 傳入初始值，用來建立全域變數。
- 傳出
  - SomeContext: 用來將值注入元件的本體。

## 範例

以下是使用 useContext 的範例。

```tsx
import { createContext, useContext, useState } from 'react';

// initialization
const ThemeContext = createContext('light');

export default function MyApp() {
  const [theme, setTheme] = useState('light');
  return (
    <>
    // inject on parent component
      <ThemeContext.Provider value={theme}>
        <Form />
      </ThemeContext.Provider>
      <Button onClick={() => {
        setTheme(theme === 'dark' ? 'light' : 'dark');
      }}>
        Toggle theme
      </Button>
    </>
  )
}

function Form({ children }) {
  return (
    <Panel title="Welcome">
      <Button>Sign up</Button>
      <Button>Log in</Button>
    </Panel>
  );
}

function Panel({ title, children }) {
 // get value from child component
  const theme = useContext(ThemeContext);
  const className = 'panel-' + theme;
  return (
    <section className={className}>
      <h1>{title}</h1>
      {children}
    </section>
  )
}

function Button({ children, onClick }) {
 // get value from child component
  const theme = useContext(ThemeContext);
  const className = 'button-' + theme;
  return (
    <button className={className} onClick={onClick}>
      {children}
    </button>
  );
}
```

上面的文件是多個 Component 在同一個檔案裡，實務上我們比較常把每個 Component 切開來，劃分為不同檔案，所以記得要在 Parent Component 將 Context (ThemeContext) 導出，讓其他檔案可以導入並取得該值使用，以下是將上面的檔案分檔後的範例，請參考。

```tsx
// App.jsx
import { createContext, useContext, useState } from 'react';
import Button from 'Button'
import Form from 'Form'

// initialization
export const ThemeContext = createContext('light');

export default function MyApp() {
  const [theme, setTheme] = useState('light');
  return (
    <>
    // inject on parent component
      <ThemeContext.Provider value={theme}>
        <Form />
      </ThemeContext.Provider>
      <Button onClick={() => {
        setTheme(theme === 'dark' ? 'light' : 'dark');
      }}>
        Toggle theme
      </Button>
    </>
  )
}

// Form.jsx
function Form({ children }) {
  return (
    <Panel title="Welcome">
      <Button>Sign up</Button>
      <Button>Log in</Button>
    </Panel>
  );
}

// Panel.jsx
import { useContext } from 'react';
import { ThemeContext } from '../App'

function Panel({ title, children }) {
 // get value from child component
  const theme = useContext(ThemeContext);
  const className = 'panel-' + theme;
  return (
    <section className={className}>
      <h1>{title}</h1>
      {children}
    </section>
  )
}

// Button.jsx
import { useContext } from 'react';
import { ThemeContext } from '../App'

function Button({ children, onClick }) {
 // get value from child component
  const theme = useContext(ThemeContext);
  const className = 'button-' + theme;
  return (
    <button className={className} onClick={onClick}>
      {children}
    </button>
  );
}
```

## 注意事項

### 1. 請妥善設定 createContext 的預設值

沒有設定 Context 的情況下，萬一被 render 出來的元件並不在 provider 的子樹裡，這樣就會造成錯誤。

如下面的程式碼就可以看到 Button 這個元件並沒有放在 provider 中，所以當 Button 使用了設定成 null 的 ThemeContext 就會 render 不出正確的 Button。

```jsx
import { createContext, useContext, useState } from 'react';

// unproper usage
const ThemeContext = createContext(null);
// proper usage
// const ThemeContext = createContext("light");

export default function MyApp() {
  const [theme, setTheme] = useState('light');
  return (
    <>
      <ThemeContext.Provider value={theme}>
        <Form />
      </ThemeContext.Provider>
      <Button onClick={() => {
        setTheme(theme === 'dark' ? 'light' : 'dark');
      }}>
        Toggle theme
      </Button>
    </>
  )
}
```

### 2.  不同層級時取代 Provider

可以 provider 中再次使用 provider 注入不同的值 / 變數，可以更靈活的使用 Context。

```jsx
<ThemeContext.Provider value="dark">
  ...
  <ThemeContext.Provider value="light">
    <Footer />
  </ThemeContext.Provider>
  ...
</ThemeContext.Provider>
```

### 3. 透過 useMemo, useCallback 避免不必要的 re-render

當頁面觸發 re-render 時(像是路由更新等等)，注入 provider 的值或是物件更新時，就會造成有使用到 useContext 的那些元件一起 re-render，所以這裡可以使用 useMemo 或是 useCallback，來避免不必要的 re-render。

```tsx
import { useCallback, useMemo } from 'react';

function MyApp() {
  const [currentUser, setCurrentUser] = useState(null);

  const login = useCallback((response) => {
    storeCredentials(response.credentials);
    setCurrentUser(response.user);
  }, []);

  const contextValue = useMemo(() => ({
    currentUser,
    login
  }), [currentUser, login]);

  return (
    <AuthContext.Provider value={contextValue}>
      <Page />
    </AuthContext.Provider>
  );
}
```

### 4. 在正確的地方使用 useContext

以下會有三個狀況會讓你無法正常使用 useContext。

- 在同一個元件裡同時使用了 createContext 以及 useContext。
  - 請將使用到 useContext 的區塊分到另外一個檔案，或是不要在這樣的狀況中使用 useContext。
- 沒有將使用到 useContext 的元件放在 provider 的子樹中。
  - 請將該元件放進子樹中。
- 因為打包工具的問題，造成 useContext 取得不同的物件。
  - 要檢查是否發生這個問題，請將 Context 放在全域的兩個變數中，像是 window.SomeContext1 和 window.SomeContext2，再判斷兩個變數是否相等 (===)。

### 5. 總是拿到 undefined

下面兩個狀況會讓你拿到 undefined，請正確使用 provider。

```jsx
// 🚩 1. Doesn't work: no value prop
<ThemeContext.Provider>
   <Button />
</ThemeContext.Provider>

// 🚩 2. Doesn't work: prop should be called "value"
<ThemeContext.Provider theme={theme}>
   <Button />
</ThemeContext.Provider>

// ✅ Passing the value prop
<ThemeContext.Provider value={theme}>
   <Button />
</ThemeContext.Provider>
```

## 結語

useContext 其實蠻好理解及使用的，只要注意不要犯以上列出的常見錯誤，就可以好好的使用這個 hook。
