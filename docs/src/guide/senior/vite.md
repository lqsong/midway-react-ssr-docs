# Vite 服务 {#index}

本项目是结合 `vite ssr` 实现的，所以在 `src` 目录下有个 `vite.server.ts` 文件。

vite.server分为以下几项：

## createViteServer 方法 {#create-vite-server}

此方法是为开发模式创建的，当处于开发环境时，此方法会启用vite server并返回。

```ts
let viteServer: vite.ViteDevServer;
export async function createViteServer(app: Application) {
  viteServer = !viteServer
    ? await vite.createServer({
        root: 'web',
        logLevel: 'error',
        server: {
          middlewareMode: true,
        },
      })
    : viteServer;

  app.use(koaConnect(viteServer.middlewares));

  return viteServer;
}
```

## renderDev 方法 {#render-dev}

此方法是为开发模式创建的，当处于开发环境时，此方法会render html。

```ts
/**
 * 开发模式 render
 * @param isStream 是否流式传输
 * @param ctx Context
 * @param viteServer vite 服务
 */
export async function renderDev(
  isStream: boolean,
  ctx: Context,
  viteServer: vite.ViteDevServer
) {
  try {
    let template = fs.readFileSync(resolve('../web/index.html'), 'utf-8');
    template = await viteServer.transformIndexHtml(ctx.originalUrl, template);
    const { render } = await viteServer.ssrLoadModule('/entry-server.tsx');
    let didError = false;
    const [stream, meta, state] = await render(ctx.originalUrl, {
      ...(isStream === true
        ? {
            onShellReady() {
              /**
               * 所有Suspense边界之上的内容都准备好了。
               */
              renderOnReady(ctx, template, didError, stream, meta, state);
              // ctx.res.end();
            },
          }
        : {
            onAllReady() {
              /**
               * 如果您不想要流式传输，请使用它而不是 onShellReady。
               * 这将在整个页面内容准备好后触发。
               * 您可以将其用于爬虫或静态生成。
               */
              renderOnReady(ctx, template, didError, stream, meta, state);
              ctx.res.end();
            },
          }),
      onShellError(err: any) {
        /**
         * 在我们完成 shell 之前发生了一些错误，所以我们发出了一个替代 shell。
         */
        ctx.res.statusCode = 500;
        ctx.res.write('<!doctype html><body>Loading...</body></html>');
        ctx.res.end();
        console.error(err);
      },
      onError(err: any) {
        didError = true;
        console.error(err);
      },
    });
  } catch (e) {
    viteServer && viteServer.ssrFixStacktrace(e);
    console.log(e.stack);
    ctx.throw(500, e.stack);
  }
}
```


## renderProd 方法 {#render-prod}

此方法是为生产模式创建的，当处于生产环境时，此方法会render html。

```ts
/** 
 * 生成模式 render
 * @param isStream 是否流式传输
 * @param ctx Context
 */
export async function renderProd(isStream: boolean, ctx: Context) {
  try {
    let didError = false;
    const [stream, meta, state] = await prodRender(ctx.originalUrl, {
      ...(isStream === true
        ? {
            onShellReady() {
              /**
               * 所有Suspense边界之上的内容都准备好了。
               */
              renderOnReady(ctx, template, didError, stream, meta, state);
              // ctx.res.end();
            },
          }
        : {
            onAllReady() {
              /**
               * 如果您不想要流式传输，请使用它而不是 onShellReady。
               * 这将在整个页面内容准备好后触发。
               * 您可以将其用于爬虫或静态生成。
               */
              renderOnReady(ctx, template, didError, stream, meta, state);
              ctx.res.end();
            },
          }),
      onShellError(err: any) {
        /**
         * 在我们完成 shell 之前发生了一些错误，所以我们发出了一个替代 shell。
         */
        ctx.res.statusCode = 500;
        ctx.res.write('<!doctype html><body>Loading...</body></html>');
        ctx.res.end();
        console.error(err);
      },
      onError(err: any) {
        didError = true;
        console.error(err);
      },
    });
  } catch (e) {
    ctx.throw(500, e.stack);
  }
}
```



