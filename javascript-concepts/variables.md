# JavaScript 변수 정리

## 1. 변수란?

변수는 값을 저장하는 이름이다.

JavaScript에서는 변수를 만들 때 보통 `let`, `const`, `var`를 사용한다.

```javascript
let age = 20;
const name = "kim";
var city = "Seoul";
```

요즘 JavaScript에서는 보통 `var`보다 `let`과 `const`를 사용한다.

## 2. var

`var`는 예전에 많이 사용하던 변수 선언 방식이다.

```javascript
var name = "kim";
```

`var`는 값 변경이 가능하다.

```javascript
var name = "kim";
name = "lee";

console.log(name); // lee
```

또한 `var`는 같은 이름으로 다시 선언할 수도 있다.

```javascript
var age = 20;
var age = 30;

console.log(age); // 30
```

하지만 이런 특징 때문에 실수해도 에러가 나지 않아 문제가 될 수 있다.

## 3. let

`let`은 값이 바뀔 수 있는 변수를 만들 때 사용한다.

```javascript
let age = 20;
age = 21;

console.log(age); // 21
```

`let`은 재할당은 가능하지만, 같은 스코프에서 재선언은 불가능하다.

```javascript
let name = "kim";
let name = "lee"; // 에러
```

## 4. const

`const`는 재할당할 수 없는 변수를 만들 때 사용한다.

```javascript
const name = "kim";
name = "lee"; // 에러
```

요즘 JavaScript에서는 기본적으로 `const`를 먼저 사용하고, 값이 바뀌어야 할 때만 `let`을 사용한다.

## 5. var, let, const 차이

```text
var
→ 재할당 가능
→ 재선언 가능
→ 함수 스코프
→ 요즘은 지양

let
→ 재할당 가능
→ 재선언 불가
→ 블록 스코프
→ 값이 바뀔 때 사용

const
→ 재할당 불가
→ 재선언 불가
→ 블록 스코프
→ 기본적으로 가장 많이 사용
```

## 6. 스코프 차이

`var`는 함수 스코프를 가진다.

```javascript
if (true) {
  var name = "kim";
}

console.log(name); // kim
```

`if` 블록 안에서 만든 변수인데 바깥에서도 접근된다.

반면 `let`과 `const`는 블록 스코프를 가진다.

```javascript
if (true) {
  let name = "kim";
}

console.log(name); // 에러
```

블록 밖에서는 접근할 수 없기 때문에 더 안전하다.

## 7. 사용 기준

```text
기본은 const
값이 바뀌면 let
var는 지양
```

예시:

```javascript
const userName = "kim";
let count = 0;

count++;
```

`userName`은 바뀌지 않으므로 `const`, `count`는 증가하므로 `let`을 사용한다.

## 8. 한 번에 외우는 답변

> JavaScript에서 변수 선언은 `var`, `let`, `const`로 할 수 있습니다. `var`는 재할당과 재선언이 모두 가능하고 함수 스코프를 가지기 때문에 실수하기 쉬워 요즘은 잘 사용하지 않습니다. `let`은 재할당이 가능하지만 재선언은 불가능하고, `const`는 재할당과 재선언이 모두 불가능합니다. 보통 기본은 `const`를 사용하고, 값이 바뀌어야 할 때만 `let`을 사용합니다.
