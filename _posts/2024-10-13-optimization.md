---
title: 웹 프론트엔드 성능 최적화(애니메이션, 지연 로딩, 사전 로딩)
date: 2024-10-13 20:30:00 +0900
categories: [Experience, Study]
tags: [Experience, Diary, Frontend, Study]
image: https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F8-optimization%2Fimage-15.png?alt=media&token=6d56b5b3-931d-45be-b9e7-6ca86f8e8c36
---

SK Open Lab 2기에서는 [프론트엔드 성능 최적화 가이드] 책을 챕터별로 읽고 격주 발표하는 방식으로 스터디를 진행하고 있습니다. 이번주는 [2장. 올림픽 통계 서비스 최적화] 챕터 발표가 진행이 되었습니다. 해당 챕터를 공부하며 배우고 느낀점을 공유하려 합니다.

## 올림픽 통계 서비스 분석
올림픽 통계 서비스를 실행해보니 리우 올림픽과 런던 올림픽을 비교하는 단일 페이지가 나타났습니다. 두 올림픽을 비교하는 표 하단에 “올림픽 사진 보기” 버튼이 있고, 이 버튼을 클릭하면 모달이 뜨면서 사진을 캐러셀 형태로 보여주는 것을 알 수 있습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F8-optimization%2Fimage-1.png?alt=media&token=bc96706f-38d5-4b95-8386-9548ecd9d61e)

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F8-optimization%2Fimage-2.png?alt=media&token=af92a2d6-e4d3-4184-8753-81024b20358d)

“올림픽 사진 보기” 버튼 아래에는 설문이 보이는데요. 설문 결과는 막대 그래프로 표시되어 있고, 하나의 항목을 클릭하면 애니메이션이 동작합니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F8-optimization%2Fimage-3.png?alt=media&token=810aaa02-cfc5-40b6-9490-c713ffef268c)

## 애니메이션 최적화

### 1. 문제점 찾기 : *_쟁크 현상_
*_쟁크(jank) : 애니메이션이 버벅이는 현상_

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F8-optimization%2Fimage-4.gif?alt=media&token=d320826b-0fa5-4c32-8f64-9532cf81c310)

쟁크 현상이 발생하는 이유를 이해하기 위해서는 브라우저 주사율과 브라우저 렌더링 과정을 살펴볼 필요가 있습니다. 일반적으로 사용하는 디스플레이의 주사율은 60Hz(1초에 60장의 정지된 화면을 보여줌)입니다. 따라서 브라우저도 이에 맞춰 최대 60FPS(Frames Per Second)로 1초에 60장의 화면을 새로 그립니다. 그래서 서비스에서 쟁크가 발생한 이유도 브라우저가 정상적으로 60FPS로 화면을 그리지 못했기 때문이라 유추할 수 있습니다.

### 2. 브라우저 렌더링 과정
브라우저는 다음과 같은 과정을 거쳐 렌더링을 합니다.

1. DOM+CSSOM : 다운로드한 HTML, CSS를 브라우저가 이해할 수 있는 트리 형태로 파싱(parsing)합니다.

2. 렌더트리(render tree) : DOM과 CSSOM을 결합합니다. 해당 정보는 레이아웃을 계산하는데 사용됩니다.

3. 레이아웃(layout) : 화면 구성 요소의 위치나 크기를 계산하고 배치합니다.

4. 페인트(paint) : 
  - 화면에 배치된 요소에 색을 채워넣습니다.
  - 이때 구성 요소를 여러개의 레이어로 나누어 작업하기도 합니다.

5. 컴포지트(composite) : 레이어를 합성합니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F8-optimization%2Fimage-5.jpeg?alt=media&token=60f4ca03-384b-4fc6-9da6-00b7d5be3d54)

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F8-optimization%2Fimage-6.png?alt=media&token=cf549087-6615-4116-a951-6d944876acb0)

