---
title: 웹 프론트엔드 성능 최적화 첫번째 스터디(LightHouse, 이미지 사이즈 최적화, 병목 코드 최적화, 텍스트 압축, 코드 분할, 지연 로딩)
date: 2024-09-24 17:20:00 +0900
categories: [Experience, Study]
tags: [Experience, Diary, Frontend, Study]
image: https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-1.gif?alt=media&token=e10c715f-03d3-4ccf-b611-d74592a837de
---

SK Open Lab에서 운영하는 프론트엔드 성능 최적화 스터디에 참여중입니다. [프론트엔드 성능 최적화 가이드](https://product.kyobobook.co.kr/detail/S000200178292){:target="\_blank"} 책을 읽고 발표하는 방식으로 진행하고 있는데요. 이번주는 [1장. 블로그 서비스 최적화] 챕터를 공부했습니다. LightHouse, 이미지 사이즈 최적화, 병목 코드 최적화, 코드 분할, 지연 로딩, 텍스트 압축에 대한 내용을 다루었습니다.

## 블로그 서비스 최적화 분석

블로그 서비스는 두 종류의 페이지(목록 페이지, 상세 페이지)로 이루어져 있습니다. 첫 화면에서는 타이틀과 게시물 내용 일부, 섬네일 이미지로 이루어진 블로그 글 목록 페이지를 볼 수 있습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-2.png?alt=media&token=4f984a08-773c-4225-9944-7cb78aca9dcf)

목록 중 하나를 클릭하면 게시물 내용 전체를 확인할 수 있는 상세 페이지로 이동합니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-3.png?alt=media&token=cc323442-0eae-4d12-b531-e3d53aa7626d)

## LightHouse

LightHouse는 구글에서 만든 웹사이트의 성능을 측정하고 개선 방향을 제시해주는 툴입니다. 웹사이트의 성능 점수를 측정하고 개선 가이드를 확인함으로써 어떤 부분을 중점적으로 분석하고 최적화해야 하는지 알 수 있습니다.

블로그 서비스를 검사했더니 성능은 74점, First Contentful Paint는 1.2초, Largest Contentful Paint는 1.9초, Total Blocking Time 280밀리초, Cumulative Layout Shift는 0.083, Speed Index는 1.6초가 나왔습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-4.png?alt=media&token=39b4d783-31a2-45b8-a382-8b701a598059)

이러한 지표를 웹 바이탈(Web Vitals)이라고 합니다. 각 지표를 살펴보면 다음과 같습니다.

- First Contentful Paint(FCP) : 페이지가 로드될때 브라우저가 DOM 콘텐츠의 첫 번째 부분을 렌더링하는 데 걸리는 시간에 관한 지표[총점 계산의 10% 가중치]
- Largest Contentful Paint(LCP) : 페이지가 로드될때 화면 내에 있는 가장 큰 이미지나 텍스트 요소가 렌더링되기까지 걸리는 시간을 나타내는 지표[총점 계산의 25% 가중치]
- Total Blocking Time(TBT) : 페이지가 클릭, 키보드 입력 등의 사용자 입력에 응답하지 않도록 차단된 시간을 총합한 지표, 메인 스레드를 독점하여 다른 동작을 방해하는 작업에 걸린 시간을 의미[총점 계산의 30% 가중치]
- Cumulative Layout Shift(CLS) : 페이지 로드 과정에서 발생하는 예기치 못한 레이아웃 이동을 측정한 지표[총점 계산의 25% 가중치]
- Speed Index(SI) : 페이지 로드 중에 콘텐츠가 시각적으로 표시되는 속도를 나타내는 지표[총점 계산의 10% 가중치]

Lighthouse 탭을 내리다보면 진단 섹션이 나타납니다. 진단 섹션에서는 성능과 관련된 기타 정보를 제공합니다. 이를 통해 해당 서비스의 어느 부분을 개선해야 성능을 향상할 수 있는지 파악할 수 있습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-5.png?alt=media&token=c535cdab-b644-4fc6-8b31-5bcf59ad447b)

