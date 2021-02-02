---
title: github pages博客
date: 2019-09-21 00:00:00
tags: [github pages,博客]
categories: github
---
## github搭建博客（hexo）
### 1.安装git
git --version
### 2.安装node.js
	安装版（记得勾选add path）

	免安装版（配置环境变量）

注意：npm是Node.js自带的模块包管理工具，跟node.exe位于同一文件夹。Hexo项目也是提供了命令行工具hexo.cmd。这个项目被所有的实例共享，Hexo一般应全局安装。它的hexo.cmd所在文件夹应被放入系统path变量中。
node -v
npm -v
### 3.安装hexo
（似乎安装到nodejs目录下了）
	npm install hexo-cli -g
hexo -v 该命令会显示hexo的版本及依赖的包。
### 4.创建blog文件夹

### 5.cmd命令  进入blog文件夹  初始化hexo
	hexo init
生成文件夹和文件
node_modules	依赖包
public		存放的是生成的页面
scaffolds		命令生成文章等的模板
source		用命令创建的各种文章
themes		主题
.gitignore		不上传文件
_config.yml	整个博客的配置文件
db.json		source解析所得到的
package.json	项目所需模块项目的配置信息

### 6.安装或下载NexT主题插件
git clone https://github.com/iissnan/hexo-theme-next themes/next
或者，直接从发布页https://github.com/iissnan/hexo-theme-next/releases下载源指定版本源代码。下载之后，解压缩主题主题文件，并把它放到themes
### 7.修改站点配置文件
#原来的值是landscape
theme: next
### 8.启动hexo的服务器
hexo server
### 9.访问 http://localhost:4000 
### 10.下载hexo插件
（可能还需要升级hexo部分插件）
字数统计	npm install hexo-wordcount --save
git部署	npm install hexo-deployer-git --save
本地搜索	npm install hexo-generator-searchdb --save
2d特效	npm install hexo-helper-live2d --save
本地图片插件 npm install hexo-asset-image --save
### 11.修改主题配置文件
themes/主题/_config.yml
根据集成的插件 选择性的开启
如：评论区 访问统计等
### 常用操作
生成 aboutme 页面
	hexo new page about
生成标签页面
	hexo new page tags
	在tags中的index.md中新增一行 type: "tags"
生成分类页面
	hexo new page categories
	在categories中的index.md中新增一行 type: "categories"
以上三个页面如果有评论 需要关掉评论 index.md加上comments: false

写文章
	hexo new 标题/文件名
写草稿
	hexo new draft 标题/文件名
预览草稿
	hexo server --draft
发表草稿
	hexo publish 草稿标题/文件名


hexo g	生成静态文件
hexo s	启动服务器
hexo clean 清除缓存和已经生成的public静态文件

源文件存放在source中
静态文件存放在public中
（都可以改）
### 2d特效
[参考](https://www.simon96.online/2018/10/12/hexo-tutorial/)
将以下代码添加到主题配置文件_config.yml
live2d:
  enable: true
  scriptFrom: local
  pluginRootPath: live2dw/
  pluginJsPath: lib/
  pluginModelPath: assets/
  tagMode: false
  log: false
  model:
    use: live2d-widget-model-<你喜欢的模型名字> #miku z16 wanko
  display:
    position: right
    width: 150
    height: 300
  mobile:
    show: true
在站点目录下建文件夹live2d_models
再在live2d_models下建文件夹<你喜欢的模型名字>
再在<你喜欢的模型名字>下建json文件：<你喜欢的模型名字>.model.json
安装模型。在命令行（即Git Bash）运行以下命令即可：npm install --save live2d-widget-model-<你喜欢的模型名字>
hexo clean && hexo g && hexo s

[参考地址](https://blog.csdn.net/u012195214/article/details/79204088)

### 本地文件上传
1.把主页配置文件_config.yml 里的post_asset_folder:这个选项设置为true
2.安装插件
3.在md文件旁生成同名文件夹（如果是hexo n "xxxx"来生成md博文 可以自动创建文件夹）
4.图片放在文件夹里，在md文件里面插入图片
```
![你想输入的替代文字](xxxx/图片名.jpg)
```



### 12.推送到github上
创建repo 名为 username.github.io（username是github名称）
配置github账户信息
部署

### 根据需求 修改域名
第一步购买域名：随便在哪个网站买一个就好了，可以在阿里云购买
第二步添加CNAME：在项目的source文件夹下新建一个名为CNAME的文件，在里面添加购买的域名
第三步再次部署一下


### 资料
https://hexo.io/themes/ hexo的主题地址
https://hexo.io/zh-cn/docs/ hexo的使用文档

## github搭建博客（VuePress）
[VuePress官方文档](https://vuepress.vuejs.org/)
[VuePress从零开始搭建自己专属博客](https://segmentfault.com/a/1190000015237352?utm_source=tag-newest)
### 1.安装
npm install -g vuepress
### 2.创建项目目录并初始化
mkdir 项目名
cd 项目名
npm init -y
### 3.创建项目文档根目录
mkdir docs
### 4.在docs目录中，创建.vuepress目录
mkdir .vuepress
### 5.在docs中创建config.js
### 6.创建public文件夹
cd ../.vuepress
mkdir public

项目结构：
project
├─── docs
│   ├── README.md
│   └── .vuepress
│       ├── public
│       └── config.js
└── package.json

config.js的基本配置
module.exports = {
    title: '标题', 
    description: '描述',
    head: [
        ['link', { rel: 'icon', href: '/img/logo.ico' }],
        ['link', { rel: 'manifest', href: '/manifest.json' }],
    ]
}
### 常用操作
.生成静态文件
vuepress build docs
.本地运行
vuepress dev docs
访问http://localhost:8080
