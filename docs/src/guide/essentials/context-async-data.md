# Context(asyncDataContext) {#index}

在一个典型的 React 应用中，数据是通过 props 属性自上而下（由父及子）进行传递的，但此种用法对于某些类型的属性而言是极其繁琐的（例如：地区偏好，UI 主题），这些属性是应用程序中许多组件都需要的。Context 提供了一种在组件之间共享此类值的方式，而不必显式地通过组件树的逐层传递 props。

`asyncDataContext` 的存在是为了满足服务端渲染的同时服务客户端。

并且本项目在 `React函数组件` 原有的基础上扩展了 [asyncData 方法](/guide/essentials/async-data.md) 和 [seo 方法](/guide/essentials/seo.md)，`asyncDataContext`是其中必不可少的环节。

## asyncDataContext 类型 {#asyncdatacontext-type}

```ts
import { ParsedUrl } from 'query-string';
import { IRouteMeta } from './router';

export interface IAsyncDataContext<T> {
  paresUrl: ParsedUrl; // 当前页面路由经过query-string处理后的pareUrl
  meta: IRouteMeta; // 当前页面对应的meta
  app: any; // 当前应用 asyncData 的所有数据
  page: T; // 当前页面的 asyncData 数据
}

```

## asyncDataContext 提供的方法 {#asyncdatacontext-func}

```ts
/**
 * 返回 AsyncData集的所有数据与当前页面的AsyncData数据、meta、paresUrl
 * @returns IAsyncDataContext: {app, page}
 */
export const useAsyncDataContext = <T>(): IAsyncDataContext<T> => {
  return useContext<IAsyncDataContext<T>>(AsyncDataContext);
};

/**
 * 返回 当前页面ParesUrl
 * @returns any IAsyncDataContext.app
 */
export const useAsyncDataParesUrlContext = () => {
  return useContext(AsyncDataContext).paresUrl;
};

/**
 * 返回 当前页面meta
 * @returns any IAsyncDataContext.app
 */
export const useAsyncDataMetaContext = () => {
  return useContext(AsyncDataContext).meta;
};

/**
 * 返回 AsyncData集的所有数据
 * @returns any IAsyncDataContext.app
 */
export const useAsyncDataAppContext = () => {
  return useContext(AsyncDataContext).app;
};

/**
 * 返回 当前页面的AsyncData数据
 * @returns T IAsyncDataContext.page
 */
export const useAsyncDataPageContext = <T>(): T => {
  return useContext<IAsyncDataContext<T>>(AsyncDataContext).page;
};

// AsyncDataUpateContext
export const AsyncDataUpateContext = createContext<
  (val: IAsyncDataContext<any>) => void
>(() => {});

/**
 * 返回 AsyncData集 的修改函数 - 慎用
 * @returns (val: IAsyncDataContext<any>) => void
 */
export const useAsyncDataUpateContext = () => {
  return useContext(AsyncDataUpateContext);
};

/**
 * 返回 AsyncData当前页面数据的修改函数
 * @returns (val: T) => void
 */
export const useAsyncDataUpatePageContext = <T>() => {
  const location = useLocation();
  const asyncData = useAsyncDataContext<T>();
  const upateAsyncData = useContext(AsyncDataUpateContext);

  return (val: T) => {
    const app = [...asyncData.app];
    const appLen = app.length;
    if (appLen > 0) {
      app.splice(appLen - 1, 1, val);
    } else {
      app.push(val);
    }
    upateAsyncData({
      ...asyncData,
      paresUrl: {
        url: location.pathname,
        query: querystring.parse(location.search),
      },
      app,
      page: val,
    });
  };
};

/**
 * 返回 发送请求当前页面数据 dispatch
 * @param cb (config: IAsyncDataProps) => Promise<T> 回调函数传入当前页面page.asyncData()
 * @returns  [loading: boolean, dispatch: () => Promise<void>, location:Location, parsedUrl: querystring.ParsedUrl]
 */
export const useEffectDispatchAsyncDataPage = <T>(
  cb: (config: IAsyncDataProps) => Promise<T | undefined>
): [boolean, () => Promise<void>, Location, querystring.ParsedUrl] => {
  const location = useLocation();
  const parsedUrl = useAsyncDataParesUrlContext();
  const upateAsyncDataPage = useAsyncDataUpatePageContext<T>();
  const [loading, setLoading] = useState<boolean>(false);

  const dispatch = async () => {
    setLoading(true);
    try {
      const asyncDataConfig: IAsyncDataProps = {
        paresUrl: {
          url: location.pathname,
          query: querystring.parse(location.search),
        },
      };
      const data = await cb(asyncDataConfig);
      data && upateAsyncDataPage(data);
    } catch (error: any) {
      console.log(error);
    }
    setLoading(false);
  };

  return [loading, dispatch, location, parsedUrl];
};

```

