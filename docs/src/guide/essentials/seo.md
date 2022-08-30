# seo 方法 {#index}

`midway-react-ssr` 扩展了 `React函数组件`，增加了一个叫 seo 的方法，使得我们可以在设置组件的数据之前能设置或重置当前页面的seo。

## 说明 {#illustrate}

`seo`方法会在组件每次加载之前被调用。它可以在服务端或当前页面组件加载之前，[asyncData 方法](/guide/essentials/async-data.md)之后被调用。在这个方法被调用的时候，参数被设定为 `ISeoProps`类型，你可以利用 seo方法来设置或重置当前页面的seo，框架会将 seo 返回的数据与[路由](/guide/essentials/routing.md#react-route-config-param)中设置的seo融合，最后一并设置到当前页面。

:::tip 注意：

如果设置了seo方法，seo方法必须返回`ISeoResponse`类型，哪怕是一个空的`ISeoResponse`类型。

seo方法返回的`ISeoResponse`字段优先级比[路由](/guide/essentials/routing.html#react-route-config-param)中设置的seo字段高。

seo方法存在的原因是为了解决特殊需求，如：详情页面的title需要根据标题进行显示。一般情况下[路由](/guide/essentials/routing.html#react-route-config-param)中seo字段已经够用。

:::

### 类型 {#illustrate-type}

```ts{18-20,25-29,39}
import { ParsedUrl } from 'query-string';
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
 * 自定义扩展方法seo的参数类型
 * <T> asyncData方法返回的数据类型
 */
export interface ISeoProps<T> extends IServerFunProps {
  asyncData: T; // asyncData方法返回的数据
}

/**
 * 自定义扩展方法seo返回数据类型
 */
export interface ISeoResponse {
  title?: string;
  keywords?: string;
  description?: string;
}

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

```tsx{29-33}
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


