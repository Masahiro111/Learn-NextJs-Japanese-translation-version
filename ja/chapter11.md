# 検索とページネーションの追加

前の章では、ストリーミングを使用してダッシュボードの初期読み込みパフォーマンスを向上させました。次に、`/invoices` ページに進み、検索とページネーションを追加する方法を学びましょう。

この章で取り上げるトピックは以下のとおりです。

- Next.js API である searchParams、usePathname、useRouter の使用方法を学びます
- URL 検索パラメータを使用して検索とページネーションを実装します

## コードの書き始め

`/dashboard/invoices/page.tsx` ファイル内に、以下のコードを貼り付けます。

`/app/dashboard/invoices/page.tsx`

```tsx
import Pagination from "@/app/ui/invoices/pagination";
import Search from "@/app/ui/search";
import Table from "@/app/ui/invoices/table";
import { CreateInvoice } from "@/app/ui/invoices/buttons";
import { lusitana } from "@/app/ui/fonts";
import { InvoicesTableSkeleton } from "@/app/ui/skeletons";
import { Suspense } from "react";

export default async function Page() {
  return (
    <div className="w-full">
      <div className="flex w-full items-center justify-between">
        <h1 className={`${lusitana.className} text-2xl`}>Invoices</h1>
      </div>
      <div className="mt-4 flex items-center justify-between gap-2 md:mt-8">
        <Search placeholder="Search invoices..." />
        <CreateInvoice />
      </div>
      {/*  <Suspense key={query + currentPage} fallback={<InvoicesTableSkeleton />}>
        <Table query={query} currentPage={currentPage} />
      </Suspense> */}
      <div className="mt-5 flex w-full justify-center">
        {/* <Pagination totalPages={totalPages} /> */}
      </div>
    </div>
  );
}
```

時間をかけて、作業するページとコンポーネントについてよく理解してください。

1. `<Search/>` を使用すると、ユーザーは特定の請求書を検索できます
1. `<Pagination/>` を使用すると、ユーザーは請求書のページ間を移動できます
1. `<Table/>` は請求書を表示します

検索機能はクライアントとサーバーにまたがります。ユーザーがクライアント上で請求書を検索すると、URL パラメーターが更新され、サーバー上でデータが取得され、新しいデータを使用してテーブルがサーバー上で再レンダリングされます。

## URL 検索パラメータを使用する理由

上述したように、URL 検索パラメータを使用して検索状態を管理します。クライアントサイドのステートで検索を行うことに慣れていると、このパターンは新鮮かもしれません。

URL パラメータを使用して検索を実装すると、次のような利点があります。

- **ブックマークとシェアが可能な URL** ： 検索パラメータは URL に含まれるため、ユーザーは検索クエリとフィルタを含むアプリケーションの現在の状態をブックマークして、将来の参照や共有に使うことができます。
- **サーバーサイドレンダリングと初期ロード** ： URL パラメータをサーバー上で直接使用して初期状態をレンダリングできるため、サーバーレンダリングの処理が容易になります。
- **分析と追跡** : URL へ直接的に検索クエリとフィルターを含めることにより、追加のクライアント側のロジックを必要とせずに、ユーザーの行動を追跡（トラッキング）することが容易になります。

## 検索機能の追加

以下は、検索機能を実装するために使用する Next.js クライアントフックです。

