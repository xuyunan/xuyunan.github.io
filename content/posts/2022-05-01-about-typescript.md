---
title: 关于 TypeScript，需要了解的几点知识
tags: [TypeScript]
date: 2022-05-01T16:43:33+08:00
---

## 为什么要用 TypeScript

类型检查: ts最最明显的一个特性就是给js添加了类型，方便在多人协作或者项目时间长了以后，能够清晰的识读代码;   
IDE自动提示: 当我们在一个对象后敲入逗号时，IDE可以自动提示对应类型的属性、函数;

Vue3就是使用ts来重构的，这足以证明ts在前端生态中起着极其重要的作用，对于前端开发人员而言，很有必要去学习和使用。在使用的过程中，有些知识点易混淆，我们需要了解他们的区别，和各自使用的场景。

## 一些类型的区别

### undefined和null的区别

都表示空，判断语句中都标示false，undefined 表示一个变量声明了，但是没有赋值，null 用来表明一个变量是空值

```js
var testVar;
alert(testVar); //shows undefined
alert(typeof testVar); //shows undefined
```

```js
var testVar = null;
alert(testVar); //shows null
alert(typeof testVar); //shows object
```

undefined是一个空类型，转为数值时为NaN，null是一个空对象，转为数值时为0.

### any和unknown的区别

都表示未知类型，但是any是自由类型，unknown是类型安全的，unknown表示我暂时不知道这是什么类型，any表示我不关心这是什么类型 (unknown is I don't know; any is I don't care )。   
当一个变量被赋值了any，在接下来的使用过程中不会有什么警告、报错、提示，但是如果变量被赋值了unknown类型，那么在真正使用之前必须确定类型，否则就会编译报错，例如:

```ts
let vAny: any = 10;          // We can assign anything to any
let vUnknown: unknown =  10; // We can assign anything to unknown just like any 

let s1: string = vAny;      // Any is assignable to anything 
let s2: string = vUnknown;  // 报错，在没有断言之前，我们不能将unknown类型赋值给其它类型

vAny.method();              // Ok; anything goes with any
vUnknown.method();          // 报错，vUnknown实际类型没有确定
```

### interface和type的区别

interface 接口，用于定义类型，用来限制对象属性，在实际应用中，通常用于定义实体对象类型，更为常用   
type 定义别名，常用于复杂的复合类型

interface 示例

```ts
interface ButtonProps {
  title: string;
  size?: ButtonSize;
  btnType?: ButtonType;
}
```

type 示例

```ts
type ButtonSize = 'lg' | 'sm'
type ButtonType = 'primary' | 'default' | 'danger' | 'link'
```

## 关于定义文件 xxx.d.ts

使用declare关键字，为实现提供定义说明，将这些定义放置在xxx.d.ts文件中，用来让一些原本不支持ts的代码支持ts，
这样，原本不支持ts的库，通过安装类型声明，就可以在ts中使用这些三方库了，示例：

```bash
npm install jquery
npm install @types/jquery

npm install redux react-redux
npm install @types/react-redux
```

> 注意，如果使用的库，本身就是用ts写的，直接install即可，不必再安装type定义文件

参考   
[What is the difference between null and undefined in JavaScript?](https://stackoverflow.com/questions/5076944/what-is-the-difference-between-null-and-undefined-in-javascript)
['unknown' vs. 'any'](https://stackoverflow.com/questions/51439843/unknown-vs-any)

