---
title: 문득 생각이 들었다. 나는 왜 Next.js를 쓰고 있지?
date: 2024-08-15 14:00:00 +0900
categories: [Experience, Frontend]
tags: [Next.js, Ajax, React, Web, Build]
image: /v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnextjs-conf%2Fnextjs.png?alt=media&token=09247773-9707-4dd1-b3ca-3fe7f943497a
---

## Next.js를 사용하는 이유
프론트엔드 개발자라면 Next.js가 프론트엔드 시장의 점유율을 빠르게 점유하고 있다는 이야기를 들어보셨을 겁니다. Next.js를 사용할 줄 아는지가 프론트엔드 개발자 구인의 조건이 되기하죠(개인적으로 기술 스택보다 문제해결력이 더 중요하다고 생각합니다). 저 또한 스타트업에서 일을 하고 싶어서 공부를 시작한 케이스였습니다. 많은 스타트업에서는 왜 Next.js를 사용하는 것일까요? 문득 그 이유가 궁금했습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F3-why-i-use-next-js%2Ftrand.png?alt=media&token=f2786298-60a5-4b9f-b17e-60d73062dece)

Next.js는 프론트엔드 기능을 추상화해서 제공하는 React Framework입니다. 프론트엔드에서 담당하는 기능에는 Routing, Rendering, Fetching, SEO 등이 있습니다. 해당 기능을 React로만 구현하려면 다양한 서드파티 라이브러리를 제각각의 방식으로 사용해야합니다. Next.js를 사용하면 해당 기능을 통일된 방식으로 구현할 수 있습니다.

## Next.js가 등장한 배경
우선 Next.js가 등장한 배경을 짚고 넘어갈 필요가 있습니다. 1980~90년대 웹 프론트엔드는 SSR(Server Side Rendering)을 채택했습니다. 서버에서 HTML과 CSS를 렌더링한 완성된 파일을 클라이언트로 보낸 것이죠. 1999년 Ajax(Asynchronous JavaScript and XML)가 등장하면서 작은 단위로 데이터를 보낼 수 있게 되었습니다. 클라이언트 요청, 서버 응답의 형태로 HTML과 CSS를 렌더링한 완성된 파일이 아닌 작은 단위의 데이터를 보낼 수 있게 된 것이죠.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F3-why-i-use-next-js%2Fajax.png?alt=media&token=6b30afb0-b5c2-4b1f-92a0-d63ad2d87abd)

Angular, React가 등장하면서 CSR(Client Side Rendering)이 유행하기 시작했습니다. 클라이언트 요청, 서버 응답으로 받아온 작은 단위의 데이터를 사용해 클라이언트 측에서 렌더링을 하는 방식으로 말이죠. CSR은 유저와의 상호작용이 빠르다는 장점이 있었지만 1)성능 문제(큰 JS 번들을 서버로부터 받아옴)와 2)SEO(HTML 파일이 비어 있음)문제를 가지고 있었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F3-why-i-use-next-js%2Freact.png?alt=media&token=5a626a3a-d4bd-4721-a9c0-0d1e92daa064)

역사는 반복된다고 이러한 문제를 해결하기 위해 SSR로 돌아가려는 시도가 생기기 시작했습니다. Next.js는 이러한 SSR에 대한 개발자의 수요가 있는 상황에서 등장했습니다. 그러나 Next.js는 SSR 하나만 고집하는 것이 아니라 SSR과 CSR을 혼합하는 방식의 프로그래밍 패턴을 가져왔습니다. 이 뿐만 아니라 다양한 프론트엔드 기능(라우팅, 렌더링, 데이터 패칭, SEO)을 추상화하여 제공했습니다. 프론트엔드 개발자는 UI/UX와 비즈니스 로직에 집중할 수 있도록 하기 위함이죠. 

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F3-why-i-use-next-js%2Fnext-js.png?alt=media&token=a5f2619c-6804-4b57-a210-0de035fc6cb4)

## Next.js가 제공하는 기능(feat. 공식문서)
공식문서는 Next.js를 웹을 위한 React framework로 소개합니다. Next.js를 사용하면 React 컴포넌트의 강력한 성능으로 고품질 웹 애플리케이션을 제작할 수 있다면서 말이죠.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F3-why-i-use-next-js%2Fnext-js-docs.png?alt=media&token=1ae0c149-0013-4678-a1a8-3bf0372ed517)

그리고 “Web”과 “Build” 두 측면에서 기능을 강조합니다.

### Web

