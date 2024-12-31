---
title: Vercel Blog - Next.js 앱 라우터의 일반적인 실수 및 해결 방법(2024.01.08) 번역
date: 2024-01-29 09:20:00 +0900
categories: [Translation, Vercel Blog]
tags: [Next.js, Next.js conf, Next.js 14, Vercel]
image: /v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnextjs-conf%2Fvercel.jpg?alt=media&token=14eb23a1-68d5-4639-9ed3-c1e82be53d9e
---

> 본 포스팅은 Vercel 공식 블로그의 [Next.js 앱 라우터의 일반적인 실수 및 해결 방법](https://vercel.com/blog/common-mistakes-with-the-next-js-app-router-and-how-to-fix-them){:target="\_blank"}를 번역한 것입니다.
{: .prompt-info }

수 백명의 개발자와 이야기를 나누고 수천개의 Next.js 레포지토리를 살펴본 결과, Next.js 앱 라우터로 빌드할때 흔히 저지르는 10가지 실수를 발견했습니다. <br/>
이 글에서는 이러한 실수가 발생하는 이유와 이를 해결하는 방법, 그리고 새로운 앱 라우터 모델을 이해하는 데 도움이 되는 몇 가지 팁을 공유합니다.

## 1. Using Route Handler with Server Components

서버 컴포넌트에 대한 다음 코드를 살펴봅시다.

```react
// app/page.tsx
// 서버 컴포넌트의 라우트 핸들러에서 JSON 데이터 가져오기

export default async function Page() {
  let res = await fetch('http://localhost:3000/api/data');
  let data = await res.json();
  return <h1>{JSON.stringify(data)}</h1>;
}
```

이 `async` 컴포넌트는 라우트 핸드러에 요청하여 일부 JSON 데이터를 검색합니다.

```react
// app/api/data/route.ts
// 정적 JSON 데이터를 반환하는 라우트 핸들러입니다.

export async function GET(request: Request) {
  return Response.json({ data: 'Next.js' });
}
```

이 접근 방식에는 두 가지 주요 문제가 있습니다.

1. [라우트 핸들러](https://nextjs.org/docs/app/building-your-application/routing/route-handlers){:target="\_blank"}와 [서버 컴포넌트](https://nextjs.org/docs/app/building-your-application/rendering/server-components){:target="\_blank"}는 모두 서버에서 안전하게 실행됩니다. 추가 네트워크 홉이 필요하지 않습니다. 대신 라우트 핸들러 내부에 배치하려는 로직을 서버 컴포넌트에서 직접 호출할 수 있습니다. 이는 외부 API 또는 모든 `Promise`일 수 있습니다.
2. 이 코드는 Node.js를 사용하여 서버에서 실행되므로 `fetch`에 대한 절대 URL과 상대 URL을 제공해야 합니다. 실제로는 `localhost`를 하드코딩하지 않고 현재 환경에 따라 몇 가지 조건부 검사를 수행해야 합니다. 로직을 직접 호출할 수 있으므로 이 작업은 필요하지 않습니다.
   대신 다음을 수행하는 것이 좋습니다.

```react
// app/page.tsx
// 서버 컴포넌트는 데이터를 직접 가져올 수 있습니다.

export default async function Page() {
  // call your async function directly
  let data = await getData(); // { data: 'Next.js' }
  // or call an external API directly
  let data = await fetch('https://api.vercel.app/blog')
  // ...
}
```

## 2. Static or dynamic Route Handlers

라우트 핸들러는 `GET` 메서드를 사용할 때 기본적으로 캐시됩니다. 이는 페이지 라우터와 API 라우트에서 이동하는 기존 Next.js 개발자에게는 종종 혼란을 줄 수 있습니다. <br />
예를 들어 `next build`는 다음 빌드 중에 미리 렌더링됩니다.

```react
// app/api/data/route.ts
// 정적 JSON 데이터를 반환하는 라우트 핸들러입니다.

export async function GET(request: Request) {
  return Response.json({ data: 'Next.js' });
}
```

이 JSON 데이터는 다른 빌드가 완료될때까지 변경되지 않습니다. 왜 그럴까요? 라우트 핸드러를 페이지의 빌딩 블록이라고 생각하면 됩니다. 경로에 대한 특정 요청에 대해 이를 처리하려고 합니다. Next.js에는 라우트 핸들러 위에 페이지 및 레이아웃과 같은 추가 추상화가 있습니다. 이것이 바로 라우트 핸들러가 기본적으로 페이지처럼 [정적이며 동일](https://nextjs.org/docs/app/building-your-application/routing/route-handlers#caching){:target="\_blank"}한 [라우트 세그먼트 구성 옵션](https://nextjs.org/docs/app/api-reference/file-conventions/route-segment-config){:target="\_blank"}을 공유하는 이유입니다. <br />
이 기능은 이전에는 페이지 라우터의 API 라우트에서 사용할 수 없었던 몇 가지 새로운 기능을 제공합니다. 예를 들어 빌드 중에 계산하여 미리 렌더링할 수 있는 JSON 파일이나 `txt`파일 또는 실제로 모든 파일을 생성하는 라우트 핸들러를 사용할 수 있습니다. 그러면 정적으로 생성된 파일이 자동으로 캐시되고 원하는 경우 [주기적으로 업데이트](https://nextjs.org/docs/app/building-your-application/routing/route-handlers#revalidating-cached-data){:target="\_blank"}될 수도 있습니다.

```react
// app/api/data/route.ts

export async function GET(request: Request) {
  let res = await fetch('https://api.vercel.app/blog');
  let data = await res.json();
  return Response.json(data);
}
```

또한 라우트 핸들러는 정적 내보내기와 호환되므로 [정적 파일 호스팅](https://nextjs.org/docs/app/building-your-application/deploying/static-exports){:target="\_blank"}을 지원하는 모든 곳에 Next.js 애플리케이션을 배포할 수 있습니다.

## 3. Route Handlers and Client Components

클라이언트 컴포넌트는 `async`로 표시할 수 없고 데이터를 가져오거나 변경할 수 없기 때문에 라우트 핸들러를 클라이언트 컴포넌트와 함께 사용해야 한다고 생각할 수 있습니다. `fetch`를 작성하고 라우트 핸들러를 생성할 필요 없이 클라이언트 컴포너트에서 직접 [서버 액션](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations){:target="\_blank"}을 호출할 수 있습니다.

```react
// app/user-form.tsx
// 이름을 저장하기 위한 양식 및 입력입니다.

'use client';

import { save } from './actions';

export function UserForm() {
  return (
    <form action={save}>
      <input type="text" name="username" />
      <button>Save</button>
    </form>
  );
}
```

이는 양식과 이벤트 핸들러 모두에서 작동합니다.

```react
// app/user-form.tsx
// 서버 액션은 이벤트 핸들러에서 호출할 수 있습니다.

'use client';

import { save } from './actions';

export function UserForm({ username }) {
  async function onSave(event) {
    event.preventDefault();
    await save(username);
  }

  return <button onClick={onSave}>Save</button>;
}
```

## 4. Using Suspense with Server Components

다음 서버 컴포넌트를 생각해 봅시다. 데이터를 가져오는 동안 표시할 폴백 UI를 정의하기 위해 Suspense를 어디에 배치해야 할까요?

```react
// app/page.tsx
// 데이터를 가져오는 비동기 컴포넌트가 포함된 페이지입니다.

async function BlogPosts() {
  let data = await fetch('https://api.vercel.app/blog');
  let posts = await data.json();
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}

export default function Page() {
  return (
    <section>
      <h1>Blog Posts</h1>
      <BlogPosts />
    </section>
  );
}
```

`Page` 컴포넌트 내부를 추측하셨다면 맞으셨습니다. `Suspense` 경계는 데이터 불러오기를 수행하는 `async` 컴포넌트보다 높은 곳에 위치해야 합니다. 경계가 `async` 컴포넌트 내부에 있으면 작동하지 않습니다.

```react
// app/page.tsx
// 리액트 서버 컴포넌트와 함께 Suspense 사용하기

import { Suspense } from 'react';

async function BlogPosts() {
  let data = await fetch('https://api.vercel.app/blog');
  let posts = await data.json();
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}

export default function Page() {
  return (
    <section>
      <h1>Blog Posts</h1>
      <Suspense fallback={<p>Loading...</p>}>
        <BlogPosts />
      </Suspense>
    </section>
  );
}
```

향후 Partial Prerendering을 사용하면 어떤 컴포넌트를 미리 렌더링하고 어떤 컴포넌트를 온디맨드로 실행할지 정의하는 등 이 패턴이 더욱 보편화될 것입니다.

```react
// 비동기 컴포넌트 내부에서 동적 렌더링을 사용하도록 설정합니다.

import { unstable_noStore as noStore } from 'next/cache';

async function BlogPosts() {
  noStore(); // This component should run dynamically
  let data = await fetch('https://api.vercel.app/blog');
  let posts = await data.json();
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}
```

## 5. Using the incoming request

서버 컴포넌트에서 들어오는 요청 객체에 [액세스할 수 없기 때문에](https://nextjs.org/docs/app#how-can-i-access-the-request-object-in-a-layout){:target="\_blank"} 들어오는 요청의 일부를 읽는 방법이 명확하지 않을 수 있습니다. 이로 인해 `useSearchParams`와 같은 클라이언트 훅을 불필요하게 사용할 수 있습니다. 서버 컴포넌트에는 들어오는 요청에 액세스할 수 있는 특정 함수와 프롭이 있습니다.

- [`cookies()`](https://nextjs.org/docs/app/api-reference/functions/cookies){:target="\_blank"}
- [`headers()`](https://nextjs.org/docs/app/api-reference/functions/headers){:target="\_blank"}
- [`params`](https://nextjs.org/docs/app/api-reference/file-conventions/page#params-optional){:target="\_blank"}
- [`searchParams`](https://nextjs.org/docs/app/api-reference/file-conventions/page#searchparams-optional){:target="\_blank"}

```react
// app/blog/[slug]/page.tsx
// URL의 일부와 검색 매개변수를 읽습니다.

export default function Page({
  params,
  searchParams,
}: {
  params: { slug: string }
  searchParams: { [key: string]: string | string[] | undefined }
}) {
  return <h1>My Page</h1>
}
```

## 6. Using Context providers with App Router

React Context를 사용하거나 컨텍스트에 의존하는 외부 의존성을 사용하고 싶을 수 있습니다. 제가 본 두 가지 일반적인 실수는 서버 컴포넌트(지원되지 않음)와 함께 컨텍스트를 사용하려고 하는 것과 앱 라우터에 공급자를 배치하는 것입니다. <br />
서버 컴포넌트와 클라이언트 컴포넌트가 상호 작용할 수 있도록 하려면 공급자(또는 여러 공급자)를 `children`을 프롭으로 가져와 렌더링하는 별도의 클라이언트 컴포넌트로 만드는 것이 중요합니다. 예를 들어

```react
// app/theme-provider.tsx
// React 컨텍스트를 사용하는 클라이언트 컴포넌트입니다.

'use client';

import { createContext } from 'react';

export const ThemeContext = createContext({});

export default function ThemeProvider({
  children,
}: {
  children: React.ReactNode;
}) {
  return <ThemeContext.Provider value="dark">{children}</ThemeContext.Provider>;
}
```

그런 다음 공급자를 클라이언트 컴포넌트로 별도의 파일에 저장하면 레이아웃 내에서 이 컴포넌트를 가져와서 사용할 수 있습니다.

```react
// app/layout.tsx
// 클라이언트 컨텍스트 제공자와 서버 컴포넌트 자식을 엮는 루트 레이아웃입니다.

import ThemeProvider from './theme-provider';

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html>
      <body>
        <ThemeProvider>{children}</ThemeProvider>
      </body>
    </html>
  );
}
```

공급자가 루트에서 렌더링되면 앱의 다른 모든 클라이언트 컴포넌트가 이 컨텍스트를 사용할 수 있습니다. 특히 이 구성은 트리의 하위에 있는 다른 서버 컴포넌트(`page`포함)를 여전히 허용합니다.

## 7. Using Server ans Client Components together

많은 React 및 Next.js 개발자가 서버 및 클라이언트 컴포넌트를 처음 사용하는 방법을 배우고 있습니다. 이 새로운 모델을 배우는 과정에서 실수도 있을 수 있고, 배울 기회도 있을 것으로 예상됩니다. 예를 들어 다음 페이지를 생각해 보세요.

```react
// app/page.tsx

export default function Page() {
  return (
    <section>
      <h1>My Page</h1>
    </section>
  );
}
```

이것은 서버 컴포넌트입니다. [컴포넌트에서 직접 데이터를 가져올 수 있는 것](https://nextjs.org/docs/app/building-your-application/data-fetching/fetching-caching-and-revalidating){:target="\_blank"}과 같은 새로운 기능이 제공되지만, 특정 클라이언트 측 [React 기능을 사용할 수 없다](https://nextjs.org/docs/app/building-your-application/rendering/composition-patterns){:target="\_blank"}는 의미이기도 합니다. <br />
예를 들어 카운터인 버튼을 만든다고 가정해 보겠습니다. 이 버튼은 상단에 `"use client"` 지시어가 표시된 새 클라이언트 컴포넌트 파일이어야 합니다.

```react
// app/counter.tsx
// 카운트를 증가시키는 클라이언트 컴포넌트 버튼입니다.

'use client';

import { useState } from 'react';

export function Counter() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

그런 다음 페이지에서 이 컴포넌트를 가져와서 사용할 수 있습니다.

```react
// app/page.tsx
// 서버 컴포넌트에서 클라이언트 컴포넌트 사용합니다.

import { Counter } from './counter';

export default function Page() {
  return (
    <section>
      <h1>My Page</h1>
      <Counter />
    </section>
  );
}
```

페이지는 서버 컴포넌트이고 `<Counter>`는 클라이언트 컴포넌트입니다. 멋지네요! 카운터보다 트리에서 아래쪽에 있는 컴포넌트는 어떨까요? 그것도 서버 컴포넌트가 될 수 있을싸요? 예, 구성을 통해 가능합니다.

```react
// app/page.tsx
// 클라이언트 컴포넌트의 자식은 서버 컴포넌트가 될 수 있습니다.

import { Counter } from './counter';

function Message() {
  return <p>This is a Server Component</p>;
}

export default function Page() {
  return (
    <section>
      <h1>My Page</h1>
      <Counter>
        <Message />
      </Counter>
    </section>
  );
}
```

```react
// app/counter.tsx
// 이제 카운터에서 자식을 수락하고 표시합니다.

'use client';

import { useState } from 'react';

export function Counter({ children }: { children: React.ReactNode }) {
  const [count, setCount] = useState(0);
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      {children}
    </div>
  );
}
```

## 8. Adding `use client` unnecessarily

앞의 예시를 바탕으로 모든 곳에 `"use client"` 지시문을 추가해야 한다는 뜻일까요? `"use client"` 지시문을 추가하면 "클라이언트 경계"로 넘어가 클라이언트 측 JavaScript를 실행할 수 있습니다(예 React Hook 또는 State 사용). 클라이언트 컴포넌트는 여전히 [서버에서 미리 렌더링](https://github.com/reactwg/server-components/discussions/4){:target="\_blank"}되며, 이는 Next.js 페이지 라우터의 컴포넌트와 유사합니다. <br />

이미 클라이언트 경계에 있으므로 `<Counter>`의 형제자매는 클라이언트 컴포넌트가 됩니다. 모든 파일에 `"use client"`를 추가할 필요는 없습니다. 이는 [앱 라우터의 점진적 채택](https://nextjs.org/docs/app/building-your-application/upgrading/app-router-migration){:target="\_blank"}을 위해 취한 접근 방식 일 수 있는데, 트리의 위쪽에 있는 컴포넌트가 클라이언트 컴포넌트가 되고 아래쪽에 있는 자식 서버 컴포넌트를 위빙하는 방식입니다.

## 9. Not revalidating data after mutations

Next.js 앱 라우터에는 [데이터 가져오기, 캐싱, 재검증](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations){:target="\_blank"}에 대한 완전한 모델이 포함되어 있습니다. 개발자들이 여전히 이 새로운 모델을 학습하고 있고 피드백을 바탕으로 계속 개선하고 있기 때문에 제가 본 일반적인 실수 중 하나는 변경 후 데이터의 유효성을 다시 검사하는 것을 잊어버리는 것입니다.
예를 들어 다음 서버 컴포넌트를 생각해 보세요. 이 컴포넌트는 [서버 액션](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations){:target="\_blank"}을 사용하여 제출을 처리하고 [Postgres 데이터베이스](https://vercel.com/docs/storage/vercel-postgres){:target="\_blank"}에 새 항목을 만드는 양식을 표시합니다.

```react
// app/page.tsx
// Postgres 데이터베이스에 이름을 삽입하는 서버 작업입니다.

export default function Page() {
  async function create(formData: FormData) {
    'use server';

    let name = formData.get('name');
    await sql`INSERT INTO users (name) VALUES (${name})`;
  }

  return (
    <form action={create}>
      <input name="name" type="text" />
      <button type="submit">Create</button>
    </form>
  );
}
```

양식이 제출되고 삽입이 성공적으로 이루어지면 이름 목록을 표시하는 데이터가 자동으로 업데이트될까요? 아니요, Next.js에 지시하지 않는 한 업데이트는 되지 않습니다. 예를 들어

```react
// app/page.tsx
// 서버 작업 내부의 데이터 재검증

import { revalidatePath } from 'next/cache';

export default async function Page() {
  let names = await sql`SELECT * FROM users`;

  async function create(formData: FormData) {
    'use server';

    let name = formData.get('name');
    await sql`INSERT INTO users (name) VALUES (${name})`;

    revalidatePath('/');
  }

  return (
    <section>
      <form action={create}>
        <input name="name" type="text" />
        <button type="submit">Create</button>
      </form>
      <ul>
        {names.map((name) => (
          <li>{name}</li>
        ))}
      </ul>
    </section>
  );
}
```

## 10. Redirects inside of try/catch blocks

서버 컴포넌트 또는 서버 작업과 같은 서버 측 코드를 실행할 때 리소스를 사용할 수 없거나 변경에 성공한 후 [리다이렉션](https://nextjs.org/docs/app/api-reference/functions/redirect){:target="\_blank"}을 수행해야 할 수 있습니다. <br />
`redirect()` 함수는 TypeScript `never` 타입을 사용하므로 `return redirect()`를 사용할 필요가 없습니다. 또한 내부적으로 이 함수는 Next.js 관련 오류를 발생시킵니다. 즉, try/catch 블록 외부에서 리다이렉션을 처리해야 합니다. 예를 들어 서버 컴포넌트 내부에서 리다이렉션을 시도하는 경우 다음과 같이 보일 수 있습니다.

```react
// app/page.tsx
// 서버 컴포넌트에서 리디렉션하기

import { redirect } from 'next/navigation';

async function fetchTeam(id) {
  const res = await fetch('https://...');
  if (!res.ok) return undefined;
  return res.json();
}

export default async function Profile({ params }) {
  const team = await fetchTeam(params.id);
  if (!team) {
    redirect('/login');
  }

  // ...
}
```

또는 클라이언트 컴포넌트에서 리다이렉션을 시도하는 경우 이벤트 핸들러가 아닌 서버 액션 내부에서 리다이렉션을 수행해야 합니다.

```react
// app/client-redirect.tsx
// 서버 액션을 통해 클라이언트 컴포넌트에서 리디렉션하기

'use client';

import { navigate } from './actions';

export function ClientRedirect() {
  return (
    <form action={navigate}>
      <input type="text" name="id" />
      <button>Submit</button>
    </form>
  );
}
```

```react
// app/actions.ts
// 새 경로로 리디렉션하는 서버 액션입니다.

'use server';

import { redirect } from 'next/navigation';

export async function navigate(data: FormData) {
  redirect('/posts');
}
```

## Conclusion

Next.js 앱 라우터는 React 애플리케이션을 구축하기 위한 개로운 접근 방식이며 몇 가지 새로운 개념을 배워야 합니다. 이러한 실수를 저질렀다고 해서 낙심하지 마세요. 저도 모델이 어떻게 작동하는지 배우면서 실수를 저질렀으니까요. <br />
더 많은 것을 배우고 이 지식을 적용하고 싶다면 [Next.js Learn course](https://nextjs.org/learn){:target="\_blank"}를 통해 앱 라우터로 실제 대시보드 애플리케이션을 구축해 보세요.

## Summary & Opinion

- Summary
  - Server component를 사용할때는 Route Handler를 사용할 필요가 없습니다. Server component를 실행하는 data fetching 로직은 Node.js를 사용하여 서버에서 실행되기 때문입니다.
  - Client component에서 Server action 로직을 실행할 수 있습니다. Server action을 양식 또는 이벤트 핸들러에서 실행할 수 있습니다.
  - Server component에서 들어오는 요청에 액세스할 수 있는 function 및 props는 `cookies()`, `headers()`, `params`, `searchParams`입니다.
  - Provider는 layout에서 감싸야합니다. Provider가 루트에서 렌더링되면 앱의 다른 모든 Client component가 이 Context를 사용할 수 있습니다. 특히 이 구성은 트리의 하위에 있는 다른 Server component(`page` 포함)를 여전히 허용합니다.
  - Client component의 자식은 자동적으로 Client component로 구성됩니다. 하지만 필요하다면 `children`으로 감싸 Server component를 구성할 수 있습니다.
  - Server component에서 revalidate하는 방법은 `revalidatePath()`, `revalidateTag()`의 방법이 있습니다. 그리고 `try`, `catch`문 안에서 `redirect()`를 사용할 수 없습니다.
- Opinion
  - Next.js 앱 라우터를 사용하면서 겪을 수 있는 문제와 해결 방법을 제시해주다니, Vercel 팀은 매우매우 친절한 것 같습니다. [Vercel 유튜브 채널](https://www.youtube.com/watch?v=RBM03RihZVs&t=3s){:target="\_blank"}에서도 추가 학습을 할 수 있어서 좋았습니다. 앞으로도 꾸준히 Next.js 공식 문서와 Vercel 블로그를 트래킹해야 겠습니다.
