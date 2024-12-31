---
title: Vercel Blog - Understanding React Server Components (2023.08.01) 번역
date: 2025-01-01 00:05:00 +0900
categories: [Translation, Vercel Blog]
tags: [Vercel, Next.js, React, Server Components]
image: /v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnextjs-conf%2Fvercel.jpg?alt=media&token=14eb23a1-68d5-4639-9ed3-c1e82be53d9e
---

> 본 포스팅은 Vercel의 공식 블로그의 [리액트 서버 컴포넌트 이해하기](https://vercel.com/blog/understanding-react-server-components){:target="\_blank"}를 번역한 것입니다.
{: .prompt-info }

## Learn the fundamentals of React Server Components, to better understand why (and when) to adopt.

[React Server Components](https://react.dev/blog/2020/12/21/data-fetching-with-react-server-components){:target="\_blank"}(RSC)는 순수한 렌더링 라이브러리를 넘어 프레임워크 내에서 데이터 불러오기와 통신을 통합하는 등 리액트의 기본을 강화합니다.

아래에서는 RSC를 만들어야 하는 이유, RSC의 기능 및 사용 시기에 대해 설명합니다. 또한 Next.js가 앱 라우터를 통해 RSC 구현 세부 사항을 완화하고 향상시키는 방법에 대해서도 설명합니다.

## Why do we need Server Components?

리액트 이전의 세상을 상상해보세요. PHP와 같은 언어에서는 클라이언트와 서버 간의 관계가 더 긴밀했습니다. 모놀리식 아키텍처에서는 제작 중인 페이지 내에서 바로 서버에 접속하여 데이터를 호출할 수 있었습니다. 하지만 팀 간 종속성과 높은 트래픽 수요로 인해 애플리케이션을 확장하기가 어렵다는 단점도 있었습니다.

리액트는 컴포저빌리티와 기존 코드베이스에 점진적으로 적용하기 위해 만들어졌습니다. 풍부한 상호작용을 갈망하는 세상에 부응하기 위해 클라이언트와 서버의 문제를 분리하여 프론트엔드를 훨씬 더 유연하게 구성할 수 있게 했습니다. 이는 특히 팀에게 중요한 기능으로, 각각 다른 개발자가 만든 두 개의 리액트 컴포넌트가 동일한 프레임워크 내에서 함께 작동할 수 있었습니다.

이를 달성하기 위해 리액트는 기존 웹 표준을 기반으로 혁신을 이루어야 했습니다. 지난 10년간 MPA(Multi Page Applications)와 SPA(Single Page Applications)라는 클라이언트 측과 서버 측 렌더링 간의 발전을 거치면서 빠른 데이터 제공, 풍부한 상호 작용, 뛰어난 개발자 경험 유지라는 [목표를 유지했습니다](https://github.com/reactwg/server-components/discussions/5){:target="\_blank"}.

### What did SSR and React Suspense solve?

지금의 Server Components가 있기까지 해결해야 할 다른 중요한 문제들이 있었습니다. RSC의 필요성을 더 잘 이해하려면 먼저 SSR(Server Side Rendering)과 Suspense의 필요성을 파악하는 것이 도움이 됩니다.

SSR은 초기 페이지 로딩에 초점을 맞춰 미리 렌더링된 HTML을 클라이언트에 전송하고, 클라이언트가 일반적인 리액트 앱처럼 작동하기 전에 다운로드한 자바스크립트로 hydrated해야 합니다. 또한 SSR은 페이지로 직접 이동할 때 한 번만 발생합니다.

SSR만 사용하면 사용자는 HTML을 더 빨리 얻을 수 있지만, 자바스크립트와 상호 작용하기 전에 'all-or-nothing'의 워터폴을 기다려야 합니다.

- 모든 데이터는 서버에서 가져와야 표시할 수 있습니다.
- 모든 자바스크립트는 서버에서 다운로드해야 클라이언트에서 hydrated할 수 있습니다.
- 모든 hydration이 클라이언트에서 완료되어야 상호 작용할 수 있습니다.

이 문제를 해결하기 위해 리액트는 서버 측 HTML 스트리밍과 클라이언트에서 선택적 hydration을 허용하는 [Suspense를 만들었습니다](https://github.com/reactwg/react-18/discussions/37){:target="\_blank"}. 컴포넌트를 `<Suspense>`로 래핑하면 서버에 해당 컴포넌트의 렌더링과 hydration의 우선순위를 낮추도록 지시하여 더 무거운 컴포넌트에 의해 차단되지 않고 다른 컴포넌트가 로드되도록 할 수 있습니다.

이렇게 하면 상황이 크게 개선되지만 여전히 몇 가지 문제가 남아 있습니다.

- 컴포넌트를 표시하기 전에 전체 페이지의 데이터를 서버에서 가져와야 합니다. 이 문제를 해결할 수 있는 유일한 방법은 `useEffect()` 훅을 통해 클라이언트 측에서 데이터를 가져오는 것인데, 이는 서버 측 가져오기보다 왕복 시간이 더 길고 컴포넌트가 렌더링되고 hydrated된 후에만 발생합니다.
- 모든 페이지 자바스크립트는 비동기식으로 브라우저에 스트리밍되더라도 결국 다운로드됩니다. 앱의 복잡성이 증가하면 사용자가 다운로드하는 코드의 양도 증가합니다.
- hydration 최적화에도 불구하고 사용자는 클라이언트 측 자바스크립트를 다운로드하여 해당 컴포넌트에 대해 구현할 때까지 컴포넌트와 상호 작용할 수 없습니다.
- 대부분의 자바스크립트 컴퓨팅 비중은 여전히 다양한 기기에서 실행되는 클라이언트에서 발생합니다. 더 강력하고 예측 가능한 서버로 옮기는 것은 어떨까요?

![img-description](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F13-understanding-react-server-components%2Fimage-1.png?alt=media&token=f61e542b-67cd-4b20-899f-f50c2a7694da)

React Server Components가 없는 Next.js에서 데이터를 가져오려면 추가 API 계층이 필요합니다.

웹 표준이 자바스크립트 프레임워크를 따라잡으면서 또 한 번의 도약이 필요한 시점입니다. 더 빠른 애플리케이션을 작성할 수 있는 더 좋은 방법이 있습니다.

## What do React Server Components do?

위의 문제를 해결하기 위해 리액트는 Server Components를 만들었습니다. RSC는 개별적으로 데이터를 가져와 서버에서 전적으로 렌더링하고, HTML 결과는 필요에 따라 다른 서버 및 클라이언트 컴포넌트와 인터리빙하면서 클라이언트 측 리액트 컴포넌트 트리로 스트리밍됩니다.

이 프로세스를 통해 클라이언트 측에서 다시 렌더링할 필요가 없으므로 성능이 향상됩니다. 모든 클라이언트 컴포넌트의 경우 컴퓨팅 부하가 클라이언트와 서버 간에 공유되므로 RSC가 스트리밍되는 동안 hydration이 동시에 발생할 수 있습니다.

다시 말해, 훨씬 더 강력하고 물리적으로 데이터 소스에 가까운 서버가 컴퓨팅 집약적인 렌더링을 처리하고 인터랙티브한 코드만 클라이언트에 전송하는 것입니다.

상태 변경으로 인해 RSC를 다시 렌더링해야 하는 경우 서버에서 새로 고침하고 하드 새로 고침 없이 기존 DOM에 원활하게 병합합니다. 따라서 뷰의 일부가 서버에서 업데이트되더라도 클라이언트 상태는 유지됩니다.

### RSCs: Performance and bundle size

RSC는 클라이언트 측 자바스크립트 번들의 크기를 줄이고 로딩 성능을 개선하는 데 도움이 될 수 있습니다.

기존에는 애플리케이션을 탐색하는 동안 클라이언트가 모든 코드와 데이터 종속성을 다운로드한 다음 실행했습니다. [코드 분할](https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating#1-code-splitting){:target="\_blank"} 기능이 있는 리액트 프레임워크가 없다면 이는 사용자가 현재 있는 페이지에 필요하지 않은 불필요한 코드를 사용자에게 전송하는 것을 의미하기도 합니다.

그러나 RSC는 앱의 데이터 소스에 더 가까운 서버에서 모든 종속성을 해결합니다. 또한 서버에서만 코드를 렌더링하므로 클라이언트 기기(예: 휴대폰)보다 이 작업이 훨씬 빠릅니다. 그런 다음 리액트는 처리된 결과와 클라이언트 컴포넌트만 브라우저로 전송합니다.

즉, Server Components를 사용하면 초기 페이지 로딩이 더 빠르고 간결해집니다. 기본 클라이언트 측 런타임은 캐시가 가능하고 크기가 예측 가능하며 애플리케이션이 커져도 증가하지 않습니다. 애플리케이션에 클라이언트 컴포넌트를 통해 더 많은 클라이언트 측 상호 작용이 필요할 때 주로 추가 유저를 대면하는 자바스크립트가 추가됩니다.

### RSCs: Interleaving and Suspense integration

RSC는 클라이언트 측 코드와 완전히 인터리빙되므로 클라이언트 컴포넌트와 서버 컴포넌트가 동일한 리액트 트리에서 렌더링될 수 있습니다. 대부분의 애플리케이션 코드를 서버로 이동시킴으로써 RSC는 클라이언트 측 데이터 불러오기 워터폴을 방지하고 서버 측 데이터 종속성을 빠르게 해결합니다.

기존의 클라이언트 측 렌더링에서 컴포넌트는 비동기 작업이 완료되기를 기다리는 동안 리액트 Suspense를 사용하여 렌더링 프로세스를 "일시 중지"하고 폴백 상태를 표시합니다. RSC를 사용하면 데이터 가져오기와 렌더링이 모두 서버에서 이루어지므로 Suspense는 서버 측에서도 대기 기간을 관리하여 총 왕복 시간을 단축하여 폴백 및 완료된 페이지의 렌더링 속도를 높입니다.

클라이언트 컴포넌트는 [초기 로드 시 여전히 SSR이 적용](https://github.com/reactwg/server-components/discussions/4){:target="\_blank"}된다는 점에 유의해야 합니다. RSC 모델은 SSR이나 Suspense를 대체하는 것이 아니라 이들과 함께 작동하여 애플리케이션의 모든 부분을 사용자에게 필요한 대로 제공합니다.

![img-description](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F13-understanding-react-server-components%2Fimage-2.png?alt=media&token=df0dcb09-1220-4634-a75a-9e61535ac184)

React Server Components가 포함된 Next.js에서는 동일한 컴포넌트에서 데이터 불러오기와 UI 렌더링을 수행할 수 있습니다. 또한 서버 액션은 페이지에서 자바스크립트가 로드되기 전에 사용자가 서버 측 데이터와 상호 작용할 수 있는 방법을 제공합니다.

### RSCs: Limitations

Server Components에 대해 작성된 모든 코드는 직렬화 가능해야 하며, 이는 `useEffect()` 또는 상태와 같은 라이프사이클 훅을 사용할 수 없음을 의미합니다.

그러나 잠시 후에 설명할 Server Actions을 통해 클라이언트에서 서버와 상호 작용할 수 있습니다.

또한 RSC는 웹소켓을 통한 지속적인 업데이트를 지원하지 않습니다. 이러한 경우 클라이언트 측 가져오기 또는 폴링 접근 방식이 필요합니다.

{% include embed/youtube.html id='g5BGoLyGjY0' %}

Vercel의 시니어 개발자인 Delba de Oliveira가 리액트 코어 팀의 Andrew Clark, Sebastian Markbåge와 함께 리액트, 서버 컴포넌트 등에 대해 이야기합니다.

## How to use React Server Components

RSC의 장점은 작동 원리를 완전히 알지 못해도 이를 활용할 수 있다는 점입니다. 가장 완벽한 기능으로 RSC를 구현하는 Next.js 13.4에 도입된 앱 라우터에서는 모든 컴포넌트가 기본적으로 서버 컴포넌트입니다.

`useEffect()` 또는 state와 같은 라이프사이클 이벤트를 사용하려면 클라이언트 컴포넌트를 엮어야 합니다. 클라이언트 컴포넌트를 선택하려면 컴포넌트 상단에 `"use client"`를 작성하면 되지만, 더 자세한 내용은 문서를 확인하는 것이 좋습니다.

### Balancing Server ans Client Components

RSC는 클라이언트 컴포넌트를 대체하기 위한 것이 아니라는 점에 유의해야 합니다. 정상적인 애플리케이션은 동적 데이터 가져오기에는 RSC를, 풍부한 상호작용을 위해서는 클라이언트 컴포넌트를 모두 활용합니다. 문제는 각 컴포넌트를 언제 사용할지 결정하는 것입니다.

개발자는 서버 측 렌더링과 데이터 가져오기에는 RSC를 활용하고 로컬 인터랙티브 기능과 사용자 경험에는 클라이언트 컴포넌트를 활용하는 것을 고려하세요. 적절한 균형을 유지하면 고성능의 효율적이고 매력적인 애플리케이션을 만들 수 있습니다.

가장 중요한 것은 비표준 환경에서 애플리케이션을 계속 테스트하는 것입니다. 느린 컴퓨터, 느린 휴대폰, 느린 Wi-Fi를 에뮬레이션하면 올바른 구성 요소 조합으로 앱이 얼마나 더 잘 작동하는지 보고 놀랄 수도 있습니다.

RSC는 과도한 클라이언트 측 자바스크립트로 사용자에게 부담을 주는 문제에 대한 완전한 해결책은 아니지만, 사용자 디바이스에 컴퓨팅 부하를 가중시킬 시점을 선택할 수 있는 권한을 부여하는 것은 분명합니다.

### Improved data fetching with Next.js

RSC는 서버에서 데이터를 가져오기 때문에 백엔드 데이터에 안전하게 액세스할 수 있을 뿐만 아니라 서버와 클라이언트의 상호 작용을 줄여 성능을 향상시킵니다. [Next.js 개선 사항](https://nextjs.org/docs/app/building-your-application/data-fetching?utm_source=vercel_site&utm_medium=web&utm_campaign=understanding_rsc){:target="\_blank"}과 함께 RSC는 스마트 데이터 캐싱, 단일 왕복 다중 가져오기, `fetch()` 요청 중복 제거를 지원하여 클라이언트 측 데이터 전송의 효율성을 극대화합니다.

가장 중요한 것은 서버에서 데이터를 가져오면 요청이 서로 쌓여 사용자가 계속 진행하기 전에 순차적으로 해결해야 하는 클라이언트 측 데이터 가져오기 워터폴을 방지하는 데 도움이 된다는 점입니다. 서버 측 가져오기는 전체 클라이언트를 차단하지 않고 훨씬 더 빠르게 해결되므로 오버헤드가 훨씬 적습니다.

또한 개별 컴포넌트를 충분히 세밀하게 제어할 수 없고 데이터를 과도하게 가져오는 경향이 있던 `getServerSideProps()` 및 `getStaticProps()`와 같은 Next.js 전용 페이지 수준 메서드가 더 이상 필요하지 않습니다(사용자가 페이지로 이동하면 실제로 어떤 컴포넌트와 상호작용했는지에 관계없이 모든 데이터를 가져왔습니다).

이제 Next.js 앱 라우터에서 가져온 모든 데이터는 기본적으로 정적이며 빌드 시점에 렌더링됩니다. 그러나 이는 쉽게 변경할 수 있습니다. Next.js는 `fetch` 옵션 객체를 확장하여 캐싱 및 규칙 재검증에 유연성을 제공합니다.

`{next: {revalidate: number}}` 옵션을 사용하여 정적 데이터를 설정된 간격으로 또는 백엔드 변경이 발생할 때 새로 고칠 수 있으며(Incremental Static Regeneration), `{cache: 'no-store'}` 옵션을 동적 데이터에 대한 가져오기 요청에 전달할 수 있습니다(SSR).

이 모든 것이 Next.js 앱 라우터 내의 리액트 서버 컴포넌트를 효율적이고 안전하며 동적인 [데이터 불러오기를 위한 강력한 도구로 만들며](https://vercel.com/blog/nextjs-app-router-data-fetching){:target="\_blank"}, 고성능 사용자 경험을 제공하기 위해 기본적으로 캐시됩니다.

### Server Actions: React’s first steps into mutability

RSC의 맥락에서 Server Actions는 서버 측의 RSC에서 정의한 다음 서버/클라이언트 경계를 넘어 전달할 수 있는 함수입니다. 사용자가 클라이언트 측에서 앱과 상호 작용할 때 서버 측에서 안전하게 실행되는 서버 액션을 직접 호출할 수 있습니다.

이 접근 방식은 클라이언트와 서버 간에 원활한 RPC([Remote Procedure Call](https://en.wikipedia.org/wiki/Remote_procedure_call){:target="\_blank"}) 환경을 제공합니다. 서버와 통신하기 위해 별도의 API 경로를 작성하는 대신 클라이언트 컴포넌트에서 서버 액션을 직접 호출할 수 있습니다.

Next.js 앱 라우터는 전적으로 스마트 데이터 캐싱, 재검증 및 변경을 중심으로 구축되었다는 점도 명심하세요. Next.js의 Server Actions는 탐색을 통해 클라이언트 캐시 무결성을 유지하면서 서버에 대한 동일한 왕복 요청으로 캐시를 변경하고 리액트 트리를 업데이트할 수 있음을 의미합니다.

특히 Server Actions은 데이터베이스 업데이트 또는 양식 제출과 같은 작업을 처리하도록 설계되었습니다. 예를 들어 양식을 점진적으로 개선할 수 있으므로 자바스크립트가 아직 로드되지 않은 경우에도 사용자가 양식과 상호 작용할 수 있으며 Server Actions이 양식 데이터의 제출 및 처리를 처리합니다.

Server Actions이 제공하는 점진적인 개선과 API에 대한 개발 작업 제거의 기회는 접근성, 사용성 및 개발자 경험 측면에서 모두 훌륭합니다.

### Let Next.js do the heavy lifting

Next.js는 Server Components, Server Actions, Suspense, Transitions 및 RSC 릴리스와 함께 변경된 모든 것을 위한 전체 리액트 아키텍처를 통합한 최초의 프레임워크입니다.

여러분이 제품 구축에 집중하는 동안 Next.js는 전략적 스트리밍과 스마트 캐싱을 사용하여 애플리케이션 렌더링이 차단되지 않고 동적 데이터를 최고 속도로 제공할 수 있도록 합니다.

Next.js는 안정성, 신뢰성 및 이전 버전과의 호환성을 유지하면서 새로운 리액트 기능의 최첨단을 유지하기 위해 노력하고 있습니다. 앞으로도 모든 범위의 프로젝트에 유연성과 확장성을 유지하면서 팀이 빠르게 반복할 수 있도록 스마트한 기본값을 제공할 것입니다.

## Where do you go from here?

다시 정리해 보겠습니다. React Server Components는 컴포넌트 내에서 바로 서버와 상호작용하는 네이티브 방식을 제공하여 동적 데이터와 상호작용하는 데 필요한 코드와 인지 부하를 모두 줄여줍니다. 클라이언트 컴포넌트는 이전과 마찬가지로 완전한 기능을 유지하며 완전히 사용할 수 있습니다. 여러분의 새로운 임무는 각각의 컴포넌트를 언제 사용할지 선택하는 것입니다.

이 주제에 대한 자세한 안내는 [Next.js 문서](https://nextjs.org/docs){:target="\_blank"}를 참조하세요. 또한 [앱 라우터 플레이그라운드](https://vercel.com/templates/next.js/app-directory){:target="\_blank"}에서 바로 시작하여 차이점을 직접 느껴볼 수 있습니다.

React Server Components에 대한 더 많은 글에 관심이 있으시다면, 이 글들이 특히 도움이 될 것입니다.

- ["We migrated 50,000 lines of code to React Server Components" - Mux](https://www.mux.com/blog/what-are-react-server-components){:target="\_blank"}

- ["Speeding up the dbt™ docs by 20x with React Server Components" - Dagster](https://dagster.io/blog/dbt-docs-on-react){:target="\_blank"}

- ["Next.js App Router and Sanity CMS in action" - Formidable](https://formidable.dev/blog/2023/powering-our-website-s-evolution-next-js-app-router-and-sanity-cms-in-action/){:target="\_blank"}

팀의 애플리케이션을 앱 라우터 및 React Server Components로 마이그레이션하는 데 직접적인 도움이 필요하시면 언제든지 [문의해 주세요](https://vercel.com/contact/sales){:target="\_blank"}.
