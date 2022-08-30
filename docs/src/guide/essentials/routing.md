# 路由 {#index}

路由是组织起一个应用的关键骨架。

本项目路由分两部分：`React前端路由` 和 `Midway后端路由`。

## 触发机制 {#trigger-mechanism}

> 区分为以下两种情况

1、 当用户新打开页面或刷新页面：

`Midway后端获取浏览器请求地址，对应到后端路由`->`经过viteServerRender处理`->`React前端路由接管，加载对应页面`

2、当页面加载完成点击对应路由链接：

`React前端路由直接接管，加载对应页面`

## React路由配置参数 {#react-route-config-param}

:::tip
本项目定义了路由参数如下：
:::

```ts
/**
 * meta 自定义
 */
export interface IRouteMeta {
  // 标题，路由在菜单、浏览器title 或 面包屑中展示的文字
  title?: string;
  keywords?: string; // 关键字
  description?: string; // 说明
  navActive?: string; // 选中的导航
  parentPath?: string[]; // 所有父元素的path,下标key按照父元素的顺序
}

export type RouteComponent = React.FC<BrowserRouterProps> | (() => any);

/**
 * 路由类型
 */
export interface IRouter {
  path: string;
  meta?: IRouteMeta;
  redirect?: string;
  component?: RouteComponent;
  children?: IRouter[];
}
```

**示例：**

```ts{5-14}
import { lazy } from 'react';
import { IRouter } from '@/@types/router';

const LayoutRoutes: IRouter[] = [
  {
    path: '/',
    meta: {
      title: '首页',
      keywords: '首页k',
      description: '首页d',
      navActive: 'home',
    },
    component: lazy(() => import('@/pages/Home')),
  },
];
export default LayoutRoutes;
```


:::info Seo 说明

`title`: seo标题，html的title标签内容

`keywords`: seo关键字，html的meta标签name="keywords"的内容

`description`: seo说明，html的meta标签name="description"的内容

高级用法请查看 [seo 方法](/guide/essentials/seo.md)。
:::

:::info 


`navActive`：用于设置选中的导航栏，需要在`.tsx`模板中配合代码判断；此参数存在的原因，一般是因为详情页需要选中对应的导航，或其他特殊情况。

```tsx{16-37}
import { memo } from 'react';
import { Link } from 'react-router-dom';
import classnames from 'classnames';
import { useAsyncDataMetaContext } from '@/store/asyncDataContext';

import './css/index.less';

export interface DefaultLayoutProps {
  children: React.ReactNode;
}

export default memo(({ children }: DefaultLayoutProps) => {
  const meta = useAsyncDataMetaContext();
  return (
    <>
      <nav>
        <Link
          to={'/'}
          className={classnames({ active: meta.navActive === 'home' })}
        >
          Home
        </Link>
        |
        <Link
          to={'/about'}
          className={classnames({ active: meta.navActive === 'about' })}
        >
          About
        </Link>
        |
        <Link
          to={{ pathname: '/localapi', search: 'uid=10' }}
          className={classnames({ active: meta.navActive === 'localapi' })}
        >
          LocalApi
        </Link>
      </nav>
      {children}
    </>
  );
});

```

:::

## React路由 {#react-route}

本项目设计了一个react路由入口配置文件 `/web/config/routes.tsx`，然后分别把路由拆分到了不同的`/web/layouts`中去配置，这样做的原因：一是在入口文件方便集中处理重新格式化；二是模块化更规范。

**/web/config/routes.tsx**

```ts
/* 
 * # /web/config/routes.tsx
 */
/**
 * 路由配置 入口
 * @author LiQingSong
 */

import { lazy, memo, Suspense } from 'react';
import { useLocation, useRoutes } from 'react-router-dom';
import { createUseRoutes, pathKeyCreateUseRoutes } from '@/utils/router';

import PageLoading from '@/components/PageLoading';

// BlankLayout
import BlankLayout from '@/layouts/BlankLayout';

// DefaultLayout
import DefaultLayoutRoutes from '@/layouts/DefaultLayout/routes';
import DefaultLayout from '@/layouts/DefaultLayout';

import { IRouter } from '@/@types/router';

/**
 * 配置所有路由
 */
export const originalRoutes: IRouter[] = [
  {
    path: '/',
    children: DefaultLayoutRoutes,
  },
];

/**
 * 配置的路由转RouteObject[]
 */
export const routes = createUseRoutes([
  ...originalRoutes,
  {
    path: '*',
    component: lazy(() => import('@/pages/404')),
  },
]);

/**
 * 配置框架对应的路由
 */
const layoutToRoutes = {
  DefaultLayout: pathKeyCreateUseRoutes([routes[0]]),
};

export const SuspenseLazy = memo(
  ({ children }: { children: React.ReactNode }) => (
    <Suspense fallback={<PageLoading />}>{children}</Suspense>
  )
);

export default memo(() => {
  const routesElement = useRoutes(routes);
  const location = useLocation();

  // 属于 DefaultLayout
  if (layoutToRoutes.DefaultLayout[location.pathname]) {
    return (
      <DefaultLayout>
        <SuspenseLazy>{routesElement}</SuspenseLazy>
      </DefaultLayout>
    );
  }

  // 默认 BlankLayout
  return (
    <BlankLayout>
      <SuspenseLazy>{routesElement}</SuspenseLazy>
    </BlankLayout>
  );
});

};
```

**/web/layouts/DefaultLayout/routes.ts**

```ts
/* 
 * # /web/layouts/DefaultLayout/routes.ts 
 */
import { lazy } from 'react';
import { IRouter } from '@/@types/router';
// import About from '@/pages/About';

const LayoutRoutes: IRouter[] = [
  {
    path: '/',
    meta: {
      title: '首页',
      keywords: '首页k',
      description: '首页d',
      navActive: 'home',
    },
    component: lazy(() => import('@/pages/Home')),
  },
  {
    path: '/about',
    meta: {
      title: '关于',
      keywords: '关于k',
      description: '关于d',
      navActive: 'about',
    },
    //component: About,
    component: lazy(() => import('@/pages/About')),
  },
  {
    path: '/detail',
    meta: {
      title: '详情',
      keywords: '详情k',
      description: '详情d',
      navActive: 'about',
    },
    component: lazy(() => import('@/pages/Detail')),
  },
  {
    path: '/localapi',
    meta: {
      title: '请求本地api样列',
      keywords: '请求本地,api样列',
      description: '请求本地midway服务api样列',
      navActive: 'localapi',
    },
    component: lazy(() => import('@/pages/Localapi')),
  },
];

export default LayoutRoutes;
```

 `React路由` 详细规则请查看 [官方文档](https://reactrouter.com/)。


## Midway路由 {#midway-route}

> Midway 提供了这些装饰器: @Get 、 @Post 、 @Put() 、 @Del() 、 @Patch() 、 @Options() 、 @Head() 和 @All() ，表示各自的 HTTP 请求方法。

本项目Demo提供的样列在 `/src/controller/home.controller.ts` 中：

```ts
import { App, Controller, Get, Inject } from '@midwayjs/decorator';

import { Application, Context } from '@midwayjs/koa';

import { render } from '../vite.server';

@Controller('/')
export class HomeController {
  @App()
  app: Application;

  @Inject()
  ctx: Context;

  @Get('/')
  @Get('/about')
  @Get('/detail')
  @Get('/localapi')
  @Get('/404')
  async home(): Promise<void> {
    render(this.ctx, this.app, true);
  }
}

```

 `Midway路由` 详细规则请查看 [官方文档](http://www.midwayjs.org/docs/env_config)。





