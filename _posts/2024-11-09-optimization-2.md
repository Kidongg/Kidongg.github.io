---
title: 웹 프론트엔드 성능 최적화 네번째 스터디(레이아웃 시프트, 지연 로딩, 렌더링 최적화, 병목 코드 최적화)
date: 2024-11-10 17:20:00 +0900
categories: [Experience, Study]
tags: [Experience, Diary, Frontend, Study]
image: https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F10-optimization-2%2Fimage-1.gif?alt=media&token=307a4b32-f880-483a-8576-f38714fbe144
---

SK Open Lab에서 운영하는 프론트엔드 성능 최적화 스터디에 참여중입니다. [프론트엔드 성능 최적화 가이드] 책을 읽고 발표하는 방식으로 진행하고 있는데요. 이번주는 [4장. 이미지 갤러리 최적화] 챕터를 공부했습니다. 이미지 지연 로딩, 레이아웃 이동 피하기, 리덕스 렌더링 최적화, 병목 코드 최적화를 중심으로 프론트엔드 최적화 방안을 다루었습니다.

## 이미지 갤러리 최적화 서비스 분석

예시 서비스는 이미지가 여러 개 나열된 사이트입니다. 헤더에 있는 버튼을 누르면 이미지가 변경됩니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F10-optimization-2%2Fimage-2.png?alt=media&token=d82e8483-21eb-4b64-b352-f388c54f8920)

이미지를 클릭하면 이미지 모달이 나타나고, 모달 배경색이 이미지의 전체적인 색깔과 비슷하게 나타납니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F10-optimization-2%2Fimage-3.png?alt=media&token=a91ff667-f132-4a68-bc26-7b4d07d8b279)

## 레이아웃 이동 피하기

### 1. 문제점 : 레이아웃 시프트

레이아웃 시프트는 화면상의 요소 변화로 레이아웃이 갑자기 밀리는 현상을 말합니다. 이미지 갤러리 서비스를 새로고침해보면 이미지가 뚝뚝 끊기면서 밀리는 현상을 발견할 수 있습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F10-optimization-2%2Fimage-23.gif?alt=media&token=3d44b4f0-a21f-434a-b97d-745511c7978a)

레이아웃 시프트는 사용자의 주의를 산만하게 만들고 위치를 순간적으로 변경시키면서 의도와 다른 클릭을 유발하는 단점이 있습니다. 즉, 사용자 경험에 좋지 않은 영향을 미치는 것입니다.

이 때문에 Lighthouse에서는 레이아웃 시프트가 얼마나 발생하는지 나타내는 지표로 CLS(Cumulative Layout Shift)라는 항목을 두고 성능 점수에 포함시키고 있습니다. CLS는 0부터 1까지의 값을 가지며, 레이아웃 시프트가 전혀 발생하지 않은 상태를 0, 그 반대를 1로 계산합니다. 권장하는 점수는 0.1 이하입니다.

이미지 갤러리 서비스를 Lighthouse로 검사를 해보니 성능은 71점, CLS 0.232점이었습니다. 그닥 좋은 점수는 아닌 것이죠.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F10-optimization-2%2Fimage-5.png?alt=media&token=d771dc95-5c01-4193-9342-aa9e1e412005)

문제의 원인은 개발자 도구의 성능 탭에서 확인할 수 있었습니다. 레이아웃 변경 영역을 보니 레이아웃 시프트가 발생한 시점과 해당 요소를 볼 수 있었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F10-optimization-2%2Fimage-6.png?alt=media&token=46129bdc-d873-4eb6-a7ca-97cc4a2d4e49)

레이아웃 시프트가 발생하는 원인은 다양합니다. 그 중 대표적인 경우는 다음과 같습니다.

1. 사이즈가 미리 정의되지 않는 이미지 요소
2. 사이즈가 미리 정의되지 않은 광고 요소
3. 동적으로 삽입된 콘텐츠
4. 웹 폰트(FOIT, FOUT)

