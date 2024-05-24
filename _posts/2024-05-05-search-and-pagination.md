---
title: Next.js Learn - 11. Adding Search and Pagination 번역
date: 2024-05-05 12:05:00 +0900
categories: [Frontend, Next.js Learn]
tags: [Next.js, Next.js conf, Next.js 14]
image: /v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnextjs-conf%2Fnextjs.png?alt=media&token=09247773-9707-4dd1-b3ca-3fe7f943497a
---

> 본 포스팅은 Next.js Learn의 [Adding Search and Pagination](https://nextjs.org/learn/dashboard-app/adding-search-and-pagination){:target="\_blank"} 내용을 번역한 것입니다.
{: .prompt-info }

이전 장에서는 스트리밍을 통해 대시보드의 초기 로딩 성능을 개선했습니다. 이제 `/invoices` 페이지로 이동하여 Search 및 Pagination을 추가하는 방법을 알아보겠습니다.

## In this chapter
앞으로 다룰 주제는 다음과 같습니다.
- Next.js API인 `searchParams`, `usePathname` 및 `useRouter`를 사용하는 방법
- URL 검색 매개변수를 사용하여 Search 및 Pagination을 구현하는 방법

## Starting code
`/dashboard/invoices/page.tsx` 파일에 다음 코드를 붙여넣습니다.

```react
// /app/dashboard/invoices/page.tsx

import Pagination from '@/app/ui/invoices/pagination';
import Search from '@/app/ui/search';
import Table from '@/app/ui/invoices/table';
import { CreateInvoice } from '@/app/ui/invoices/buttons';
import { lusitana } from '@/app/ui/fonts';
import { InvoicesTableSkeleton } from '@/app/ui/skeletons';
import { Suspense } from 'react';
 
export default async function Page() {
  return (
    <div className="w-full">
      <div className="flex w-full items-center justify-between">
        <h1 className={`${lusitana.className} text-2xl`}>Invoices</h1>
      </div>
      <div className="mt-4 flex items-center justify-between gap-2 md:mt-8">
        <Search placeholder="Search invoices..." />
        <CreateInvoice />
      </div>
      {/*  <Suspense key={query + currentPage} fallback={<InvoicesTableSkeleton />}>
        <Table query={query} currentPage={currentPage} />
      </Suspense> */}
      <div className="mt-5 flex w-full justify-center">
        {/* <Pagination totalPages={totalPages} /> */}
      </div>
    </div>
  );
}
```

페이지와 요소에 익숙해지는데 시간을 사용하세요.
- `<Search />`는 사용자가 특정 인보이스를 검색할 수 있도록 합니다.
- `<Pagination />`은 사용자가 인보이스 페이지를 탐색할 수 있도록 합니다.
- `<Table />`은 인보이스를 표시합니다.

Search 기능은 클라이언트와 서버에 걸쳐 제공됩니다. 사용자가 클라이언트에서 인보이스를 검색하면 URL 매개변수가 업데이트되고 서버에서 데이터를 가져온 다음 새 데이터가 포함된 테이블이 서버에서 다시 렌더링됩니다.

## Why use URL search params?
위에서 언급했듯이 URL 검색 매개변수를 사용하여 상태를 관리하게 됩니다. 클라이언트 측 상태를 사용하는 데 익숙하다면 이 패턴이 생소할 수 있습니다. <br />

URL 매개변수로 Search를 구현하면 몇 가지 이점이 있습니다.
- **Bookmarkable and Shareable URLs** : 검색 쿼리 및 필터를 포함한 애플리케이션의 현재 상태를 북마크에 추가하여 나중에 참조하거나 공유할 수 있습니다.
- **Server-Side Rendering and Initial Load** : URL 매개변수를 서버에서 직접 사용하여 초기 상태를 렌더링 할 수 있으므로 서버 렌더링을 쉽게 처리할 수 있습니다.
- **Analytics and Tracking** : 클라이언트 로직 없이도 사용자 행동을 쉽게 추적 할 수 있습니다.

## Adding the search functionality
다음은 Search 기능을 구현하는데 사용할 Next.js 클라이언트 Hooks입니다.
- `useSearchParams` : 현재 URL 매개변수에 액세스할 수 있습니다. 예를 들어 `/dashboard/invoices?page=1&query=pending`에 대한 검색 매개변수는 `{page: '1', query: 'pending'}`입니다.
- `usePathname` : 현재 URL 경로의 이름을 읽을 수 있습니다. 예를 들어 `/dashboard/invoices` 경로의 경우, 경로명은 `'/dashboard/invoices'`를 반환합니다.
- `useRouter` : 클라이언트 구성 요소 내에서 경로 간 탐색을 활성화합니다. [여러 가지 방법](https://nextjs.org/docs/app/api-reference/functions/use-router#userouter){:target="\_blank"}을 사용할 수 있습니다.

다음은 구현 단계에 대한 간략한 개요입니다.
1. Capture the user's input
2. Update the URL with the search params
3. Keep the URL in sync with the input field
4. Update the table to reflect the search query

### 1. Capture the user's input
`<Search>` 컴포넌트(`/app/ui/search.tsx`)로 이동하면 확인할 수 있습니다.
- `"use client"` : 클라이언트 컴포넌트이며, 이벤트 리스너와 훅을 사용할 수 있습니다.
- `<input>` : 검색 입력입니다.

`handleSearch` 함수를 만들고, `<input>` 요소에 `onChange` 리스너를 추가합니다. 입력값이 변경될때마다 `onChange`가 `handleSearch` 함수를 호출합니다.

```react
// /app/ui/search.tsx

'use client';
 
import { MagnifyingGlassIcon } from '@heroicons/react/24/outline';
 
export default function Search({ placeholder }: { placeholder: string }) {
  function handleSearch(term: string) {
    console.log(term);
  }
 
  return (
    <div className="relative flex flex-1 flex-shrink-0">
      <label htmlFor="search" className="sr-only">
        Search
      </label>
      <input
        className="peer block w-full rounded-md border border-gray-200 py-[9px] pl-10 text-sm outline-2 placeholder:text-gray-500"
        placeholder={placeholder}
        onChange={(e) => {
          handleSearch(e.target.value);
        }}
      />
      <MagnifyingGlassIcon className="absolute left-3 top-1/2 h-[18px] w-[18px] -translate-y-1/2 text-gray-500 peer-focus:text-gray-900" />
    </div>
  );
}
```

개발자 도구에서 콘솔을 열어 제대로 작동하는지 테스트한 다음 검색 필드에 입력합니다. 검색어가 콘솔에 기록된 것을 볼 수 있을 것입니다. 잘됐네요! 사용자의 검색 입력을 캡처하고 있습니다. 이제 검색어로 URL을 업데이트해야 합니다.

### 2. Update the URL with the search params
`next/navigation`에서 `useSearchParams` 훅을 가져와서 변수에 할당합니다.

```react
// /app/ui/search.tsx

'use client';
 
import { MagnifyingGlassIcon } from '@heroicons/react/24/outline';
import { useSearchParams } from 'next/navigation';
 
export default function Search() {
  const searchParams = useSearchParams();
 
  function handleSearch(term: string) {
    console.log(term);
  }
  // ...
}
```

`handleSearch` 내에서 새 `searchParams` 변수를 사용하여 새 [`URLSearchParams`](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams){:target="\_blank"} 인스턴스를 만듭니다.

```react
// /app/ui/search.tsx

'use client';
 
import { MagnifyingGlassIcon } from '@heroicons/react/24/outline';
import { useSearchParams } from 'next/navigation';
 
export default function Search() {
  const searchParams = useSearchParams();
 
  function handleSearch(term: string) {
    const params = new URLSearchParams(searchParams);
  }
  // ...
}
```

`URLSearchParams`는 URL 쿼리 매개변수를 조작하기 위한 유틸리티 메서드를 제공하는 웹 API입니다. 복잡한 문자열 리터럴을 만드는 대신 이 메서드를 사용하여 `?page=1&query=a`와 같은 파라미터 문자열을 가져올 수 있습니다. 그런 다음 사용자의 입력에 따라 params 문자열을 `set`합니다. 입력이 비어 있으면 `delete`합니다.

```react
// /app/ui/search.tsx

'use client';
 
import { MagnifyingGlassIcon } from '@heroicons/react/24/outline';
import { useSearchParams } from 'next/navigation';
 
export default function Search() {
  const searchParams = useSearchParams();
 
  function handleSearch(term: string) {
    const params = new URLSearchParams(searchParams);
    if (term) {
      params.set('query', term);
    } else {
      params.delete('query');
    }
  }
  // ...
}
```

이제 쿼리 문자열이 생겼습니다. Next.js의 `useRouter` 및 `usePathname` 훅을 사용하여 URL을 업데이트할 수 있습니다. `next/navigation`에서 `useRouter`와 `usePathname`을 가져온 다음 `handleSearch` 내부의 `useRouter()` 메서드에서 `replace`를 사용합니다.

```react
// /app/ui/search.tsx

'use client';
 
import { MagnifyingGlassIcon } from '@heroicons/react/24/outline';
import { useSearchParams, usePathname, useRouter } from 'next/navigation';
 
export default function Search() {
  const searchParams = useSearchParams();
  const pathname = usePathname();
  const { replace } = useRouter();
 
  function handleSearch(term: string) {
    const params = new URLSearchParams(searchParams);
    if (term) {
      params.set('query', term);
    } else {
      params.delete('query');
    }
    replace(`${pathname}?${params.toString()}`);
  }
}
```

현재 상황을 설명하면 다음과 같습니다.
- `${pathname}`은 현재 경로는 이 경우 `"/dashboard/invoices"`입니다.
- `params.toString()`이 사용자 입력을 URL 친화적인 타입으로 변환합니다.
- `replace(${pathname}?${params.toString()})`는 사용자의 검색 데이터로 URL을 업데이트합니다. 예를 들어, 사용자가 "Lee"를 검색한 경우 `/dashboard/invoices?query=lee`가 됩니다.
- [페이지 사이를 탐색하는 장](https://nextjs.org/learn/dashboard-app/navigating-between-pages){:target="\_blank"}에서 배운 Next.js의 클라이언트 측 탐색 기능 덕분에 페이지를 다시 로드하지 않고도 URL이 업데이트됩니다.

### 3. Keeping the URL and input in sync
입력 필드가 URL과 동기화되고 공유할 때 입력 필드가 채워지도록 하려면 `searchParams`에서 `defaultValue`를 읽어서 입력에 전달하면 됩니다.

```react
// /app/ui/search.tsx

<input
  className="peer block w-full rounded-md border border-gray-200 py-[9px] pl-10 text-sm outline-2 placeholder:text-gray-500"
  placeholder={placeholder}
  onChange={(e) => {
    handleSearch(e.target.value);
  }}
  defaultValue={searchParams.get('query')?.toString()}
/>
```

> `defaultValue` vs `value` / Controlled vs Uncontrolled  <br />  State를 사용하여 입력의 값을 관리하는 경우, `value` 속성을 사용하여 제어되는 컴포넌트로 만들 수 있습니다. 이는 React가 입력의 상태를 관리한다는 뜻입니다. 하지만 State를 사용하지 않는다면 `defaultValue`를 사용할 수 있습니다. 이는 네이티브 입력이 자체 상태를 관리한다는 뜻입니다. 상태 대신 URL에 검색 쿼리를 저장하기 때문에 괜찮습니다.
{: .prompt-tip }

### 4. Updating the table
마지막으로 검색 쿼리를 반영하도록 테이블 컴포넌트를 업데이트해야 합니다. 인보이스 페이지로 이동합니다. Page 컴포넌트는 [`searchParams`라는 프로퍼티를 허용](https://nextjs.org/docs/app/api-reference/file-conventions/page){:target="\_blank"}하므로 현재 URL 매개변수를 `<Table />` 구성 요소에 전달할 수 있습니다.

```react
// /app/dashboard/invoices/page.tsx

import Pagination from '@/app/ui/invoices/pagination';
import Search from '@/app/ui/search';
import Table from '@/app/ui/invoices/table';
import { CreateInvoice } from '@/app/ui/invoices/buttons';
import { lusitana } from '@/app/ui/fonts';
import { Suspense } from 'react';
import { InvoicesTableSkeleton } from '@/app/ui/skeletons';
 
export default async function Page({
  searchParams,
}: {
  searchParams?: {
    query?: string;
    page?: string;
  };
}) {
  const query = searchParams?.query || '';
  const currentPage = Number(searchParams?.page) || 1;
 
  return (
    <div className="w-full">
      <div className="flex w-full items-center justify-between">
        <h1 className={`${lusitana.className} text-2xl`}>Invoices</h1>
      </div>
      <div className="mt-4 flex items-center justify-between gap-2 md:mt-8">
        <Search placeholder="Search invoices..." />
        <CreateInvoice />
      </div>
      <Suspense key={query + currentPage} fallback={<InvoicesTableSkeleton />}>
        <Table query={query} currentPage={currentPage} />
      </Suspense>
      <div className="mt-5 flex w-full justify-center">
        {/* <Pagination totalPages={totalPages} /> */}
      </div>
    </div>
  );
}
```

`<Table />` 컴포넌트로 이동하면 `query`와 `currentPage`라는 두 개의 프로퍼티가 쿼리와 일치하는 인보이스를 반환하는 `fetchFilteredInvoices()` 함수에 전달되는 것을 볼 수 있습니다.

```react
// /app/ui/invoices/table.tsx

// ...
export default async function InvoicesTable({
  query,
  currentPage,
}: {
  query: string;
  currentPage: number;
}) {
  const invoices = await fetchFilteredInvoices(query, currentPage);
  // ...
}
```

이러한 변경 사항을 적용했으면 이제 테스트해보세요. 용어를 검색하면 URL이 업데이트되어 서버에 새 요청이 전송되고 서버에서 데이터를 가져와서 쿼리와 일치하는 인보이스가 반환됩니다.

> 언제 `useSearchParams()` 훅을 사용하고, 언제 `searchParams` 프로퍼티를 사용하나요?  <br />  검색 매개변수를 추출하는 데 두 가지 다른 방법을 사용했다는 것을 눈치채셨을 것입니다. 어느 쪽을 사용할지는 클라이언트에서 작업하는지 서버에서 작업하는지에 따라 달라집니다. 일반적으로 클라이언트에서 매개변수를 읽으려면 서버로 돌아갈 필요가 없으므로 `useSearchParams()` 훅을 사용합니다.
- `<Search />`는 클라이언트 컴포넌트이므로 클라이언트에서 파라미터에 액세스하기 위해 `useSearchParams()`훅을 사용했습니다.
- `<Table />`은 자체 데이터를 가져오는 서버 컴포넌트이므로 페이지에서 컴포넌트로 `searchParams` 프로퍼티를 
전달할 수 있습니다.
{: .prompt-tip }

## Best practice: Debouncing
축하합니다. Next.js로 Search를 구현했습니다! 하지만 최적화를 위해 할 수 있는 일이 있습니다. `handleSearch` 함수 안에 다음 `console.log`를 추가하세요.

```react
// /app/ui/search.tsx

function handleSearch(term: string) {
  console.log(`Searching... ${term}`);
 
  const params = new URLSearchParams(searchParams);
  if (term) {
    params.set('query', term);
  } else {
    params.delete('query');
  }
  replace(`${pathname}?${params.toString()}`);
}
```

그런 다음 검색창에 "Emil"을 입력하고 개발 도구에서 콘솔을 확인합니다. 무슨 일이 일어나고 있나요?

```shell
Searching... E
Searching... Em
Searching... Emi
Searching... Emil
```

모든 키 입력에 대해 URL을 업데이트하고 있으므로 모든 키 입력에 대해 데이터베이스를 쿼리하고 있습니다. 애플리케이션의 규모가 작을때는 문제가 되지 않지만, 수천 명의 사용자가 애플리케이션에 있고 각 사용자의 키 입력 시마다 데이터베이스에 새 요청을 보낸다고 상상해 보세요. 디바운싱은 함수가 실행될 수 있는 속도를 제한하는 프로그래밍 기법입니다. 여기서는 사용자가 입력을 중단했을때만 데이터베이스를 쿼리하려고 합니다.

> 디바운스 작동 방식
1. **Trigger Event** : 검색창의 키 입력과 같이 디바운스해야하는 이벤트가 발생하면 타이머가 시작됩니다.
2. **Wait** : 타이머가 만료되기 전에 새 이벤트가 발생하면 타이머가 재설정됩니다.
3. **Execution** : 타이머의 카운트다운이 끝나면 디바운스된 함수가 실행됩니다.
{: .prompt-tip }

디바운스 함수를 수동으로 생성하는 등 몇 가지 방법으로 디바운스를 구현할 수 있습니다. 간단하게 하기 위해 [`use-debounce`](https://www.npmjs.com/package/use-debounce){:target="\_blank"}라는 라이브러리를 사용하겠습니다. `use-debounce`를 설치합니다.

```shell
npm i use-debounce
```

`<Search />` 컴포넌트에서 `useDebouncedCallback`이라는 함수를 가져옵니다.

```react
// /app/ui/search.tsx

// ...
import { useDebouncedCallback } from 'use-debounce';
 
// Inside the Search Component...
const handleSearch = useDebouncedCallback((term) => {
  console.log(`Searching... ${term}`);
 
  const params = new URLSearchParams(searchParams);
  if (term) {
    params.set('query', term);
  } else {
    params.delete('query');
  }
  replace(`${pathname}?${params.toString()}`);
}, 500);
```

이 함수는 `handleSearch`의 내용을 래핑하고 사용자가 입력을 중지한 후 특정 시간(500밀리초) 후에만 코드를 실행합니다. 이제 검색창에 다시 입력하고 개발 도구에서 콘솔을 엽니다. 다음과 같은 내용이 표시되어야 합니다.

```shell
Searching... Emil
```

디바운싱을 통해 데이터베이스에 전송되는 요청의 수를 줄여 리소스를 절약할 수 있습니다.

## Adding pagination
Search 기능을 도입하고 나면 테이블에 한 번에 6개의 인보이스만 표시되는 것을 알 수 있습니다. 이는 `data.ts`의 `fetchFilteredInvoices()` 함수가 페이지당 최대 6개의 인보이스를 반환하기 때문입니다. Pagination을 추가하면 사용자가 여러 페이지를 탐색하여 모든 인보이스를 볼 수 있습니다. Search에서와 마찬가지로 URL 매개변수를 사용하여 Pagination을 구현하는 방법을 살펴보겠습니다.
<br />
`<Pagination />` 컴포넌트로 이동하면 클라이언트 컴포넌트라는 것을 알 수 있습니다. 클라이언트에서 데이터를 가져오면 데이터베이스 비밀이 노출될 수 있으므로 데이터를 가져오지 않는 것이 좋습니다(API 계층을 사용하지 않는다는 점을 기억하세요). 대신 서버에서 데이터를 가져와서 컴포넌트에 prop으로 전달할 수 있습니다. `dashboard/invoices/page.tsx`에서 `fetchInvoicesPages`라는 새 함수를 가져와 `searchParams`의 `query`를 인수로 전달합니다.

```react
// /app/dashboard/invoices/page.tsx

// ...
import { fetchInvoicesPages } from '@/app/lib/data';
 
export default async function Page({
  searchParams,
}: {
  searchParams?: {
    query?: string,
    page?: string,
  },
}) {
  const query = searchParams?.query || '';
  const currentPage = Number(searchParams?.page) || 1;
 
  const totalPages = await fetchInvoicesPages(query);
 
  return (
    // ...
  );
}
```

`fetchInvoicesPages`는 검색 쿼리를 기준으로 총 페이지 수를 반환합니다. 예를 들어 검색 쿼리와 일치하는 인보이스가 12개이고 각 페이지에 6개의 인보이스가 표시되는 경우 총 페이지 수는 2개가 됩니다. 다음으로 `totalPages` prop을 `<Pagination />` 컴포넌트에 전달합니다.

```react
// /app/dashboard/invoices/page.tsx

// ...
 
export default async function Page({
  searchParams,
}: {
  searchParams?: {
    query?: string;
    page?: string;
  };
}) {
  const query = searchParams?.query || '';
  const currentPage = Number(searchParams?.page) || 1;
 
  const totalPages = await fetchInvoicesPages(query);
 
  return (
    <div className="w-full">
      <div className="flex w-full items-center justify-between">
        <h1 className={`${lusitana.className} text-2xl`}>Invoices</h1>
      </div>
      <div className="mt-4 flex items-center justify-between gap-2 md:mt-8">
        <Search placeholder="Search invoices..." />
        <CreateInvoice />
      </div>
      <Suspense key={query + currentPage} fallback={<InvoicesTableSkeleton />}>
        <Table query={query} currentPage={currentPage} />
      </Suspense>
      <div className="mt-5 flex w-full justify-center">
        <Pagination totalPages={totalPages} />
      </div>
    </div>
  );
}
```

`<Pagination />` 컴포넌트로 이동하여 `usePathname` 및 `useSearchParams` 훅을 가져옵니다. 이를 사용하여 현재 페이지를 가져오고 새 페이지를 설정합니다. 이 컴포넌트에서 코드의 주석도 해제해야 합니다. 아직 `<Pagination />` 로직을 구현하지 않았기 때문에 애플리케이션이 일시적으로 중단될 것입니다. 지금 구현해 보겠습니다!

```react
// /app/ui/invoices/pagination.tsx

'use client';
 
import { ArrowLeftIcon, ArrowRightIcon } from '@heroicons/react/24/outline';
import clsx from 'clsx';
import Link from 'next/link';
import { generatePagination } from '@/app/lib/utils';
import { usePathname, useSearchParams } from 'next/navigation';
 
export default function Pagination({ totalPages }: { totalPages: number }) {
  const pathname = usePathname();
  const searchParams = useSearchParams();
  const currentPage = Number(searchParams.get('page')) || 1;
 
  // ...
}
```

다음으로 `<Pagination />` 컴포넌트 안에 `createPageURL`이라는 새 함수를 만듭니다. Search와 마찬가지로 `URLSearchParams`를 사용하여 새 페이지 번호를 설정하고 `pathName`을 사용하여 URL 문자열을 만듭니다.

```react
// /app/ui/invoices/pagination.tsx

'use client';
 
import { ArrowLeftIcon, ArrowRightIcon } from '@heroicons/react/24/outline';
import clsx from 'clsx';
import Link from 'next/link';
import { generatePagination } from '@/app/lib/utils';
import { usePathname, useSearchParams } from 'next/navigation';
 
export default function Pagination({ totalPages }: { totalPages: number }) {
  const pathname = usePathname();
  const searchParams = useSearchParams();
  const currentPage = Number(searchParams.get('page')) || 1;
 
  const createPageURL = (pageNumber: number | string) => {
    const params = new URLSearchParams(searchParams);
    params.set('page', pageNumber.toString());
    return `${pathname}?${params.toString()}`;
  };
 
  // ...
}
```

현재 상황을 설명하면 다음과 같습니다.
- `createPageURL`은 현재 검색 매개변수의 인스턴스를 생성합니다.
- 그런 다음 'page' 매개변수를 입력받은 페이지 번호로 업데이트합니다.
- 마지막으로 경로 이름과 업데이트된 검색 매개변수를 사용하여 전체 URL을 구성합니다.

나머지 `<Pagination />` 컴포넌트는 스타일링과 다양한 상태(first, last, active, disabled, etc)를 처리합니다. 이 강좌에서는 자세히 다루지 않겠지만 코드를 살펴보면서 `createPageURL`이 어디에서 호출되는지 확인하시기 바랍니다. 마지막으로 사용가자 새 검색 쿼리를 입력할 때 페이지 번호를 1로 재설정하고 싶습니다. `<Search />` 컴포넌트에서 `handleSearch` 함수를 업데이트하면 됩니다.

```react
// /app/ui/search.tsx

'use client';
 
import { MagnifyingGlassIcon } from '@heroicons/react/24/outline';
import { usePathname, useRouter, useSearchParams } from 'next/navigation';
import { useDebouncedCallback } from 'use-debounce';
 
export default function Search({ placeholder }: { placeholder: string }) {
  const searchParams = useSearchParams();
  const { replace } = useRouter();
  const pathname = usePathname();
 
  const handleSearch = useDebouncedCallback((term) => {
    const params = new URLSearchParams(searchParams);
    params.set('page', '1');
    if (term) {
      params.set('query', term);
    } else {
      params.delete('query');
    }
    replace(`${pathname}?${params.toString()}`);
  }, 500);
```

## Summary
축하합니다. 방금 URL 매개변수 및 Next.js API를 사용하여 Search 및 Pagination을 구현했습니다. 요약하자면, 이 장에서는
- 클라이언트 상태 대신 URL 검색 매개변수를 사용하여 Search 및 Pagination을 처리했습니다.
- 서버에서 데이터를 가져왔습니다.
- 보다 원활한 클라이언트 측 전환을 위해 `useRouter` 훅을 사용했습니다.

이러한 패턴은 클라이언트 React로 작업할 때 익숙한 것과는 다릅니다. URL 검색 매개변수를 사용하고 이 상태를 서버로 가져올때의 장점을 이해하셨기를 바랍니다.