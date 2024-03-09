+++
title = '在 Vite 專案中簡化複雜的 import path'
date = 2023-12-22T21:03:49+08:00
draft = false
categories = ['Frontend']
tags = ['vue', 'vite']
+++

## 目的

在以前的版本中曾經使用過以 @ 代替 `./src` 的語法，但是在新的版本突然不支援了，於是去查了一下該如何加到專案中。

## 方法

需要在兩個檔案中建立 alias，分別是 `tsconfig.json` 和 `vite.config.ts`。

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
  // 省略...
  // 重點
    "paths": {
      "@/*": [
        "./src/*"
      ]
    }
  // 重點
  }
},
"include": ["src/**/*.ts", "src/**/*.tsx", "src/**/*.vue"],
"references": [{"path": "./tsconfig.node.json"}]
}
```

```tsx
// vite.config.ts
export default defineConfig({
    plugins: [vue()],
  // 重點
    resolve: {
        alias: {
            '@': '/src'
        }
    }
  // 重點
})
```

## 如何 import

```tsx
// ts file
import {router} from "@/router.ts";

// component
import App from '@/App.vue'
```

## 結語

真的好方便好直觀，喜歡😀
