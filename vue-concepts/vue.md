# Vue 정리

## 1. Vue란?

Vue는 JavaScript 기반 프론트엔드 프레임워크이다.

HTML, CSS, JavaScript만으로도 화면을 만들 수 있지만, 화면이 커질수록 데이터 변경, 이벤트 처리, API 호출, 페이지 이동을 직접 관리하기 복잡해진다.

Vue는 이런 화면 개발을 쉽게 하기 위해 다음 기능을 제공한다.

```text
데이터가 바뀌면 화면 자동 변경
컴포넌트 단위로 화면 분리
라우터로 페이지 이동 관리
Pinia로 전역 상태 관리
템플릿 문법으로 조건, 반복, 이벤트 처리
```

면접 답변:

> Vue는 JavaScript 기반 프론트엔드 프레임워크입니다. 데이터와 화면을 연결해서 데이터가 변경되면 화면이 자동으로 갱신되도록 도와주고, 컴포넌트 단위로 UI를 나누어 관리할 수 있게 해줍니다.

## 2. Vue를 하려면 알아야 하는 것

Vue를 공부하기 전에 JavaScript 기본 문법을 알고 있어야 한다.

```text
let, const
함수
객체
배열
map, filter, find
Promise
async / await
axios
import / export
```

Vue에서 추가로 알아야 할 개념은 다음과 같다.

```text
ref
reactive
template 문법
v-if
v-for
v-model
@click
computed
watch
props
emit
component
router
store
```

정리:

```text
JavaScript 문법
→ 언어 자체 문법

Vue 문법
→ Vue에서 화면과 데이터를 연결하기 위해 쓰는 문법
```

## 3. Vue 프로젝트 기본 구조

일반적인 Vue 프로젝트 구조는 다음과 같다.

```text
src
├── main.js
├── App.vue
├── router
│   └── index.js
├── api
│   ├── http-commons.js
│   ├── member.js
│   └── post.js
├── stores
│   ├── member.js
│   └── menu.js
├── views
│   ├── MainView.vue
│   ├── LoginView.vue
│   └── DetailView.vue
├── components
│   ├── common
│   ├── members
│   └── posts
└── assets
    ├── images
    └── styles
```

역할:

```text
main.js
→ Vue 앱 시작 파일

App.vue
→ 최상위 화면 컴포넌트

router
→ URL과 화면 컴포넌트를 연결

api
→ 백엔드 API 호출 함수 관리

stores
→ 로그인 정보 같은 전역 상태 관리

views
→ 페이지 단위 컴포넌트

components
→ 재사용 가능한 작은 컴포넌트

assets
→ 이미지, CSS 같은 정적 파일
```

## 4. main.js

`main.js`는 Vue 앱이 처음 시작되는 파일이다.

예시:

```javascript
import { createApp } from "vue";
import { createPinia } from "pinia";
import App from "./App.vue";
import router from "./router";

const app = createApp(App);

app.use(createPinia());
app.use(router);

app.mount("#app");
```

흐름:

```text
App.vue 가져오기
Vue 앱 생성
Pinia 등록
Router 등록
index.html의 #app 위치에 화면 붙이기
```

정리:

```text
main.js는 Vue 프로젝트의 시작점이다.
```

## 5. App.vue

`App.vue`는 최상위 컴포넌트이다.

보통 공통 레이아웃을 둔다.

```vue
<template>
  <Header />
  <RouterView />
  <Footer />
</template>
```

`RouterView`는 현재 URL에 맞는 화면을 보여주는 자리이다.

예시:

```text
/members
→ MemberListView.vue 표시

/members/1
→ MemberDetailView.vue 표시
```

정리:

```text
App.vue는 전체 화면의 틀이고,
RouterView는 URL에 따라 바뀌는 페이지 영역이다.
```

## 6. Vue 파일 구조

Vue 컴포넌트 파일은 보통 세 부분으로 나뉜다.

```vue
<script setup>
// JavaScript 코드
</script>

<template>
  <!-- 화면 HTML -->
</template>

<style scoped>
/* 이 컴포넌트에만 적용되는 CSS */
</style>
```

