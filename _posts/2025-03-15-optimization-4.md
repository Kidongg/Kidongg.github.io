---
title: 웹 프론트엔드 성능 최적화 세번째 스터디(이미지 지연 로딩, 이미지 최적화, 동영상 최적화, 폰트 최적화)
date: 2024-10-22 22:20:00 +0900
categories: [Experience, Study]
tags: [Experience, Diary, Frontend, Study]
image: https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-1.gif?alt=media&token=bd5d0f79-859e-4469-b65a-8eacaf02f5d5
---

SK오픈랩에서 프론트엔드 성능 최적화를 주제로 스터디를 진행했습니다. 3번째 스터디의 주제는 홈페이지 성능 최적화였습니다. 스터디 당일 워크샵이 있어 참여를 하지 못했었는데요. 3개월이 지난 시점이지만 따로 공부했던 내용을 기록해보려합니다.

## 홈페이지 서비스 최적화 분석

예시 서비스는 히어로 영역에 비디오가 있는 랜딩 페이지입니다. 스크롤이 내려가면서 롱보드를 소개하는 이미지 영역과 상품을 소개하는 이미지 영역이 있습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-1-1.png?alt=media&token=fbce0480-3c34-44d7-8a99-df313d180fe1)

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-1-2.png?alt=media&token=22deb11e-4376-452f-8bce-571c8cc62ed3)

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-1-3.png?alt=media&token=718efd3e-a108-46d8-921f-94945fbeb72a)

보러가기 버튼을 클릭하면 해당 상품의 상세 페이지로 이동합니다. 하지만 오늘 소개드릴 최적화에는 상세 페이지를 다루지 않는다는 점 참고해주시기 바랍니다.

## 이미지 지연 로딩

### 1. 문제점

처음 화면에 진입하면 비디오가 보입니다. 하지만 네트워크 패널을 보면 비디오를 다운로드하기 전에 이미지를 다운로드를 하고 있습니다. 이미지는 스크롤이 아래로 내려갔을때 필요한 파일이라 초반에는 필요하지 않습니다. 따라서 비디오를 먼저 다운로드 시킬 필요가 있어보입니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-2.png?alt=media&token=c2672b37-fbb5-4a77-94b4-cfaa54306317)

### 2. 해결

이미지가 보이는 뷰포트에 진입했을때 이미지를 보여주면 좋을 것 같습니다. Intersection Observer는 뷰포트에 진입한 시점을 잡을때 사용하는 브라우저 API입니다. 웹 페이지의 특정 요소를 관찰하면서 페이지 스크롤 시 해당 요소가 화면에 들어왔는지 알려줍니다. 스크롤 이벤트처럼 스크롤할 때마다 함수를 호출하는 것이 아니라 요소가 화면에 들어왔을 때만 함수를 호출하기 때문에 성능 면에서 스크롤 이벤트보다 효율적입니다.

기존 코드는 다음과 같습니다.

```js
import React from "react";

function Card(props) {
  return (
    <div className="Card text-center">
      <img src={props.image} />
      <div className="p-5 font-semibold text-gray-700 text-xl md:text-lg lg:text-xl keep-all">
        {props.children}
      </div>
    </div>
  );
}

export default Card;
```

src 태그는 코드가 읽힐때 네트워크 요청을 통해 이미지를 가져옵니다. 따라서 data-src로 코드를 바꿔주고, useRef를 통해 특정 이미지 요소를 가져왔습니다. 그리고 useEffect를 통해 이미지 요소가 관찰되면 이미지를 가져오도록 처리를 했습니다.

```js
import React, { useEffect, useRef } from "react";

function Card(props) {
  const imageRef = useRef(null);

  useEffect(() => {
    const options = {};
    const callback = (entries, observer) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          console.log("is intersecting: ", entry.target.dataset.src);
          entry.target.src = entry.target.dataset.src;
          observer.unobserve(entry.target);
        }
      });
    };

    let observer = new IntersectionObserver(callback, options);

    observer.observe(imageRef.current);

    return () => observer.disconnect();
  }, []);

  return (
    <div className="Card text-center">
      <img data-src={props.image} ref={imageRef} />
      <div className="p-5 font-semibold text-gray-700 text-xl md:text-lg lg:text-xl keep-all">
        {props.children}
      </div>
    </div>
  );
}

export default Card;
```

