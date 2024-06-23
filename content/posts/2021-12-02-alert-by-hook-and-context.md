---
title: React--使用Hook和Context创建全局提示框
tags: [react]
date: 2021-12-02T13:00:24+08:00
---

做一个全局的提示框，是非常有必要的，在使用的地方直接调用，而非在每个页面都插入一个提示框组件，可以使用这种技术的还有modal、notfication、dialog、spinner。

<!--truncate-->

## 创建应用

环境准备，安装node、npm 或 yarn

```bash
npx create-react-app react-alert
cd react-alert
npm start
```

## 创建alert

创建context文件

```bash
mkdir src/context
touch src/context/alert-context.js
```

创建完成之后的目录

```bash
src
├ ...
├── context
│   └── alert-context.js
```

文件创建好了之后，接下来就要往里填充内容，修改alert-context.js, 先导入依赖

```jsx
import React, { useState, createContext, useContext, useEffect } from "react";
```

创建context对象

```jsx
const AlertContext = createContext();
```

定义alert组件

```jsx
const Alert = ({ onClick }) => {
  return (
    <div className="alert-wrapper">
      <div className="alert">
        <div>提示信息</div>
        <button onClick={onClick}>关闭</button>
      </div>
    </div>
  );
};
```

创建context provider

```jsx
  const AlertProvider = (props) => {
  const [isDisplaying, setIsDisplaying] = useState(false);

  const show = () => setIsDisplaying(true);
  const dismiss = () => setIsDisplaying(false);
  const onClick = () => setIsDisplaying(false);

  return (
    <AlertContext.Provider value={{ show, dismiss }} {...props}>
      {props.children}
      {isDisplaying && <Alert onClick={onClick} />}
    </AlertContext.Provider>
  );
};
```

AlertProvider创建完成，一般我们会用其包裹应用入口组件，一般是`<App />` 。但是，使用前需要导出，现在我们创建一个自定义 hook，将外部使用到的内容导出

```jsx
const useAlert = () => {
  return useContext(AlertContext);
};
export default { AlertContext, useAlert };
```

## 使用alert

下面就是如何使用以上制作的组件，首先配置AlertProvider，修改index.js

```jsx
import { AlertProvider } from "./context/alert-context";
// ...
ReactDOM.render(
  <AlertProvider>
    <App />
  </AlertProvider>,
  document.getElementById("root")
);
```

然后，在需要显示提示框，也就是需要调用show，dismiss的地方引入userAlert，修改App.js，在其中添加一个测试按钮

```jsx
import { useAlert } from './context/alert-context';
// ...
const alert = useAlert();
const onClick = () => alert.show();
// ...
<button onClick={onClick}>提示</button>;
```

以上就是基本的创建和使用，接下来可以对alert进行一点改造，使其更加完善

## 补充，给show添加参数

如果想让alert更加灵活，我们需要在show函数添加params 参数，最终将params传入alert，参数内容可以是title、icon等信息，修改alert-context.js

```jsx
//...
const [params, setParams] = useState();
//...
const show = (params) => {
  params && setParams(params);
  setIsDisplaying(true);
};
// ...
{isDisplaying && <Alert onClick={onClick} {...params} />;}
```

## 补充，按ESC键，隐藏提示框

使用useEffect, 在组件挂载时绑定ESC键，组件卸载时移除键绑定，在alert组件添加

```jsx
useEffect(() => {
  const bind = (e) => {
    if (e.keyCode !== 27) return;
    if (document.activeElement && ["INPUT", "SELECT"].includes(document.activeElement.tagName)) return;
    onClick && onClick();
  };
  document.addEventListener("keyup", bind);
  return () => document.removeEventListener("keyup", bind);
}, []);
```

至此，一个全局的提示框就处理好了，至于样式，每个应用的风格各不相同，可以自由发挥。
