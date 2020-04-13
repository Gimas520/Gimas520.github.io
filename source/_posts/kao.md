---
title: kao学习
date: 2020-04-12 19:08:38
avatar: https://s1.ax1x.com/2020/04/10/Go1m4K.gif
authorLink: http://github.com/Gimas520
authorAbout: 一个好奇的人
authorDesc: 一个好奇的人
categories: 技术
comments: true
tags: web
description: koa的学习
photos: https://yremp.live/wp-content/uploads/2020/03/ios.jpg
---

第一次弄hexo和Sakura主题不知道写些什么，那就来一篇我看过的文章吧

# koa学习笔记
## 1. 判断客户端希望接受的数据类型（HTTP Request 的Accept字段)
```
const Koa = require('koa');
const app = new Koa();
const main = ctx => {
  let arr = [
    { type: 'xml' ,data:'<data>Hello World</data>'}, 
    { type: 'json' ,data:{ data: 'Hello World' }}, 
    { type: 'html' ,data:'<p>Hello World</p>'}, 
    { type: 'text' ,data:'Hello World'}
  ];
  for (const item of arr) {
    if (ctx.request.accepts(item.type)) { // 判断类型
      ctx.response.type = item.type;
      ctx.response.body = item.data;
      break;
    }
  }
};
app.use(main);
app.listen(3000);
```

## 2. 读取本地文件
```
const fs = require('fs');
const Koa = require('koa');
const app = new Koa();
const main = ctx => {
  ctx.response.type = 'html';
  ctx.response.body = fs.createReadStream('./demos/template.html'); // 读取本地文件
};
app.use(main);
app.listen(3000);
```

## 3. 获取用户请求路径
```
const Koa = require('koa');
const app = new Koa();
const main = ctx => {
  if (ctx.request.path !== '/') {
    ctx.response.type = 'html';
    ctx.response.body = '<a href="/">Index Page</a>';
  } else {
    ctx.response.body = 'Hello World';
  }
};
app.use(main);
app.listen(3000);
```

## 4. koa-router
```
const route = require('koa-route');
const about = ctx => {
  ctx.response.type = 'html';
  ctx.response.body = '<a href="/">Index Page</a>';
};
const main = ctx => {
  ctx.response.body = 'Hello World';
};
app.use(route.get('/', main));
app.use(route.get('/about', about));
```

## 5.  访问静态资源
```
const Koa = require('koa');
const app = new Koa();
const path = require('path');
const serve = require('koa-static');
const main = serve(path.join(__dirname)); // __dirname 下的静态文件即可访问
app.use(main);
app.listen(3000);
```

## 6.  重定向
```
// 将 /redirect 重定向到 /
const redirect = ctx => {
  ctx.response.redirect('/'); 
  ctx.response.body = '<a href="/">Index Page</a>';
};
app.use(route.get('/redirect', redirect));
```

## 7.  中间件方式
### 方式 1
```
const Koa = require('koa');
const app = new Koa();
// 在main函数中实现 日志中间件
const main = ctx => {
  console.log(`${Date.now()} ${ctx.request.method} ${ctx.request.url}`);
  ctx.response.body = 'Hello World';
};
app.use(main);
app.listen(3000);
```

### 方式 2
```
const Koa = require('koa');
const app = new Koa();
const logger = (ctx, next) => {
  console.log(`${Date.now()} ${ctx.request.method} ${ctx.request.url}`);
  // 使用next让下一个中间件执行
  next();
}
const main = ctx => {
  ctx.response.body = 'Hello World';
};
// 使用use加载中间件
app.use(logger);
app.use(main);
app.listen(3000);
```

## 8.  中间件的next
 ```
 const Koa = require('koa');
const app = new Koa();
const one = (ctx, next) => {
  console.log('>> one');
  next();
  console.log('<< one');
}
const two = (ctx, next) => {
  console.log('>> two');
  next();
  console.log('<< two');
}
const three = (ctx, next) => {
  console.log('>> three');
  next();
  console.log('<< three');
}
app.use(one);
app.use(two);
app.use(three)
app.listen(3000);
```

## 9.  中间件的next
```
const fs = require('fs.promised');
const Koa = require('koa');
const app = new Koa();
const main = async function (ctx, next) {
  ctx.response.type = 'html';
  ctx.response.body = await fs.readFile('./demos/template.html', 'utf8');
};
app.use(main);
app.listen(3000);
```

## 10.  中间件合成
```
const Koa = require('koa');
const compose = require('koa-compose');
const app = new Koa();
const logger = (ctx, next) => {
  console.log(`${Date.now()} ${ctx.request.method} ${ctx.request.url}`);
  next();
}
const main = ctx => {
  ctx.response.body = 'Hello World';
};
// middlewares合成了各个中间件
const middlewares = compose([logger, main]);
app.use(middlewares);
app.listen(3000);
```