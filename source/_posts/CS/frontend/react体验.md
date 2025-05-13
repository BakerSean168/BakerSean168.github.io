---
title: react体验
categories:
  - null
description: react体验，尝试做一个 react demo
date: 2025-05-12 10:46:44
tags:
---

[React 中文文档](https://zh-hans.react.dev/learn)

# 前置知识

## JSX

JSX 是 JavaScript 语法扩展，可以让你在 JavaScript 文件中书写类似 HTML 的标签。  
虽然还有其它方式可以编写组件，但大部分 React 开发者更喜欢 JSX 的简洁性，并且在大部分代码库中使用它。

_JSX and React 是相互独立的 东西。但它们经常一起使用，但你 可以 单独使用它们中的任意一个，JSX 是一种语法扩展，而 React 则是一个 JavaScript 的库。_

### JSX 规则

```js
<h1>海蒂·拉玛的待办事项</h1>
<img
  src="https://i.imgur.com/yXOvdOSs.jpg"
  alt="Hedy Lamarr"
  class="photo"
>
<ul>
  ...
</ul>
```

#### 1. 只能返回一个根元素

例子：
  上方代码外侧必须使用`<div></div`或`<></>`标签包裹  

原因：  
 JSX 虽然看起来很像 HTML，但在底层其实被转化为了 JavaScript 对象，你不能在一个函数中返回多个对象，除非用一个数组把他们包装起来。

#### 2. 标签必须闭合

`<img ...>` `<img .../>`

#### 3. 使用驼峰式命名法给大部分属性命名！

JSX 最终会被转化为 JavaScript，而 JSX 中的属性也会变成 JavaScript 对象中的键值对。  
在你自己的组件中，经常会遇到需要用变量的方式读取这些属性的时候。但 JavaScript 对变量的命名有限制。  
  例如，变量名称不能包含 - 符号或者像 class 这样的保留字。
这就是为什么在 React 中，大部分 HTML 和 SVG 属性都用驼峰式命名法表示。  
  例如，需要用 strokeWidth 代替 stroke-width。由于 class 是一个保留字，所以在 React 中需要用 className 来代替。
```js
<img 
  src="https://i.imgur.com/yXOvdOSs.jpg" 
  alt="Hedy Lamarr" 
  className="photo"
/>
```

**建议使用 [转化器](https://transform.tools/html-to-jsx) 将 HTML 和 SVG 标签转化为 JSX。**

## 添加样式  

React 并没有规定你如何添加 CSS 文件。

## 显示数据

通过 `{}`（转义到JavaScript）
```js
const user = {
  name: 'Hedy Lamarr',
  imageUrl: 'https://i.imgur.com/yXOvdOSs.jpg',
  imageSize: 90,
};

export default function Profile() {
  return (
    <>
      <h1>{user.name}</h1>
      <img
        className="avatar"
        src={user.imageUrl}
        alt={'Photo of ' + user.name}
        style={{
          width: user.imageSize,
          height: user.imageSize
        }}
      />
    </>
  );
}

```

## 条件渲染

React 没有特殊的语法来编写条件语句，因此你使用的就是普通的 JavaScript 代码。  
- if-else
- a ? b : c
- a && b

## 渲染列表

- for
- map

## 响应事件

```js
function MyButton() {
  function handleClick() {
    alert('You clicked me!');
  }

  return (
    <button onClick={handleClick}>
      点我
    </button>
  );
}
```

## 更新界面

```js
import { useState } from 'react';

function MyButton() {
  const [count, setCount] = useState(0);
  // count 为当前值， setCount 为用于更新它的函数
  function handleClick() {
    setCount(count + 1);
  }

  return (
    <button onClick={handleClick}>
      Clicked {count} times
    </button>
  );
}
```

## Hook

- 以 use 开头的函数被称为 Hook。  
- useState 是 React 提供的一个内置 Hook。  
- 可以在 [React API 参考](https://zh-hans.react.dev/reference/react) 中找到其他内置的 Hook。  
- 可以通过组合现有的 Hook 来编写属于你自己的 Hook。

Hook 比普通函数更为严格。  
  只能在你的组件（或其他 Hook）的 顶层 调用 Hook。  
如果想在一个条件或循环中使用 useState，请提取一个新的组件并在组件内部使用它。

## 组件间共享数据

父组件使用 JSX 的大括号向 MyButton 传递信息：  
```js
export default function MyApp() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <div>
      <h1>共同更新的计数器</h1>
      <MyButton count={count} onClick={handleClick} /> 
      <MyButton count={count} onClick={handleClick} />
    </div>
  );
}
```

子组件通过参数读取：  
```js
function MyButton({ count, onClick }) {
  return (
    <button onClick={onClick}>
      点了 {count} 次
    </button>
  );
}
```

# 创建 demo

## 配置

从零开始构建 React 应用

构建工具： vite

react + typescript、swc

## 注意

### onClick to button

DOM `<button>` 元素的 onClick props 对 React 有特殊意义，因为它是一个内置组件。  
对于像 Square 这样的自定义组件，命名由你决定。你可以给 Square 的 onSquareClick props 或 Board 的 handleClick 函数起任何名字，代码还是可以运行的。  
在 React 中，通常使用 onSomething 命名代表事件的 props，使用 handleSomething 命名处理这些事件的函数。

### 不变性很重要

创建新副本，并用新副本来替换原始数据；而非直接更新原始数据。  

好处：  
- 使复杂功能更容易实现  
  回溯  
- 方便组件比较数据是否更改，以跳过部分内容的重新渲染  

## 代码

```tsx
import { useState } from 'react';


function Square({ value, onSquareClick } : { value: string | null; onSquareClick: () => void }) {
  return (
    <button className="square" onClick={onSquareClick}>
      {value}
    </button>
  );
}

function Board({ xIsNext, squares, onPlay } : { xIsNext: boolean; squares: Array<string | null>; onPlay: (squares: Array<string | null>) => void }) {
  function handleClick(i : number) {
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    const nextSquares = squares.slice();
    if (xIsNext) {
      nextSquares[i] = 'X';
    } else {
      nextSquares[i] = 'O';
    }
    onPlay(nextSquares);
  }

  const winner = calculateWinner(squares);
  let status;
  if (winner) {
    status = 'Winner: ' + winner;
  } else {
    status = 'Next player: ' + (xIsNext ? 'X' : 'O');
  }

  return (
    <>
      <div className="status">{status}</div>
      <div className="board-row">
        <Square value={squares[0]} onSquareClick={() => handleClick(0)} />
        <Square value={squares[1]} onSquareClick={() => handleClick(1)} />
        <Square value={squares[2]} onSquareClick={() => handleClick(2)} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} onSquareClick={() => handleClick(3)} />
        <Square value={squares[4]} onSquareClick={() => handleClick(4)} />
        <Square value={squares[5]} onSquareClick={() => handleClick(5)} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} onSquareClick={() => handleClick(6)} />
        <Square value={squares[7]} onSquareClick={() => handleClick(7)} />
        <Square value={squares[8]} onSquareClick={() => handleClick(8)} />
      </div>
    </>
  );
}

export default function Game() {
  const [history, setHistory] = useState([Array(9).fill(null)]);
  const [currentMove, setCurrentMove] = useState(0);
  const xIsNext = currentMove % 2 === 0;
  const currentSquares = history[currentMove];

  function handlePlay(nextSquares : Array<string | null>) {
    const nextHistory = [...history.slice(0, currentMove + 1), nextSquares];
    setHistory(nextHistory);
    setCurrentMove(nextHistory.length - 1);
  }

  function jumpTo(nextMove : number) {
    setCurrentMove(nextMove);
  }

  const moves = history.map((squares, move) => {
    let description;
    if (move > 0) {
      description = 'Go to move #' + move;
    } else {
      description = 'Go to game start';
    }
    return (
      <li key={move}>
        <button onClick={() => jumpTo(move)}>{description}</button>
      </li>
    );
  });

  return (
    <div className="game">
      <div className="game-board">
        <Board xIsNext={xIsNext} squares={currentSquares} onPlay={handlePlay} />
      </div>
      <div className="game-info">
        <ol>{moves}</ol>
      </div>
    </div>
  );
}

function calculateWinner(squares: Array<string | null>) {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6],
  ];
  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a];
    }
  }
  return null;
}
```