### 3. 리플로우와 리페인트
만약 화면이 전부 그려진 후 애니메이션처럼 일부 요소의 스타일을 변경하거나 추가, 제거되면 어떻게 될까요? 이런 경우 주요 렌더링 경로에서 거친 과정을 다시 한 번 실행하면서 새로운 화면을 그립니다. 이것을 리플로우(reflow) 또는 리페인트(repaint)라고 합니다.

- 리플로우
  - 개념 : 브라우저 렌더링 과정을 레이아웃을 포함하여 다시 실행
  - 조건 : `width`, `height` 변경시(위치나 크기)

  ![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F8-optimization%2Fimage-7.jpeg?alt=media&token=e4c5138e-56ba-453c-b896-81a85a34411a)

- 리페인트
  - 개념 : 브라우저 렌더링 과정 중 레이아웃을 제외하고 다시 실행
  - 조건 : `color`, `background-color` 변경시(색깔)

  ![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F8-optimization%2Fimage-8.jpeg?alt=media&token=3d4bcc51-7b2a-4f20-abd7-b0da3014715e)

리페인트 작업은 레이아웃 단계를 건너뛰기 때문에 리플로우 작업보다는 조금 더 빠릅니다. 하지만 리페인트 역시 거의 모든 단계를 거치기 때문에 브라우저의 리소스를 꽤 잡아먹습니다. 정리하자면 모두 브라우저의 리소스를 많이 잡아먹기 때문에 화면을 새로 그리는 것이 느릴 수 밖에 없습니다.

### 4. 하드웨어 가속(GPU 가속)
다행히 `transform`, `opacity`와 같은 속성을 사용해 스타일을 처리하면 리플로우와 리페인트를 피할 수 있습니다. 두 속성을 사용하면 해당 요소를 별도의 레이어로 분리하고 작업을 GPU에 위임합니다. 이로써 레이아웃 단계와 페인트 단계를 건너뛸 수 있게 됩니다. 이를 하드웨어 가속이라고 합니다.

### 5. 해결 : `transform`을 통한 애니메이션 가속화
문제의 원인과 해결 방법을 알았으니 width로 되어 있는 기존의 애니메이션 코드를 `transform`으로 바꾸어 주었습니다. 애니메이션 코드 최적화 이후 쟁크 현상이 사라졌음을 확인할 수 있었습니다.

- 기존 코드

```
// @/src/components/Bar.js

// ...생략
const Bar = (props) => {
    return (
        <BarWrapper onClick={props.handleClickBar} isSelected={props.isSelected}>
            <BarInfo>
                <Percent>{props.percent}%</Percent>
                <ItemVaue>{props.itemValue}</ItemVaue>
                <Count>{props.count}</Count>
            </BarInfo> 
            <BarGraph width={props.percent} isSelected={props.isSelected}></BarGraph>
        </BarWrapper> 
    )
}

// ...생략
const BarGraph = styled.div`
    position: absolute;
    left: 0;
    top: 0;
		width: ${({width}) => width}%;
    transition: width 1.5s ease;
    height: 100%;
    background: ${({isSelected}) => isSelected ? 'rgba(126, 198, 81, 0.7)' : 'rgb(198, 198, 198)'};
    z-index: 1;
`   
```

- 최적화 코드

```
// ...생략
const Bar = (props) => {
    return (
        <BarWrapper onClick={props.handleClickBar} isSelected={props.isSelected}>
            <BarInfo>
                <Percent>{props.percent}%</Percent>
                <ItemVaue>{props.itemValue}</ItemVaue>
                <Count>{props.count}</Count>
            </BarInfo> 
            <BarGraph width={props.percent} isSelected={props.isSelected}></BarGraph>
        </BarWrapper> 
    )
}

// ...생략
const BarGraph = styled.div`
    position: absolute;
    left: 0;
    top: 0;
		width: 100%;
    transform: scaleX(${({width}) => width /100});
    transform-origin: left;
    transition: transform 1.5s ease;
    height: 100%;
    background: ${({isSelected}) => isSelected ? 'rgba(126, 198, 81, 0.7)' : 'rgb(198, 198, 198)'};
    z-index: 1;
