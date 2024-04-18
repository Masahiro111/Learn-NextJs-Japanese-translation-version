# 静的レンダリングと動的レンダリング

前章では、ダッシュボードの概要ページのデータを取得しました。しかし、現状のセットアップでの 2 つの制限について簡単に説明したいと思います。

1. データリクエストにより、意図しないウォーターフォールが発生すること
2. ダッシュボードは静的であるため、データの更新はアプリケーションに反映されないこと

この章で取り上げるトピックは以下のとおりです。

- 静的レンダリングとは何か、また静的レンダリングによってアプリケーションのパフォーマンスがどのように向上するか
- 動的レンダリングとは何か、どのような場合に使用するのか
- ダッシュボードを動的にするためのさまざまなアプローチ
- 低速なデータ取得をシミュレートして、何が起こるかを確認する

## 静的レンダリングとは？

静的レンダリングでは、データのフェッチとレンダリングは、ビルド時 (デプロイ時) または [再検証] (https://nextjs.org/docs/app/building-your-application/data-fetching/fetching-caching-and-revalidating#revalidating-data) 時にサーバー上で行われます。結果は、[コンテンツデリバリーネットワーク (CDN)](https://nextjs.org/docs/app/building-your-application/rendering/server-components#static-rendering-default) で配信され、キャッシュされます。

![ページをリクエストするときにユーザーがサーバーではなく CDN にアクセスする様子](/_images/dashboard-route.avif)

ユーザーがアプリケーションにアクセスするたびに、キャッシュされた結果が提供されます。静的レンダリングにはいくつかの利点があります。

- **より高速な Web サイト** - 事前にレンダリングされたコンテンツをキャッシュしてから配信できます。これにより、世界中のユーザーがより迅速かつ確実に Web サイトのコンテンツにアクセスできるようになります
- **サーバー負荷の軽減** - コンテンツがキャッシュされるため、サーバーはユーザーのリクエストごとにコンテンツを動的に生成する必要がありません
- **SEO** - 事前にレンダリングされたコンテンツは、ページの読み込み時にすでに利用可能であるため、検索エンジンのクローラーにとってインデックス付けが容易です。これにより、検索エンジンのランキングが向上につながります

静的レンダリングは、静的なブログ記事や製品ページなど **データのない** または **ユーザー間で共有されるデータ** を含む UI に役立ちます。定期的に更新されるパーソナライズされたデータを含むダッシュボードには適さない可能性があります。

静的レンダリングの反対は動的レンダリングです。

## 動的レンダリングとは？

動的レンダリングでは、**リクエスト時** (ユーザーがページにアクセスした時) に、各ユーザーのコンテンツがサーバー上でレンダリングされます。動的レンダリングにはいくつかの利点があります。

- **リアルタイムなデータ** - 動的レンダリングにより、アプリケーションはリアルタイムまたは頻繁に更新されるデータを表示できます。これは、データが頻繁に変更されるアプリケーションに最適です
- **ユーザー固有のコンテンツ** - ダッシュボードやユーザープロフィールなどのパーソナライズされたコンテンツの提供が容易になり、ユーザーインタラクションに基づいてデータを更新するできます
- **リクエスト時の情報** - 動的レンダリングにより、Cookie や URL 検索パラメーターなど、リクエスト時にしかわからない情報にアクセスできます

## ダッシュボードを動的にする

デフォルトでは、`@vercel/postgres` は独自のキャッシュ セマンティクスを設定しません。これにより、フレームワークは独自の静的および動的動作を設定できるようになります。

サーバーコンポーネントやデータ取得関数の内部で `unstable_noStore` と呼ばれる Next.js API を使用して、静的レンダリングを無効にできます。これを追加してみましょう。

`data.ts` で、`next/cache` から `unstable_noStore` をインポートし、それをデータを取得する関数の先頭で呼び出します。

`/app/lib/data.ts`

```ts diff
  // ...
+ import { unstable_noStore as noStore } from 'next/cache';

  export async function fetchRevenue() {
    // Add noStore() here to prevent the response from being cached.
    // This is equivalent to in fetch(..., {cache: 'no-store'}).
+   noStore();

    // ...
  }

  export async function fetchLatestInvoices() {
+   noStore();
    // ...
  }

  export async function fetchCardData() {
+   noStore();
    // ...
  }

  export async function fetchFilteredInvoices(
    query: string,
    currentPage: number,
  ) {
+   noStore();
    // ...
  }

  export async function fetchInvoicesPages(query: string) {
+   noStore();
    // ...
  }

  export async function fetchFilteredCustomers(query: string) {
+   noStore();
    // ...
  }

  export async function fetchInvoiceById(query: string) {
+   noStore();
    // ...
  }
```

> [!note] > `unstable_noStore` is an experimental API and may change in the future. If you prefer to use a stable API in your own projects, you can also use the [Segment Config Option]() `export const dynamic = "force-dynamic"`.

## Simulating a Slow Data Fetch

Making the dashboard dynamic is a good first step. However... there is still one problem we mentioned in the previous chapter. What happens if one data request is slower than all the others?

Let's simulate a slow data fetch. In your `data.ts` file, uncomment the `console.log` and `setTimeout` inside `fetchRevenue()`:

`/app/lib/data.ts`

```ts diff
export async function fetchRevenue() {
  try {
    // We artificially delay a response for demo purposes.
    // Don't do this in production :)
    +console.log("Fetching revenue data...");
    +(await new Promise((resolve) => setTimeout(resolve, 3000)));

    const data = await sql<Revenue>`SELECT * FROM revenue`;

    +console.log("Data fetch completed after 3 seconds.");

    return data.rows;
  } catch (error) {
    console.error("Database Error:", error);
    throw new Error("Failed to fetch revenue data.");
  }
}
```

Now open http://localhost:3000/dashboard/ in a new tab and notice how the page takes longer to load. In your terminal, you should also see the following messages:

```
Fetching revenue data...
Data fetch completed after 3 seconds.
```

Here, you've added an artificial 3-second delay to simulate a slow data fetch. The result is that now your whole page is blocked while the data is being fetched.

Which brings us to a common challenge developers have to solve:

With dynamic rendering, **your application is only as fast as your slowest data fetch**.
