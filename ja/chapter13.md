# エラーの処理

前の章では、サーバーアクションを使用してデータを変更する方法を学びました。JavaScript の try/catch 文と Next.js API を使用してエラーを適切に処理する方法を見てみましょう。

この章で取り上げるトピックは以下のとおりです。

- 特別な error.tsx ファイルを使用して、ルートセグメントでエラーを捕捉し、ユーザーにフォールバック UI を表示する方法
- notFound 関数と not-found ファイルを使用して 404 エラー（存在しないリソースの場合）を処理する方法

## サーバーアクションに `try/catch` を追加

まず、JavaScript の `try/catch` 文をサーバーアクションに追加して、エラーを適切に処理できるようにしましょう。

以下のコードをコピーして実装してみましょう。

`/app/lib/actions.ts`

```ts
export async function createInvoice(formData: FormData) {
  const { customerId, amount, status } = CreateInvoice.parse({
    customerId: formData.get("customerId"),
    amount: formData.get("amount"),
    status: formData.get("status"),
  });

  const amountInCents = amount * 100;
  const date = new Date().toISOString().split("T")[0];

  try {
    await sql`
      INSERT INTO invoices (customer_id, amount, status, date)
      VALUES (${customerId}, ${amountInCents}, ${status}, ${date})
    `;
  } catch (error) {
    return {
      message: "Database Error: Failed to Create Invoice.",
    };
  }

  revalidatePath("/dashboard/invoices");
  redirect("/dashboard/invoices");
}
```

`/app/lib/actions.ts`

```ts
export async function updateInvoice(id: string, formData: FormData) {
  const { customerId, amount, status } = UpdateInvoice.parse({
    customerId: formData.get("customerId"),
    amount: formData.get("amount"),
    status: formData.get("status"),
  });

  const amountInCents = amount * 100;

  try {
    await sql`
        UPDATE invoices
        SET customer_id = ${customerId}, amount = ${amountInCents}, status = ${status}
        WHERE id = ${id}
      `;
  } catch (error) {
    return { message: "Database Error: Failed to Update Invoice." };
  }

  revalidatePath("/dashboard/invoices");
  redirect("/dashboard/invoices");
}
```

`/app/lib/actions.ts`

```ts
export async function deleteInvoice(id: string) {
  try {
    await sql`DELETE FROM invoices WHERE id = ${id}`;
    revalidatePath("/dashboard/invoices");
    return { message: "Deleted Invoice." };
  } catch (error) {
    return { message: "Database Error: Failed to Delete Invoice." };
  }
}
```

`try/catch` ブロックの外側で `redirect` が呼び出されていることに注目してください。これは、`redirect` がエラーをスローすることで動作し、そのエラーは `catch` ブロックによってキャッチされるためです。これを回避するには、`try/catch` の後に `redirect` を呼び出します。`redirect` は、`try` が成功した場合にのみ到達可能です。

ここで、サーバーアクションでエラーがスローされた場合に何が起こるかを確認してみましょう。これを行うには、事前にエラーをスローします。たとえば、`deleteInvoice` アクションでは、関数の先頭でエラーをスローします。

`/app/lib/actions.ts`

```diff ts
  export async function deleteInvoice(id: string) {
+   throw new Error('Failed to Delete Invoice');

    // Unreachable code block
    try {
      await sql`DELETE FROM invoices WHERE id = ${id}`;
      revalidatePath('/dashboard/invoices');
      return { message: 'Deleted Invoice' };
    } catch (error) {
      return { message: 'Database Error: Failed to Delete Invoice' };
    }
  }
```

請求書を削除しようとすると、ローカルホストでエラーが表示されるはずです。

このようなエラーが表示されると、潜在的な問題を早期に発見できるため、開発中には役立つでしょう。しかし、突然の障害を避けてアプリケーションの実行を継続できるように、ユーザーにエラーを表示することも必要です。

そこで Next.js [error.tsx]() ファイルの出番です。

## `error.tsx` ですべてのエラーを処理する

`error.tsx` ファイルは、ルートセグメントの UI 境界を定義するために使用できます。これは、予期しないエラーのための **キャッチオール** として機能し、ユーザーにフォールバック UI を表示できるようにします。

`/dashboard/invoices` フォルダ内に、`error.tsx` という新しいファイルを作成し、次のコードを貼り付けます。

`/dashboard/invoices/error.tsx`

