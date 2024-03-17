# フォントと画像の最適化

前の章では、Next.js アプリケーションのスタイルを設定する方法を学びました。カスタムフォントとヒーロー画像を追加して、ホームページ作成を続けましょう。

この章で取り上げるトピックは以下のとおりです。

- `next/font` でカスタムフォントを追加する方法
- `next/image` で画像を追加する方法
- Next.js でフォントと画像がどのように最適化されるか

## フォントを最適化する理由

フォントは Web サイトのデザインにおいて重要な役割を果たしますが、プロジェクトでカスタムフォントを使用する際、フォントファイルをフェッチしてロードする必要があるとき、パフォーマンスに影響を与える可能性があります。

[Cumulative Layout Shift](https://web.dev/cls/) は、Google が Web サイトのパフォーマンスとユーザーエクスペリエンスを評価するために使用する指標です。フォントの場合、ブラウザが最初にフォールバックフォントまたはシステムフォントでテキストをレンダリングし、読み込まれた後にカスタムフォントに置き換えるときに、レイアウトのシフトが発生します。この入れ替えにより、テキストのサイズ、間隔、レイアウトが変更され、周囲の要素が移動する可能性があります。

![ページの初期読み込みとそれに続くカスタムフォントの読み込みに伴うレイアウトのシフトを示すモック UI](/_images/font-layout-shift.avif)

Next.js は `next/font` モジュールを使用すると、アプリケーション内のフォントを自動的に最適化します。ビルド時にフォントファイルをダウンロードし、他の静的アセットとともにホストします。ユーザーがアプリケーションにアクセスしたときに、フォントの読み込みによってパフォーマンスに影響するようなネットワークリクエストが追加されないということです。

## プライマリフォントの追加

カスタム Google フォントをアプリケーションに追加して、この仕組みを確認してみましょう。

`/app/ui` フォルダーに、`fonts.ts` という名前の新しいファイルを作成します。このファイルを使用して、アプリケーション全体で使用されるフォントを保存します。

`next/font/google` モジュールから `Inter` フォントをインポートします。これがプライマリフォントになります。次に、どの [サブセット](https://fonts.google.com/knowledge/glossary/subsetting) を読み込むかを指定します。今回の場合は `latin` となります。

```ts:/app/ui/fonts.ts
import { Inter } from 'next/font/google';

export const inter = Inter({ subsets: ['latin'] });
```

最後に、フォントを `/app/layout.tsx` の `<body>` 要素に追加します。

```tsx:/app/layout.tsx
import '@/app/ui/global.css';
import { inter } from '@/app/ui/fonts';

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

`Inter` を `<body>` 要素に追加すると、フォントがアプリケーション全体に適用されます。ここでは、フォントを滑らかにする Tailwind [antialiased](https://tailwindcss.com/docs/font-smoothing) クラスも追加しています。このクラスを使用する必要はありませんが、使用すると便利です。

ブラウザに移動し、開発ツールを開いて `body` 要素を選択します。`Inter` と `Inter_Fallback` がスタイルに適用されていることがわかります。

## （練習）セカンダリフォントを追加

アプリケーションの特定の要素にフォントを追加することもできます。

ここであなたの番です！`fonts.ts` ファイルで、`Lusitana` というセカンダリフォントをインポートし、それを `/app/page.tsx` ファイルの `<p>` 要素に渡します。前と同じようにサブセットを指定することに加えて、フォントの太さも指定する必要があります。

> ヒント
>
> - フォントにどのウェイトオプションを渡すべきかわからない場合は、コードエディタで TypeScript エラーを確認してください。
> - [Google Fonts](https://fonts.google.com/) の Web サイトにアクセスし、`Lusitana` を検索して、利用可能なオプションを確認します。
> - [複数のフォントの追加](https://nextjs.org/docs/app/building-your-application/optimizing/fonts#using-multiple-fonts) と [オプションの完全なリスト](https://nextjs.org/docs/app/api-reference/components/font#font-function-arguments) のドキュメントを参照してください。

答え

```ts diff:/app/ui/fonts.ts
+ import { Inter, Lusitana } from 'next/font/google';

  export const inter = Inter({ subsets: ['latin'] });

+ export const lusitana = Lusitana({
+   weight: ['400', '700'],
+   subsets: ['latin'],
+ });
```

```tsx diff:/app/page.tsx
  import AcmeLogo from "@/app/ui/acme-logo";
  import { ArrowRightIcon } from "@heroicons/react/24/outline";
  import Link from "next/link";
+ import { lusitana } from "@/app/ui/fonts";

  export default function Page() {
    return (
      // ...
+     <p
+       className={`${lusitana.className} text-xl text-gray-800 md:text-3xl md:leading-normal`}
+     >
        <strong>Welcome to Acme.</strong> This is the example for the{" "}
        <a href="https://nextjs.org/learn/" className="text-blue-500">
          Next.js Learn Course
        </a>
        , brought to you by Vercel.
      </p>
      // ...
    );
  }
```

最後に `<AcmeLogo />` コンポーネントも Lusitana を使用します。エラーを防ぐためにコメントアウトされていますが、コメントを解除できるようになりました。

```tsx:/app/page.tsx
// ...

  export default function Page() {
    return (
      <main className="flex min-h-screen flex-col p-6">
        <div className="flex h-20 shrink-0 items-end rounded-lg bg-blue-500 p-4 md:h-52">
+         <AcmeLogo />
          {/* ... */}
        </div>
      </main>
    );
  }
```

無事、アプリケーションに 2 つのカスタムフォントが追加されました。次にヒーロー画像をホームページに追加しましょう。

## 画像を最適化する理由

Next.js は、最上位の [/public](https://nextjs.org/docs/app/building-your-application/optimizing/static-assets) フォルダの下で画像などの「静的アセット」を提供できます。`/public` 内のファイルはアプリケーションで参照できます。

通常の HTML では、以下のように画像を追加します。

```html
<img
  src="/hero.png"
  alt="Screenshots of the dashboard project showing desktop version"
/>
```

しかし、これは手動で以下のことを行う必要があることを意味します：

- 異なる画面サイズに対応できるようにする
- デバイスごとに画像サイズを指定する
- 画像の読み込みに伴うレイアウトのずれを防ぐ
- ユーザーのビューポート外にある画像を遅延ロードする

画像の最適化は Web 開発における大きなトピックであり、それ自体が専門分野であると考えられます。これらの最適化を手動で実装する代わりに `next/image` コンポーネントを使用して画像を自動的に最適化できます。

## `<Image>` コンポーネント

`<Image>` コンポーネントは HTML の `<img>` タグの拡張であり、以下のような自動画像最適化機能が付属しています。

- 画像読み込み時に自動的にレイアウトがずれるのを防ぎます
- 小さなビューポートを備えたデバイスに大きな画像が送信されるのを避けるために、画像のサイズを変更します
- デフォルトで画像を遅延読み込みします (画像はビューポートに入るときに読み込まれます)
- ブラウザーがサポートしている場合、WebP や AVIF などの最新の形式で画像を提供します

## デスクトップにヒーロー画像を追加

Let's use the `<Image>` component. If you look inside the `/public` folder, you'll see there are two images: `hero-desktop.png` and `hero-mobile.png`. These two images are completely different, and they'll be shown depending if the user's device is a desktop or mobile.

In your `/app/page.tsx` file, import the component from [next/image](). Then, add the image under the comment:

```tsx diff:/app/page.tsx
  import AcmeLogo from '@/app/ui/acme-logo';
  import { ArrowRightIcon } from '@heroicons/react/24/outline';
  import Link from 'next/link';
  import { lusitana } from '@/app/ui/fonts';
+ import Image from 'next/image';

  export default function Page() {
    return (
      // ...
      <div className="flex items-center justify-center p-6 md:w-3/5 md:px-28 md:py-12">
        {/* Add Hero Images Here */}
+       <Image
+         src="/hero-desktop.png"
+         width={1000}
+         height={760}
+         className="hidden md:block"
+         alt="Screenshots of the dashboard project showing desktop version"
+       />
      </div>
      //...
    );
  }
```

ここでは `width` を 1000 ピクセル、`height` を 760 ピクセルに設定しています。レイアウトのずれを避けるために、画像の `width` と `height` を設定することをお勧めします。これらのアスペクト比はソース画像と **同じ** である必要があります。

また、モバイル画面の DOM から画像を削除するクラス `hidden` と、デスクトップ画面に画像を表示するクラス `md:block` にも注意してください。

これで、ホームページは以下のようになります。

![カスタムフォントとヒーロー画像を使用したスタイル設定されたホームページ](/_images/home-page-with-hero.avif)

## （練習）モバイル画面にヒーロー画像を追加

次はあなたの番です！追加した画像の下に `hero-mobile.png` の別の `<Image>` コンポーネントを追加してみましょう。

- この画像は `width` が `560` ピクセル、`height`は `620` ピクセルのものです。
- モバイル画面では表示され、デスクトップでは非表示になるはずです。開発ツールを使用して、デスクトップとモバイルの画像が正しく入れ替わっているかどうか確認してみましょう。

コードが書けたら、以下のコードスニペットを展開して解決策を確認してみてください。

```tsx diff:/app/page.tsx
  import AcmeLogo from '@/app/ui/acme-logo';
  import { ArrowRightIcon } from '@heroicons/react/24/outline';
  import Link from 'next/link';
  import { lusitana } from '@/app/ui/fonts';
  import Image from 'next/image';

  export default function Page() {
    return (
      // ...
      <div className="flex items-center justify-center p-6 md:w-3/5 md:px-28 md:py-12">
        {/* Add Hero Images Here */}
        <Image
          src="/hero-desktop.png"
          width={1000}
          height={760}
          className="hidden md:block"
          alt="Screenshots of the dashboard project showing desktop version"
        />
+       <Image
+         src="/hero-mobile.png"
+         width={560}
+         height={620}
+         className="block md:hidden"
+         alt="Screenshot of the dashboard project showing mobile version"
+       />
      </div>
      //...
    );
  }
```

素晴らしい！これで、ホームページにカスタムフォントとヒーロー画像が追加されました。

## おすすめ記事

これらのトピックについては、リモート画像の最適化やローカルフォントファイルの使用など、学ぶべきことがたくさんあります。フォントと画像についてさらに詳しく知りたい場合は、以下を参照してください。

- [画像最適化ドキュメント](https://nextjs.org/docs/app/building-your-application/optimizing/images)
- [フォント最適化ドキュメント](https://nextjs.org/docs/app/building-your-application/optimizing/fonts)
- [画像による Web パフォーマンスの向上 (MDN)](https://developer.mozilla.org/en-US/docs/Learn/Performance/Multimedia)
- [Web フォント (MDN)](https://developer.mozilla.org/en-US/docs/Learn/CSS/Styling_text/Web_fonts)
