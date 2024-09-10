---
title: 리액트 클린코드 스터디 회고(feat.동탄모각코)
date: 2024-09-10 01:15:00 +0900
categories: [Experience, Study]
tags: [Experience, Diary, Frontend, Study]
image: https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F6-react-clean-code%2Fimage-1.png?alt=media&token=d4a51a9e-e6d2-404a-b869-dc7e58bbd726
---

## clean-code-react를 고른 이유
리액트 클린코드를 공부한 이유는 두 가지입니다. 1)내가 짜고 있는 리액트 코드를 더 효율적으로 짜는 방법을 알고 싶었고, 2)Next.js를 공부하면서 리액트를 Deep하게 공부하고 싶었습니다(Next.js 공부의 결론이 React를 알고 싶다였습니다..!).

- [문득 생각이 들었다. 나는 왜 Next.js를 쓰고 있지?](https://kidongg.github.io/posts/why-i-use-next-js/){:target="\_blank"}
- [Next.js Learn 번역](https://kidongg.github.io/categories/next-js-learn/){:target="\_blank"}

실무에서 적용 가능한 리액트 공부를 하고 싶었고, 여러 인강을 찾던 중 리액트 클린코드를 발견했습니다. "리액트의 주요 안티 패턴을 파악하고 개선한다"는 목표가 저를 사로 잡았습니다. 때마침 동탄에서 모각코를 함께하고 있는 스터디원도 리액트를 공부하고 싶어 하셨고, [스터디 형태](https://github.com/Kidongg/clean-code-react){:target="\_blank"}로 진행을 하기로 했습니다.

## clean-code-react에서 배운 내용

### 0. Overview
React+TypeScript 조합과 Tanstack Query가 일반화되고 있으며, 프론트 상태관리(Jotai[like Recoil]과 Zustand[like Redux])를 위한 다양한 라이브러리가 경쟁을 하고 있다는 말에 공감이 되었습니다. 

React Server Component의 등장으로 프론트엔드의 생태계가 HTML, CSS, JavaScript처럼 브라우저 단계만 보는 세상이 허물어지고 있는 것 같습니다. 클라이언트와 서버와의 경계가 희미해지고 있는 것 같습니다.

### 1. State

#### Q1. 리액트 상태는 어디에 저장할까?
- 리액트 상태는 브라우저 메모리에 저장된다.
- 브라우저 메모리 종류
    - 휘발성 : 리액트 상태
    - 비휘발성 : localStorage, sessionStorage, IndexedDB

#### Q2. Date Fetching을 Custom Hook으로 묶는 예제

```react
// 초기 상태
const INIT_STATE = {
    isLoading: false,
    isFinish: false,
    isError: false
}

// 액션 타입
const ACTION_TYPE = {
    FETCH_LOADING: "FETCH_LOADING",
    FETCH_FINISH: "FETCH_FINISH",
    FETCH_ERROR: "FETCH_ERROR"
}

// 리듀서 함수
const reducer = (state, action) => {
    switch (action.type) {
        case "FETCH_LOADING":
            return { isLoading: true, isFinish: false, isError: false }

        case "FETCH_FINISH":
            return { isLoading: false, isFinish: true, isError: false }

        case "FETCH_ERROR":
            return { isLoading: false, isFinish: false, isError: true }

        default:
            INIT_STATE
    }
}

// 커스텀 훅
const useFetchData = (url) => {
    const [state, dispatch] = useReducer(reducer, INIT_STATE)

    useEffect(() => {
        const fetchData = async () => {
            dispatch({ type: ACTION_TYPE.FETCH_LOADING })

            await fetch(url)
            .then(() => {
                // fetch Data 성공
                dispatch({ type: ACTION_TYPE.FETCH_FINISH })
            })
            .catch(() => {
                // fetch Data 실패
                dispatch({ type: ACTION_TYPE.FETCH_ERROR })
            })
        }

        fetchData()
    }, [url])

    return state
}

const Component = () => {
    const { isLoading, isFinish, isError } = useFetchData("url")

    if (isLoading) return <LoadingComponent />
    if (isFinish) return <SuccessComponent />
    if (isError) return <ErrorComponent />
}
```

#### State에서 배운점
- 비휘발성 상태가 필요할때 localStorage, sessionStorage, IndexedDB를 사용한다.
- Tanstack Query는 위 Custom Hook 로직을 추상화하여 제공하기 때문에 실무 코드에 적용은 힘들 것 같다.

### 2. Props

#### Q1. 좋은 Props 네이밍의 특징은 무엇일까?
- 네이밍 규칙이 일관적이다.
- 역할과 의미가 명확하다.

#### Q2. 객체 Props를 지양해야하는 이유는 무엇일까?
- 불필요한 렌더링 : 동일한 참조를 유지하지 못하는 경우 컴포넌트가 불필요하게 렌더링 될 수 있다.
- 불변성 원칙 위반 : 객체 내부의 값이 변하더라고 객체의 참조가 동일하게 유지할 수 될 수 있다.
- 컴포넌트 복잡도 증가
- 테스트와 디버깅의 어려움
- PropTypes 검증의 복잡성 증가

#### Q3. 복잡한 Props를 개선하는 방법은 무엇일까?
- 필요한 필드만 Props로 전달하기
- 상태를 분할하여 관리하기(ex. Redux, Zustand)
- 메모이제이션 적용하기(ex. `React.memo`, `useMemo`)

#### Props에서 배운점
- Props 관리 측면을 고려하여 코드를 짠다.
    - Props 네이밍 규칙 정하기
    - Props 네이밍 시 역할과 의미를 충분히 고민하기
    - Q3의 방법을 단계적으로 적용하기(with 객체 Props 지양)

### 3. Components

#### Q1. 리액트 컴포넌트란 무엇인가?
- 개념 : UI를 구성하는 기본 단위이다. 독립적이고 재사용 가능한 코드 블록으로 화면을 렌더링한다.
- 필요성
    - 애플리케이션의 UI 구조를 나눌 때
    - 반복되는 UI 패턴이 있을 때
    - 상태를 관리해야 할 때
- 장점/특징
    - 재사용성
    - 유지보수성
    - 상태 관리 : 상태를 관리하고, 상태 변화에 따라 UI를 자동으로 업데이트할 수 있다.
    - 최적화 : 메모이제이션을 통해 컴포넌트 단위로 최적화할 수 있다.

#### Q2. Fragment란 무엇인가?
- 개념 : DOM에 별도의 노드를 추가하지 않고 여러 자식을 그룹화할 수 있는 문법
- 필요성 : 단일 요소를 반환할 때(JSX 문법)

```react
const Component = () => {
  return (
    <React.Fragment>
      <ChildA />
      <ChildA />
    </React.Fragment>
  )
}
```

#### Components에서 배운점
- 나만의 단계에 따라 컴포넌트를 만들기
  1. 애플리케이션 UI 구조를 나눌때
  2. 반복되는 UI 패턴이 발견됐을때
  3. 반복되는 상태 패턴이 발견됐을때
  → 재사용성과 유지보수성을 고려하여 코드를 짜는 것이 목표
- 단일 요소를 반환할 때 `React.Fragment`를 사용하기. DOM 노드가 생성되지 않아 좋은 것 같다.

### 4. Rendering

#### Q1. JSX에서 공백을 처리하는 방법은?
- `{' '}`로 처리할 수 있다.

```react
// 두 문법은 똑같이 동작한다. 가독성을 위해 후자를 사용하는 것을 추천한다.

Welcome Clean Code&nbsp;
Welcome Clean Code{' '}
```

#### Q2. Raw HTML이란 무엇인가?
- 개념 : 브라우저나 기타 도구에 의해 처리되기 전에 있는 순수한 HTML 태그와 텍스트

```react
<div>
  <h1>Hello, World!</h1>
  <p>This is a paragraph of text.</p>
  <a href="https://example.com">Visit Example.com</a>
</div>
```

- 필요성
  - HTML 포맷의 콘텐츠를 서버에서 받아와 렌더링 할때
  - 동적으로 생성된 HTML을 렌더링 할때

- 보안 위협 : 유저 입력모드로 렌더링하는 데이터의 경우 XSS 공격에 취약하다. 이럴 경우 보안작업이 필요한데 이때 DOMPurify, HTML Sanitiazer API, eslint-plugin-risxss를 통해 쉽게 처리할 수 있다.

- 코드 작성방법

```react
// 동작하지 않는 코드
const SERVER_DATA = '<p>some row html</p>'

const markup = { __html: SERVER_DATA }

return <div>{markup}</div>
```

```react
// 동작하는 코드
const SERVER_DATA = '<p>some row html</p>'

const markup = { __html: SERVER_DATA }

return <div dangerousSetInnerHTML={markup} />
```

```react
// 보안 처리를 한 코드
const SERVER_DATA = '<p>some row html</p>'

const sanitizeContent = { __html: DOMPurify.sanitize(SERVER_DATA) }

return <div dangerousSetInnerHTML={sanitizeContent} />
```

#### Rendering에서 배운점
- JSX에서 0처리에 주의하기(0은 JSX에서는 truthy, JavaScript에서는 falsy)

```react
// 수정 전
return <div>{items.length && items.map((item)...)}</div>

// 수정 후
return <div>{items.length > 0 && items.map((item)...)}</div>
```

- JSX에서 공백을`{' '}`를 통해 처리하기

```react
// 수정 전
Welcome Clean Code&nbsp;

// 수정 후
Welcome Clean Code{' '}
```

### 5. Hooks

#### Q1. useEffect를 사용할때 신경써야할 점은?
- 단일책임 원칙을 지킨다. 의존성이 명확히 분리되기 때문이다. 의도치 않은 로직을 타는 위험을 줄일 수 있다.

```react
// 수정 전
useEffect(() => {
  redirect(newPath)
  const userInfo = setLogin(token)
}, [newPath, token])
```

```react
// 수정 후
useEffect(() => {
  redirect(newPath)
}, [newPath])

useEffect(() => {
  const userInfo = setLogin(token)
}, [token])
```

#### Q2. Custom Hook의 반환타입을 객체로 하는 이유는?
- 이름을 바꿔서 가져올 수 있다.
- 필요한 데이터만 뽑아서 가져올 수 있다.

```react
const useCustomHook = () => {
  // some logic

  return {first, second, third, unnecessaryData, state}
}
```

#### Hooks에서 배운점
- 단일 책임 원칙으로 useEffect 사용하기
- Custom Hook의 반환 타입을 객체로 설정하기
- useEffect보단 TanStack Query를 통해서 데이터 통신하기

### 6. Etc

#### Q1. 리액트 디렉토리 구조를 어떻게 해야할까?
- 기능에 따라 분리하는 방법

```shell
components
  ㄴ @shared
    ㄴ 공통 컴포넌트
  ㄴ Todo
    ㄴ Todo.jsx
    ㄴ Todo.hook.js
    ㄴ Todo.css
```

- 역할에 따라 분리하는 방법

```shell
hooks
  ㄴ useTodo.js
style
  ㄴ Todo.css
components
  ㄴ Todo.jsx
```

#### Q2. Duck 패턴이 무엇일까?
- 개념 : 상태 관리 관련 파일을 기능별로 그룹화하여, 하나의 모듈(혹은 Duck) 내에서 액션 타입, 액션 생성자, 리듀서, 셀렉터 등을 모두 포함하는 구조
- 예제

```shell
src/
├── features/
│   ├── auth/
│   │   ├── authSlice.js
│   │   ├── authActions.js
│   │   └── authSelectors.js
│   ├── products/
│   │   ├── productSlice.js
│   │   ├── productActions.js
│   │   └── productSelectors.js
│   └── cart/
│       ├── cartSlice.js
│       ├── cartActions.js
│       └── cartSelectors.js
└── store.js
```

#### Q3. Atomic 패턴이 무엇일까?
- 개념 : UI 컴포넌트를 원자(Atom), 분자(Molecule), 유기체(Organism), 템플릿(Template), 페이지(Page) 등으로 계층화하는 구조
- 예제

```shell
src/
├── components/
│   ├── atoms/
│   │   ├── Button.js
│   │   ├── Input.js
│   │   └── Label.js
│   ├── molecules/
│   │   ├── FormInput.js
│   │   └── NavbarItem.js
│   ├── organisms/
│   │   ├── Header.js
│   │   └── Footer.js
│   ├── templates/
│   │   ├── MainTemplate.js
│   │   └── FormTemplate.js
│   └── pages/
│       ├── HomePage.js
│       └── AboutPage.js
```

#### Q4. Primitive UI가 무엇일까?
- 개념 : 디자인과 개발에서 가장 기본적이고 단순한 형태의 UI 요소
- 예시
  - Button: 다양한 스타일이나 상태를 적용할 수 있는 기본 버튼 컴포넌트
  - Input: 텍스트 입력을 위한 기본 입력 필드 컴포넌트
  - Label: 폼 요소에 레이블을 붙이기 위한 기본 텍스트 컴포넌트
  - Icon: UI에서 사용되는 기본 아이콘 컴포넌트
  - Text: 일반 텍스트를 렌더링하기 위한 기본 컴포넌트

#### Q5. 모노레포가 무엇일까?
- 개념 : 다수의 프로젝트(또는 패키지) 하나의 레포지토리에서 관리하는 방식
- 특징
  - 코드 공유 : 동일한 레포지토리에서 여러 패키지를 관리하기 때문에 공통된 코드나 유틸리티를 쉽게 공유할 수 있다.
  - 일관성 유지 : 모든 프로젝트가 동일한 버전의 종속성을 사용하고 같은 빌드 및 테스트 설정을 가지기 때문에 일관성을 유지하기가 용이하다.

#### Etc에서 배운점
- Next.js 프로젝트에 Atomic 패턴을 적용한다면 `page.tsx`가 pages를, `layout.tsx`가 templates 역할을 하고, `components/`에서 organisms, molecules, atoms 관리할 수 있을 것 같다.
- 애플리케이션 UI 구조를 나눌때 컴포넌트를 만들고, 반복되는 UI 패턴이 발견된다면 공통 폴더에서 관리하면 좋을 것 같다. atoms 같은 경우는 디자인 시스템 확장까지 고려할 수 있도록  만들어놔야겠다.

### 7. React 생태계 흐름 읽기

#### Q1. Storybook은 무엇일까?
- 개념 : 프론트엔드 개발에서 UI 컴포넌트 개발과 테스트를 독립적으로 수행할 수 있도록 도와주는 도구
- 필요성
  - 독립적인 컴포넌트 개발 환경
  - 자동화된 UI 기능 테스트
  - 문서화 및 공유

#### Q2. Jest는 무엇일까?
- 개념 : 프론트엔드에서 테스트를 작성하고 실행하는 데 사용되는 프레임워크
- 필요성
  - 독립적인 함수 개발 환경
  - 자동화된 도메인 기능 테스트

#### Q3. headless UI가 무엇일까?
- 개념 : UI 컴포넌트를 제공할 때 스타일링과 레이아웃을 전혀 포함하지 않는 라이브러리
- 특징
  - 스타일링이 없다(작은 번들 크기).
  - UI가 로직으로부터 분리되어 있다.

#### Q4. Radix UI, Chakra UI가 무엇일까?
- Radix UI
  - 개념 : 리액트 애플리케이션을 위한 저수준의(headless) UI 컴포넌트 라이브러리이다. 이를 통해 개발자가 UI의 동작과 구조를 손쉽게 구현하면서도, 스타일링에 대한 완전한 제어권을 유지할 수 있다.
  - 예제

```react
import { Button } from '@radix-ui/react';

function App() {
  return (
    <Button>
      Click me
    </Button>
  );
}
```

- Chakra UI
  - 개념 : 리액트 애플리케이션을 위한 고수준의 UI 컴포넌트 라이브러리이다. Chakra UI는 사용자 경험과 빠른 개발을 목표로 하며, 미리 스타일링된 컴포넌트를 제공한다. 개발자는 제공된 컴포넌트를 바로 사용할 수 있으며, 필요한 경우 간단히 스타일을 수정하여 자신만의 디자인을 적용할 수 있다.
  - 예제

```react
import { Button } from '@chakra-ui/react';

function App() {
  return (
    <Button colorScheme="teal" size="md">
      Click me
    </Button>
  );
}
```

#### Q5. Presentational and Container Components가 무엇일까?
- Presentational Component
  - 개념 : UI와 관련된 컴포넌트
  - 특징
    - markup과 스타일을 사용한다.
    - 상태를 거의 가지지 않는다. 상태를 가진다면 데이터에 관한것이 아닌 UI 상태에 관한 것이다.

- Container Component
  - 개념 : 데이터와 관련된 컴포넌트
  - 특징
    - markup과 스타일을 사용하지 않는다.
    - 데이터/상태와 상태 조작에 관한 함수만 관리한다.

#### Q6. BFF(Backend For Frontend) 아키텍처가 무엇일까?
  - 개념 : 프론트엔드와 백엔드 간의 효율적인 통신을 위해 설계된 소프트웨어 아키텍처 패턴
  - 특징
    - 맞춤화된 API 제공 : BFF는 특정 프론트엔드 요구 사항에 맞게 API를 제공하는 역할을 한다. 예를 들어, 웹 애플리케이션, 모바일 앱, IoT 기기 등 각 프론트엔드는 서로 다른 데이터를 필요로 할 수 있으며, BFF는 이를 각각의 특성에 맞춰 최적화된 API를 제공할 수 있다.
    - 단순화된 클라이언트 코드 : BFF는 클라이언트가 여러 개의 마이크로서비스를 호출하고 데이터를 조합하는 복잡성을 덜어준다. 클라이언트는 BFF에서 제공하는 API만 호출하면 되므로, 클라이언트 코드가 단순해진다.

#### React 생태계 흐름 읽기에서 배운점
- 프론트엔드에서 중요하게 다루어지는 다양한 개념들을 살펴볼 수 있었다.

## 인사이트
1)리액트에서 코드를 관리하는 효율적인 방법을 알 수 있었습니다. 배운 내용을 고민하고 적용하면서 코드를 작성하고 싶습니다.  2)리액트 생태계의 발전을 보며 앞으로의 공부 방향성을 생각할 수 있었습니다. 다음 스텝으로 인프라, CI/CD를 공부하려고 했는데, 그 이전에 Design Pattern, File Structure, Type Safe, Test Code에 대해 먼저 공부하는 것으로 바뀌었습니다. 앞선 것등이 프론트엔드의 기본에 더 가깝다고 생각하기 때문에 맞는 순서라 생각을 했습니다.

최소 하나 이상의 변화만 가져가도 성공적인 스터디였다고 할 수 있을 겁니다. 단순한 공부를 넘어 변화를 가져올 수 있는 공부가 될 수 있도록 노력해야겠습니다. 다음 스텝은 TypeScript입니다. 화이팅 해보겠습니다 ~!

## 참고자료
- [리액트 클린코드](https://www.udemy.com/course/clean-code-react/?couponCode=KEEPLEARNING){:target="\_blank"}
- [우아한테크코스 스터디](https://github.com/woowacourse-study){:target="\_blank"}

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F6-react-clean-code%2Fimage-2.jpg?alt=media&token=325c64da-4d35-410d-9660-174238fc0a47)