`
```

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F8-optimization%2Fimage-9.gif?alt=media&token=f1d7b4a7-9008-4933-952c-75de0fb9876f)

## 컴포넌트 지연 로딩

### 1. 문제점 찾기 : 번들 파일 분석
우선은 Network를 통해 페이지에 접근할때, 모달이 나타날 때 무슨 파일이 로드 되는지 확인을 해보았습니다. 페이지에 접근할때 하나의 청크파일이 로드되고, 모달이 나타날 때는 따로 청크파일이 로드되고 있지 않았습니다.

- 페이지에 접근할때

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F8-optimization%2Fimage-10.png?alt=media&token=0b3871ba-385d-4be7-9e71-e7a64742cd3f)

- 모달이 나타날때

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F8-optimization%2Fimage-11.png?alt=media&token=e72db597-d240-49fc-916a-c5254f74e08b)

그리고 cra-bundle-ananlyzer를 통해 번들 파일을 분석을 진행했습니다.

```
npm install --save-dev cra-bundle-analyzer
npx cra-bundle-analyzer
```

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F8-optimization%2Fimage-12.png?alt=media&token=85239ce9-60fe-4ae6-8605-e50370e19f54)

- `static/js/2.chunk.js`(왼쪽 갈색 블록) : node.modules에 있는 라이브러리 코드를 담고 있는 청크

- `static/js/main.chunk.js`(오른쪽 파랑 블록) : 모달 컴포넌트를 포함한 올림픽 통계 서비스 코드

`static/js/2.chunk.js`의 내용을 보면 react-image-gallery 라이브러리(26KB)가 들어있습니다. 해당 라이브러리는 모달 화면에서만 사용이 되기 때문에 첫 화면에서는 필요하지 않습니다. 그래서 해당 라이브러리 코드를 분할하고 지연 로딩을 할 수 있습니다.

### 2. 해결 : `Suspense`, `lazy()`를 통한 컴포넌트 코드 분할 및 지연 로딩

페이지를 기준으로 할때와 마찬가지로 컴포넌트도 `Suspense`와 `lazy()`를 통해 코드 분할 및 지연 로딩을 할 수 있습니다. `ImageModal`을 직접 `import`하는 구문을 주석처리하고 분할하고자 하는 컴포넌트인 `ImageModal`을 `import` 함수와 함께 `lazy` 함수의 인자로 넣어주었습니다.

```
// @/src/App.js

import React, { lazy, Suspense, useEffect, useState } from 'react'
// import ImageModal from './components/ImageModal'

const LazyImageModal = lazy(() => import('./components/ImageModal'))
```

그리고 `ImageModal`이 로드되기 전에 발생하는 에러를 방지하기 위해서 `Suspense`로 `LazyImageModal` 컴포넌트를 감싸주었습니다.

```
function App() {
    const [showModal, setShowModal] = useState(false)

    return (
        <div className="App">
            <Header />
            <InfoTable />
            <ButtonModal onClick={() => { setShowModal(true) }}>올림픽 사진 보기</ButtonModal>
            <SurveyChart />
            <Footer />
            <Suspense fallback={null}>
                {showModal ? <LazyImageModal closeModal={() => { setShowModal(false) }} /> : null}
            </Suspense>
            {/* {showModal ? <ImageModal closeModal={() => { setShowModal(false) }} /> : null} */}
        </div>
    )
}
```

페이지에 접근할때 chunk 파일의 용량이 줄어들고, 모달이 나타날 때(모달 버튼을 클릭할때) 새로운 청크 파일인 `2.chunk.js`, `3.chunk.js`가 로드되는 것을 확인할 수 있습니다. 이 두 개가 `ImageModal` 컴포넌트와 react-image-gallery 라이브러리가 담긴 파일입니다.

- 페이지에 접근할때

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F8-optimization%2Fimage-13.png?alt=media&token=5e498278-3e59-4532-9b4c-5ab40c639427)

