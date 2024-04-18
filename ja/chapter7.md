# データの取得

データベースを作成しシーディングを行ったので、アプリケーションのデータを取得し、ダッシュボードの概要ページを構築するさまざまな方法について説明しましょう。

この章で取り上げるトピックは以下のとおりです。

- API、ORM、SQL など、データを取得するためのいくつかのアプローチについての説明
- サーバーコンポーネントがバックエンドリソースへのより安全なアクセスにどのように役立つのか
- ネットワークウォーターフォールとは何か
- JavaScript パターンを使用して並列データフェッチを実装する方法

## データを取得する方法の選択

### API レイヤー

API は、アプリケーションコードとデータベースの間の中間層です。API を使用するケースとして以下のようなケースがあります。

- API を提供するサードパーティのサービスを使用している場合
- クライアントからデータを取得する場合、データベースのシークレットキーがクライアントに公開されることを避けるためにサーバー上で実行される API レイヤーが必要

Next.js では、[Route Handlers](https://nextjs.org/docs/app/building-your-application/routing/route-handlers) を使用して API のエンドポイントを作成できます。

### データベースクエリ

フルスタックアプリケーションを作成する場合は、データベースと対話するロジックも作成する必要があります。 Postgres などの [リレーショナルデータベース](https://aws.amazon.com/relational-database/) の場合、SQL または [Prisma](https://www.prisma.io/) などの [ORM](https://vercel.com/docs/storage/vercel-postgres/using-an-orm#) を使用してこれを行うことができます。

データベースクエリを作成する必要があるケースがいくつかあります。

- API エンドポイントを作成するときは、データベースと対話するロジックを作成する必要がある
- React Server Components (サーバー上のデータを取得) を使用している場合は、API レイヤーをスキップして、データベースのシークレットキーをクライアントに公開する危険を冒さずにデータベースに直接クエリを実行できる

React サーバーコンポーネントについて詳しく学びましょう。

## サーバーコンポーネントを使用したデータ取得

デフォルトでは、Next.js アプリケーションは **React Server Components** を使用します。サーバーコンポーネントを使用したデータの取得は比較的新しいアプローチであり、サーバーコンポーネントを使用することでいくつかの利点があります。

- サーバーコンポーネントは Promise をサポートし、データのフェッチなどの非同期タスクに対するよりシンプルなソリューションを提供します。`useEffect`、`useState`、またはデータ取得ライブラリを使用せずに、`async/await` 構文を使用できる
- サーバーコンポーネントはサーバー上で実行されるため、高価なデータ取得やロジックをサーバー上に保持し、結果のみをクライアントに送信できる
- 前述したように、サーバーコンポーネントはサーバー上で実行されるため、API レイヤーを追加することなくデータベースに直接クエリを実行できる

## SQL の使用

このダッシュボードプロジェクトでは、[Vercel Postgres SDK](https://vercel.com/docs/storage/vercel-postgres/sdk) と SQL を使用してデータベースクエリを作成します。SQL を使用する理由はいくつかあります。

- SQL は、リレーショナルデータベースにクエリを実行するための業界標準である (たとえば、ORM は内部で SQL を生成)
- SQL の基本を理解すると、リレーショナルデータベースの基礎を理解するのに役立ち、その知識を他のツールに応用できる
- SQL は多用途であり、特定のデータを取得して操作できる
- Vercel Postgres SDK は、[SQL インジェクション](https://vercel.com/docs/storage/vercel-postgres/sdk#preventing-sql-injections) に対する保護を提供している

これまで SQL を使用したことがなくても、心配する必要はありません。すでにこちら側でクエリが用意されています。

`/app/lib/data.ts` に移動すると、@vercel/postgres から [sql](https://vercel.com/docs/storage/vercel-postgres/sdk#sql) 関数をインポートしていることがわかります。この関数を使用すると、データベースにクエリを実行できます。

`/app/lib/data.ts`

```ts
import { sql } from "@vercel/postgres";

// ...
```

任意のサーバーコンポーネント内で `sql` を呼び出すことができます。ただし、コンポーネントをより簡単に操作できるようにするために、すべてのデータクエリを `data.ts` ファイルに保存し、それらをコンポーネントにインポートできます。

> [!note]
> Chapter 6 で独自のデータベースプロバイダを使用した場合は、プロバイダで動作するようにデータベースクエリを更新する必要があります。クエリは `/app/lib/data.ts` にあります。

## ダッシュボード概要ページのデータ取得

データを取得するさまざまな方法を理解したところで、ダッシュボードの概要ページのデータを取得してみましょう。 `/app/dashboard/page.tsx` に移動し、以下のコードを貼り付け、時間をかけてコードを見てみましょう。

`/app/dashboard/page.tsx`

```tsx
import { Card } from "@/app/ui/dashboard/cards";
import RevenueChart from "@/app/ui/dashboard/revenue-chart";
import LatestInvoices from "@/app/ui/dashboard/latest-invoices";
import { lusitana } from "@/app/ui/fonts";

export default async function Page() {
  return (
    <main>
      <h1 className={`${lusitana.className} mb-4 text-xl md:text-2xl`}>
        Dashboard
      </h1>
      <div className="grid gap-6 sm:grid-cols-2 lg:grid-cols-4">
        {/* <Card title="Collected" value={totalPaidInvoices} type="collected" /> */}
        {/* <Card title="Pending" value={totalPendingInvoices} type="pending" /> */}
        {/* <Card title="Total Invoices" value={numberOfInvoices} type="invoices" /> */}
        {/* <Card
          title="Total Customers"
          value={numberOfCustomers}
          type="customers"
        /> */}
      </div>
      <div className="mt-6 grid grid-cols-1 gap-6 md:grid-cols-4 lg:grid-cols-8">
        {/* <RevenueChart revenue={revenue}  /> */}
        {/* <LatestInvoices latestInvoices={latestInvoices} /> */}
      </div>
    </main>
  );
}
```

上記のコードでは

- Page は **非同期** コンポーネントです。これにより、`await` を使用してデータを取得できるようになります
- `<Card>`、`<RevenueChart>`、`<latestInvoices>` という 3 つのデータを受け取るためのコンポーネントもあります。これらは現在、アプリケーションのエラーを防ぐためにコメントアウトされています。

## `<RevenueChart/>` のデータを取得

`<RevenueChart/>` コンポーネントのデータを取得するには、`data.ts` から `fetchRevenue` 関数をインポートしてからコンポーネント内で呼び出します。

`/app/dashboard/page.tsx`

```tsx diff
  import { Card } from '@/app/ui/dashboard/cards';
  import RevenueChart from '@/app/ui/dashboard/revenue-chart';
  import LatestInvoices from '@/app/ui/dashboard/latest-invoices';
  import { lusitana } from '@/app/ui/fonts';
+ import { fetchRevenue } from '@/app/lib/data';

  export default async function Page() {
+   const revenue = await fetchRevenue();
    // ...
  }
```

次に、`<RevenueChart/>` コンポーネントのコメントを解除し、コンポーネントファイル (`/app/ui/dashboard/revenue-chart.tsx`) に移動して、その中のコードのコメントを解除します。ローカルホストを確認すると、`revenue` データを使用したグラフが表示されるはずです。

![過去 12 か月の総収益を示す収益グラフ](/_images/recent-revenue.avif)

さらにデータクエリのインポートを続けましょう。

## `<LatestInvoices/>` のデータを取得

`<LatestInvoices />` コンポーネントでは、日付順に並べ替えられた最新の 5 件の請求書を取得する必要があります。

すべての請求書情報を取得し、JavaScript でソートすることもできます。今のところはデータ量が小さいため問題ではありませんが、アプリケーションが大きくなるにつれて、リクエストごとに転送されるデータ量と、それをソートするために必要な JavaScript の量が大幅に増える可能性があります。

最新の請求書をメモリ内でソートする代わりに、SQL クエリを使用して直近の 5 件の請求書のみを取得できます。たとえば、以下は `data.ts` ファイルの SQL クエリです。

`/app/lib/data.ts`

```ts
// Fetch the last 5 invoices, sorted by date
const data = await sql<LatestInvoiceRaw>`
  SELECT invoices.amount, customers.name, customers.image_url, customers.email
  FROM invoices
  JOIN customers ON invoices.customer_id = customers.id
  ORDER BY invoices.date DESC
  LIMIT 5`;
```

page ファイルに `fetchLatestInvoices` 関数をインポートしましょう。

`/app/dashboard/page.tsx`

```tsx diff
  import { Card } from "@/app/ui/dashboard/cards";
  import RevenueChart from "@/app/ui/dashboard/revenue-chart";
  import LatestInvoices from "@/app/ui/dashboard/latest-invoices";
  import { lusitana } from "@/app/ui/fonts";
+ import { fetchRevenue, fetchLatestInvoices } from "@/app/lib/data";

  export default async function Page() {
    const revenue = await fetchRevenue();
+   const latestInvoices = await fetchLatestInvoices();
    // ...
  }
```

次に、`<LatestInvoices />` コンポーネントのコメントを解除します。また `/app/ui/dashboard/latest-invoices` にある `<LatestInvoices />` コンポーネント自体の関連するコードのコメントを解除する必要もあります。

ローカルホストにアクセスすると、データベースから最後の 5 件だけが返されていることがわかります。データベースに直接クエリを実行する利点が見えてきたと思います。

![収益グラフと並んだ最新の請求書コンポーネント](/_images/latest-invoices.avif)

## 実践 `<Card>` コンポーネントのデータを取得

次は、`<Card>` コンポーネントのデータを取得してみましょう。カードには次のデータが表示されます。

- 回収された請求書の合計金額
- 保留中の請求書の合計金額
- 請求書の総数
- 顧客の総数

繰り返しになりますが、すべての請求書と顧客を取得し、JavaScript を使用してデータを操作したくなるかもしれません。たとえば、`Array.length` を使用して請求書と顧客の総数を取得するとしましょう。

```ts
const totalInvoices = allInvoices.length;
const totalCustomers = allCustomers.length;
```

しかし SQL を使用すれば、必要なデータのみを取得することができます。`Array.length` を使用するよりも少し時間がかかりますが、リクエスト中に転送するデータが少なくて済みます。以下は SQL の代替です。

`/app/lib/data.ts`

```ts
const invoiceCountPromise = sql`SELECT COUNT(*) FROM invoices`;
const customerCountPromise = sql`SELECT COUNT(*) FROM customers`;
```

インポートする必要がある関数は `fetchCardData` となります。この関数から返される値を再構築する必要があります。

> [!tip]
>
> - カードコンポーネントをチェックして、必要なデータを確認します
> - `data.ts` ファイルをチェックして、関数が何を返すかを確認します

以上を参考にして、最終的なコードは以下のようになります。

`/app/dashboard/page.tsx`

```tsx diff
  import { Card } from '@/app/ui/dashboard/cards';
  import RevenueChart from '@/app/ui/dashboard/revenue-chart';
  import LatestInvoices from '@/app/ui/dashboard/latest-invoices';
  import { lusitana } from '@/app/ui/fonts';
  import {
    fetchRevenue,
    fetchLatestInvoices,
+   fetchCardData,
  } from '@/app/lib/data';

  export default async function Page() {
    const revenue = await fetchRevenue();
    const latestInvoices = await fetchLatestInvoices();
+   const {
+     numberOfInvoices,
+     numberOfCustomers,
+     totalPaidInvoices,
+     totalPendingInvoices,
+   } = await fetchCardData();

    return (
      <main>
        <h1 className={`${lusitana.className} mb-4 text-xl md:text-2xl`}>
          Dashboard
        </h1>
        <div className="grid gap-6 sm:grid-cols-2 lg:grid-cols-4">
          <Card title="Collected" value={totalPaidInvoices} type="collected" />
          <Card title="Pending" value={totalPendingInvoices} type="pending" />
          <Card title="Total Invoices" value={numberOfInvoices} type="invoices" />
          <Card
            title="Total Customers"
            value={numberOfCustomers}
            type="customers"
          />
        </div>
        <div className="mt-6 grid grid-cols-1 gap-6 md:grid-cols-4 lg:grid-cols-8">
          <RevenueChart revenue={revenue} />
          <LatestInvoices latestInvoices={latestInvoices} />
        </div>
      </main>
    );
  }
```

素晴らしい！これで、ダッシュボードの概要ページのすべてのデータが取得できました。ページは以下のようになっていると思います。

![取得されたすべてのデータを含むダッシュボード画面](/_images/complete-dashboard.avif)

ただし...注意しなければならないことが 2 つあります。

1. データリクエストが意図せずお互いにブロックし合い、`リクエストウォーターフォール` を作り出してしまうこと
2. デフォルトでは、Next.js はパフォーマンスを向上させるためにルートをプリレンダリングします。これは「静的レンダリング」と呼ばれます。そのため、データが変更されてもダッシュボードには反映されません。

この章では 1 について説明し、次の章で 2 について詳しく説明します。

## リクエストウォーターフォールについて

「ウォーターフォール」とは、前のリクエストの完了に依存する一連のネットワークリクエストのことです。データ取得の際に、各リクエストは、前のリクエストがデータを返した後にのみ開始できます。

![シーケンシャルなデータフェッチとパラレルなデータフェッチの時間を示す図](/_images/sequential-parallel-data-fetching.avif)

たとえば、`fetchLatestInvoices()` の実行を開始するには、`fetchRevenue()` の実行を待つ必要があります。

`/app/dashboard/page.tsx`

```tsx
const revenue = await fetchRevenue();
const latestInvoices = await fetchLatestInvoices(); // wait for fetchRevenue() to finish
const {
  numberOfInvoices,
  numberOfCustomers,
  totalPaidInvoices,
  totalPendingInvoices,
} = await fetchCardData(); // wait for fetchLatestInvoices() to finish
```

このパターンが必ずしも悪いわけではありません。次のリクエストを行う前に条件を満たす目的でウォーターフォールが必要な場合もあります。たとえば、最初にユーザーの ID とプロフィール情報を取得したい場合があります。ID を取得したら、友達のリストを取得します。この場合、各リクエストは前のリクエストから返されたデータに依存しています。

ただし、この動作は意図的せず、パフォーマンスに影響を与える可能性もあります。

## パラレルなデータフェッチ

ウォーターフォールを回避する一般的な方法は、すべてのデータリクエストを同時に、つまり並行して開始することです。

JavaScript では、[Promise.all()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all) 関数または [Promise.allSettled()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/allSettled) 関数を使用して、すべての Promise を同時に開始できます。たとえば、`data.ts` では、`fetchCardData()` 関数内で `Promise.all()` を使用しています。

`/app/lib/data.js`

```js diff
  export async function fetchCardData() {
    try {
      const invoiceCountPromise = sql`SELECT COUNT(*) FROM invoices`;
      const customerCountPromise = sql`SELECT COUNT(*) FROM customers`;
      const invoiceStatusPromise = sql`SELECT
          SUM(CASE WHEN status = 'paid' THEN amount ELSE 0 END) AS "paid",
          SUM(CASE WHEN status = 'pending' THEN amount ELSE 0 END) AS "pending"
          FROM invoices`;

+     const data = await Promise.all([
+       invoiceCountPromise,
+       customerCountPromise,
+       invoiceStatusPromise,
+     ]);
      // ...
    }
  }
```

このパターンを使用すると、次のことが可能になります。

- すべてのデータ取得の実行を同時に開始することで、パフォーマンスの向上につながる可能性があります
- どのようなライブラリやフレームワークにも適用できるネイティブの JavaScript パターンを使用します。

しかし、この JavaScript パターンだけに頼ることには 1 つの欠点があります。1 つのデータリクエストが他のすべてのデータリクエストよりも遅い場合はどうなるでしょうか？
