# asyncData 方法 {#index}

`midway-react-ssr` 扩展了 `React函数组件`，增加了一个叫 asyncData 的方法，使得我们可以在设置组件的数据之前能获取或处理数据。

## 说明 {#illustrate}

`asyncData`方法会在组件每次加载之前被调用。它可以在服务端或当前页面组件加载之前被调用。在这个方法被调用的时候，参数被设定为 `IAsyncDataProps`类型，你可以利用 asyncData方法来获取数据。


### 类型 {#illustrate-type}

```ts{11,20}
/**
 * 自定义扩展方法的公共参数类型
 */
export interface IServerFunProps {
  paresUrl: ParsedUrl; // 当前地址栏url paresUrl格式后的值
}

/**
 * 自定义扩展方法asyncData的参数类型
 */
export type IAsyncDataProps = IServerFunProps;

/**
 * 字段莹莹页面函数组件定义类型
 * <T> 函数组件 props 类型
 * <D> 函数组件 asyncData方法 返回数据的类型
 */
export interface IServerPage<D = any, T = any> {
  (props: T): JSX.Element;
  asyncData?(config: IAsyncDataProps): Promise<D>;
  seo?(config: ISeoProps<D>): ISeoResponse;
}
```

## 使用 {#use}



### async {#use-async}

```tsx{7-21}
import { ResponseData } from '@/utils/request';
import { ResponseDataType, TableListItem, ITableData } from './data';

import { IServerPage } from '@/@types/server';
const About: IServerPage<ITableData> = () => {};

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

### 数据展示 {#use-data}

```tsx{12, 40-61}
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
```

### 监听 query 参数改变 {#use-query}

默认情况下，query 的改变不会调用 `asyncData` 方法。如果需要，可以监听这个行为。例如，在构建分页组件时，您可以设置 `useEffect page` 监听参数。如下样列：

```tsx{27,33-38}
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
```