역할:

```text
script
→ 변수, 함수, API 호출, 라우터 이동 로직

template
→ 실제 화면 구조

style
→ 디자인
```

`scoped`를 붙이면 해당 컴포넌트 안에서만 CSS가 적용된다.

## 7. ref

`ref`는 Vue에서 반응형 데이터를 만들 때 사용한다.

반응형 데이터란 값이 바뀌면 Vue가 감지해서 화면을 자동으로 다시 그리는 데이터이다.

예시:

```vue
<script setup>
import { ref } from "vue";

const count = ref(0);

const increase = () => {
  count.value++;
};
</script>

<template>
  <p>{{ count }}</p>
  <button @click="increase">증가</button>
</template>
```

주의:

```text
script 안에서는 count.value
template 안에서는 count
```

면접 답변:

> ref는 Vue에서 반응형 데이터를 만들 때 사용합니다. 값이 변경되면 Vue가 이를 감지해서 화면을 자동으로 업데이트합니다.

## 8. reactive

`reactive`는 객체를 반응형으로 만들 때 사용한다.

예시:

```vue
<script setup>
import { reactive } from "vue";

const member = reactive({
  name: "",
  email: ""
});
</script>

<template>
  <input v-model="member.name" />
  <input v-model="member.email" />
</template>
```

정리:

```text
ref
→ 문자열, 숫자, 배열, 객체 모두 가능
→ script에서 .value 필요

reactive
→ 객체에 많이 사용
→ .value 없이 사용
```

처음에는 `ref` 위주로 써도 된다.

## 9. template 문법

Vue의 `<template>` 안에서는 HTML처럼 작성하면서 Vue 문법을 함께 사용할 수 있다.

데이터 출력:

```vue
<p>{{ name }}</p>
```

속성 바인딩:

```vue
<img :src="imageUrl" />
```

`:`는 `v-bind:`의 줄임말이다.

```vue
<img v-bind:src="imageUrl" />
```

## 10. v-if

`v-if`는 조건에 따라 화면을 보여주거나 숨긴다.

예시:

```vue
<p v-if="isLogin">로그인 상태입니다.</p>
<p v-else>로그인이 필요합니다.</p>
```

정리:

```text
v-if
→ 조건이 true일 때만 화면에 보임
```

## 11. v-for

`v-for`는 배열을 반복해서 화면에 출력할 때 사용한다.

예시:

```vue
<ul>
  <li v-for="member in members" :key="member.id">
    {{ member.name }}
  </li>
</ul>
```

`members`가 배열이면 배열 개수만큼 `li`가 반복된다.

```javascript
const members = ref([
  { id: 1, name: "kim" },
  { id: 2, name: "lee" }
]);
```

`key`를 쓰는 이유:

```text
Vue가 각각의 요소를 구분하기 위해 사용한다.
반복 렌더링할 때는 보통 고유 id를 key로 넣는다.
```

정리:

```text
v-for는 Vue 전용 문법이다.
JavaScript의 반복문은 for, forEach, map이다.
```

## 12. v-model

`v-model`은 입력값과 데이터를 양방향으로 연결한다.

예시:

```vue
<script setup>
import { ref } from "vue";

const name = ref("");
</script>

<template>
  <input v-model="name" />
  <p>{{ name }}</p>
</template>
```

동작:

```text
input에 입력
→ name 값 변경
→ 화면도 자동 변경
```

회원가입, 로그인 폼에서 자주 사용한다.

## 13. @click

`@click`은 클릭 이벤트를 처리한다.

예시:

```vue
<script setup>
const save = () => {
  console.log("저장");
};
</script>

<template>
  <button @click="save">저장</button>
</template>
```

`@click`은 `v-on:click`의 줄임말이다.

```vue
<button v-on:click="save">저장</button>
```

## 14. form submit

폼 제출은 `@submit.prevent`를 많이 사용한다.

예시:

```vue
<form @submit.prevent="handleRegister">
  <input v-model="name" />
  <button type="submit">회원가입</button>
</form>
```

