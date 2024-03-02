---
title: Next.js Learn - 2. CSS Styling 번역
date: 2024-01-22 23:15:00 +0900
categories: [Frontend, Nextjs]
tags: [Next.js, Next.js conf, Next.js 14]
image: /v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnextjs-conf%2Fnextjs.png?alt=media&token=09247773-9707-4dd1-b3ca-3fe7f943497a
---

> 본 포스팅은 Next.js Learn의 [CSS Styling](https://nextjs.org/learn/dashboard-app/css-styling){:target="\_blank"} 내용을 번역한 것입니다.
{: .prompt-info }

현재 홈 페이지에는 스타일이 없습니다. Next.js 애플리케이션의 스타일을 지정할 수 있는 다양한 방법을 살펴보겠습니다.

## In this chapter...

앞으로 다룰 주제는 다음과 같습니다.

- 애플리케이션에 글로벌 CSS 파일을 추가하는 방법
- 두 가지 스타일링 방법: Tailwind 및 CSS Modules
- `clsx` 유틸리티 패키지로 클래스 이름을 조건부로 추가하는 방법

## Global styles

`app/ui` 폴더를 살펴보면 `global.css`라는 파일을 볼 수 있습니다. 이 파일을 사용하여 애플리케이션의 모든 경로에 CSS 재설정 규칙, 링크와 같은 HTML 요소에 대한 사이트 전체 스타일 등과 같은 CSS 규칙을 추가할 수 있습니다. <br />
`global.css`는 애플리케이션의 모든 컴포넌트에서 가져올 수 있지만 일반적으로 최상위 컴포넌트에 추가하는 것이 좋습니다. Next.js에서는 이것이 [root layout](https://nextjs.org/docs/app/building-your-application/routing/pages-and-layouts#root-layout-required){:target="\_blank"}입니다(나중에 자세히 설명). <br />
`app/layout.tsx`로 이동하여 `global.css` 파일을 가져와서 애플리케이션에 전역 스타일을 추가하세요.

```react
// /app/layout.tsx

import "@/app/ui/global.css";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

개발 서버가 실행 중인 상태에서 변경 사항을 저장하고 브라우저에서 미리 봅니다. 이제 홈페이지는 다음과 같이 표시됩니다.
![img-description](/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fcss-styling%2Fglobal-styling.png?alt=media&token=471ec630-ddcc-4f15-912f-f0d442d59c22)
_출처 : Next.js Learn_
CSS 규칙을 추가하지 않았는데 스타일은 어디에서 가져온 걸까요? <br />
`global.css` 내부를 살펴보면 `@tailwind` 지시문이 있습니다.

```css
/* /app/ui/global.css */

@tailwind base;
@tailwind components;
@tailwind utilities;
```

## Tailwind

[Tailwind](https://tailwindcss.com/){:target="\_blank"}는 TSX 마크업에서 직접 [utility classes](https://tailwindcss.com/docs/utility-first){:target="\_blank"}를 빠르게 작성할 수 있도록 하여 개발 프로세스의 속도를 높여주는 CSS 프레임워크입니다. <br />
Tailwind는 클래스 이름을 추가하여 요소의 스타일을 지정합니다. 예를 들어 `"text-blue-500"` 클래스를 추가하면 `<h1>` 텍스트가 파란색으로 바뀝니다.

```react
<h1 className="text-blue-500">I'm blue!</h1>
```

CSS 스타일은 전역으로 공유되지만 각 클래스는 각 요소에 개별적으로 적용됩니다. 즉, 요소를 추가하거나 삭제할 때 별도의 스타일시트를 유지하거나 스타일 충돌이 발생하거나 애플리케이션 확장에 따라 CSS 번들의 크기가 커지는 것에 대해 걱정할 필요가 없습니다. <br />
`create-next-app`을 사용하여 새 프로젝트를 시작하면 Next.js에서 Tailwind를 사용할 것인지 묻습니다. `yes`를 선택하면 Next.js가 필요한 패키지를 자동으로 설치하고 애플리케이션에 Tailwind를 구성합니다. `app/page.tsx`를 보면 예제에서 Tailwind 클래스를 사용하고 있음을 알 수 있습니다.

```react
// /app/page.tsx

import AcmeLogo from '@/app/ui/acme-logo';
import { ArrowRightIcon } from '@heroicons/react/24/outline';
import Link from 'next/link';

export default function Page() {
  return (
    // These are Tailwind classes:
    <main className="flex min-h-screen flex-col p-6">
      <div className="flex h-20 shrink-0 items-end rounded-lg bg-blue-500 p-4 md:h-52">
    // ...
  )
}
```

기존 CSS 규칙을 작성하거나 스타일을 JSX와 별도로 유지하는 것을 선호한다면 CSS Modules가 훌륭한 대안이 될 수 있습니다.

## CSS Modules

[CSS 모듈](https://nextjs.org/docs/pages/building-your-application/styling){:target="\_blank"}을 사용하면 고유한 클래스 이름을 자동으로 생성하여 컴포넌트에 CSS의 범위를 지정할 수 있으므로 스타일 충돌에 대해 걱정할 필요가 없습니다. <br />
이 강좌에서는 Tailwind를 계속 사용하겠지만, 잠시 시간을 내어 CSS 모듈을 사용하여 위 퀴즈와 동일한 결과를 얻을 수 있는 방법을 살펴보겠습니다. <br />
`app/ui`에 `home.module.css`라는 새 파일을 만들고 다음 CSS 규칙을 추가합니다.

```css
/* /app/ui/home.module.css */

.shape {
  height: 0;
  width: 0;
  border-bottom: 30px solid black;
  border-left: 20px solid transparent;
  border-right: 20px solid transparent;
}
```

그런 다음 `/app/page.tsx`파일 내에서 스타일을 가져오고 추가한 `<div>`의 Tailwind 클래스 이름을 `styles.shape`로 바꿉니다.

```react
// /app/page.tsx

import styles from '@/app/ui/home.module.css';

//...
<div className="flex flex-col justify-center gap-6 rounded-lg bg-gray-50 px-6 py-10 md:w-2/5 md:px-20">
    <div className={styles.shape}></div>;
// ...
```

변경 내용을 저장하고 브라우저에서 미리 봅니다. 이전과 동일한 모양이 표시될 것입니다. <br />
Tailwind와 CSS 모듈은 Next.js 애플리케이션의 스타일을 지정하는 가장 일반적인 두 가지 방법입니다. 어느 쪽을 사용하든 선호도에 따라 선택할 수 있으며 동일한 애플리케이션에서 두 가지 모두 사용할 수도 있습니다.

## Using the `clsx` library to toggle class names

상태나 다른 조건에 따라 요소의 스타일을 조건부로 지정해야 하는 경우가 있을 수 있습니다. `clsx`는 클래스 이름을 쉽게 전환할 수 있는 라이브러리입니다. 자세한 내용은 [문서](https://github.com/lukeed/clsx){:target="\_blank"}를 참조하는 것이 좋지만 기본적인 사용법은 다음과 같습니다.

- 상태를 받아들이는 `InvoiceStatus` 컴포넌트를 만들고 싶다고 가정해 보겠습니다. 상태는 `'pending'` 또는 `'paid'`일 수 있습니다.
- `'paid'`인 경우 색상을 녹색으로 지정하면 됩니다. `'pending'`인 경우 색상을 회색으로 지정합니다.

다음과 같이 `clsx`를 사용하여 클래스를 조건부로 적용할 수 있습니다.

```react
// /app/ui/invoices/status.tsx

import clsx from 'clsx';

export default function InvoiceStatus({ status }: { status: string }) {
  return (
    <span
      className={clsx(
        'inline-flex items-center rounded-full px-2 py-1 text-sm',
        {
          'bg-gray-100 text-gray-500': status === 'pending',
          'bg-green-500 text-white': status === 'paid',
        },
      )}
    >
    // ...
)}
```

## Other styling solutions

앞서 설명한 접근 방식 외에도 다음과 같은 방법으로 Next.js 애플리케이션의 스타일을 지정할 수 있습니다.

- `.css` 및 `.scss` 파일을 가져올 수 있는 Sass
- [styled-jsx](https://github.com/vercel/styled-jsx){:target="\_blank"}, [styled-components](https://github.com/vercel/next.js/tree/canary/examples/with-styled-components){:target="\_blank"} 및 [emotion](https://github.com/vercel/next.js/tree/canary/examples/with-emotion){:target="\_blank"}과 같은 CSS-in-JS 라이브러리

자세한 내용은 [CSS 문서](https://nextjs.org/docs/app/building-your-application/styling){:target="\_blank"}를 참조하세요.

## Summary

- Global Styling 방법 : `global.css`, `app/layout.tsx`
- Styling 방법 : Tailwind CSS, CSS Modules, Sass, CSS-in-JS
- `clsx` 라이브러리 : 상태나 조건에 따라 요소의 클래스 이름(즉, 스타일)을 조건부로 지정할 수 있다.
