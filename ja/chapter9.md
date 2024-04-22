# ストリーミング

前の章では、ダッシュボードページを動的にしましたが、データ取得の遅さがアプリケーションのパフォーマンスにどのような影響を与える可能性があるかについて説明しました。データリクエストが遅い場合にユーザーエクスペリエンスを改善させる方法を見てみましょう。

この章で取り上げるトピックは以下のとおりです。

- ストリーミングとは何か。また、いつ使用するのか
- `loading.tsx` と Suspense を使用したストリーミングの実装方法
- ローディングスケルトンとは何か
- ルートグループとは何か。また、いつ使用するか
- アプリケーションのどこに Suspense の境界を配置するか

## ストリーミングとは？

ストリーミングはデータ転送技術のひとつで、ルートをより小さな「チャンク」に分割し、準備が整ったらサーバーからクライアントへ順次配信することができます。

![順次データ取得と並列データ取得の時間を示す図](/_images/server-rendering-with-streaming.avif)

ストリーミングすることで、遅いデータリクエストによってページ全体がブロックされるのを防ぐことができます。これにより、UI がユーザーに表示される前にすべてのデータが読み込まれるのを待つことなく、ページの一部を表示して操作できるようになります。

![順次データ取得と並列データ取得の時間を示す図](/_images/server-rendering-with-streaming-chart.avif)

各々のコンポーネントはチャンクとみなすことができるため、ストリーミングは React のコンポーネントモデルとうまく連携します。

Next.js でストリーミングを実装する方法は 2 つあります。

1. ページレベルで、`loading.tsx` ファイルを使用
2. 特定のコンポーネントに `<Suspense>` を使用

これがどのように機能するかを見てみましょう。

## `loading.tsx` を使用してページ全体をストリーミング

`/app/dashboard` フォルダに、`loading.tsx` という名前の新しいファイルを作成します。

`/app/dashboard/loading.tsx`

```tsx
export default function Loading() {
  return <div>Loading...</div>;
}
```

http://localhost:3000/dashboard を更新すると、以下が表示されるはずです。

![「Loading...」テキストを含むダッシュボードページ](/_images/loading-page.avif)

ここでは、いくつかのことが起きています。

1. `loading.tsx` は Suspense の上に構築された特別な Next.js ファイルで、ページコンテンツの読み込み中に代替として表示するフォールバック UI を作成できます。
2. `<SideNav>` は静的であるため、すぐに表示されます。ユーザーは、動的コンテンツの読み込み中に `<SideNav>` を操作できます。
3. ユーザーは、ページの読み込みが完了するのを待ってから、ページ移動をする必要はありません (これを中断可能なナビゲーションと呼びます)。

おめでとうございます！これでストリーミングが実装されました。しかし、ユーザーエクスペリエンスを向上させるためにできることは他にもあります。`Loading…` テキストの代わりに、読み込み中のスケルトンを表示してみましょう。

## ローディングスケルトンの追加

ローディングスケルトンは、UI の簡略化版です。多くの Web サイトでは、コンテンツがロード中であることをユーザーに示すためのプレースホルダー (またはフォールバック) として使用します。`loading.tsx` に埋め込む UI はすべて静的ファイルの一部として埋め込まれ、最初に送信されます。その後、残りの動的コンテンツがサーバーからクライアントにストリーミングされます。

`loading.tsx` ファイルの中に、`<DashboardSkeleton>` という新しいコンポーネントをインポートします。

`/app/dashboard/loading.tsx`

```tsx diff
+ import DashboardSkeleton from '@/app/ui/skeletons';

  export default function Loading() {
+   return <DashboardSkeleton />;
  }
```

そして、http://localhost:3000/dashboard を更新すると、次のように表示されるはずです。

![スケルトンをロードしているダッシュボードページ](/_images/loading-page-with-skeleton.avif)

### ルートグループのローディングスケルトンのバグを修正

現在、ローディングスケルトンは請求書と顧客ページにも適用されます。

`loading.tsx` はファイルシステムの `/invoices/page.tsx` や ​​ `/customers/page.tsx` より上位階層のレベルであるため、これらのページにも適用されます。

