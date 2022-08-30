# 配置 {#index}

配置是开发一个项目的重要环节，它是一个项目的基础。想要了解一个项目，先要了解它的配置。

## web前端站点配置 {#web-site-config}

`midway-react-ssr` 内置了一个web前端站点配置文件 `/web/config/settings.ts`。

```ts
/**
 * 站点配置
 * @author LiQingSong
 */
export interface SettingsType {
  /**
   * 站点名称
   */
  siteTitle: string;

  /**
   * 站点本地存储Token 的 Key值
   */
  siteTokenKey: string;

  /**
   * Ajax请求头发送Token 的 Key值
   */
  ajaxHeadersTokenKey: string;

  /**
   * Ajax返回值不参加统一验证的api地址
   */
  ajaxResponseNoVerifyUrl: string[];
}

const settings: SettingsType = {
  siteTitle: 'MIDWAY-REACT-SSR',

  siteTokenKey: 'midway_react_ssr_token',
  ajaxHeadersTokenKey: 'x-token',
  ajaxResponseNoVerifyUrl: [
    '/user/login', // 用户登录
  ],
};

export default settings;


```

## web前端路由入口配置 {#web-route-config}

`midway-react-ssr` 独立出了一个前端路由入口配置文件 `/web/config/routes.tsx`，其目的主要是：统一引入`/web/layouts`下不同`layout`的路由，集中处理重新格式化路由。

目前 `/web/config/routes.tsx` 的内容为：

```ts
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

```

::: tip 说明：
详细文档请查看：[路由](/guide/essentials/routing.md)
:::



## vite 配置 {#web-vite-config}

`midway-react-ssr` web前端是基于 `vite` 来进行构建，所以有个 vite 配置文件 `/web/vite.config.ts`。

[官方文档](https://cn.vitejs.dev/config/)


## web前端环境变量 {#web-env-config}
`midway-react-ssr` web前端是基于 `vite` 来进行构建，所以有环境变量配置文件 `/web/.env.development`、`/web/.env.production`。

[官方文档](https://cn.vitejs.dev/guide/env-and-mode.html)

## midway 多环境配置 {#midway-env-config}

`midway-react-ssr` Node.js服务端是基于 `Midway` 来进行构建，所以有多环境配置配置文件 `/src/config/**`。

[官方文档](http://www.midwayjs.org/docs/env_config)

