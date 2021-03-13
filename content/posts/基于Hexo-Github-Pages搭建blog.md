---
title: 基于Hexo+Github Pages搭建blog
date: 2021-02-25 23:34:03
tags:
---
---
## 安装Hexo
[https://hexo.io/zh-cn/docs/](https://hexo.io/zh-cn/docs/)
```shell
npm install hexo
```
安装 Hexo 完成后，请执行下列命令，Hexo 将会在指定文件夹中新建所需要的文件。
```shell
$ hexo init <folder>
$ cd <folder>
$ npm install
```
启动服务

```shell
$ hexo g  //  hexo generate  生成静态文件
$ hexo s  // 启动本地预览服务
```


## 修改主题为Next主题
#### 安装主题
#### Using Git  推荐,可修改模板文件
```javascript
$ cd hexo-site
$ git clone https://github.com/next-theme/hexo-theme-next themes/next
```
#### Using npm   安装后Theme文件夹下没有主题文件
```javascript
$ cd hexo-site
$ npm install hexo-theme-next
```
#### 配置主题
```javascript
hexo/_config.yml
theme: next
```
#### Next文档
[https://theme-next.js.org/docs/getting-started/](https://theme-next.js.org/docs/getting-started/)
#### 去掉 Powered by 信息
/themes/(你的主题)/layout/_partials/footer.***

```javascript
{%- if theme.footer.powered %}
  <div class="powered-by">
    {%- set next_site = 'https://theme-next.js.org' if theme.scheme === 'Gemini' else 'https://theme-next.js.org/' + theme.scheme | lower + '/' %}
    {{- __('footer.powered', next_url('https://hexo.io', 'Hexo', {class: 'theme-link'}) + ' & ' + next_url(next_site, 'NexT.' + theme.scheme, {class: 'theme-link'})) }}
  </div>
{%- endif %}
```


## 写作
具体查看文档 [https://hexo.io/zh-cn/docs/commands](https://hexo.io/zh-cn/docs/commands)
```shell
$ hexo new "Hello World"  // 简写 hexo n "Hello World"
```
**指定目录创建Page**
```shell
hexo new page --path about/me "About me"
```


## 常用命令
```shell
hexo s // 启动本地服务
hexo g // 生成静态文件  
hexo g -d // 生成后自动部署  -> hexo d -g 部署前预先生成静态文件
hexo clean  // 清除缓存文件 (db.json) 和已生成的静态文件 (public)
```


## Github Page一键部署
[https://hexo.io/zh-cn/docs/github-pages](https://hexo.io/zh-cn/docs/github-pages)

**基于私有 Repository**

1. 安装 hexo-deployer-git
2. 在 **_config.yml**（如果有已存在的请删除）添加如下配置：
```shell
deploy:
  type: git
  repo: https://github.com/<username>/<project>
  # example, https://github.com/hexojs/hexojs.github.io
  branch: master  # gh-pages这里改为master
```

1. 运行hexo clean  &  hexo g -d
---


## 参考
1. [GitHub+Hexo 搭建个人网站详细教程](https://zhuanlan.zhihu.com/p/26625249)
