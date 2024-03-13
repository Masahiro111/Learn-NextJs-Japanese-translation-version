# フォントと画像の最適化

前の章では、Next.js アプリケーションのスタイルを設定する方法を学びました。カスタムフォントとヒーロー画像を追加して、ホームページ作成を続けましょう。

この章で取り上げるトピックは以下のとおりです。

- `next/font` でカスタムフォントを追加する方法
- `next/image` で画像を追加する方法
- Next.js でフォントと画像がどのように最適化されるか

## フォントを最適化する理由

フォントは Web サイトのデザインにおいて重要な役割を果たしますが、プロジェクトでカスタムフォントを使用する際、フォントファイルをフェッチしてロードする必要があるとき、パフォーマンスに影響を与える可能性があります。

[Cumulative Layout Shift](https://web.dev/cls/) は、Google が Web サイトのパフォーマンスとユーザーエクスペリエンスを評価するために使用する指標です。フォントの場合、ブラウザが最初にフォールバックフォントまたはシステムフォントでテキストをレンダリングし、読み込まれた後にカスタムフォントに置き換えるときに、レイアウトのシフトが発生します。この入れ替えにより、テキストのサイズ、間隔、レイアウトが変更され、周囲の要素が移動する可能性があります。

![] (ページの初期読み込みと、それに続くカスタム フォントの読み込みに伴うレイアウトのシフトを示すモック UI。)

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

By adding `Inter` to the `<body>` element, the font will be applied throughout your application. Here, you're also adding the Tailwind (antialiased)[] class which smooths out the font. It's not necessary to use this class, but it adds a nice touch.

Navigate to your browser, open dev tools and select the `body` element. You should see `Inter` and `Inter_Fallback` are now applied under styles.

## Practice: Adding a secondary font

You can also add fonts to specific elements of your application.

Now it's your turn! In your `fonts.ts` file, import a secondary font called `Lusitana` and pass it to the `<p>` element in your `/app/page.tsx` file. In addition to specifying a subset like you did before, you'll also need to specify the font `weight`.

Once you're ready, expand the code snippet below to see the solution.

> Hints:
>- If you're unsure what weight options to pass to a font, check the TypeScript errors in your code editor.
> - Visit the [Google Fonts](https://fonts.google.com/) website and search for Lusitana to see what options are available.
> - See the documentation for [adding multiple fonts](https://nextjs.org/docs/app/building-your-application/optimizing/fonts#using-multiple-fonts) and the [full list of options](https://nextjs.org/docs/app/api-reference/components/font#font-function-arguments).

Finally, the `<AcmeLogo />` component also uses Lusitana. It was commented out to prevent errors, you can now uncomment it:

```tsx:/app/page.tsx
// ...
 
export default function Page() {
  return (
    <main className="flex min-h-screen flex-col p-6">
      <div className="flex h-20 shrink-0 items-end rounded-lg bg-blue-500 p-4 md:h-52">
        <AcmeLogo />
        {/* ... */}
      </div>
    </main>
  );
}
```

Great, you've added two custom fonts to your application! Next, let's add a hero image to the home page.

## Why optimize images?

Next.js can serve `static assets`, like images, under the top-level [/public](https://nextjs.org/docs/app/building-your-application/optimizing/static-assets) folder. Files inside `/public` can be referenced in your application.

With regular HTML, you would add an image as follows:

```html
<img
  src="/hero.png"
  alt="Screenshots of the dashboard project showing desktop version"
/>
```

However, this means you have to manually:

- Ensure your image is responsive on different screen sizes.
- Specify image sizes for different devices.
- Prevent layout shift as the images load.
- Lazy load images that are outside the user's viewport.

Image Optimization is a large topic in web development that could be considered a specialization in itself. Instead of manually implementing these optimizations, you can use the `next/image` component to automatically optimize your images.

## The `<Image>` component

The `<Image>` Component is an extension of the HTML `<img>` tag, and comes with automatic image optimization, such as:

- Preventing layout shift automatically when images are loading.
- Resizing images to avoid shipping large images to devices with a smaller viewport.
- Lazy loading images by default (images load as they enter the viewport).
- Serving images in modern formats, like WebP and AVIF, when the browser supports it.

## Adding the desktop hero image

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
+     <Image
+       src="/hero-desktop.png"
+       width={1000}
+       height={760}
+       className="hidden md:block"
+       alt="Screenshots of the dashboard project showing desktop version"
+     />
    </div>
    //...
  );
}
```

## （練習）モバイルヒーロー画像を追加する

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