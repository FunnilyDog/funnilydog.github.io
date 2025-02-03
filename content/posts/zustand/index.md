---
title: "React 中使用 Zustand 状态管理库 及其源码解析"
date: 2025-01-13T13:55:00+08:00
draft: false
summary: "文章主要介绍了Zustand这一React状态管理库，包括其简介、优势、使用方法、与Redux对比、踩坑点、中间件、选择理由、工作原理等。Zustand轻量简洁，API友好，支持TypeScript，具有性能优化、灵活可扩展等优点，可轻松集成和处理异步操作，能完美替代某些传统状态管理库的不足，在实际使用中有多种优化和配置方式。"
tags: ["react", "zustand", "状态管理"]
---

## 快速上手

Zustand 是基于 React Hook 构建的状态管理库，它的设计理念是让状态管理变得简单、直观。只需寥寥数行代码，就能轻松管理应用的状态。

### 特点

- 轻量级：Zustand 的体积非常小，几乎没有任何依赖，生产级包体积不到 1kb，这使得它在项目中引入时不会带来过多的负担，能够保持项目的轻量化。
- 基于 React Hook：完全基于 React Hook 构建，契合函数式编程，开发者可以利用熟悉的 Hook 语法来管理状态，避免了复杂的类组件写法和样板代码。
- 简单易用：API 简洁明了，有完备的使用文档，源码清晰且不多，学习成本低、曲线顺畅。
- 高性能：能够精确地追踪状态的变化，只重新渲染依赖该状态的组件，从而提高应用的性能和响应速度。
- 可扩展性：支持中间件和插件机制，开发者可以根据项目需求轻松地扩展其功能，如添加日志记录、持久化状态等。

### 安装 Zustand

使用 npm 或 yarn 安装 Zustand，为项目引入这个强大的状态管理工具：

```zsh
npm install zustand
# 或者
yarn add zustand
```

### 创建第一个状态

以一个简单的计数器为例，看看如何使用 Zustand 创建和管理状态：

```javascript
import create from "zustand";

const useCounter = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 }))
}));
```

在这段代码中，`create`函数接收一个回调函数，通过`set`方法来更新状态。`set`方法既可以接收一个新的状态对象，也可以接收一个函数，该函数接收当前状态并返回新状态，确保状态更新的不可变性。

### 在 React 使用

在 React 中，通过`useCounter`这个自定义 Hook 来获取和更新状态：

```javascript
import React from "react";
import useCounter from "./useCounter";

const CounterComponent = () => {
  const { count, increment, decrement } = useCounter();
  //   const increment = useCounter((state) => state.increment);
  //   const decrement = useCounter((state) => state.decrement);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
      <button onClick={decrement}>Decrement</button>
    </div>
  );
};

export default CounterComponent;
```

这里使用`useCounter`的回调函数，精准地获取和操作所需的状态。这种方式被称为“selector”，它可以只返回组件关心的状态部分，避免不必要的重新渲染，极大地提升了性能。

## 处理复杂状态

### 复杂状态管理与 Immer 的结合

当处理复杂的状态结构时，Zustand 可以与 Immer 库结合使用，让状态更新变得更加简洁和直观。Immer 允许我们以可变的方式编写代码，而底层会自动处理不可变数据的更新。

```javascript
import create from "zustand";
import produce from "immer";

const useTodoStore = create((set) => ({
  todos: [],
  addTodo: (text) =>
    set(
      produce((state) => {
        state.todos.push({ id: Date.now(), text, completed: false });
      })
    ),
  toggleTodo: (id) =>
    set(
      produce((state) => {
        const todo = state.todos.find((t) => t.id === id);
        if (todo) {
          todo.completed = !todo.completed;
        }
      })
    )
}));
```

通过 Immer 的`produce`函数，我们可以像操作普通对象一样更新状态，而无需手动处理复杂的不可变数据结构。

### 中间件

