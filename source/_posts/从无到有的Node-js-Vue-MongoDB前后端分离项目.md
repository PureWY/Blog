---
title: "从无到有的 Node.js+Vue.js+MongoDB 前后端分离项目(一)"
catalog: true
date: 2018-11-2 10:43:19
subtitle: "一个自制的旅游网站"
header-img: "code.jpg"
tags:
- Site
- Blog
- Code
catagories:
- Hexo
---
>在学完 Node.js 与 Express 之后，手写了一个以 Node 为后台，并利用 Handlebars 引擎进行前台渲染的小项目。其实并没有完整的做完，只是大致了解一下  Node，以及如何写后台接口。因为临近毕业，需要一个完整的项目用于毕业设计，所以准备重新搭建一个以 Vue+Node 的前后端分离的完整项目，数据库用的是与    Node搭档效果不错的 MongoDB。本项目是一个旅游网站，我将尽可能仔细的介绍整个搭建过程，那就跟着我一步步走吧。


# 搭建后台
---
> 本项目使用 Git 作为代码管理工具，不熟悉 Git 操作的请移步[这里](https://juejin.im/post/5ae072906fb9a07a9e4ce596)

先进入个人的 GitHub,创建个人项目仓库。然后 git clone 到本地。(可自行创建 dev 分支)
进入项目目录，新建两个文件夹 client、server。顾名思义，client 为前台文件，server 为后台文件。

> 安装 Node.js,安装步骤请移步[这里](https://www.runoob.com/nodejs/nodejs-install-setup.html).

进入文件夹 server，执行命令 npm init，按照 cmd 的提示一步步走，至于项目名称等随意。这样第一小步已经完成了，接下来便是安装本项目有关的依赖。如下：
![项目依赖](https://purewy.github.io/img/Vue+Node/dependencies.png)
+ bcrypt:  用于加密密码
+ body-parser: 用于解析 URL
+ cookie-parser: 用于解析 cookie
+ cors: 用于处理跨域请求
+ dotenv: 用于 env 配置
+ express: 框架
+ express-session: 用于 session 设置
+ jsonwebtoken: 用于后端生成 token
+ mongoose: 数据库
+ nodemon: 用于自动化部署
*未作解释的可不装*

相关依赖装好以后，开始创建真正的项目，项目目录如下(酌情参考)：
![项目目录](https://purewy.github.io/img/Vue+Node/mulu.png)
+ config: 用于项目的整体配置
+ models: 数据库 MongoDB 的模型及 Schema
+ node_modules: 详细依赖
+ routes: 路由
+ .gitignore: 上传 git 的忽略文件，例如 node_modules
+ index.js: 入口文件
+ nodemon.js 自动化配置
+ package-lock.json: 自动生成
+ package.json: 依赖目录，项目信息
+ READEME.md: 项目介绍
*目录很简洁*

> 接下来进入 index.js,不熟悉 Node.js 的请移步[这里](https://www.liaoxuefeng.com/wiki/001434446689867b27157e896e74d51a89c25cc8b43bdb3000/001434501245426ad4b91f2b880464ba876a8e3043fc8ef000).

引入各项所需文件:
```
var express = require('express');
var session = require('express-session');
var mongoose = require('./config/mongoose.js');
var bodyParser = require('body-parser');
var cors = require('cors');
```
然后使用 express:
`var app = express();`

解决跨域:
`app.use(cors());`
(使用cors是最方便的使用中间件解决跨域的办法)

设置项目启动端口:
`app.set('port',process.env.PORT || 3333);`

设置URl解析:
```
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({
  extended: true
}));
```

如果使用 session 验证便按如下配置:
```
app.use(session({
    secret: 'cookieSecret', //作为服务器端生成session的签名
    resave: true,           //是否允许当客户端并行发送多个请求时，其中一个请求在另一个请求结束时对session进行修改覆并保存
    saveUninitialized: false, //初始化session时是否保存到存储
    cookie: {
        maxAge: 1000 * 60 * 5  //设置session的有效时间，单位毫秒
    }
}))
```
最后开启监听端口:
```
app.listen(app.get('port'), () => {
    console.log('Express started on http://localhost:' +
	app.get('port') + '; press Ctrl-c to terminate.');
});
```

此时运行 node index.js 应该控制台会显示:
```
Express started on http://localhost:3333; press Ctrl-c to terminate.
MongoDB连接成功!
```

说明运行成功，配置无误。

但是每次都需要运行 node xxx.js 未免太过麻烦，nodemon 的作用此刻就可以显示出来了。nodemon 用来监视node.js应用程序中的任何更改并自动重启服务,非常适合用在开发环境中。nodemon 将监视启动目录中的文件，如果有任何文件更改，nodemon 将自动重新启动node应用程序。nodemon 不需要对代码或开发方式进行任何更改。 nodemon 只是简单的包装你的 node 应用程序，并监控任何已经改变的文件。nodemon 只是node的替换包，只是在运行脚本时将其替换命令行上的node。

下面进行 nodemon 的相关配置，没错，就是之前创建的 nodemon.json.

```
{
    "restartable": "rs",
    "ignore": [
        ".git",
        ".svn",
        "node_modules/**/node_modules"
    ],
    "verbose": true,
    "execMap": {
        "js": "node --harmony"
    },
    "watch": [

    ],
    "env": {
        "NODE_ENV": "development"
    },
    "ext": "js json"
}
```
+ restartable: 设置重启模式 
+ ignore: 设置忽略文件 
+ verbose: 设置日志输出模式，true 详细模式 
+ execMap: 设置运行服务的后缀名与对应的命令,表示使用 nodemon 代替 node 
+ watch: 监听哪些文件的变化，当变化的时候自动重启 
+ ext: 监控指定的后缀文件名
*如果你的入口文件为 index.js，只需在命令行执行 nodemon 即可*

如上，Ctrl+C 关闭之前执行进程，执行 nodemon，会发现项目成功运行，并且之后的每一次修改保存以后都会自动监听执行。到此基本的后台搭建已经完成，继而进行数据库的连接配置。

> 到此可以提交一下代码啦，先行配置 .gitignore 文件，防止将不需要的文件提交上去。在文件中输入以下代码即可忽略掉该文件。
```
node_modules/
/dist/
```

# 连线数据库

> MongoDB 是一个基于分布式文件存储的数据库。由 C++ 语言编写。旨在为 WEB 应用提供可扩展的高性能数据存储解决方案。
MongoDB 是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。
关于其他数据库的选择与介绍，请移步[这里](https://blog.csdn.net/xingxuexx/article/details/64122687).

接下来先进行数据库的基本连接配置。
打开 config 文件夹，新建 config.js。配置数据库连接如下：
```
module.exports = {
    mongodb: 'mongodb://localhost:27017/index'
}
```

在此目录下新建文件 mongoose.js，代码如下：
```
const mongoose = require('mongoose');
const config = require('./config');

module.exports = () => {
    mongoose.connect(config.mongodb);   //连接数据库
    //实例化连接对象
    var db = mongoose.connection;
    db.on('error',console.error.bind(console,'连接错误:'));
    db.once('open',(callback) => {
        console.log('MongoDB连接成功!');
    });
    return db;
}
```

此时打开 cmd，输入 mongod，便可运行数据库。若无异常控制台便会显示：

`waiting for connections on port 27017`

此时数据库已成功连接。