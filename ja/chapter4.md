# layouts と pages の作成

これまでのところ、アプリケーションにはホームページのみがあります。**layouts** と **pages** を使用してさらに多くのルートを作成する方法を学びましょう。

ここで取り上げるトピックは次のとおりです。

- ファイルシステムルーティングを使用してダッシュボードルートを作成します。
- 新しいルートセグメントを作成するときのフォルダとファイルの役割を理解します。
- 複数のダッシュボードページ間で共有できるネストされたレイアウトを作成します。
- コロケーション、部分レンダリング、ルートレイアウトとは何かを理解します。

## ネストされたルーティング

Next.js はファイルシステムルーティングを採用しており、**フォルダ** を使用してネストされたルートを作成します。各フォルダは、`URL セグメント` に対応する `ルートセグメント` を表します。

![フォルダが URL セグメントにどのように対応するかを示す図](/_images/folders-to-url-segments.avif)

`layout.tsx` ファイルと `page.tsx` ファイルを使用して、ルートごとに個別の UI を作成できます。

`page.tsx` は React コンポーネントをエクスポートする特別な Next.js ファイルで、ルートにアクセスできるようにするために必要です。アプリケーションには、すでにページファイル `/app/page.tsx` があります。これは、ルート `/` に関連付けられたホームページです。

ネストされたルートを作成するには、フォルダを相互にネストし、その中に `page.tsx` ファイルを追加します。例えば

![`dashboard` というフォルダを追加すると、新しいルート `/dashboard` がどのように作成されるかを示す図](/_images/dashboard-route.avif)

`/app/dashboard/page.tsx` は `/dashboard` パスに関連付けられます。ページを作成して、どのように機能するか見てみましょう。

## ダッシュボードページの作成

`/app` 内に `dashboard` という新しいフォルダを作成します。そして以下の内容を含む新しい `page.tsx` ファイルを `dashboard` フォルダ内に作成します。

`/app/dashboard/page.tsx`

```tsx
export default function Page() {
  return <p>Dashboard Page</p>;
}
```

次に、開発サーバーが実行されていることを確認し、http://localhost:3000/dashboard にアクセスします。"Dashboard Page" というテキストが表示されるはずです。

Next.js でさまざまなページを作成する方法は以下のとおりです。フォルダを使用して新しいルートセグメントを作成し、その中に `page` ファイルを追加します。

Next.js では、`page` ファイルに特別な名前をつけることで、UI コンポーネント、テストファイル、その他の関連コードをルートと [コロケーション](https://nextjs.org/docs/app/building-your-application/routing#colocation) することができます。`page` ファイル内のコンテンツのみが公開されます。例えば、`/ui` フォルダと `/lib` フォルダはルートと一緒に `/app` フォルダ内に配置されます。

## （練習）ダッシュボードページの作成

さらにルートを作成する練習をしてみましょう。ダッシュボードに、さらに 2 つのページを作成します。

1, 顧客ページ: http://localhost:3000/dashboard/customers からアクセスできます。現時点では、`<p>Customers Page</p>` 要素が返します。
2, 請求書ページ: http://localhost:3000/dashboard/invoices から請求書ページにアクセスできます。現時点では、`<p>Invoices Page</p>` 要素を返します。

練習の内容を解決するためのコードを書けましたか？以下を参考にして答え合わせをしてみましょう。

次のようなフォルダ構造になっているはずです。

`login` というフォルダを追加すると、新しいルート `/login` がどのように作成されるかを示す図

**顧客ページ:**

`/app/dashboard/customers/page.tsx`

```tsx
export default function Page() {
  return <p>Customers Page</p>;
}
```

**請求書ページ:**

`/app/dashboard/invoices/page.tsx`

```tsx
export default function Page() {
  return <p>Invoices Page</p>;
}
```

## ダッシュボードレイアウトの作成

ダッシュボードには、複数のページにわたって共有されるある種のナビゲーションがあります。Next.js では、特別な `layout.tsx` ファイルを使用して、複数のページ間で共有される UI を作成できます。ダッシュボードページのレイアウトを作成しましょう。

`/dashboard` フォルダ内に `layout.tsx` という名前の新しいファイルを追加し、以下のコードを貼り付けます。

`/app/dashboard/layout.tsx`

```tsx
import SideNav from "@/app/ui/dashboard/sidenav";

export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <div className="flex h-screen flex-col md:flex-row md:overflow-hidden">
      <div className="w-full flex-none md:w-64">
        <SideNav />
      </div>
      <div className="flex-grow p-6 md:overflow-y-auto md:p-12">{children}</div>
    </div>
  );
}
```

このコードではいくつかの処理が行われているので、詳しく見てみましょう。

まず、`<SideNav />` コンポーネントをレイアウトにインポートします。このファイルにインポートするコンポーネントはすべてレイアウトの一部になります。

`<Layout />` コンポーネントは `children` プロパティを受け取ります。この子コンポーネントは、1 つのページか、または別のレイアウトのいずれかになります。今回の場合、`/dashboard` 内のページは以下のように `<Layout />` 内に自動的にネストされます。

![ダッシュボードページを子としてネストしたダッシュボード レイアウトのフォルダ構造](/_images/shared-layout.avif)

変更を保存し、ローカルホストを確認して、すべてが正しく動作していることを確認します。以下の内容が表示されるはずです。

![サイドナビゲーションとメインコンテンツエリアを備えたダッシュボードページ](/_images/shared-layout-page.avif)

Next.js でレイアウトを使用する利点の 1 つは、ナビゲーション時にページコンポーネントのみが更新され、レイアウトは再レンダリングされないことです。これは [部分レンダリング](https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating#3-partial-rendering) と呼ばれます。

![ダッシュボードページをネストしたダッシュボードレイアウトを示すフォルダ構造ですが、ナビゲーションではページ UI のみが切り替わります](/_images/partial-rendering-dashboard.avif)

## ルートレイアウト

第 3 章では、`Inter` フォントを別のレイアウト `/app/layout.tsx` にインポートしました。

`/app/layout.tsx`

```tsx
import "@/app/ui/global.css";
import { inter } from "@/app/ui/fonts";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body className={`${inter.className} antialiased`}>{children}</body>
    </html>
  );
}
```

これは [ルートレイアウト](https://nextjs.org/docs/app/building-your-application/routing/pages-and-layouts#root-layout-required) と呼ばれ、必須です。ルートレイアウトに追加した UI は、アプリケーション内のすべてのページで共有されます。ルートレイアウトを使用して、`<html>` タグと `<body>` タグを変更し、メタデータを追加できます（メタデータについては [後の章](https://nextjs.org/learn/dashboard-app/adding-metadata) で詳しく学びます）。

作成した新しいレイアウト (`/app/dashboard/layout.tsx`) はダッシュボードページに固有であるため、上記のルートレイアウトに UI を追加する必要はありません。