이미지 갤러리 서비스에서는 이 네 가지 중 "사이즈가 미리 정의되지 않은 이미지 요소" 때문에 레이아웃 시프트가 발생했습니다. 브라우저가 이미지를 다운로드 하기 전에 이미지 사이즈를 알지 못해 미리 해당 영역을 확보하지 못한 것입니다. 그렇기 때문에 이미지가 화면에 표시되기 전까지는 해당 영역의 높이(또는 너비)가 0이고, 이미지가 로드되면 높이가 해당 이미지의 높이로 변경되면서 다른 요소를 밀어낸 것입니다.

### 2. 해결 : Padding-Hack 기법, Aspect Ratio CSS 프로퍼티

이미지 갤러리의 이미지 사이즈는 브라우저의 가로 사이즈에 따라 변하기 때문에 16:9 비율로 공간을 잡혀있습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F10-optimization-2%2Fimage-7.png?alt=media&token=0b77c49b-4799-4558-90d3-85e33011325f)

이미지를 비율로 설정하는 대표적인 방법은 다음과 같습니다.

1. Padding-Hack 기법

```jsx
<div class="wrapper">
  <img class="image" src="..." />
</div>
```

```css
.wrapper {
  position: relative;
  width: 160px;
  padding-top: 56.25%;
}

.image {
  position: absolute;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
}
```

wrapper의 너비인 160px의 56.26%만큼 상단 여백이 설정하는 방법입니다. 즉, 너비는 160px이 되고 높이는 90px이 되는 것이죠. 이 상태에서 이미지를 absolute로 넣어주면 부모 요소인 div와 사이즈가 동일하게 맞춰집니다. 16:9의 이미지가 화면에 표시되는 것이죠.

2. Aspect Ratio CSS 프로퍼티

```css
.wrapper {
  width: 100%;
  aspect-ratio: 16 /9;
}

.image {
  width: 100%;
  height: 100%;
}
```

Aspect Ratio CSS 프로퍼티는 좀 더 직관적으로 비율을 설정하는 방법입니다. padding의 퍼센트를 매번 개선해야하는 번거로움을 줄여줍니다. 하지만 최신 기술이라 일부 브라우저에 대한 호환성 문제가 있습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F10-optimization-2%2Fimage-9.png?alt=media&token=871c79a1-e2b9-4529-b04d-2f575f6d9667)

저는 브라우저 호환성을 고려해 Padding-Hack 기법으로 레이아웃 시프트를 일으키는 요소의 사이즈를 지정해 주었습니다.

```jsx
function PhotoItem({ photo: { urls, alt } }) {
  const dispatch = useDispatch();

  const openModal = () => {
    dispatch(showModal({ src: urls.full, alt }));
  };

  return (
    <ImageWrap>
      <Image
        src={urls.small + "&t=" + new Date().getTime()}
        alt={alt}
        onClick={openModal}
      />
    </ImageWrap>
  );
}

const ImageWrap = styled.div`
  position: relative;
  width: 100%;
  padding-bottom: 56.25%;
`;

const Image = styled.img`
  cursor: pointer;
  width: 100%;
  position: absolute;
  height: 100%;
  top: 0;
  left: 0;
`;

export default PhotoItem;
```

코드 리팩토링 후 Lighthouse 점수가 성능은 83점(+12), CLS 0점(-0.232)으로 개선되었음을 확인할 수 있었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F10-optimization-2%2Fimage-8.png?alt=media&token=a4984d18-2fd1-44e4-a222-351d91cb80e7)

## 이미지 지연 로딩

### 1. 문제점 : 불필요한 이미지 다운로드

네트워크 탭을 통해 화면 진입 시 불필요한 이미지가 다운로드 되는 것을 확인할 수 있습니다. 필요한 이미지는 18개인데 서버로부터 297개의 이미지를 받아오고 있었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F10-optimization-2%2Fimage-10.png?alt=media&token=f3f73c7e-0ed6-4782-ab86-f4cd6f68fa25)

