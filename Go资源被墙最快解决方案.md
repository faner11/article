最近在学`golang`,因为文化自信,`Go`语言好多包国内是无法获取的,写这篇教程希望可以帮助大家最快的解决资源被墙的问题,不要把时间浪费这种无意义的事情上.

## 环境

```bash
  $ go version  # go version go1.12 darwin/amd64
```

- 把 golang 升级到 1.11（建议使用 **1.12**,1.11 之后，go 官方引入了 go module 来解决依赖管理问题.这是快速解决被墙的核心.

## 设置 GO111MODULE 和 GOPROXY

- mac 系统下,打开终端
  ```bash
    $ vim .bash_profile
  ```
  加上
  ```
    export GO111MODULE=on
    export GOPROXY=https://goproxy.io
  ```
  保存退出;
- 执行`source ~/.bash_profile`;使刚才的修改立即生效;
- 打开`新的终端`(重要),不要在上一终端界面
  输入`go env`;如果看到`GOPROXY="https://goproxy.io"`则配置成功;
- 自由自在的下载包吧.

## 相关

- [goproxy.io](https://goproxy.io/)
- 如何使用[go modules](https://juejin.im/post/5c8e503a6fb9a070d878184a)