Zustand 支持中间件，通过中间件可以轻松扩展状态管理的功能。例如，使用`zustand - middleware - logger`中间件来记录状态的变化，方便调试：

```bash
npm install zustand-middleware-logger
# 或者
yarn add zustand-middleware-logger
```

```javascript
import create from "zustand";
import logger from "zustand-middleware-logger";

const useCounter = create(
  logger((set) => ({
    count: 0,
    increment: () => set((state) => ({ count: state.count + 1 })),
    decrement: () => set((state) => ({ count: state.count - 1 }))
  }))
);
```

添加中间件后，每次状态更新时，控制台都会输出详细的日志信息，帮助我们快速定位问题。

## 源码浅析（react 部分）

### create

`create`函数是 Zustand 的核心，它负责创建状态管理实例。其基本实现思路是创建一个包含状态和更新方法的对象，并返回一个自定义 Hook。这个 Hook 内部维护了一个状态的订阅者列表，当状态发生变化时，会通知所有订阅者重新渲染。

```javascript
function create(setter) {
  let state;
  let listeners = new Set();

const setState: StoreApi<TState>['setState'] = (partial, replace) => {

    const nextState =
      typeof partial === 'function'
        ? (partial as (state: TState) => TState)(state)
        : partial
    if (!Object.is(nextState, state)) {
      const previousState = state
      state =
        (replace ?? (typeof nextState !== 'object' || nextState === null))
          ? (nextState as TState)
          : Object.assign({}, state, nextState)
      listeners.forEach((listener) => listener(state, previousState))
    }
  }
  const getState: StoreApi<TState>['getState'] = () => state

  const subscribe: StoreApi<TState>['subscribe'] = (listener) => {
    listeners.add(listener)
    // Unsubscribe
    return () => listeners.delete(listener)
  }

  const getInitialState: StoreApi<TState>['getInitialState'] = () =>
    initialState

  const initialState = setter(setState);

  const api = { setState, getState, getInitialState, subscribe }
  const initialState = (state = createState(setState, getState, api))

  return api as any
}
```

在这个简化的实现中， `create`函数接收一个`setter`函数，通过`setState`方法来更新状态，并通知所有订阅者。返回的自定义 Hook 则负责根据`selector`获取状态，并在状态变化时重新计算`selector`的值。

### useStore Hook

`useStore` Hook 是 Zustand 与 React 组件交互的桥梁。它通过 React 的`useState`和`useEffect` Hook 来实现状态的订阅和更新。当组件使用`useStore`时，会将自身添加到状态的订阅者列表中，当状态发生变化时，会触发组件的重新渲染。

```ts
import { useSyncExternalStore } from "react";

const identity = <T>(arg: T): T => arg;

export function useStore<TState, StateSlice>(
  api: ReadonlyStoreApi<TState>,
  selector: (state: TState) => StateSlice = identity as any
) {
  const slice = useSyncExternalStore(
    api.subscribe,
    () => selector(api.getState()),
    () => selector(api.getInitialState())
  );

  React.useDebugValue(slice);
  return slice;
}
```

在实际的 Zustand 源码中，`useStore`的实现更加复杂，它还处理了`selector`的缓存、中间件的调用等功能，但基本原理与上述代码类似。

## 总结

Zustand 以其简洁的 API、高效的性能和灵活的扩展性，为 React 开发者提供了一种优秀的状态管理解决方案。通过深入了解 Zustand 在 React 中的使用方法和源码实现，我们不仅能够更好地运用这一工具，还能从中汲取设计灵感，提升自己的编程能力。

在未来，随着 React 生态的不断发展，相信 Zustand 也会持续演进，为开发者带来更多便利和惊喜。无论是小型项目的快速迭代，还是大型应用的复杂状态管理，Zustand 都值得我们深入学习和使用。

希望本文能帮助你对 Zustand 有更深入的理解，如果你在使用 Zustand 的过程中有任何问题或心得，欢迎在评论区分享交流！
