<!--
 * @Date: 2022-05-02 19:08:48
 * @LastEditors: Cosima
 * @LastEditTime: 2022-05-02 19:42:23
 * @FilePath: /cosima/readme.md
-->
Cosima-blog
================
基于hexo搭建的个人博客 博客地址[Cosima](https://cosimac.github.io/) 
项目源码仓库[Cosima](https://github.com/Cosimac/Cosimac.github.io)

## 项目介绍
- 个人博客记录分享

## 项目启动
``` bash
$ git clone git@github.com:Cosimac/Cosimac.github.io.git
```

```
$ npm i
```

```
$ npm install -g hexo-cli
```

```
$ hexo s
```
## 项目开启本地服务debug模式
```
$ hexo s --debug
```
## 项目新增页面
hexo new "post title with whitespace"
```
$ hexo new [layout] <title>
```
## 项目代码打包构建并发布
```
$ hexo generate -d
```
## 清除打包文件
```
$ hexo clean
```