탭의 가장 아래 부분에서는 CPU와 네트워크를 얼만큼 제한하여 검사를 진행했는지에 환경 정보를 확인할 수 있습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-6.png?alt=media&token=4ca4f087-26c4-42c5-8d6e-5c202d1ea314)

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-7.png?alt=media&token=660219d4-1029-43ee-9910-045397c223f5)

## 이미지 사이즈 최적화

### 1. 문제점 : 비효율적인 이미지 분석

Lighthouse 분석 결과 목록 페이지의 이미지에 대하여 [이미지 크기 적절하게 설정하기]를 제안하고 있습니다. 제안대로 이미지를 적절한 사이즈로 변경하면 이미지당 대략 200KiB를 줄일 수 있다고 합니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-8.png?alt=media&token=486a2652-c24f-4d6e-909f-f10324cfc521)

해당하는 이미지 요소를 확인해보니 실제 이미지 사이즈는 1200X1200px인데, 렌더링된 이미지 사이즈는 120X120px이라고 합니다. 그러니까 어차피 큰 사이즈의 이미지를 사용해도 1200X1200px로 표시하지 못하니, 처음부터 120X120px를 사용하면 좋을 것이라 생각할 수 있습니다. 하지만 요즘 사용되는 디스플레이는 같은 공간에 더 많은 픽셀을 그릴 수 있기 때문에 너비 기준으로 두 배 정도 큰 이미지를 사용하는 것이 적절합니다. 즉 240X240px 사이즈를 사용하는 것이죠.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-9.png?alt=media&token=85f2e153-0793-45b0-b271-af593866f9d2)

이미지 크기를 조절하기 위해서는 이미지가 어디서 오는지 파악을 해야합니다. 네트워크 탭을 확인해보니 API 서버에서 넘겨준 데이터임을 알 수 있습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-10.png?alt=media&token=baef6682-565c-458f-9482-f5568e86b3a1)

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-11.png?alt=media&token=ae1f3557-bb50-4efc-a79f-53b5557950a5)

### 2. 해결 : CDN, 이미지 사이즈 조절

자체적으로 가지고 있는 정적 이미지라면 사진 편집 툴을 이용해서 이미지 사이즈를 조절하면 됩니다. 하지만 이렇게 API를 통해서 받아오는 경우에는 이미지 CDN을 통해 사이즈를 조절해야 합니다.

CDN(Content Delivery Network)은 물리적 거리의 한계를 극복하기 위해 사용자와 가까운 곳에 콘텐츠 서버를 두는 기술입니다. 이미지 CDN은 기본적인 CDN 기능과 더불어 이미지를 사용자에게 보내기 전에 특정 현태로 가공하여 전해주는 기능을 포함하고 있습니다. 이미지 사이즈를 줄이거나 특정 포맷으로 변경하는 등의 작업처럼 말이죠.

일반적인 이미지 CDN에서 제공하는 주소는 다음과 같습니다. 이미지 CDN 서버의 주소(http://cdn.image.com)에 쿼리스트링으로 가져올 경로(src)를 입력해줍니다. 그리고 필요에 따라 변경하고자 하는 형태(width, height)를 명시해줍니다.

```
http://cdn.image.com?src=[img src]&width=240&height=240
```

목록 이미지 코드를 분석해보니 블로그 서비스에서는 이미지 CDN을 직접 만들고 있지 않았습니다. Unsplash라는 서비스가 이미지 CDN역할을 하고 있었습니다.

```js
function getParametersForUnsplash({ width, height, quality, format }) {
  return `?w=${width}&h=${height}&q=${quality}&fm=${format}&fit=crop`;
}
```

```js
<img
  src={
    props.image +
    getParametersForUnsplash({
      width: 1200,
      height: 1200,
      quality: 80,
      format: "jpg",
    })
  }
  alt="thumbnail"
/>
```

따라서 getParametersForUnsplash로 전달되는 width와 height를 240으로 변경해 주었습니다.

```js
<img
  src={
    props.image +
    getParametersForUnsplash({
      width: 240,
      height: 240,
      quality: 80,
      format: "jpg",
    })
  }
  alt="thumbnail"
/>
```

이미지 요소를 확인해보니 실제 이미지 사이즈가 240X240px로 바뀌어 있는 것을 확인할 수 있었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-12.png?alt=media&token=b89c0337-1327-493e-8d4a-cf8077c9bc57)

