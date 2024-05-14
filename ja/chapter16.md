# メタデータの追加

メタデータは SEO とシェアビリティに必要不可欠です。この章では、Next.js アプリケーションにメタデータを追加する方法について説明します。

ここで取り上げるトピックは次のとおりです

- メタデータとは何か
- メタデータの種類
- メタデータを使用して Open Graph 画像を追加する方法
- メタデータを使用してファビコンを追加する方法

## メタデータとは？

Web 開発において、メタデータは Web ページに関する補足情報を付与します。メタデータは、ページを訪問するユーザーには表示されません。その代わり、ページの HTML（通常は`<head>`要素内）に埋め込まれ、見えないところで機能します。この隠れた情報は、検索エンジンやその他のシステムがウェブページの内容をよりよく理解するために重要です。

## なぜメタデータが重要なのか？

メタデータは Web ページの SEO を強化する上で重要な役割を果たし、検索エンジンやソーシャルメディアプラットフォームにとってよりアクセスしやすく、理解しやすいものにします。適切なメタデータは、検索エンジンが Web ページを効果的にインデックスし、検索結果でのランキングを向上させるのに役立ちます。さらに、Open Graph のようなメタデータは、ソーシャルメディア上で共有されたリンクの見栄えを改善し、コンテンツをユーザーにとってより魅力的で有益なものにします。

## メタデータの種類

メタデータにはさまざまな種類があり、それぞれが独自の目的を果たします。一般的なタイプには次のようなものがあります。

**タイトルメタデータ** ブラウザのタブに表示される Web ページのタイトルを担当します。これは検索エンジンがウェブページの内容を理解するのに役立つため、SEO にとって非常に重要です。

```html
<title>Page Title</title>
```

**説明メタデータ** このメタデータは Web ページのコンテンツの簡単な概要を提供し、多くの場合、検索エンジンの検索結果に表示されます。

```html
<meta name="description" content="A brief description of the page content." />
```

**キーワードメタデータ** このメタデータには Web ページのコンテンツに関連するキーワードが含まれ、検索エンジンがページをインデックスするのに役立ちます。

```html
<meta name="keywords" content="keyword1, keyword2, keyword3" />
```

**オープングラフメタデータ** このメタデータは、ソーシャルメディアプラットフォームで Web ページが共有される際の表現方法を強化し、タイトル、説明文、プレビュー画像などの情報を提供します。

```html
<meta property="og:title" content="Title Here" />
<meta property="og:description" content="Description Here" />
<meta property="og:image" content="image_url_here" />
```

**ファビコンメタデータ** このメタデータは、ブラウザのアドレスバーやタブに表示されるファビコン（小さなアイコン）を Web ページにリンクします。

```html
<link rel="icon" href="path/to/favicon.ico" />
```

## メタデータの追加

Next.js には、アプリケーションのメタデータを定義するためのメタデータ API が用意されています。アプリケーションにメタデータを追加するには、2 つの方法があります。

