---
title: 판교 퇴근길밋업을 다녀와서(feat.FE 테스트 코드)
date: 2024-09-03 01:00:00 +0900
categories: [Experience, Conference]
tags: [Experience, Diary, Frontend, Evening Meetup, Inflearn]
image: https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F5-evening-meetup-storybook%2Fimage-1.png?alt=media&token=f1a9a6b5-95a4-4acb-86bf-44d6c3e58297
---

## 판교 퇴근길밋업을 신청한 이유

회사에서 기능 구현에 집중을 해왔습니다. 기능을 구현하는데만 해도 일정이 빠듯했기 때문입니다. 그러다보니 "페이지 중심" 개발을 진행했던 것 같습니다. 동일한 코드를 반복해서 사용하고(재사용성이 적음), 파일 내 코드량이 많아 디버깅이 어렵다고 느끼기도 했습니다. 이 불편함을 해결하기위한 방법을 찾던 중 "컴포넌트 중심" 개발이라는 개념을 알게 되었습니다.

스토리북은 컴포넌트 중심으로 사고하는 것을 도와줍니다. 그래서 스토리북에 대해 더 알고 싶었고 [FE 테스트 코드 : 스토리북 & 자동화 테스트와 함께 하는 컴포넌트 주도 프론트엔드 개발]을 주제로 한 퇴근길밋업을 신청했습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F5-evening-meetup-storybook%2Fimage-3.png?alt=media&token=2ee37981-9137-4e51-a173-794eb6b7bb87)

## 내용 요약

### 1. 프론트엔드 TDD

TDD를 도입하면 다양한 문제에 직면합니다. 테스트를 잘짜면 이러한 문제는 쉽게 해결이 됩니다. 따라서 개발자는 테스트를 잘짜는 것에 집중해야합니다.

- 생산성을 올려주는 테스트
- 사용자 입장에서 직관적으로 테스트
- 많은 경우의 수를 빠르게 검증할 수 있는 테스트
- 최소한으로 모킹하고 실제 환경과 유사한 테스트
- 웹 표준과 접근성에 기반해서 신뢰할 수 있는 테스트

장기적으로, 언젠가 도움이 되는 것이 아니라 지금 도움이 되는 테스트를 짜는 것이 중요합니다.

### 2. 테스트 방법론

- Integration : 컴포넌트를 렌더링하고 클릭, 입력 등의 상호작용을 통해 상태를 테스트(vitest + @testing-library, storybook 등)
- Unit : 순수함수, 클래스, API 등의 입출력을 테스트(Jest, @testing-library 등)
- E2E : 프로덕션과 최대한 동일한 환경에서 실제 백엔드 API로 사용자 동작과 흐름의 전과정을 테스트(playwright, cypress, nightwatch 등)

### 3. 스토리북

- 개념(feat. 리액트 공식문서) : 컴포넌트가 많은 시각적 state를 가지고 있다면 한 페이지에서 모두 보여주는게 편할 때가 있습니다. 이런 페이지는 자주 “living styleguide” 혹은 “storybook”으로 부릅니다.
- 특징
  - 컴포넌트를 프로덕트 페이지에서 따로 독자적으로 렌더링할 수 있다.
  - 다양한 컴포넌트의 상태를 한 눈에 볼 수 있다. 즉, 다양한 경우의 수를 쉽게 재현할 수 있다.
- 장점
  - 복잡한 모킹이 필요없어 테스트가 쉬워진다.
  - 자연스럽게 컴포넌트 사이에 경계를 명확히 긋게된다.
  - 디버깅에 시간을 낭비하지 않고 구현에 집중할 수 있다.

### 4. 네트워킹

저는 A조에 배정을 받았습니다. 저희 조는 "스토리북"에 관심이 있는 분들이 모여 있었습니다. 자연스레 스토리북에 대한 대화를 나누었습니다. 그 밖에는 개발과 관련한 소프트한 이야기를 나누었습니다.

- 프론트엔드 트랜드 : 도메인 주도 개발
- 프론트엔드 테스트
  - 절차 : 스토리북 개발 -> 테스트 -> 피드백 -> 배포
  - 방법
    - Unit : 로직 테스트 목적, JEST 활용
    - E2E : UI 테스트 목적, 스토리북 활용
- 프론트엔드 공부 방법 : 오픈소스(사이드 프로젝트 등)
- 디자인 시스템 레퍼런스 : [G마켓 디자인 시스템](https://gds.gmarket.co.kr/){:target="\_blank"}

## 인사이트

회사에서 컴포넌트 주도 개발을 적용하려 노력해야겠습니다. 스토리북도 도입해보고 싶습니다. 이를 통해 컴포넌트 주도 개발, 디자인 시스템, 테스트 편의성, 협업의 용이성을 가져가면 좋겠습니다.

네트워킹을 통해 페이지와 코드를 보며 스토리북에 대한 이야기를 나누면서 이론적, 추상적으로 가지고 있던 스토리북의 개념을 실무적, 직관적으로 이해할 수 있었습니다.

정말 유익한 시간이었고, 앞으로 네트워킹에 적극적으로 참여해야겠다는 생각을 했습니다. 서스럼없이 경험을 공유해주신 개발자분들께 감사의 인사를 드리고 싶습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F5-evening-meetup-storybook%2Fimage-2.jpeg?alt=media&token=e1117fce-f393-4fe3-b8e4-37fa502d730c)