```tsx
"use client";

import { useEffect } from "react";

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => {
    // Optionally log the error to an error reporting service
    console.error(error);
  }, [error]);

  return (
    <main className="flex h-full flex-col items-center justify-center">
      <h2 className="text-center">Something went wrong!</h2>
      <button
        className="mt-4 rounded-md bg-blue-500 px-4 py-2 text-sm text-white transition-colors hover:bg-blue-400"
        onClick={
          // Attempt to recover by trying to re-render the invoices route
          () => reset()
        }
      >
        Try again
      </button>
    </main>
  );
}
```

上記のコードについては、いくつか注意すべき箇所があります。

- "use client" - `error.tsx` はクライアントコンポーネントである必要があります
- 2 つのプロップを受け取ります
  - `error`：このオブジェクトは、JavaScript のネイティブの [Error](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error) オブジェクトのインスタンスです
  - `reset`：エラー境界をリセットする機能です。実行されると、この関数はルートセグメントの再レンダリングを試みます

請求書を再度削除しようとすると、次の UI が表示されます。

![受け取るプロップを示す error.tsx ファイル]()

## `notFound` 関数で 404 エラーを処理する

エラーを適切に処理するもう 1 つの方法は、`notFound` 関数を使用することです。`error.tsx` は **すべての** エラーをキャッチするのに便利ですが、存在しないリソースを取得しようとする場合は `notFound` を使用できます。

たとえば、 http://localhost:3000/dashboard/invoices/2e94d1ed-d220-449f-9f11-f0bbceed9645/edit にアクセスしたとします。

これはデータベースに存在しない偽の UUID です。

You'll immediately see `error.tsx` kicks in because this is a child route of `/invoices` where `error.tsx` is defined.

However, if you want to be more specific, you can show a 404 error to tell the user the resource they're trying to access hasn't been found.

You can confirm that the resource hasn't been found by going into your `fetchInvoiceById` function in `data.ts`, and console logging the returned `invoice`:

`/app/lib/data.ts`

```diff ts
  export async function fetchInvoiceById(id: string) {
    noStore();
    try {
      // ...

+     console.log(invoice); // Invoice is an empty array []
      return invoice[0];
    } catch (error) {
      console.error('Database Error:', error);
      throw new Error('Failed to fetch invoice.');
    }
  }
```

Now that you know the invoice doesn't exist in your database, let's use `notFound` to handle it. Navigate to `/dashboard/invoices/[id]/edit/page.tsx`, and import `{ notFound }` from `'next/navigation'`.

Then, you can use a conditional to invoke `notFound` if the invoice doesn't exist:

`/dashboard/invoices/[id]/edit/page.tsx`

```diff tsx
  import { fetchInvoiceById, fetchCustomers } from '@/app/lib/data';
  import { updateInvoice } from '@/app/lib/actions';
+ import { notFound } from 'next/navigation';

  export default async function Page({ params }: { params: { id: string } }) {
    const id = params.id;
    const [invoice, customers] = await Promise.all([
      fetchInvoiceById(id),
      fetchCustomers(),
    ]);

+   if (!invoice) {
+     notFound();
+   }

    // ...
  }
```

Perfect! `<Page>` will now throw an error if a specific invoice is not found. To show an error UI to the user. Create a `not-found.tsx` file inside the `/edit` folder.

![The not-found.tsx file inside the edit folder]()

Then, inside the `not-found.tsx` file, paste the following the code:

`/dashboard/invoices/[id]/edit/not-found.tsx`

```tsx
import Link from "next/link";
import { FaceFrownIcon } from "@heroicons/react/24/outline";

export default function NotFound() {
  return (
    <main className="flex h-full flex-col items-center justify-center gap-2">
      <FaceFrownIcon className="w-10 text-gray-400" />
      <h2 className="text-xl font-semibold">404 Not Found</h2>
      <p>Could not find the requested invoice.</p>
      <Link
        href="/dashboard/invoices"
        className="mt-4 rounded-md bg-blue-500 px-4 py-2 text-sm text-white transition-colors hover:bg-blue-400"
      >
        Go Back
      </Link>
    </main>
  );
}
```

Refresh the route, and you should now see the following UI:

![404 Not Found Page]()

That's something to keep in mind, `notFound` will take precedence over `error.tsx`, so you can reach out for it when you want to handle more specific errors!

## Further reading

To learn more about error handling in Next.js, check out the following documentation:

- Error Handling
- `error.js` API Reference
- `notFound()` API Reference
- `not-found.js` API Reference
