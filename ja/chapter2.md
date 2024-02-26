# CSS スタイル

現在、あなたのホームページにはスタイル設定がされていません。Next.js アプリケーションをスタイル設定するさまざまな方法を見てみましょう。

**この章では...**

ここで取り上げるトピックは次のとおりです

> グローバル CSS ファイルをアプリケーションに追加する方法
> Tailwind モジュールと CSS モジュールを使用した、2つの異なるスタイル設定方法
> clsx ユーティリティパッケージを使用して条件付きでクラス名を追加する方法

## グローバルスタイル

`/app/ui` フォルダの中を見ると、`global.css` というファイルがあることがわかります。このファイルを使用すると、CSS リセットルール、リンクなどの HTML 要素のサイト全体のスタイルなど、CSS ルールをアプリケーション内のすべてのルートに追加できます。

`global.css` はアプリケーションのどのコンポーネントにもインポートできますが、通常はそれを最上位コンポーネントに追加することをお勧めします。Next.js では、これは [ルートレイアウト](https://nextjs.org/docs/app/building-your-application/routing/pages-and-layouts#root-layout-required) となっています (これについては後で詳しく説明します) 。

`/app/layout.tsx` に移動し、`global.css` ファイルをインポートして、グローバルスタイルをアプリケーションに追加します。

```diff ts:/app/layout.tsx
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

With the development server still running, save your changes and preview them in the browser. Your home page should now look like this:

![](Styled page with the logo 'Acme', a description, and login link.)

But wait a second, you didn't add any CSS rules, where did the styles come from?

If you take a look inside global.css, you'll notice some @tailwind directives:

```css /app/ui/global.css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

## Tailwind

[Tailwind](https://tailwindcss.com/) is a CSS framework that speeds up the development process by allowing you to quickly write [utility classes](https://tailwindcss.com/docs/utility-first) directly in your TSX markup.

In Tailwind, you style elements by adding class names. For example, adding the class `"text-blue-500"` will turn the `<h1>` text blue:

```html
<h1 className="text-blue-500">I'm blue!</h1>
```

Although the CSS styles are shared globally, each class is singularly applied to each element. This means if you add or delete an element, you don't have to worry about maintaining separate stylesheets, style collisions, or the size of your CSS bundle growing as your application scales.

When you use `create-next-app` to start a new project, Next.js will ask if you want to use Tailwind. If you select `yes`, Next.js will automatically install the necessary packages and configure Tailwind in your application.

If you look at `/app/page.tsx`, you'll see that we're using Tailwind classes in the example.

```ts /app/page.tsx
import AcmeLogo from '@/app/ui/acme-logo';
import { ArrowRightIcon } from '@heroicons/react/24/outline';
import Link from 'next/link';
 
export default function Page() {
  return (
    // These are Tailwind classes:
    <main className="flex min-h-screen flex-col p-6">
      <div className="flex h-20 shrink-0 items-end rounded-lg bg-blue-500 p-4 md:h-52">
    // ...
  )
}
```

Don't worry if this is your first time using Tailwind. To save time, we've already styled all the components you'll be using.

Let's play with Tailwind! Copy the code below and paste it above the `<p>` element in `/app/page.tsx`:

```ts /app/page.tsx
<div
  className="h-0 w-0 border-b-[30px] border-l-[20px] border-r-[20px] border-b-black border-l-transparent border-r-transparent"
/>
```

**It’s time to take a quiz!**

Test your knowledge and see what you’ve just learned.

What shape do you see when using the code snippet above?

> A : A yellow star
> B : A blue triangle
> C : A black triangle
> D : A red circle

If you prefer writing traditional CSS rules or keeping your styles separate from your JSX - CSS Modules are a great alternative.

## CSS Modules

[CSS Modules](https://nextjs.org/docs/basic-features/built-in-css-support) allow you to scope CSS to a component by automatically creating unique class names, so you don't have to worry about style collisions as well.

We'll continue using Tailwind in this course, but let's take a moment to see how you can achieve the same results from the quiz above using CSS modules.

Inside `/app/ui`, create a new file called `home.module.css` and add the following CSS rules:

```css /app/ui/home.module.css
.shape {
  height: 0;
  width: 0;
  border-bottom: 30px solid black;
  border-left: 20px solid transparent;
  border-right: 20px solid transparent;
}
```

Then, inside your `/app/page.tsx` file import the styles and replace the Tailwind class names from the `<div>` you've added with `styles.shape`:

```ts /app/page.tsx
import styles from '@/app/ui/home.module.css';
<div className={styles.shape} />;
```

Save your changes and preview them in the browser. You should see the same shape as before.

Tailwind and CSS modules are the two most common ways of styling Next.js applications. Whether you use one or the other is a matter of preference - you can even use both in the same application!

**It’s time to take a quiz!**

Test your knowledge and see what you’ve just learned.

What is one benefit of using CSS modules?

> A : Increase the global scope of CSS classes, making them easier to manage across different files.
> B : Provide a way to make CSS classes locally scoped to components by default, reducing the risk of styling conflicts.
> C : Automatically compress and minify CSS files for faster page loading.

## Using the `clsx` library to toggle class names
There may be cases where you may need to conditionally style an element based on state or some other condition.

[clsx](https://www.npmjs.com/package/clsx) is a library that lets you toggle class names easily. We recommend taking a look at [documentation](https://github.com/lukeed/clsx) for more details, but here's the basic usage:

* Suppose that you want to create an `InvoiceStatus` component which accepts `status`. The status can be `'pending'` or `'paid'`.
* If it's `'paid'`, you want the color to be green. If it's `'pending'`, you want the color to be gray.

You can use `clsx` to conditionally apply the classes, like this:

```diff /app/ui/invoices/status.tsx
import clsx from 'clsx';
 
export default function InvoiceStatus({ status }: { status: string }) {
  return (
    <span
      className={clsx(
        'inline-flex items-center rounded-full px-2 py-1 text-sm',
        {
+         'bg-gray-100 text-gray-500': status === 'pending',
+         'bg-green-500 text-white': status === 'paid',
        },
      )}
    >
    // ...
)}
```

**It’s time to take a quiz!**

Test your knowledge and see what you’ve just learned.

Search for "clsx" in your code editor, what components use it to conditionally apply class names?

> A : `status.tsx` and `pagination.tsx`
> B : `table.tsx` and `status.tsx`
> C : `nav-links.tsx` and `table.tsx`

## Other styling solutions

In addition to the approaches we've discussed, you can also style your Next.js application with:

* Sass which allows you to import `.css` and `.scss` files.
* CSS-in-JS libraries such as [styled-jsx](), [styled-components](), and [emotion]().

Take a look at the [CSS documentation]() for more information.