1. [Built-in Optimizations](https://nextjs.org/docs/app/building-your-application/optimizing/images){:target="\_blank"} : Image, Font, Script 자동 최적화를 통해 UX 및 핵심 웹 바이탈을 개선합니다.

2. [Dynamic HTML Streaming](https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming){:target="\_blank"} : App Router 및 React Suspense와 통합된 서버에서 UI를 즉시 스트리밍합니다.

3. [React Server Components](https://nextjs.org/docs/app/building-your-application/rendering){:target="\_blank"} : 클라이언트로 자바스크립트를 추가로 전송하지 않고도 컴포넌트를 추가합니다. 최신 React 기능을 기반으로 구축되었습니다.

4. [Data Fetching](https://nextjs.org/docs/app/building-your-application/data-fetching){:target="\_blank"} : React 컴포넌트를 비동기식으로 만들고 데이터를 기다립니다. Next.js는 서버와 클라이언트 데이터 불러오기를 모두 지원합니다.

5. [Css Support](https://nextjs.org/docs/app/building-your-application/styling){:target="\_blank"} : CSS Modules, Tailwind CSS, 인기 커뮤니티 라이브러리 지원 등 선호하는 도구로 애플리케이션의 스타일을 지정합니다.

6. [Client and Server Rendering](https://nextjs.org/docs/app/building-your-application/rendering){:target="\_blank"} : 페이지 단위의  Incremental Static Regeneration(ISR)을 비롯한 유연한 렌더링 및 캐싱 옵션을 제공합니다.

7. [Server Actions](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations){:target="\_blank"} : 함수를 호출하여 서버 코드를 실행합니다(API 엔드포인트 생략). 캐시된 데이터를 간편하게 재검증하고 한 번의 네트워크 왕복으로 UI를 업데이트합니다.

8. [Route Handlers](https://nextjs.org/docs/app/building-your-application/routing/route-handlers){:target="\_blank"} : API 엔드포인트를 구축하여 타사 서비스와 안전하게 연결하여 인증을 처리하거나 웹 훅을 들을 수 있습니다.

9. [Advanced Routing & Nested Layouts](https://nextjs.org/docs/app/building-your-application/routing){:target="\_blank"} : 고급 라우팅 패턴 및 UI 레이아웃 지원을 포함하여 파일 시스템을 사용하여 경로를 생성합니다.

10. [Middleware](https://nextjs.org/docs/app/building-your-application/routing/middleware){:target="\_blank"} : 요청 또는 응답을 제어합니다. 미들웨어를 사용하여 인증, 실험, 국제화를 위한 라우팅 및 액세스 규칙을 정의할 수 있습니다.

11. [Next.js 14](https://nextjs.org/blog/next-14){:target="\_blank"} : 풀스택의 강력한 기능을 프론트엔드에 적용할 수 있습니다. 릴리스 노트를 읽어보세요.

### Build
1. [React](https://react.dev/){:target="\_blank"} : 웹 및 네이티브 사용자 인터페이스를 위한 라이브러리입니다. Next.js는 서버 컴포넌트 및 액션을 포함한 최신 React 기능을 기반으로 구축되었습니다.

2. [Turbopack](https://turbo.build/){:target="\_blank"} : 자바스크립트 및 타입스크립트에 최적화된 증분 번들러입니다. Rust로 작성되어 Next.js에 내장되어 있습니다.

3. [Speedy Web Compiler(SWC)](https://swc.rs/){:target="\_blank"} : 차세대 고속 개발자 도구를 위한 확장 가능한 Rust 기반 플랫폼입니다. 컴파일과 코드 경량화 모두에 사용할 수 있습니다.

위의 기능들이 필요할때 Next.js의 공식 문서를 살펴보면 좋을 것 같습니다. Next.js에서는 프론트엔드뿐만 아니라 웹 전반에 대한 기능을 제공해주는 것을 알 수 있습니다. 스타트업에서 Next.js를 채택하는 이유가 여기 있었네요.

## 결론 : React를 더 알고 싶다..!
Next.js 공식문서에 React에 대한 내용이 많다는 것을 발견했습니다. 그래서 React를 Deep하게 공부하고 싶습니다. 앞으로 1)React Best practice 2)React 동작 원리에 대해 공부를 해야겠습니다. 모쪼록 Next.js 생태계와 특징을 알아갈 수 있는 유익한 시간이었습니다.

Next.js를 만드는 Vercel팀의 업데이트는 정말 빠릅니다(1년에 2~4회 릴리즈함). 커뮤니티도 활발하고, 피드백에 적극적이면서 말이죠. 그리고 React와 협업도 긴밀하게 진행하고 있습니다. `Early React`라는 수식어가 생길 정도로 말이죠. 그래서 Next.js가 매력적인 것 같습니다.

## 참고한 자료
- [Next,js가 등장한 기술적인 배경(프론트엔드 개발 역사와 함께 살펴보기)](https://www.youtube.com/watch?v=EwY6hbAxdV8){:target="\_blank"}
- [Next.js 공식문서](https://nextjs.org/){:target="\_blank"}
