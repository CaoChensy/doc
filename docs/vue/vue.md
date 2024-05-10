> npm

检查当前的下包地址：
```bash
npm config get registry
```

把下包的地址切换为国内的淘宝服务器

```bash
npm config set registry=https://registry.npm.taobao.org/
```

> vue

```bash
#安装vue.js
npm install vue -g

# 安装webpack模板
npm install webpack -g
npm install webpack-cli -g
webpack -v

# 安装脚手架vue-cli
npm install vue-cli -g
vue --version

# vue-router
npm install vue-router -g

# npm安装vue CLI
npm install -g @vue/cli
```


创建一个基于webpack模板的vue应用程序

```bash
vue init webpack 项目名
```

1. 项目名是？回车
1. 项目描述？回车
1. 作者？回车
1. 是否安装编译器 回车
1. 是否安装vue-router y 回车
1. 是否使用ESLint做代码检查 n 回车
1. 是否安装单元测试工具 n 回车
1. 单元测试相关 n 回车
1. 创建完成后直接初始化 n 回车

```bash
cd myvue
npm run dev
```
