---
layout: post
title:  "Javascript object"
author: 악어새62
categories: [TIL, WEB, Frontend ]
tags: [ Javascript, Web, Frontend]
image: assets/images/5.jpg
---
## 개요

자바스크립트의 객체는 키와 값으로 이루어진 딕셔너리 자료구조이다.  
그 사용법이 간단하고 객체안에 객체가 들어갈 수 있고 객체안에 함수도 들어갈 수 있어서 굉장히 유용하게 사용할 수 있다.

## 사용법

### 선언과 접근

객체는 다음과 같이 선언하고 사용할 수 있다.
```javascript
let obj = {name : "user1", age: "20"};

console.log(obj["name"]);
console.log(obj.age);
```
자바스크립트의 객체구조를 본따 서버와 웹 브라우저간에 데이터를 주고받을 때 정의한 JSON이라는 것이 있다.

### 객체의 추가/삭제

```javascript
let obj = {name : "user1", age: "20"};

//추가
obj["key1"] = "test";
//삭제
delete obj.name;
delete obj["key1"];
```

### 객체의 탐색
```javascript
let obj = {name : "user1", age: "20"};

// 방법 1
for(value in obj) {
  console.log(obj[value]);
}
// 방법2
Object.keys(obj).forEach(function(key) {
  console.log(obj[key]);
});
```

## 객체 리터럴

```javascript
let healthObj = {
  name : "달리기",
  lastTime : "PM10:12",
  showHealth : function() {
    console.log(this.name + "님, 오늘은 " + this.lastTime + "에 운동을 하셨네요");
  }
}

healthObj.showHealth();
```
이와 같은 형태를 객체 리터럴이라 부른다.  
그러나 이러한 형태의 객체가 여러개 필요하다면?  
healthObj1,healthObj2 .....  
이런 식으로 형태는 같고 디테일한 내용만 다른 객체들을 여러번 선언해줘야할 것이다.  
개발자들이 끔찍히도 싫어하는 코드가 중복되는 문제가 생긴다.  
이런 문제를 해결하는 방법은 다른 객체 지향 프로그래밍언어처럼 new 키워드를 이용해 객체를 동적으로 생성하는 방법이다.

```javascript
function Health(name, lastTime) {
  this.name = name;
  this.lastTime = lastTime;
  this.showHealth = function(){...}
}
const h = new Health("달리기", "10:12");

h2 = new Health("걷기", "20:11"); 
```
이런 식으로 사용할 수 있다.  
그러나 아직 내부 함수가 새롭게 정의되며 중복되고 있다.
이를 해결할 수 있는 방법이 바로  
> Prototype

## Prototype

모든 자바스크립트 객체는 한 곳을 바라본다. 바로 prototype이라는 공간이다.  
위에서 살펴본 것 같이 새로운 객체를 new 키워드를 이용해 생성하면 생성된 모든 객체는 프로토 타입을 바라본다.

