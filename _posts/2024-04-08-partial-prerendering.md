---
title: Next.js Learn - 10. Partial Prerendering (Optional)
date: 2024-04-08 22:10:00 +0900
categories: [Frontend, Nextjs]
tags: [Next.js, Next.js conf, Next.js 14]
image: /v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnextjs-conf%2Fnextjs.png?alt=media&token=09247773-9707-4dd1-b3ca-3fe7f943497a
---

> 본 포스팅은 Next.js Learn의 [Streaming](https://nextjs.org/learn/dashboard-app/partial-prerendering){:target="\_blank"} 내용을 번역한 것입니다.
{: .prompt-info }

> Partial Prerendering은 Next.js 14에 도입된 실험적 기능입니다. 이 페이지의 내용은 기능의 안정화가 진행됨에 따라 업데이트될 수 있습니다. 실험적 기능을 사용하지 않으려면 이 장을 건너뛰는 것이 좋습니다. 이 장은 과정을 완료하는 데 필요하지 않습니다.
{: .prompt-tip }

## In this chapter
앞으로 다룰 주제는 다음과 같습니다.
- Partial Prerendering이란 무엇인가요?
- Partial Prerendering의 작동 방식

## Combining Static and Dynamic Content
현재 경로 내에서 [동적 함수](https://nextjs.org/docs/app/building-your-application/routing/route-handlers#dynamic-functions){:target="\_blank"}(예: `noStore()`, `cookies()`)를 호출하면 전체 경로가 동적이 됩니다. 이는 오늘날 대부분의 웹앱이 구축되는 방식과 일치하며 전체 애플리케이션 또는 특정 경로에 대해 정적 렌더링과 동적 렌더링 중 하나를 선택할 수 있습니다.

<br />

그러나 대부분의 경로는 완전히 정적이나 동적이지 않습니다. 정적 콘텐츠와 동적 콘텐츠가 모두 포함된 경로가 있을 수 있습니다. 소셜 미디어 피드가 있고 게시물은 정적이지만 게시물에 대한 좋아요는 동적인 것을 예로 들 수 있습니다.. 또는 제품 세부 정보는 정적이지만 사용자의 카트는 동적인 전자상거래 사이트를 예로 들 수 있습니다. 

<br />

대시보드 페이지로 돌아가서 어떤 구성 요소를 정적 요소와 동적 요소로 간주할 수 있을까요? 대시보드 경로를 어떻게 분할할지 생각해 보세요.

![img-description](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fpartial-prerendering%2Fimage_1.png?alt=media&token=28fe3f04-e722-4f80-8d96-1dd5511cc6a8)
_출처 : Next.js Learn_

- `<SideNav>` 컴포넌트 : 데이터에 의존하지 않으며 사용자에게 맞춤화되지 않으므로 정적일 수 있습니다.
- `<Page>` 컴포넌트 : 자주 변경되는 데이터에 의존하고 사용자에게 맞춤화되므로 동적일 수 있습니다.

## What is Partial Prerendering?
Next.js 14에는 Partial Prerendering이라는 새로운 렌더링 모델의 미리보기가 있습니다. Partial Prerendering은 정적 로딩 shell을 사용하여 경로를 렌더링하면서 일부 부분을 동적으로 유지할 수 있는 실험적인 기능입니다. 즉, 경로의 동적 부분을 분리할 수 있습니다.

![img-description](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fpartial-prerendering%2Fimage.png?alt=media&token=0661c494-9644-4835-b3a2-c83f729d74c7)
_출처 : Next.js Learn_

사용자가 경로를 방문하는 경우에
- 정적 경로 shell이 제공되므로 초기 로딩이 빨라집니다.
- shell은 동적 콘텐츠가 비동기식으로 로드되는 hole을 남깁니다.
- 비동기 hole은 병렬로 로드되므로 페이지의 전체 로드 시간이 단축됩니다.

Partial Prerendering은 정적 엣지 전송과 동적 기능을 결합한 것으로 정적 사이트 생성과 동적 전송의 장점을 결합하여 [웹 애플리케이션의 기본 렌더링 모델](https://vercel.com/blog/partial-prerendering-with-next-js-creating-a-new-default-rendering-model){:target="\_blank"}이 될 수 있는 잠재력이 있다고 생각합니다.

## How does Partial Prerendering work?
Partial Prerendering은 [Concurrent API](https://react.dev/blog/2021/12/17/react-conf-2021-recap#react-18-and-concurrent-features){:target="\_blank"}를 활용하고 [Suspense](https://react.dev/reference/react/Suspense){:target="\_blank"}를 사용하여 특정 조건(예: 데이터가 로드될 때까지)이 충족될 때까지 애플리케이션의 일부 렌더링을 연기합니다. 폴백은 다른 정적 콘텐츠와 함께 초기 정적 파일에 임베드됩니다. 빌드 시(또는 재검증 시) 경로의 정적 부분은 미리 렌더링되고, 나머지는 사용자가 경로를 요청할 때까지 연기됩니다.

<br />

컴포넌트를 Suspense로 래핑하면 컴포넌트 자체가 동적이 되는 것이 아니라(이 동작을 구현하기 위해 `unstable_noStore`를 사용했음을 기억하세요), Suspense가 경로의 정적 부분과 동적 부분 사이의 경계로 사용된가는 점에 주목할 필요가 있습니다. Partial Prerendering의 가장 큰 장점은 이를 사용하기 위해 코드를 변경할 필요가 없다는 것입니다. Suspense를 사용하여 경로의 동적 부분을 래핑하기만 하면 Next.js는 경로의 어느 부분이 정적이고 어느 부분이 동적인지 알 수 있습니다.

> 참고 : Partial Prerendering을 구성하는 방법에 대해 자세히 알아보려면 [Partial Prerendering(실험적) 문서](https://nextjs.org/docs/app/api-reference/next-config-js/partial-prerendering){:target="\_blank"}를 참조하거나 [Partial Prerendering 템플릿 및 데모](https://vercel.com/templates/next.js/partial-prerendering-nextjs){:target="\_blank"}를 사용해 보세요. 이 기능은 실험적인 기능으로 아직 프로덕션 배포에 사용할 준비가 되지 않았다는 점에 유의하세요.
{: .prompt-tip }

## Summary
대시보드에서는 애플리케이션에서 데이터 가져오기를 최적화하기 위해 몇 가지 작업을 수행했습니다.

1. 서버와 데이터베이스 사이의 지연 시간을 줄이기 위해 애플리케이션 코드와 동일한 영역에 데이터베이스를 생성했습니다.
2. React Server Component를 사용하여 서버에서 데이터를 불러왔습니다. 비용이 많이 드는 데이터 불러오기와 로직을 서버에 유지하여 클라이언트의 JavaScript 번들을 줄일 수 있습니다. 그리고 데이터베이스 비밀이 클라이언트에 노출되는 것을 방지할 수 있습니다.
3. SQL을 사용하여 데이터를 불러왔습니다. 필요한 데이터만 가져오기 때문에 각 요청에 대해 전송되는 데이터의 양과 데이터를 인메모리로 변환하는데 필요한 JavaScript의 양을 줄일 수 있습니다.
4. 필요한 경우 JavaScript로 데이터를 병렬적으로 불러올 수 있습니다.
5. 느린 데이터 요청으로 인해 전체 페이지가 차단되는 것을 방지하고, 사용자가 모든 페이지가 로드될때까지 기다리지 않고 UI와 상호 작용을 시작할 수 있도록 스트리밍을 구현했습니다.
6. 데이터 가져오기를 필요한 컴포넌트로 이동하여 Partial Prerendering를 활용한 동적인 경로를 분리할 수 있습니다.

다음 장에서는 데이터를 불러올 때 구현할 수 있는 두 가지 일반적인 패턴인 검색과 페이지네이션에 대해 살펴보겠습니다.