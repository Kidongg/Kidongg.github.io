---
title: Next.js Learn - 16. Adding Metadata 번역
date: 2024-06-14 23:00:00 +0900
categories: [Frontend, Next.js Learn]
tags: [Next.js, Next.js conf, Next.js 14]
image: /v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnextjs-conf%2Fnextjs.png?alt=media&token=09247773-9707-4dd1-b3ca-3fe7f943497a
---

> 본 포스팅은 Next.js Learn의 [Adding Metadata](https://nextjs.org/learn/dashboard-app/adding-metadata){:target="\_blank"} 내용을 번역한 것입니다.
{: .prompt-info }

메타데이터는 SEO와 공유 가능성을 위해 매우 중요합니다. 이 장에서는 Next.js 애플리케이션에 메타데이터를 추가하는 방법을 설명합니다.

## In this chapter
앞으로 다룰 주제는 다음과 같습니다.
- 메타데이터란 무엇인가요?
- 메타데이터의 유형
- 메타데이터를 사용하여 오픈 그래프 이미지를 추가하는 방법
- 메타데이터를 사용하여 파비콘을 추가하는 방법

## What is metadata?
웹 개발에서 메타데이터는 웹 페이지에 대한 추가 세부 정보를 제공합니다. 메타데이터는 페이지를 방문하는 사용자에게 표시되지 않습니다. 대신 페이지의 HTML, 일반적으로 `<head>` 요소 내에 포함된 백그라운드에서 작동합니다. 이 숨겨진 정보는 웹페이지의 콘텐츠를 더 잘 이해해야 하는 검색 엔진 및 기타 시스템에 매우 중요합니다.

## Why is metadata important?
메타데이터는 검색 엔진과 소셜 미디어 플랫폼이 웹페이지에 더 쉽게 접근하고 이해할 수 있도록 웹페이지의 SEO를 향상시키는데 중요한 역할을 합니다. 적절한 메타데이터는 검색 엔진이 웹페이지를 효과적으로 색인화하여 검색 결과에서 웹페이지의 순위를 높이는 데 도움이 됩니다. 또한 오픈 그래프와 같은 메타데이터는 소셜 미디어에서 공유 링크의 모양을 개선하여 사용자에게 더 매력적이고 유익한 콘텐츠를 제공합니다.

## Types of metadata
메타데이터에는 다양한 유형이 있으며 각각 고유한 용도로 사용됩니다. 몇 가지 일반적인 유형은 다음과 같습니다. 

- **Title Metadata** : 브라우저 탭에 표시되는 웹페이지의 제목을 담당합니다. 검색 엔진이 웹페이지의 내용을 이해하는 데 도움이 되므로 SEO에 매우 중요합니다.

```html
<title>Page Title</title>
```

- **Description Metadata** : 웹페이지 콘텐츠에 대한 간략한 개요를 제공하며 검색 엔진 결과에 표시되는 경우가 많습니다.

```html
<meta name="description" content="A brief description of the page content." />
```

- **Keyword Metadata** : 웹페이지 콘텐츠와 관련된 키워드가 포함되어 있어 검색 엔진이 페이지 색인을 생성하는 데 도움이 됩니다.

```html
<meta name="keywords" content="keyword1, keyword2, keyword3" />
```

- **Open Graph Metadata** : 소셜 미디어 플랫폼에서 공유할 때 웹페이지가 표시되는 방식을 개선하여 제목, 설명 및 미리보기 이미지와 같은 정보를 제공합니다.

```html
<meta property="og:title" content="Title Here" />
<meta property="og:description" content="Description Here" />
<meta property="og:image" content="image_url_here" />
```

- **Favicon Metadata** : 브라우저의 주소 표시줄 또는 탭에 표시되는 파비콘을 웹페이지로 연결합니다.

```html
<link rel="icon" href="path/to/favicon.ico" />
```

## Adding metadata
Next.js에는 애플리케이션 메타데이터를 정의하는 데 사용할 수 있는 메타데이터 API가 있습니다. 애플리케이션에 메타데이터를 추가하는 방법에는 두 가지가 있습니다.

- Config-based : [static `metadata` 객체](https://nextjs.org/docs/app/api-reference/functions/generate-metadata#metadata-object){:target="\_blank"} 또는 [dynamic `generateMetadata` 함수](https://nextjs.org/docs/app/api-reference/functions/generate-metadata#generatemetadata-function){:target="\_blank"}을 `layout.js` 또는 `page.js` 파일로 내보냅니다.

- File-based : Next.js에는 메타데이터 용도로 특별히 사용되는 다양한 특수 파일이 있습니다.
    - `favicon.ico`, `apple-icon.jpg`, `icon.jpg` : 파비콘과 아이콘에 활용됩니다.
    - `opengraph-image.jpg` 및 `twitter-image.jpg` : 소셜 미디어 이미지에 사용됩니다.
    - `robots.txt` : 검색 엔진 크롤링에 대한 지침 제공
    - `sitemap.xml` : 웹사이트 구조에대한 정보 제공

이러한 파일을 정적 메타데이터에 유연하게 사용하거나 프로젝트 내에서 프로그래밍 방식으로 생성할 수 있습니다. 이 두 가지 옵션을 모두 사용하면 Next.js가 페이지에 관련된 `<head>` 요소를 자동으로 생성합니다.

### Favicon and Open Graph image
`/public` 폴더에 두 개의 이미지, 즉 `favicon.ico`와 `opengraph-image.jpg`가 있는 것을 확인할 수 있습니다. 이 이미지를 `/app` 폴더의 루트로 옮깁니다. 이렇게 하면 Next.js가 자동으로 이 파일을 식별하여 파비콘 및 OG 이미지로 사용합니다. 개발자 도구에서 애플리케이션의 `<head>` 요소를 확인하면 이를 확인할 수 있습니다.

> Good to know  <br /> [`ImageResponse`](https://nextjs.org/docs/app/api-reference/functions/image-response){:target="\_blank"} 생성자를 사용하여 동적 OG 이미지를 만들 수도 있습니다.
{: .prompt-tip }

### Page title and descriptions
`layout.js` 또는 `page.js` 파일에서 [`metadata` 객체](https://nextjs.org/docs/app/api-reference/functions/generate-metadata#metadata-fields){:target="\_blank"}를 포함시켜 제목 및 설명과 같은 페이지 정보를 추가할 수도 있습니다. `layout.js`의 모든 메타데이터는 이를 사용하는 모든 페이지에서 상속됩니다. 루트 레이아웃에서 다음 필드를 사용하여 새 `metadata` 객체를 만듭니다.

```react
// /app/layout.tsx

import { Metadata } from 'next';
 
export const metadata: Metadata = {
  title: 'Acme Dashboard',
  description: 'The official Next.js Course Dashboard, built with App Router.',
  metadataBase: new URL('https://next-learn-dashboard.vercel.sh'),
};
 
export default function RootLayout() {
  // ...
}
```

Next.js는 애플리케이션에 제목과 메타데이터를 자동으로 추가합니다. 하지만 특정 페이지에 사용자 정의 제목을 추가하려면 어떻게 해야 할까요? 페이지 자체에 `metadata` 객체를 추가하면 됩니다. 중첩된 페이지의 메타데이터는 상위 페이지의 메타데이터를 재정의합니다. 예를 들어 `/dashboard/invoices` 페이지에서 페이지 제목을 업데이트할 수 있습니다. 

```react
// /app/dashboard/invoices/page.tsx

import { Metadata } from 'next';
 
export const metadata: Metadata = {
  title: 'Invoices | Acme Dashboard',
};
```

이 방법은 작동하지만 모든 페이지에서 애플리케이션 제목을 반복하고 있습니다. 회사 이름과 같은 변경 사항이 있으면 모든 페이지에서 업데이트해야 합니다. 대신 `metadata` 객체의 `title.template` 필드를 사용하여 페이지 제목에 대한 템플릿을 정의할 수 있습니다. 이 템플릿에는 페이지 제목과 포함하려는 다른 모든 정보를 포함할 수 있습니다. 루트 레이아웃에서 템플릿을 포함하도록 `metadata` 객체를 업데이트합니다.

```react
// /app/layout.tsx

import { Metadata } from 'next';
 
export const metadata: Metadata = {
  title: {
    template: '%s | Acme Dashboard',
    default: 'Acme Dashboard',
  },
  description: 'The official Next.js Learn Dashboard built with App Router.',
  metadataBase: new URL('https://next-learn-dashboard.vercel.sh'),
};
```

템플릿의 `%s`가 특정 페이지 제목으로 대체됩니다. 이제 `/dashboard/invoices` 페이지에서 페이지 제목을 추가할 수 있습니다.

```react
// /app/dashboard/invoices/page.tsx

export const metadata: Metadata = {
  title: 'Invoices',
};
```

`/dashboard/invoices` 페이지로 이동하여 `<head>` 요소를 확인합니다. 이제 페이지 제목이 `Invoices | Acme Dashboard`로 바뀐 것을 확인할 수 있습니다.

## Practice: Adding metadata
이제 메타데이터에 대해 배웠으니 다른 페이지에 제목을 추가하여 연습해 보세요.

1. `/login` page
2. `/dashboard` page
3. `/dashboard/customers` page
4. `/dashboard/invoices/create` page
5. `/dashboard/invoices/[id]/edit` page

Next.js 메타데이터 API는 강력하고 유연하여 애플리케이션의 메타데이터를 완벽하게 제어할 수 있습니다. 여기에서는 몇 가지 기본 메타데이터를 추가하는 방법을 보여드렸지만 `keywords`, `robots`, `canonical` 등 여러 필드를 추가할 수 있습니다. [문서](https://nextjs.org/docs/app/api-reference/functions/generate-metadata){:target="\_blank"}를 살펴보고 애플리케이션에 원하는 메타데이터를 추가해 보세요.