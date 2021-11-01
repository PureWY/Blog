---
title: 从无到有的 Node+js+Vue+MongoDB 前后端分离项目(三)
catalog: true
date: 2018-11-09 09:08:20
subtitle: "一个自制的旅游网站"
header-img: "code.jpg"
tags:
- Site
- Blog
- Code
catagories:
- Hexo
---
>到此为止，前端跟后端的框架已经大致搭建完成了，下一步就是进行前后端通信。其实主要做的工作就是后台写好接口，处理数据库的数据，前台调用接口，拿到数据，展示出来。没错就是这么简单，其实用Express框架写起接口来异常方便，这估计也是Express框架排行第一的原因，好用谁不用呢。

>接口其实很好写，我只拿一个简单的接口举例，我的项目中需要用到查询当前城市的酒店信息。那么我们就从这开始。

>不熟悉 Mongoose 的请看[这里](https://segmentfault.com/a/1190000012095054#articleHeader7)，mongoose.js 是用来连接mongodb 数据库并引用定义 Schema 和 Model，这样对于数据库处理起来会更方便。

# 定义 Schema 和 Model

比如说要创建一个酒店的表，就需要这样定义Schema:
先引入 Mongoose

```
const mongoose = require('mongoose');
const Schema = mongoose.Schema;
```

然后根据需要创建 Schema：

```
//创建houseSchema
var houseSchema = new Schema({
    roomId: {type: String,trim: true,required: true},       //房间表Id
    commentId: {type: String,trim: true,required: true},    //评论表Id
    houseName: {type: String,trim: true},                   //酒店名称
    houseStar: {type: Number,trim: true},                   //酒店星级
    housePhone: {type: Number,trim: true},                  //酒店电话
    houseTag: {type: String,trim: true},                    //酒店标签类型
    houseGrade: {type: Number,trim: true},                  //酒店评分
    houseCityPlace: {type: String,trim: true},              //酒店所属市
    houseAreaPlace: {type: String,trim: true},              //酒店所属区
    houseDetailPlace: {type: String,trim: true},            //酒店详细位置
    houseType: {type: String,trim: true},                   //酒店住宿类型
    houseFreeSer: {type: Array,default: []},                //酒店提供服务种类
    ...
  })
```

>字段太多就不一一列举，大概就是这么创建，很简单，需要注意的是MongoDB支持的数据类型，详情查看[这里](https://www.cnblogs.com/firstForEver/p/6843605.html)

然后就是建立 Model:

```
//创建model
var House = mongoose.model('House',houseSchema);
```

然后别忘了导出：
`module.exports = House;`

完成到这里以后，你的数据库文档就已经完成了，可以试着向里面塞一条数据，然后打开你的数据库管理工具，我用的是 Navicat,他对 MongoDB 有很好的支持，创建数据就像这样：

```       
House.create({houseName:"Shanghai",...各种字段},function(err,doc){
    //{ __v: 0, houseName:"Shanghai",...各种字段， _id: 59720d83ad8a953f5cd04664 }
    console.log(doc); 
});  
```

如果你没有设置id，它会帮你自动生成一个。doc 就是插入数据以后返回的House文档内容。
这样数据就插入成功了，更新，删除，修改等等就参照文档去做，大同小异。
接下来我们看看用Express如何写一个接口，以查询酒店为例。
先引入需要的东西：

```
const express = require('express');
const router = express.Router();

//引入模型
const House = require('../../models/hotel/house')
```

然后定义路由，写查询语句以及返回结果：
```
//整体查询酒店信息
router.post('/queryHotel', function(req, res, next) {
  House.find(req.body), (err, house) => {
    if (err) {
      res.json({
        code: 201,
        message: '数据库异常'
      })
    } else {
      if (!house.length) {
        res.json({
          code: 202,
          message: '该地区暂无酒店信息'
        })
      } else {
        res.json({
          code: 200,
          body: house,
          message: '酒店信息查询成功'
        })
      }
    }
  })
})
```

这就是根据前端传过来的查询条件去数据库查询符合条件的数据，查到结果以后，通过res.json返回给前端，处理格式按照自己的需要。
这时候这个接口就已经可以拿来使用了，我们在routes文件夹里面新建文件，index.js，然后引入刚才的接口文件：
`const hotel = require('./hotel/hotel.js')`
然后通过路由去定义接口并使用接口：
```
const routers = function (app) {
    app.use('/hotel',hotel);

    //定制404页面
    app.use(function(req,res){
        res.status(404);
        res.render('404');
    });

    //定制500页面
    app.use(function(err,req,res,next){
        console.log('err.stack');
        res.status(500);
        res.send('500');
    });
}

module.exports = routers;
```
这样定义以后，导出routers模块，在入口的index.js中应用一下就行了，具体代码参照前面的内容。这样一个酒店查询接口就完成了。你可以在postman测一下这个接口，如果正常返回数据就没有问题了，其他的接口也是这样写，就不一一说明了。

# 与后端通信

后端接口准备就绪以后，我们就可以拿来使用了。
在前面的request.js 文件中，我们已经封装过请求了，这时候我们就拿来使用就可以了。创建一个api文件夹，然后创建文件hotel.js文件，代码如下：

```
import request from '@/utils/request'

//查询酒店
export function queryHotel(data) { 
    return request({
        url: '/hotel/queryHotel',
        method: 'post',
        data: JSON.stringify(data)
      })
 }
 ```

然后在你的前端页面引入这个文件：
`import {queryHotel} from '../../../api/hotel/index.js'`
然后使用它：

```
queryHotel({houseCityPlace: this.queryCity}).then(res => {
    if(res.data.code == 200){
        this.houseInfo = res.data.body
        console.log(this.houseInfo)
    }
    }).catch(() => {

    })
```

没错就是这样简单，接口就对接完成了，拿到的 res.data.body 就是之前在后台定义的。
到此我们前后端的通信就已经完成了，剩下的就需要自己在项目中多学多敲了。