# 异常处理 {#index}

因为是基于 Midway、React 组装的 SSR 框架，所以 Midway 自带了异常处理机制。

## Midway 异常处理 {#midway-exce}

你可以查看 [官网文档](http://www.midwayjs.org/docs/error_filter)。

## 404 处理 {#404-exce}

两种处理方式

### 第一种：直接Midway处理 {#404-exce-midway}

框架内部，如果未匹配到路由，会抛出一个 NotFoundError 的异常。通过异常处理器，我们可以自定义其行为。

比如跳转到某个页面，或者返回特定的结果：

```ts{10,13-15}
// src/filter/notfound.filter.ts
import { Catch } from '@midwayjs/decorator';
import { httpError, MidwayHttpError } from '@midwayjs/core';
import { Context } from '@midwayjs/koa';

@Catch(httpError.NotFoundError)
export class NotFoundFilter {
  async catch(err: MidwayHttpError, ctx: Context) {
    // 404 错误会到这里
    ctx.redirect('/404.html');

    // 或者直接返回一个内容
    return {
      code: 404,
      msg: err.message,
    }
  }
}
```

然后在 `src/configuration.ts` 中将404处理过滤器应用上

```ts{4,17}
import { Configuration, App, Catch } from '@midwayjs/decorator';
import { join } from 'path';
import * as koa from '@midwayjs/koa';
import { NotFoundFilter } from './filter/notfound.filter';

@Configuration({
  imports: [
    koa
  ],
})
export class ContainerConfiguration {

  @App()
  app: koa.Application;

  async onReady() {
    this.app.useFilter([NotFoundFilter]);
  }
}

```


### 第二种：Midway过度路由，React前端处理 {#404-exce-react}

Midway过度跳转到/404：

```ts{10}
// src/filter/notfound.filter.ts
import { Catch } from '@midwayjs/decorator';
import { httpError, MidwayHttpError } from '@midwayjs/core';
import { Context } from '@midwayjs/koa';

@Catch(httpError.NotFoundError)
export class NotFoundFilter {
  async catch(err: MidwayHttpError, ctx: Context) {
    // 404 错误会到这里
    ctx.redirect('/404');    
  }
}
```

然后在 `src/controller/home.controller.ts` 中绑定404路由

```ts{13}
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
  @ContentType('text/html')
  async home(): Promise<void> {
     render(this.ctx, this.app, true);
  }
}
```

然后在 `src/configuration.ts` 中将404处理过滤器应用上

```ts{4,17}
import { Configuration, App, Catch } from '@midwayjs/decorator';
import { join } from 'path';
import * as koa from '@midwayjs/koa';
import { NotFoundFilter } from './filter/notfound.filter';

@Configuration({
  imports: [
    koa
  ],
})
export class ContainerConfiguration {

  @App()
  app: koa.Application;

  async onReady() {
    this.app.useFilter([NotFoundFilter]);
  }
}

```

React前端接手路由处理

```tsx
// /web/config/router.tsx

export const routes = createUseRoutes([
  ...originalRoutes,
  {
    path: '/404',
    component: lazy(() => import('@/pages/404')),
  },
]);

```






## 500 处理 {#500-exce}

两种处理方式

### 第一种：直接Midway处理 {#500-exce-midway}

当不传递装饰器参数时，将捕获所有的错误。

比如，捕获所有的错误，并返回特定的 JSON 结构，示例如下。

```ts{11-14}
// src/filter/default.filter.ts

import { Catch } from '@midwayjs/decorator';
import { MidwayError } from '@midwayjs/core';
import { Context } from '@midwayjs/koa';

@Catch()
export class DefaultErrorFilter {
  async catch(err: MidwayError, ctx: Context) {
    console.log('err', err.cause);
    // 所有的未分类错误会到这里
    return {
      code: Number(err.code ?? 500),
      msg: err.message,
    };
  }
}
```

然后在 `src/configuration.ts` 中将500处理过滤器应用上

```ts{4,17}
import { Configuration, App, Catch } from '@midwayjs/decorator';
import { join } from 'path';
import * as koa from '@midwayjs/koa';
import { DefaultErrorFilter } from './filter/default.filter';

@Configuration({
  imports: [
    koa
  ],
})
export class ContainerConfiguration {

  @App()
  app: koa.Application;

  async onReady() {
    this.app.useFilter([DefaultErrorFilter]);
  }
}

```

### 第二种：Midway过度路由，React前端处理 {#500-exce-react}

Midway过度跳转到/500：

```ts{10}
// src/filter/default.filter.ts

import { Catch } from '@midwayjs/decorator';
import { Context } from '@midwayjs/koa';

@Catch()
export class DefaultErrorFilter {
  async catch(err: Error, ctx: Context) {
    // 500 错误会到这里
    ctx.redirect('/500'); 
  }

}
```

然后在 `src/controller/home.controller.ts` 中绑定500路由

```ts{13}
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
  @Get('/500')
  @ContentType('text/html')
  async home(): Promise<void> {
    render(this.ctx, this.app, true);
  }
}
```

然后在 `src/configuration.ts` 中将500处理过滤器应用上

```ts{4,17}
import { Configuration, App, Catch } from '@midwayjs/decorator';
import { join } from 'path';
import * as koa from '@midwayjs/koa';
import { DefaultErrorFilter } from './filter/default.filter';

@Configuration({
  imports: [
    koa
  ],
})
export class ContainerConfiguration {

  @App()
  app: koa.Application;

  async onReady() {
    this.app.useFilter([DefaultErrorFilter]);
  }
}

```

React前端接手路由处理

```ts{8-15}
// /web/config/router.tsx

export const routes = createUseRoutes([
  ...originalRoutes,
  {
    path: '/404',
    component: lazy(() => import('@/pages/404')),
  },
]);
```


