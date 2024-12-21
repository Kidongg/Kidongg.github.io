---
title: Cursor IDE를 내 입맛에 맞게 설정하기
date: 2024-12-21 17:00:00 +0900
categories: [Experience, Frontend]
tags: [Experience, Diary, Frontend, Study]
image: https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F12-cursor-ide%2Fimage-1.png?alt=media&token=29176e71-8190-4d09-8ceb-83379a8203f7
---

## Cursor IDE를 사용하는 이유 & 재설정하는 이유

### Cursor를 사용하는 이유

Cursor IDE(이하 Cursor)는 AI 기반 코드 에디터입니다. 사내 프론트엔드 개발을 혼자서 담당하고 있습니다. 주변에 물어볼 통로가 부족하다보니 자연스럽게 대화형 AI 서비스에 관심이 갔습니다. 기존에는 Copilot을 사용했었지만 현재는 Cursor를 사용하고 있습니다.

Cursor는 IDE 내에서 코드 자동 추천, 즉답형 질문, 대화형 질문 등의 기능을 제공합니다. Copilot과 차별점은 학습을 하는 데이터가 깃허브에 한정되어 있지 않다는 점입니다. ChatGPT, claude를 사용하다 보니 인터넷 정보를 광범위하게 학습하고 코드를 추천해줍니다. 또한 내 코드베이스를 학습한 상태에서 코드를 추천한다는 점이 특징입니다. 기존 시스템에 맞게 코드를 추천해주기 때문에 추천해주는 코드의 퀄리티가 높은 편입니다.

### 재설정하는 이유

불필요한 Extension을 지우고, Cursor에서 제공하는 기능을 더 효율적으로 사용하고 싶었습니다. Tab을 통한 코드 자동 추천 기능을 막고자함도 있었습니다.

## 내 입맛에 맞게 세팅하기

### 1. 기존 Cursor 제거하기

우선 모든 세팅을 초기화하고자 기존의 Cursor를 삭제하고 재설치했습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F12-cursor-ide%2Fimage-2.png?alt=media&token=93ec6b58-4f66-42cf-a2aa-c90a76eb1b2a)

### 2. Extension을 포함한 Cursor 제거하기

Extension을 포함한 기존 설정이 남아있는 문제가 있었습니다. mac을 사용하고 있었기 때문에 다음과 같은 명령어로 기존 설정을 포함하여 삭제를 재진행했습니다.

```powershell
rm -rf ~/Library/Application Support/Cursor
rm -rf ~/.cursor*
```

명령어 실행 후 기존의 Extension이 삭제되었음을 확인할 수 있었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F12-cursor-ide%2Fimage-3.png?alt=media&token=66f71767-655f-4eb7-823c-3840439c1f11)

### 3. Extension 설치하기

Cursor는 VSCode를 포크떠서 만든 프로젝트이기 때문에 VSCode의 모든 Extension을 사용할 수 있습니다. 심지어는 기존 VSCode에서 사용하고 있는 것을 복사해 사용할 수도 있습니다. 하지만 저는 불필요한 Extension을 제거하고, 저에게 맞는 Extension을 정리하는 목적도 있었기 때문에 재설치를 진행했습니다.

#### 1) Git Graph

- Git 그래프를 보고 Git 작업을 편하게 하는데 사용합니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F12-cursor-ide%2Fimage-4.png?alt=media&token=8183c39f-9873-4027-a2e7-0bf3ca94b9e5)

#### 2) Korean Language Pack for Visual Studio Code

- 한국어용 확장팩입니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F12-cursor-ide%2Fimage-5.png?alt=media&token=251c81d7-14a9-4195-be7c-40d25c089180)

#### 3) ESLint

- ESLint를 통합합니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F12-cursor-ide%2Fimage-6.png?alt=media&token=d9f5784d-70fd-405e-a5c9-22d60c2f5158)

#### 4) Prettier - Code formatter

- Prettier를 통합합니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F12-cursor-ide%2Fimage-7.png?alt=media&token=49a0227d-e2a8-4681-a31d-1df082f7493d)

#### 5) Error Lens

- ESLint에서 오류, 경고를 표현합니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F12-cursor-ide%2Fimage-8.png?alt=media&token=82473be6-10be-42ba-87c1-ec615817c258)

#### 6) Better Comments

- 경고, 질문 등의 주석을 강조하여 표현합니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F12-cursor-ide%2Fimage-9.png?alt=media&token=850e67cb-667c-42bc-955c-62ec06382cba)

#### 7) Import Cost

- Import로 가져오는 패키지의 크기를 표시합니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F12-cursor-ide%2Fimage-10.png?alt=media&token=61dfd12f-8bcc-4c93-9fb2-405f3c4e4ad1)

#### 8) Auto Rename Tag

- 페어링된 HTML 태그의 이름을 자동으로 바꿔줍니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F12-cursor-ide%2Fimage-11.png?alt=media&token=3ae08980-a5bf-49ff-8d80-a306edd2b3ee)

#### 9) Auto Close Tag