`prevent`를 쓰는 이유:

```text
form submit 기본 동작은 페이지 새로고침이다.
Vue에서는 새로고침하지 않고 함수만 실행하기 위해 prevent를 붙인다.
```

## 15. computed

`computed`는 기존 데이터를 기반으로 계산된 값을 만들 때 사용한다.

예시:

```vue
<script setup>
import { ref, computed } from "vue";

const firstName = ref("kim");
const lastName = ref("seong");

const fullName = computed(() => {
  return `${lastName.value} ${firstName.value}`;
});
</script>

<template>
  <p>{{ fullName }}</p>
</template>
```

정리:

```text
computed
→ 기존 값으로 새로운 값을 계산
→ 관련 데이터가 바뀌면 자동으로 다시 계산
```

## 16. watch

`watch`는 특정 값이 바뀌는 것을 감지해서 코드를 실행한다.

예시:

```vue
<script setup>
import { ref, watch } from "vue";

const area = ref("");

watch(area, (newArea) => {
  console.log("지역 변경:", newArea);
});
</script>
```

객체 안의 값을 감시할 수도 있다.

```javascript
watch(
  () => searchParams.value.area,
  (newArea) => {
    if (newArea) {
      updateGugunList(newArea);
    }
  }
);
```

정리:

```text
watch
→ 값이 바뀌는 순간 특정 로직 실행
→ 검색 조건 변경, 지역 선택 변경, API 재호출에 자주 사용
```

## 17. onMounted

`onMounted`는 컴포넌트가 화면에 처음 나타났을 때 실행된다.

예시:

```vue
<script setup>
import { onMounted } from "vue";

onMounted(() => {
  console.log("화면이 처음 열림");
});
</script>
```

자주 쓰는 경우:

```text
페이지 처음 열릴 때 목록 조회
상세 페이지 처음 열릴 때 상세 정보 조회
지역 목록 처음 불러오기
로그인 사용자 정보 조회
```

## 18. props

`props`는 부모 컴포넌트가 자식 컴포넌트에게 값을 넘길 때 사용한다.

부모:

```vue
<MemberCard :member="member" />
```

자식:

```vue
<script setup>
const props = defineProps({
  member: Object
});
</script>

<template>
  <p>{{ member.name }}</p>
</template>
```

정리:

```text
props
→ 부모에서 자식으로 데이터 전달
```

## 19. emit

`emit`은 자식 컴포넌트가 부모 컴포넌트에게 이벤트를 보낼 때 사용한다.

자식:

```vue
<script setup>
const emit = defineEmits(["select"]);

const selectMember = () => {
  emit("select", 1);
};
</script>

<template>
  <button @click="selectMember">선택</button>
</template>
```

부모:

```vue
<MemberCard @select="handleSelect" />
```

정리:

```text
emit
→ 자식에서 부모로 이벤트 전달
```

## 20. component

컴포넌트는 화면을 작은 단위로 나눈 것이다.

예시:

```text
Header.vue
MemberList.vue
MemberCard.vue
MemberDetail.vue
LoginForm.vue
```

컴포넌트를 쓰는 이유:

```text
화면을 역할별로 나눌 수 있다.
재사용할 수 있다.
파일이 너무 커지는 것을 막을 수 있다.
유지보수가 쉬워진다.
```

## 21. router

Router는 URL과 화면 컴포넌트를 연결한다.

예시:

```javascript
import { createRouter, createWebHistory } from "vue-router";

const router = createRouter({
  history: createWebHistory(),
  routes: [
    {
      path: "/",
      name: "main",
      component: () => import("@/views/MainView.vue")
    },
    {
      path: "/members/:id",
      name: "member-detail",
      component: () => import("@/views/MemberDetailView.vue")
    }
  ]
});

export default router;
```

페이지 이동:

```vue
<script setup>
import { useRouter } from "vue-router";

const router = useRouter();

const goLogin = () => {
  router.push("/user/login");
};
</script>

<template>
  <button @click="goLogin">로그인으로 이동</button>
</template>
```