```js
import React, { useEffect, useRef } from "react";

function Card(props) {
  const imageRef = useRef(null);

  useEffect(() => {
    const options = {};
    const callback = (entries, observer) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          console.log("is intersecting: ", entry.target.dataset.src);
          entry.target.src = entry.target.dataset.src;
          observer.unobserve(entry.target);
        }
      });
    };

    let observer = new IntersectionObserver(callback, options);

    observer.observe(imageRef.current);

    return () => observer.disconnect();
  }, []);

  return (
    <div className="Card text-center">
      <img data-src={props.image} ref={imageRef} />
      <div className="p-5 font-semibold text-gray-700 text-xl md:text-lg lg:text-xl keep-all">
        {props.children}
      </div>
    </div>
  );
}

export default Card;
```

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-3.gif?alt=media&token=dde9ef85-1ed5-46bc-b593-d6c9b3a607ae)

화면에 입장했을때는 당장 필요한 비디오 파일만 다운로드를 하고 스크롤을 통해 이미지 요소가 보일때 이미지 요소를 다운로드하는 것을 볼 수 있습니다. 메인 페이지의 상품을 소개하는 영역도 같은 방식으로 이미지 지연 로딩 처리를 했습니다.

## 이미지 최적화

### 1. 문제점

위 동영상에서 확인할 수 있듯이 이미지가 끊기면서 로드되고 있습니다. 이는 다운로드 되는 속도가 느리기 때문에 발생하는 현상입니다. 다운로드 속도가 느린 이유를 찾기 위해 네트워크 패널을 살펴보니, 이미지 파일의 크기가 크다는 점을 확인할 수 있었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-4.png?alt=media&token=89c2c19f-b06c-4b05-8672-0f654e449048)

### 2. 해결

이를 해결하기 위해 이미지 사이즈 최적화를 진행할 수 있습니다. 이미지 사이즈 최적화는 이미지의 가로, 세로 사이즈를 줄여 이미지 용량을 줄여 다운로드 속도를 늘리는 방법입니다.

실무에서 주로 사용하는 이미지 포맷은 다음과 같습니다.

1. PNG : 무손실 압축 방식으로 원본을 훼손 없이 압축하는 이미지 포맷입니다. 배경색을 투명하게 하여 뒤에 있는 요소가 보이는 이미지를 만들 수 있습니다.
2. JPG : 압축 과정에서 정보 손실이 발생합니다. 하지만 그만큼 이미지를 더 작은 사이즈로 줄일 수 있습니다. 일반적으로 웹에서 사용할 때는 고화질이어야 하거나 투명도가 필요한 것이 아니라면 JPG를 사용합니다.
3. WebP : 무손실 압축과 손실 압축을 모두 제공하는 최신 이미지 포맷입니다. 기존은 PNG나 JPG에 비해서 효율적으로 이미지를 압축할 수 있습니다. [공식 문서](https://developers.google.com/speed/webp?hl=ko){:target="\_blank"}에 따르면 WebP 방식은 PNG 대비 26%, JPG 대비 25~34% 더 나은 효율을 가지고 있다고 합니다.

[Squoosh](https://squoosh.app/){:target="\_blank"}를 사용하면 PNG 또는 JPG 파일을 Webp로 변환할 수 있습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-5.png?alt=media&token=c2070c08-737b-4c17-afb0-8bceeed98760)

다양한 변환 설정이 있지만 Width와 Height를 600px로 설정했습니다. 그 이유는 화면에 보이는 이미지의 사이즈가 300X300이기 때문입니다. 물론 반응형으로 구현되어 있어서 브라우저의 가로 사이즈에 따라 이미지 사이즈도 변하긴 하지만, 화면이 가장 클 때 300X300px이므로 그 두 배인 600X600px 사이즈로 변환했습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-6.png?alt=media&token=499e49c5-4ebe-4747-8421-9c162cf6d8c8)

화면 하단에 이미지가 원본 대비 몇 퍼센트 줄어들었는지, 결과적으로 이미지가 몇 kB로 압축되었는지 보여줍니다. 최종적으로 14.7kB로 압축되어 원본 대비 100%로 표시되고 있습니다.

다운로드하는 파일을 WebP로 변경해주고 네트워크 패널을 확인했더니 다운로드 용량과 속도가 확연히 줄어들었음을 확인할 수 있었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-4.png?alt=media&token=89c2c19f-b06c-4b05-8672-0f654e449048)

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-8.png?alt=media&token=e4e43bae-cd7d-4b41-9cf2-a6da43533068)

하지만 WebP 포맷은 효율은 좋지만 특정 브라우저에 대한 호환성 문제가 있습니다. 이 문제를 해결하기 위해 picture 태그를 컨테이너로 사용했습니다. 이를 통해 WebP 렌더링을 지원하지 않는 브라우저에서는 JPG 이미지로 렌더링을 대체할 수 있습니다.

