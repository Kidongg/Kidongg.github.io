---
title: 사이드 프로젝트로 한단계 도약하기
date: 2025-04-07 18:00:00 +0900
categories: [Experience, Retrospect]
tags: [Experience, Diary]
image: https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F19-side-project%2Fimage-1.png?alt=media&token=832e6e62-bdf7-4f35-989f-e930cd3bda70
---

## 사이드 프로젝트를 시작한 이유

신입 개발자에게 사수가 중요하다는 이야기를 종종 듣곤합니다. 문제 해결을 위해 비슷한 고민을 하고, 개발하면서 겪는 시행착오를 줄일 수 있기 때문일 것입니다.

저는 신입때부터 지금까지 혼자서 프론트엔드 개발을 해오고 있습니다. 동료가 아예 없는 것은 아니고 [Cursor](https://www.cursor.com/){:target="\_blank"}와 [CodeRabbit](https://www.coderabbit.ai/){:target="\_blank"}이 있긴 합니다. 그럼에도 불구하고 마음속 한곳에는 제 코드에 대한 의문이 자리잡고 있습니다.

어떻게 하면 의문을 해결할 수 있을까요? 의문을 해결하기 위해 [컨퍼런스 참여](https://kidongg.github.io/posts/infcon-2024/){:target="\_blank"}, [모각코 운영](https://sepia-session-b25.notion.site/51d83b079bdb4e088ae9d559866cc477?v=2bcb73796bb240e0890d6f84bf06a55a){:target="\_blank"}, [스터디 참여](https://kidongg.github.io/posts/optimization-3/){:target="\_blank"}, [인터넷 강의](https://kidongg.github.io/posts/react-clean-code-study/){:target="\_blank"}와 같은 다양한 시도를 했었습니다. 사이드 프로젝트도 여러 시도 중 하나입니다.

## 멤버들과의 만남 그리고 프로젝트 선정

F-lab에서 수익화 프로젝트를 함께할 개발자를 모집하고 있었습니다. 여러 모집공고 중에서 "언니네 이발관"이 결성된 계기를 소개한 팀에 관심이 갔습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F19-side-project%2Fimage-2.png?alt=media&token=418fc809-5e68-47cf-b077-36bf11c9c30c)

미팅 이후 "저지르고 보는 팀"에 프론트엔드 개발자로 참여를 했습니다. 처음에는 아이디어 회의에 많은 시간을 보냈었습니다. 게이미피게이션 서비스 -> 지하철 빈 좌석 서비스의 흐름을 거쳐 시장 분석을 통과한 키보드 공제 서비스로 의견이 모아졌습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F19-side-project%2Fimage-3.png?alt=media&token=a728757e-7677-43ff-8cab-736c6d4e3070)

## 개발자로써 협업하는 방법을 배우다

MVP 단계에서 필요한 기능을 정리하고, 2주 스프린트 방식으로 테스크를 나누어 작업을 진행했습니다. 프론트엔드 기술 스택은 TypeScript, Next.js, SCSS, Zustand, Tanstack Query를 사용했습니다. 렌더링은 기본적으로 서버에서 하고, 필요시 클라이언트에서 하는 방법을 채택했습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F19-side-project%2Fimage-6.png?alt=media&token=94b020e2-5a3f-4934-b003-f72d6188510a)

PR마다 코드 리뷰도 진행했습니다. 처음에는 무엇을 리뷰해야하는지도 몰랐고, 그래서 실력이 까발려지는게 아닌가 하는 두려움이 있었습니다. 정확히는 코드 리뷰를 잘 할 자신이 없었습니다. 그럼에도 불구하고 코드에 리뷰가 달렸을때 기분이 정말 좋았습니다. 협업을 하는 느낌을 느껴서이기도 했고, 업무적으로 많은 도움이 되어서기도 했습니다.

협업의 경험은 함께 개발하는 동료를 배려하면서 개발해야한다는 점을 알게 해주었습니다. 피드백 이후 파일명, 변수명을 지을때 그 의미를 생각해보게 되었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F19-side-project%2Fimage-7.png?alt=media&token=00f705ab-8a0b-4989-9381-5af64546a09a)

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F19-side-project%2Fimage-8.png?alt=media&token=7fc3db61-ed46-4430-aeca-7e1fc06ef281)

머지를 하면서 충돌이 났을때는 동료에게 정말 죄송했습니다. 다행인 것은 이러한 경험을 사이드 프로젝트에서 했다는 점일 것입니다. 충돌 경험을 통해 버전에 주의하여 브랜치를 따야한다는 점을 배웠습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F19-side-project%2Fimage-9.png?alt=media&token=749a3181-a357-411f-a662-a74166f260a2)

그 밖에도 동료의 코드와 제 코드를 비교해보면서 개선해야할 점에 대해서도 생각해 볼 수 있었습니다. 이러한 경험은 업무에 도움이 되었습니다. 실제로 회사 코드에 프로젝트에서 사용했던 라이브러리를 적용했고, 파일구조를 일관성 있게 바꾸었고, 재사용 가능한 코드에 대해 컴포넌트화, 모듈화를 진행했습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F19-side-project%2Fimage-10.png?alt=media&token=2dabefdf-bc2a-44ae-9981-0fb5d74b8af1)

## 퇴근 후 다른 작업을 하는 것은 쉽지 않다, 끝이 보인다

퇴근하고 또 다른 개발을 하는 것은 쉬운 일이 아니었습니다. 무엇보다 피곤한 상태에서 집중력을 끌어올리는 것이 힘들었습니다. 다른 일이 있다면 우선순위에서 밀렸었는데요, 그러다보니 호흡이 길어지고 이는 컨택스트 스위칭에 리소스를 들어가게끔 했습니다.

한편으로는 수익화가 얼마나 어려운 일인지 느꼈습니다. 고객 분석을 통한 서비스 방향성 및 전략, 시장 분석을 통한 수익화 전략, 언라인된 동기부여를 통한 효율적인 스프린트 운영, 심지어는 운까지 들어맞아야 수익을 창출할 수 있다는 점을 배웠습니다.

긴 호흡이었음에도 불구하고 한 명의 이탈자도 없이 프로젝트를 진행하고 있습니다. 현재는 서비스를 홍보하고 유저의 반응을 보고 있는 단계인데요. 불안하긴 하지만 유의미한 데이터가 나오면 좋겠습니다.

모쪼록 메이커로써 그리고 개발자로써 시야를 넓힐 수 있었던 소중한 경험이었습니다. 함께 해준 팀원분들에게 무한한 감사를 드립니다.

## 키보드 공제 정보를 찾고 계신가요? SoKey를 소개합니다!

마지막으로 키보드 공제 정보를 제공하는 SoKey를 소개합니다. 흩어져있는 다양한 키보드 공제 정보를 모아 전해드리는 서비스입니다. Sokey는 다음 3가지 기능을 제공합니다.

1. 국내/해외 다양한 벤더사의 공제 정보를 모아 한곳에 보여드립니다.
2. 공제 시작, 마감 소식 알림을 통해 공제를 놓치지 않게 도와드립니다.
3. 해당 공제 제품을 주문 할 수 있는 벤더사 링크를 직접 제공해 드립니다.

- URL : [https://www.sokey.kr/](https://www.sokey.kr/){:target="\_blank"}
- 소스코드 : [https://github.com/Kidongg/keyboard-notifier-fe](https://github.com/Kidongg/keyboard-notifier-fe){:target="\_blank"}
