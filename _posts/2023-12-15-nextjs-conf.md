---
title: Next.js Conf (2023.10.26) 번역
date: 2023-12-15 22:50:00 +0900
categories: [Frontend, Nextjs]
tags: [Next.js, Next.js-conf, Next.js-14]
image: /v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnextjs-conf%2Fnextjs.png?alt=media&token=09247773-9707-4dd1-b3ca-3fe7f943497a
---

> 본 포스팅은 [Next.js의 공식 블로그](https://nextjs.org/blog/next-14)의 Next.js conf 내용을 번역한 것입니다.
> {: .prompt-tip }

## Next.js 14

---

Next.js conf에서 발표했듯이 Next.js 14에 중점을 둔 릴리스입니다.

- `Turbopack` : 앱 및 페이지 라우터에 대한 5,000회 테스트 통과
  - 53% 더 빠른 로컬 서버 시작
  - 빠른 새로고침으로 94% 더 빠른 코드 업데이트
- `Server Actions` (Stable)
  - 캐싱 및 재검증과 통합
  - 간단한 함수 호출 또는 양식과 함께 동작
- `Partial Prerendering` (Preview) : 빠른 초기 정적 응답 + 동적 콘텐츠 스트리밍
- `Next.js Learn` (New) : App router, Authentication, Database 등을 가르치는 무료 교육 과정

지금 업그레드하거나 시작하세요:

```shell
npx create-next-app@latest
```

## Next.js Compiler: Turbocharged

---

`next dev`를 위한 5,000건의 통합 테스트가 현재 기본 Rust 엔진인 [Turbopack](https://turbo.build/pack)을 통해 진행 중입니다. 이러한 테스에는 7년간의 버그 수정이 포함됩니다. <br>
대규모 Next.js 애플리케이션인 `vercel.com`에서 테스트하는 동안 다음과 같은 사실을 확인했습니다.

- 최대 53.3% 빨라진 로컬 서버 시작 속도
- 빠른 새로 고침으로 최대 94.7% 더 빠른 코드 업데이트

이 벤치마크는 대규모 애플리케이션에서 기대할 수 있는 성능 개선의 실질적인 결과입니다. 현재 다음 개발 테스트의 90%가 통과되었음으로 `--turbo`를 사용할 때 더 빠르고 안정적인 성능을 일관되게 경험할 수 있습니다. 테스트 통과율이 100%에 도달하면 다음 마이너 릴리스에서 터보팩을 안정 버전으로 전환할 예정입니다. 또한 사용자 정의 구성 및 에코 시스템 플러그인에 대한 웹팩 사용도 계속 지원할 예정입니다. 테스트 통과율을 [`areweturboyet.com`](https://areweturboyet.com/)에서 확인할 수 있습니다.

## Forms and Mutations

---

### Server Actions (Stable)

### Caching, Revalidating, Redirecting and more

## Partial Prerendering (Preview)

---

### Motivation

### Built on React Suspense

### Coming soon

## Metadata Improvements

---

## Next.js Learn Course

---

## Other Changes

---

## My opinion

---
