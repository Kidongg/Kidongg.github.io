---
title: Next.js에서 ESLint와 Prettier 설정하기(feat. VScode)
date: 2024-07-09 00:30:00 +0900
categories: [Frontend, Experience]
tags: [Next.js, ESLint, Prettier, VScode]
image: /v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnextjs-conf%2Fnextjs.png?alt=media&token=09247773-9707-4dd1-b3ca-3fe7f943497a
---

## 배경
문득 Next.js에서 ESLint와 Prettier을 설정하는 방법이 궁금해졌습니다. 특히 두 가지가 궁금했는데요. 첫째는 저도 모르게 사용하고 있는 ESLint, Prettier가 무슨 기능을 제공하는지 입니다. 둘째는 ESLint, Prettier 설정을 통해 개발자간의 협업을 하는 예제입니다. 그래서 [공식문서](https://nextjs.org/docs/app/building-your-application/configuring/eslint){:target="\_blank"}를 참고해 공부를 했습니다.

Next.js에서는 통합 [ESLint](https://eslint.org/){:target="\_blank"} 환경을 제공합니다. 프레임워크 규칙에 따라 쉽고 간편하게 설정할 수 있습니다.

ESLint와 Prettier를 한 줄 요약하자면 다음과 같습니다.
- ESLint : 코드 문제 발견 도구
- Prettier : 코드 포맷팅 도구

## 설정 방법

### 1. Next.js 프로젝트 생성
Next.js는 통합 ESLint 환경을 제공합니다. 따라서 프로젝트를 생성하는 시점에 ESLint를 설정할 수 있습니다.

```shell
npx create-next-app@latest [프로젝트명]
```

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnextjs-eslint-and-prettier%2Fimage-1.png?alt=media&token=ee532642-691a-4b9f-8b1c-d3f6b9519cc8)

### 2. ESLint 설정 확인
`package.json`에서 ESLint가 잘 설치되었는지 확인을 해 볼 수 있습니다.

```react
// package.json

"devDependencies": {
	...
  "eslint": "^8",
  "eslint-config-next": "14.2.4",
	...
}
```

`.eslintre.json`에서는 설치된 ESLint 구성 옵션을 확인해볼 수 있습니다. next/core-web-vitals은 [Core Web Vitals rule-set](https://web.dev/articles/vitals?hl=ko){:target="\_blank"}과 함께 Next.js의 기본 ESLint 구성을 포함하는 권장하는 옵션입니다.

```react
// .eslintre.json

{
  "extends": ["next/core-web-vitals"]
}
```

ESLint 구성 옵션을 확인했다면 의도한대로 동작하는지 실행을 해봅니다.

```shell
npm run lint
```

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnextjs-eslint-and-prettier%2Fimage-2.png?alt=media&token=418773dc-8ad9-408e-8daf-e372467fe248)

### 3. Prettier 설치
ESLint에는 코드 포맷팅 규칙이 포함되어 있어 기존 [Prettier](https://prettier.io/){:target="\_blank"} 설정과 충돌할 수 있습니다. 따라서 ESLint와 Prettier가 함께 동작할 수 있도록 [eslint-config-prettier](https://github.com/prettier/eslint-config-prettier){:target="\_blank"}을 설치해야합니다.

```shell
npm install --save-dev eslint-config-prettier
```

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnextjs-eslint-and-prettier%2Fimage-3.png?alt=media&token=169ccd84-cb3f-4fe8-9883-d130beb26935)

`package.json`에서 Prettier가 잘 설치되었는지 확인을 합니다.

```react
// package.json

"devDependencies": {
	...
	"eslint-config-prettier": "^9.1.0",
	...
}
```

ESLint에 Prettier를 사용할 것임을 알려줍니다.

```react
// .eslintre.json

{
  "extends": ["next/core-web-vitals", "prettier"]
}
```

### 4. Prettier 설정
환경 설정은 끝났습니다. 이제 프로젝트 루트에 `.prettierrc` 파일을 만들어 팀에서 정한 규칙을 적용하기만 하면 됩니다.

```react
// .prettierrc

{
  "semi": false,
  "singleQuote": true,
  "trailingComma": "all"
}
```

### 5. VSCode 설정
VScode를 사용하고 있다면 저장을 누르는 시점에 Prettier 규칙을 적용할 수 있습니다. VScode 설정에서 [edit format on Save를 활성화시켜주면 됩니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnextjs-eslint-and-prettier%2Fimage-4.png?alt=media&token=dbbf8789-5933-4ae1-9d18-6a8e5706db8d)

```react
// settings.json

{
	...
  "editor.formatOnSave": true,
  "editor.tabSize": 2,
  "editor.defaultFormatter": "esbenp..-vscode",
  ...
}
```

`settings.json`파일의 `editor.formatOnSave`가 true로 바뀐 것을 확인할 수 있습니다. 탭 사이즈 같은 다른 코드 포맷팅 설정도 이곳에서 변경할 수 있습니다.

## 느낀점
당연한게 아니지만 당연히 누리고 있던 ESLint와 Prettier를 이해할 수 있었던 시간이었습니다. 다음 프로젝트의 ESLint, Prettier 설정을 직접 해보고 싶습니다. 그리고 미래의 팀원과 코드 규칙에 대해서도 논의를 해보고 싶습니다.