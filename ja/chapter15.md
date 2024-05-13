# 認証機能の追加

前の章では、フォームのバリデーションを追加し、アクセシビリティを改善することで請求書ルートの構築を完了しました。この章では、ダッシュボードに認証機能を追加します。

ここで取り上げるトピックは次のとおりです

- 認証とは何か
- NextAuth.js を使用してアプリに認証を追加する方法
- ミドルウェアを使用してユーザーをリダイレクトし、ルートを保護する方法
- React の `useFormStatus` と `useFormState` を使用して保留状態やフォームエラーを処理する方法

## 認証とは？

認証は、今日の多くの web アプリケーションの重要な役割を担っています。これは、ユーザーが本人であるかどうかをシステムがチェックする方法です。

セキュアな Web サイトでは、多くの場合、ユーザーの身元を確認するために複数の方法が使用さることがよくあります。たとえば、ユーザー名とパスワードを入力すると、サイトからデバイスに確認コードが送信されたり、Google Authenticator のような外部アプリを使用したりします。この 2 要素認証（2FA）はセキュリティの向上に役立ちます。たとえ誰かがあなたのパスワードを知ったとしても、あなたの固有のトークンがなければあなたのアカウントにアクセスすることはできません。

### 認証と認可

Web 開発では、認証と認可は異なる役割を果たします。

- **認証** は、ユーザーが本人であることを確認することです。ユーザー名やパスワードなど、自分が持っているもので自分の身元を証明することになります。
- **認可** は次のステップです。ユーザーの身元が確認されると、認可によってアプリケーションのどの部分の使用が許可されるかを決定します。

つまり、認証はあなたが誰であるかをチェックし、認可はあなたがアプリケーションで何ができるのか、何にアクセスできるのかを決定します。

## ログインルートの作成

まず、アプリケーションに `/login` という名前の新しいルートを作成し、次のコードを貼り付けます。

`/app/login/page.tsx`

```tsx
import AcmeLogo from "@/app/ui/acme-logo";
import LoginForm from "@/app/ui/login-form";

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

このページで `<LoginForm />` がインポートされていますが、これはこの章の後半で更新します。

## NextAuth.js

アプリケーションに認証を追加するために、[NextAuth.js](https://nextjs.authjs.dev/) を使います。NextAuth.js は、セッション管理、サインイン、サインアウトなど、 認証にかかわる複雑な処理の多くを抽象化してくれます。手作業でこれらの機能を実装することもできますが、そのプロセスには時間がかかり、エラーも発生しがちです。NextAuth.js はこのプロセスを簡略化し、Next.js アプリケーションにおける認証のための統一されたソリューションを提供します。

## NextAuth.js のセットアップ

ターミナルで次のコマンドを実行して、NextAuth.js をインストールします。

`_ Terminal`

```
npm install next-auth@beta
```

ここでは、Next.js 14 と互換性のある NextAuth.js の `beta` 版をインストールします。

次に、アプリケーションの秘密鍵を生成します。このキーは Cookie の暗号化に使用され、ユーザーセッションのセキュリティが確保されます。ターミナルで次のコマンドを実行してください。

`_ Terminal`

```
openssl rand -base64 32
```

次に、`.env` ファイルで、生成されたキーを `AUTH_SECRET` 変数に追加します。

`.env`

```diff
+ AUTH_SECRET=your-secret-key
```

実稼働環境で認証を機能させるには、Vercel プロジェクトの環境変数も更新する必要があります。Vercel に環境変数を追加する方法については、こちらの [ガイド](https://vercel.com/docs/projects/environment-variables) を確認してください。

### ページオプションの追加

プロジェクトのルートに `auth.config.ts` ファイルを作成し、`authConfig` オブジェクトをエクスポートする。このオブジェクトには NextAuth.js の設定オプションが含まれます。今のところ、`pages` オプションだけが含まれています。

`/auth.config.ts`

```ts
import type { NextAuthConfig } from "next-auth";