### 2. 해결 : react-lazyload 라이브러리

react-lazyload 라이브러리는 Intersection Observer API 기능을 추상화하여 제공합니다. Intersection Observer API를 사용해도 지연 로딩을 할 수 있지만 그걸 직접 구현하는 시간을 아낄 수 있습니다.

```
npm install --save react-lazyload
```

```jsx
import LazyLoad from "react-lazyload";

function Component() {
  return (
    <div>
      <LazyLoad>
        <img src="이미지 주소" />
      </LazyLoad>
    </div>
  );
}
```

위와 같이 코드를 작성하면 LazyLoad의 자식으로 들어간 요소들은 화면에 표시되기 전까지는 렌더링되지 않다가 스크롤을 통해 화면에 들어오는 순간 로드가 됩니다.

그래서 저는 다음과 같이 코드를 리팩토링했습니다.

```jsx
import Lazyload from 'react-lazyload'

function PhotoItem({ photo: { urls, alt } }) {
  ...

  return (
    <ImageWrap>
      <Lazyload>
        <Image src="이미지 주소" alt={alt} onClick={openModal} />
      </Lazyload>
    </ImageWrap>
  );
}
```

코드 리팩토링 후 이미지 다운로드 수가 20(-277)으로 개선되었음을 확인할 수 있었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F10-optimization-2%2FImage-11.png?alt=media&token=939a08a4-1121-4db9-8f40-8565b28ab7d1)

하지만 문제가 있었습니다. 이미지가 지연 로드되기 때문에 초기 화면의 리소스를 절약할 수 있는 것은 좋으나, 스크롤을 내려 화면에 이미지가 들어올 때 이미지를 로드하기 때문에 처음에는 이미지가 보이지 않는다는 점입니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F10-optimization-2%2Fimage-12.gif?alt=media&token=31b0bc28-fe9b-4ecf-9b72-ea33453fb1cb)

이 문제를 해결하기 위해서 이미지가 화면에 들어오는 시점보다 조금 더 미리 이미지를 불러와야만 했습니다. react-lazyload 라이브러리에서는 offset이라는 옵션으로 해당 기능을 제공합니다. 이 옵션에는 얼마나 미리 이미지를 로드할지 픽셀값을 넣어주면 됩니다. 예를 들어 offset을 100으로 설정하면 화면에 들어오기 100px 전에 이미지를 로드하는 식입니다.

```jsx
import Lazyload from 'react-lazyload'

function PhotoItem({ photo: { urls, alt } }) {
  ...

  return (
    <ImageWrap>
      <Lazyload offset={300}>
        <Image src="이미지 경로" alt={alt} onClick={openModal} />
      </Lazyload>
    </ImageWrap>
  );
}
```

코드 리팩토링 후 이미지가 나타나기 300px전에 로드가 되었습니다. 이로 인해 스크롤 중 이미지가 보이지 않는 현상을 해결할 수 있었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F10-optimization-2%2Fimage-13.gif?alt=media&token=cacd9104-f20d-47a4-a931-f7da4ec2333c)

## 리덕스 렌더링 최적화

### 1. 문제점 : 불필요한 렌더링

리액트를 렌더링 사이클을 갖습니다. 서비스의 상태가 변경되면 화면에 반영하기 위해 리렌더링 과정을 거칩니다. 그렇기 떄문에 렌더링에 시간이 오래 걸리는 코드가 있거나 렌더링하지 않아도 되는 컴포넌트에서 불필요하게 리렌더링이 발생하면 메인 스레드의 리소스를 차지하여 서비스 성능에 영향을 줍니다.

React Dev Tools를 사용하면 리액트 컴포넌트를 분석할 수 있습니다. React Dev Tools에는 Components와 Profiler라는 패널이 있는데, 각 패널은 다음의 기능을 제공합니다.

- Components : 리액트 컴포넌트를 계층 구조로 탐색할 수 있는 툴
- Profiler : 리액트의 렌더링이 어느 시점에 일어났는지 분석하는 툴