![프로토타입](https://github.com/jh10253267/TIL/assets/108499717/1e19a9c4-92ee-4704-9d2f-af5c3f430242)
다르게 말해 프로토 타입에 있는 정보를 같은 타입의 객체라면 가져다 쓸 수 있다는 의미이다.

우리는 이 공간에 해당 객체에 필요한 정보, 혹은 함수들을 넣어서 객체에서 사용할 수 있다는 것이다.

마치 상속의 느낌이다.  
여기서 부터 마치 다른 프로그래밍 언어를 사용하는 것처럼 느껴지기도 한다.
```javascript
function Health(name, lastTime) {
  this.name = name;
  this.lastTime = lastTime;
}

Health.prototype.showHealth = function() {
    console.log(this.name + "," + this.lastTime);
}

const h = new Health("달리기", "10:12");
console.log(h);  
h.showHealth();
```
이전에 객체를 매번 새롭게 정의했다. -> 코드가 중복되는 문제가 생김  
그 다음엔 객체를 동적으로 생성했다. -> 같은 함수가 새롭게 정의되는 문제가 생김

new 키워드로 만들어진 객체들의 공통 기능을 프로토 타입 공간에 선언해서 중복되는 코드를 줄일 수 있게 되었다. -> 문제 해결

이런식으로 함수들을 설계한다면 마치 자바 프로그램을 작성하는 것과 같이 프로그램을 작성할 수 있다.

최근 자바스크립트에서도 클래스의 개념이 도입되었는데 프로토 타입을 좀더 정돈한 것이라. 새로운 패러다임이라고 보긴 어렵다. 그래도 한 번 배워보자 

## Class

### 기본 문법
프로토 타입을 사용할 때
```js
function DoSomething() {

}

DoSomething.prototype.start() = function() {
  console.log("start!!");
}
const doSomething = new DoSomething();
doSomething.start(); // "start!!"
```
클래스를 사용할 때
```js
class DoSomething {
  constructor() {

  }
  start() {
    console.log("start!!");
  }
}
const doSomething = new DoSomething();
doSomething.start(); // "start!!"
```
클래스로 작성했을 때 start라는 함수를 정의한 것은 아래의 코드와 대응된다.
```js
DoSomething.prototype.start() = function() {
  console.log("start!!");
}
```
클래스로 작성한 코드에서 `console.log(doSomething);`를 확인해보면 필드에는 start라는 메소드가 보이지 않고 아래의 `[[protptype]]`토글을 열어보면 그 곳에 우리가 정의한 메소드가 있는 것을 확인할 수 있다.

### Getter/Setter

Getter란 클래스에서 값을 얻어오기 위한 함수이고 Setter란 클래스 내부에 값을 설정하기 위한 방법이다.
```js
class DoSomething {
  constructor(first, last) {
    this.firstName = first;
    this.lastName = last;
    this.fullName = `${first} ${last}`;
  }
  start() {
    console.log("start!!");
  }
}

const doSomething = new DoSomething('first', 'last');
console.log(doSomething);
doSomething.firstName = 'last';
console.log(doSomething);
```
이렇게 작성하고 출력 값을 보면 firstName속성의 값을 변경했음에도 적용이 되지 않는 모습을 볼 수 있다.

이는 생성자 함수로 클래스를 호출했을 때 최초로 설정되도록 했기 때문이다.  
따라서 동적으로 값이 변경되지 않는다.  
코드를 작성하며 내부의 값을 가져오거나 변경하고 싶은 경우가 있을 것이다. 이럴 때 사용하는 것이 바로 Getter/Setter 메소드다.
```js
class DoSomething {
  constructor(first, last) {
    this.firstName = first;
    this.lastName = last;
    this.fullName = `${first} ${last}`;
  }
  getFullName() {
    return `${this.firstName} ${this.lastName}`
  }
  start() {
    console.log("start!!");
  }
}
```

이렇게 작성할 수 있다. 그러나 Getter/Setter는 꽤 자주쓰이기 때문에 이를 위한 문법이 존재한다.

```js
class DoSomething {
  constructor(first, last) {
    this.firstName = first;
    this.lastName = last;
    this.fullName = `${first} ${last}`;
  }
  get fullName() {
    return `${this.firstName} ${this.lastName}`
  }
  start() {
    console.log("start!!");
  }
}

const doSomething = new DoSomething('first', 'last');
console.log(doSomething.fullName);
```
마치 객체의 속성에 접근하듯이 사용하면 get 키워드가 붙은 함수가 호출된다.  
이와 마찬가지로 set 키워드를 붙이면 Setter를 작성할 수 있다.
```js
class DoSomething {
  constructor(first, last) {
    this.firstName = first;
    this.lastName = last;
    this.fullName = `${first} ${last}`;
  }
  get fullName() {
    return `${this.firstName} ${this.lastName}`
  }
  set fullName(value) {
    this.fullName = value;
  }
  start() {
    console.log("start!!");
  }
}

const doSomething = new DoSomething('first', 'last');
doSomething.fullName = 'test';
console.log(doSomething);
```
### 정적 메소드

mdn 사이트에서 함수에 대한 설명이되어있다. 그중에서는 prototype이 붙은 메소드도 있지만 바로 접근해서 사용하는 메소드도 있다. 이 둘의 차이점은 특정한 데이터를 가지고 동적으로 작동하는 기능이라면 prototype으로 작성되어있고 그게 아니라 isArray()와 같이 외부에서 데이터를 가지고 판별을 한다던지 하는 기능이라면 바로 접근해서 사용할 수 있게 되어있다.

위에서 본 DoSomething을 예로 들어보면 `doSomething(이건 인스턴스).start();`와 같이 실행할 수 없는 일반 메소드에 해당하고 `DoSomething.start();`와 같이 인스턴스 없이 호출할 수 있는 메소드는 정적 메소드에 해당한다.

클래스에서 정적 메소드는 단순히 `static` 키워드를 붙여줌으로써 정의하고 사용할 수 있다.
```js
class User {
  constructor(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }
  getFullName() {
    return `${this.firstName} ${this.lastName}`;
  }
  static isUser(user) {
    if(user.firstName && user.lastName) {
      return true;
    } else {
      return false;
    }
  }
}

const son = {
  name: 'son',
  age: 31
}

console.log(User.isUser(son)); // false
```

### 상속

이 부분은 다른 객체지향 언어들과 크게 다르지 않다.  
많은 종류의 비슷한 클래스를 반복해서 사용해야할 경우 하나하나 직접 정의하는 것은 비효율적이다.
```js
class Vehicle {
  constructor(acceleration = 1) {
    this.spped = 0
    this.acceleration = acceleration
    this.isStarted = false
  }
  start() {
    console.log("시동을 겁니다.")
    this.isStarted = true
  }
  moveForward() {
    if (this.isStarted) {
      this.speed += this.acceleration
    } else {
      console.error("시동이 걸려 있지 않습니다.")
    }
  }
  moveBackward() {
    if (this.isStarted) {
      this.speed -= this.acceleration
    } else {
      console.error("시동이 걸려 있지 않습니다.")
    }
  }
}
```
```js
class Car extends vehicle {
  constructor(acceleration, type) {
    super(acceleration);
    this.type = type;
  }
  printInfo() {
    console.log(this.type);  
  }
}

const car = new Car(1, "suv");
console.log(car instanceof Vehicle) // true
```