- `useSearchParams` - 現在の URL のパラメータにアクセスできます。たとえば、`/dashboard/invoices?page=1&query=pending` という URL の検索パラメータは `{page: '1', query: 'pending'}` となります
- `usePathname` - 現在の URL のパス名を読み取ることができます。たとえば、ルート `/dashboard/invoices` の場合、`usePathname` は `'/dashboard/invoices'` を返します
- `useRouter` - クライアントコンポーネント内のルート間のナビゲーションをプログラム的に有効にします。使用できる [複数のメソッド](https://nextjs.org/docs/app/api-reference/functions/use-router#userouter) があります

実装手順の概要を次に示します。

1. ユーザーの入力を取得
1. 検索パラメータを使用して URL を更新
1. URL を入力フィールドと同期する
1. 検索クエリを反映するためにテーブルを更新

### 1. ユーザーの入力を取得

`<Search>` コンポーネント（`/app/ui/search.tsx`）に移動すると、次のことに気づくでしょう。

- `"use client"` - これはクライアントコンポーネントで、イベントリスナーとフックを使用できます
- `<input>` - これは検索の入力ボックスです

新しく `handleSearch` 関数を作成し、`onChange` リスナーを `<input>` 要素に追加します。`onChange` は、入力値が変更されるたびに `handleSearch` を呼び出します。

`/app/ui/search.tsx`

```diff tsx
  'use client';

  import { MagnifyingGlassIcon } from '@heroicons/react/24/outline';

  export default function Search({ placeholder }: { placeholder: string }) {
+   function handleSearch(term: string) {
+     console.log(term);
+   }

    return (
      <div className="relative flex flex-1 flex-shrink-0">
        <label htmlFor="search" className="sr-only">
          Search
        </label>
        <input
          className="peer block w-full rounded-md border border-gray-200 py-[9px] pl-10 text-sm outline-2 placeholder:text-gray-500"
          placeholder={placeholder}
+         onChange={(e) => {
+           handleSearch(e.target.value);
+         }}
        />
        <MagnifyingGlassIcon className="absolute left-3 top-1/2 h-[18px] w-[18px] -translate-y-1/2 text-gray-500 peer-focus:text-gray-900" />
      </div>
    );
  }
```

開発者ツールでコンソールを開き、検索フィールドにワードを入力して、正しく動作しているかテストします。検索したワードがコンソールに記録されるはずです。

素晴らしい！ユーザーの検索ワードをキャプチャできましたね。ここで検索ワードを使用して URL を更新する必要があります。

### 2. 検索パラメータを使用して URL を更新

`useSearchParams` フックを `'next/navigation'` からインポートし、変数に割り当てます。

`/app/ui/search.tsx`

```diff tsx
  'use client';

  import { MagnifyingGlassIcon } from '@heroicons/react/24/outline';
+ import { useSearchParams } from 'next/navigation';

  export default function Search() {
+   const searchParams = useSearchParams();

    function handleSearch(term: string) {
      console.log(term);
    }
    // ...
  }
```

`handleSearch` 内で、`searchParams` 変数を使用して、新規に [URLSearchParams](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams) インスタンスを作成します。

`/app/ui/search.tsx`

```diff tsx
  'use client';

  import { MagnifyingGlassIcon } from '@heroicons/react/24/outline';
  import { useSearchParams } from 'next/navigation';

  export default function Search() {
    const searchParams = useSearchParams();

    function handleSearch(term: string) {
+     const params = new URLSearchParams(searchParams);
    }
    // ...
  }
```

`URLSearchParams` は、URL クエリパラメータを操作するためのユーティリティメソッドを提供する Web API です。複雑な文字列リテラルを作成せずに、`?page=1&query=a` のようなパラメータ文字列を取得できます。

次に、ユーザーの入力に基づいたパラメータ文字列をセットします。もし、入力が空の場合は、`delete` メソッドを使用してください。

`/app/ui/search.tsx`

```diff tsx
  'use client';

  import { MagnifyingGlassIcon } from '@heroicons/react/24/outline';
  import { useSearchParams } from 'next/navigation';

  export default function Search() {
    const searchParams = useSearchParams();

    function handleSearch(term: string) {
      const params = new URLSearchParams(searchParams);
+     if (term) {
+       params.set('query', term);
+     } else {
+       params.delete('query');
+     }
    }
    // ...
}
```

これでクエリ文字列が得られました。Next.js の `useRouter` フックと `usePathname` フックを使用して URL を更新できます。

`next/navigation` から `useRouter` と `usePathname` をインポートし、`handleSearch` 内の `useRouter()` から `replace` メソッドを使用します。

`/app/ui/search.tsx`

```diff tsx
  'use client';

  import { MagnifyingGlassIcon } from '@heroicons/react/24/outline';
+ import { useSearchParams, usePathname, useRouter } from 'next/navigation';

  export default function Search() {
    const searchParams = useSearchParams();
+   const pathname = usePathname();
+   const { replace } = useRouter();

    function handleSearch(term: string) {
      const params = new URLSearchParams(searchParams);
      if (term) {
        params.set('query', term);
      } else {
        params.delete('query');
      }
+     replace(`${pathname}?${params.toString()}`);
    }
  }
```

何が起こっているかの説明は次のとおりです。

- `${pathname}` は現在のパスです。今回の場合は、`"/dashboard/invoices"` となります
- ユーザーが検索バーに入力すると、`p​​arams.toString()` がこの入力を URL に適した形式に変換します
- `replace(${pathname}?${params.toString()})` は、URL をユーザーの検索データで更新します。たとえば、ユーザーが「Lee」を検索する場合は、`/dashboard/invoices?query=lee` となります
- Next.js のクライアントサイドナビゲーション（[ページ間のナビゲーション](https://nextjs.org/learn/dashboard-app/navigating-between-pages) の章で学びました）のおかげで、ページをリロードすることなく URL が更新されます

### 3. URL と入力を同期させる

入力フィールドが URL と同期し、共有時に入力フィールドの値を URL 内に入れるために、`searchParams` からクエリを読み取り input に `defaultValue` を渡します。

`/app/ui/search.tsx`

```diff tsx
  <input
    className="peer block w-full rounded-md border border-gray-200 py-[9px] pl-10 text-sm outline-2 placeholder:text-gray-500"
    placeholder={placeholder}
    onChange={(e) => {
      handleSearch(e.target.value);
    }}
+   defaultValue={searchParams.get('query')?.toString()}
  />
```

    `defaultValue` と `value` / 制御されたものと制御されていないもの

    state を使用して入力の値を管理している場合は、value 属性を使用して制御されたコンポーネントにします。これは、React が入力の state を管理することを意味します。

    しかし、state を使用していないため、`defaultValue` を使用できます。これは、ネイティブ入力が自信の state を管理することを意味します。検索クエリを state ではなく URL に保存しているため、これは問題ありません。

### 4. テーブルの更新

最後に、検索クエリを反映させるため、テーブルコンポーネントを更新する必要があります。

請求書ページに戻ってください。

ページコンポーネントは [`searchParams` というプロパティを受け入れる](https://nextjs.org/docs/app/api-reference/file-conventions/page) ので、現在の URL パラメータを `<Table>` コンポーネントに渡すことができます。

`/app/dashboard/invoices/page.tsx`

```diff tsx
  import Pagination from '@/app/ui/invoices/pagination';
  import Search from '@/app/ui/search';
  import Table from '@/app/ui/invoices/table';
  import { CreateInvoice } from '@/app/ui/invoices/buttons';
  import { lusitana } from '@/app/ui/fonts';
  import { Suspense } from 'react';
  import { InvoicesTableSkeleton } from '@/app/ui/skeletons';

+ export default async function Page({
+   searchParams,
+ }: {
+   searchParams?: {
+     query?: string;
+     page?: string;
+   };
+ }) {
+   const query = searchParams?.query || '';
+   const currentPage = Number(searchParams?.page) || 1;

    return (
      <div className="w-full">
        <div className="flex w-full items-center justify-between">
          <h1 className={`${lusitana.className} text-2xl`}>Invoices</h1>
        </div>
        <div className="mt-4 flex items-center justify-between gap-2 md:mt-8">
          <Search placeholder="Search invoices..." />
          <CreateInvoice />
        </div>
+       <Suspense key={query + currentPage} fallback={<InvoicesTableSkeleton />}>
+         <Table query={query} currentPage={currentPage} />
+       </Suspense>
        <div className="mt-5 flex w-full justify-center">
          {/* <Pagination totalPages={totalPages} /> */}
        </div>
      </div>
    );
  }
```

If you navigate to the `<Table>` Component, you'll see that the two props, `query` and `currentPage`, are passed to the `fetchFilteredInvoices()` function which returns the invoices that match the query.

`/app/ui/invoices/table.tsx`

```tsx
// ...
export default async function InvoicesTable({
  query,
  currentPage,
}: {
  query: string;
  currentPage: number;
}) {
  const invoices = await fetchFilteredInvoices(query, currentPage);
  // ...
}
```

With these changes in place, go ahead and test it out. If you search for a term, you'll update the URL, which will send a new request to the server, data will be fetched on the server, and only the invoices that match your query will be returned.

    When to use the `useSearchParams()` hook vs. the `searchParams` prop?

    You might have noticed you used two different ways to extract search params. Whether you use one or the other depends on whether you're working on the client or the server.

    - `<Search>` is a Client Component, so you used the `useSearchParams()` hook to access the params from the client.
    - `<Table>` is a Server Component that fetches its own data, so you can pass the `searchParams` prop from the page to the component.

    As a general rule, if you want to read the params from the client, use the `useSearchParams()` hook as this avoids having to go back to the server.

### Best practice: Debouncing

Congratulations! You've implemented search with Next.js! But there's something you can do to optimize it.

Inside your `handleSearch` function, add the following `console.log`:

`/app/ui/search.tsx`

```diff tsx
  function handleSearch(term: string) {
+   console.log(`Searching... ${term}`);

    const params = new URLSearchParams(searchParams);
    if (term) {
      params.set('query', term);
    } else {
      params.delete('query');
    }
    replace(`${pathname}?${params.toString()}`);
  }
```

Then type "Emil" into your search bar and check the console in dev tools. What is happening?

`Dev Tools Console`

```
Searching... E
Searching... Em
Searching... Emi
Searching... Emil
```

You're updating the URL on every keystroke, and therefore querying your database on every keystroke! This isn't a problem as our application is small, but imagine if your application had thousands of users, each sending a new request to your database on each keystroke.

`Debouncing` is a programming practice that limits the rate at which a function can fire. In our case, you only want to query the database when the user has stopped typing.

    How Debouncing Works:

    1. Trigger Event: When an event that should be debounced (like a keystroke in the search box) occurs, a timer starts.
    1. Wait: If a new event occurs before the timer expires, the timer is reset.
    1. Execution: If the timer reaches the end of its countdown, the debounced function is executed.

You can implement debouncing in a few ways, including manually creating your own debounce function. To keep things simple, we'll use a library called use-debounce.

Install `use-debounce`:

`Terminal`

```
npm i use-debounce
```

In your `<Search>` Component, import a function called `useDebouncedCallback`:

`/app/ui/search.tsx`

```diff tsx
  // ...
+ import { useDebouncedCallback } from 'use-debounce';

  // Inside the Search Component...
+ const handleSearch = useDebouncedCallback((term) => {
    console.log(`Searching... ${term}`);

    const params = new URLSearchParams(searchParams);
    if (term) {
      params.set('query', term);
    } else {
      params.delete('query');
    }
    replace(`${pathname}?${params.toString()}`);
+ }, 300);
```

This function will wrap the contents of handleSearch, and only run the code after a specific time once the user has stopped typing (300ms).

Now type in your search bar again, and open the console in dev tools. You should see the following:

`Dev Tools Console`

```
Searching... Emil
```

By debouncing, you can reduce the number of requests sent to your database, thus saving resources.

## Adding pagination

After introducing the search feature, you'll notice the table displays only 6 invoices at a time. This is because the `fetchFilteredInvoices()` function in data.ts returns a maximum of 6 invoices per page.

Adding pagination allows users to navigate through the different pages to view all the invoices. Let's see how you can implement pagination using URL params, just like you did with search.

Navigate to the `<Pagination/>` component and you'll notice that it's a Client Component. You don't want to fetch data on the client as this would expose your database secrets (remember, you're not using an API layer). Instead, you can fetch the data on the server, and pass it to the component as a prop.

In `/dashboard/invoices/page.tsx`, import a new function called fetchInvoicesPages and pass the `query` from `searchParams` as an argument:

`/app/dashboard/invoices/page.tsx`

```diff tsx
  // ...
+ import { fetchInvoicesPages } from '@/app/lib/data';

  export default async function Page({
    searchParams,
  }: {
    searchParams?: {
      query?: string,
      page?: string,
    },
  }) {
    const query = searchParams?.query || '';
    const currentPage = Number(searchParams?.page) || 1;

+   const totalPages = await fetchInvoicesPages(query);

    return (
      // ...
    );
  }
```

`fetchInvoicesPages` returns the total number of pages based on the search query. For example, if there are 12 invoices that match the search query, and each page displays 6 invoices, then the total number of pages would be 2.

Next, pass the `totalPages` prop to the `<Pagination/>` component:

`/app/dashboard/invoices/page.tsx`

```diff tsx
  // ...

  export default async function Page({
    searchParams,
  }: {
    searchParams?: {
      query?: string;
      page?: string;
    };
  }) {
    const query = searchParams?.query || '';
    const currentPage = Number(searchParams?.page) || 1;

    const totalPages = await fetchInvoicesPages(query);

    return (
      <div className="w-full">
        <div className="flex w-full items-center justify-between">
          <h1 className={`${lusitana.className} text-2xl`}>Invoices</h1>
        </div>
        <div className="mt-4 flex items-center justify-between gap-2 md:mt-8">
          <Search placeholder="Search invoices..." />
          <CreateInvoice />
        </div>
        <Suspense key={query + currentPage} fallback={<InvoicesTableSkeleton />}>
          <Table query={query} currentPage={currentPage} />
        </Suspense>
        <div className="mt-5 flex w-full justify-center">
+         <Pagination totalPages={totalPages} />
        </div>
      </div>
    );
  }
```

Navigate to the `<Pagination/>` component and import the `usePathname` and `useSearchParams` hooks. We will use this to get the current page and set the new page. Make sure to also uncomment the code in this component. Your application will break temporarily as you haven't implemented the `<Pagination/>` logic yet. Let's do that now!

`/app/ui/invoices/pagination.tsx`

```diff tsx
  'use client';

  import { ArrowLeftIcon, ArrowRightIcon } from '@heroicons/react/24/outline';
  import clsx from 'clsx';
  import Link from 'next/link';
  import { generatePagination } from '@/app/lib/utils';
+ import { usePathname, useSearchParams } from 'next/navigation';

  export default function Pagination({ totalPages }: { totalPages: number }) {
+   const pathname = usePathname();
+   const searchParams = useSearchParams();
+   const currentPage = Number(searchParams.get('page')) || 1;

    // ...
  }
```

Next, create a new function inside the `<Pagination>` Component called `createPageURL`. Similarly to the search, you'll use `URLSearchParams` to set the new page number, and `pathName` to create the URL string.

`/app/ui/invoices/pagination.tsx`

```diff tsx
  'use client';

  import { ArrowLeftIcon, ArrowRightIcon } from '@heroicons/react/24/outline';
  import clsx from 'clsx';
  import Link from 'next/link';
  import { generatePagination } from '@/app/lib/utils';
  import { usePathname, useSearchParams } from 'next/navigation';

  export default function Pagination({ totalPages }: { totalPages: number }) {
    const pathname = usePathname();
    const searchParams = useSearchParams();
    const currentPage = Number(searchParams.get('page')) || 1;

+   const createPageURL = (pageNumber: number | string) => {
+     const params = new URLSearchParams(searchParams);
+     params.set('page', pageNumber.toString());
+     return `${pathname}?${params.toString()}`;
+   };

    // ...
  }
```

Here's a breakdown of what's happening:

- `createPageURL` creates an instance of the current search parameters.
- Then, it updates the "page" parameter to the provided page number.
- Finally, it constructs the full URL using the pathname and updated search parameters.

The rest of the `<Pagination>` component deals with styling and different states (first, last, active, disabled, etc). We won't go into detail for this course, but feel free to look through the code to see where `createPageURL` is being called.

Finally, when the user types a new search query, you want to reset the page number to 1. You can do this by updating the `handleSearch` function in your `<Search>` component:

`/app/ui/search.tsx`

```diff tsx
  'use client';

  import { MagnifyingGlassIcon } from '@heroicons/react/24/outline';
  import { usePathname, useRouter, useSearchParams } from 'next/navigation';
  import { useDebouncedCallback } from 'use-debounce';

  export default function Search({ placeholder }: { placeholder: string }) {
    const searchParams = useSearchParams();
    const { replace } = useRouter();
    const pathname = usePathname();

    const handleSearch = useDebouncedCallback((term) => {
      const params = new URLSearchParams(searchParams);
+     params.set('page', '1');
      if (term) {
        params.set('query', term);
      } else {
        params.delete('query');
      }
      replace(`${pathname}?${params.toString()}`);
    }, 300);

```

## Summary

Congratulations! You've just implemented search and pagination using URL Params and Next.js APIs.

To summarize, in this chapter:

- You've handled search and pagination with URL search parameters instead of client state.
- You've fetched data on the server.
- You're using the `useRouter` router hook for smoother, client-side transitions.

These patterns are different from what you may be used to when working with client-side React, but hopefully, you now better understand the benefits of using URL search params and lifting this state to the server.
