+++
title = '隨手筆記 - Vue3 with Vite 專案初始化'
date = 2023-12-23T22:14:03+08:00
draft = false
tags = ['Vue', 'Vite']
categories = ['筆記']
+++

## 撰文動機

因為我是忘忘人所以做個筆記😀

## 安裝步驟

- 用 Vite 建立新專案
- 安裝 Vue Router & Pinia
- 安裝 Element plus
- 加載套件

## 開始實作

### 用 Vite 建立新專案

```bash
npm create vite@latest
```

接下來按照步驟建立專案就好

- 專案名稱
- 選擇框架
- 是否使用 Type Script
- Done

### 安裝 Vue Router & Pinia

請自己按需要增減

```bash
npm install pinia vue-router@4 --save
```

### 安裝 Element plus

```bash
npm install element-plus --save
```

## 加載套件

建立 vue router 檔案

```tsx
import {createRouter, createWebHistory} from 'vue-router'

import HelloWorld from '@/components/HelloWorld.vue';

export const router = createRouter({
    history: createWebHistory(),
    routes: [
        {path: '/', component: HelloWorld},
    ],
});
```

安裝完之後進到 `main.ts` 內，把前面裝好的東西都綁到 Vue 的實體就好了 ，如下。

```tsx
import {createApp} from 'vue'
import {createPinia} from "pinia";
import ElementPlus from 'element-plus';

import {router} from "@/router.ts";
import '@/style.css'
import 'element-plus/dist/index.css'

import App from '@/App.vue'

createApp(App)
    .use(createPinia())
    .use(ElementPlus)
    .use(router)
    .mount('#app')
```

這樣就大功告成啦!

## 結語

其實這篇文章是我在前前東家寫的，稍微修修補補就丟過來了，好讚😀
