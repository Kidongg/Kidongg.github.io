---
title: Next.js Learn - 15. Adding Authentication 번역
date: 2024-06-12 21:00:00 +0900
categories: [Frontend, Next.js Learn]
tags: [Next.js, Next.js conf, Next.js 14]
image: /v0/b/blog-a27f7.appspot.com/o/images%2Fposts%2Fnextjs-conf%2Fnextjs.png?alt=media&token=09247773-9707-4dd1-b3ca-3fe7f943497a
---

> 본 포스팅은 Next.js Learn의 [Adding Authentication](https://nextjs.org/learn/dashboard-app/adding-authentication){:target="\_blank"} 내용을 번역한 것입니다.
{: .prompt-info }

이전 장에서는 form 유효성 검사를 추가하고 접근성을 개선하여 인보이스 경로 구축을 완료했습니다. 이 장에서는 대시보드에 인증을 추가하겠습니다.

## In this chapter
앞으로 다룰 주제는 다음과 같습니다.
- Authentication에 대하여
- NextAuth.js를 사용하여 앱에 인증을 추가하는 방법
- Middleware를 사용하여 사용자를 리다이렉션하고 경로를 보호하는 방법
- React의 `useFormStatus`, `useFormState`를 사용하여 보류 중인 상태와 form 오류를 처리하는 방법

## What is authentication?
인증은 오늘날 많은 웹 애플리케이션에서 핵심적인 부분입니다. 인증은 시스템에서 사용자가 본인이 맞는지 확인하는 방법입니다. 보안 웹사이트에서는 사용자의 신원을 확인하기 위해 여러 가지 방법을 사용하는 경우가 많습니다. 예를 들어 사용자 이름과 비밀번호를 입력한 후 인증 코드를 디바이스로 보내거나 구글 인증기와 같은 외부 앱을 사용할 수 있습니다. 이러한 2단계 인즌(2FA)는 보안을 강화하는 데 도움이 됩니다. 누군가 내 비밀번호를 알아내더라도 고유 토큰 없이는 내 계정에 액세스할 수 없습니다.

### Authentication vs Authorization
웹 개발에서 Authentication(인증)과 Authorization(인가)은 서로 다른 역할을 합니다.
- Authentication은 사용자가 자신이 말한 사람이 맞는지 확인하는 것입니다. 이름과 비밀번호 같은 정보를 통해 신원을 증명하는 것입니다.
- Authorization은 다음 단계입니다. 사용자의 신원이 확인되면 권한 부여를 통해 애플리케이션의 어떤 부분을 사용할 수 있는지 결정합니다.

즉, Authentication은 사용자가 누구인지 확인하고 Authorization은 애플리케이션에서 수행하거나 액세스할 수 있는 작업을 결정합니다.

## Creating the login route
먼저 애플리케이션에 `/login`이라는 새 경로를 만들고 다음 코드를 붙여넣습니다.

```react
// /app/login/page.tsx

import AcmeLogo from '@/app/ui/acme-logo';
import LoginForm from '@/app/ui/login-form';
 
export default function LoginPage() {
  return (
    <main className="flex items-center justify-center md:h-screen">
      <div className="relative mx-auto flex w-full max-w-[400px] flex-col space-y-2.5 p-4 md:-mt-32">
        <div className="flex h-20 w-full items-end rounded-lg bg-blue-500 p-3 md:h-36">
          <div className="w-32 text-white md:w-36">
            <AcmeLogo />
          </div>
        </div>
        <LoginForm />
      </div>
    </main>
  );
}
```
이 장의 뒷부분에서 업데이트할 `<LoginForm>`을 가져오는 페이지를 볼 수 있습니다.

## NextAuth.js
애플리케이션에 인증을 추가하기 위해 [NextAuth.js](https://authjs.dev/reference/nextjs){:target="\_blank"}를 사용할 것입니다. NextAuth.js는 세션 관리, 로그인 및 로그아웃, 기타 인증 측면과 관련된 복잡성을 상당 부분 추상화합니다. 이러한 기능을 수동으로 구현할 수 있지만 시간이 많이 걸리고 오류가 발생하기 쉽습니다. NextAuth.js는 프로세스를 간소화하여 Next.js 애플리케이션에서 인증을 위한 통합 솔루션을 제공합니다.

## Setting NextAuth.js
터미널에서 다음 명령을 실행하여 NextAuth.js를 설치합니다.

```shell
pnpm i next-auth@beta
```

여기에서는 Next.js 14와 호환되는 NextAuth.js `beta` 버전을 설치하는 것입니다. 다음으로 애플리케이션의 비밀 키를 생성합니다. 이 키는 쿠키를 암호화하여 사용자 세션의 보안을 보장하는 데 사용됩니다. 터미널에서 다음 명령을 실행하여 이 작업을 수행할 수 있습니다.

```shell
openssl rand -base64 32
```

그런 다음 `.env` 파일에서 생성한 키를 `AUTH_SECRET` 변수에 추가합니다:

```react
// /.env

AUTH_SECRET=your-secret-key
```

인증이 프로덕션 환경에서 작동하려면 Vercel 프로젝트에서도 환경 변수를 업데이트해야 합니다. Vercel에서 환경 변수를 추가하는 방법에 대한 이 [가이드](https://vercel.com/docs/projects/environment-variables){:target="\_blank"}를 확인하세요.

### Adding the pages option
프로젝트의 루트에서 `authConfig` 객체를 내보내는 `auth.config.ts` 파일을 만듭니다. 이 객체에는 NextAuth.js에 대한 구성 옵션이 포함됩니다. 지금은 `pages` 옵션만 포함됩니다.

```react
// /auth.config.ts

import type { NextAuthConfig } from 'next-auth';
 
export const authConfig = {
  pages: {
    signIn: '/login',
  },
};
```

`pages` 옵션을 사용하여 사용자 정의 로그인, 로그아웃 및 오류 페이지의 경로를 지정할 수 있습니다. 이 옵션은 필수는 아니지만 `pages` 옵션에 `signIn: '/login'`을 추가하면 사용자가 NextAuth.js 기본 페이지가 아닌 사용자 정의 로그인 페이지로 리다이렉션됩니다.

## Protecting your routes with Next.js Middleware
다음으로 경로를 보호하는 로직을 추가합니다. 이렇게 하면 사용자가 로그인하지 않으면 대시보드 페이지에 액세스할 수 없게 됩니다.

```react
// /auth.config.ts

import type { NextAuthConfig } from 'next-auth';
 
export const authConfig = {
  pages: {
    signIn: '/login',
  },
  callbacks: {
    authorized({ auth, request: { nextUrl } }) {
      const isLoggedIn = !!auth?.user;
      const isOnDashboard = nextUrl.pathname.startsWith('/dashboard');
      if (isOnDashboard) {
        if (isLoggedIn) return true;
        return false; // Redirect unauthenticated users to login page
      } else if (isLoggedIn) {
        return Response.redirect(new URL('/dashboard', nextUrl));
      }
      return true;
    },
  },
  providers: [], // Add providers with an empty array for now
} satisfies NextAuthConfig;
```

`authorized` callback은 요청이 [Next.js Middleware](https://nextjs.org/docs/app/building-your-application/routing/middleware){:target="\_blank"}를 통해 페이지에 액세스할 수 있는 권한이 있는지 확인하는 데 사용됩니다. 요청이 완료되기 전에 호출 되며, `auth` 및 `request` 속성을 가진 객체를 받습니다. `auth` 속성에는 사용자의 세션이 포함되고 `request` 속성에는 들어오는 요청이 포함됩니다.

`providers` 옵션은 다양한 로그인 옵션을 나열하는 배열입니다. 지금은 NextAuth config를 충족하기 위해 빈 배열입니다. 이에 대한 자세한 내용은 [Adding the Credentials provider](https://nextjs.org/learn/dashboard-app/adding-authentication#adding-the-credentials-provider){:target="\_blank"} 섹션에서 확인할 수 있습니다.

다음으로 `authConfig` 객체를 미들웨어 파일로 가져와야 합니다. 프로젝트 루트에서 `middleware.ts`라는 파일을 만들고 다음 코드를 붙여넣습니다.

```react
// /middleware.ts

import NextAuth from 'next-auth';
import { authConfig } from './auth.config';
 
export default NextAuth(authConfig).auth;
 
export const config = {
  // https://nextjs.org/docs/app/building-your-application/routing/middleware#matcher
  matcher: ['/((?!api|_next/static|_next/image|.*\\.png$).*)'],
};
```

`authConfig` 객체를 사용하여 NextAuth.js를 초기화하고 `auth` 속성을 내보내고 있습니다. 또한 미들웨어의 `matcher` 옵션을 사용하여 특정 경로에서 실행되도록 지정하고 있습니다. 이 작업에 미들웨어를 사용하면 미들웨어가 인증을 확인할 때까지 보호된 경로가 렌더링을 시작하지 않으므로 애플리케이션의 보안과 성능이 모두 향상된다는 이점이 있습니다.

### Password hashing
비밀번호를 데이터베이스에 저장하기 전에 해싱을 하는 것이 좋습니다. 해싱은 비밀번호를 무작위로 표시되는 고정 길이의 문자열로 변환하여 사용자의 데이터가 노출되더라도 보안 계층을 제공합니다. `seed.js` 파일에서는 데이터베이스에 저장하기 전에 사용자의 비밀번호를 해싱하기 위해 `bcrypt`라는 패키지를 사용했습니다. 이 장의 뒷부분에서 사용자가 입력한 비밀번호를 데이터베이스에 있는 비밀번호와 일치하는지 비교하기 위해 다시 사용할 것입니다. `bcrypt` 패키지를 위한 별도의 파일을 만들어야 합니다. 이는 `bcrypt`가 Next.js 미들웨어에서 사용할 수 없는 Node.js API에 의존하기 때문입니다. 

`authConfig` 객체를 퍼뜨리는 `auth.ts`라는 새 파일을 만듭니다.

```react
// /auth.ts

import NextAuth from 'next-auth';
import { authConfig } from './auth.config';
 
export const { auth, signIn, signOut } = NextAuth({
  ...authConfig,
});
```

### Adding the Credentials provider
다음으로 NextAuth.js에 `providers` 옵션을 추가해야 합니다. 공급자는 Google 또는 Github와 같은 다양한 로그인 옵션을 나열하는 배열입니다. 이 과정에서는 [Credentials provider](https://authjs.dev/getting-started/providers/credentials){:target="\_blank"}만 사용하는 데 중점을 두겠습니다.

Credentials provider을 통해 사용자는 사용자 이름과 비밀번호로 로그인 할 수 있습니다.

```react
// /auth.ts

import NextAuth from 'next-auth';
import { authConfig } from './auth.config';
import Credentials from 'next-auth/providers/credentials';
 
export const { auth, signIn, signOut } = NextAuth({
  ...authConfig,
  providers: [Credentials({})],
});
```

> Good to know  <br /> Credentials provider를 사용하고 있지만 일반적으로 [OAuth](https://authjs.dev/getting-started/authentication/oauth){:target="\_blank"} 또는 [email](https://authjs.dev/getting-started/authentication/email){:target="\_blank"} 공급업체와 같은 대체 공급업체를 사용하는 것이 좋습니다. 전체 옵션 목록은 [NextAuth.js 문서](https://authjs.dev/getting-started/authentication/oauth){:target="\_blank"}를 참조하세요.
{: .prompt-tip }

### Adding the sign in functionality
`authorize` 함수를 사용하여 인증 로직을 처리할 수 있습니다. Server Action과 마찬가지로 `zod`를 사용하여 이메일과 비밀번호의 유효성을 검사한 후 데이터베이스에 사용자가 존재하는지 확인할 수 있습니다.

```react
// /auth.ts

import NextAuth from 'next-auth';
import { authConfig } from './auth.config';
import Credentials from 'next-auth/providers/credentials';
import { z } from 'zod';
 
export const { auth, signIn, signOut } = NextAuth({
  ...authConfig,
  providers: [
    Credentials({
      async authorize(credentials) {
        const parsedCredentials = z
          .object({ email: z.string().email(), password: z.string().min(6) })
          .safeParse(credentials);
      },
    }),
  ],
});
```

자격 증명의 유효성을 검사항 후 데이터베이스에서 사용자를 쿼리하는 새 `getUser` 함수를 만듭니다.

```react
// /auth.ts

import NextAuth from 'next-auth';
import Credentials from 'next-auth/providers/credentials';
import { authConfig } from './auth.config';
import { z } from 'zod';
import { sql } from '@vercel/postgres';
import type { User } from '@/app/lib/definitions';
import bcrypt from 'bcrypt';
 
async function getUser(email: string): Promise<User | undefined> {
  try {
    const user = await sql<User>`SELECT * FROM users WHERE email=${email}`;
    return user.rows[0];
  } catch (error) {
    console.error('Failed to fetch user:', error);
    throw new Error('Failed to fetch user.');
  }
}
 
export const { auth, signIn, signOut } = NextAuth({
  ...authConfig,
  providers: [
    Credentials({
      async authorize(credentials) {
        const parsedCredentials = z
          .object({ email: z.string().email(), password: z.string().min(6) })
          .safeParse(credentials);
 
        if (parsedCredentials.success) {
          const { email, password } = parsedCredentials.data;
          const user = await getUser(email);
          if (!user) return null;
        }
 
        return null;
      },
    }),
  ],
});
```

그런 다음 `bcrypt.compare`를 호출하여 비밀번호가 일치하는지 확인합니다.

```react
// /auth.ts

import NextAuth from 'next-auth';
import Credentials from 'next-auth/providers/credentials';
import { authConfig } from './auth.config';
import { sql } from '@vercel/postgres';
import { z } from 'zod';
import type { User } from '@/app/lib/definitions';
import bcrypt from 'bcrypt';
 
// ...
 
export const { auth, signIn, signOut } = NextAuth({
  ...authConfig,
  providers: [
    Credentials({
      async authorize(credentials) {
        // ...
 
        if (parsedCredentials.success) {
          const { email, password } = parsedCredentials.data;
          const user = await getUser(email);
          if (!user) return null;
          const passwordsMatch = await bcrypt.compare(password, user.password);
 
          if (passwordsMatch) return user;
        }
 
        console.log('Invalid credentials');
        return null;
      },
    }),
  ],
});
```

마지막으로 비밀번호가 일치하면 `user`를 반환하고, 그렇지 않으면 `null`을 반환하여 사용자가 로그인하지 못하도록 합니다.

### Updating the login form
이제 인증 로직을 로그인 form과 연결해야 합니다. `actions.ts` 파일에서 `authenticate`라는 새 작업을 만듭니다. 이 액션은 `auth.ts`에서 `signIn` 함수를 가져와야 합니다.

```react
// /app/lib/actions.ts

import { signIn } from '@/auth';
import { AuthError } from 'next-auth';
 
// ...
 
export async function authenticate(
  prevState: string | undefined,
  formData: FormData,
) {
  try {
    await signIn('credentials', formData);
  } catch (error) {
    if (error instanceof AuthError) {
      switch (error.type) {
        case 'CredentialsSignin':
          return 'Invalid credentials.';
        default:
          return 'Something went wrong.';
      }
    }
    throw error;
  }
}
```

`CredentialsSignin` 오류가 발생하면 적절한 오류 메세지를 표시하고 싶을 것입니다. NextAuth.js 오류에 대한 자세한 내용은 [다음 문서](https://authjs.dev/reference/core/errors){:target="\_blank"}에서 확인할 수 있습니다.

마지막으로 `login-form.tsx` 컴포넌트에서 React의 `useFormState`를 사용하여 서버 액션을 호출하고 form 오류를 처리하고 `useFormStatus`를 사용하여 form의 보류 중인 상태를 처리할 수 있습니다.

```react
// app/ui/login-form.tsx

'use client';
 
import { lusitana } from '@/app/ui/fonts';
import {
  AtSymbolIcon,
  KeyIcon,
  ExclamationCircleIcon,
} from '@heroicons/react/24/outline';
import { ArrowRightIcon } from '@heroicons/react/20/solid';
import { Button } from '@/app/ui/button';
import { useFormState, useFormStatus } from 'react-dom';
import { authenticate } from '@/app/lib/actions';
 
export default function LoginForm() {
  const [errorMessage, dispatch] = useFormState(authenticate, undefined);
 
  return (
    <form action={dispatch} className="space-y-3">
      <div className="flex-1 rounded-lg bg-gray-50 px-6 pb-4 pt-8">
        <h1 className={`${lusitana.className} mb-3 text-2xl`}>
          Please log in to continue.
        </h1>
        <div className="w-full">
          <div>
            <label
              className="mb-3 mt-5 block text-xs font-medium text-gray-900"
              htmlFor="email"
            >
              Email
            </label>
            <div className="relative">
              <input
                className="peer block w-full rounded-md border border-gray-200 py-[9px] pl-10 text-sm outline-2 placeholder:text-gray-500"
                id="email"
                type="email"
                name="email"
                placeholder="Enter your email address"
                required
              />
              <AtSymbolIcon className="pointer-events-none absolute left-3 top-1/2 h-[18px] w-[18px] -translate-y-1/2 text-gray-500 peer-focus:text-gray-900" />
            </div>
          </div>
          <div className="mt-4">
            <label
              className="mb-3 mt-5 block text-xs font-medium text-gray-900"
              htmlFor="password"
            >
              Password
            </label>
            <div className="relative">
              <input
                className="peer block w-full rounded-md border border-gray-200 py-[9px] pl-10 text-sm outline-2 placeholder:text-gray-500"
                id="password"
                type="password"
                name="password"
                placeholder="Enter password"
                required
                minLength={6}
              />
              <KeyIcon className="pointer-events-none absolute left-3 top-1/2 h-[18px] w-[18px] -translate-y-1/2 text-gray-500 peer-focus:text-gray-900" />
            </div>
          </div>
        </div>
        <LoginButton />
        <div
          className="flex h-8 items-end space-x-1"
          aria-live="polite"
          aria-atomic="true"
        >
          {errorMessage && (
            <>
              <ExclamationCircleIcon className="h-5 w-5 text-red-500" />
              <p className="text-sm text-red-500">{errorMessage}</p>
            </>
          )}
        </div>
      </div>
    </form>
  );
}
 
function LoginButton() {
  const { pending } = useFormStatus();
 
  return (
    <Button className="mt-4 w-full" aria-disabled={pending}>
      Log in <ArrowRightIcon className="ml-auto h-5 w-5 text-gray-50" />
    </Button>
  );
}
```

## Adding the logout functionality
`<SideNav />`에 로그아웃 기능을 추가하려면 `<form>` 요소의 `auth.ts`에서 `signOut` 함수를 호출합니다.

```react
// /ui/dashboard/sidenav.tsx

import Link from 'next/link';
import NavLinks from '@/app/ui/dashboard/nav-links';
import AcmeLogo from '@/app/ui/acme-logo';
import { PowerIcon } from '@heroicons/react/24/outline';
import { signOut } from '@/auth';
 
export default function SideNav() {
  return (
    <div className="flex h-full flex-col px-3 py-4 md:px-2">
      // ...
      <div className="flex grow flex-row justify-between space-x-2 md:flex-col md:space-x-0 md:space-y-2">
        <NavLinks />
        <div className="hidden h-auto w-full grow rounded-md bg-gray-50 md:block"></div>
        <form
          action={async () => {
            'use server';
            await signOut();
          }}
        >
          <button className="flex h-[48px] grow items-center justify-center gap-2 rounded-md bg-gray-50 p-3 text-sm font-medium hover:bg-sky-100 hover:text-blue-600 md:flex-none md:justify-start md:p-2 md:px-3">
            <PowerIcon className="w-6" />
            <div className="hidden md:block">Sign Out</div>
          </button>
        </form>
      </div>
    </div>
  );
}
```

## Try it out
이제 사용해 보세요. 아래 자격 증명을 사용하여 애플리케이션에 로그인 및 로그아웃할 수 있어야 합니다.
- Email : `user@nextmail.com`
- Password : `123456`