Components 패널의 설정 버튼을 클릭 후 'Highlight updates when components render' 항목을 체크하면 컴포넌트가 어느 시점에 리렌더링이되었는지 파악할 수 있습니다. 이를 통해 렌더링이 필요없는 경우를 파악해 최대한 렌더링이 일어나지 않도록하여 성능을 최적화할 수 있습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F10-optimization-2%2Fimage-14.png?alt=media&token=fe6807a0-bf87-4294-b012-cea9f711eb01)

이미지 갤러리 서비스에서는 불필요한 렌더링이 일어나고 있습니다. 이미지를 클릭해서 이미지 모달을 띄웠을 때 모달만 렌더링되지 않고 모달과 상관이 없는 헤더와 이미지 리스트 컴포넌트가 리렌더링이 되고 있었습니다. 이 현상은 모달을 띄우는 순간, 이미지가 로드된 후 배경 색이 바뀌는 순간, 그리고 모달이 닫히는 순간 총 3번 일어나고 있었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F10-optimization-2%2Fimage-15.gif?alt=media&token=5b188ca9-ffc9-4faf-9c75-89ea4a1d98ec)

이런 원하지 않는 리렌더링이 발생한 이유는 리덕스에서 관리하는 상태 때문이었습니다. 서비스에서 사용하는 이미지 리스트, 헤더의 카테고리, 모달과 관련된 상태는 모두 리덕스에서 관리하고 있습니다. 컴포넌트들이 이 리덕스 상태를 구독하고 있었기 때문에 상태가 변경했을 때를 감지하고 리렌더링이 된것 이었습니다.

이미지 리스트 컨테이너의 `useSelector` 코드를 살펴보면 인자로 들어간 함수가 객체를 반환하는 것을 살펴볼 수 있습니다.

```jsx
const { photos, loading } = useSelector((state) => ({
  photos:
    state.category.category === "all"
      ? state.photos.data
      : state.photos.data.filter(
          (photo) => photo.category === state.category.category
        ),
  loading: state.photos.loading,
}));
```

객체 내부의 photos와 loading의 값을 보면 달라진게 없어 보일 수 있기만 객체를 새로 만들어서 새로운 참조값을 반환하는 형태이므로 `useSelector`는 리덕스를 통해 구독한 값이 변했다고 판단합니다. 따라서 `useSelector`를 사용할때 함수가 객체 형태를 반환하게 하면 매번 새로운 값으로 인지하여 상관없는 리덕스 상태 변화에도 리렌더링이 발생하는 것입니다.

```jsx
const { modalVisible, bgColor, src, alt } = useSelector((state) => ({
  modalVisible: state.imageModal.modalVisible,
  bgColor: state.imageModal.bgColor,
  src: state.imageModal.src,
  alt: state.imageModal.alt,
}));
```

### 2. 해결 : 반환 값 나누기, Equality Function

이 문제를 해결하는 방법은 크게 두 가지가 있습니다.

1. 반환 값을 나누기

```jsx
const modalVisible = useSelector((state) => state.imageModal.modalVisible);
const bgColor = useSelector((state) => state.imageModal.bgColor);
const src = useSelector((state) => state.imageModal.src);
const alt = useSelector((state) => state.imageModal.alt);

const category = useSelector((state) => state.imageModal.category);
```

객체로 묶어서 한 번에 반환하던 것을 단일 값으로 반환하는 방법입니다. 이렇게하면 참조 값이 바뀌는 것이 아니므로 `useSelector`가 반환하는 값은 다른 상태 변화에 영향을 받지 않을 것이고, 리렌더링을 발생시키지 않을 것입니다.

2. Equality Function

Equality Function은 리덕스 상태가 변했을 때 useSelector가 반환해야 하는 값에도 영향을 미쳤는지 판단하는 함수입니다. 쉽게 말해 이전 반환 값과 현재 반환값을 비교하는 함수입니다. 만약 두 값이 동일하면 리더링을 하지 않고, 다르면 리렌더링을 하는 식입니다.

