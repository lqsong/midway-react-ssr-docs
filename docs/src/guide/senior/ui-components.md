# UI 组件 {#index}

`midway-react-ssr` 默认集成了 `Ant Design` UI组件库，你可以删除它换成以下其它的组件库。


## Ant Design {#ant-design}

Ant Design ，是基于 Ant Design 设计体系的 React UI 组件库，主要用于研发企业级中后台产品。

### 安装 {#ant-design-install}

```sh
pnpm add antd 
```

### 引入 {#ant-design-import}

```tsx
import React, { FC } from 'react';
import { Button } from 'antd';
import 'antd/dist/antd.css';

const App: FC = () => (
  <div className="App">
    <Button type="primary">Button</Button>
  </div>
);

export default App;
```



详细内容请查看 [官方文档](https://ant.design/index-cn) 、 [Github](https://github.com/ant-design/ant-design/)。




## Tdesign React {#tdesign-react}

鹅厂优质 UI 组件，配套工具完整，设计工整，文档清晰。

### 安装 {#tdesign-react-install}

```sh
pnpm add tdesign-react
```

### 引入 {#tdesign-react-import}

```tsx
import React, { FC } from 'react';
import { Button } from 'tdesign-react';
import 'tdesign-react/es/style/index.css'; // 少量公共样式

const App: FC = () => (
  <div className="App">
    <Button type="primary">Button</Button>
  </div>
);

export default App;
```


详细内容请查看 [官方文档](https://tdesign.tencent.com/react/getting-started) 、 [Github](https://github.com/Tencent/tdesign-react)。



## Arco Design React {#arco-design-react}

字节跳动 UI 组件库开源，大厂逻辑，设计文档完美。

### 安装 {#arco-design-react-install}

```sh
pnpm add  @arco-design/web-react
```

### 引入 {#arco-design-react-import}

```tsx
import React, { FC } from 'react';
import { Button } from "@arco-design/web-react";
import "@arco-design/web-react/dist/css/arco.css";

const App: FC = () => (
  <div className="App">
    <Button type="primary">Button</Button>
  </div>
);

export default App;
```


详细内容请查看 [官方文档](https://arco.design/react/docs/start) 、 [Github](https://github.com/arco-design)。



## Vant {#vant}

有赞团队开源，并由社区团队维护，轻量、可靠的移动端 React 组件库。


### 安装 {#vant-install}

```sh
pnpm add react-vant
```

### 引入 {#vant-import}

```tsx
import React, { FC } from 'react';
import { Button } from 'react-vant';

const App: FC = () => (
  <div className="App">
    <Button type="primary">Button</Button>
  </div>
);

export default App;
```


详细内容请查看 [官方文档](https://react-vant.3lang.dev/) 、 [Github](https://github.com/3lang3/react-vant)。




