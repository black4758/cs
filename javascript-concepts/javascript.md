# JavaScript 정리

## 1. JavaScript란?

JavaScript는 웹 브라우저에서 동작하는 프로그래밍 언어이다.

HTML이 화면의 구조를 담당하고, CSS가 디자인을 담당한다면, JavaScript는 화면의 동작을 담당한다.

예시:

```text
버튼 클릭하면 메뉴 열기
입력값 검사하기
서버 API 호출하기
화면 데이터 바꾸기
로그인 토큰 저장하기
```

Vue도 JavaScript 위에서 동작하는 프론트엔드 프레임워크이다. 그래서 Vue를 공부하려면 JavaScript 기본 개념을 먼저 알아야 한다.

면접 답변:

> JavaScript는 웹 브라우저에서 동작하는 프로그래밍 언어로, 웹 페이지의 동적인 동작을 처리합니다. HTML은 구조, CSS는 스타일, JavaScript는 이벤트 처리, 데이터 변경, API 호출 같은 동작을 담당합니다. Vue나 React 같은 프론트엔드 프레임워크도 JavaScript를 기반으로 동작합니다.

## 2. Vue 전에 알아야 할 JavaScript 필수 개념

Vue를 공부하기 전에 최소한 다음 개념은 알아야 한다.

```text
변수: let, const, var
함수: function, arrow function
객체
배열
배열 메서드: map, filter, find
조건문
삼항 연산자
구조 분해 할당
import / export
비동기 처리: Promise, async/await
API 호출: fetch, axios
JSON
DOM 기본
```

## 3. 변수

JavaScript에서는 변수를 만들 때 `let`, `const`, `var`를 사용한다.

```javascript
let age = 20;
const name = "kim";
var city = "Seoul";
```

요즘은 보통 `var`보다 `let`과 `const`를 사용한다.

정리:

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

사용 기준:

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

## 4. 함수

함수는 특정 동작을 묶어두고 필요할 때 호출하는 코드이다.

기본 함수:

```javascript
function add(a, b) {
  return a + b;
}

console.log(add(1, 2)); // 3
```

화살표 함수:

```javascript
const add = (a, b) => {
  return a + b;
};
```

짧게 쓸 수도 있다.

```javascript
const add = (a, b) => a + b;
```

Vue나 React에서는 화살표 함수를 많이 사용한다.

## 5. 객체

객체는 여러 값을 key-value 형태로 묶은 데이터이다.

```javascript
const user = {
  id: 1,
  name: "kim",
  email: "kim@test.com"
};

console.log(user.name); // kim
```

Spring 백엔드에서 JSON으로 응답한 데이터는 JavaScript에서 객체처럼 다룰 수 있다.

```json
{
  "id": 1,
  "name": "kim",
  "email": "kim@test.com"
}
```

## 6. 배열

배열은 여러 값을 순서대로 저장하는 데이터이다.

```javascript
const names = ["kim", "lee", "park"];

console.log(names[0]); // kim
```

객체 배열도 자주 사용한다.

```javascript
const members = [
  { id: 1, name: "kim" },
  { id: 2, name: "lee" }
];
```

API 응답에서 목록 데이터는 보통 배열 형태로 온다.

## 7. 배열 메서드

Vue에서 목록을 화면에 보여주거나 데이터를 가공할 때 배열 메서드를 자주 사용한다.

### map

배열의 각 요소를 다른 형태로 변환한다.

```javascript
const members = [
  { id: 1, name: "kim" },
  { id: 2, name: "lee" }
];

const names = members.map(member => member.name);

console.log(names); // ["kim", "lee"]
```

정리:

```text
map → 배열을 다른 배열로 변환
```

### filter

조건에 맞는 요소만 남긴다.

```javascript
const members = [
  { id: 1, name: "kim" },
  { id: 2, name: "lee" }
];

const filtered = members.filter(member => member.id > 1);

console.log(filtered); // [{ id: 2, name: "lee" }]
```

정리:

```text
filter → 조건에 맞는 것만 남김
```

### find

조건에 맞는 첫 번째 요소를 찾는다.

