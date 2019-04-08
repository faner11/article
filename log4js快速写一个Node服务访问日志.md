## log4js快速写一个Node服务访问日志
log4js是Node.js中非常棒的日志模块,使用起来非常简单且好用。这一篇和大家分享下如何很快的写一个日志记录服务，因为node的中间件模式，所以我们将日志服务写为一个中间件。

下面将以`koa`为例写一个访问日志中间件
## 安装log4js
```bash
npm install log4js
```
## log4js配置讲解
```js
log4js.configure({
  appenders: {
    visitor: {
      type: "dateFile", 
      filename: `./logs/visitor-`,
      pattern: ".yyyy-MM-dd.log",
      alwaysIncludePattern: true,
      daysToKeep: 30,
      layout: {
        type: "messagePassThrough"
      }
    }
  },
  categories: { default: { appenders: ["visitor"], level: "info" } }
});
```
`appenders.visitor`我们做了一些个性化配置，为什么要这样做呢？
- `type: "dateFile"`：日志文件可以安特定的日期模式滚动，例如今天输出为**visitor-2019-03-08.log**，明天输为**visitor-2019-03-09.log**；
- `filename: './logs/visitor.log'`:设置log输出的文件路劲和文件名
- `alwaysIncludePattern: true`: 和上面同时使用 设置每天生成log名
- `daysToKeep: 30`: 日志只保存30天
`layout.type` 我们设为`messagePassThrough`,不适用log4js提供的日志模版，完全使用自己的代码，这样做的好处是当我们的日志要和阿里云日志关联的时候必须要写成JSON的，为了方便查询以及可控性，我们要使用自己的定义的JSON。


## 完整代码
```js
const log4js = require("log4js");
log4js.configure({
  appenders: {
    visitor: {
      type: "dateFile", 
      filename: `./logs/visitor.log`,
      alwaysIncludePattern: true,
      daysToKeep: 30,
      layout: {
        type: "messagePassThrough"
      }
    }
  },
  categories: { default: { appenders: ["visitor"], level: "info" } }
});
const logger = log4js.getLogger("visitor");

module.exports = function() {
  return async function(ctx, next) {
    const start = new Date();
    await next();
    // 根据文件结尾筛选
    const index = ctx.url.lastIndexOf(".");
    const ext = ctx.url.substr(index + 1);
    const extArr = ["js", "png", "otf", "txt", "html", "xml", "json", "ico"]; 
    if (extArr.indexOf(ext) === -1 ) {
      const end = new Date();
      // 这里是取nginx反向代理后的ip信息， ctx.ip + "i"这个是给坏人准备的，如果ip结尾带了i且不是你们公司的ip，那是有很大概率是坏人的。如果你们没有使用nginx那就只取 ctx.ip。
      const Uip =
        ctx.get("HTTP_X_REAL_IP") ||
        ctx.get("X-Read-IP") ||
        ctx.get("HTTP_X_FORWARDED_FOR") ||
        ctx.get("X-Forwarded-For") ||
        ctx.get("Remote_Addr") ||
        ctx.ip + "i";
      let data = {
        startTime: start.toJSON(),
        endTime: end.toJSON(),
        uri: ctx.url,
        ip: Uip || "",
        status: ctx.status,
        referrer: ctx.headers.referer || "",
        msec: end - start,
        ua: ctx.header["user-agent"]
      };
      logger.info(JSON.stringify(data));
    }
  };
};

```