- 모달이 나타날때

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F8-optimization%2Fimage-14.png?alt=media&token=42648256-9d86-4152-ab00-f767764c2e59)

다시 cra-bundle-ananlyzer를 통해 번들 파일을 분석을 진행했습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F8-optimization%2Fimage-15.png?alt=media&token=6d56b5b3-931d-45be-b9e7-6ca86f8e8c36)

- `static/js/2.chunk.js`(왼쪽 갈색 블록) : node.modules에 있는 라이브러리 코드를 담고 있는 청크

- `static/js/3.chunk.js`(오른쪽 상단 파랑 블록) : 모달 컴포넌트에서 사용하는 라이브러리 코드를 담고 있는 청크

- `static/js/4.chunk.js`(오른쪽 하단 진한 하늘색 블록) : 모달 컴포넌트 코드

- `static/js/main.chunk.js`(오른쪽 하단 연한 파랑 블록) : 모달 컴포넌트 제외한 올림픽 통계 서비스 코드

결과를 확인해 보니 파란색 블록으로 react-image-gallery 라이브러리가 분리되어 있고 아래 진한 하늘색 블록으로 ImageModal 컴포넌트가 분리되어 있습니다. react-image-gallery에서 참조하고 있는 모든 라이브러리가 함께 분할되어 26KB보다 큰 56KB가 분할되었습니다. 물론 지금은 모당의 코드가 크지 않아 성능 개선이 느껴지지 않을 수 있지만 나중에 모달안에 더 많은 콘텐츠와 라이브러리가 들어간다면 의미있는 최적화가 될 것입니다.

## 컴포넌트 사전 로딩

### 1. 컴포넌트 지연 분할과 지연 로딩의 단점
컴포넌트 코드 분할과 지연 로딩은 당장 필요없는 코드가 번들에 포함되지 않아 로드할 파일의 크기가 작아지고 초기 로딩 속도나 자바스크립트의 실행 타이밍이 빨라진다는 장점이 있습니다. 그러나 해당 코드가 필요한 시점에는 한계가 존재합니다.

### 2. 문제점 찾기 : 모달 화면 깨짐 현상
모달이 나타날 때 네트워크를 통해 모달 코드를 새로 로드해야 하고 로드가 완료가 되어야만 화면에 보여줄 수 있습니다. 따라서 모달 코드가 로드될때까지 의도하지 않았던 화면을 보여줄 수 있습니다. 예제에서는 모달 화면이 깨지는 현상이 있었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F8-optimization%2Fimage-16.png?alt=media&token=7c710113-0f26-4bc9-9f79-606dc92b2881)

Performance탭에서 모달 코드가 로드될때 화면 깨짐이 일어나고 있다는 점을 확인할 수 있습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F8-optimization%2Fimage-17.png?alt=media&token=dc187f05-bf3d-4197-a8ea-b986fe71f5e4)

### 3. 해결
이 문제를 해결하기 위해 사용할 수 있는 기법이 사전 로딩(Preloading)입니다. 사전 로딩은 모듈이 필요해지기 전에 미리 로드하는 기법입니다. 사전 로딩을 하기 위해서는 어느 타이밍에 코드를 로드해올 것인가를 정해야합니다. 대표적인 타이밍은 2가지가 있습니다.

#### 3-1) 버튼 위에 마우스를 올려놓았을 때 사전 로딩 : `onMouseEnter`
마우스가 버튼에 올라오면 사용자가 버튼을 클릭해서 모달을 띄울 것이라고 예측할 수 있습니다. 따라서 아직 버튼을 클릭하지 않았지만 곧 클릭할 것이기에 모달 컴포넌트를 미리 로드해 두는 방법입니다. 마우스가 버튼에 올라왔는지 아닌지는 `button` 요소의 속성인 `onMouseEnter`를 통해 알 수 있습니다. 따라서 이 이벤트에서 `ImageModal` 컴포넌트를 `import` 하여 로드를 해주었습니다.