export const authConfig = {
  pages: {
    signIn: "/login",
  },
};
```

`pages` オプションを使うと、カスタムサインイン、カスタムサインアウト、カスタムエラーページのルートを指定することができます。これは必須ではありませんが、`pages` オプションに `signIn: '/login'` を追加することで、ユーザは NextAuth.js のデフォルトページではなく、カスタムログインページにリダイレクトされます。

## Next.js ミドルウェアでルートを保護する

次に、ルートを保護するロジックを追加します。ログインしていないユーザーがダッシュボードのページにアクセスできないようにします。

`/auth.config.ts`

```diff ts
  import type { NextAuthConfig } from 'next-auth';

  export const authConfig = {
    pages: {
      signIn: '/login',
    },
+   callbacks: {
+     authorized({ auth, request: { nextUrl } }) {
+       const isLoggedIn = !!auth?.user;
+       const isOnDashboard = nextUrl.pathname.startsWith('/dashboard');
+       if (isOnDashboard) {
+         if (isLoggedIn) return true;
+         return false; // Redirect unauthenticated users to login page
+       } else if (isLoggedIn) {
+         return Response.redirect(new URL('/dashboard', nextUrl));
+       }
+       return true;
+     },
+   },
+   providers: [], // Add providers with an empty array for now
+ } satisfies NextAuthConfig;
```

`authorized` コールバックは、リクエストが [Next.js Middleware](https://nextjs.org/docs/app/building-your-application/routing/middleware) を経由したページへのアクセスを許可されているかどうかを確認するために使用します。このコールバックはリクエストが完了する前に呼び出され、`auth` プロパティと `request` プロパティを持つオブジェクトを受け取ります。`auth` プロパティにはユーザーのセッションが格納され、`request` プロパティには受信したリクエストが格納されます。

`providers` オプションは、さまざまなログインオプションをリストする配列となっています。現時点では、NextAuth の設定を満たすための空の配列です。詳細については、[認証情報プロバイダの追加](https://nextjs.org/learn/dashboard-app/adding-authentication#adding-the-credentials-provider) のセクションで説明します。

次に、`authConfig` オブジェクトをミドルウェアファイルにインポートします。プロジェクトのルートに `middleware.ts` というファイルを作成し、次のコードを貼り付けます。

`/middleware.ts`

```ts
import NextAuth from "next-auth";
import { authConfig } from "./auth.config";

export default NextAuth(authConfig).auth;

export const config = {
  // https://nextjs.org/docs/app/building-your-application/routing/middleware#matcher
  matcher: ["/((?!api|_next/static|_next/image|.*\\.png$).*)"],
};
```

ここでは、NextAuth.js を `authConfig` オブジェクトで初期化し、`auth` プロパティをエクスポートしています。また、ミドルウェアの `matcher` オプションを使って、特定のパスで実行するように指定しています。

このタスクにミドルウェアを使用する利点は、ミドルウェアが認証を確認するまで保護されたルートのレンダリングが開始されないため、アプリケーションのセキュリティとパフォーマンスの両方が向上することです。

### パスワードのハッシュ化

パスワードをデータベースに保存する前に、**ハッシュ化** することをお勧めします。ハッシュ化により、パスワードがランダムに見える固定長の文字列に変換され、ユーザーのデータが漏洩した場合でもセキュリティ層を確保することができます。

`seed.js` ファイルでは、`bcrypt` というパッケージを使用して、ユーザーのパスワードをデータベースに保存する前にハッシュ化しました。この章の後半で、ユーザが入力したパスワードがデータベースのパスワードと一致するかどうかを比較するために、再びこのパッケージを使用することになります。ただし、`bcrypt` パッケージ用に別のファイルを作成する必要があります。`bcrypt` は Next.js ミドルウェアでは利用できない Node.js API に依存しているからです。

auth.ts という新しいファイルを作成し、`authConfig` オブジェクトを展開します。

`/auth.ts`

```ts
import NextAuth from "next-auth";
import { authConfig } from "./auth.config";

export const { auth, signIn, signOut } = NextAuth({
  ...authConfig,
});
```

### 認証情報プロバイダの追加

次に、NextAuth.js に `providers` オプションを追加します。`providers` は、Google や GitHub など、さまざまなログインオプションをリストする配列です。このコースでは、[認証情報プロバイダ](https://authjs.dev/getting-started/providers/credentials-tutorial) のみに焦点を当てます。

認証情報プロバイダを使用すると、ユーザーはユーザー名とパスワードを使用してログインできます。

`/auth.ts`

```diff ts
  import NextAuth from 'next-auth';
  import { authConfig } from './auth.config';
+ import Credentials from 'next-auth/providers/credentials';

  export const { auth, signIn, signOut } = NextAuth({
    ...authConfig,
+   providers: [Credentials({})],
  });
```

> [!tip]
>
> Although we're using the Credentials provider, it's generally recommended to use alternative providers such as [OAuth](https://authjs.dev/getting-started/providers/oauth-tutorial) or [email](https://authjs.dev/getting-started/providers/email-tutorial) providers. See the NextAuth.js docs for a full list of options.

### Adding the sign in functionality

You can use the `authorize` function to handle the authentication logic. Similarly to Server Actions, you can use `zod` to validate the email and password before checking if the user exists in the database:

`/auth.ts`

```diff ts
  import NextAuth from 'next-auth';
  import { authConfig } from './auth.config';
  import Credentials from 'next-auth/providers/credentials';
+ import { z } from 'zod';

  export const { auth, signIn, signOut } = NextAuth({
    ...authConfig,
    providers: [
+     Credentials({
+       async authorize(credentials) {
+         const parsedCredentials = z
+           .object({ email: z.string().email(), password: z.string().min(6) })
+           .safeParse(credentials);
+       },
+     }),
    ],
  });
```

After validating the credentials, create a new `getUser` function that queries the user from the database.

`/auth.ts`

```diff ts
  import NextAuth from 'next-auth';
  import Credentials from 'next-auth/providers/credentials';
  import { authConfig } from './auth.config';
  import { z } from 'zod';
