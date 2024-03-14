---
title: Next.js Learn - 5. Navigating Between Pages 번역
date: 2024-03-02 12:00:00 +0900
categories: [Frontend, Nextjs]
tags: [Next.js, Next.js conf, Next.js 14]
image: /v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnextjs-conf%2Fnextjs.png?alt=media&token=09247773-9707-4dd1-b3ca-3fe7f943497a
---

> 본 포스팅은 Next.js Learn의 [Navigating Between Pages](https://nextjs.org/learn/dashboard-app/navigating-between-pages){:target="\_blank"} 내용을 번역한 것입니다.
{: .prompt-info }

이전 장에서 대시보드 레이아웃과 페이지를 만들었습니다. 이제 사용자가 대시보드 경로를 탐색할 수 있도록 몇 가지 링크를 추가해 보겠습니다.

## In this chapter...

앞으로 다룰 주제는 다음과 같습니다.

- `next/link` 컴포넌트 사용 방법
- `usePathname()` 훅으로 활성 링크를 표시하는 방법
- Next.js에서 탐색이 작동하는 방식

## Why optimize navigation

페이지 간에 링크하려면 일반적으로 `<a>` HTML 요소를 사용합니다. 현재 사이드바 링크는 `<a>` 요소를 사용하지만 브라우저에서 home, invoices 및 customers 사이를 탐색할 때 어떤 일이 발생하는지 주목하세요. 보셨나요? 각 페이지 탐색에서 전체 페이지가 새로 고쳐집니다!

## The `<Link>` component

Next.js에서 `<Link />` 컴포넌트를 사용하여 애플리케이션의 페이지 간에 링크할 수 있습니다. `<Link>`를 사용하면 자바스크립트로 [클라이언트 사이드 탐색](https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating#how-routing-and-navigation-works){:target="\_blank"}을 수행할 수 있습니다. `<Link />` 컴포넌트를 사용하려면 `/app/ui/dashboard/nav-links.tsx`를 열고 [next/link](https://nextjs.org/docs/app/api-reference/components/link){:target="\_blank"} 에서 링크 컴포넌트를 가져옵니다. 그런 다음 `<a>` 태그를 `<Link>`로 바꿉니다:

```react
// /app/ui/dashboard/nav-links.tsx

import {
  UserGroupIcon,
  HomeIcon,
  DocumentDuplicateIcon,
} from '@heroicons/react/24/outline';
import Link from 'next/link';

// ...

export default function NavLinks() {
  return (
    <>
      {links.map((link) => {
        const LinkIcon = link.icon;
        return (
          <Link
            key={link.name}
            href={link.href}
            className="flex h-[48px] grow items-center justify-center gap-2 rounded-md bg-gray-50 p-3 text-sm font-medium hover:bg-sky-100 hover:text-blue-600 md:flex-none md:justify-start md:p-2 md:px-3"
          >
            <LinkIcon className="w-6" />
            <p className="hidden md:block">{link.name}</p>
          </Link>
        );
      })}
    </>
  );
}
```

보시다시피 링크 구성 요소는 `<a>` 태그를 사용하는 것과 비슷하지만 `<a href="...">` 대신 `<Link href="...">`를 사용합니다. 변경 사항을 저장하고 로컬 호스트에서 작동하는지 확인합니다. 이제 전체 새로 고침 없이 페이지 사이를 탐색할 수 있을 것입니다. 애플리케이션의 일부가 서버에서 렌더링되지만 전체 페이지 새로 고침이 없어 웹 앱처럼 느껴집니다. 왜 그럴까요?

### Automatic code-splitting and prefetching

탐색 환경을 개선하기 위해 Next.js는 경로 세그먼트별로 애플리케이션을 자동으로 코드 분할합니다. 이는 브라우저가 초기 로드 시 모든 애플리케이션 코드를 로드하는 기존 React [SPA](https://developer.mozilla.org/en-US/docs/Glossary/SPA){:target="\_blank"}와는 다릅니다. 경로별로 코드를 분할한다는 것은 페이지가 격리된다는 것을 의미합니다. 특정 페이지에서 오류가 발생해도 나머지 애플리케이션은 계속 작동합니다. <br />

또한 프로덕션 환경에서 `<Link>` 컴포넌트가 브라우저의 뷰포트에 표시될 때마다 Next.js는 백그라운드에서 링크된 경로에 대한 코드를 자동으로 prefetch합니다. 사용자가 링크를 클릭할 때쯤이면 대상 페이지의 코드가 이미 백그라운드에서 로드되어 있으므로 페이지 전환이 거의 즉각적으로 이루어집니다. [내비게이션의 작동 방식](https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating#how-routing-and-navigation-works){:target="\_blank"}에 대해 자세히 알아보세요.

## Pattern: Showing active links

일반적인 UI 패턴은 사용자가 현재 어떤 페이지에 있는지 알려주는 활성 링크를 표시하는 것입니다. 이렇게 하려면 URL에서 사용자의 현재 경로를 가져와야 합니다. Next.js는 경로를 확인하고 이 패턴을 구현하는 데 사용할 수 있는 `usePathname()`이라는 훅을 제공합니다. <br />
[`usePathname()`](https://nextjs.org/docs/app/api-reference/functions/use-pathname){:target="\_blank"}은 훅이므로 `nav-links.tsx`를 클라이언트 컴포넌트로 바꿔야 합니다. 파일 맨 위에 React의 `"use client"` 지시문을 추가한 다음, `next/navigation`에서 `usePathname()`을 가져옵니다.

```react
// /app/ui/dashboard/nav-links.tsx

'use client';

import {
  UserGroupIcon,
  HomeIcon,
  InboxIcon,
} from '@heroicons/react/24/outline';
import Link from 'next/link';
import { usePathname } from 'next/navigation';

// ...
```

다음으로, `<NavLinks />` 컴포넌트 내부의 `pathname`이라는 변수에 경로를 할당합니다.

```react
// /app/ui/dashboard/nav-links.tsx

export default function NavLinks() {
  const pathname = usePathname();
  // ...
}
```

[`CSS styling`](https://kidongg.github.io/posts/css-styling/){:target="\_blank"} 장에서 소개한 clsx 라이브러리를 사용하여 링크가 활성화되어 있을 때 클래스 이름을 조건부로 적용할 수 있습니다. `link.href`가 `pathname`과 일치하면 링크는 파란색 텍스트와 하늘색 배경으로 표시되어야 합니다.

```react
// /app/ui/dashboard/nav-links.tsx

'use client';

import {
  UserGroupIcon,
  HomeIcon,
  DocumentDuplicateIcon,
} from '@heroicons/react/24/outline';
import Link from 'next/link';
import { usePathname } from 'next/navigation';
import clsx from 'clsx';

// ...

export default function NavLinks() {
  const pathname = usePathname();

  return (
    <>
      {links.map((link) => {
        const LinkIcon = link.icon;
        return (
          <Link
            key={link.name}
            href={link.href}
            className={clsx(
              'flex h-[48px] grow items-center justify-center gap-2 rounded-md bg-gray-50 p-3 text-sm font-medium hover:bg-sky-100 hover:text-blue-600 md:flex-none md:justify-start md:p-2 md:px-3',
              {
                'bg-sky-100 text-blue-600': pathname === link.href,
              },
            )}
          >
            <LinkIcon className="w-6" />
            <p className="hidden md:block">{link.name}</p>
          </Link>
        );
      })}
    </>
  );
}
```

저장하고 로컬호스트를 확인합니다. 이제 활성 링크가 파란색으로 강조 표시된 것을 볼 수 있습니다.

## Summary

- Next.js는 라우팅과 탐색에 하이브리드 방식을 사용한다. 서버에서는 애플리케이션 코드가 경로 세그먼트별로 자동으로 코드 분할이 되며, 클라이언트에서는 경로 세그먼트를 프리패치하고 캐시한다. 즉, 경로를 이동할때 브라우저는 페이지를 새로고침하지 않고 변경되는 클라이언트만 다시 렌더링하여 탐색 환경과 성능을 개선한다.
- `<Link>` : HTML `<a>` 요소를 확장하여 경로 간 프리패칭 및 클라이언트 탐색을 제공하는 React 컴포넌트이다.
- `usePathname()` : 현재 URL의 경로명을 읽을 수 있는 클라이언트 컴포넌트 훅이다.