```js
import React, { useEffect, useRef } from "react";

function Card(props) {
  const imageRef = useRef(null);

  useEffect(() => {
    const options = {};
    const callback = (entries, observer) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          const target = entry.target;
          const previousSibling = target.previousSibling;

          console.log("is intersecting: ", entry.target.dataset.src);
          target.src = target.dataset.src;
          previousSibling.srcset = previousSibling.dataset.srcset;
          observer.unobserve(target);
        }
      });
    };

    let observer = new IntersectionObserver(callback, options);

    observer.observe(imageRef.current);

    return () => observer.disconnect();
  }, []);

  return (
    <div className="Card text-center">
      <picture>
        <source data-srcset={props.webp} type="image/webp" />
        <img data-src={props.image} ref={imageRef} />
      </picture>
      <div className="p-5 font-semibold text-gray-700 text-xl md:text-lg lg:text-xl keep-all">
        {props.children}
      </div>
    </div>
  );
}

export default Card;
```

Intersection Observer도 picture 태그에 맞게 코드를 수정했습니다. 메인 페이지의 상품을 소개하는 영역도 같은 방식으로 이미지 지연 로딩 처리를 했습니다.

## 동영상 최적화

### 1. 문제점

네트워크 패널에서 알 수 있듯이 동영상 파일은 이미지처럼 하나의 요청으로 다운로드하지 않습니다. 동영상 콘텐츠의 특성상 파일 크기가 크기 때문에 당장 재생이 필요한 앞부분을 먼저 다운로드한 뒤 순차적으로 나머지 내용을 다운로드 합니다. 그래서 다운로드 요청이 여러개로 나누어져 있음을 알 수 있습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-9.png?alt=media&token=fcdeffa4-855e-4f26-a44c-a0c1aab2078b)

그럼에도 불구하고 파일 크기가 크기때문에 동영상을 재생할때까지 화면이 멈춰 있음을 알 수 있습니다(성능 패널을 통해 확인).

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-16.png?alt=media&token=b3787a2b-5525-44f3-9f8b-bfab3aa502aa)

동영상 파일을 확인해보니 파일 크기가 무려 54.5MB인 것을 알 수 있습니다. 웹에서 사용하기에는 너무 큰 것 같습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-10.png?alt=media&token=0eff659c-b997-49b1-969c-9ca0e9de9e86)

### 2. 해결

동영상 파일 또한 최적화를 진행할 수 있습니다. 동영상의 가로와 세로 사이즈를 줄이고, 압축 방식을 변경하는 방식으로 말입니다. 주의해야 할 점은 동영상 최적화는 동영상의 화질을 낮추는 작업이라는 점입니다. 따라서 동영상이 서비스의 메인 콘텐츠라면 최적화 작업 적용을 다시 검토해볼 필요가 있습니다.

[Media.io](https://www.media.io/apps/converter/){:target="\_blank"} 서비스를 사용해 동영상 파일 최적화를 할 수 있습니다. 동영상 포맷을 변환하고 Bitrate와 Audio 설정을 조절하는 방식으로 말입니다. Bitrate는 동영상 1초에 얼마나 많은 정보를 포함하는가를 의미합니다. 값이 크면 1초에 더 많은 정보를 포함하기 때문에 화질은 좋아지지만 파일의 사이즈가 커집니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-11.png?alt=media&token=a3717a2d-5156-468d-a0a4-26b63873d97a)

동영상 포맷을 WebM으로 변환시켰습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-12.png?alt=media&token=85c97672-1300-4b27-80a9-2aafe084e2ac)

Bitrate는 512Kbps로 설정하고, Audio는 체크를 해제했습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-13.png?alt=media&token=1df4714e-e57a-4554-ba20-1b5776d2a938)

동영상 파일을 확인해보니 용량이 원본의 1/5인 14.3MB로 줄어들었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-14.png?alt=media&token=039f3db1-30dc-4498-b774-259e8d24b1b0)

다운로드 용량과 속도가 확연히 줄어들었음을 확인할 수 있었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-9.png?alt=media&token=fcdeffa4-855e-4f26-a44c-a0c1aab2078b)

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-15.png?alt=media&token=2f674865-a9d1-4851-840e-f9bdd8353369)

WebM도 WebP 포맷과 마찬가지로 특정 브라우저에 대한 호환성 문제가 있습니다. 이 문제를 해결하기 위해 video 태그를 컨테이너화하여 WebM을 지원하지 않는 브라우저에 대응을 했습니다.