+ import { sql } from '@vercel/postgres';
+ import type { User } from '@/app/lib/definitions';
+ import bcrypt from 'bcrypt';

+ async function getUser(email: string): Promise<User | undefined> {
+   try {
+     const user = await sql<User>`SELECT * FROM users WHERE email=${email}`;
+     return user.rows[0];
+   } catch (error) {
+     console.error('Failed to fetch user:', error);
+     throw new Error('Failed to fetch user.');
+   }
+ }

  export const { auth, signIn, signOut } = NextAuth({
    ...authConfig,
    providers: [
      Credentials({
        async authorize(credentials) {
          const parsedCredentials = z
            .object({ email: z.string().email(), password: z.string().min(6) })
            .safeParse(credentials);

+         if (parsedCredentials.success) {
+           const { email, password } = parsedCredentials.data;
+           const user = await getUser(email);
+           if (!user) return null;
          }

          return null;
        },
      }),
    ],
  });
```

Then, call `bcrypt.compare` to check if the passwords match:

`/auth.ts`

```diff ts
  import NextAuth from 'next-auth';
  import Credentials from 'next-auth/providers/credentials';
  import { authConfig } from './auth.config';
  import { sql } from '@vercel/postgres';
  import { z } from 'zod';
  import type { User } from '@/app/lib/definitions';
+ import bcrypt from 'bcrypt';

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
+           const passwordsMatch = await bcrypt.compare(password, user.password);

+           if (passwordsMatch) return user;
          }

+         console.log('Invalid credentials');
+         return null;
        },
      }),
    ],
  });
```

Finally, if the passwords match you want to return the user, otherwise, return `null` to prevent the user from logging in.

### Updating the login form

Now you need to connect the auth logic with your login form. In your `actions.ts` file, create a new action called `authenticate`. This action should import the `signIn` function from `auth.ts`:

`/app/lib/actions.ts`

```ts
import { signIn } from "@/auth";
import { AuthError } from "next-auth";

// ...

export async function authenticate(
  prevState: string | undefined,
  formData: FormData
) {
  try {
    await signIn("credentials", formData);
  } catch (error) {
    if (error instanceof AuthError) {
      switch (error.type) {
        case "CredentialsSignin":
          return "Invalid credentials.";
        default:
          return "Something went wrong.";
      }
    }
    throw error;
  }
}
```

If there's a `'CredentialsSignin'` error, you want to show an appropriate error message. You can learn about NextAuth.js errors in [the documentation](https://errors.authjs.dev/)

Finally, in your `login-form.tsx` component, you can use React's `useFormState` to call the server action and handle form errors, and use `useFormStatus` to handle the pending state of the form:

`app/ui/login-form.tsx`

```diff tsx
+ 'use client';

  import { lusitana } from '@/app/ui/fonts';
  import {
    AtSymbolIcon,
    KeyIcon,
    ExclamationCircleIcon,
  } from '@heroicons/react/24/outline';
  import { ArrowRightIcon } from '@heroicons/react/20/solid';
  import { Button } from '@/app/ui/button';
+ import { useFormState, useFormStatus } from 'react-dom';
+ import { authenticate } from '@/app/lib/actions';

  export default function LoginForm() {
+   const [errorMessage, dispatch] = useFormState(authenticate, undefined);

    return (
+     <form action={dispatch} className="space-y-3">
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
+         <div
+           className="flex h-8 items-end space-x-1"
+           aria-live="polite"
+           aria-atomic="true"
+         >
+           {errorMessage && (
+             <>
+               <ExclamationCircleIcon className="h-5 w-5 text-red-500" />
+               <p className="text-sm text-red-500">{errorMessage}</p>
+             </>
+           )}
+         </div>
        </div>
      </form>
    );
  }

  function LoginButton() {
+   const { pending } = useFormStatus();

    return (
+     <Button className="mt-4 w-full" aria-disabled={pending}>
+       Log in <ArrowRightIcon className="ml-auto h-5 w-5 text-gray-50" />
+     </Button>
    );
  }
```

## Adding the logout functionality

To add the logout functionality to `<SideNav />`, call the `signOut` function from `auth.ts` in your `<form>` element:

`/ui/dashboard/sidenav.tsx`

```diff tsx
  import Link from 'next/link';
  import NavLinks from '@/app/ui/dashboard/nav-links';
  import AcmeLogo from '@/app/ui/acme-logo';
  import { PowerIcon } from '@heroicons/react/24/outline';
+ import { signOut } from '@/auth';

  export default function SideNav() {
    return (
      <div className="flex h-full flex-col px-3 py-4 md:px-2">
        // ...
        <div className="flex grow flex-row justify-between space-x-2 md:flex-col md:space-x-0 md:space-y-2">
          <NavLinks />
          <div className="hidden h-auto w-full grow rounded-md bg-gray-50 md:block"></div>
          <form
+           action={async () => {
+             'use server';
+             await signOut();
+           }}
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

Now, try it out. You should be able to log in and out of your application using the following credentials:

- Email: `user@nextmail.com`
- Password: `123456`