これは [ルートグループ](https://nextjs.org/docs/app/building-your-application/routing/route-groups) で変更できます。`dashboard` フォルダ内に `/(overview)` という名前の新しいフォルダを作成します。次に、`loading.tsx` ファイルと `page.tsx` ファイルをフォルダ内に移動します。

![丸カッコを使用したルートグループの作成方法を示すフォルダ構造](/_images/route-group.avif)

これで、`loading.tsx` ファイルはダッシュボードの概要ページにのみ適用されます。

ルートグループを使用すると、URL パス構造に影響を与えることなく、ファイルを論理的グループにまとめることができます。括弧 `()` を使用して新しいフォルダを作成すると、その名前は URL パスに含まれません。つまり、`/dashboard/(overview)/page.tsx` は `/dashboard` になります。

ここでは `loading.tsx` がダッシュボードの概要ページにのみ適用されるようにルートグループを使用しています。しかし、ルートグループを使用して、アプリケーションをセクション（たとえば、`(marketing)` ルートと `(shop)` ルート）に分割したり、大規模なアプリケーションの場合はチームごとに分割したりすることもできます。

### コンポーネントのストリーミング

ここまでは、ページ全体をストリーミングしています。ただし、その代わりに、React Suspense を使用すると、より細かく特定のコンポーネントをストリーミングできます。

Susppense を使用すると、何らかの条件が満たされるまで (たとえば、データがロードされるまで)、アプリケーションの一部のレンダリングを延期できます。動的コンポーネントを Susppense でラップできます。次に、動的コンポーネントのロード中に表示するフォールバックコンポーネントを渡します。

低速なデータリクエストである `fetchRevenue()` を覚えていると思いますが、これはページ全体の速度を低下させているリクエストです。ページをブロックする代わりに、Suspense を使用してこのコンポーネントのみをストリーミングし、ページの残りの UI をすぐに表示することができます。

そのためには、データ取得をコンポーネントに移す必要があります。コードを更新して、それがどのようになるかを確認してみましょう。

`/dashboard/(overview)/page.tsx` から、`fetchRevenue()` のすべてのインスタンスとそのデータを削除します。

`/app/dashboard/(overview)/page.tsx`

```diff tsx
  import { Card } from '@/app/ui/dashboard/cards';
  import RevenueChart from '@/app/ui/dashboard/revenue-chart';
  import LatestInvoices from '@/app/ui/dashboard/latest-invoices';
  import { lusitana } from '@/app/ui/fonts';
- import { fetchLatestInvoices, fetchCardData } from '@/app/lib/data'; // remove fetchRevenue

  export default async function Page() {
-   const revenue = await fetchRevenue // delete this line
    const latestInvoices = await fetchLatestInvoices();
    const {
      numberOfInvoices,
      numberOfCustomers,
      totalPaidInvoices,
      totalPendingInvoices,
    } = await fetchCardData();

    return (
      // ...
    );
  }
```

Then, import `<Suspense>` from React, and wrap it around `<RevenueChart />`. You can pass it a fallback component called `<RevenueChartSkeleton>`.

`/app/dashboard/(overview)/page.tsx`

```tsx diff
  import { Card } from '@/app/ui/dashboard/cards';
  import RevenueChart from '@/app/ui/dashboard/revenue-chart';
  import LatestInvoices from '@/app/ui/dashboard/latest-invoices';
  import { lusitana } from '@/app/ui/fonts';
  import { fetchLatestInvoices, fetchCardData } from '@/app/lib/data';
+ import { Suspense } from 'react';
+ import { RevenueChartSkeleton } from '@/app/ui/skeletons';

  export default async function Page() {
    const latestInvoices = await fetchLatestInvoices();
    const {
      numberOfInvoices,
      numberOfCustomers,
      totalPaidInvoices,
      totalPendingInvoices,
    } = await fetchCardData();

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
+         <Suspense fallback={<RevenueChartSkeleton />}>
+           <RevenueChart />
+         </Suspense>
          <LatestInvoices latestInvoices={latestInvoices} />
        </div>
      </main>
    );
  }
```

Finally, update the `<RevenueChart>` component to fetch its own data and remove the prop passed to it:

`/app/ui/dashboard/revenue-chart.tsx`

```tsx diff
  import { generateYAxis } from '@/app/lib/utils';
  import { CalendarIcon } from '@heroicons/react/24/outline';
  import { lusitana } from '@/app/ui/fonts';
+ import { fetchRevenue } from '@/app/lib/data';

  // ...

+ export default async function RevenueChart() { // Make component async, remove the props
+   const revenue = await fetchRevenue(); // Fetch data inside the component

    const chartHeight = 350;
    const { yAxisLabels, topLabel } = generateYAxis(revenue);

    if (!revenue || revenue.length === 0) {
      return <p className="mt-4 text-gray-400">No data available.</p>;
    }

    return (
      // ...
    );
  }
```

Now refresh the page, you should see the dashboard information almost immediately, while a fallback skeleton is shown for `<RevenueChart>`:

![Dashboard page with revenue chart skeleton and loaded Card and Latest Invoices components](/_images/loading-revenue-chart.avif)

### Practice: Streaming `<LatestInvoices>`

Now it's your turn! Practice what you've just learned by streaming the `<LatestInvoices>` component.

Move `fetchLatestInvoices()` down from the page to the `<LatestInvoices>` component. Wrap the component in a `<Suspense>` boundary with a fallback called `<LatestInvoicesSkeleton>`.

Once you're ready, expand the toggle to see the solution code:

Dashboard Page:

`/app/dashboard/(overview)/page.tsx`

```tsx diff
  import { Card } from "@/app/ui/dashboard/cards";
  import RevenueChart from "@/app/ui/dashboard/revenue-chart";
  import LatestInvoices from "@/app/ui/dashboard/latest-invoices";
  import { lusitana } from "@/app/ui/fonts";
+ import { fetchCardData } from "@/app/lib/data"; // Remove fetchLatestInvoices
  import { Suspense } from "react";
  import {
    RevenueChartSkeleton,
+   LatestInvoicesSkeleton,
  } from "@/app/ui/skeletons";

  export default async function Page() {
    // Remove `const latestInvoices = await fetchLatestInvoices()`
    const {
      numberOfInvoices,
      numberOfCustomers,
      totalPaidInvoices,
      totalPendingInvoices,
    } = await fetchCardData();

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
          <Suspense fallback={<RevenueChartSkeleton />}>
            <RevenueChart />
          </Suspense>
+         <Suspense fallback={<LatestInvoicesSkeleton />}>
+           <LatestInvoices />
+         </Suspense>
        </div>
      </main>
    );
  }
```

`<LatestInvoices>` component. Remember to remove the props!:

`/app/ui/dashboard/latest-invoices.tsx`

```tsx diff
  import { ArrowPathIcon } from '@heroicons/react/24/outline';
  import clsx from 'clsx';
  import Image from 'next/image';
  import { lusitana } from '@/app/ui/fonts';
+ import { fetchLatestInvoices } from '@/app/lib/data';

+ export default async function LatestInvoices() { // Remove props
+   const latestInvoices = await fetchLatestInvoices();

    return (
      // ...
    );
  }
```

## Grouping components

Great! You're almost there, now you need to wrap the `<Card>` components in Suspense. You can fetch data for each individual card, but this could lead to a popping effect as the cards load in, this can be visually jarring for the user.

So, how would you tackle this problem?

To create more of a staggered effect, you can group the cards using a wrapper component. This means the static `<SideNav/>` will be shown first, followed by the cards, etc.

In your `page.tsx` file:

- Delete your `<Card>` components.
- Delete the `fetchCardData()` function.
- Import a new **wrapper** component called `<CardWrapper />`.
- Import a new skeleton component called `<CardsSkeleton />`.
- Wrap `<CardWrapper />` in Suspense.

`/app/dashboard/page.tsx`

```tsx diff
+ import CardWrapper from "@/app/ui/dashboard/cards";
  // ...
  import {
    RevenueChartSkeleton,
    LatestInvoicesSkeleton,
+   CardsSkeleton,
  } from "@/app/ui/skeletons";

  export default async function Page() {
    return (
      <main>
        <h1 className={`${lusitana.className} mb-4 text-xl md:text-2xl`}>
          Dashboard
        </h1>
        <div className="grid gap-6 sm:grid-cols-2 lg:grid-cols-4">
+         <Suspense fallback={<CardsSkeleton />}>
+           <CardWrapper />
+         </Suspense>
        </div>
        // ...
      </main>
    );
  }
```

Then, move into the file `/app/ui/dashboard/cards.tsx`, import the `fetchCardData()` function, and invoke it inside the `<CardWrapper/>` component. Make sure to uncomment any necessary code in this component.

`/app/ui/dashboard/cards.tsx`

```tsx diff
  // ...
+ import { fetchCardData } from "@/app/lib/data";

  // ...

  export default async function CardWrapper() {
+   const {
+     numberOfInvoices,
+     numberOfCustomers,
+     totalPaidInvoices,
+     totalPendingInvoices,
+   } = await fetchCardData();

    return (
      <>
        <Card title="Collected" value={totalPaidInvoices} type="collected" />
        <Card title="Pending" value={totalPendingInvoices} type="pending" />
        <Card title="Total Invoices" value={numberOfInvoices} type="invoices" />
        <Card
          title="Total Customers"
          value={numberOfCustomers}
          type="customers"
        />
      </>
    );
  }
```

Refresh the page, and you should see all the cards load in at the same time. You can use this pattern when you want multiple components to load in at the same time.

## Deciding where to place your Suspense boundaries

Where you place your Suspense boundaries will depend on a few things:

1. How you want the user to experience the page as it streams.
2. What content you want to prioritize.
3. If the components rely on data fetching.

Take a look at your dashboard page, is there anything you would've done differently?

Don't worry. There isn't a right answer.

- You could stream the **whole page** like we did with `loading.tsx`... but that may lead to a longer loading time if one of the components has a slow data fetch.
- You could stream **every component** individually... but that may lead to UI popping into the screen as it becomes ready.
- You could also create a staggered effect by streaming **page sections**. But you'll need to create wrapper components.

Where you place your suspense boundaries will vary depending on your application. In general, it's good practice to move your data fetches down to the components that need it, and then wrap those components in Suspense. But there is nothing wrong with streaming the sections or the whole page if that's what your application needs.

Don't be afraid to experiment with Suspense and see what works best, it's a powerful API that can help you create more delightful user experiences.

## Looking ahead

Streaming and Server Components give us new ways to handle data fetching and loading states, ultimately with the goal of improving the end user experience.

In the next chapter, you'll learn about Partial Prerendering, a new Next.js rendering model built with streaming in mind.
