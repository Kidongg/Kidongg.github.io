---
title: Next.js Learn - 8. Static and Dynamic Rendering 번역
date: 2024-03-28 00:30:00 +0900
categories: [Frontend, Next.js Learn]
tags: [Next.js, Next.js conf, Next.js 14]
image: /v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnextjs-conf%2Fnextjs.png?alt=media&token=09247773-9707-4dd1-b3ca-3fe7f943497a
---

> 본 포스팅은 Next.js Learn의 [Static and Dynamic Rendering](https://nextjs.org/learn/dashboard-app/static-and-dynamic-rendering){:target="\_blank"} 내용을 번역한 것입니다.
{: .prompt-info }

이전 장에서는 대시보드 개요 페이지에 대한 데이터를 가져왔습니다. 그러나 현재 설정의 두 가지 제한 사항에 대해 간략하게 설명했습니다.
- 데이터 요청이 의도하지 않은 waterfall을 만들고 있습니다.
- 대시보드는 정적이므로 데이터 업데이트가 애플리케이션에 반영되지 않습니다.

## In this chapter
앞으로 다룰 주제는 다음과 같습니다.
- Static rendering이란 무엇이며 이를 통해 애플리케이션의 성능을 향상시킬 수 있는 방법
- Dynamic rendering이란 무엇이며 언제 사용해야 하는지에 대한 설명
- 대시보드를 동적으로 만들기 위한 다양한 접근 방식
- 느린 데이터를 가져오기를 시뮬레이션

## What is Static Rendering
Static rendering을 사용하면 빌드 시(배포할 때) 또는 [재검증](https://nextjs.org/docs/app/building-your-application/data-fetching/fetching-caching-and-revalidating#revalidating-data){:target="\_blank"} 중에 서버에서 데이터 가져오기 및 렌더링이 이루어집니다. 그런 다음 결과는 [Content Delivery Network](https://nextjs.org/docs/app/building-your-application/rendering/server-components#static-rendering-default){:target="\_blank"}(CDN)에 배포 및 캐시될 수 있습니다.

![img-description](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fstatic-and-dynamic-rendering%2Fimage_1.png?alt=media&token=03236a70-e289-4961-a44f-a46ad08a5d25)
_출처 : Next.js Learn_

사용자가 애플리케이션을 방문할 때마다 캐시된 결과가 제공됩니다. Static rendering에는 몇 가지 이점이 있습니다.
- Faster Websites : 미리 렌더링된 콘텐츠를 캐시하여 전 세계에 배포할 수 있습니다. 따라서 전 세계 사용자가 웹사이트의 콘텐츠에 더 빠르고 안정적으로 액세스 할 수 있습니다.
- Reduced Server Load : 콘텐츠가 캐시되므로 서버에서 각 사용자 요청에 대해 콘텐츠를 동적으로 생성할 필요가 없습니다.
- SEO : 미리 렌더링된 콘텐츠는 페이지가 로드될 때 이미 콘텐츠를 사용할 수 있으므로 검색 엔진 크롤러가 색인을 생성하기가 더 쉽습니다. 이는 검색 엔진 순위 향상으로 이어질 수 있습니다.

Static rendering은 정적 블로그 게시물이나 제품 페이지와 같이 데이터가 없거나 사용자 간에 공유되는 데이터가 없는 UI에 유용합니다. 정기적으로 업데이트되는 개인화된 데이터가 있는 대시보드에는 적합하지 않을 수 있습니다. Static rendering의 반대 개념은 dynamic rendering입니다.

## What is Dynamic Rendering
Dynamic rendering을 사용하면 요청 시(사용자가 페이지를 방문할 때) 각 사용자에 대한 콘텐츠가 서버에서 렌더링됩니다. Dynamic rendering에는 몇 가지 이점이 있습니다.
- Real-Time Data : Dynamic rendering을 사용하면 애플리케이션에서 실시간 또는 자주 업데이트되는 데이터를 표시할 수 있습니다. 데이터가 자주 변경되는 애플리케이션에 이상적입니다.
- User-Specific Content : 대시보드나 사용자 프로필과 같은 개인화된 콘텐츠를 제공하고 사용자 상호 작용에 따라 데이터를 업데이트하기가 더 쉽습니다.
- Request Time Information : Dynamic rendering을 사용하면 cookies나 URL search parameters와 같이 요청 시점에만 알 수 있는 정보에 액세스할 수 있습니다.

## Making the dashboard dynamic
기본적으로 `@vercel/postgres`는 자체 캐싱 시맨틱을 설정하지 않습니다. 따라서 프레임워크가 자체 정적 및 동적 동작을 설정할 수 있습니다. Server Component 또는 data fetching function 내부에서 `unstable_noStore`라는 Next.js API를 사용하여 정적 렌더링을 선택 해제할 수 있습니다. 이것을 추가해 보겠습니다. <br />
`data.ts`에서 `next/cache`에서 `unstable_noStore`를 가져와서 데이터 가져오기 함수의 맨 위에 호출합니다.

```react
// /app/lib/data.ts

// ...
import { unstable_noStore as noStore } from 'next/cache';
 
export async function fetchRevenue() {
  // Add noStore() here to prevent the response from being cached.
  // This is equivalent to in fetch(..., {cache: 'no-store'}).
  noStore();
 
  // ...
}
 
export async function fetchLatestInvoices() {
  noStore();
  // ...
}
 
export async function fetchCardData() {
  noStore();
  // ...
}
 
export async function fetchFilteredInvoices(
  query: string,
  currentPage: number,
) {
  noStore();
  // ...
}
 
export async function fetchInvoicesPages(query: string) {
  noStore();
  // ...
}
 
export async function fetchFilteredCustomers(query: string) {
  noStore();
  // ...
}
 
export async function fetchInvoiceById(query: string) {
  noStore();
  // ...
}
```

> Note : `unstable_noStore`는 실험적인 API이며 향후 변경될 수 있습니다. 자체 프로젝트에서 안정적인 API를 사용하려면 [Segment Config Option](https://nextjs.org/docs/app/api-reference/file-conventions/route-segment-config){:target="\_blank"}인 `export const dynamic = "force-dynamic"`을 사용할 수도 있습니다.
{: .prompt-tip }

## Simulating a Slow Data Fetch
대시보드를 동적으로 만드는 것은 좋은 첫 번째 단계입니다. 하지만 이전 장에서 언급한 한 가지 문제가 여전히 남아 있습니다. 하나의 데이터 요청이 다른 모든 데이터 요청보다 느리면 어떻게 될까요? 느린 데이터 가져오기를 실험해보겠습니다. `data.ts` 파일에서 `console log`의 주석 처리를 해제하고 `fetchRevenue()` 내에서 `setTimeout`을 설정합니다.

```react
// /app/lib/data.ts

export async function fetchRevenue() {
  try {
    // We artificially delay a response for demo purposes.
    // Don't do this in production :)
    console.log('Fetching revenue data...');
    await new Promise((resolve) => setTimeout(resolve, 3000));
 
    const data = await sql<Revenue>`SELECT * FROM revenue`;
 
    console.log('Data fetch completed after 3 seconds.');
 
    return data.rows;
  } catch (error) {
    console.error('Database Error:', error);
    throw new Error('Failed to fetch revenue data.');
  }
}
```

이제 새 탭에서 [http://localhost:3000/dashboard/](http://localhost:3000/dashboard/){:target="\_blank"}을 열고 페이지가 로드되는 데 시간이 오래 걸리는지 확인합니다. 터미널에 다음과 같은 메세지도 표시됩니다.

```bash
Fetching revenue data...
Data fetch completed after 3 seconds.
```
여기에서는 느린 데이터 가져오기를 실험하기 위해 인위적인 3초 지연을 추가했습니다. 그 결과 이제 데이터를 가져오는 동안 페이지 전체가 차단됩니다. 이제 개발자가 해결해야 하는 일반적인 문제가 생겼습니다.