# CSS スタイル

現在、あなたのホームページにはスタイル設定がされていません。Next.js アプリケーションをスタイル設定するさまざまな方法を見てみましょう。

この章で取り上げるトピックは以下のとおりです。

> - グローバル CSS ファイルをアプリケーションに追加する方法
> - Tailwind モジュールと CSS モジュールを使用した、2つの異なるスタイル設定方法
> - clsx ユーティリティパッケージを使用して条件付きでクラス名を追加する方法

## グローバルスタイル

`/app/ui` フォルダの中を見ると、`global.css` というファイルがあることがわかります。このファイルを使用すると、CSS リセットルール、リンクなどの HTML 要素のサイト全体のスタイルなど、CSS ルールをアプリケーション内のすべてのルートに追加できます。

`global.css` はアプリケーションのどのコンポーネントにもインポートできますが、通常はそれを最上位コンポーネントに追加することをお勧めします。Next.js では、これは [ルートレイアウト](https://nextjs.org/docs/app/building-your-application/routing/pages-and-layouts#root-layout-required) となっています (これについては後で詳しく説明します) 。

`/app/layout.tsx` に移動し、`global.css` ファイルをインポートして、グローバルスタイルをアプリケーションに追加します。

```ts diff:/app/layout.tsx
+ import '@/app/ui/global.css';
 
  export default function RootLayout({
    children,
  }: {
    children: React.ReactNode;
  }) {
    return (
      <html lang="en">
        <body>{children}</body>
      </html>
    );
  }
```

開発サーバーを起動したまま、変更を保存し、ブラウザでプレビューしてください。あなたのホームページは以下のようになるはずです。

![ロゴ「Acme」説明、ログイン リンクを含むスタイル付きページ](/_images/home-page-with-tailwind.avif)

しかし、ちょっと待ってください。CSS ルールを追加していません。スタイルはどこから来たのでしょうか？

global.css の内部を見ると、いくつかの @tailwind ディレクティブに気づくでしょう。

```css:/app/ui/global.css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

## Tailwind

[Tailwind](https://tailwindcss.com/) は、TSX マークアップ内に [ユーティリティクラス](https://tailwindcss.com/docs/utility-first) を直接すばやく記述できるようにすることで、開発プロセスを高速化する CSS フレームワークです。

Tailwind では、クラス名を追加することで要素をスタイル設定します。たとえば `"text-blue-500"` というクラスを追加すると `<h1>` のテキストが青に変わります。

```html
<h1 className="text-blue-500">I'm blue!</h1>
```

CSSスタイルはグローバルに共有されますが、各クラスは各要素に個別に適用されます。つまり、要素を追加したり削除したりしても、別々のスタイルシートを維持したり、スタイルが衝突したり、アプリケーションの拡張に伴って CSS バンドルのサイズが大きくなったりする心配はありません。

`create-next-app` を使用して新しいプロジェクトを開始すると、Next.js は Tailwind を使用するかどうかを尋ねます。`yes` を選択すると、Next.js は必要なパッケージを自動的にインストールし、アプリケーションに Tailwind を設定します。

`/app/page.tsx` を見ると、サンプルで Tailwind のクラスを使っていることがわかります。

```ts:/app/page.tsx
import AcmeLogo from '@/app/ui/acme-logo';
import { ArrowRightIcon } from '@heroicons/react/24/outline';
import Link from 'next/link';
 
export default function Page() {
  return (
    // これらは Tailwind のクラスです
    <main className="flex min-h-screen flex-col p-6">
      <div className="flex h-20 shrink-0 items-end rounded-lg bg-blue-500 p-4 md:h-52">
    // ...
  )
}
```

Tailwind を初めて使用する場合でも心配する必要はありません。時間を節約するために、使用するすべてのコンポーネントのスタイルがすでに設定されています。

Tailwind で遊んでみましょう！以下のコードをコピーして `/app/page.tsx` の `<p>` 要素の上に貼り付けてください。

```ts:app/page.tsx
<div
  className="h-0 w-0 border-b-[30px] border-l-[20px] border-r-[20px] border-b-black border-l-transparent border-r-transparent"
/>
```

**クイズの時間です！**

あなたの知識をテストして、今学んだことを確認しましょう。

上記のコードスニペットを使用すると、どのような形状が表示されますか？

1. 黄色い星
2. 青い三角形
3. 黒い三角形
4. 赤い丸

<details>
<summary>答え：</summary>
3
</details>

従来の CSS ルールを記述したい場合、またはスタイルを JSX から分離しておきたい場合は、CSS モジュールが優れた代替手段となります。

## CSS モジュール

[CSS モジュール](https://nextjs.org/docs/basic-features/built-in-css-support) を使用すると、一意のクラス名を自動的に作成することで CSS のスコープをコンポーネントに設定できます。なので、スタイルの衝突については心配する必要はありません。

このコースでは Tailwind を使用しますが、CSS モジュールを使用して上記のクイズと同じ結果を得る方法を少し見てみましょう。

`/app/ui` 内に、`home.module.css` という名前の新しいファイルを作成し、以下の CSS ルールを追加します。

```css:/app/ui/home.module.css
.shape {
  height: 0;
  width: 0;
  border-bottom: 30px solid black;
  border-left: 20px solid transparent;
  border-right: 20px solid transparent;
}
```

次に、`/app/page.tsx` ファイル内でスタイルをインポートし、追加した `<div>` の Tailwind クラス名を `styles.shape` に置き換えます。

```ts:/app/page.tsx
import styles from '@/app/ui/home.module.css';
<div className={styles.shape} />;
```

変更を保存し、ブラウザでプレビューしてください。以前と同じ形状が表示されるはずです。

Tailwind と CSS モジュールは、Next.js アプリケーションをスタイリングする最も一般的な2つの方法です。どちらを使うかは好みの問題ですが、同じアプリケーションで両方を使うこともできます！

**クイズの時間です！**

あなたの知識をテストして、今学んだことを確認しましょう。

CSS モジュールを使う利点の1つは何でしょう？

1. CSS クラスのグローバルスコープを広げ、異なるファイル間での管理を容易にする
2. CSS クラスをデフォルトでコンポーネントに対してローカルにスコープさせる方法を提供し、スタイルの衝突のリスクを減らす
3. CSS ファイルを自動的に圧縮・最小化して、ページの読み込みを速くする

<details>
<summary>答え：</summary>
2
</details>

## `clsx` ライブラリを使用してクラス名を切り替える
状態やその他の条件に基づいて要素を条件付きでスタイル設定する必要がある場合があります。

[clsx](https://www.npmjs.com/package/clsx) は、クラス名を簡単に切り替えることができるライブラリです。詳細については、[ドキュメント](https://github.com/lukeed/clsx) を参照することをお勧めしますが、基本的な使用法は以下のとおりです。

* `InvoiceStatus`コンポーネントを作成して `status`を受け入れます。ステータスは `'pending'` （保留中） または `'paid'` （支払い済み） になります。
* もし `'paid'` なら、色をグリーンにします。`pending'`の場合はグレーにします。

`clsx` を使用うと、以下のように条件付きでクラスを適用できます。

```ts diff:/app/ui/invoices/status.tsx
  import clsx from 'clsx';
  
  export default function InvoiceStatus({ status }: { status: string }) {
    return (
      <span
        className={clsx(
          'inline-flex items-center rounded-full px-2 py-1 text-sm',
          {
+           'bg-gray-100 text-gray-500': status === 'pending',
+           'bg-green-500 text-white': status === 'paid',
          },
        )}
      >
      // ...
  )}
```

**クイズの時間です！**

あなたの知識をテストして、今学んだことを確認しましょう。

あなたのコードエディタで「clsx」を検索してください。クラス名を条件付きで適用するために、どのコンポーネントが "clsx "を使っているでしょうか？

1. `status.tsx` と `pagination.tsx`
2. `table.tsx` と `status.tsx`
3. `nav-links.tsx` と `table.tsx`

<details>
<summary>答え：</summary>
1
</details>

## その他のスタイリング方法

これまで説明してきたアプローチに加えて、以下のように Next.js アプリケーションのスタイルを設定することもできます。

* `.css` および `.scss` ファイルをインポートできる Sass。
* [styled-jsx](https://github.com/vercel/styled-jsx)、[styled-components](https://github.com/vercel/next.js/tree/canary/examples/with-styled-components)、[emotion](https://github.com/vercel/next.js/tree/canary/examples/with-emotion) などの CSS-in-JS ライブラリ。

詳細については、[CSS ドキュメント](https://nextjs.org/docs/app/building-your-application/styling) を参照してください。