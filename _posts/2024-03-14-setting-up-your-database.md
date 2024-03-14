---
title: Next.js Learn - 6. Setting Up Your Database 번역
date: 2024-03-14 23:00:00 +0900
categories: [Frontend, Nextjs]
tags: [Next.js, Next.js conf, Next.js 14]
image: /v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnextjs-conf%2Fnextjs.png?alt=media&token=09247773-9707-4dd1-b3ca-3fe7f943497a
---

> 본 포스팅은 Next.js Learn의 [Setting Up Your Database](https://nextjs.org/learn/dashboard-app/setting-up-your-database){:target="\_blank"} 내용을 번역한 것입니다.
{: .prompt-info }

대시보드 작업을 계속하려면 먼저 몇 가지 데이터가 필요합니다. 이 장에서는 `@vercel/postgres`를 사용하여 PostgreSQL 데이터베이스를 설정합니다. 이미 PostgreSQL에 익수하고 자체 공급자를 사용하려는 경우, 이 장을 건너뛰고 직접 설정할 수 있습니다. 그렇지 않다면 계속 진행하세요!

## In this chapter
앞으로 다룰 주제는 다음과 같습니다.

- 프로젝트를 Github에 푸시합니다.
- 즉시 미리 보기 및 배포를 위해 Vercel 계정을 설정하고 Github 레포지토리를 연결합니다.
- 프로젝트를 생성하고 Postgres 데이터베이스에 연결합니다.
- 데이터베이스에 초기 데이터를 시드합니다.

## Create a Github repository
먼저 아직 레포지토리를 Github에 푸시하지 않닸다면 푸시해 보겠습니다. 이렇게 하면 데이터베이스를 설정하고 배포하기 쉬워집니다. 레포지토리를 설정하는 데 도움이 필요하면 [Github 가이드](https://docs.github.com/en/repositories/creating-and-managing-repositories/quickstart-for-repositories){:target="\_blank"}를 참고하세요.

> Good to know
- Gitlab이나 Bitbucket과 같은 다른 Git 제공업체를 사용할 수도 있습니다.
- Github를 처음 사용하는 경우 단순화된 개발 워크플로우를 위해 [Github 데스크톱 앱](https://desktop.github.com/){:target="\_blank"}을 추천합니다.
{: .prompt-tip }

## Create a Vercel account
[vercel.com/signup](http://vercel.com/signup){:target="\_blank"}을 방문하여 계정을 만드세요. 무료 'hobby' 요금제를 선택합니다. Github로 계속을 선택하여 Github와 vercel 계정을 연결합니다.

## Connect and deploy your project
다음으로 방금 만든 Github 레포지토리를 선택하여 가져올 수 있는 이 화면으로 이동합니다.

![img-description](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fsetting-up-your-database%2Fimage_1.png?alt=media&token=d629ef65-61e6-4b09-a9fc-9d0204243ea1)
_출처 : Next.js Learn_

프로젝트 이름을 지정하고 Deploy를 클릭합니다.

![img-description](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fsetting-up-your-database%2Fimage_2.png?alt=media&token=d511a19b-8389-4c55-9910-de40263846d8)
_출처 : Next.js Learn_

![img-description](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fsetting-up-your-database%2Fimage_3.png?alt=media&token=8952b66f-43d0-4e27-ab1b-501bbdfd0bc5)
_출처 : Next.js Learn_

프로젝트가 배포되었습니다. Github 레포지토리를 연결하면 메인 브랜치에서 변경 사항을 푸시할때마다 별도의 설정 없이 애플리케이션을 배포할 수 있습니다. 풀 리퀘스트를 열면 [미리 보기](https://vercel.com/docs/deployments/preview-deployments#preview-urls){:target="\_blank"}가 제공되므로 배포 오류를 조기에 발견하고 팀원들과 프로젝트 미리 보기를 공유하여 피드백을 받을 수 있습니다.

## Create a Postgres database
다음으로 데이터베이스를 설정하려면 Continue to Dashboard를 클릭하고 프로젝트 대시보드에서 Storage 탭을 선택합니다. Connect Store -> Create New -> Postgres -> Continue를 선택합니다.

![img-description](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fsetting-up-your-database%2Fimage_4.png?alt=media&token=55307e81-c0b0-4fd9-9ef0-0ccf59654857)
_출처 : Next.js Learn_

약관에 동의하고, 데이터베이스에 이름을 지정하고, 데이터베이스 지역을 Washingthon D.C(iad1)로 설정하세요. 이 지역은 모든 새 Vercel 프로젝트의 [기본 지역](https://vercel.com/docs/functions/configuring-functions/region#select-a-default-serverless-region){:target="\_blank"}이기도 합니다. 데이터베이스를 동일한 지역 또는 애플리케이션 코드와 가까운 곳에 배치하면 데이터 요청에 대한 [대기 시간](https://developer.mozilla.org/en-US/docs/Web/Performance/Understanding_latency){:target="\_blank"}을 줄일 수 있습니다.

![img-description](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fsetting-up-your-database%2Fimage_5.png?alt=media&token=fe803f95-ec31-455e-bb7a-bcafc2a1fca3)
_출처 : Next.js Learn_

> Good to know
데이터 베이스 지역을 초기화한 후에는 변경할 수 없습니다. 다른 [지역](https://vercel.com/docs/storage/vercel-postgres/limits#supported-regions){:target="\_blank"}을 사용하려면 데이터베이스를 만들기 전에 해당 지역을 설정해야 합니다.
{: .prompt-tip }

연결이 되었다면 `.env.local` 탭으로 이동하여 Show secret 및 Copy Snippet을 클릭합니다. secret을 복사하기 전에 secret을 공개했는지 확인하세요.
![img-description](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fsetting-up-your-database%2Fimage_6.png?alt=media&token=5f2ecbf1-3866-4540-a997-0e0094e701b0)
_출처 : Next.js Learn_

코드 편집기로 이동하여 `.env.example` 파일의 이름을 `.env`로 바꿉니다. Vercel에서 복사한 내용을 붙여넣습니다. `.gitignore` 파일로 이동하여 `.env`가 파일에 포함되어 있는지 확인하여 Github에 푸시할 때 데이터베이스 비밀이 노출되지 않도록 하세요. 마지막으로 터미널에서 `npm i @vercel/postgres`를 실행하여 [Vercel Postgres SDK](https://vercel.com/docs/storage/vercel-postgres/sdk){:target="\_blank"}를 설치합니다.

## Seed your database
이제 데이터베이스가 만들어졌으므로 초기 데이터로 시딩해 보겠습니다. 이렇게 하면 대시보드를 만들때 작업할 데이터를 확보할 수 있습니다. <br />
프로젝트의 `/scripts` 폴더에 `seed.js`라는 파일이 있습니다. 이 스크립트에는 `invoices`, `customers`, `user`, `revenue` 테이블을 만들고 시딩하는 지침이 포함되어 있습니다. 코드가 수행하는 모든 작업을 이해하지 못하더라도 걱정하지 마시고 개요를 알려드리기 위해 스크립트에서는 SQL을 사용하여 테이블을 만들고 테이블이 생성된 후 `placeholder-data.js` 파일의 데이터를 사용하여 테이블을 채웁니다. <br />
다음으로 `package.json` 파일에서 스크립트에 다음 줄을 추가합니다.

```typescript
// /package.json

"scripts": {
  "build": "next build",
  "dev": "next dev",
  "start": "next start",
  "seed": "node -r dotenv/config ./scripts/seed.js"
},
```

`seed.js`를 실행하는 명령입니다. 이제 `npm run seed`를 실행합니다. 터미널에 스크립트가 실행 중임을 알려주는 몇 가지 `console.log` 메시지가 표시되어야 합니다.

> Troubleshooting
- `.env` 파일에 복사하기 전에 데이터베이스 비밀번호를 공개해야 합니다.
- 이 스크립트는 `bcrypt`를 사용하여 사용자 비밀번호를 해시하는데 `bcrypt`가 사용중인 환경과 호환되지 않는 경우 대신 [`bcryptjs`](https://www.npmjs.com/package/bcryptjs){:target="\_blank"}를 사용하도록 스크립트를 업데이트 할 수 있습니다.
- 데이터베이스를 시드하는 동안 문제가 발생하여 스크립트를 다시 실행하려는 경우 데이터베이스 쿼리 인터페이스에서 `DROP TABLE tablename`를 실행하여 기존 테이블을 삭제할 수 있습니다. 자세한 내용은 아래의 Executing queries 섹션을 참조하세요. 하지만 이 명령은 테이블과 해당 테이블의 모든 데이터를 삭제하므로 주의하세요. placeholder 데이터로 작업하기 때문에 예제 앱에서 이 작업을 수행하는 것은 괜찮지만 프로덕션 앱에서 이 명령을 실행해서는 안됩니다.
- Vercel Postgres데이터베이스를 시딩하는 동안 문제가 게속 발생하면 [discussion on Github](https://github.com/vercel/next-learn/issues){:target="\_blank"}를 열어주세요.
{: .prompt-tip }

## Exploring your database
데이터베이스가 어떻게 생겼는지 살펴봅시다. Vercel로 돌아가서 사이드바에서 Data를 클릭합니다. 이 섹션에는 user, customers, invoices, revenue의 네 가지 테이블이 있습니다.

![img-description](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fsetting-up-your-database%2Fimage_7.png?alt=media&token=899c5092-77bd-4ed2-a61e-db6717669b10)
_출처 : Next.js Learn_

각 테이블을 선택하면 해당 레코드를 보고 항목이 `placeholder-data.js` 파일의 데이터와 일치하는지 확인할 수 있습니다.

## Executing queries
'query' 탭으로 전환하여 데이터베이스와 상호 작용할 수 있습니다. 이 섹션에서는 표준 SQL 명령을 지원합니다. 예를 들어 `DROP TABLE customers`를 입력하면 모든 데이터와 함께 "customers" 테이블이 삭제되므로 주의하세요. 첫 번째 데이터베이스 쿼리를 실행해 보겠습니다. 다음 SQL 코드를 Vercel 인터페이스에 붙여넣고 실행합니다.

```sql
SELECT invoices.amount, customers.name
FROM invoices
JOIN customers ON invoices.customer_id = customers.id
WHERE invoices.amount = 666;
```

## Learning
- Vercel에서는 다음과 같은 서비스를 제공한다.
    1. 코드 빌드 및 배포 : 깃허브 메인 레포지토리에 코드를 푸시하면 자동으로 빌드하고 배포가 된다. 풀 리퀘스트를 하면 미리보기를 할 수 있는데 이를 통해 미리 빌드 에러를 파악할 수 있다.
    2. 데이터베이스 : Postgres 데이터베이스를 제공하며 직관적인 UI로 쿼리문을 실행할 수 있다.