## asyncDataContext 使用 {#asyncdatacontext-use}

项目默认嵌入并且结合 [asyncData 方法](/guide/essentials/async-data.md)，进行传递。

以下为样列：



```tsx{4-7,28,43-54}
import { memo, useEffect, useMemo } from 'react';
import { Link, useSearchParams } from 'react-router-dom';
import { Spin } from 'antd';
import {
  useAsyncDataPageContext,
  useEffectDispatchAsyncDataPage,
} from '@/store/asyncDataContext';

import PaginationBase from '@/components/Pagination/Base';

import { ResponseData } from '@/utils/request';
import { queryList } from './service';
import { ResponseDataType, TableListItem, ITableData } from './data';

import styles from './index.module.less';

import { IServerPage } from '@/@types/server';

const About: IServerPage<ITableData> = () => {
  const [searchParams] = useSearchParams();
  // 获取当前页码
  const page = useMemo(
    () => Number(searchParams.get('page') || 1),
    [searchParams]
  );

  // 取出数据
  const asyncDataStore = useAsyncDataPageContext<ITableData>();
  const list = useMemo<TableListItem[]>(
    () => asyncDataStore.list || [],
    [asyncDataStore]
  );
  const total = useMemo<number>(
    () => asyncDataStore?.pagination?.total || 0,
    [asyncDataStore]
  );
  const current = useMemo<number>(
    () => asyncDataStore?.pagination?.current || 1,
    [asyncDataStore]
  );

  // 生成请求数据 dispatch
  const [loading, dispatch, location, parsedUrl] =
    useEffectDispatchAsyncDataPage<ITableData>(async config => {
      return About.asyncData && (await About.asyncData(config));
    });

  // 客户端 - 请求数据
  useEffect(() => {
    const parsedUrlQueryPage = Number(parsedUrl.query.page || 1);
    if (parsedUrl.url === location.pathname && parsedUrlQueryPage !== page) {
      dispatch();
    }
  }, [page]);

  return (
    <div className="about">
      <h1>This is an about page</h1>
      <Spin spinning={loading}>
        <div className={styles.box}>
          <ul>
            {list.map(item => (
              <li key={item.id}>
                <Link to={`/detail?id=${item.id}`}>{item.title}</Link>
                <span>{item.addtime}</span>
              </li>
            ))}
          </ul>
        </div>
        <div>
          <PaginationBase
            total={total}
            currentPage={current}
            pageUrl="/about?page={page}"
          />
        </div>
      </Spin>
    </div>
  );
};

About.asyncData = async ({ paresUrl }) => {
  const current = Number(paresUrl.query.page || 1);
  const response: ResponseData<ResponseDataType> = await queryList({
    current,
  });
  const data = response.data || { list: [], total: 0 };
  return {
    list: data.list || [],
    pagination: {
      total: data.total || 0,
      current,
      pageSize: 10,
    },
  };
};

export default memo(About);


```





