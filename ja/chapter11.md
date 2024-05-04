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

`<Table>` コンポーネントに移動すると、2 つのプロパティ、`query` と `currentPage` が、クエリに一致する請求書を返す `fetchFilteredInvoices()` 関数に渡されていることがわかります。

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

これらの変更を行って、テストしてみましょう。用語を検索すると、URL が更新され、新しいリクエストがサーバーに送信され、サーバー上でデータが取得され、クエリに一致する請求書のみが返されます。

    `useSearchParams()` フックと `searchParams` プロパティの使い分けは？

    検索パラメータを抽出するために 2 つの異なる方法を使用していることに気づいたかもしれません。どちらを使用するかは、クライアントとサーバーのどちらで作業しているかによります。

    - `<Search>` はクライアントコンポーネントであるため、クライアントからパラメータにアクセスするには `useSearchParams()` フックを使用しました
    - `<Table>` は独自のデータを取得するサーバーコンポーネントであるため、ページからコンポーネントに `searchParams` プロパティを渡すことができます

    一般的なルールとして、クライアントからパラメータを読み取りたい場合は、`useSearchParams()` フックを使用します。これにより、サーバーに戻る必要がなくなります

### ベストプラクティス： デバウンス

おめでとうございます！Next.js で検索機能を実装できましたね。しかし、それを最適化するためにできることはまだあります。

`handleSearch` 関数内に、次の `console.log` を追加してください。

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

次に、検索バーに「Emil」と入力し、開発ツールのコンソールを確認してください。どうなっているでしょうか？

`Dev Tools Console`

```
Searching... E
Searching... Em
Searching... Emi
Searching... Emil
```

キーストロークごとに URL を更新するため、キーストロークごとにデータベースにクエリを実行することになります。私たちのアプリケーションは小さいため、これは問題ではありませんが、アプリケーションに数千人のユーザーがいて、各ユーザーがキーストロークごとに新しいリクエストをデータベースに送信すると想像してください。

「デバウンス」とは、関数が起動するレートを制限するプログラミング手法です。この例では、ユーザーが入力をやめたときにのみデータベースにクエリの問い合わせを行います。

デバウンスの仕組み

1. トリガーイベント：デバウンスすべきイベント（検索ボックスのキーストロークなど）が発生すると、タイマーが開始します
1. 待機：タイマーが期限切れになる前に新しいイベントが発生すると、タイマーはリセットされます
1. 実行：タイマーのカウントダウンが終了すると、デバウンス関数が実行されます

デバウンスは、独自のデバウンス関数を手動で作成するなど、いくつかの方法で実装できます。シンプルにするために、`use-debounce` というライブラリを使用します。

`use-debounce` をインストールします。

`Terminal`

```
npm i use-debounce
```

`<Search>` コンポーネントで、`useDebouncedCallback` という関数をインポートします。

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

この関数は handleSearch の内容をラップし、ユーザーが入力を停止してから特定の時間（300 ミリ秒）が経過した後にのみコードを実行します。

もう一度、検索バーに入力し、開発ツールでコンソールを開きます。次の内容が表示されるはずです。

`Dev Tools Console`

```
Searching... Emil
```

デバウンスすると、データベースに送信されるリクエストの数を減らし、リソースが節約することができます。

## ページネーションの追加

検索機能を導入すると、表には一度に 6 件の請求書しか表示されないことがわかります。これは、data.ts の `fetchFilteredInvoices()` 関数が 1 ページあたり最大 6 件の請求書を返すためです。

ページネーションを追加すると、ユーザーはさまざまなページを移動してすべての請求書を表示できるようになります。検索の場合と同じように、URL パラメータを使用してページネーションを実装する方法を見てみましょう。

`<Pagination/>` コンポーネントに移動すると、それがクライアントコンポーネントであることがわかります。クライアント上でデータを取得すると、データベースの秘密が漏洩してしまうため、これは望ましくありません（API レイヤーを使用していないことに注意してください）。代わりに、サーバーサイドでデータを取得し、それをプロパティとしてコンポーネントに渡すことができます。

`/dashboard/invoices/page.tsx` で、`fetchInvoicesPages` という新しい関数をインポートし、引数として `searchParams` から `query` を渡します。

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

`fetchInvoicesPages` は、検索クエリに基づいたページの総数を返します。たとえば、検索クエリに一致する請求書が 12 件あり、各ページに 6 件の請求書が表示される場合、総ページ数は 2 ページとなります。

次に、`totalPages` プロパティを `<Pagination/>` コンポーネントに渡します。

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

`<Pagination/>` コンポーネントに移動して、`usePathname` フックと `useSearchParams` フックをインポートします。これを使用して現在のページを取得し、新しいページを設定します。このコンポーネント内のコードのコメントも解除してください。`<Pagination/>` ロジックをまだ実装していないため、アプリケーションは一時的に中断してしまいます。今すぐ実装してみましょう。

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

次に、`<Pagination>` コンポーネント内に `createPageURL` という新しい関数を作成します。検索と同様に、`URLSearchParams` を使用して新しいページ番号を設定し、`pathName` を使用して URL 文字列を作成します。

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

何が起こっているかの内訳は次のとおりです。

- `createPageURL` は、現在の検索パラメータのインスタンスを作成します
- 次に、「page」パラメータを指定されたページ番号に更新します
- 最後に、パス名と更新された検索パラメータを使用して完全な URL を構築します

`<Pagination>` コンポーネントの残りの部分は、スタイルとさまざまな状態（first、last、active、disabled など）を処理します。このコースでは詳細には触れませんが、コードを参照して `createPageURL` がどこで呼び出されているかを確認してください。

最後に、ユーザーが新しい検索クエリを入力したときに、ページ番号を 1 にリセットします。これを行うには、`<Search>` コンポーネントの `handleSearch` 関数を更新します。

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

## まとめ

おめでとうございます！URL Params と Next.js API を使用して、検索とページネーションを実装しました。

この章の内容を要約すると、次のようになります。

- クライアントの state ではなく URL 検索パラメータを使用して検索とページネーションを処理しました
- サーバー上のデータを取得しました
- クライアントサイドの移行をよりスムーズにするために `useRouter` ルーターフックを使用しました

これらのパターンは、クライアントサイドの React を使用するときに慣れ親しんでいるものとは異なりますが、URL 検索パラメータを使用し、この状態をサーバーに移すことの利点をよりよく理解できたことと思います。
