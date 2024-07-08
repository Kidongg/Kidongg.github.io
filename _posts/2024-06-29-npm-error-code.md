---
title: npm error code 1 에러 해결하기(feat. npm install)
date: 2024-06-29 16:30:00 +0900
categories: [Frontend, Experience]
tags: [Node.js, npm, nvm, macOS]
image: https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnpm-error-code-1%2Fnodejs.png?alt=media&token=169b7c52-15f6-41c6-a8c0-840522ccc563
---

## 문제 발생
다른 컴퓨터에서 제가 작성한 코드를 실행해보고 싶었습니다. 그래서 깃허브에 업로드된 코드를 클론하고 `npm install`을 실행했습니다. 그랬더니 다음 에러를 만났습니다.

```shell
npm error code 1
npm error path /.../node_modules/canvas
npm error command failed
npm error command sh -c node-pre-gyp install --fallback-to-build --update-binary
```

`code 1`, `node_module/canvas`, `node-pre-gyp` 에러..? 죄다 처음 보는 것들이었습니다. 에러 메세지를 통해 canvas 또는 node-pre-gyp 문제라는 것을 유추할 수 있었습니다. 그래서 챗선생님의 도움을 받아 문제 해결을 위한 다양한 시도를 해보았습니다.

## 시도했던 방법
### 1. Xcode Command Line Tools 설치
우선은 Xcode Command Line Tools이 잘 설치되었는지 확인을 했습니다. Xcode Command Line Tools은 macOS 개발 환경에서 Xcode를 설치하지 않고도 커맨드 라인에서 macOS 및 Unix 기반 시스템에서 개발 및 빌드를 수행할 수 있게 도와주는 도구입니다. macOS에서 네이티브 모듈을 빌드하기 위해 필수적인 도구입니다.

![img-description](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnpm-error-code-1%2Fxcode.png?alt=media&token=9209507a-90d7-48f0-aa09-370f24c965fd)

### 2. Python 버전 확인 및 업데이트
둘째로는 Python이 설치되었는지 확인했습니다. node-gyp는 빌드 도구로 Python을 필요하기 때문입니다. node는 네이티브 애드온을 지원하며, 이러한 애드온은 C/C++로 작성된 코드를 포함할 수 있습니다.

![img-description](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnpm-error-code-1%2Fpython.png?alt=media&token=1b3f3768-5e9f-4ac2-a92e-e57ad9713661)

### 3. node-gyp 업데이트
node-gyp가 최신 버전인지 확인하고 업데이트를 진행했습니다. node-gyp는 node 환경에서 C++로 작성된 네이티브 애드온을 빌드하고 설치하는 도구이다. node는 JavaScript 런타임 환경이지만, 때로는 JavaScript 외에 C++로 작성된 모듈이나 라이브러리를 사용합니다(ex. 시스템 리소스에 직접 접근할때).

![img-description](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnpm-error-code-1%2Fnode-gyp.png?alt=media&token=754f5b06-e213-43ad-94c2-7b89a2cccf7e)

### 4. npm 캐시 클린
`npm install` 과정에서 캐시 데이터를 읽으면서 문제가 발생했을 수도 있기때문에 캐시를 지웠습니다.

![img-description](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnpm-error-code-1%2Fnpm-cache.png?alt=media&token=a9ca1880-6b5f-40de-9210-41d3ec4bda71)

### 5. canvas 패키지 재설치
`canvas`를 제거하고 다시 설치해 보았습니다. 그럼에도 불구하고 여전히 같은 에러를 뱉었습니다.

```shell
npm uninstall canvas
npm install canvas
```

### 6. node 버전 다운그레이드
node 버전 문제일 확률이 높을 것이라 생각했습니다. 최근에 제 컴퓨터의 node 버전을 업데이트 했던 적이 있었기 때문입니다. 그래서 node 버전(LTS)을 다운그레이드(20.15.0 → 18.20.3 / 18.20.3 →  16.20.2)했습니다.

#### 1) nvm 설치
nvm(node version manager)은 여러 개의 node 버전을 관리하고 전환할 수 있는 도구입니다. 한 컴퓨터에서 여러 버전의 node를 유연하게 사용할 수 있도록 도와줍니다.

```shell
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.4/install.sh | bash
source ~/.nvm/nvm.sh
```

#### 2) 설치 가능한 버전 확인

```shell
nvm ls-remote
```

![img-description](https://firebasestorage.googleapis.com/v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnpm-error-code-1%2Fnode-version.png?alt=media&token=d11278de-e6c7-4517-b150-a6473e205558)

#### 3) 원하는 버전 설치

```shell
nvm install 18.20.3
```

#### 4) 설치된 버전 사용

```shell
nvm use 18.20.3
```

#### 5) 디폴트 버전 설정

```shell
nvm alias default 18.20.3
```
 
### 7. 환경 변수 설정
node 버전 문제가 아니었기 때문에 node-gyp를 사용하기 위한 환경 변수를 설정했습니다. macOS에서는 node-gyp의 알려진 문제를 해결하기 위해 다음과 같이 환경 변수를 설정할 수 있습니다.

```shell
export NODE_OPTIONS=--openssl-legacy-provider
```

## 해결했던 방법 & 느낀점

중간에 node를 지웠다가 다시 설치하고, 버전도 이리저리 바꾸어 보았습니다. 결론적으로는 macOS와 node-gyp의 연결 문제/경로 지정 문제였습니다. 

문제 해결 과정에서 챗선생님의 도움을 받았습니다. 처음부터 챗선생님에게 질문을 잘했더라면 시간을 아낄 수 있었을 것입니다. 질문은 잘하는 것이 개발자의 중요한 역량이라는 점을 다시금 깨닫게 되었다. 의미있는 삽질이었네요.