```jsx
const { modalVisible, bgColor, src, alt } = useSelector(
  (state) => ({
    modalVisible: state.imageModal.modalVisible,
    bgColor: state.imageModal.bgColor,
    src: state.imageModal.src,
    alt: state.imageModal.alt,
  }),
  shallowEqual
);
```

```jsx
const { photos, loading } = useSelector(
  (state) => ({
    photos:
      state.category.category === "all"
        ? state.photos.data
        : state.photos.data.filter(
            (photo) => photo.category === state.category.category
          ),
    loading: state.photos.loading,
  }),
  shallowEqual
);
```

`useSelector`의 두 번째 인자로 `shallowEqual`이라는 값을 반환했습니다. 이것은 리덕스에서 제공하는 객체를 얕은 비교하는 함수 입니다. 즉, 참조 값을 비교하는 것이 아니라 객체 내부에 있는 modalVisible, bgColor, src, alt를 직접 비교하여 동일한지 아닌지 판단하는 것입니다. 코드 리펙토링를 리팩토링하니 All 카테고리에서 이미지 모달을 띄울때 이전과 달리 뒤의 헤더의 카테고리와 이미지 리스트가 렌더링되지 않았습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F10-optimization-2%2Fimage-16.gif?alt=media&token=fec3b2a8-5d82-476d-9a0b-c17cbc164ec9)

하지만 All을 제외한 다른 카테고리에서는 여전히 같은 문제가 발생했습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F10-optimization-2%2Fimage-17.gif?alt=media&token=631e37b7-0586-4ec3-b5f5-4c308e8d464e)

그 이유는 All 카테고리가 아니면 `filter` 메소드를 통해 필터링된 이미지를 리스트를 가져왔기 때문입니다. 이때 가져온 이미지 리스트는 새롭게 만들어진 배열이기 때문에 이전에 만들어진 배열과 참조값이 달라서 발생한 문제였습니다. 따라서 `filter`로 새로운 배열을 꺼내는 대신 state.photos.data와 state.category.category를 따로 꺼낸 후 `useSelector` 밖에서 필터링을 해주었습니다.

```jsx
const { category, allPhotos, loading } = useSelector(
  (state) => ({
    category: state.category.category,
    allPhotos: state.photos.data,
    loading: state.photos.loading,
  }),
  shallowEqual
);

const photos =
  category === "all"
    ? allPhotos
    : allPhotos.filter((photo) => photo.category === category);
```

코드 리펙토링 후 다른 카테고리에서만 발생했던 불필요한 리렌더링이 사라졌습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F10-optimization-2%2Fimage-24.gif?alt=media&token=7c9c58b1-a1ed-486c-b2b3-358aaa73e6d7)

## 병목 코드 최적화

### 1. 문제점 : CPU 병목 현상

모달이 뜨는 과정에서 메인 스레드의 작업을 확인해 보았습니다. 이미지를 클릭하고 나서 모달이 뜨고 이미지가 나타기전에 꽤 오랜 시간 자바스크립트 코드가 실행되는 문제가 있었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F10-optimization-2%2Fimage-19.png?alt=media&token=132e7aa6-b788-4f4f-b320-ac6ecb0f5be2)

모달이 뜨고 이미지가 로드된 다음 실행되는 자바스크립트 코드는 `getAverageColorOfImage` 함수입니다. 제일 마지막에 이미지 디코드라는 작업이 보입니다. 이 작업에서 페인팅을 하고 모달이 새롭게 렌더링 되면서 화면의 배경색이 변경되고 있음을 알 수 있습니다.

### 2. 해결 : 메모이제이션, 함수 로직 개선

1. 메모이제이션

메모이제이션은 한 번 실행된 함수에 대해 해당 반환 값을 기억해 두고 있다가 똑같은 조건으로 실행되었을 때 함수의 코드를 모두 실행하지 않고 바로 전에 기억해 둔 값을 반환하는 기술입니다. 여기서 조건이란 함수에서 인자 값을 의미합니다. 동일한 인자가 들어오면 결국 반환 값도 같기 때문입니다.

