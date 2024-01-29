---
title: Next.js Learn - 1. Getting Started 번역
date: 2024-01-14 12:35:00 +0900
categories: [Frontend, Nextjs]
tags: [Next.js, Next.js conf, Next.js 14]
image: /v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnextjs-conf%2Fnextjs.png?alt=media&token=09247773-9707-4dd1-b3ca-3fe7f943497a
---

> 본 포스팅은 Next.js Learn의 [Getting Started](https://nextjs.org/learn/dashboard-app/getting-started){:target="\_blank"} 내용을 번역한 것입니다.
{: .prompt-info }

## Creating a new project

Next.js 앱을 만들려면 터미널을 열고 프로젝트를 보관하려는 폴더에 `cd`를 한 다음 다음 명령을 실행합니다.

```shell
npx create-next-app@latest nextjs-dashboard --use-npm --example "https://github.com/vercel/next-learn/tree/main/dashboard/starter-example"
```

이 명령은 Next.js 애플리케이션을 설정하는 CLI(명령줄 인터페이스) 도구인 `create-next-app`을 사용합니다. 위의 명령에서는 이 강좌의 [시작 예제](https://github.com/vercel/next-learn/tree/main/dashboard/starter-example){:target="\_blank"}와 함께 `--example`` 플래그를 사용하고 있습니다.

## Exploring the project

처음부터 코드를 작성해야 하는 튜토리얼과 달리 이 강좌의 코드 대부분은 이미 작성되어 있습니다. 따라서 기존 코드베이스로 작업할 가능성이 높은 실제 개발 환경을 더 잘 반영합니다. <br />
이 강좌의 목표는 모든 애플리케이션 코드를 작성할 필요 없이 Next.js의 주요 기능을 학습하는 데 집중할 수 있도록 돕는 것입니다. <br />
설치 후 코드 편집기에서 프로젝트를 열고 `nextjs-dashboard`로 이동합니다.

```shell
cd nextjs-dashboard
```

프로젝트를 살펴보는 시간을 가져보겠습니다.

## Folder structure

프로젝트의 폴더 구조가 다음과 같다는 것을 알 수 있습니다.
![img-description](/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fgetting-started%2Ffolder-structure.png?alt=media&token=238b6c3f-f741-487b-930b-2a95662b0d95)
_출처 : Next.js Learn_

- `/app` : 애플리케이션의 모든 경로, 컴포넌트, 로직이 포함되어 있으며 대부분 이곳에서 작업하게 됩니다.
- `/app/lib` : 재사용 가능한 유틸리티 함수, 데이터 불러오기 함수 등 애플리케이션에서 사용되는 함수가 들어 있습니다.
- `/app/ui` : 카드, 표, 양식 등 애플리케이션의 모든 UI 구성 요소가 들어 있습니다. 시간을 절약하기 위해 이러한 구성 요소는 미리 스타일이 지정되어 있습니다.
- `/public` : 이미지와 같은 애플리케이션의 모든 정적 에셋을 포함합니다.
- `/scripts` : 이후 장에서 데이터베이스를 채우는 데 사용할 시딩 스크립트가 포함되어 있습니다.
- Config Files : 애플리케이션의 루트에는 `next.config.js`와 같은 구성 파일도 있습니다. 이러한 파일의 대부분은 `create-next-app`을 사용하여 새 프로젝트를 시작할 때 생성되고 미리 구성됩니다. 이 과정에서는 이러한 파일을 수정할 필요가 없습니다.

이러한 폴더를 자유롭게 탐색하고 코드가 수행하는 모든 작업을 아직 이해하지 못하더라도 걱정하지 마세요.

## Placeholder data

사용자 인터페이스를 구축할 때는 플레이스홀더 데이터가 있으면 도움이 됩니다. 데이터베이스나 API를 아직 사용할 수 없는 경우에는 사용할 수 있습니다.

- 플레이스홀더 데이터를 JSON 형식 또는 JavaScript 객체로 사용합니다.
- [mockAPI](https://mockapi.io/){:target="\_blank"}와 같은 타사 서비스를 사용합니다.

이 프로젝트에서는 `app/lib/placeholder-data.js`에 일부 플레이스홀더 데이터를 제공했습니다. 파일의 각 JavaScript 객체는 데이터베이스의 테이블을 나타냅니다. 예를 들어, 송장 테이블의 경우

```javascript
// /app/lib/placeholder-data.js

const invoices = [
  {
    customer_id: customers[0].id,
    amount: 15795,
    status: "pending",
    date: "2022-12-06",
  },
  {
    customer_id: customers[1].id,
    amount: 20348,
    status: "pending",
    date: "2022-11-14",
  },
  // ...
];
```

데이터베이스 설정 장에서는 이 데이터를 사용하여 데이터베이스를 시드(초기 데이터로 채우기)합니다.

## TypeScript

대부분의 파일에 `.ts` 또는 `.tsx` 접미사가 붙는 것을 볼 수도 있습니다. 이는 프로젝트가 TypeScript로 작성되었기 때문입니다. 저희는 최신 웹 환경을 반영하는 강좌를 만들고 싶었습니다. <br />
TypeScript를 모르셔도 괜찮습니다. 필요한 경우 TypeScript 코드 스니펫을 제공해 드리겠습니다. <br/>
지금은 `/app/lib/definitions.ts` 파일을 살펴보세요. 여기에서는 데이터베이스에서 반환할 유형을 수동으로 정의합니다. 예를 들어, 송장 테이블에는 다음과 같은 유형이 있습니다.

```typescript
// /app/lib/definitions.ts

export type Invoice = {
  id: string;
  customer_id: string;
  amount: number;
  date: string;
  // In TypeScript, this is called a string union type.
  // It means that the "status" property can only be one of the two strings: 'pending' or 'paid'.
  status: "pending" | "paid";
};
```

TypeScript를 사용하면 송장 `amount`에 ` number`` 대신  `string`을 전달하는 등 실수로 잘못된 데이터 형식을 컴포넌트나 데이터베이스에 전달하지 않도록 할 수 있습니다.

> TypeScript 개발자인 경우

1. 데이터 유형을 수동으로 선언하고 있지만 유형 안전성을 높이려면 데이터베이스 스키마에 따라 유형을 자동으로 생성하는 [`Prisma`](https://www.prisma.io/){:target="\_blank"}를 사용하는 것이 좋습니다.
2. Next.js는 프로젝트에서 TypeScript를 사용하는지 감지하여 필요한 패키지와 구성을 자동으로 설치합니다. Next.js는 자동 완성 및 유형 안전성을 지원하기 위해 코드 편집기용 [`TypeScript plugin`](https://nextjs.org/docs/app/building-your-application/configuring/typescript#typescript-plugin){:target="\_blank"}도 함께 제공됩니다.
   {: .prompt-tip }

## Running the development server

`npm install`를 실행하여 프로젝트의 패키지를 설치합니다.

```shell
npm install
```

이어서 `npm run dev`를 실행하여 개발 서버를 시작합니다.

```shell
npm run dev
```

`npm run dev`는 포트 3000에서 Next.js 개발 서버를 시작합니다. 제대로 작동하는지 확인해 봅시다. 브라우저에서 [http://localhost:3000](http://localhost:3000){:target="\_blank"}을 엽니다. 홈페이지는 다음과 같이 표시되어야 합니다.
![img-description](/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fgetting-started%2Fhomepage.png?alt=media&token=d3373c19-7753-4143-8246-0e160a17daad)
_출처 : Next.js Learn_

## Opinion

- Next.js에서 공식적으로 제안하는 앱 라우터 파일 구조에 대해 알 수 있었습니다. `/app`, `/app/lib`, `/app/ui`, `/public`, `/scripts`의 구조는 기능에 따라 파일을 관리할 수 있다는 장점이 있는 것같습니다.
- Next.js Learn이 입문자들의 눈높이에서 작성되었다는 것을 단번에 알 수 있었습니다. TypeScript에 대한 내용을 모르더라도 커리큘럼을 따라가다보면 TypeScript의 필요성과 사용법을 이해할 수 있도록 구성했기 때문입니다.