- **設定ベース** [静的 `metadata` オブジェクト](https://nextjs.org/docs/app/api-reference/functions/generate-metadata#metadata-object) または、動的な [`generateMetadata` 関数](https://nextjs.org/docs/app/api-reference/functions/generate-metadata#generatemetadata-function) を layout.js または page.js ファイル内に記述します。
- **ファイルベース** Next.js には、メタデータのために使用される特別なファイルがあります。
  - `favicon.ico`、`apple-icon.jpg`、`icon.jpg` はファビコンとアイコンに使用されます
  - `opengraph-image.jpg`、`twitter-image.jpg` はソーシャルメディア画像に使用されます
  - `robots.txt` は検索エンジンのクロールの仕方を指示します
  - `sitemap.xml` は Web サイトの構造に関する情報を提供します

これらのファイルを静的なメタデータとして柔軟に利用することもできますし、プロジェクト内でプログラム的に生成することもできます。

どちらのオプションでも、Next.js はページに関連する `<head>` 要素を自動的に生成します。

## ファビコンとオープングラフ画像

`/public` フォルダーに、`favicon.ico` と `opengraph-image.jpg` という 2 つの画像があることに気づくでしょう。

これらの画像を `/app` フォルダのルートに移動させましょう。

こうすると、Next.js がこれらのファイルを自動的に識別して、ファビコンと OG 画像として使用します。このことは、開発ツールでアプリケーションの `<head>` 要素をチェックすることで確認できます。

> [!tip]
>
> [`ImageResponse`](https://nextjs.org/docs/app/api-reference/functions/image-response) コンストラクタを使用して動的に OG 画像を作成することもできます。

### ページのタイトルと説明

`layout.js` または `page.js` ファイルから [`metadata`オブジェクト](https://nextjs.org/docs/app/api-reference/functions/generate-metadata#metadata-fields) を取り込むことで、タイトルや説明文のような追加のページ情報を加えることができます。layout.js` に含まれるメタデータは、それを使用するすべてのページに継承されます。

ルートレイアウトで、以下のフィールドを持つ新しい `metadata` オブジェクトを作成します。

`/app/layout.tsx`

```diff tsx
+ import { Metadata } from 'next';

+ export const metadata: Metadata = {
+   title: 'Acme Dashboard',
+   description: 'The official Next.js Course Dashboard, built with App Router.',
+   metadataBase: new URL('https://next-learn-dashboard.vercel.sh'),
+ };

  export default function RootLayout() {
    // ...
  }
```

Next.js は自動的にタイトルとメタデータをアプリケーションに追加します。

しかし、特定のページにカスタムタイトルを追加したい場合はどうすればよいでしょうか？その場合は、ページ自体に `metadata` オブジェクトを追加します。ネストされたページのメタデータは親のメタデータを上書きします。

たとえば、`/dashboard/invoices` ページでは、ページタイトルを以下のようにして更新することができます。

`/app/dashboard/invoices/page.tsx`

```diff tsx
+ import { Metadata } from 'next';

+ export const metadata: Metadata = {
+   title: 'Invoices | Acme Dashboard',
+ };
```

これは機能はするのですが、すべてのページでアプリケーションのタイトルを繰り返してしまいます。会社名のように、何か変更があれば、すべてのページで更新する必要があります。

代わりに、`metadata` オブジェクトの `title.template` フィールドを使って、ページタイトルのテンプレートを定義することができます。このテンプレートにはページタイトルや その他の必要な情報を含めることができます。

ルートレイアウトで、`metadata` オブジェクトを更新して、テンプレートを含めます。

`/app/layout.tsx`

```diff tsx
+ import { Metadata } from 'next';

+ export const metadata: Metadata = {
+   title: {
+     template: '%s | Acme Dashboard',
+     default: 'Acme Dashboard',
+   },
+   description: 'The official Next.js Learn Dashboard built with App Router.',
+   metadataBase: new URL('https://next-learn-dashboard.vercel.sh'),
+ };
```

テンプレート内の `%s` は特定のページタイトルに置き換えられます。

ここで、`/dashboard/invoices` ページにページタイトルを追加できます。

`/app/dashboard/invoices/page.tsx`

```tsx
export const metadata: Metadata = {
  title: "Invoices",
};
```

`/dashboard/invoices` ページに移動し、`<head>` 要素を確認します。ページのタイトルが `Invoices | Acme Dashboard` になっていると思います。

## 実践：メタデータの追加

メタデータについて学習したので、他のページにタイトルを追加して練習してみましょう。

1. `/login` page.
1. `/dashboard/` page.
1. `/dashboard/customers` page.
1. `/dashboard/invoices/create` page.
1. `/dashboard/invoices/[id]/edit` page.

Next.js のメタデータ API は高機能かつ柔軟で、アプリケーションのメタデータを最大限にコントロールできます。ここでは、基本的なメタデータの追加方法を紹介しましたが `keywords`、`robots`、`canonical` など、複数のフィールドを追加することもできます。お気軽に [ドキュメント](https://nextjs.org/docs/app/api-reference/functions/generate-metadata) を調べて、あなたのアプリケーションに必要なメタデータを追加してください。