```
// @/src/App.js

function App() {
    const [showModal, setShowModal] = useState(false)

    // 모달 컴포넌트 사전 로딩 - 마우스 올려놨을때
    const handleMouseEnter = () => {
        const component = import('./components/ImageModal')
    }

    return (
        <div className="App">
            <Header />
            <InfoTable />
            <ButtonModal onClick={() => { setShowModal(true) }} onMouseEnter={handleMouseEnter}>올림픽 사진 보기</ButtonModal>
            <SurveyChart />
            <Footer />
            <Suspense fallback={null}>
                {showModal ? <LazyImageModal closeModal={() => { setShowModal(false) }} /> : null}
            </Suspense>
        </div>
    )
}
```

`ButtonModal` 컴포넌트에 마우스를 올렸더니 모달에 사용하는 파일들이 미리 로드되는 것을 확인할 수 있었습니다(`1.chunk.js`, `2.chunk.js`). 버튼을 클릭할 때 마우스 커서를 버튼 위에 올리고 클릭하기까지 대략 300~600밀리초 정도의 시간차가 있다고 생각한다면, 브라우저가 새로운 파일을 로드하기에는 충분한 시간일 것입니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F8-optimization%2Fimage-18.png?alt=media&token=116f6a4a-86f1-4e0b-958e-3ac1d1095672)

#### 3-2) 컴포넌트 마운트가 완료된 후에 사전 로딩 : `useEffect()`
만약에 모달 컴포넌트의 크기가 커서 로드하는 데 오랜 시간이 필요할 수 있습니다. 이런 경우에는 마우스 커서를 버튼에 올렸을 때보다 더 먼저 파일을 로드해야 합니다. 이때 생각해 볼 수 있는 타이밍은 모든 컴포넌트의 마운트가 완료된 후로, 브라우저에 여유가 생겼을 때 뒤이어 모달을 추가로 로드하는 것입니다. 리액트 함수형 컴포넌트에서는 `useEffect`로 시점을 잡을 수 있습니다.

```
// @/src/App.js

function App() {
    const [showModal, setShowModal] = useState(false)

    // 모달 컴포넌트 사전 로딩 - 컴포넌트 마운트 이후
    useEffect(() => {
        const component = import('./components/ImageModal')
    }, [])

    return (
        <div className="App">
            <Header />
            <InfoTable />
            <ButtonModal onClick={() => { setShowModal(true) }}>올림픽 사진 보기</ButtonModal>
            <SurveyChart />
            <Footer />
            <Suspense fallback={null}>
                {showModal ? <LazyImageModal closeModal={() => { setShowModal(false) }} /> : null}
            </Suspense>
        </div>
    )
}
```

Network를 통해 타임라인을 확인해 보면 초기 페이지 로드에 필요한 파일(`3.chunk.js`, `bundle.js`)를 우선 다운로드하고 페이지 로드가 완료된 후에야 모달 코드(`1.chunk.js`, `2.chunk.js`)를 다운로드하는 것을 확인할 수 있었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F8-optimization%2Fimage-19.png?alt=media&token=02d693ae-ff47-4e25-b92b-15bb329064f7)

사전 로드하는 방법이 이 두 가지만 있는 것은 아닙니다. 서비스나 기능의 특성에 따라서 다양한 방법을 적용할 수 있습니다. 중요한 것은 어느 타이밍에 사전 로드하는 것이 해당 서비스에서 가장 합리적인지 판단하는 일입니다.

## 이미지 사전 로딩

### 1. 문제점 찾기 : 모달 화면 깨짐 현상
모달 컴포넌트를 코드 분할하고 지연 로딩했음에도 불구하고 여전히 모달 화면이 깨지는 현상이 있습니다. 그 이유는 앞선 모달 코드와 마찬가지로 이미지를 새로 로드해야 하고, 로드가 완료가 되어야만 화면에 보여줄 수 있기 때문입니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F8-optimization%2Fimage-20.png?alt=media&token=b2f4b513-6e6d-4ef4-bc9a-c66117ce54c7)

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F8-optimization%2Fimage-plus.gif?alt=media&token=8b35f0d1-e673-406a-a483-17876b81c4b0)

