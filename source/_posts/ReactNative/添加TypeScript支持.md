---
title: React Native-添加TypeScript支持
categories: React Native
comments: true
tags: [React Native]
description: 添加TypeScript支持
date: 2020-01-12 10:00:00
---

## 添加TypeScript支持

初始化RN工程
```
react-native init MyProject
cd MyProject
```
安装TypeScript相关依赖
```
yarn add tslib @types/react @types/react-native
yarn add --dev react-native-typescript-transformer typescript
```
配置tsconfig.json  
您可以从您之前的TS项目中复制这个文件，也可以使用tsc初始化（具体参考TypeScript的教程）。注意务必设置"jsx":"react"
注意多余的注释可能会不兼容，需要移除

```
tsc --init --pretty --sourceMap --target es2015 --outDir ./lib --rootDir ./ --module commonjs --jsx react
```

http://www.typescriptlang.org/docs/handbook/tsconfig-json.html

```
{
  "compilerOptions": {
    "importHelpers": true,
    "target": "es2015",
    "jsx": "react",
    "noEmit": true,
    "moduleResolution": "node",
  },
  "exclude": [
    "node_modules",
  ],
}
```

在项目目录中新建或编辑rn-cli.config.js，使用支持typescript的transfomer


```
module.exports = {
  getTransformModulePath() {
    return require.resolve('react-native-typescript-transformer');
  },
  getSourceExts() {
    return ['ts', 'tsx'];
  }
}

```

最后修改你的源代码
入口index.js和App.js的文件名请不要改变，你可以新建src文件夹，在其中建立index.tsx，然后把App.js的内容改成

```
import './src';
```

然后在index.tsx里重新编写你的入口代码。注意引用react的写法有所区别：

```
-import React, { Component } from 'react';
+import React from 'react'
+import { Component } from 'react';
```
