---
title: React Docs - React Compiler(experimental) 번역
date: 2024-06-21 22:00:00 +0900
categories: [Translation, React Docs]
tags: [React, React Compiler]
image: https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Freact-compiler%2Freact.png?alt=media&token=537171f4-ba52-4cb3-8178-6c8bfa0508a3
---

> 본 포스팅은 [React의 공식 문서](https://react.dev/learn/react-compiler){:target="\_blank"}의 React Compiler를 번역한 것입니다.
{: .prompt-info }

## Overview
이 페이지에서는 새로운 실험 단계의 React 컴파일러에 대한 소개와 성공적인 사용 방법을 알려드립니다. 이 문서는 아직 진행 중인 작업입니다. 더 많은 문서는 [React Compiler Working Group repo](https://github.com/reactwg/react-compiler/discussions){:target="\_blank"}에서 확인할 수 있으며, 안정화되면 이 문서로 업스트림될 예정입니다.

> Note  <br /> React 컴파일러는 커뮤니티의 초기 피드백을 얻기 위해 오픈소스로 공개한 새로운 실험적인 컴파일러입니다. 아직 거친 부분이 있으며 프로덕션용으로 완전히 준비되지 않았습니다. React 컴파일러를 사용하려면 React 19 Beta가 필요합니다.
{: .prompt-tip }

컴파일러는 빌드 시간에만 사용할 수 있는 도구로 React 앱을 자동으로 최적화합니다. 일반 자바스크립트와 함께 동작하며 [React 규칙](https://react.dev/reference/rules){:target="\_blank"}을 이해하므로 코드를 다시 작성할 필요가 없습니다.

컴파일러에는 에디터에서 바로 컴파일러의 분석 결과를 표시하는 [eslint 플러그인](https://react.dev/learn/react-compiler#installing-eslint-plugin-react-compiler){:target="\_blank"}도 표함되어 있습니다. 이 플러그인은 컴파일러와 독립적으로 실행되며 앱에서 컴파일러를 사용하지 않더라도 사용할 수 있습니다. 모든 React 개발자는 코드베이스의 품질을 개선하기 위해 이 eslint 플러그인을 사용할 것을 권장합니다.

### What does the compiler do?
컴파일러는 일반 자바스크립트 의미론과 [React 규칙](https://react.dev/reference/rules){:target="\_blank"}에 대한 이해를 통해 코드를 심층적으로 이해합니다. 이를 통해 코드에 자동 최적화를 추가할 수 있습니다.

`useMemo`, `useCallback`, `React.memo`를 통한 수동 메모이제이션에 익술할 것입니다. 코드가 [React 규칙](https://react.dev/reference/rules){:target="\_blank"}을 따르는 경우 컴파일러가 자동이로 이 작업을 수행할 수 있습니다. 규칙 위반을 감지하면 해당 컴포넌트나 후크를 자동으로 건너뛰고 다른 코드를 안전하게 계속 컴파일합니다.

코드베이스가 이미 잘 메모이제이션되어 있다면 컴파일러로 큰 성능 향상을 기대하지 않을 수도 있습니다. 그러나 실제로는 성능 문제를 일으키는 정확한 종속성을 손으로 직접 메모하는 것은 까다로운 일입니다.

### Should I try out the compiler?
컴파일러는 아직 실험 단게이며 거친 부분이 많다는 점을 유의하세요. Meta와 같은 회사에서 프로덕션에 사용되었지만 컴파일러를 앱의 프로덕션에 배포하는 것은 코드베이스의 상태와 [React 규칙](https://react.dev/reference/rules){:target="\_blank"}을 얼마나 잘 준수했는지에 따라 달라질 수 있습니다.

지금 컴파일러를 서둘러 사용할 필요는 없습니다. 안정적인 릴리스가 나올 때까지 기다렸다가 도입해도 괜찮습니다. 하지만 컴파일러를 개선하는 데 도움이 되는 [피드백을 제공](https://react.dev/learn/react-compiler#reporting-issues){:target="\_blank"}할 수 있도록 앱에서 소규모 실험을 통해 사용해 보시면 감사하겠습니다. 

## Getting Started
이 문서 외에도 컴파일러에 대한 추가 정보와 토론은 [React Compiler Working Group repo](https://github.com/reactwg/react-compiler/discussions){:target="\_blank"}에서 확인하실 것을 권장합니다.

### Checking compatibility
컴파일러를 설치하기 전에 먼저 코드베이스가 호환되는지 확인할 수 있습니다.

```shell
npx react-compiler-healthcheck@latest
```

이 스크립트는
- Check how many components can be successfully optimized : 많을수록 좋습니다.
- Check for `<StrictMode>` usage : 이 기능을 활성화하고 준수하면 [React 규칙](https://react.dev/reference/rules){:target="\_blank"}이 준수될 가능성이 높아집니다.
- Check for incompatible library usage : 컴파일러와 호환되지 않는 알려진 라이브러리 확인

예를 들어

```shell
Successfully compiled 8 out of 9 components.
StrictMode usage not found.
Found no usage of incompatible libraries.
```

### Installing eslint-plugin-react-compiler
React 컴파일러는 eslint 플러그인도 지원합니다. eslint 플러그인은 컴파일러와 독립적으로 사용할 수 있으므로 컴파일러를 사용하지 않더라도 eslint 플러그인을 사용할 수 있습니다.

```shell
npm install eslint-plugin-react-compiler
```

그런 다음 eslint config에 추가합니다.

```react

// /.eslintrc
module.exports = {
  plugins: [
    'eslint-plugin-react-compiler',
  ],
  rules: {
    'react-compiler/react-compiler': "error",
  },
}
```

eslint 플러그인은 에디터에서 React의 규칙을 위반하는 모든 것을 표시합니다. 이렇게 표시되면 컴파일러가 해당 컴포넌트나 훅 최적화를 건너뛰었다는 뜻입니다. 이는 완전히 괜찮으며 컴파일러는 코드베이스의 다른 컴포넌트를 복구하고 계속 최적화할 수 있습니다.

모든 eslint 위반을 바로 수정할 필요는 없습니다. 최적화되는 컴포넌트와 후크의 양을 늘리기 위해 원하는 속도로 해결할 수 있지만 컴파일러를 사용하기 전에 모든 것을 수정할 필요는 없습니다.

### Rolling out the compiler to your codebase

#### Existing projects
컴파일러는 React의 규칙을 따르는 함수형 컴포넌트와 훅을 컴파일하도록 설계되었습니다. 또한 컴파일러는 해당 컴포넌트나 훅을 건너뛰는(건너뛰는) 방식으로 규칙을 위반하는 코드를 처리할 수 있습니다. 그러나 자바스크립트의 유연한 특성으로 인해 컴파일러는 가능한 모든 위반을 포착할 수 없으며, 컴파일러가 실수로 정의되지 않은 동작을 유발할 수 있는 React 규칙을 위반하는 컴포넌트/후크를 컴파일할 수 있는 잘못된 네거티브 컴파일을 할 수 있습니다.

따라서 기존 프로젝트에 컴파일러를 성공적으로 적용하려면 먼저 제품 코드의 작은 디렉토리에서 컴파일러를 실행하는 것이 좋습니다. 컴파일러가 특정 디렉토리 집합에서만 실행되도록 구성하면 됩니다.

```react
// /babel.config.js

const ReactCompilerConfig = {
  sources: (filename) => {
    return filename.indexOf('src/path/to/dir') !== -1;
  },
};
```

드문 경우지만 `compilationMode: "annotation"` 옵션을 사용하여 컴파일러를 `"opt-in"` 모드로 실행하도록 설정할 수도 있습니다. 이렇게 하면 컴파일러가 `"use memo"` 지시어로 주석이 달린 컴포넌트와 훅만 컴파일하도록 합니다. `annotation` 모드는 얼리 어답터를 돕기 위한 임시 모드이며 `"use memo"` 지시문을 장기적으로 사용할 의도는 없다는 점에 유의하세요.

```react
// /babel.config.js

const ReactCompilerConfig = {
  compilationMode: "annotation",
};
```

```react
// src/app.jsx

export default function App() {
  "use memo";
  // ...
}
```

컴파일러 배포에 자심감이 생기면 다른 디렉토리로 적용 범위를 확장하고 전체 앱에 천천히 배포할 수 있습니다.

#### New projects
새 프로젝트를 시작하는 경우 전체 코드베이스에서 컴파일러를 활성화할 수 있으며, 이는 기본 동작입니다.

## Usage

### Usage with Babel
```shell
npm install babel-plugin-react-compiler
```

컴파일러에는 빌드 파이프라인에서 컴파일러를 실행하는 데 사용할 수 있는 Babel 플러그인이 포함되어 있습니다. 설치 후 Babel 설정에 추가하세요. 파이프라인에서 컴파일러를 먼저 실행하는 것이 중요합니다.

```react
// /babel.config.js

const ReactCompilerConfig = { /* ... */ };

module.exports = function () {
  return {
    plugins: [
      ['babel-plugin-react-compiler', ReactCompilerConfig], // must run first!
      // ...
    ],
  };
};
```

컴파일러는 사운드 분석을 위 입력 소스 정보를 필요로 하므로 다른 Babel 플러그인 보다 먼저 `babel-plugin-react-compiler`를 실행해야 합니다.

### Usage with Next.js
Next.js에는 React 컴파일러를 활성화하기 위한 실험적인 구성이 있습니다. 이 설정은 자동으로 Babel이 `babel-plugin-react-compiler`로 설정되도록 합니다.

- React 19 릴리스 후보를 사용하는 Next.js 카나리아를 설치합니다.
- `babel-plugin-react-compiler`를 설치합니다.

```shell
npm install next@canary babel-plugin-react-compiler
```

그런 다음 `next.config.js`에서 실험적인 옵션을 구성합니다.

```react
// next.config.js

/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    reactCompiler: true,
  },
};

module.exports = nextConfig;
```

실험적 옵션을 사용하면 React 컴파일러가 지원됩니다.
- App Router
- Pages Router
- Webpack (default)
- Turbopack (opt-in through `-turbo`)