Lighthouse 검사를 해보니 성능이 81점(+7)로 향상되었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-13.png?alt=media&token=d8d5b780-5a5f-414f-8eb6-e7865b4466c6)

[이미지 크기 적절하게 설정하기] 제안도 사라졌음을 확인할 수 있었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-14.png?alt=media&token=1f8b7895-4308-4639-bfb9-3e86d60bc2b6)

## 병목 코드 최적화

### 1. 문제점 : 페이지 로드 과정 분석

진단 섹션을 보면 [자바스크립트 실행 시간] 제안이 있습니다. 항목을 펼쳐 상세 정보를 확인해 보니 0.chunk.js라는 파일이 오랫동안 실행되고 있습니다. 제가 실행한 환경에서는 247밀리초동안 실행되고 있습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-15.png?alt=media&token=3ebc9aba-e292-4d65-a545-12e58d350ce6)

오랫동안 자바스크립트가 실행되었고 그 때문에 서비스가 느려졌음을 알 수 있습니다. 무슨 자바스크립트 코드가 문제인지 파악하기 위해 성능 탭에서 메인 스레드 작업을 분석해보았습니다.

localhost라는 네트워크 요청을 통해 HTML 파일을 받아오고 이어서 bundle.js, 0.chuck.js, main.chuck.js 자바스크립트 파일을 받아오고 있습니다. 0.chuck.js 다운로드가 끝난 시점부터 메인 스레드에서 HTML을 파싱하고 자바스크립트를 평가하고 있습니다. 분홍색 영역에 마우스를 호버하니 App.js가 실행되고 있음을 알 수 있습니다. 이 작업들은 리액트 코드를 실행하는 작업이라고 볼 수 있습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-17.png?alt=media&token=53f3178e-d8c1-4b1a-9666-6861da600d9f)

소요 시간 탭을 보니 메인 스레드의 자바스크립트 작업이 끝나는 시점에 컴포넌트에 대한 렌더링(App [mount]) 작업이 기록되어 있었습니다. 그리고 컴포넌트가 마운트되면 ArticleList 컴포넌트에서는 블로그 글 데이터를 네트워크를 통해 요청하고 있었습니다(articles (localhost)).

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-18.png?alt=media&token=9470d0c5-f927-4c1d-badc-c21861d4542b)

다시 소요 시간 탭을 보니 articles 데이터가 모두 다운로드 되면 메인 스레드에서는 해당 컴포넌트를 렌더링하기 위해 자바스크립트를 실행하고 있었습니다. 하지만 이곳에서 실행 시간이 146밀리초가 걸린다는 점을 확인 할 수 있습니다. 네트워크 시간을 포함하는 것이 아니라 모든 데이터가 준비된 상태에서 단순히 데이터를 화면에 렌더링하는 것 치고는 오랜 시간이 걸린 것입니다.

메인 스레드를 내려가다보면 Article이라는 작업이 있습니다. 그 아래에는 removeSpecialCharacter 작업이 보입니다. 그 옆에는 Minor GC라는 작업이 보이지만 이 작업은 가비지 컬렉션 작업이기 때문에 실제 코드와는 상관이 없습니다. 그렇다면 결국 removeSpecialCharacter 작업이 Article 컴포넌트의 렌더링 시간을 길어지게 한 것이라 유추할 수 있습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-16.png?alt=media&token=9cf4d99b-f696-466b-b6f6-9db1b6e58f71)

### 2. 해결 : 병목 코드 개선

