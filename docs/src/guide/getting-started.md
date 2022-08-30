# 快速开始 {#index}


## 安装 {#install}

```bash
# 克隆项目
git clone https://github.com/lqsong/midway-react-ssr.git

# 进入项目目录
cd midway-react-ssr

# 复制文件
copy src/config/config.default.ts  src/config/config.local.ts # 启用或修改里面的参数
copy src/config/config.default.ts  src/config/config.prod.ts # 启用或修改里面的参数
copy web/.env.development  web/.env.development.local # 启用或修改里面的参数

# 安装依赖，请使用 pnpm 
pnpm i 

# 本地开发 启动项目
pnpm dev

```

> 推荐使用 pnpm , **[pnpm的安装与使用](http://liqingsong.cc/article/detail/26)** 。


<br/>

启动完成后，打开浏览器访问 [http://127.0.0.1:8002](http://127.0.0.1:8002)， 你看到下面的页面就代表操作成功了。

![Home](/images/home.png)

接下来你可以修改代码进行业务开发了，本项目内创建了常见的`Demo`：页面模板、模拟数据、路由、数据请求等等，你可以继续阅读和探索左侧的其它文档。



## 目录结构 {#directory-structure}

本项目已经为你生成了一个完整的开发框架，下面是整个项目的目录结构。

```bash
├── src                        # Midway 生成的源代码目录
│   ├── config                 # 配置目录
│   │   ├── config.default.ts  # 默认配置
│   │   ├── config.local.ts    # 本地配置
│   │   ├── config.prod.ts     # 生产配置(prod优先级比production高)
│   │   └── config.production.ts # 生产配置
│   ├── controller             # Web Controller 目录
│   │   ├── api.controller.ts  # api Controller demo
│   │   └── home.controller.ts # 默认入口控制器（在此执行前端实现ssr）
│   ├── filter                 # 过滤器目录
│   ├── middleware             # 中间件目录
│   ├── service                # 服务逻辑目录
│   ├── configuration.ts       # Midway 配置入口
│   ├── interface.ts           # 全局ts 接口文件
│   └── vite.server.ts         # vite 服务文件
├── test                       # Midway 生成的测试目录
├── web                        # React 源代码
│   ├── @types                 # ts类型定义目录
│   │   ├── client.d.ts        # 客服端ts类型定义
│   │   ├── router.d.ts        # 前端路由类型定义
│   │   ├── server.d.ts        # 服务类型定义
│   │   ├── settings.d.ts      # 配置类型定义
│   │   └── store.d.ts         # store类型定义
│   ├── assets                 # 静态资源目录
│   │   ├── css                # 公用 CSS 样式目录
│   │   └── images             # 图片目录
│   ├── components             # React全局公用组件目录
│   ├── config                 # 配置目录
│   │   │── routes.tsx         # React路由配置入口
│   │   └── settings.ts        # 站点配置
│   ├── hooks                  # React hooks
│   ├── layout                 # React项目 layout
│   │   ├── DefaultLayout      # React项目默认Layout
│   │   │   ├── index.tsx      # DefaultLayout 模板入口
│   │   │   └── routes.ts      # 使用 DefaultLayout 的页面路由配置
│   │   └── BlankLayout.tsx    # 空 Layout
│   ├── pages                  # 页面组件目录(所有页面放在这里)
│   │   └── About              # 页面-关于(这里作为说明样例)
│   │       ├── components     # 当前页面组件目录(可选)
│   │       ├── hooks          # 当前页面hooks(可选)
│   │       ├── data.d.ts      # TS 类型定义文件(可选)
│   │       ├── index.tsx      # 当前页面入口
│   │       └── service.ts     # 当前页面数据接口文件(可选)
│   ├── public                 # 静态资源
│   ├── store                  # 全局 Store 数据模型目录（Pinia）
│   │   └── asyncDataContext   # 自定义页面asyncData createContext
│   ├── utils                  # 全局工具函数目录
│   ├── .env.development       # 开发环境变量配置
│   ├── .env.production        # 生产环境变量配置
│   ├── App.tsx                # App 入口
│   ├── entry-client.tsx       # 客户端入口文件
│   ├── entry-server.tsx       # 服务端入口文件
│   ├── index.html             # html模板
│   ├── tsconfig.json          # web目录ts配置文件
│   └── vite.config.ts         # vite 配置
├── .editorconfig              # 编辑配置
├── .eslintrc.json             # eslint 配置项
├── .gitignore                 # Git忽略文件配置
├── .prettierrc.js             # prettier 配置
├── jest.config.js             # jest config
├── bootstrap.js               # Midway 生产模式启动入口
├── package.json               # 项目信息
├── README.md                  # readme
└── tsconfig.json              # Midway的ts配置，不包括web目录
```

:::tip 
`src/config/config.local.ts` 本地开发环境重置，修改端口参数等等，已追加到 `.gitignore` 文件中，禁止提交。

`src/config/config.prod.ts`  本地生产环境重置，修改端口参数等等，已追加到 `.gitignore` 文件中，禁止提交。
:::

