---
title: Next.js Learn - 3. Optimizing Fonts and Images 번역
date: 2024-02-11 14:30:00 +0900
categories: [Frontend, Next.js Learn]
tags: [Next.js, Next.js conf, Next.js 14]
image: /v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnextjs-conf%2Fnextjs.png?alt=media&token=09247773-9707-4dd1-b3ca-3fe7f943497a
---

> 본 포스팅은 Next.js Learn의 [Optimizing Fonts and Images](https://nextjs.org/learn/dashboard-app/optimizing-fonts-images){:target="\_blank"} 내용을 번역한 것입니다.
{: .prompt-info }

이전 장에서는 Next.js 애플리케이션의 스타일을 지정하는 방법을 배웠습니다. 사용자 정의 글꼴과 이미지를 추가하여 홈페이지 작업을 계속해 보겠습니다.

## In this chapter...
이 장에서 다룰 주제는 다음과 같습니다.

- `next/font`로 사용자 정의 글꼴을 추가하는 방법
- `next/image`로 이미지를 추가하는 방법
- Next.js에서 글꼴과 이미지가 최적화되는 방법

## Why optimize fonts

글꼴은 웹사이트 디자인에서 중요한 역할을 하지만 프로젝트에서 사용자 정의 글꼴을 사용하면 글꼴 파일을 가져와 로드해야 하는 경우 성능에 영향을 줄 수 있습니다.

[Cumulative Layout Shift](https://web.dev/articles/cls?hl=ko){:target="\_blank"}은 Google에서 웹사이트의 성능과 사용자 경험을 평가하는 데 사용하는 지표입니다. 글꼴의 경우 레이아웃 이동은 브라우저가 처음에 대체 글꼴 또는 시스템 글꼴로 텍스트를 렌더링한 다음 로드된 후 사용자 정의 글꼴로 교체할 때 발생합니다. 이 교체로 인해 텍스트 크기, 간격 또는 레이아웃이 변경되어 주변의 요소가 이동할 수 있습니다.
![img-description](/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Foptimizing-fonts-and-images%2Flayout-shift.png?alt=media&token=01d65fe5-5407-4afa-b1f5-c9ee84bd5e21)
_출처 : Next.js Learn_
Next.js는 next/font 모듈을 사용할 때 애플리케이션의 글꼴을 자동으로 최적화합니다. 빌드 시점에 글꼴 파일을 다운로드하여 다른 정적 에셋과 함께 호스팅합니다. 즉, 사용자가 애플리케이션을 방문할 때 성능에 영향을 줄 수 있는 글꼴에 대한 추가 네트워크 요청이 없습니다.

## Adding a primary font 

애플리케이션에 맞춤 Google 글꼴을 추가하여 어떻게 작동하는지 확인해 보겠습니다. `/app/ui` 폴더에 `fonts.ts`라는 새 파일을 만듭니다. 이 파일을 사용하여 애플리케이션 전체에서 사용할 글꼴을 보관할 수 있습니다. `next/font/google` 모듈에서 `Inter` 글꼴을 가져오면 기본 글꼴이 됩니다. 그런 다음 로드할 [subset](https://fonts.google.com/knowledge/glossary/subsetting){:target="\_blank"}을 지정합니다. 이 경우 `'latin'`입니다.
```react
// /app.ui.font.ts

import { Inter } from 'next/font/google';
 
export const inter = Inter({ subsets: ['latin'] });
```
마지막으로 `/app/layout.tsx`의 `<body>` 요소에 글꼴을 추가합니다.
```react
// /app/layout.tsx

import '@/app/ui/global.css';
import { inter } from '@/app/ui/fonts';
 
export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body className={`${inter.className} antialiased`}>{children}</body>
    </html>
  );
}
```
`<body>` 요소에 `Inter`를 추가하면 글꼴이 애플리케이션 전체에 적용됩니다. 여기에는 글꼴을 부드럽게 처리하는 Tailwind [antialiased](https://tailwindcss.com/docs/font-smoothing){:target="\_blank"} 클래스도 추가합니다. 이 클래스를 반드시 사용할 필요는 없지만 멋진 터치를 추가합니다. 브라우저로 이동하여 개발 도구를 열고 본문 요소를 선택합니다. 이제 `Inter` 및 `Inter_Fallback`이 스타일 아래에 적용된 것을 볼 수 있습니다.

## Practice: Adding a secondary font
애플리케이션의 특정 요소에 글꼴을 추가할 수도 있습니다. 이제 여러분 차례입니다. `fonts.ts` 파일에서 `Lusitana`라는 보조 글꼴을 가져와서 `/app/page.tsx` 파일의 `<p>` 요소에 전달합니다. 이전과 같이 하위 집합을 지정하는 것 외에도 글꼴 굵기를 지정해야 합니다. 준비가 완료되면 아래 코드 조각을 확장하여 해결 방법을 확인하세요.

> Hint:
  - 글꼴에 어떤 가중치 옵션을 전달할지 잘 모르겠다면 코드 편집기에서 TypeScript 오류를 확인하세요.
  - [Google Fonts 웹사이트](https://fonts.google.com/){:target="\_blank"}를 방문하여 `Lusitana`를 검색하여 어떤 옵션을 사용할 수 있는지 확인하세요.
  - [adding multiple fonts](https://nextjs.org/docs/app/building-your-application/optimizing/fonts#using-multiple-fonts){:target="\_blank"}에 대한 문서와 [full list of options](https://nextjs.org/docs/app/api-reference/components/font#font-function-arguments){:target="\_blank"}을 참조하세요.
{: .prompt-tip }

- Reveal the solution

```react
// /app/ui/fonts.ts

import { Inter, Lusitana } from 'next/font/google';
 
export const inter = Inter({ subsets: ['latin'] });
 
export const lusitana = Lusitana({
  weight: ['400', '700'],
  subsets: ['latin'],
});
```

```react
// /app/page.tsx

import AcmeLogo from '@/app/ui/acme-logo';
import { ArrowRightIcon } from '@heroicons/react/24/outline';
import Link from 'next/link';
import { lusitana } from '@/app/ui/fonts';
 
export default function Page() {
  return (
    // ...
    <p
      className={`${lusitana.className} text-xl text-gray-800 md:text-3xl md:leading-normal`}
    >
      <strong>Welcome to Acme.</strong> This is the example for the{' '}
      <a href="https://nextjs.org/learn/" className="text-blue-500">
        Next.js Learn Course
      </a>
      , brought to you by Vercel.
    </p>
    // ...
  );
}
```
마지막으로 `<AcmeLogo />` 컴포넌트도 `Lusitana`를 사용합니다. 오류를 방지하기 위해 주석을 달았으므로 이제 주석을 해제할 수 있습니다. 애플리케이션에 두 개의 사용자 정의 글꼴을 추가했습니다. 다음으로 홈페이지에 히어로 이미지를 추가해 보겠습니다.

## Why optimize images

Next.js는 이미지와 같은 정적 에셋을 최상위 폴더인 `/public` 폴더에서 제공할 수 있습니다. `/public` 폴더에 있는 파일은 애플리케이션에서 참조할 수 있습니다. 일반 HTML을 사용하면 다음과 같이 이미지를 추가할 수 있습니다.
```react
<img
  src="/hero.png"
  alt="Screenshots of the dashboard project showing desktop version"
/>
```
하지만 이는 수동으로 해야 한다는 의미입니다.

- 이미지가 다양한 화면 크기에서 반응하는지 확인합니다.
- 다양한 디바이스에 맞는 이미지 크기를 지정합니다.
- 이미지가 로드될 때 레이아웃이 이동하지 않도록 합니다.
- 사용자 뷰포트 외부에 있는 이미지를 레이지 로드합니다.

이미지 최적화는 그 자체로 하나의 전문 분야로 간주될 수 있는 웹 개발의 큰 주제입니다. 이러한 최적화를 수동으로 구현하는 대신 `next/image` 컴포넌트를 사용하여 이미지를 자동으로 최적화할 수 있습니다.

## The `<Image>` component
`<Image>` 컴포넌트는 HTML `<img>` 태그의 확장으로, 자동 이미지 최적화와 같은 기능이 포함되어 있습니다.

- 이미지가 로드될 때 레이아웃이 자동으로 이동하는 것을 방지합니다.
- 뷰포트가 작은 기기에 큰 이미지가 전송되지 않도록 이미지 크기를 조정합니다.
- 기본적으로 이미지 레이지 로딩(이미지가 뷰포트에 들어갈 때 이미지가 로드됨)을 합니다.
- 브라우저에서 [WebP](https://developer.mozilla.org/en-US/docs/Web/Media/Formats/Image_types#webp){:target="\_blank"} 및 [AVIF](https://developer.mozilla.org/en-US/docs/Web/Media/Formats/Image_types#avif_image){:target="\_blank"}와 같은 최신 형식을 지원하는 경우 이미지를 제공합니다.

## Adding the desktop hero image 

`<Image>` 컴포넌트를 사용하겠습니다. `/public` 폴더 안을 살펴보면 `hero-desktop.png`와 `hero-mobile.png`라는 두 개의 이미지가 있는 것을 볼 수 있습니다. 이 두 이미지는 완전히 다르며 사용자의 기기가 데스크톱인지 모바일인지에 따라 표시됩니다. <br />

`/app/page.tsx` 파일에서 [`next/image`](https://nextjs.org/docs/pages/api-reference/components/image){:target="\_blank"}에서 컴포넌트를 가져옵니다. 그런 다음 댓글 아래에 이미지를 추가합니다.
```react
// /app/page.tsx

import AcmeLogo from '@/app/ui/acme-logo';
import { ArrowRightIcon } from '@heroicons/react/24/outline';
import Link from 'next/link';
import { lusitana } from '@/app/ui/fonts';
import Image from 'next/image';
 
export default function Page() {
  return (
    // ...
    <div className="flex items-center justify-center p-6 md:w-3/5 md:px-28 md:py-12">
      {/* Add Hero Images Here */}
      <Image
        src="/hero-desktop.png"
        width={1000}
        height={760}
        className="hidden md:block"
        alt="Screenshots of the dashboard project showing desktop version"
      />
    </div>
    //...
  );
}
```
여기서는 `width`를 `1000`으로, `height`를 `760`픽셀로 설정하고 있습니다. 레이아웃이 바뀌지 않도록 이미지의 `1000`와 `760`를 설정하는 것이 좋으며, 원본 이미지와 동일한 가로 세로 비율이어야 합니다. 또한 모바일 화면의 DOM에서 이미지를 제거하기 위해 `hidden` 클래스와 데스크톱 화면에서 이미지를 표시하기 위해 `md:block` 클래스를 사용할 수 있습니다. <br />
이제 홈페이지는 이렇게 표시되어야 합니다.
![img-description](/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Foptimizing-fonts-and-images%2Fhero.png?alt=media&token=612dcb51-b317-4daf-901a-71b4eeeca207)
_출처 : Next.js Learn_

## Practice: Adding the mobile hero image
이제 여러분의 차례입니다. 방금 추가한 이미지 아래에 `hero-mobile.png`에 대한 또 다른 `<Image>` 컴포넌트를 추가합니다.

- 이미지의 `width`는 `560`, `height`는 `620`픽셀이어야 합니다.
- 모바일 화면에서는 표시되고 데스크톱에서는 숨겨져야 합니다. 

- Reveal the solution

```react
// /app/page.tsx

import AcmeLogo from '@/app/ui/acme-logo';
import { ArrowRightIcon } from '@heroicons/react/24/outline';
import Link from 'next/link';
import { lusitana } from '@/app/ui/fonts';
import Image from 'next/image';
 
export default function Page() {
  return (
    // ...
    <div className="flex items-center justify-center p-6 md:w-3/5 md:px-28 md:py-12">
      {/* Add Hero Images Here */}
      <Image
        src="/hero-desktop.png"
        width={1000}
        height={760}
        className="hidden md:block"
        alt="Screenshots of the dashboard project showing desktop version"
      />
      <Image
        src="/hero-mobile.png"
        width={560}
        height={620}
        className="block md:hidden"
        alt="Screenshot of the dashboard project showing mobile version"
      />
    </div>
    //...
  );
}
```

멋지네요. 이제 홈페이지에 사용자 정의 글꼴과 히어로 이미지가 생겼습니다.

## Recommended reading
원격 이미지 최적화 및 로컬 글꼴 파일 사용 등 이러한 주제에 대해 배울 수 있는 내용이 훨씬 더 많습니다. 글꼴과 이미지에 대해 자세히 알아보고 싶다면 다음을 참조하세요.

- [Image Optimization Docs](https://nextjs.org/docs/app/building-your-application/optimizing/images){:target="\_blank"}
- [Font Optimization Docs](https://nextjs.org/docs/app/building-your-application/optimizing/fonts){:target="\_blank"}
- [Improving Web Performance with Images (MDN)](https://developer.mozilla.org/en-US/docs/Learn/Performance/Multimedia){:target="\_blank"}
- [Web Fonts (MDN)](https://developer.mozilla.org/en-US/docs/Learn/CSS/Styling_text/Web_fonts){:target="\_blank"}

## Summary
- 폰트 : `next/font`를 사용해 사용자 폰트를 적용할 수 있습니다. 전역 폰트는 layout 페이지에서 설정하며, 필요시 지역 폰트를 설정할 수 있습니다.
- 이미지 : `<Image>` 컴포넌트를 사용해 레이아웃 시프트 방지, 이미지 크기 조절, 레이지 로딩, WebP/AVIF 변환을 할 수 있습니다.