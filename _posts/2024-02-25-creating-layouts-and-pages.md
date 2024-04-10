---
title: Next.js Learn - 4. Creating Layouts and Pages 번역
date: 2024-02-25 17:15:00 +0900
categories: [Frontend, Nextjs Learn]
tags: [Next.js, Next.js conf, Next.js 14]
image: /v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnextjs-conf%2Fnextjs.png?alt=media&token=09247773-9707-4dd1-b3ca-3fe7f943497a
---

> 본 포스팅은 Next.js Learn의 [Creating Layouts and Pages](https://nextjs.org/learn/dashboard-app/creating-layouts-and-pages){:target="\_blank"} 내용을 번역한 것입니다.
{: .prompt-info }

지금까지 애플리케이션에는 홈페이지만 있습니다. 레이아웃과 페이지로 더 많은 경로를 만드는 방법을 알아보겠습니다.

## In this chapter...

앞으로 다룰 주제는 다음과 같습니다.

- 파일 시스템 라우팅을 사용하여 `dashboard` 경로를 만듭니다.
- Route segement를 만들 때 폴더와 파일의 역할을 이해합니다.
- 여러 대시보드 페이지 간에 공유할 수 있는 중첩된 레이아웃을 만듭니다.
- Colocation, Partial rendering 및 Root layout이 무엇인지 이해합니다.

## Nested routing

Next.js는 폴더가 중첩 경로를 만드는데 사용되는 파일 시스템 라우팅을 사용합니다. 각 폴더는 URL segement에 매핑되는 route segement를 나타냅니다.
![img-description](/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fcreating-layouts-and-pages%2Fnested-routing-1.png?alt=media&token=96a8d0d9-4f85-4aec-80f9-8ea09819ff69)
_출처 : Next.js Learn_
`layout.tsx` 및 `page.tsx` 파일을 사용하여 각 경로에 대해 별도의 UI를 만들 수 있습니다. `page.tsx`는 React 컴포넌트를 내보내는 특별한 Next.js 파일로, 경로에 접근하기 위해 필요합니다. 애플리케이션에 이미 page 파일이 있습니다. `/app/page.tsx`인 `/`와 연결된 홈페이지 입니다. 중첩된 경로를 만들려면 폴더를 서로 중첩하고 그 안에 `page.tsx` 파일을 추가하면 됩니다. 예를 들어
![img-description](/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fcreating-layouts-and-pages%2Fnested-routing-2.png?alt=media&token=831b2696-64e3-42dc-adc9-653c052f4fbd)
_출처 : Next.js Learn_
`/app/dashboard/page.tsx`는 `/dashboard` 경로와 연결됩니다. 페이지를 만들어서 어떻게 작동하는지 확인해 보겠습니다.

## Creating the dashboard page

`/app` 내에 `dashboard`라는 새 폴더를 만듭니다. 그런 다음 대시보드 폴더 안에 다음 내용으로 새 `page.tsx` 파일을 만듭니다.

```react
// /app/dashboard/page.tsx

export default function Page() {
  return <p>Dashboard Page</p>;
}
```

이제 개발 서버가 실행 중인지 확인하고 [http://localhost:3000/dashboard](http://localhost:3000/dashboard){:target="\_blank"}를 방문합니다. "Dashboard Page" 텍스트가 표시되어야 합니다.

## Practice : Creating the dashboard pages

더 많은 경로를 만드는 연습을 해보겠습니다. 대시보드에서 페이지를 두 개 더 만듭니다.

1. Customers Page : 고객 페이지는 [http://localhost:3000/dashboard/customers](http://localhost:3000/dashboard/customers){:target="\_blank"}에서 액세스할 수 있어야 합니다. 지금은 `<p>Customer Page</p>`요소를 반환해야 합니다.
2. Invoices Page : 인보이스 페이지는 [http://localhost:3000/dashboard/invoices](http://localhost:3000/dashboard/invoices){:target="\_blank"}에서 액세스할 수 있어야 합니다. 지금은 `<p>Invoices Page</p>`요소도 반환해야 합니다.

시간을 들여 이 연습 문제를 해결하고 준비가 되면 아래의 해결책을 찾아보세요.

- Reveal the solution - 폴더 구조는 다음과 같아야 합니다.
  ![img-description](/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fcreating-layouts-and-pages%2Fpractice.png?alt=media&token=0aebdbd4-c62d-48d8-9ad4-d91f77fe08ce)
  _출처 : Next.js Learn_ - Customers Page

          ```react
                  // /app.dashboard/customers/page.tsx

          export default function Page() {
          return <p>Customers Page</p>;
          }
          ```

      - Invoices Page

          ```react
                  // /app.dashboard/customers/page.tsx

          export default function Page() {
          return <p>Customers Page</p>;
          }
          ```

## Creating the dashboard layout

대시보드에는 여러 페이지에서 공유되는 사이드바 기능이 있습니다. Next.js에서는 특별한 `layout.tsx` 파일을 사용하여 여러 페이지 간에 공유되는 UI를 만들 수 있습니다. 대시보드 페이지의 레이아웃을 만들어 봅시다. <br />
`/dashboard` 폴더에 `layout.tsx`라는 새 파일을 추가하고 다음 코드를 붙여넣습니다.

```react
// /app/dashboard/layout.tsx

import SideNav from '@/app/ui/dashboard/sidenav';

export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <div className="flex h-screen flex-col md:flex-row md:overflow-hidden">
      <div className="w-full flex-none md:w-64">
        <SideNav />
      </div>
      <div className="flex-grow p-6 md:overflow-y-auto md:p-12">{children}</div>
    </div>
  );
}
```

이 코드에서 몇 가지 일이 일어나고 있으므로 세분화해 보겠습니다. 먼저 `<SideNav />` 컴포넌트를 레이아웃으로 가져오고 있습니다. 이 파일로 가져오는 모드 컴포넌트는 레이아웃의 일부가 됩니다. `<Layout />` 컴포넌트는 `children` 프로퍼티를 받습니다. 이 자식은 페이지 또는 다른 레이아웃일 수 있습니다. 이 경우 `/dashboard` 내의 페이지는 다음과 같이 `<Layout />` 안에 자동으로 중첩됩니다.

![img-description](/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fcreating-layouts-and-pages%2Fcreating-layout-1.png?alt=media&token=113a2a2f-db95-4bab-af38-075b4e47e857)
_출처 : Next.js Learn_

변경 사항을 저장하고 로컬호스트를 확인하여 모든 것이 올바르게 작동하는지 확인합니다. 다음과 같은 내용이 표시되어야 합니다.

![img-description](/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fcreating-layouts-and-pages%2Fcreating-layout-2.png?alt=media&token=ba61781a-3a16-4930-af1e-0502c030204d)
_출처 : Next.js Learn_

Next.js에서 레이아웃을 사용하면 탐색 시 페이지 컴포넌트만 업데이트되고 레이아웃은 다시 렌더링되지 않는다는 이점이 있습니다. 이를 [Partial rendering](https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating#4-partial-rendering){:target="\_blank"}이라고 합니다.

![img-description](/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fcreating-layouts-and-pages%2Fcreating-layout-3.png?alt=media&token=d6753e36-3ccb-4799-9ac7-0613c9753110)
_출처 : Next.js Learn_

## Root layout

챕터 3에서 `Inter` 글꼴을 다른 레이아웃으로 가져왔습니다. `/app/layout.tsx`를 복기하자면

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

이를 [Root layout](https://nextjs.org/docs/app/building-your-application/routing/pages-and-layouts#root-layout-required){:target="\_blank"}이라고 하며 필수입니다. Root layout에 추가하는 모든 UI는 애플리케이션의 모든 페이지에서 공유됩니다. Root layout을 사용하여 `<html>` 및 `<body>` 태그를 수정하고 metadata를 추가할 수 있습니다(metadata에 대해서는 [이후 장](https://nextjs.org/learn/dashboard-app/adding-metadata){:target="\_blank"}에서 자세히 살펴보겠습니다). <br />
방금 만든 새 레이아웃(`/app/dashboard/layout.tsx`)은 대시보드 페이지에 고유한 레이아웃이므로 위의 루트 레이아웃에 UI를 추가할 필요가 없습니다.

## Summary

- Next.js는 파일 시스템 라우팅을 사용한다.
- `page.tsx` : 페이지 마다 독립된 UI를 그릴 수 있다. 탐색시 렌더링 된다.
- `layout.tsx` : 라이팅 경로마다 공통된 UI를 그릴 수 있다. 탐색시 렌더링 되지 않는다. 