```js
import video from "../assets/banner-video.mp4";
import video_webm from "../assets/_banner-video.webm";

<video
  className="absolute translateX--1/2 h-screen max-w-none min-w-screen -z-1 bg-black min-w-full min-h-screen"
  autoPlay
  loop
  muted
>
  <source src={video_webm} type="video/webm" />
  <source src={video} type="video/mp4" />
</video>;
```

## 폰트 최적화

### 1. 문제

메인 페이지가 처음 렌더링될때 배너에 있는 텍스트가 변하는 현상을 볼 수 있습니다. 이 현상은 텍스트가 보이는 시점에 폰트 다운로드가 완료되지 않아 생기는 현상입니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-17.gif?alt=media&token=8c657b4a-5e69-4c04-a08d-59ac135be04b)

이 현상은 사용성에 영향을 줍니다. 폰트가 바뀌면서 깜빡이는 모습은 페이지가 느리다는 느낌을 줄 수도 있고 또는 다른 요소를 밀어낼 수도 있기 때문입니다. 폰트의 변화로 발생하는 현상을 FOUT(Flash of Unstyled Text) 또는 FOIT(Flash of Invisible Text)으로 구분합니다.

- FOUT : Edge 브라우저에서 폰트를 로드하는 방식으로, 폰트의 다운로드 여부와 상관없이 텍스트를 먼저 보여준 후 폰트가 다운로드되면 그때 폰트를 적용하는 방식입니다.
- FOIT : 크롬, 사파리, 파이어폭스 등에서 폰트를 로드하는 방식으로, 폰트가 완전히 다운로드되기 전까지 텍스트 자체를 보여주지 않습니다. 그리고 폰트 다운로드가 완료되면 폰트가 적용된 텍스트를 보여줍니다.

그리고 네트워크 패널에서 폰트 파일 크기가 751kB인 것을 확인할 수 있습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-18.png?alt=media&token=f8c9465f-4c79-481d-92af-4a6d02f10f90)

### 2. 해결

폰트를 최적화하는 방법은 크게 두 가지가 있습니다.

1. 폰트 적용 시점 제어하기
2. 폰트 파일 크기 줄이기

#### 1. 폰트 적용 시점 제어하기

FOUT 방식으로 폰트를 렌더링하는 Edge에 FOIT 방식을 적용하거나, FOIT 방식으로 폰트를 렌더링하는 크롬에 FOUT 방식을 적용할 수 있습니다. 이는 CSS의 font-display 속성을 이용하여 제어할 수 있습니다. font-display는 @font-face에서 설정할 수 있고 다음 값을 갖습니다.

- auto: 브라우저 기본 동작(기본 값)
- block: FOIT(timeout = 3s)
- swap: FOUT
- fallback: FOIT(timeout = 0.1s) / 3초 후에도 불러오지 못한 경우 기본 폰트로 유지, 이후 캐시
- optional: FOIT(timeout = 0.1s) / 이후 네트워크 상태에 따라 기본 폰트로 유지할지 결정, 이후 캐시

저는 FOIT 방식인 block을 사용했습니다. 왜냐하면 'KEEP CALM AND RIDE LONGBOARD' 텍스트는 빠르게 보여줘야 하거나 중요한 내용의 텍스트가 아니기 때문입니다. 문제는 block 옵션을 설정하면 안보이던 폰트가 갑자기 나타나서 어색할 수 있다는 점입니다. 이 문제를 해결하기 위해 페이드 인 애니메이션을 적용했습니다. 폰트의 다운로드 시점을 잡기 위해 fontfaceobserver라는 라이브러리를 활용했습니다.

```js
import React, { useEffect, useState } from "react";
import video from "../assets/banner-video.mp4";
import video_webm from "../assets/_banner-video.webm";
import FontFaceObserver from "fontfaceobserver";

const font = new FontFaceObserver("BMYEONSUNG");

function BannerVideo() {
  const [isFontLoaded, setIsFontLoaded] = useState(false);

  useEffect(() => {
    font.load(null, 20000).then(function () {
      console.log("BMYEONSUNG has loaded");
      setIsFontLoaded(true);
    });
  }, []);

  return (
    <div
      className="BannerVideo w-full h-screen overflow-hidden relative bg-texture"
      style={{ opacity: isFontLoaded ? 1 : 0, transition: "opacity 0.3s ease" }}
    >
      <div className="absolute h-screen w-full left-1/2">
        <video
          className="absolute translateX--1/2 h-screen max-w-none min-w-screen -z-1 bg-black min-w-full min-h-screen"
          autoPlay
          loop
          muted
        >
          <source src={video_webm} type="video/webm" />
          <source src={video} type="video/mp4" />
        </video>
      </div>
      <div className="w-full h-full flex justify-center items-center">
        <div className="text-white text-center">
          <div className="text-6xl leading-none font-semibold">KEEP</div>
          <div className="text-6xl leading-none font-semibold">CALM</div>
          <div className="text-3xl leading-loose">AND</div>
          <div className="text-6xl leading-none font-semibold">RIDE</div>
          <div className="text-5xl leading-tight font-semibold">LONGBOARD</div>
        </div>
      </div>
    </div>
  );
}

export default BannerVideo;
```

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-19.gif?alt=media&token=ea4c1d9d-c2a5-4edc-913c-8ac657f84157)