### 2. 해결 : Image 객체를 통한 사전 로딩
이 문제도 이미지 사전 로딩으로 해결할 수 있습니다. 자바스크립트 Image 객체를 생성해 src 속성에 원하는 이미지 주소를 입력하면 이미지를 로드할 수 있습니다. 저는 모달에서 가장 먼저 보이는 [10번 선수가 드리블 하는 대표사진]을 `useEffect`를 통해 사전 로드 했습니다.

```
// @/src/App.js

function App() {
    const [showModal, setShowModal] = useState(false)

    // 모달 컴포넌트 사전 로딩 - 컴포넌트 마운트 이후
    useEffect(() => {
        const component = import('./components/ImageModal')
    }, [])

    // 모달 이미지 사전 로딩 - 컴포넌트 마운트 이후
    useEffect(() => {
        const img = new Image()
        img.src="https://stillmed.olympic.org/media/Photos/2016/08/20/part-1/20-08-2016-Football-Men-01.jpg?interpolation=lanczos-none&resize=*:800"
    }, [])

    return (
        <div className="App">
            <Header />
            <InfoTable />
            <ButtonModal onClick={() => { setShowModal(true) }}>올림픽 사진 보기</ButtonModal>
            <SurveyChart />
            <Footer />
            <Suspense fallback={null}>
                {showModal ? <LazyImageModal closeModal={() => { setShowModal(false) }} /> : null}
            </Suspense>
        </div>
    )
}
```

Network를 통해 타임라인을 확인해 보면 초기 페이지 로드에 필요한 이미지를 우선 다운로드하는 것을 확인할 수 있습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F8-optimization%2Fimage-21.png?alt=media&token=ac80e13f-daa2-4094-a314-0d24842f38aa)

그리고 모달이 나타날때 이미지는 초기 페이지 로드때보다 용량과 속도가 줄었다는 것을 확인할 수 있었습니다(143kB → disk cache / 298ms → 8ms). 이는 이미지를 네트워크가 아니라 아니라 cache를 통해 가져오기 때문입니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F8-optimization%2Fimage-22.png?alt=media&token=a30e2a9c-be9a-452a-8a7f-e8d26c7ed7e4)

사용성 측면에서도 이미지를 포함한 모달이 자연스럽게 나타남을 느낄 수 있었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F8-optimization%2Fimage-23.gif?alt=media&token=7aac7a79-6d3e-4366-8b21-c7089138311e)

추가로 고민해 볼 것은 몇 장의 이미지까지 사전 로드를 할 것인가 입니다. 모달의 첫 화면으로 보이는 이미지는 [10번 선수가 드리블 하는 대표사진]뿐만 아니라 하단 썸네일 이미지도 있습니다. 썸네일 이미지까지 사전 로딩할 수도 있지만 그렇게 하면 페이지가 로드될때를 문제를 고려해야 합니다. 브라우저의 리소스를 그만큼 많이 사용하기 때문에 다른 성능 문제를 야기할 수도 있기 때문입니다.

## SK OpenLab을 적극 추천합니다

매우 유익한 시간이었습니다. 지연 로딩, 사전 로딩이 어떤 로직으로 동작하는지, 그리고 해당 로직을 구현할때 고민할 지점이 무엇인지를 배울 수 있었습니다. 그리고 스터디원의 발표를 들으면서 책 내용 이외에 새로운 것들을 배울 수 있어 좋았습니다(레이어, 모듈 크기 확인 방법, `lazy-loading` 속성 등).

SK OpenLab은 스터디에 참여하시는 모든분들에게 저녁, 장소 등 많은 것들을 지원하고 있습니다. [데보션](https://devocean.sk.com/){:target="\_blank"}에서 OpenLab 모집 공고를 확인하실 수 있으니 관심이 있다면 방문해보시길 추천드립니다. 감사합니다.