정리:

```text
router/index.js
→ 전체 라우터 설정

useRouter()
→ 컴포넌트 안에서 페이지 이동할 때 사용
```

## 22. RouterLink

단순히 버튼이나 링크를 눌러 이동할 때는 `RouterLink`를 사용할 수 있다.

```vue
<RouterLink to="/">메인</RouterLink>
<RouterLink to="/user/login">로그인</RouterLink>
```

로직 처리 후 이동해야 하면 `useRouter()`를 사용한다.

```text
단순 이동
→ RouterLink

회원가입 성공 후 이동
→ router.replace("/user/login")
```

## 23. api 폴더

백엔드 호출 함수는 보통 `src/api` 폴더에 만든다.

예시:

```text
src/api/http-commons.js
src/api/member.js
src/api/post.js
```

`http-commons.js`:

```javascript
import axios from "axios";

const { VITE_VUE_API_URL } = import.meta.env;

function localAxios() {
  const instance = axios.create({
    baseURL: VITE_VUE_API_URL
  });

  instance.defaults.headers.post["Content-Type"] = "application/json";
  instance.defaults.headers.put["Content-Type"] = "application/json";

  return instance;
}

export { localAxios };
```

`member.js`:

```javascript
import { localAxios } from "@/api/http-commons";

const local = localAxios();

function joinMember(request, success, fail) {
  local.post("/members", request).then(success).catch(fail);
}

function findMember(id, success, fail) {
  local.get(`/members/${id}`).then(success).catch(fail);
}

export { joinMember, findMember };
```

정리:

```text
컴포넌트
→ api 함수 호출
→ axios
→ 백엔드 API 요청
```

## 24. axios 결과값

`axios`로 요청하면 결과는 `response`로 받는다.

예시:

```javascript
const response = await axios.post("http://localhost:8080/members", {
  name: "kim",
  email: "kim@test.com"
});

console.log(response.data);
```

백엔드가 다음처럼 응답하면:

```json
{
  "id": 1,
  "name": "kim",
  "email": "kim@test.com"
}
```

프론트에서는 다음 위치에 들어온다.

```javascript
response.data
```

목록이면:

```json
[
  { "id": 1, "name": "kim" },
  { "id": 2, "name": "lee" }
]
```

프론트에서는:

```javascript
members.value = response.data;
```

그리고 화면에서는:

```vue
<li v-for="member in members" :key="member.id">
  {{ member.name }}
</li>
```

## 25. stores

`stores`는 전역 상태를 관리하는 폴더이다.

전역 상태란 여러 컴포넌트에서 같이 써야 하는 데이터이다.

예시:

```text
로그인 여부
사용자 정보
accessToken
메뉴 표시 상태
장바구니
선택된 테마
```

Pinia store 예시:

```javascript
import { ref } from "vue";
import { defineStore } from "pinia";

export const useMemberStore = defineStore(
  "memberStore",
  () => {
    const isLogin = ref(false);
    const userInfo = ref(null);

    const setLogin = (user) => {
      isLogin.value = true;
      userInfo.value = user;
    };

    const logout = () => {
      isLogin.value = false;
      userInfo.value = null;
      localStorage.removeItem("accessToken");
    };

    return {
      isLogin,
      userInfo,
      setLogin,
      logout
    };
  },
  {
    persist: true
  }
);
```

정리:

```text
props
→ 부모와 자식 사이 데이터 전달

store
→ 여러 페이지와 컴포넌트가 함께 쓰는 데이터 관리
```

## 26. 로그인 권한 체크

라우터에서 특정 페이지는 로그인한 사용자만 들어가게 할 수 있다.

```javascript
const onlyAuthUser = async (to, from, next) => {
  const token = localStorage.getItem("accessToken");

  if (!token) {
    next({ name: "user-login" });
    return;
  }

  next();
};
```

라우터에 적용:

```javascript
{
  path: "/mypage",
  name: "user-mypage",
  beforeEnter: onlyAuthUser,
  component: () => import("@/views/MyPageView.vue")
}
```

