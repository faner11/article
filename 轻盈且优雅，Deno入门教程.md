Deno 1.0 正式版本刚刚发布，今天用 Deno 开发一个简单 web 服务先吃螃蟹。

## 关于 Deno

### Deno 的发音

目前有大家的发音有两种"德诺","蒂诺" ,我个人认为"蒂诺"是更合适的

### Deno 的身世

Deno 是 Ryan Dahl 在 2017 年创立的,Ryan Dahl 同时也是 node 的创始人，具体 Ry 为什么对 node 不满，为什么创立 Deno，感兴趣的话请自行 google。

### Deno 环境

Deno 是个使用 `Rust` 开发的一个服务器运行时，同时支持多种语言，可以直接运行 `JavaScript`、`TypeScript` 和 `WebAssembly` 程序。

## Deno vs Node

- 可以直接运行`TypeScript`
- 全新的包管理，所有模块通过 url 引入，没有`npm`，没有沉重的`node_modules`
- 安全控制，默认情况下脚本不具有读写权限，网络权限，必须使用参数显式打开权限，妈妈再也不用担心别人的代码在你的程序里乱跑了
  - --allow-read 打开读权限，可以指定可读的目录，比如---allow-read=/temp。
  - --allow-write 打开写权限。
  - --allow-net 允许网络通信，可以指定可请求的域
  - --allow-env 允许读取环境变量
  - 权限声明要放在前面 例：`deno run --allow-net index.ts`

## 安装 node

- mac 和 linux

  ```shell
  # 自己配置环境变量
  $ curl -fsSL https://deno.land/x/install/install.sh | sh

  # 一步到位，不用自己配置环境变量(建议)
  $ curl -fsSL https://deno.land/x/install/install.sh | sudo DENO_INSTALL=/usr/local sh
  ```

- windows
  ```
  $env:DENO_INSTALL = "C:\Program Files\deno" iwr https://deno.land/x/install/install.ps1 -useb | iex
  ```

## VSCode 插件

在 VSCode 中搜索 deno 会出现两个插件，一个作者是 axetroy，一个是 justjavac，这里我们先使用 justjavac 开发的，axetroy 开发的虽然被官网推荐但是很早就不维护了，已经无法正常使用，后续官方正式插件会推出

## 万物起源 Hello world

deno 准备工作就绪了，按照惯例先写个 hello world

- 新建 deno-demo 文件夹

  ```shell
  $ mkdir deno-demo
  $ cd deno-demo
  ```

- 新建 index.ts

  ```js
  // deno-demo/index.ts
  console.log('hello world');
  ```

- 运行

  ```shell
  $ deno run index.ts
  ```

- 输出 hello world

## Web 服务

- 代码
  ```js
  // deno-demo/index.ts
  import { Application } from 'https://deno.land/x/abc/mod.ts';
  const app = new Application();
  app.get('/hello', (c) => {
    return {
      msg: 'hello world for get',
    };
  });
  app.post('/hello', (c) => {
    return {
      msg: 'hello world for post',
    };
  });
  app.start({ port: 8080 });
  ```
- 运行

```shell
# 需要网络权限，加上--allow-net，
$ deno run --allow-net index.ts
```

`--allow-net`要写前面，写在后边会报错

## 后记

对于 Deno，我是非常看好的，TS，JS 大家本来就会不存在语言学习障碍，而且使用 Deno 写 TS 轻盈既更优雅，更好的开发体验，更安全的运行时，潜力无穷。
