# nuxt+pm2 自动化部署及打包后文件自动上传阿里云 oss

在读这篇文档时，希望你对 nuxt 及 pm2，有简单的了解

- [nuxt](https://zh.nuxtjs.org/guide/installation)
- [pm2](http://pm2.keymetrics.io/docs/usage/deployment/)

## 前期准备

### 安装 pm2 及构建 nuxt

```s
$ npm i pm2 -g
$ npx create-nuxt-app <项目名>
```

### ssh 密钥配置

- pm2 代码自动发布依赖于 git 工具，先将 ssh 密钥配置在你的代码仓库（github 或者 gitLab），具体操作自行 google 或者点击[github 配置 ssh](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/)。
- 使用 ssh 密钥链接服务器
  ```s
  $ ssh-copy-id root@1.2.3.4
  # 把本机的 SSH 秘钥添加至服务器，配置成功后，以后就不需要再执行这条 SSH 命令了
  ```

## pm2 自动部署

### 生成 pm2 配置文件

```s
$ pm2 ecosystem
```

运行后会在项目根目录生成 ecosystem.config.js 文件

### 这是个简单的配置文件，供大家参考

```js
module.exports = {
  apps: [
    {
      name: "my-app",
      autorestart: true,
      script: "server/index.js",
      env: {
        NODE_ENV: "development"
      },
      env_production: {
        NODE_ENV: "production"
      }
    }
  ],
  deploy: {
    dev: {
      // 服务器操作用户
      user: "root",
      // 服务器ip
      host: "1.2.3.4",
      ref: "origin/master",
      repo: "https://github.com/faner11/angular-case.git",
      path: "/root/my-app",
      "post-deploy":
        "rm -rf node_modules && npm install && npm run build && pm2 startOrReload ecosystem.config.js --env production"
    }
};

```

`post-deploy`中做了哪些操作

- `rm -rf node_modules` 删除 node_modules
- `npm install` 重新安装包
- `npm run build` 运行打包
- `pm2 startOrReload ecosystem.config.js --env production` pm2 启动应用

### 初始化项目并发布

- 本机初始化远程服务器上的项目`pm2 deploy dev setup`,命令中的`dev`是在上面配置文件中写的部署环境的名称。
- git 提交代码，`git push origin master`将代码提交至远程仓库。
- 部署项目`pm2 deploy dev`，这个命令执行后服务器把前面从本机提交至 git 仓库上的最新代码拉下拉，并且运行`post-deploy`中的命令。一般没什么问题的话，经过这几步操作，就能部署成功了。

## 打包后文件上传 oss

参考文档

- [oss CDK node.js 版](https://help.aliyun.com/document_detail/32068.html?spm=a2c4g.11186623.6.927.347d69cbcOsMQs)
- [nuxt dist 文件上传到 CDN](https://zh.nuxtjs.org/api/configuration-build/#publicpath)
  我们需要将 `.nuxt/dist/client`上传至 cdn

### 上传代码

在根目录新建`upload.js`文件

```js
const OSS = require("ali-oss");
const fs = require("fs");
const path = require("path");
const os = require("os");
const PUBLIC_PATH = path.join(__dirname, "/");

const client = new OSS({
  accessKeyId: "your access key",
  accessKeySecret: "your access secret",
  bucket: "your bucket name",
  region: "oss-cn-hangzhou"
});

/**
 *获取文件目录并删除
 * @param {*} dir //文件目录
 */
async function deleteDir(dir) {
  let result = await client.list({
    prefix: dir + "/",
    delimiter: "/"
  });
  if (result.objects) {
    let aa = [];
    result.objects.forEach(function(obj) {
      aa.push(obj.name);
    });
    try {
      await client.deleteMulti(aa, {
        quiet: true
      });
      console.log("删除成功");
    } catch (e) {
      console.log("文件删除失败", e);
    }
  }
}

/**
 * 遍历文件夹递归上传
 * @param {path} src 本地路径
 * @param {string} dist oos文件夹名 www|kouzi
 */
function addFileToOSSSync(src, dist) {
  let docs = fs.readdirSync(src);
  docs.forEach(function(doc) {
    let _src = src + "/" + doc,
      _dist = dist + "/" + doc;
    let st = fs.statSync(_src);
    // 判断是否为文件
    if (st.isFile() && doc !== ".DS_Store") {
      putOSS(_src, _dist);
    }
    // 如果是目录则递归调用自身
    else if (st.isDirectory()) {
      addFileToOSSSync(_src, _dist);
    }
  });
}
/**
 *单个文件上传至oss
 */
async function putOSS(src, dist) {
  try {
    await client.put("/" + dist, src);
  } catch (e) {
    console.log("上传失败".e);
  }
}
/**
 *上传文件启动
 *@param {string} dirName 将要上传的文件名
 */
async function upFile(dirName) {
  try {
    await deleteDir(dirName);
    await addFileToOSSSync(PUBLIC_PATH + ".nuxt/dist/client", dirName);
    console.log(dirName + "上传oss成功");
  } catch (err) {
    console.log(dirName + "上传oss成功失败", err);
  }
}

upFile("www");
```

### 修改`package.json`

将`scripts`中的`build`改为如下：

```json
{
  "scripts": {
    "build": "nuxt build && node upload.js"
  }
}
```

### 修改`nuxt.config.js`

```js
export default {
  build: {
    publicPath: "https://cdn.nuxtjs.org"
  }
};
```

## 结束

至此我们的自动化部署加文件自动上传阿里云 oss 就完成了。
以后只需执行`pm2 deploy dev`就可以了。