#### 2. 폰트 파일 크기 줄이기

폰트 파일 크기를 줄이는 방법은 폰트 파일 포맷을 WOFF(WebOpenFontFormat) 또는 WOFF2로 변경하는 것입니다. WOFF는 이름 그대로 웹을 위한 폰트입니다. 이 포맷은 TFF 폰트를 압축하여 웹에서 더욱 빠르게 로드할 수 있도록 만들었습니다. 더 나아가서 WOFF2라는 더욱 향상된 압축 방식이 적용된 포맷도 있습니다. [Transfonter](https://transfonter.org/){:target="\_blank"} 서비스를 사용하면 폰트를 변환할 수 있습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-20.png?alt=media&token=216319ae-45c6-4846-be4d-313db3334923)

폰트를 TFF에서 WOFF와 WOFF2로 변환했더니 1.9MB 였던 폰트 파일이 790KB, 447KB로 줄어들었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-21.png?alt=media&token=78f5d735-633b-4012-b854-bb46e7b09de4)

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-22.png?alt=media&token=3c1fdd22-9a5e-4381-ae5c-da4e30fbe1f2)

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-23.png?alt=media&token=43c1108d-9617-4d5c-84ba-4c061ff3065e)

WOFF와 WOFF2는 WebP와 마찬가지로 브라우저 호환성 문제가 있습니다. 따라서 WOFF2를 우선적으로 적용하고 브라우저가 이를 적용하지 않는다면 다운그레이드하는 방식을 적용했습니다.

```css
@font-face {
  font-family: BMYEONSUNG;
  src: url("./assets/fonts/BMYEONSUNG.woff1") format("woff2"), 
    url("./assets/fonts/BMYEONSUNG.woff") format("woff"), 
    url("./assets/fonts/BMYEONSUNG.ttf") format("truetype");
}
```

여기서 폰트 크기를 더 줄일 수도 있습니다. 잘 생각해 보면 홈페이지에서 웹 폰트를 사용하는 텍스트는 배너 영역 하나입니다. 그것도 'KEEP CALM AND RIDE LONGBOARD' 텍스트에서만 사용됩니다. 즉, 모든 문자의 폰트 정보를 가지고 있을 필요 없이 해당 문자의 정보만 있으면 될 것 입니다. 이렇게 모든 문자가 아닌 일부 문자의 폰트 정보만 가지고 있는 것을 서브셋 폰트라고 합니다. 서브셋 폰트 역시 Transfonter 서비스에서 생성할 수 있습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-25.png?alt=media&token=5ffe1822-a104-48e7-8efe-17a03ac6a0f0)

폰트 파일 크기를 확인해보니 21KB, 10KB, 8KB로 줄어들었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-26.png?alt=media&token=c1cd4ea2-9904-4f92-a083-528fa7b39780)

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-27.png?alt=media&token=1c4024d0-e022-4181-b0cf-9fda7e2843be)

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-28.png?alt=media&token=741d0377-4ed0-46aa-bd92-bca889231705)

네트워크 패널에서 폰트 파일의 크기와 다운로드 시간이 확연히 줄었음을 확인할 수 있었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-18.png?alt=media&token=f8c9465f-4c79-481d-92af-4a6d02f10f90)

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F17-optimization-4%2Fimage-29.png?alt=media&token=174e06b4-d97c-40b3-9982-0edcf9b434da)

## 포스팅을 마무리하며

웹 프론트엔드 최적화 스터디가 마무리되었습니다. 퇴근 후 시간을 쥐어짜내는(?) 방식으로 스터디에 참여를 했기에 쉽지만은 않았던 것 같습니다. 그럼에도 불구하고 스터디를 끝까지 마무리한 스스로에게 칭찬을 해주고 싶습니다. 개인적으로 코드 분할, 사전로딩, 지연로딩, 이미지/동영상/폰트 최적화, 레이아웃 시프트 방지 등 실무에 도움이 되는 지식을 배울 수 있어서 좋았습니다. 배운 개념들을 실무에서 어떻게 활용할 수 있을지 고민이 필요한 시점입니다.
