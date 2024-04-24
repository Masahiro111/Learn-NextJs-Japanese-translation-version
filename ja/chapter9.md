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

```diff tsx
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
+ import { fetchLatestInvoices, fetchCardData } from '@/app/lib/data'; // remove fetchRevenue

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

次に、React から `<Suspense>` をインポートし、それを `<RevenueChart />` で囲みます。 `<RevenueChartSkeleton>` というフォールバックコンポーネントに渡すことができます。

`/app/dashboard/(overview)/page.tsx`

```diff tsx
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

最後に、`<RevenueChart>` コンポーネントを更新して、それ自身のデータを取得し、渡された prop を削除します。

`/app/ui/dashboard/revenue-chart.tsx`

```diff tsx
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

ここでページを更新すると、`<RevenueChart>` のフォールバックスケルトンが表示され、その後すぐにダッシュボード情報が表示されるはずです。

![収益グラフのスケルトンとロードされたカードおよび最新の請求書のコンポーネントを含むダッシュボードページ](/_images/loading-revenue-chart.avif)

### 練習：`<LatestInvoices>` のストリーミング

それでは実際にストリーミングを体験してみましょう。`<LatestInvoices>` コンポーネントをストリーミングして、学んだ内容を実践してください。

`fetchLatestInvoices()` をページから `<LatestInvoices>` コンポーネントに移動します。 `<LatestInvoicesSkeleton>` というフォールバックを使用して、コンポーネントを `<Suspense>` 境界でラップします。

Dashboard Page:

`/app/dashboard/(overview)/page.tsx`

```diff tsx
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

`<LatestInvoices>` コンポーネントについては、props を忘れずに削除しておいてください。:

`/app/ui/dashboard/latest-invoices.tsx`

```diff tsx
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

## コンポーネントのグループ化

素晴らしい！もうすぐ完成です。次に、`<Card>` コンポーネントを Suspense でラップする必要があります。個々のカードデータを取得できますが、カードが読み込まれるときにポップエフェクトが発生する可能性があり、ユーザーにとって視覚的に不快になる可能性があります。

では、この問題にどのように対処すればよいでしょうか？

より多くの時差エフェクトを表示するには、ラッパーコンポーネントを使用してカードをグループ化します。これは、静的な `<SideNav/>` が最初に表示され、その後にカードなどが表示されることを意味します。

`page.tsx` ファイルを開いてみましょう。

- `<Card>` コンポーネントを削除します
- `fetchCardData()` 関数を削除します
- `<CardWrapper />` という名前の新しい **wrapper** コンポーネントをインポートします
- `<CardsSkeleton />` という名前の新しいスケルトンコンポーネントをインポートします
- `<CardWrapper />` を Suspense でラップします

`/app/dashboard/page.tsx`

```diff tsx
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

次に、`/app/ui/dashboard/cards.tsx` ファイルに移動し、`fetchCardData()` 関数をインポートし、`<CardWrapper/>` コンポーネント内で呼び出します。このコンポーネント内の必要なコードのコメントを必ず解除してください。

`/app/ui/dashboard/cards.tsx`

```diff tsx
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

ページを更新すると、すべてのカードが同時に読み込まれていることがわかります。複数のコンポーネントを同時にロードする場合は、このパターンを使用できます。

## Suspense の境界はどこに置くか

Suspense の境界をどこに置くかは、いくつかの要素によって決まります。

1. ストリーミング中にユーザーにページをどのように体験してもらいたいか
2. どのコンテンツを優先したのか
3. コンポーネントがデータの取得に依存している場合

ダッシュボードページを見てください。別の方法で実行したかったことはありますか?

心配しないでください。正解はありません。

- `loading.tsx` で行ったように **ページ全体** をストリーミングすることもできます。しかし、コンポーネントの 1 つでもデータの取得が遅い場合、読み込み時間が長くなる可能性があります。
- **各々のコンポーネント** を個別にストリーミングすることもできます。ただし、その場合、UI の準備が整うと画面にポップアップされる可能性があります
- **ページセクション** をストリーミングすることで、時差エフェクトを作成することもできます。ただし、ラッパーコンポーネントを作成する必要があります。

Suspense の境界をどこに置くかは、アプリケーションによって異なります。一般に、データの取得を必要なコンポーネントに移して、そのコンポーネントを Suspense でラップするのが良い方法です。ただし、アプリケーションが必要とするのであれば、セクションまたはページ全体をストリーミングする方法でも問題はありません。

Suspense は、より快適なユーザーエクスペリエンスを作成するのに役立つ強力な API なので、遠慮せずに試して何が最適かを確認してください。

## 将来を見据えて

ストリーミングコンポーネントとサーバーコンポーネントは、データのフェッチとロードの状態を処理する新しい方法を提供し、最終的にはエンドユーザーエクスペリエンスを向上させることを目的としています。

次の章では、ストリーミングを念頭に置いて構築された新しい Next.js レンダリングモデルである部分プリレンダリングについて学びます。