즉 Article 컴포넌트가 렌더링되는 과정에서 실행되는 removeSpecialCharacter 함수를 최적화하면 실행 시간을 단축시킬 수 있을 것입니다. removeSpecialCharacter는 인자로 넘어온 문자열에서 특수 문자를 제거하는 함수입니다. 마크다운으로 된 블로그 글에서 특수 문자를 모두 지우고 본문 일부를 보여주기 위해서 사용되고 있었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-19.png?alt=media&token=6fd1cea4-3788-4e8e-be17-630d168e419c)

removeSpecialCharacter 함수는 다음과 같습니다.

```js
function removeSpecialCharacter(str) {
  const removeCharacters = [
    "#",
    "_",
    "*",
    "~",
    "&",
    ";",
    "!",
    "[",
    "]",
    "`",
    ">",
    "\n",
    "=",
    "-",
  ];
  let _str = str;
  let i = 0,
    j = 0;

  for (i = 0; i < removeCharacters.length; i++) {
    j = 0;
    while (j < _str.length) {
      if (_str[j] === removeCharacters[i]) {
        _str = _str.substring(0, j).concat(_str.substring(j + 1));
        continue;
      }
      j++;
    }
  }

  return _str;
}
```

병목 코드를 최적화 시도한 부분은 두 가지 입니다.

1. 작업량 줄이기 : 순회 대상을 90000에서 300자로 줄이기

2. 특수 문자를 효율적으로 제거하기 : `concat` -> `replace`

```js
function removeSpecialCharacter(str) {
  let _str = str.substring(0, 300);
  _str = str.replace(/[#_*~&;![\]`>\n=-]/g, "");

  return _str;
}
```

병목 코드 최적화 이후 0.chunk.js 파일의 실행 시간이 64밀리초(-183)로 단축되어 성능이 91점(+10)으로 향상되었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-21.png?alt=media&token=dadcf9e6-50db-4779-926c-db23d88b7781)

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-20.png?alt=media&token=796dc3a8-4305-4038-a0c2-dc6df256382a)

Article의 실행 시간도 0.17밀리초(-145.83)로 줄어들었습니다. 그리고 코드량도 줄어들어 코드 가독성이 좋아졌습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-22.png?alt=media&token=1edd81b9-7070-4b31-8a82-430566271aec)

## 텍스트 압축

### 1. 문제점 : 텍스트 로드 과정 분석

production 환경일 때는 webpack에서 경량화라든지 난독화 같은 추가적인 최적화 작업을 진행합니다. 하지만 development 환경에서는 그런 최적화 작업 없이 서비스를 실행합니다. 따라서 production 환경과 development 환경에서는 성능 최적화에 있어 차이가 존재합니다. 최종 서비스의 성능을 측정할 때는 실제 사용자에게 제공되는 production 환경으로 빌드된 서비스의 성능을 측정하는 것이 좋습니다.

블로그 서비스의 목록 페이지 번들 파일 크기만 보아도 production이 절반정도 적습니다.

- development 환경 : 202B, 204B
- production 환경 : 113B, 113B

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-35.png?alt=media&token=158a20ab-4daf-4d79-92ed-3596d4b588be)

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-34.png?alt=media&token=232de4d6-d1a1-4025-968a-398c31da10b8)

Lighthouse로 상세 페이지를 검사해봤더니 진단에서 [텍스트 압축 사용]을 추천합니다. 텍스트를 압축하면 612KiB인 번들 파일의 크기를 402KiB로 줄일 수 있다고 합니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-36.png?alt=media&token=8f5e7b74-8e46-409c-9708-86cd505a8d83)

### 2. 해결 : 텍스트 압축

텍스트 압축은 말 그대로 텍스트를 압축해서 통신하는 기법입니다. HTML, CSS, 자바스크립트는 텍스트 기반의 파일이기 때문에 텍스트 압축 기법을 적용할 수 있습니다. 이런 파일을 압축하여 더 작은 크기로 빠르게 전송한 뒤, 사용하는 시점에 압축을 해제하는 방식입니다. 이때 압축한 만큼 파일 사이즈가 작아질 테니 리소스를 전송하는 시간이 단축됩니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-37.jpeg?alt=media&token=e5d8b998-2a95-4d78-b71a-adf6a1c79095)

압축 여부를 확인하기 위해 HTTP의 헤더를 살펴보았습니다. 네트워크 탭의 articles API 항목을 확인해보니 응답 헤더에 'Content-Encoding: gzip'이라고 되어 있었습니다. 이 리소스는 gzip 방식으로 압축되어 전송되었다는 사실을 알 수 있었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-38.png?alt=media&token=d17db41a-6a59-4849-89c6-41095db4952c)

그에 반해 main 번들 파일을 확인해보면 응답 헤더에 'Content-Encoding'이라는 항목이 없었습니다. 즉, 텍스트 압축이 적용되어 있지 않다는 의미입니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-39.png?alt=media&token=27acc1c5-9f0b-4243-b5a9-7c6a4a4c2b35)

텍스트 압축은 이 리소스를 제공하는 서버에서 설정해야 합니다. 따라서 텍스트 압축 후에 리소스를 전송하도록 -u옵션을 추가하는 방식으로 서버측 코드를 수정했습니다.

```json
"scripts": {
  "start": "react-scripts start",
  "build": "react-scripts build",
  "serve": "npm run build && node ./node_modules/serve/bin/serve.js -s build",
  "server": "node ./node_modules/json-server/lib/cli/bin.js --watch ./server/database.json -c ./server/config.json"
}
```

다시 실행한 후 네트워크 탭을 살펴보면 번들 파일 크기도 줄어들었습니다. 응답 헤더에 'Content-Encoding' 값이 gzip으로 설정된 것을 확인할 수 있었습니다. 그리고 [텍스트 압축 사용] 진단도 사라졌네요.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-40.png?alt=media&token=7cc992c1-0199-4efb-8fe4-14111832c4e8)

## 코드 분할 & 지연 로딩

### 1. 문제점 : 번들 파일 분석

앞서 성능 탭에서 검사를 해보았을때 유난히 크고 다운로드가 오래 걸렸던 0.chunk.js파일이 있었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-24.png?alt=media&token=463be342-49c6-45f6-8bdc-37e938fac732)

cra-bundle-analyzer 라이브러리를 사용해 번들 파일을 분석했더니 2.chunk.js 파일이 화면에 크게 나타났습니다. 파일의 비중을 봤을 때 앞서 성능 탭에서 검사할때 0.chunk.js 파일인 것 같습니다. 그리고 하위에 있는 요소의 이름이 node_modules인 것을 보니 이 번들 파일이 담고 있는 코드는 npm을 통해 설치된 외부 라이브러리인 것 같습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-23.png?alt=media&token=c5962d35-986f-4f81-a926-7e9d9e027bb0)

오른쪽 상단을 보면 파란색 블록이 있습니다. 그 안의 파일명으로 유추해보았을때 서비스에서 작성된 코드임을 알 수 있었습니다. 정리해보면 직접 작성한 서비스 코드는 main.chunk.js라는 이름으로, 외부 모듈은 2.chuck.js라는 이름으로 번들링 된 것입니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-25.png?alt=media&token=dea939d5-cebb-455b-a4da-6962ae1e3ec6)

2.chuck 내부를 살펴보면 react-dom과 refractor이 큰 비중을 차지하고 있습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-28.png?alt=media&token=cd45eb1d-0f69-4336-a798-04ed6218e364)

react-dom은 리액트를 위한 코드이므로 생략하고, refractor 패키지의 출처를 확인해보았습니다. package-lock.json 파일에서 refractor 패키지에 대한 내용을 찾아보았습니다. 찾아보니 react-syntax-highlighter라는 패키지에서 refractor를 참조하고 있습니다. react-syntax-highlighter는 마크다운의 코드 블록에 스타일을 입히는 데 사용되는 라이브러리입니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-27.png?alt=media&token=5a83bffd-5467-40d8-8375-809086efd038)

생각해보면 react-syntax-highlighter는 블로그 상세 페이지에서만 필요하지 글 목록 페이지에서는 필요하지 않았습니다. 즉, 크기가 큰 react-syntax-highlighter 모듈은 사용자가 처음 진입하는 글 목록 페이지에서는 다운로드 할 필요가 없는 것입니다.

### 2. 해결 : 코드 분할, 지연 로딩

코드 분할은 페이지 또는 컴포넌트별로 코드를 분할하는 방법이고, 지연 로딩은 분할된 코드를 필요한 시점에 다운로드하는 방법입니다. 코드 분할 & 지연 로딩 기법을 사용하면 페이지별로 코드를 분할하여 필요할때 사용할 수 있습니다. 블로그 목록 페이지에 접근하면 목록 페이지에 관련된 코드만 로드하고, 블로그 상세 페이지에 접근하면 상세 페이지에 관련된 코드만 로드하는 식으로 말이죠. 필요 없는 코드를 다운로드 하지 않으니 다운로드 속도가 빨라지는 장점이 있습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-29.jpeg?alt=media&token=8cf0fbc2-f54f-49bf-888a-5535ecfa6436)

코드 분할 & 지연 로딩을 위해 dynamic import를 적용했습니다. 이때 리액트에서 제공하는 lazy 함수와 Suspense를 활용했습니다.

```js
import React, { lazy, Suspense } from "react";
import { Switch, Route } from "react-router-dom";
import "./App.css";
// import ListPage from './pages/ListPage/index'
// import ViewPage from './pages/ViewPage/index'

const ListPage = lazy(() => import("./pages/ListPage/index"));
const ViewPage = lazy(() => import("./pages/ViewPage/index"));

function App() {
  return (
    <div className="App">
      <Suspense fallback={<div>Loading...</div>}>
        <Switch>
          <Route path="/" component={ListPage} exact />
          <Route path="/view/:id" component={ViewPage} exact />
        </Switch>
      </Suspense>
    </div>
  );
}

export default App;
```

코드 최적화 이후 번들 파일을 확인해보니 다음과 같았습니다.

- 0.chuck.js : 글 목록 페이지에서 사용하는 외부 패키지를 모아둔 번들 파일(ex. axios)
- 3.chunk.js : 상세 페이지에서 사용하는 외부 패키지를 모아둔 번들 파일(ex. react-syntax-highlighter)
- 4.chunk.js : 리액트 공통 패키지를 모아둔 번들 파일(ex. react-dom)
- 5.chunk.js : 글 목록 페이지 컴포넌트 번들 파일
- 6.chunk.js : 상세 페이지 컴포넌트 번들 파일

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-30.png?alt=media&token=df316f13-b27a-4bf3-bd81-1f46d6e85a51)

코드 최적화 이후 성능 패널을 확인해보니 번들 파일을 나누어서 받아오고 있었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-31.png?alt=media&token=30f082e4-d00a-43a5-821f-d1edf295b857)

글 목록 페이지에서 상세 페이지로 이동 시 새로운 번들 파일이 다운로드 되는 것도 확인할 수 있었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-32.gif?alt=media&token=3670c1d7-2436-463d-88c2-8400c556abea)

코드 최적화 이후 Lighthouse 성능 점수는 97점(+6)으로 향상되었네요.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F11-optimization-3%2Fimage-33.png?alt=media&token=73645833-5368-4c4a-aa86-bdad682d003e)

## 포스팅을 마무리하며

매우 유익한 시간이었습니다. Lighthouse를 분석하면서 이전부터 공부하고 싶었던 웹 바이탈(Web Vitals) 지표를 살펴볼 수 있었습니다. 그리고 개발자 도구를 뜯어보면서 요소, 네트워크, 성능 탭에서 각 데이터들이 의미하는 바를 살펴볼 수 있었습니다. 앞으로 웹 서비스를 분석하는데 도움이 될 것 같습니다.
