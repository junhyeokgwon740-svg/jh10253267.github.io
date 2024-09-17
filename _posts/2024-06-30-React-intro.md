---
layout: post
title:  "React intro"
author: 악어새62
categories: [ TIL, WEB, Frontend ]
excerpt_image: https://github.com/user-attachments/assets/4cd0ff2e-6ff3-4d28-9804-76abe19b05fc
image: assets/images/4.jpg
tags: [ Web, Frontend, React ]
---

![banner](https://github.com/user-attachments/assets/4cd0ff2e-6ff3-4d28-9804-76abe19b05fc)

## 개요

이전에 살펴보았던 새로운 방식의 프론트 작업방식.
이런 방식을 지원하는 라이브러리가 바로 React이다.
화면을 컴포넌트 단위로 설계하고 렌더링한다.

리엑트에 대해 공부해보고 간단한 애플리케이션을 만들어보자.  
리엑트 공식 사이트에서 틱택토 앱 튜토리얼을 제공하고있다.  

## 설치

과거에는 웹팩과 바벨을 이용했었다. 웹팩이란 오픈 소스 자바스크립트 모듈 번들러로 여러개의 파일로 나누어져있는 자바스크립트 코드를 압축하고 최적화하는 라이브러리다.  
바벨은 구형 브라우저에서는 지원하지 않는 자바스크립트의 최신 문법도 사용할 수 있게 변환시켜주는 라이브러리를 말한다.  

예전에는 이 둘을 이용해서 리엑트를 설치했는데 요즘은 Create-React-App을 이용해서 설치하는 것이 일반적이다.

한 번 설치해보자
1. 리엑트 앱을 작업할 폴더를 생성한다.
2. 터미널에서 `npx create-react-app ./` 명렁어를 실행한다.

일반적으로 `node.js`를 사용할 때 `npm`이라는 명령어를 사용하는데 여기서는 `npx`명령어를 사용했다. `npx`는 노드 패키지의 실행을 도와주는 도구로 `npm` 레지스트리에 있는 패키지를 해당 위치에 실행하여 리엑트를 설치하는 것이다.

## 리엑트 패키지 구조

<img width="192" alt="스크린샷 2024-06-30 시간: 03 23 18" src="https://github.com/jh10253267/TIL/assets/108499717/baaa9398-5979-4aaa-b5ac-3f9df453dfbf">
`src` 폴더는 우리가 개발하는 파일들이 있는 부분이다.  
`.gitignore`은 소스코드 저장소에 올라가면 안되는 파일들을 등록하는 부분이고 `package.json`파일은 의존성을 관리하고 프로젝트의 메타데이터를 관리하는 파일이다.   

public은 정적파일들을 관리하는 폴더이다.

## 실행해보기

위에서 살펴본 `package.json`의 `scripts`부분을 보면 리엑트 앱을 실행시키는 명령어가 있다.

`npm run start` 혹은 `npm start`명렁어를 입력하여 실행시켜보자.

<img width="407" alt="스크린샷 2024-06-30 시간: 03 32 58" src="https://github.com/jh10253267/TIL/assets/108499717/eda57c18-bf63-4b25-bcd4-2ebef24b22e3">
<img width="642" alt="스크린샷 2024-06-30 시간: 03 32 44" src="https://github.com/jh10253267/TIL/assets/108499717/c252573e-422c-44f3-b219-ad96b6e56d4c">

여기서 출력된 텍스트나 이미지는 src폴더의 App.js에 작성되어있다.

이 부분을 한 번 바꿔보자.
<img width="475" alt="스크린샷 2024-06-30 시간: 03 36 10" src="https://github.com/jh10253267/TIL/assets/108499717/e428d3d1-4c2e-450e-83f5-3c503f4f24aa">
여기에서 가장 상위의 `<div className="App">`태그 안의 내용을 다 지우고 아무 텍스트나 입력하고 화면을 확인해보면 내가 작성한 텍스트가 잘 나오는 것을 알 수 있다.

## 틱텍토 앱 만들기

리엑트 공식사이트를 보면 튜토리얼을 안내해주고있다.  
틱택토앱은 두명의 참가자가 9개의 칸에서 한 턴에 한 칸씩 선택하고 연달아 3개의 칸을 선택하면 이기는 게임으로 미니 오목같은 느낌이다.

## 컴포넌트 식별

틱택토앱은 크게 게임, 게임 정보의 컴포넌트로 이루어져있고 게임 컴포넌트는 게임 판에 해당하는 Board컴포넌트와 그 안의 9개의 칸에 해당하는 Square컴포넌트로 이루어져있다.

리엑트의 시작은 위에서 살펴봤듯이 App.js부터 시작한다.
```js
function App() {
  render() {
    return (
      <div className="game">
        <div className="game-board">
          game-board
        </div>
        <div className="game-info">
          game-info
        </div>
      </div>
    );
  }
}
```
그런 다음 컴포넌트를 생성해보자. 보통 컴포넌트들을 모아서 src하위 디렉토리로 components디렉토리를 만들어 관리한다.

extends키워드를 사용하여 Compnent클래스를 상속받는다. 이건 리엑트에서 제공하는 Component의 기능들을 사용하되 디테일한 부분은 만들고자하는 컴포넌트에 맞추어 커스텀할 수 있는 것이다.

```js
import React, { Component } from 'react'

export default class Board extends Component {
  renderSquare(i) {
    return <Square value={i} />;
  }
  render() {
    const status = 'Next player: x';

    return (
      <div>
        <div className="status">{status}</div>
        <div className="board-row">
          {this.renderSquare(0)}
          {this.renderSquare(1)}
          {this.renderSquare(2)}
        </div>
        <div className="board-row">
          {this.renderSquare(3)}
          {this.renderSquare(4)}
          {this.renderSquare(5)}
        </div>
        <div className="board-row">
          {this.renderSquare(6)}
          {this.renderSquare(7)}
          {this.renderSquare(8)}
        </div>
      </div>
    )
  }
}
```
```js
export default class Square extends Component {
  render() {
    return (
      <button className="square">
        {this.props.value}
      </button>
    );
  }
}
```
코드를 보면 자바스크립트에서 html코드를 작성하는데 문자열로 작성하거나 하는 것이 아니라 바로 html코드를 작성하고있다.
이건 jsx라고 하며 바벨의 확장 기능으로 저렇게 작성하고 나면 바벨이 이를 리엑트 문법으로 변환시켜준다.
```js
React.createElement('태그 이름', );
```
Squre컴포넌트는 안에서 태그를 사용하지만 Board의 경우 `<Square />`라는 태그를 사용하고있다. 직접 태그를 리턴할수도 있고 컴포넌트를 태그처럼 통채로 사용할 수도 있다.

this.renderSquare(1)과 같이 부모 컴포넌트에서 자식 컴포넌트에게 값을 전달하고있다. 이를 props라고 한다. 부모 컴포넌트가 자식 컴포넌트에서 이 값을 value라는 이름으로 내려주고 있고 자식 컴포넌트에서 사용하는 방법은 `this.props.설정한 이름`형식이다.

다음은 네모 칸을 클릭했을 때 체크를 하는 부분이다.  
Square컴포넌트의 button의 속성으로 사용할 수 있는 onClick을 이용해보자.  
나는 자바스크립트 로직을 테스트할 때 먼저 작동하는지 부터 테스트하곤한다.
```js
<button className="square" 
  onClick={() => { console.log('clicked!!'); }}>
</button>
```
이제 state를 사용할 차례다. state는 그 이름 그대로 컴포넌트의 현재 상태 값을 기억하는 존재다.  
`this.setState`문법을 사용하면 된다.
```js
constructor(props) {
  super(props);
  this.state = {
    value: null,
  };
}
...
<button className="square" 
  onClick={() => { this.setState({value: 'X'}) }}>
  {this.state.value}
</button>
```

이렇게 하면 클릭했을 때 X가 표시된다.  
`super()`라는 부분이 보이는데 이는 자식 클래스 내에서 부모 클래스의 생성자를 호출할 때 사용되는 문법이다. 이렇게 해서 자식 클래스 내에서 부모 클래스의 메소드를 호출할 수 있다.

그러나 한 명이 X를 표시하면 다음 사람은 O를 표시해야하는데 상태 값을 각각의 칸이 가지고있으니 이를 구현할 수가 없어진다. 따라서 상태 값을 상위 요소인 Board에서 관리하도록 수정한다.  
상위 요소인 Board에서 Square에 상태 값을 내려주는 것이다. 부모 요소에서 자식 요소에게 값을 전달해주기 위한 개념은? -> props!

Board 컴포넌트의 생성자에서 `squares: Array(9).fill(null)` 코드를 작성해준다.

```js
export default class Board extends Component {
  constructor(props) {
    super(props);
    this.state = {
      squares: Array(9).fill(null)
    }
  }
}
```
```js
squares: [
 null, null, null,
 'X', 'O', null,
 'O'. 'X', null 
]
```
이 것은 위와 같이 게임의 상태를 관리하려는 목적이다.  
기존 스퀘어에 작성했던 아래의 코드는 필요가 없으니 삭제해준다.
```js
constructor(props) {
  super(props);
  this.state = {
    value: null,
  };
}
```
그리고 이 값을 스퀘어에 내려준다.
```js
renderSquare(i) {
  return <Square value={this.state.squares[i]} />
}
```
state를 상위 요소에서 관리하고 하위 요소에 props로 내려준다.
```js
<button className="square" 
  onClick={() => { this.props.value }}>
</button>
```

클릭했을 경우 상태를 변경시키기 위해 별도의 함수를 작성해준다.
```js
handelClick(i) {
  const squares = this.state.squares.slice();
  squares[i] = 'X';
  this.setState({ squares: squares });
}
renderSquare(i) {
    return (
      <Square
        value={this.state.squares[i]}
        onClick={() => this.handleClick(i)}
      />
    );
}
```
```js
export default class Square extends Component {
  render() {
    return {
      <button className="square" 
        onClick={() => { this.props.onClick() }}>
        {this.props.value}
      </button>
    }
  }
}
```
`this.state.squares.slice()`는 어떤 배열의 시작부터 끝까지에 대한 얕은 복사본을 새로운 배열 객체로 반환하는 함수로 원본 배열은 바뀌지 않는다.  
즉, 위의 코드는 원본 `squares`는 변경하지 않고 새로운 배열을 만들어서 그 배열을 변경하는 것이다.

이렇게까지 하면 내장된 DOM 버튼 컴포넌트에 있는 onClick prop은 리엑트에게 클릭 이벤트 리스너를 설정하라고 알려준다. 버튼을 클릭하면 리엑트는 Square의 render함수에 정의된 onClick이벤트 핸들러를 호출한다. 이벤트 핸들러는 this.props.onClick()을 호출하고 Board에서 Square로 onClick={() => this.handleClick(i)} props를 전달했기 때문에 Square를 클릭하면 Board의 handleClick을 호출한다.

기존 배열을 변경하지 않고 새로운 배열을 복사해서 사용하는 이유는 불변성 때문이다.  
참조 타입에서 객체나 배열의 값이 변할 때 원본 데이터가 변경되기 때문에 원본 데이터를 참조하고 있는 다른 객체에서 예상치 못한 오류가 발생할 수 있고 리엑트에서 화면을 업데이트할 때 불변성을 지키면 값을 이전 값과 비교하여 변경된 사항을 확인한 후 업데이트 하기 때문이다.

배열 데이터의 경우 참조타입으로 값을 변경했을 때 힙 메모리 값이 변경되기 때문에 아예 새로운 배열을 리턴하도록 함으로써 불변성을 지킬 수 있다.

## 함수형 컴포넌트로 바꾸기

위와 같이 작성하는 방법을 클래스형 컴포넌트라고 한다.  
함수형 컴포넌트는 말 그대로 클래스가 아닌 함수로 컴포넌트를 작성한다. 실제론 함수형 컴포넌트로 작성하는 경우가 많다고 한다.  
둘의 장단점을 비교해보면...  

클래스형:
* 많은 기능 제공
* 더 길고 복잡한 코드
* 더딘 성능

함수형:
* 적은 기능 제공
* 짧고 심플한 코드
* 빠른 성능

리엑트에는 라이프사이클이 있다.  
![image](https://github.com/user-attachments/assets/10690b2c-27d7-42d6-ad22-85351d63265c)
리엑트는 DOM애서 특정한 시간에 특정한 행동을 일어나게 할 수 있도록 생명주기를 만들었다.

크게 3단계로 나눌 수 있는데 왼쪽에서 오른쪽 순서로 일어난다.  
`Mounting` -> `Updating` -> `Unmounting`

컴포넌트가 처음 실행이 될 때 생성 단계를 `Mounting`라고 한다.  
* `constructor` : 컴포넌트가 처음 생성될 때 호출
* `render` : JSX를 반환하여 브라우저에서 어떤 HTML을 렌더링할지 결정
* `componentDidMount` : 컴포넌트가 화면에 완전히 그려진 뒤에 호출되고 이 시점에 데이터를 불러오거나 DOM 요소에 접근할 수 있다.

컴포넌트가 업데이트될 때의 단계를 `Updating`이라고 한다.
* `render` : 상태나 props가 변경되면 다시 호출되어 컴포넌트를 재렌더링한다.
* `componentDidUpdate` : 컴포넌트가 업데이트된 후에 호출되고 이전 props나 상태에 접근하여 변경 사항을 처리할 수 있다.

컴포넌트가 더 이상 필요 없을 때 화면에서 제거되는 단계를 `Unmounting`이라고 한다.
* `componentWillUnmount`

이런 라이프사이클을 함수형 컴포넌트에서는 사용하지 못했다.  
그렇기 때문에 클래스형 컴포넌트를 사용했지만 리엑트 16.8에서 `Hooks`업데이트로 함수형 컴포넌트에서도 생명주기를 사용할 수 있게 되었다. 각 방식의 장점만 취해서 사용할 수 있는 것이다.

그래서 기존에 작성했던 클래스형 컴포넌트를 함수형 컴포넌트로 바꿔보자.

class로 선언되어있는 키워드를 지우고 다음과 같이 바꿔준다.
```js
const Board = () => {
  return() {
    
  }
}
```
함수형 컴포넌트에서는 render()메소드를 사용하지 않고 바로 return키워드를 사용한다.

그런 다음 state를 Hooks를 사용해서 다음과 같이 바꿔준다.
```js
const [squares, setSquares ] = userState(Array(9).fill(null));
```
useState는 첫번째 인수로 변수의 이름을 적고 두 번째 인수로 state를 정하는 함수를 적는다.  
각각 getter, setter에 대응되는 개념이다.

```js
const Board = () => {
  const [ squares, setSquares ] = userState(Array(9).fill(null));

  const handleClick = (i) => {
    const newSquares = squares.slice();
    newSquares[i]  = 'X';
    setSquares(newSquares);
  }

  const renderSquare = (i) => {
    return <Square value={squares[i]}
      onclick={() => handelClick(i)} />
  }

  return {
    <div className="board-row">
      {renderSquare(0)}
      ...
    </div>
  }
}
export default Board
```
스퀘어 컴포넌트도 마찬가지로 수정해준다.

```js
const Square = ({ onClick, value}) => {
  return (
    <button className="square"
      onClick={onClick}>
      {value}

    </button>
  )
}
export default Square
```

## 게임규칙 적용하기

지금까지의 코드는 x만 표시가 된다. 이 것을 다음 턴에는 O표시로 바꿔보자.  
이를 위해 xIsNext라는 불리언 변수를 만든다.
```js
const Board = () => {
  const [ squares, setSquares ] = useState(Array(9).fill(null));
  const [ xIsNext, setXIsNext ] = useState(true);
  const status = `Next Player: ${xIsNext ? 'X' : 'O'}`

  handelClick(i) {
    const newSquares = this.state.squares.slice();
    newSquares[i] = xIsNext ? 'X' : 'O';
    setSquares(newSquares);
    setXIsNext(current => !current);
  }

  return (
    <div>
      <div className="status">{status}</div>
      <div className="board-row">
        {this.renderSquare(0)}
        {this.renderSquare(1)}
        {this.renderSquare(2)}
      </div>
      <div className="board-row">
        {this.renderSquare(3)}
        {this.renderSquare(4)}
        {this.renderSquare(5)}
      </div>
      <div className="board-row">
        {this.renderSquare(6)}
        {this.renderSquare(7)}
        {this.renderSquare(8)}
      </div>
    </div>
  );
}
```

이제 게임의 승자를 결정하는 부분을 구현해보자.
예제에서는 이기는 경우의수를 리스트로 관리하고 여기에 해당하는 경우 승자를 정하고 게임을 종료시킨다.

```js
const calculateWinner = (squares) => {
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
```js
const winner = calculateWinner(squares);
let status;
if(winner) {
  status = `Winner: ${winner}`
} else {
  status = `Next player: ${xIsNext ? 'X' : 'O'}`
}
```
만약 승자가 정해졌다면 게임을 종료한다.  
handleClick 함수에 다음의 내용을 추가해준다.
```js
if(calculateWinner(newSquares) || newSquares[i]) {
  return;
}
```

**전체코드**
```js
import React, { useState } from 'react';
import Square from './Square';

const Board = () => {
  const [squares, setSquares] = useState(Array(9).fill(null));
  const [xIsNext, setXIsNext] = useState(true);

  const calculateWinner = (squares) => {
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
  const winner = calculateWinner(squares);
  let status;
  if(winner) {
    status = `Winner: ${winner}`;
  } else {
    status = 'Next player: ' + (xIsNext ? 'X' : 'O');
  }

  const handleClick = (i) => {
    const newSquares = squares.slice();

    if(calculateWinner(newSquares) || newSquares[i]) {
      return;
    }
    
    newSquares[i] = xIsNext ? 'X' : 'O';
    setSquares(newSquares);
    setXIsNext(prev => !prev);
  };

  const renderSquare = (i) => {
    return (
      <Square 
        value={squares[i]} 
        onClick={() => handleClick(i)} 
      />
    );
  };

  return (
    <div>
      <div className='status'>{status}</div>
      <div className='board-row'>
        {renderSquare(0)}
        {renderSquare(1)}
        {renderSquare(2)}
      </div>
      <div className='board-row'>
        {renderSquare(3)}
        {renderSquare(4)}
        {renderSquare(5)}
      </div>
      <div className='board-row'>
        {renderSquare(6)}
        {renderSquare(7)}
        {renderSquare(8)}
      </div>
    </div>
  );
};

export default Board;
```
```js
import React, { Component } from 'react'

const Square = ({ onClick, value }) => {
    return (
      <button className='square' 
        onClick={onClick}>
        {value}
      </button>
    )
}

export default Square;
```

## 시간 여행 추가하기

이전에 리엑트에 대해서 설명할 때 SPA 방식에서 화면이 바뀐 것처럼 하기 위한 것이 window.history api라고 한 적이 있다. 이 부분을 실습해보자. 바둑이나 오목에서 N 수를 어디에 두었는지 볼 수 있는 기능을 구현하며 상태를 위로 끌어올려 개선하는 경험도 하니 일석이조다.

이전 상태를 기억하고 렌더링하려면 state를 Board보다 상위 요소인 App에 상태를 옮겨서 관리해야한다.
만약 원본 게임 배열을 직접 변경했다면 시간에 따라 어떻게 게임판이 채워졌는지 알기 힘들었을 것이다. 그러나 원본 배열의 새로운 복사본을 만들어서 사용했고 불변성을 지켰기 때문에 모든 버전을 기억하고 추적할 수 있게 된 것이다. 

각 턴의 상태를 기록하기 위해서 배열 데이터를 만들어준다.
```js
const App = () => {
  const [ history, setHistory ] = useState([
    {squares: Array(9).fill(null)}
  ]);
}
```
그리고 Board의 상태를 다른 컴포넌트에서도 알아야하기 때문에 Board컴포넌트에서 작성한 코드도 App에 옮겨준다.
```js
const calculateWinner = (squares) => {
    const lines = [
      [0, 1, 2],
      [3, 4, 5],
      [6, 7, 8],
      [0, 3, 6],
      [1, 4, 7],
      [2, 5, 8],
      [0, 4, 8],
      [2, 4, 6]
    ]
    for (let index = 0; index < lines.length; index++) {
      const [a, b, c] = lines[index];
      if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
        return squares[a];
      }
    }
    return null;
  }

  const current = history[stepNumber];
  const winner = calculateWinner(current.squares);

  let status;
  if (winner) {
    status = 'Winner: ' + winner;
  } else {
    status = `Next player: ${xIsNext ? 'X' : 'O'}`;
  }

  const handleClick = (i) => {
    const newHistory = history.slice(0, stepNumber + 1);
    const newCurrent = newHistory[newHistory.length - 1];
    const newSquares = newCurrent.squares.slice();
    if (calculateWinner(newSquares) || newSquares[i]) {
      return;
    }

    newSquares[i] = xIsNext ? 'X' : 'O';
    // 전개연산자
    setHistory([...newHistory, { squares: newSquares }]);
    setXIsNext(prev => !prev);

    setStepNumber(newHistory.length);
  }
```
이렇게 상태를 기록하였고 이제 N수를 클릭하면 그 상태로 이동하는 부분을 구현할 차례다.

알아둬야하는 메소드는 map 메소드로 배열 내의 요소들에 대해 일괄적인 처리를 하여 새로운 배열을 반환하는 메소드다.  
예를 들어 배열의 모든 요소에 +10을 한 결과를 새로운 배열로 만들고싶다면 map메소드를 사용하면 된다.

```js
function App() {
  const [history, setHistory] = useState([{ squares: Array(9).fill(null) }]);
  const [xIsNext, setXIsNext] = useState(true);
  const [stepNumber, setStepNumber] = useState(0);

  const calculateWinner = (squares) => {
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

  const current = history[history.length - 1];
  const winner = calculateWinner(current.squares);

  let status;
  if (winner) {
    status = `Winner: ${winner}`;
  } else {
    status = 'Next player: ' + (xIsNext ? 'X' : 'O');
  }

  const handleClick = (i) => {
    const newHistory = history.slice(0, stepNumber + 1);
    const newCurrent = newHistory[newHistory.length - 1];
    const newSquares = newCurrent.squares.slice();
    if (calculateWinner(current, newSquares) || newSquares[i]) {
      return;
    }

    newSquares[i] = xIsNext ? 'X' : 'O';
    setHistory([...newHistory, { squares: newSquares }]);
    setXIsNext(current => !current);

    setStepNumber(newHistory.length)
  }

  const moves = history.map((step, move) => {
    const desc = move ?
      `Go to move #${move}` :
      `Go to game start`;
    return (
      <li key={move}>
        <button className='move-btn' onClick={() => jumpTo(move)}>
          {desc}
        </button>
      </li>
    );
  })

  const jumpTo = (step) => {
    setHistory(history.slice(0, step + 1));
    setXIsNext(step % 2 === 0);
  }

  return (
    <div className='game'>
      <div className='game-board'>
        <Board
          squares={current.squares}
          onClick={(i) => handleClick(i)}
        />
      </div>
      <div className='game-info'>
        <div className='status'>{status}</div>
        <ol>{moves}</ol>
      </div>
    </div>
  );
}

export default App;
```
