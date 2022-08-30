# 构建与部署 {#index}

## 安装依赖 {#install-rely}

在编译代码前先安装依赖：

```sh
pnpm i  # 安装开发期依赖
```

## 编译代码 {#compile-code}

由于本项目基于`Midway`、`React`是 TypeScript 编写，在部署前，我们先进行编译。在示例中，我们预先写好了构建脚本，执行以下内容即可。

```sh
pnpm run build  # 构建项目
```


`pnpm run build` 编译需要几个步骤：

- 1、pnpm run clean（清空编译的历史文件）
- 2、pnpm run build:midway（编译Midway代码）
- 3、pnpm run build:web:client（编译React客户端代码）
- 4、pnpm run build:web:server（编译React服务端代码）


## 移除开发依赖 {#remove-develop-rely}

因为在生产环境中，开发依赖是没有必要的而且占用存储空间，所以我们可以移除它：

```sh
pnpm prune --production  # 移除开发依赖
```

:::tip
一般安装依赖会指定 `NODE_ENV=production` 或 `pnpm install --production` ，在构建正式包的时候只安装 dependencies 的依赖。因为 devDependencies 中的模块过大而且在生产环境不会使用，安装后也可能遇到未知问题。
:::


## 启动项目 {#start-project}

项目一般都需要一个入口文件，比如，我们在根目录创建一个 `bootstrap.js`(基于`@midwayjs/bootstrap`组件) 作为我们的部署文件。

:::tip
注意，这里不含 http 的启动端口，如果你需要，可以[参考Midway多环境配置文档](/guide/essentials/config.md#midway-env-config) 修改端口。
:::

我们这里推荐用`pm2`启动项目，[PM2常用命令](http://liqingsong.cc/article/detail/3)。

本项目对应的 pm2 启动命令为：

```sh
NODE_ENV=production pm2 start ./bootstrap.js --name midway_react_ssr -i 4
```

- --name 用于指定应用名
- -i 用于指定启动的实例数（进程），会使用 cluster 模式启动

效果如下：

![deploy](/images/deploy.png)



## 总结 {#summarize}

根据以上说明，如下操作即可：

```sh
## 服务器构建部署（已经下载好代码）
pnpm i                   # 安装开发期依赖
pnpm run build           # 构建项目
pnpm prune --production  # 移除开发依赖
NODE_ENV=production pm2 start ./bootstrap.js --name midway_react_ssr -i 4 # 启动项目
```