- HTML 닫기 태그를 자동으로 생성해줍니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F12-cursor-ide%2Fimage-12.png?alt=media&token=504c3343-55b3-4906-b6b7-fc058fdb04dd)

#### 10) Material Icon Theme

- 폴더, 파일 등의 아이콘을 디자인해줍니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F12-cursor-ide%2Fimage-13.png?alt=media&token=0339b84a-6c87-4cae-a770-1a7b4abde09c)

#### 11) Reactjs code snippets

- React 개발을 위한 코드 스니펫을 제공합니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F12-cursor-ide%2Fimage-14.png?alt=media&token=d71a93ed-a200-4806-b515-4479e2ad6e6a)

#### 12) Code Spell Checker

- 코드 내 맞춤법을 자동으로 검사합니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F12-cursor-ide%2Fimage-15.png?alt=media&token=c3a26b4a-2c2c-467b-a685-b3bb6b004ea2)

### 4. AI 기능 설정하기

그리고 AI 답변을 프롬프팅했습니다. 정확하고 단계적으로 설명하고, 한국어로 답변하도록 했습니다. 참고로 IDE 우측 상단에 설정 버튼이 존재합니다.

```
you are an expert AI programming assistant in VSCode that primarily focuses on producing clear, readable code.
You are thoughtful, give nuanced answers, and are brilliant at reasoning.
You carefully provide accurate, factual, and thoughtful answers, and you are a genius at reasoning.

1. Follow the user's requirements carefully and precisely.
2. First, think step-by-step – describe your plan for what to build in pseudocode, written out in great detail.
3. Confirm, then write the code!
4. Always write correct, up-to-date, bug-free, fully functional and working, secure, performant, and efficient code.
5. Focus on **readability** over performance.
6. Fully implement all requested functionality.
7. Leave **NO** to-dos, placeholders, or missing pieces.
8. Ensure the code is complete! Thoroughly verify the final version.
9. Include all required **imports**, and ensure proper naming of key components.
10. Be concise. Minimize any unnecessary explanations.
11. If you think there might not be a correct answer, say so. If you do not know the answer, admit it instead of guessing.
12. Always provide concise answers.
13. Please answer in Korean
```

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F12-cursor-ide%2Fimage-16.png?alt=media&token=a7d8e2b4-cab8-475f-a89f-3328e3784063)

AI 모델은 gpt-4o로 설정했습니다. Web을 통해 검색할때 gpt가 이점이 있을 것이라 생각했기 때문입니다. 사용 결과 gpt-4o가 claude-3.5-sonnet에 비해서 정확한 답변을 주는 것 같습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F12-cursor-ide%2Fimage-17.png?alt=media&token=adbda451-5e0e-4941-944b-e7b0eba78d16)

Tab을 통한 코드 추천 기능은 중단했습니다. 추천한 코드에 의지하는 스스로를 발견했기 때문입니다. 의도적으로 작성한 코드에 대해 생각할 기회를 열어 놓고 싶었습니다. 결과적으로 코드에 대해 생각하고 타이핑하는 시간이 늘어나 만족하고 있습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F12-cursor-ide%2Fimage-18.png?alt=media&token=8e3d12f4-bd5d-4aa4-afd5-e38659892415)

마지막으로 터미널에서 버튼을 통해 힌트를 주는 기능도 중단했습니다. 버튼이 터미널 메세지를 가려 불편하다고 느꼈기 때문입니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F12-cursor-ide%2Fimage-20.png?alt=media&token=ea6f8396-85a4-4e23-aec3-74a63ab4d66f)

## 결론

ChatGPT와 Cursor는 둘다 GPT-4o라는 모델을 사용함에도 동일한 질문에 대한 답변 퀄리티가 다르다는 점은 의아합니다. IDE 화면 내에서 실시간으로 응답해주는 프론트엔드 개발자 동료를 얻은 것 같습니다. 개발 생산성도 이전에 비해 체감상 50% 이상 올라갔다고 느끼고 있습니다.

한달에 20달러라는 가격이 저렴하다고는 할 수 없지만 그 이상의 가치를 제공하는 것 같습니다.

## 참고한 아티클

- [Cursor 공식 문서](https://docs.cursor.com/get-started/migrate-from-vscode){:target="\_blank"}
- [개발하는데 유용한 VSCode 확장팩 - 유료 IDE 못지않게](https://inpa.tistory.com/entry/VS-Code-%E2%8F%B1%EF%B8%8F-%EC%BD%94%EB%94%A9%EC%97%90-%EC%9C%A0%EC%9A%A9%ED%95%9C-%EB%8F%84%EA%B5%AC-%EC%B6%94%EC%B2%9C#%EC%BD%94%EB%93%9C_%EA%B0%80%EB%8F%85%EC%84%B1%EC%9D%B4_%EC%A2%8B%EC%95%84%EC%A7%80%EB%8A%94_%ED%99%95%EC%9E%A5%ED%8C%A9){:target="\_blank"}
- [Cursor IDE 동작원리 및 설치, 사용방법 - 외계공룡 개발블로그](https://chucoding.tistory.com/143){:target="\_blank"}
