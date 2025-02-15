---
title: 코드 레빗으로부터 코드 리뷰 받기
date: 2025-02-14 23:00:00 +0900
categories: [Experience, Frontend]
tags: [Experience, Diary, Frontend, Study]
image: https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F16-code-rabbit%2Fimage-1.png?alt=media&token=bfc2e4de-5f2d-4c08-9161-98513b4c8f9e
---

## 코드 레빗을 사용하게 된 이유

회사의 코드가 비대해지고, 이를 유지, 보수를 하는 시점이 오면서 코드를 관리하는 방법에 관심이 갔습니다. 그러다 자연스럽게 코드를 잘 관리하기 위해 내가 할 수 방법들에 대해 고민을 하게 되었습니다.

코드 리뷰는 양질의 코드를 작성하는데 도움이 되는 방법론입니다. 코드 리뷰는 내가 작성한 코드를 다시 생각하고, 수정을 하는 과정을 통해 코드의 품질을 높일 수 있다는 장점이 있습니다. 한편으로는 코드 리뷰를 위한 시간과 노력이 꽤 많이 든다는 단점도 존재합니다.

AI가 코드 리뷰를 해준다면 코드 리뷰의 장점만 취할 수 있을 것이라 생각했습니다. 그러던 중 링크드인에서 한 개발자 분이 코드 레빗 서비스를 소개하던 것이 생각이 났고, 이를 업무에 적용해봐야겠다는 생각을 했습니다.

코드 레빗은 PR에 대한 간단한 설명과 리뷰 해주는 서비스입니다. 리뷰는 깃허브 코멘트 형식으로 제공이 됩니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F16-code-rabbit%2Fimage-2.png?alt=media&token=f00d14e3-3caf-45e6-b8b4-4c6f9b25e683)

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F16-code-rabbit%2Fimage-3.png?alt=media&token=551b3845-25c1-4a75-8b59-276a1ec00591)

## 내 입맛에 맞게 세팅하기

코드 레빗은 레포지토리별로 연결 및 설정을 할 수 있습니다. 한국어로도 서비스를 제공합니다(디폴트 언어는 영어입니다).

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F16-code-rabbit%2Fimage-11.png?alt=media&token=4263c4a6-2da3-4d41-9fcf-8aa7e07bfcf3)

간단한 리뷰를 원하면 Chill을, 세심한 리뷰를 원하면 Assertive를 선택할 수 있습니다. 디테일한 리뷰를 받고 취사 선택을 하면 좋겠다는 생각에 Assertive를 선택했습니다(변경된 코드가 많은 경우에 리뷰가 많아집니다).

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F16-code-rabbit%2Fimage-12.png?alt=media&token=2ef71714-3dcb-4751-83e1-a00d36d8029d)

PR이 올라왔을때 리뷰를 자동으로 트리거할 수 있습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F16-code-rabbit%2Fimage-13.png?alt=media&token=b358dcbe-f6f4-40e5-99e9-6f9cd940cc08)

## 코드 레빗 연결하기

코드 레빗을 레포지토리에 연결하는 방법은 다음과 같습니다.

### 1. [코드레빗 회원 가입](https://www.coderabbit.ai/){:target="\_blank"}

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F16-code-rabbit%2Fimage-14.png?alt=media&token=c186274d-2ba4-42e2-a3f1-d9cbadf1e3de)

### 2. 레포지토리 연결

회원 가입이 완료되면 대시보드에 접속할 수 있습니다. 이곳에서 코드 리뷰를 하고 싶은 레포지토리를 연결 할 수 있습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F16-code-rabbit%2Fimage-15.png?alt=media&token=802e084f-1b6c-43f8-a669-3572befe19c0)

레포지토리마다 요금이 측정되기 때문에 레포지토리를 신중하게 선택해야합니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F16-code-rabbit%2Fimage-6.png?alt=media&token=c2712d26-7948-406a-b6be-42d9aeb4304c)

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F16-code-rabbit%2Fimage-7.png?alt=media&token=d4518e94-7761-4191-917e-7005b399ff6e)

### 3. 디폴트 브랜치 설정하기

코드 리뷰는 디폴트 브랜치 PR에서만 트리거가 됩니다. 따라서 디폴트 브랜치를 정확히 설정해주어야합니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F16-code-rabbit%2Fimage-9.png?alt=media&token=ded6c64c-5708-4922-97b9-afcac95231e7)

## 결론

코드 리뷰가 힘든 환경에 계신분들에게 추천하고 싶습니다. 코드 레빗도 여느 AI 서비스와 마찬가지로 무료는 아닙니다. Lite버전은 월12달러, Pro버전은 월 24달러입니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F16-code-rabbit%2Fimage-10.png?alt=media&token=0dea6381-5a95-4c40-b05a-ada3ed6a21af)

회원가입 후 14일동안 무료로 서비스를 제공하고 있습니다. 무료 기간동안 사용해보시고 도입 여부를 판단해보시는 것을 추천드립니다. 저도 무료 버전을 사용하다가 현재는 Lite버전을 사용하고 있습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F16-code-rabbit%2Fimage-16.png?alt=media&token=791fe3cc-b428-423c-b56e-e375800fc2a3)

## 참고한 글

- [코드레빗 공식문서](https://docs.coderabbit.ai/){:target="\_blank"}
- [AI 코드리뷰 붙여보기](https://velog.io/@heyday_xz/AI-%EC%BD%94%EB%93%9C%EB%A6%AC%EB%B7%B0-%EB%B6%99%EC%97%AC%EB%B3%B4%EA%B8%B0){:target="\_blank"}
- [[Tech] CodeRabbit.. 그게 뭐지?](https://velog.io/@songju7920/Tech-CodeRabbit){:target="\_blank"}
