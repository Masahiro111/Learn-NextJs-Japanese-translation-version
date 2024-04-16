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

You can call `sql` inside any Server Component. But to allow you to navigate the components more easily, we've kept all the data queries in the `data.ts` file, and you can import them into the components.

> [!note]
> If you used your own database provider in Chapter 6, you'll need to update the database queries to work with your provider. You can find the queries in `/app/lib/data.ts`.

## ダッシュボード概要ページのデータ取得

Now that you understand different ways of fetching data, let's fetch data for the dashboard overview page. Navigate to `/app/dashboard/page.tsx`, paste the following code, and spend some time exploring it:

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

In the code above:

- Page is an **async** component. This allows you to use `await` to fetch data.
- There are also 3 components which receive data: `<Card>`, `<RevenueChart>`, and `<LatestInvoices>`. They are currently commented out to prevent the application from erroring.

## Fetching data for `<RevenueChart/>`

To fetch data for the `<RevenueChart/>` component, import the `fetchRevenue` function from `data.ts` and call it inside your component:

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

Then, uncomment the `<RevenueChart/>` component, navigate to the component file (`/app/ui/dashboard/revenue-chart.tsx`) and uncomment the code inside it. Check your localhost, you should be able to see a chart that uses `revenue` data.

![Revenue chart showing the total revenue for the last 12 months](/_images/recent-revenue.avif)

Let's continue importing some more data queries!

## Fetching data for `<LatestInvoices/>`

For the `<LatestInvoices />` component, we need to get the latest 5 invoices, sorted by date.

You could fetch all the invoices and sort through them using JavaScript. This isn't a problem as our data is small, but as your application grows, it can significantly increase the amount of data transferred on each request and the JavaScript required to sort through it.

Instead of sorting through the latest invoices in-memory, you can use an SQL query to fetch only the last 5 invoices. For example, this is the SQL query from your `data.ts` file:

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

In your page, import the `fetchLatestInvoices` function:

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

Then, uncomment the `<LatestInvoices />` component. You will also need to uncomment the relevant code in the `<LatestInvoices />` component itself, located at `/app/ui/dashboard/latest-invoices`.

If you visit your localhost, you should see that only the last 5 are returned from the database. Hopefully, you're beginning to see the advantages of querying your database directly!

![Latest invoices component alongside the revenue chart](/_images/latest-invoices.avif)

## Practice: Fetch data for the `<Card>` components

Now it's your turn to fetch data for the `<Card>` components. The cards will display the following data:

- Total amount of invoices collected.
- Total amount of invoices pending.
- Total number of invoices.
- Total number of customers.

Again, you might be tempted to fetch all the invoices and customers, and use JavaScript to manipulate the data. For example, you could use `Array.length` to get the total number of invoices and customers:

```ts
const totalInvoices = allInvoices.length;
const totalCustomers = allCustomers.length;
```

But with SQL, you can fetch only the data you need. It's a little longer than using `Array.length`, but it means less data needs to be transferred during the request. This is the SQL alternative:

`/app/lib/data.ts`

```ts
const invoiceCountPromise = sql`SELECT COUNT(*) FROM invoices`;
const customerCountPromise = sql`SELECT COUNT(*) FROM customers`;
```

The function you will need to import is called `fetchCardData`. You will need to destructure the values returned from the function.

> [tips]
>
> - Check the card components to see what data they need.
> - Check the data.ts file to see what the function returns.

Once you're ready, expand the toggle below for the final code:

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

Great! You've now fetched all the data for the dashboard overview page. Your page should look like this:

![Dashboard page with all the data fetched](/_images/complete-dashboard.avif)

However... there are two things you need to be aware of:

1. The data requests are unintentionally blocking each other, creating a `request waterfall`.
2. By default, Next.js `prerenders` routes to improve performance, this is called `Static Rendering`. So if your data changes, it won't be reflected in your dashboard.

Let's discuss number 1 in this chapter, then look into detail at number 2 in the next chapter.

## What are request waterfalls?

A "waterfall" refers to a sequence of network requests that depend on the completion of previous requests. In the case of data fetching, each request can only begin once the previous request has returned data.

![Diagram showing time with sequential data fetching and parallel data fetching](/_images/sequential-parallel-data-fetching.avif)

For example, we need to wait for `fetchRevenue()` to execute before `fetchLatestInvoices()` can start running, and so on.

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

This pattern is not necessarily bad. There may be cases where you want waterfalls because you want a condition to be satisfied before you make the next request. For example, you might want to fetch a user's ID and profile information first. Once you have the ID, you might then proceed to fetch their list of friends. In this case, each request is contingent on the data returned from the previous request.

However, this behavior can also be unintentional and impact performance.

## Parallel data fetching

A common way to avoid waterfalls is to initiate all data requests at the same time - in parallel.

In JavaScript, you can use the [Promise.all()]() or [Promise.allSettled()]() functions to initiate all promises at the same time. For example, in data.ts, we're using `Promise.all()` in the `fetchCardData()` function:

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

By using this pattern, you can:

- Start executing all data fetches at the same time, which can lead to performance gains.
- Use a native JavaScript pattern that can be applied to any library or framework.

However, there is one `disadvantage` of relying only on this JavaScript pattern: what happens if one data request is slower than all the others?