```javascript
const member = members.find(member => member.id === 1);

console.log(member); // { id: 1, name: "kim" }
```

정리:

```text
find → 조건에 맞는 첫 번째 값 찾음
```

## 8. 조건문과 삼항 연산자

조건문은 조건에 따라 다른 코드를 실행할 때 사용한다.

```javascript
const isLogin = true;

if (isLogin) {
  console.log("로그인됨");
} else {
  console.log("로그인 필요");
}
```

삼항 연산자는 간단한 조건을 한 줄로 표현할 때 사용한다.

```javascript
const buttonText = isLogin ? "로그아웃" : "로그인";
```

Vue 템플릿에서도 조건에 따라 화면을 다르게 보여줄 때 자주 사용한다.

## 9. 구조 분해 할당

구조 분해 할당은 객체나 배열에서 값을 쉽게 꺼내는 문법이다.

객체:

```javascript
const user = {
  name: "kim",
  email: "kim@test.com"
};

const { name, email } = user;

console.log(name);  // kim
console.log(email); // kim@test.com
```

배열:

```javascript
const names = ["kim", "lee"];

const [first, second] = names;

console.log(first);  // kim
console.log(second); // lee
```

## 10. import / export

JavaScript는 파일을 나누어 코드를 작성할 수 있다.

내보내기:

```javascript
export const API_URL = "http://localhost:8080";
```

가져오기:

```javascript
import { API_URL } from "./config";
```

Vue 프로젝트에서는 컴포넌트나 함수, 설정 파일을 나눠서 관리하기 때문에 `import`, `export`를 자주 사용한다.

## 11. 비동기 처리

서버 API 호출은 시간이 걸리기 때문에 비동기 처리가 필요하다.

대표적으로 `Promise`, `async/await`를 사용한다.

```javascript
async function fetchMembers() {
  const response = await fetch("/api/members");
  const data = await response.json();

  console.log(data);
}
```

의미:

```text
async → 비동기 함수를 만든다.
await → 결과가 올 때까지 기다린다.
```

Spring API를 호출할 때 거의 필수로 사용한다.

## 12. fetch와 axios

프론트엔드에서 백엔드 API를 호출할 때 `fetch`나 `axios`를 사용한다.

### fetch

```javascript
const response = await fetch("http://localhost:8080/members");
const data = await response.json();
```

### axios

```javascript
const response = await axios.get("http://localhost:8080/members");

console.log(response.data);
```

POST 요청:

```javascript
await axios.post("http://localhost:8080/members", {
  name: "kim",
  email: "kim@test.com"
});
```

토큰을 헤더에 담아 보내기:

```javascript
await axios.get("http://localhost:8080/members/me", {
  headers: {
    Authorization: `Bearer ${accessToken}`
  }
});
```

## 13. fetch와 axios 비교

`fetch`와 `axios`는 둘 다 프론트엔드에서 백엔드 API를 호출할 때 사용한다.

### fetch

`fetch`는 브라우저에 기본으로 내장된 API이다.

따로 설치할 필요가 없다.

```javascript
const response = await fetch("http://localhost:8080/members");
const data = await response.json();
```

POST 요청:

```javascript
await fetch("http://localhost:8080/members", {
  method: "POST",
  headers: {
    "Content-Type": "application/json"
  },
  body: JSON.stringify({
    name: "kim",
    email: "kim@test.com"
  })
});
```

특징:

- 브라우저 기본 제공
- 설치 필요 없음
- 응답 데이터를 직접 `.json()`으로 변환해야 함
- 400, 500 같은 HTTP 에러가 와도 자동으로 `catch`로 가지 않음

예를 들어 404 응답이 와도 직접 확인해야 한다.

```javascript
const response = await fetch("/members/999");

if (!response.ok) {
  throw new Error("요청 실패");
}
```

### axios

`axios`는 외부 라이브러리이다.

설치가 필요하다.

```bash
npm install axios
```

GET 요청:

```javascript
const response = await axios.get("http://localhost:8080/members");

console.log(response.data);
```

POST 요청:

