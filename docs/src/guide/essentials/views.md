# 视图 {#index}

本项目视图分两部分：`React前端页面` 和 `Midway后端模板渲染`。

## React html模板 {#react-html}

你可以修改默认的React html模板。

默认的React html模板，位置在 `/web/index.html`,默认内容如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title><!--app-title--></title>
  <meta name="keywords" content="!--app-keywords--">
  <meta name="description" content="!--app-description--">

  <link rel="icon" href="/favicon.ico" />

</head>
<body>
<div id="app"><!--app-html--></div>
<script type="module" src="/entry-client.tsx"></script>
<script>
  window.__INITIAL_DATA__ = "<!--app--state-->";
</script>
</body>
</html>


```

举个例子，你可以添加自定义的`script`标签或代码块等其他内容：

```html{20}
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title><!--app-title--></title>
  <meta name="keywords" content="!--app-keywords--">
  <meta name="description" content="!--app-description--">

  <link rel="icon" href="/favicon.ico" />

</head>
<body>
<div id="app"><!--app-html--></div>
<script type="module" src="/entry-client.tsx"></script>
<script>
  window.__INITIAL_DATA__ = "<!--app--state-->";
</script>

<script src="http://echarts.com/echarts.js"></script>

</body>
</html>
```

::: warning 
注意：`<!--app-title-->`、`!--app-keywords--`、`!--app-description--`、`<!--app-html-->`、`<!--app--state-->`等插槽名称切记不要修改或删除。
:::


## React 布局 {#react-layout}

在 `/web/layouts` 目录下你可以修改默认布局或创建自定义的布局。

::: tip 
 别忘了在布局文件中添加 `{children}` 组件用于显示页面的主体内容。
:::

### 默认布局 {#react-layout-default}

`/web/layouts/DefaultLayout/index.tsx` 源码如下：

```tsx
<script lang="ts" setup>
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

### 自定义布局 {#react-layout-customize}

假设我们要创建一个 **博客布局** 并将其保存到 `/web/layouts/BlogLayout/index.tsx`:

```tsx
import { memo } from 'react';

export interface BlogLayoutProps {
  children: React.ReactNode;
}

export default memo(({ children }: BlogLayoutProps) => {
  return (
    <>
       <div>这个是我的博客导航栏</div>
      {children}
    </>
  );
});

```

然后我们必须再次创建一个 `/web/layouts/BlogLayout/routes.ts` 文件设置路由并确定哪些页面用到此布局：

```ts
import { lazy } from 'react';
import { IRouter } from '@/@types/router';
// import About from '@/pages/About';

const LayoutRoutes: IRouter[] = [
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
];

export default LayoutRoutes;


```

最后我们需要到 `/web/config/router.tsx` 中配置上：

```tsx
/**
 * 配置所有路由
 */
export const originalRoutes: IRouter[] = [
  {
    path: '/',
    children: DefaultLayoutRoutes,
  },
  {
    path: '/blog',
    children: BlogLayoutRoutes,
  },
];

/**
 * 配置框架对应的路由
 */
const layoutToRoutes = {
  BlogLayout: pathKeyCreateUseRoutes([routes[1]]),
};

export default memo(() => {
  const routesElement = useRoutes(routes);
  const location = useLocation();

  // 属于 BlogLayout
  if (layoutToRoutes.BlogLayout[location.pathname]) {
    return (
      <BlogLayout>
        <SuspenseLazy>{routesElement}</SuspenseLazy>
      </BlogLayout>
    );
  }

});
```



## React 页面 {#react-page}

页面组件实际上是 React 组件，只不过 `midway-react-ssr` 为这些组件添加了一些特殊的配置项（对应 `midway-react-ssr` 提供的功能特性）以便你能快速开发通用应用。

```tsx{10,23-33}
import { memo } from 'react';
import { useAsyncDataPageContext } from '@/store/asyncDataContext';

import { ResponseData } from '@/utils/request';
import { queryDetail } from './service';
import { Article } from './data';

import { IServerPage } from '@/@types/server';

const Detail: IServerPage<Article> = () => {
  // 取出数据
  const article = useAsyncDataPageContext<Article>();

  return (
    <div className="detail">
      <p>{article.title}</p>
      <p>{article.addtime}</p>
      <p>{article.content}</p>
    </div>
  );
};

Detail.asyncData = async ({ paresUrl }) => {
  const id = paresUrl.query.id?.toString() || '1';
  const response: ResponseData<Article> = await queryDetail(id);
  return response.data || {};
};

Detail.seo = ({ asyncData }) => {
  return {
    title: asyncData.title + '-详情测试',
  };
};

export default memo(Detail);

```

`midway-react-ssr` 在 `react 函数组件` 原有的基础上扩展了 [asyncData 方法](/guide/essentials/async-data.md) 和 [seo 方法](/guide/essentials/seo.md)，使用规则你可以点击链接查看明细。

::: tip 
 为了TS友好，请一定绑定 `IServerPage`类型。
:::


## Midway模板渲染 {#midway-html}

使用此框架你也可以不用 `React ssr`，选择原始的模板渲染。

 `Midway模板渲染` 详细规则请查看 [官方文档](http://www.midwayjs.org/docs/extensions/render)。

