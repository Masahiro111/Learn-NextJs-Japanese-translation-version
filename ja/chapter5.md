# ページ間の移動

前の章では、ダッシュボードのレイアウトとページを作成しました。次は、ユーザーがダッシュボードのルート間を移動できるようにするためのリンクをいくつか追加してみましょう。

この章で取り上げるトピックは以下のとおりです。

- `next/link` コンポーネントの使用方法
- `usePathname()` フックを使用してアクティブなリンクを表示する方法
- Next.js でのナビゲーションの仕組み

## ナビゲーションを最適化する理由

ページ間をリンクするには、伝統的に `<a>` HTML 要素を使用します。現時点では、サイドバーのリンクには `<a>` 要素が使用されていますが、ブラウザーでホーム、請求書、顧客のページ間を移動するとどうなるかに注目してください。

ご覧いただけましたか？

各ページのナビゲーションでは、ページ全体が更新されます。

## `<Link>` コンポーネント

Next.js では、`<Link />` コンポーネントを使用してアプリケーション内のページ間をリンクできます。`<Link>` を使用すると、JavaScript を使用してクライアント側のナビゲーションを行うことができます。

`<Link />` コンポーネントを使用すると、`/app/ui/dashboard/nav-links.tsx` を開き、[`next/link`](https://nextjs.org/docs/app/api-reference/components/link) から `Link` コンポーネントをインポートして `<a>` タグを `<Link>` に置き換えることができます。

`/app/ui/dashboard/nav-links.tsx`

```tsx diff
  import {
    UserGroupIcon,
    HomeIcon,
    DocumentDuplicateIcon,
  } from "@heroicons/react/24/outline";
+ import Link from "next/link";

  // ...

  export default function NavLinks() {
    return (
      <>
        {links.map((link) => {
          const LinkIcon = link.icon;
          return (
+           <Link
              key={link.name}
              href={link.href}
              className="flex h-[48px] grow items-center justify-center gap-2 rounded-md bg-gray-50 p-3 text-sm font-medium hover:bg-sky-100 hover:text-blue-600 md:flex-none md:justify-start md:p-2 md:px-3"
            >
              <LinkIcon className="w-6" />
              <p className="hidden md:block">{link.name}</p>
+           </Link>
          );
        })}
      </>
    );
  }
```

ご覧のとおり、`Link` コンポーネントは `<a>` タグの使用に似ていますが、`<a href="…">` の代わりに `<Link href="…">` を使用します。

変更を保存し、ローカルホストで動作するか確認してみてください。これで、完全なリフレッシュをすることなくページ間を移動できるようになります。アプリケーションの一部はサーバー上でレンダリングされますが、ページ全体が更新されることはないため、Web アプリのように感じられます。なぜでしょう？

### 自動コード分割とプリフェッチ

ナビゲーション体験を向上させるために、Next.js はアプリケーションをルートセグメントごとに自動的にコード分割します。これは、初期ロード時にブラウザがすべてのアプリケーションコードをロードする従来の React [SPA](https://developer.mozilla.org/en-US/docs/Glossary/SPA) とは異なります。

コードをルートごとに分割すると、ページが分離されます。特定のページでエラーが発生しても、アプリケーションの残りの部分は引き続き動作します。

さらに、本番運用環境では、[`<Link>`](https://nextjs.org/docs/api-reference/next/link) コンポーネントがブラウザーのビューポートに表示されるたびに、Next.js がコードを自動的に **プリフェッチ** します。ユーザーがリンクをクリックするころには、リンク先ページのコードはすでにバックグラウンドで読み込まれており、これによりページがほぼ瞬時に遷移します。

[ナビゲーションの仕組み](https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating#how-routing-and-navigation-works) の詳細をご覧ください。

## アクティブリンクの表示パターン

アクティブリンクを表示することは、現在ユーザーがどのページにいるかを示すための一般的な UI パターンとなっています。これを行うには、URL からユーザーの現在のパスを取得する必要があります。Next.js は、パスを確認してこのパターンを実装するために使用できる [`usePathname()`](https://nextjs.org/docs/app/api-reference/functions/use-pathname) というフックを提供しています。

[`usePathname()`](https://nextjs.org/docs/app/api-reference/functions/use-pathname) はフックであるため、`nav-links.tsx` を Client Component にする必要があります。React の `"use client"` ディレクティブをファイルの先頭に追加し、`next/navigation` から `usePathname()` をインポートします。

`/app/ui/dashboard/nav-links.tsx`

```tsx diff
+ 'use client';

  import {
    UserGroupIcon,
    HomeIcon,
    InboxIcon,
  } from '@heroicons/react/24/outline';
  import Link from 'next/link';
+ import { usePathname } from 'next/navigation';

  // ...
```

次に、`<NavLinks />` コンポーネント内の `pathname` という変数にパスを割り当てます。

`/app/ui/dashboard/nav-links.tsx`

```tsx diff
  export default function NavLinks() {
+   const pathname = usePathname();
    // ...
  }
```

[CSS スタイリング](https://nextjs.org/learn/dashboard-app/css-styling) の章で紹介されている `clsx` ライブラリを使用して、リンクがアクティブなときに条件付きでクラス名を適用できます。 `link.href` が `pathname` と一致する場合、リンクは青いテキストと水色の背景で表示されます。

`nav-links.tsx` の最終的なコードは以下のとおりです。

`/app/ui/dashboard/nav-links.tsx`

```tsx diff
  'use client';

  import {
    UserGroupIcon,
    HomeIcon,
    DocumentDuplicateIcon,
  } from "@heroicons/react/24/outline";
  import Link from "next/link";
  import { usePathname } from "next/navigation";
+ import clsx from "clsx";

  // ...

  export default function NavLinks() {
    const pathname = usePathname();

    return (
      <>
        {links.map((link) => {
          const LinkIcon = link.icon;
          return (
            <Link
              key={link.name}
              href={link.href}
+             className={clsx(
+               "flex h-[48px] grow items-center justify-center gap-2 rounded-md bg-gray-50 p-3 text-sm font-medium hover:bg-sky-100 hover:text-blue-600 md:flex-none md:justify-start md:p-2 md:px-3",
+               {
+                 "bg-sky-100 text-blue-600": pathname === link.href,
+               }
+             )}
            >
              <LinkIcon className="w-6" />
              <p className="hidden md:block">{link.name}</p>
            </Link>
          );
        })}
      </>
    );
  }
```

保存して確認しましょう。アクティブなリンクが青色で強調表示されるはずです。
