---
title: Kakao techmeet 간단 후기
date: 2024-04-13 15:10:00 +0900
categories: [Diary, Meetup]
tags: [Diary, Record, Kakao, Kakao techmeet]
image: https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fkakao-techmeet%2Fimage_1.png?alt=media&token=62d22ba7-05a7-4634-b1c1-978c39b57980
---

## 카카오 테크밋

이번주 목요일에 카카오에서 주최하는 테크밋에 다녀왔습니다. 카카오 테크밋은 카카오의 공개 기술 세미나입니다. 카카오 프론트엔드팀에서 개발중인 라이브러리에 대한 소개로 진행되었습니다. 발표한 라이브러리는 다음과 같습니다.

1. 이미지 뷰어
2. 웹 텍스트 에디터
3. 웹뷰 디버깅 도구

## 웹뷰 라이브러리

회사에서 서비스 운영 필요한 라이브러리를 직접 개발하는게 신기했습니다(역시 카카오!). 모든 세션이 재미있고 의미 있었지만 개인적으로 [웹뷰 디버깅 도구]세션이 기억에 남습니다. 이전부터 웹뷰에 관심이 있었기 때문입니다.
<br />

웹뷰란 네이티브 앱에 내재되어 있는 웹 브라우저입니다. Andriod, IOS 환경에 대한 일원화된 개발이 가능하고, 앱 스토어에 종속되어 있지 않기 때문에 즉시 배포가 가능하다는 장점이 있습니다.
<br />

웹뷰는 데스크탑, 모바일 두 환경에서 디버깅이 이루어집니다. 데스크탑은 브라우저 개발도구를 통해 쉽게 디버깅을 할 수 있지만, 모바일은 디버깅을 하기 위해 거쳐야하는 과정이 많습니다. 특히나 IOS에서는 더욱 그러합니다.
<br />

웹뷰 디버깅시 발생하는 불편함을 개선하기 위해 카카오에서는 Chrome Devtools를 활용한 라이브러리를 만들고 있었습니다. SDK 스크립트 삽입을 통해 OS 구분 없이 간편하게 세팅이 가능하고, Chrome Devtools를 통해 디버깅을 할 수 있는 장점이 있었습니다. 웹뷰와 Chrome Devtools의 통신과정에서 SDK와 Web Socket을 거치는 것이 특징이었습니다. 통신 프로토콜도 독특한 것이었는데, 이름은 기억이 나지 않습니다.

## 다녀오며 느낀점

테크밋을 들으면서 이해되는 것도 있고, 이해가 되지 않는 것도 있었습니다. 프론트엔드 개발자로써 더 공부가 필요한 부분인 것 같습니다. 그럼에도 불구하고 프론트엔드 개발에 있어 다양한 분야가 있다는 것을 알게 되었습니다. 열정있는 동료분들이 많다는 사실도 알게 되었습니다. 좋은 자극을 얻어가는 시간이었습니다.

무엇보다 귀여운 선물들을 받아서 좋았습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fkakao-techmeet%2Fimage_2.jpg?alt=media&token=741549f4-9dcf-422f-a1cd-73532b220b15)
