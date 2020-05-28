---
layout: post
title: react-webpack+antd创建项目!
categories: react ,webpack 
description: 使用react-webpack+antd创建项目
keywords: react
---

## 1、创建初始化react+ts
```shell
   npx create-react-app react-ts --template typescript
```
## 2、安装antd
```shell
   npm install antd --save
```
 + 同时引入样式,修改 src/App.css，在文件顶部引入 antd/dist/antd.css。
   ```js
   import 'antd/dist/antd.css'; // or 'antd/dist/antd.less'
   ``` 

## 3、脚手架创建成功后的项目结构是如下：
```shell
├── README.md
├── node_modules
├── package-lock.json
├── package.json
├── public
└── src

```
## 4、使用npm run eject 进行项目扩展 项目结构如下
``` shell
├── README.md
├── config
├── node_modules
├── package-lock.json
├── package.json
├── public
├── scripts
└── src
```

