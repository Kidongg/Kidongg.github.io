---
title: Next.js Learn - Introduction 번역
date: 2024-01-05 01:05:00 +0900
categories: [Frontend, Nextjs]
tags: [Next.js, Next.js conf, Next.js 14]
image: /v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnextjs-conf%2Fnextjs.png?alt=media&token=09247773-9707-4dd1-b3ca-3fe7f943497a
---

> 본 포스팅은 Next.js Learn의 [Introduction](https://nextjs.org/learn/dashboard-app){:target="\_blank"} 내용을 번역한 것입니다.
{: .prompt-info }

## Learn Next.js

Next.js 앱 라우터 강좌에 오신 것을 환영합니다! 이 무료 과정에서는 풀스택 웹 애플리케이션을 구축하여 Next.js의 주요 기능을 배웁니다.

---

## What we'll be building

![img-description](/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fintroduction%2Fresult.png?alt=media&token=d6f2eab9-f499-45e4-af94-3179f818a37d)
_출처 : Next.js Learn_

이 과정에서는 대시보드의 간소화된 버전을 만들 것입니다.

- A public home page
- A login page
- Dashboard pages that are protected by authentication
- The ability for users to add, edit, and delete invoices

대시보드에는 데이터베이스도 함께 제공되며, 이 데이터베이스는 이후 장에서 설정하게 됩니다. <br/>
이 과정을 마치면 풀스택 Next.js 애플리케이션을 구축하는 데 필요한 필수 기술을 갖추게 됩니다.

---

## Overview

다음은 이 과정에서 배우게 될 기능에 대한 개요입니다.

- **Styling** : Next.js에서 애플리케이션의 스타일을 지정하는 다양한 방법
- **Optimizations** : 이미지, 링크 및 글꼴을 최적화하는 방법
- **Routing** : 파일 시스템 라우팅을 사용하여 중첩된 레이아웃과 페이지를 만드는 방법
- **Data Fetching** : Vercel에서 데이터베이스를 설정하는 방법과 데이터 패칭 및 스트리밍에 대한 모범 사례
- **Search and Pagination** : URL 검색 매개변수를 사용하여 검색 및 페이지 네이션을 구현하는 방법
- **Mutating Data** : React Server Action을 사용하여 데이터를 변경하고 Next.js 캐시의 유효성을 다시 검사하는 방법
- **Error Handling** : 일반 오류 및 404 오류를 처리하는 방법
- **Form Validation and Accessibility** : 서버 측 양식 유효성 검사를 수행하는 방법과 접근성을 개선하기 위한 팁
- **Authentication** : [`NextAuth.js`](https://next-auth.js.org/){:target="\_blank"} 및 미들웨어를 사용하여 애플리케이션에 인증을 추가하는 방법
- **Metadata** : 메타데이터를 추가하고 소셜 공유를 위해 애플리케이션을 준비하는 방법

---

## Prerequisite knowledge

이 강좌는 React와 JavaScript에 대한 기본적인 이해가 있다고 가정합니다. React를 처음 접하는 분이라면 먼저 [React 기본](https://nextjs.org/learn/react-foundations){:target="\_blank"} 과정을 통해 components, props, state, hooks와 같은 React 기본 사항과 서버 컴포넌트 및 서스펜스 같은 최신 기능을 학습하는 것이 좋습니다.

---

## System requirements

이 과정을 시작하기 전에 시스템이 다음 요구 사항을 충족하는지 확인하세요.

- Node.js 18.17.0 이상 설치. [다운로드 하는 곳](https://nodejs.org/en){:target="\_blank"}.
- 운영 체제 : macOS, Windows(WSL 포함) 또는 Linux.

---

## Join the conversation

이 과정에 대해 궁금한 점이 있거나 피드백을 제공하려면 [Discord](https://discord.com/invite/Q3AsD4efFC){:target="\_blank"} 또는 [GitHub](https://github.com/vercel/next-learn){:target="\_blank"}의 커뮤니티에 문의하세요.

---

## My Opinion

개인적으로 Next.js를 이해하는데 Next.js Learn이 큰 도움이 되었습니다. Next.js에 관심있는 분들이 해당 문서를 통해 공부하시는 날이 오면 좋겠습니다. 본 번역 시리즈가 프론트엔드 개발자께 도움이 되길 바랍니다. 감사합니다.
