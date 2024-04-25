# 部分プリレンダリング（オプション）

> 部分プリレンダリングは、Next.js 14 で導入された実験的な機能です。このページの内容は、機能の安定性が進むにつれて更新される可能性があります。実験的な機能を使用したくない場合は、この章をスキップしてもよいでしょう。この章はコースを完了するために必須ではありません。

この章で取り上げるトピックは以下のとおりです。

- 部分プリレンダリングとは何か
- 部分プリレンダリングの仕組み

## 静的コンテンツと動的コンテンツの結合

現在、ルート内で [動的関数](https://nextjs.org/docs/app/building-your-application/routing/route-handlers#dynamic-functions) を呼び出すと（例：`noStore()`、`cookies()` など） ルート全体が動的になります。

今日、ほとんどの Web アプリはこのように作成されています。静的レンダリングと動的レンダリングのどちらかを **アプリケーション全体** または **特定のルート** に対して選択します。

ただし、たいていのルートが完璧に静的または動的というわけではありません。静的コンテンツと動的コンテンツの両方を含むルートがある場合があります。たとえば、[e コマースサイト](https://partialprerendering.com/) について考えてみましょう。商品ページの大部分をプリレンダリングできる場合もありますが、ユーザーのカートとおすすめ商品を必要に応じて動的に取得したい場合もあります。

ダッシュボードページに戻って、静的なコンポーネントと動的なコンポーネントはどれだと思いますか？？

準備ができたら、ダッシュボードのルートをどのように分割するかを見てみましょう。

![サイドナビゲーションが静的で、ページの子コンポーネントが動的であることを示す図](/_images/dashboard-static-dynamic-components.avif)

- `<SideNav>` コンポーネントはデータに依存せず、ユーザーに合わせてパーソナライズされないので `静的` であることができます
- `<Page>` 内のコンポーネントは頻繁に変更されるデータに依存しており、ユーザーに合わせてカスタマイズされるため `動的` にすることができます。

## 部分プリレンダリングとは？

Next.js 14 contains a preview of **Partial Prerendering** – an experimental feature that allows you to render a route with a static loading shell, while keeping some parts dynamic. In other words, you can isolate the dynamic parts of a route. For example:

![Partially Prerendered Product Page showing static nav and product information, and dynamic cart and recommended products]()

When a user visits a route:

- A static route shell is served, ensuring a fast initial load.
- The shell leaves holes where dynamic content will load in asynchronous.
- The async holes are streamed in parallel, reducing the overall load time of the page.

This is different from how your application behaves today, where entire routes are either entirely static or dynamic.

Partial Prerendering combines ultra-quick static edge delivery with fully dynamic capabilities and we believe it has the potential to [become the default rendering model for web applications](https://vercel.com/blog/partial-prerendering-with-next-js-creating-a-new-default-rendering-model), bringing together the best of static site generation and dynamic delivery.

## How does Partial Prerendering work?

Partial Prerendering leverages React's [Concurrent APIs](https://react.dev/blog/2021/12/17/react-conf-2021-recap#react-18-and-concurrent-features)
and uses [Suspense](https://react.dev/reference/react/Suspense) to defer rendering parts of your application until some condition is met (e.g. data is loaded).

The fallback is embedded into the initial static file along with other static content. At build time (or during revalidation), the static parts of the route are prerendered, and the rest is postponed until the user requests the route.

It's worth noting that wrapping a component in Suspense doesn't make the component itself dynamic (remember you used `unstable_noStore` to achieve this behavior), but rather Suspense is used as a boundary between the static and dynamic parts of your route.

The great thing about Partial Prerendering is that you don't need to change your code to use it. As long as you're using Suspense to wrap the dynamic parts of your route, Next.js will know which parts of your route are static and which are dynamic.

> [!note]
>
> To learn more about how Partial Prerendering can be configured, see the [Partial Prerendering (experimental) documentation](https://nextjs.org/docs/app/api-reference/next-config-js/partial-prerendering) or try the [Partial Prerendering template and demo](https://vercel.com/templates/next.js/partial-prerendering-nextjs). It's important to note that this feature is **experimental** and **not yet ready for production deployment**.

## Summary

To recap, you've done a few things to optimize data fetching in your application, you've:

1. Created a database in the same region as your application code to reduce latency between your server and database.
1. Fetched data on the server with React Server Components. This allows you to keep expensive data fetches and logic on the server, reduces the client-side JavaScript bundle, and prevents your database secrets from being exposed to the client.
1. Used SQL to only fetch the data you needed, reducing the amount of data transferred for each request and the amount of JavaScript needed to transform the data in-memory.
1. Parallelize data fetching with JavaScript - where it made sense to do so.
1. Implemented Streaming to prevent slow data requests from blocking your whole page, and to allow the user to start interacting with the UI without waiting for everything to load.
1. Move data fetching down to the components that need it, thus isolating which parts of your routes should be dynamic in preparation for Partial Prerendering.

In the next chapter, we'll look at two common patterns you might need to implement when fetching data: search and pagination.