```javascript
await axios.post("http://localhost:8080/members", {
  name: "kim",
  email: "kim@test.com"
});
```

특징:

- 별도 설치 필요
- JSON 변환을 자동으로 처리함
- 응답 데이터는 `response.data`에 들어 있음
- 400, 500 같은 HTTP 에러를 자동으로 `catch`로 보냄
- 요청/응답 인터셉터 설정이 편함
- 로그인 토큰 자동 첨부 같은 공통 처리에 좋음

토큰 자동 첨부 예시:

```javascript
axios.interceptors.request.use(config => {
  const token = localStorage.getItem("accessToken");

  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }

  return config;
});
```

비교 정리:

```text
fetch
→ 브라우저 기본 제공
→ 가볍다
→ 직접 처리할 것이 많다
→ 간단한 요청에 좋다

axios
→ 설치 필요
→ JSON 처리 편함
→ 에러 처리 편함
→ 인터셉터로 토큰 처리 편함
→ 실무 프로젝트에서 자주 사용
```

Spring + Vue를 연결할 때는 `axios`를 많이 사용한다.

특히 로그인 토큰 첨부, 401 처리, Refresh Token 재발급 같은 공통 로직을 인터셉터로 관리하기 좋다.

한 줄 정리:

> fetch는 브라우저 기본 API이고, axios는 API 통신을 더 편하게 해주는 라이브러리이다. 간단한 요청은 fetch로 충분하고, 실무 프로젝트에서는 axios가 에러 처리와 토큰 관리에 더 편하다.

## 14. JSON

JSON은 프론트엔드와 백엔드가 데이터를 주고받을 때 많이 사용하는 형식이다.

```json
{
  "id": 1,
  "name": "kim",
  "email": "kim@test.com"
}
```

JavaScript에서는 JSON 데이터를 객체처럼 사용할 수 있다.

```javascript
console.log(user.name);
```

Spring REST API도 보통 JSON으로 요청과 응답을 주고받는다.

## 15. DOM 기본

DOM은 HTML 문서를 JavaScript가 다룰 수 있게 객체 형태로 표현한 것이다.

순수 JavaScript에서는 DOM을 직접 조작할 수 있다.

```javascript
document.querySelector("#title").textContent = "Hello";
```

하지만 Vue를 사용하면 DOM을 직접 조작하기보다 데이터 상태를 바꾸고, Vue가 화면을 자동으로 업데이트하게 만든다.

```text
순수 JavaScript
→ DOM을 직접 찾아서 수정

Vue
→ 데이터를 바꾸면 화면이 자동 업데이트
```

## 16. Vue와 연결되는 개념

Vue에서 쓰는 `ref`, `reactive`, `computed`, `watch` 같은 기능은 JavaScript 자체 문법은 아니고 Vue 문법이다.

하지만 내부에서는 JavaScript를 사용한다.

예시:

```vue
<script setup>
import { ref, computed } from "vue";

const count = ref(0);

const doubleCount = computed(() => count.value * 2);
</script>

<template>
  <p>{{ doubleCount }}</p>
  <button @click="count++">증가</button>
</template>
```

여기서:

```text
ref, computed, template, @click
→ Vue 문법

const, arrow function, count.value * 2
→ JavaScript 문법
```

## 17. 학습 순서

추천 학습 순서:

```text
1. 변수, 함수, 조건문
2. 객체, 배열
3. map, filter, find
4. 구조 분해 할당
5. import, export
6. async, await
7. fetch, axios
8. JSON
9. Vue ref, reactive, template
```

## 18. 한 번에 외우는 답변

> JavaScript는 웹 브라우저에서 동작하는 프로그래밍 언어로, 웹 페이지의 동적인 동작을 처리합니다. Vue도 JavaScript를 기반으로 동작하기 때문에 Vue를 공부하기 전에 변수, 함수, 객체, 배열, 배열 메서드, 비동기 처리, API 호출, JSON 같은 기본 개념을 알아야 합니다. 특히 Spring 백엔드와 연결할 때는 `fetch`나 `axios`로 API를 호출하고 JSON 데이터를 주고받는 흐름이 중요합니다.