모달이 나타날때 메모이제이션을 적용하기 위해 다음과 같이 코드를 리펙토링 했습니다.

```js
const cache = {};

export function getAverageColorOfImage(imgElement) {
  if (cache.hasOwnProperty(imgElement.src)) {
    return cache[imgElement.src];
  }

  ...

  cache[imgElement.src] = averageColor;

  return averageColor;
}
```

동일한 이미지에 대해 첫 번째 실행을 했을 때보다 두 번째 실행을 했을 때 배경 색이 확실히 빠르게 바뀌는 것을 확인할 수 있었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F10-optimization-2%2Fimage-20.png?alt=media&token=977ffc75-7f32-46f2-91dc-095af0211c2d)

메모이제이션은 값을 저장하여 재활용하기 때문에 두 번째 실행부터는 성능이 대폭 향상된다는 장점이 있습니다. 그러나 항상 새로운 인자가 들어오는 함수는 메모이제이션을 적용해도 재활용할 수 있는 조건이 충족되지 않기 때문에 오히려 메모리만 잡아먹는 골칫거리가 될 수 있습니다. 따라서 메모이제이션을 적용할 때는 해당 로직이 동일한 조건에서 충분히 반복 실행되는지 먼저 체크해야 합니다.

2. 함수 로직 개선

메모이제이션의 단점은 첫 번째 실행 시에는 성능 향상이 없다는 점입니다. 그래서 첫 번째 실행 시간을 단축하기 위해 `getAverageColorOfImage` 함수 로직을 수정했습니다. 이미지 사이즈에 따라 계산량이 달라지는 로직을 포함하고 있었기 때문에 이미지 크기를 줄이고자 했습니다. DB의 photos의 이미지를 살펴보니 small(섬네일)과 full(원본)이 있었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F10-optimization-2%2FImage-21.png?alt=media&token=9fa81565-e503-45d4-85bf-5c06f0847504)

우선은 full 이미지로 배경색을 계산하고 있던 것을 small 이미지로 바꾸어 주었습니다. 그리고 배경색 계산 시점을 이미지가 다운로드 된 후가 아니라 이미지가 다운로드 되기 전으로 바꾸어 주었습니다.

```jsx
function PhotoItem({ photo: { urls, alt } }) {
  const dispatch = useDispatch();

  const openModal = (e) => {
    dispatch(showModal({ src: urls.full, alt }));

    // 섬네일 이미지로 배경색 계산 후, 리덕스에 저장
    const averageColor = getAverageColorOfImage(e.target);
    dispatch(setBgColor(averageColor));
  };

  return (
    <ImageWrap>
      <Lazyload offset={1000}>
        <Image
          src={urls.small + "&t=" + new Date().getTime()}
          alt={alt}
          onClick={openModal}
          crossOrigin="*"
        />
      </Lazyload>
    </ImageWrap>
  );
}
```

리팩토링 후 배경색을 계산하는 이미지 크기가 줄어들어 계산하는 시간이 줄어들고, 배경색을 미리 계산하여 이미지가 다운로드 되기 전에 배경색이 나타나는 것을 확인할 수 있었습니다.

![Desktop View](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2F10-optimization-2%2Fimage-22.png?alt=media&token=607790ab-c691-4a19-9c30-441be5509325)

## 포스팅을 마무리하며

이번 스터디에서는 레이아웃 시프트 피하기, 메모이제이션의 원리를 살펴볼 수 있어서 좋았습니다. 그리고 React Dev Tools를 사용법을 익힐 수 있어 도움이 많이 되었습니다. 다음 스터디에서는 실제 개발하는 곳에 최적화 기법을 적용해본 것을 공유하기로 했습니다. 다음주 한주 동안 개발 중인 코드에 적용할 수 있는 최적화가 무엇이 있을지 고민해보고 리팩토링을 해보아야겠습니다.