정리:

```text
beforeEnter
→ 해당 페이지에 들어가기 전에 실행되는 함수
→ 로그인 여부 검사에 자주 사용
```

## 27. 회원가입 버튼 흐름

예시:

```vue
<script setup>
import { ref } from "vue";
import axios from "axios";
import { useRouter } from "vue-router";

const router = useRouter();

const name = ref("");
const email = ref("");
const password = ref("");
const message = ref("");

const handleRegister = async () => {
  try {
    const response = await axios.post("http://localhost:8080/members", {
      name: name.value,
      email: email.value,
      password: password.value
    });

    message.value = response.data.message || "회원가입이 완료되었습니다.";
    router.replace("/user/login");
  } catch (error) {
    message.value =
      error.response?.data?.message || "회원가입 중 오류가 발생했습니다.";
  }
};
</script>

<template>
  <form @submit.prevent="handleRegister">
    <input v-model="name" placeholder="이름" />
    <input v-model="email" placeholder="이메일" />
    <input v-model="password" type="password" placeholder="비밀번호" />

    <p v-if="message">{{ message }}</p>

    <button type="submit">회원가입</button>
  </form>
</template>
```

동작 흐름:

```text
사용자가 input에 입력
→ v-model 때문에 ref 값이 바뀜
→ 회원가입 버튼 클릭
→ form submit 발생
→ @submit.prevent가 새로고침 막고 handleRegister 실행
→ axios로 백엔드 회원가입 API 호출
→ 성공하면 로그인 페이지로 이동
→ 실패하면 에러 메시지 표시
```

## 28. 함수는 어디에 만들까?

컴포넌트 안에서만 쓰는 함수는 컴포넌트 안에 만든다.

예시:

```text
input 초기화
버튼 클릭 처리
모달 열기
현재 화면에서만 쓰는 검색 조건 변경
```

여러 컴포넌트에서 재사용하거나 백엔드 API를 호출하는 함수는 따로 분리한다.

```text
src/api
→ 백엔드 API 호출 함수

src/stores
→ 전역 상태와 관련된 함수

src/utils
→ 공통 유틸 함수
```

정리:

```text
화면 전용 함수
→ 컴포넌트 안

API 호출 함수
→ api 폴더

여러 화면이 같이 쓰는 상태
→ stores 폴더
```

## 29. Vue 문법 한 번에 정리

```text
{{ }}
→ 데이터 출력

:속성
→ 속성에 변수 연결

v-if
→ 조건부 렌더링

v-for
→ 배열 반복 출력

v-model
→ input 값과 데이터 양방향 연결

@click
→ 클릭 이벤트 처리

@submit.prevent
→ form 제출 시 새로고침 막고 함수 실행

computed
→ 계산된 값

watch
→ 값 변경 감지

onMounted
→ 화면이 처음 열릴 때 실행

props
→ 부모에서 자식으로 데이터 전달

emit
→ 자식에서 부모로 이벤트 전달

RouterView
→ 현재 URL에 맞는 페이지 표시

RouterLink
→ 페이지 이동 링크

useRouter
→ 코드로 페이지 이동
```

## 30. 면접 답변

> Vue는 JavaScript 기반 프론트엔드 프레임워크입니다. 데이터를 반응형으로 관리해서 데이터가 바뀌면 화면이 자동으로 업데이트됩니다. 화면은 컴포넌트 단위로 나누어 관리하고, Router를 통해 URL별 페이지를 연결하며, Pinia 같은 store를 사용해 로그인 정보처럼 여러 컴포넌트에서 공유해야 하는 상태를 관리합니다.

> Vue 문법에는 데이터를 출력하는 이중 중괄호, 조건부 렌더링을 위한 v-if, 배열 반복을 위한 v-for, 입력값과 데이터를 연결하는 v-model, 이벤트 처리를 위한 @click 등이 있습니다. 컴포넌트가 처음 화면에 나타날 때는 onMounted를 사용하고, 값의 변화를 감지해야 할 때는 watch를 사용합니다.
