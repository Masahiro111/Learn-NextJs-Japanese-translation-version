# データの変更

前の章では、URL 検索パラメータ と Next.js API を使用して検索とページネーションを実装しました。請求書を作成、更新、削除する機能を追加して、請求書ページの作業を続けましょう。

この章で取り上げるトピックは以下のとおりです。

- React Server Actions とは何か、またそれを使用してデータを変更する方法
- フォームとサーバーコンポーネントの扱い方
- 型の検証を含む、ネイティブの formData オブジェクトを使用する際のベストプラクティス
- revalidatePath API を使用してクライアントキャッシュを再検証する方法
- 特定の ID を持つ動的ルートセグメントを作成する方法

## サーバーアクションとは？

React Server Actions を使用すると、サーバー上で非同期コードを直接実行できます。データを変更するために API エンドポイントを作成する必要がなくなります。その代わりに、サーバー上で実行される非同期関数を記述し、クライアントまたはサーバーコンポーネントから呼び出すことができます。

Web アプリケーションは様々な脅威にさらされやすいため、セキュリティは最優先事項です。そこでサーバーアクションの出番です。サーバーアクションは効果的なセキュリティソリューションを提供し、さまざまなタイプの攻撃から保護し、データを保護し、許可されたアクセスを保証します。サーバーアクションは、POST リクエスト、暗号化されたクロージャ、厳密な入力チェック、エラーメッセージのハッシュ化、ホスト制限などの技術によってこれを実現し、これらすべてが連携してアプリの安全性を大幅に向上させます。

## サーバーアクションでのフォームの使用

React では、`<form>` 要素の `action` 属性を使用してアクションを呼び出すことができます。アクションは、取り込んだデータを含むネイティブ [FormData](https://developer.mozilla.org/en-US/docs/Web/API/FormData) オブジェクトを自動的に受け取ります。

たとえば

```ts
// Server Component
export default function Page() {
  // Action
  async function create(formData: FormData) {
    "use server";

    // Logic to mutate data...
  }

  // Invoke the action using the "action" attribute
  return <form action={create}>...</form>;
}
```

サーバーコンポーネント内でサーバーアクションを呼び出すことの利点は、段階的な機能拡張です。クライアントで JavaScript が無効になっている場合でもフォームは機能します。

## サーバーアクションと Next.js

サーバーアクションは、Next.js の [キャッシュ](https://nextjs.org/docs/app/building-your-application/caching) とも緊密に統合されています。サーバーアクションを使ってフォームが送信されると、そのアクションを使用してデータを変更できるだけでなく、`revalidatePath` や `revalidateTag` などの API を使って、関連するキャッシュを再検証することもできます。

どのように機能するかを見てみましょう！

## 請求書の作成

新しい請求書を作成する手順は次のとおりです。

1. ユーザーの入力を取得するフォームを作成します
1. サーバーアクションを作成し、フォームから呼び出します
1. サーバーアクション内で、formData オブジェクトからデータを抽出します
1. データベースに挿入するデータを検証して準備します
1. データを挿入し、エラーがあれば処理します
1. キャッシュを再検証し、ユーザーを請求書ページにリダイレクトします

### 1. 新しいルートとフォームの作成

まず、`/invoices` フォルダー内に、`/create` という新しいルートセグメントと `page.tsx` ファイルを追加します。

![請求書フォルダの中に入れ子になった create フォルダがあり、その中に page.tsx ファイルがある]()

このルートを使って新しい請求書を作成します。`page.tsx` ファイル内に次のコードを貼り付けて、時間をかけてコードを読んでみましょう。

`/dashboard/invoices/create/page.tsx`

```tsx
import Form from "@/app/ui/invoices/create-form";
import Breadcrumbs from "@/app/ui/invoices/breadcrumbs";
import { fetchCustomers } from "@/app/lib/data";

export default async function Page() {
  const customers = await fetchCustomers();

  return (
    <main>
      <Breadcrumbs
        breadcrumbs={[
          { label: "Invoices", href: "/dashboard/invoices" },
          {
            label: "Create Invoice",
            href: "/dashboard/invoices/create",
            active: true,
          },
        ]}
      />
      <Form customers={customers} />
    </main>
  );
}
```

あなたのページは、`customers` をフェッチし、それを `<Form>` コンポーネントに渡すサーバーコンポーネントです。時間を節約するために、`<Form>` コンポーネントはすでに作成されています。

`<Form>` コンポーネントに移動すると、フォームが次のようになっていることがわかります。

- 顧客リストの `<select>` （ドロップダウン）要素があります
- `type="number"` で金額を指定する `<input>` 要素があります
- `type="radio"` のステータスを表す `<input>` 要素が 2 つあります
- `type="submit"` のボタンがあります

http://localhost:3000/dashboard/invoices/create に、次の UI が表示されるはずです。

![パンくずリストとフォーム2で請求書ページを作成する]()

### サーバーアクションの作成

フォームの送信時に呼び出されるサーバーアクションを作成しましょう。

`lib` ディレクトリに移動し、`actions.ts` という名前の新しいファイルを作成します。このファイルの先頭に、React [use server](https://react.dev/reference/react/use-server) ディレクティブを追加します。

`/app/lib/actions.ts`

```ts
"use server";
```

`use server` を追加すると、ファイル内のエクスポートされるすべての関数がサーバー関数としてマークされます。これらのサーバー機能はクライアントコンポーネントとサーバー コンポーネントにインポートできるため、非常に多用途なものになります。

アクション内に `use server` を追加することで、サーバーコンポーネント内にサーバーアクションを直接記述することもできます。ただし、このコースでは、それらをすべて別のファイルに整理しています。

`actions.ts` ファイルに、`formData` を受け取る新しい非同期関数を作成します。

`/app/lib/actions.ts`

```diff ts
  'use server';

+ export async function createInvoice(formData: FormData) {}
```

次に、`<Form>` コンポーネントに actions.ts ファイルから `createInvoice` をインポートします。`action` 属性を `<form>` 要素に追加し、`createInvoice` アクションを呼び出します。

`/app/ui/invoices/create-form.tsx`

```diff tsx
  import { customerField } from '@/app/lib/definitions';
  import Link from 'next/link';
  import {
    CheckIcon,
    ClockIcon,
    CurrencyDollarIcon,
    UserCircleIcon,
  } from '@heroicons/react/24/outline';
  import { Button } from '@/app/ui/button';
+ import { createInvoice } from '@/app/lib/actions';

  export default function Form({
    customers,
  }: {
    customers: customerField[];
  }) {
    return (
+     <form action={createInvoice}>
        // ...
    )
  }
```

> [!tip]
>
> **知っておくと便利**：HTML では、`action` 属性に URL を記述します。この URL は、フォームデータの送信先（通常は API エンドポイント）になります。
>
> ただし、React では、`action` 属性は特別なプロパティとみなされます。つまり、React はその上に構築され、アクションを呼び出すことができるようになります。
>
> サーバーアクションは背後で `POST` API エンドポイントを作成します。このため、サーバーアクションを使用する際に API エンドポイントを手動で作成する必要がありません。

### 3. `formData` からデータを抽出

`actions.ts` ファイルに戻って、`formData` の値を抽出する必要があります。[いくつかの方法](https://developer.mozilla.org/en-US/docs/Web/API/FormData/append) がありますが、今回の例では [.get(name)](https://developer.mozilla.org/en-US/docs/Web/API/FormData/get) メソッドを使用してみましょう。

`/app/lib/actions.ts`

```diff ts
  'use server';

+ export async function createInvoice(formData: FormData) {
+   const rawFormData = {
+     customerId: formData.get('customerId'),
+     amount: formData.get('amount'),
+     status: formData.get('status'),
+   };
+   // Test it out:
+   console.log(rawFormData);
  }
```

> [!tip]
>
> 多くのフィールドを持つフォームを扱う場合、JavaScript の [Object.fromEntries()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/fromEntries) と一緒に [entries()](https://developer.mozilla.org/en-US/docs/Web/API/FormData/entries) メソッドを使うことを検討するとよいでしょう。たとえば
>
> `const rawFormData = Object.fromEntries(formData.entries())`

すべてが正しく接続されていることを確認するために、先に進んでフォームをテストしてください。送信すると、フォームに入力したデータがターミナルに記録されるはずです。

データがオブジェクトの形になったので、より簡単に作業できるようになります。

### 4. データを検証と準備

フォームデータをデータベースに送る前に、正しいフォーマットと正しいデータ型であることを確認します。このコースの最初のほうで、請求書テーブルが次のような形式のデータを必要としていることを思い出してください。

`/app/lib/definitions.ts`

```ts
export type Invoice = {
  id: string; // Will be created on the database
  customer_id: string;
  amount: number; // Stored in cents
  status: "pending" | "paid";
  date: string;
};
```

これまでのところ、フォームから取得できるのは `customer_id`、`amount`、`status` のみです。

#### 型の検証と強制

フォームからのデータがデータベースで期待される型と一致しているかどうかを検証することは重要です。たとえば、アクションの中に `console.log` を追加したとすると次のようになります。

```js
console.log(typeof rawFormData.amount);
```

`amount` の型は `number` ではなく `string` であることがわかります。これは、`type="number"` を持つ `input` 要素が実際には数値ではなく文字列を返すためです。

To handle type validation, you have a few options. While you can manually validate types, using a type validation library can save you time and effort. For your example, we'll use [Zod](https://zod.dev/), a TypeScript-first validation library that can simplify this task for you.

In your `actions.ts` file, import Zod and define a schema that matches the shape of your form object. This schema will validate the `formData` before saving it to a database.

`/app/lib/actions.ts`

```diff ts
  'use server';

+ import { z } from 'zod';

+ const FormSchema = z.object({
+   id: z.string(),
+   customerId: z.string(),
+   amount: z.coerce.number(),
+   status: z.enum(['pending', 'paid']),
+   date: z.string(),
+ });

+ const CreateInvoice = FormSchema.omit({ id: true, date: true });

  export async function createInvoice(formData: FormData) {
    // ...
  }
```

The `amount` field is specifically set to coerce (change) from a string to a number while also validating its type.

You can then pass your `rawFormData` to `CreateInvoice` to validate the types:

`/app/lib/actions.ts`

```diff ts
  // ...
  export async function createInvoice(formData: FormData) {
+   const { customerId, amount, status } = CreateInvoice.parse({
      customerId: formData.get('customerId'),
      amount: formData.get('amount'),
      status: formData.get('status'),
    });
  }
```

#### Storing values in cents

It's usually good practice to store monetary values in cents in your database to eliminate JavaScript floating-point errors and ensure greater accuracy.

Let's convert the amount into cents:

`/app/lib/actions.ts`

```diff ts
  // ...
  export async function createInvoice(formData: FormData) {
    const { customerId, amount, status } = CreateInvoice.parse({
      customerId: formData.get('customerId'),
      amount: formData.get('amount'),
      status: formData.get('status'),
    });
+   const amountInCents = amount * 100;
  }
```

#### Creating new dates

Finally, let's create a new date with the format "YYYY-MM-DD" for the invoice's creation date:

`/app/lib/actions.ts`

```diff ts
  // ...
  export async function createInvoice(formData: FormData) {
    const { customerId, amount, status } = CreateInvoice.parse({
      customerId: formData.get('customerId'),
      amount: formData.get('amount'),
      status: formData.get('status'),
    });
    const amountInCents = amount * 100;
+   const date = new Date().toISOString().split('T')[0];
  }
```

5.  Inserting the data into your database

Now that you have all the values you need for your database, you can create an SQL query to insert the new invoice into your database and pass in the variables:

`/app/lib/actions.ts`

```diff ts
  import { z } from 'zod';
+ import { sql } from '@vercel/postgres';

  // ...

  export async function createInvoice(formData: FormData) {
    const { customerId, amount, status } = CreateInvoice.parse({
      customerId: formData.get('customerId'),
      amount: formData.get('amount'),
      status: formData.get('status'),
    });
    const amountInCents = amount * 100;
    const date = new Date().toISOString().split('T')[0];

+   await sql`
+     INSERT INTO invoices (customer_id, amount, status, date)
+     VALUES (${customerId}, ${amountInCents}, ${status}, ${date})
+   `;
  }
```

Right now, we're not handling any errors. We'll do it in the next chapter. For now, let's move on to the next step.

### 6. Revalidate and redirect

Next.js has a [Client-side Router Cache](https://nextjs.org/docs/app/building-your-application/caching#router-cache) that stores the route segments in the user's browser for a time. Along with [prefetching](https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating#1-prefetching), this cache ensures that users can quickly navigate between routes while reducing the number of requests made to the server.

Since you're updating the data displayed in the invoices route, you want to clear this cache and trigger a new request to the server. You can do this with the [revalidatePath](https://nextjs.org/docs/app/api-reference/functions/revalidatePath) function from Next.js:

`/app/lib/actions.ts`

```diff ts
  'use server';

  import { z } from 'zod';
  import { sql } from '@vercel/postgres';
+ import { revalidatePath } from 'next/cache';

  // ...

  export async function createInvoice(formData: FormData) {
    const { customerId, amount, status } = CreateInvoice.parse({
      customerId: formData.get('customerId'),
      amount: formData.get('amount'),
      status: formData.get('status'),
    });
    const amountInCents = amount * 100;
    const date = new Date().toISOString().split('T')[0];

    await sql`
      INSERT INTO invoices (customer_id, amount, status, date)
      VALUES (${customerId}, ${amountInCents}, ${status}, ${date})
    `;

+   revalidatePath('/dashboard/invoices');
  }
```

Once the database has been updated, the `/dashboard/invoices` path will be revalidated, and fresh data will be fetched from the server.

At this point, you also want to redirect the user back to the `/dashboard/invoices` page. You can do this with the [redirect](https://nextjs.org/docs/app/api-reference/functions/redirect) function from Next.js:

`/app/lib/actions.ts`

```diff ts
  'use server';

  import { z } from 'zod';
  import { sql } from '@vercel/postgres';
  import { revalidatePath } from 'next/cache';
+ import { redirect } from 'next/navigation';

  // ...

  export async function createInvoice(formData: FormData) {
    // ...

    revalidatePath('/dashboard/invoices');
+   redirect('/dashboard/invoices');
  }
```

Congratulations! You've just implemented your first Server Action. Test it out by adding a new invoice, if everything is working correctly:

1. You should be redirected to the `/dashboard/invoices` route on submission.
1. You should see the new invoice at the top of the table.

## Updating an invoice

The updating invoice form is similar to the create an invoice form, except you'll need to pass the invoice `id` to update the record in your database. Let's see how you can get and pass the invoice `id`.

These are the steps you'll take to update an invoice:

1. Create a new dynamic route segment with the invoice `id`.
1. Read the invoice `id` from the page params.
1. Fetch the specific invoice from your database.
1. Pre-populate the form with the invoice data.
1. Update the invoice data in your database.

### 1. Create a Dynamic Route Segment with the invoice `id`

Next.js allows you to create [Dynamic Route Segments](https://nextjs.org/docs/app/building-your-application/routing/dynamic-routes) when you don't know the exact segment name and want to create routes based on data. This could be blog post titles, product pages, etc. You can create dynamic route segments by wrapping a folder's name in square brackets. For example, `[id]`, `[post]` or `[slug]`.

In your `/invoices` folder, create a new dynamic route called `[id]`, then a new route called `edit` with a `page.tsx` file. Your file structure should look like this:

![Invoices folder with a nested [id] folder, and an edit folder inside it]()

In your `<Table>` component, notice there's a `<UpdateInvoice />` button that receives the invoice's `id` from the table records.

`/app/ui/invoices/table.tsx`

```diff tsx
  export default async function InvoicesTable({
    query,
    currentPage,
  }: {
    query: string;
    currentPage: number;
  }) {
    return (
      // ...
      <td className="flex justify-end gap-2 whitespace-nowrap px-6 py-4 text-sm">
+       <UpdateInvoice id={invoice.id} />
        <DeleteInvoice id={invoice.id} />
      </td>
      // ...
    );
  }
```

Navigate to your `<UpdateInvoice />` component, and update the `href` of the `Link` to accept the `id` prop. You can use template literals to link to a dynamic route segment:

`/app/ui/invoices/buttons.tsx`

```diff tsx
  import { PencilIcon, PlusIcon, TrashIcon } from '@heroicons/react/24/outline';
  import Link from 'next/link';

  // ...

  export function UpdateInvoice({ id }: { id: string }) {
    return (
      <Link
+       href={`/dashboard/invoices/${id}/edit`}
        className="rounded-md border p-2 hover:bg-gray-100"
      >
        <PencilIcon className="w-5" />
      </Link>
    );
  }
```

2. Read the invoice `id` from page `params`

Back on your `<Page>` component, paste the following code:

`/app/dashboard/invoices/[id]/edit/page.tsx`

```tsx
import Form from "@/app/ui/invoices/edit-form";
import Breadcrumbs from "@/app/ui/invoices/breadcrumbs";
import { fetchCustomers } from "@/app/lib/data";

export default async function Page() {
  return (
    <main>
      <Breadcrumbs
        breadcrumbs={[
          { label: "Invoices", href: "/dashboard/invoices" },
          {
            label: "Edit Invoice",
            href: `/dashboard/invoices/${id}/edit`,
            active: true,
          },
        ]}
      />
      <Form invoice={invoice} customers={customers} />
    </main>
  );
}
```

Notice how it's similar to your `/create` invoice page, except it imports a different form (from the `edit-form.tsx` file). This form should be **pre-populated** with a `defaultValue` for the customer's name, invoice amount, and status. To pre-populate the form fields, you need to fetch the specific invoice using `id`.

In addition to `searchParams`, page components also accept a prop called `params` which you can use to access the `id`. Update your `<Page>` component to receive the prop:

`/app/dashboard/invoices/[id]/edit/page.tsx`

```diff tsx
  import Form from '@/app/ui/invoices/edit-form';
  import Breadcrumbs from '@/app/ui/invoices/breadcrumbs';
  import { fetchCustomers } from '@/app/lib/data';

+ export default async function Page({ params }: { params: { id: string } }) {
+   const id = params.id;
    // ...
  }
```

### 3. Fetch the specific invoice

Then:

- Import a new function called `fetchInvoiceById` and pass the `id` as an argument.
- Import `fetchCustomers` to fetch the customer names for the dropdown.

You can use `Promise.all` to fetch both the invoice and customers in parallel:

`/dashboard/invoices/[id]/edit/page.tsx`

```diff tsx
  import Form from '@/app/ui/invoices/edit-form';
  import Breadcrumbs from '@/app/ui/invoices/breadcrumbs';
+ import { fetchInvoiceById, fetchCustomers } from '@/app/lib/data';

  export default async function Page({ params }: { params: { id: string } }) {
    const id = params.id;
+   const [invoice, customers] = await Promise.all([
+     fetchInvoiceById(id),
+     fetchCustomers(),
+   ]);
    // ...
  }
```

You will see a temporary TS error for the `invoice` prop in your terminal because `invoice` could be potentially `undefined`. Don't worry about it for now, you'll resolve it in the next chapter when you add error handling.

Great! Now, test that everything is wired correctly. Visit http://localhost:3000/dashboard/invoices and click on the Pencil icon to edit an invoice. After navigation, you should see a form that is pre-populated with the invoice details:

![Edit invoices page with breadcrumbs and form]()

The URL should also be updated with an `id` as follows: http://localhost:3000/dashboard/invoice/uuid/edit

> [!info]
>
> **UUIDs vs. Auto-incrementing Keys**
>
> We use UUIDs instead of incrementing keys (e.g., 1, 2, 3, etc.). This makes the URL longer; however, UUIDs eliminate the risk of ID collision, are globally unique, and reduce the risk of enumeration attacks - making them ideal for large databases.
>
> However, if you prefer cleaner URLs, you might prefer to use auto-incrementing keys.

### 4. Pass the id to the Server Action

Lastly, you want to pass the `id` to the Server Action so you can update the right record in your database. You **cannot** pass the `id` as an argument like so:

`/app/ui/invoices/edit-form.tsx`

```tsx
// Passing an id as argument won't work
<form action={updateInvoice(id)}>
```

Instead, you can pass `id` to the Server Action using JS `bind`. This will ensure that any values passed to the Server Action are encoded.

`/app/ui/invoices/edit-form.tsx`

```diff tsx
  // ...
+ import { updateInvoice } from '@/app/lib/actions';

  export default function EditInvoiceForm({
    invoice,
    customers,
  }: {
    invoice: InvoiceForm;
    customers: CustomerField[];
  }) {
+   const updateInvoiceWithId = updateInvoice.bind(null, invoice.id);

    return (
+     <form action={updateInvoiceWithId}>
        <input type="hidden" name="id" value={invoice.id} />
      </form>
    );
  }
```

> [!note]
>
> Using a hidden input field in your form also works (e.g. `<input type="hidden" name="id" value={invoice.id} />`). However, the values will appear as full text in the HTML source, which is not ideal for sensitive data like IDs.

Then, in your `actions.ts` file, create a new action, `updateInvoice`:

`/app/lib/actions.ts`

```ts
// Use Zod to update the expected types
const UpdateInvoice = FormSchema.omit({ id: true, date: true });

// ...

export async function updateInvoice(id: string, formData: FormData) {
  const { customerId, amount, status } = UpdateInvoice.parse({
    customerId: formData.get("customerId"),
    amount: formData.get("amount"),
    status: formData.get("status"),
  });

  const amountInCents = amount * 100;

  await sql`
    UPDATE invoices
    SET customer_id = ${customerId}, amount = ${amountInCents}, status = ${status}
    WHERE id = ${id}
  `;

  revalidatePath("/dashboard/invoices");
  redirect("/dashboard/invoices");
}
```

Similarly to the `createInvoice` action, here you are:

- Extracting the data from `formData`.
- Validating the types with Zod.
- Converting the amount to cents.
- Passing the variables to your SQL query.
- Calling `revalidatePath` to clear the client cache and make a new server request.
- Calling `redirect` to redirect the user to the invoice's page.

Test it out by editing an invoice. After submitting the form, you should be redirected to the invoices page, and the invoice should be updated.

## Deleting an invoice

To delete an invoice using a Server Action, wrap the delete button in a `<form>` element and pass the `id` to the Server Action using `bind`:

`/app/ui/invoices/buttons.tsx`

```diff tsx
+ import { deleteInvoice } from '@/app/lib/actions';

  // ...

+ export function DeleteInvoice({ id }: { id: string }) {
+   const deleteInvoiceWithId = deleteInvoice.bind(null, id);

    return (
+     <form action={deleteInvoiceWithId}>
        <button className="rounded-md border p-2 hover:bg-gray-100">
          <span className="sr-only">Delete</span>
          <TrashIcon className="w-4" />
        </button>
+     </form>
    );
  }
```

Inside your `actions.ts` file, create a new action called `deleteInvoice`.

`/app/lib/actions.ts`

```ts
export async function deleteInvoice(id: string) {
  await sql`DELETE FROM invoices WHERE id = ${id}`;
  revalidatePath("/dashboard/invoices");
}
```

Since this action is being called in the `/dashboard/invoices` path, you don't need to call `redirect`. Calling `revalidatePath` will trigger a new server request and re-render the table.

## Further reading

In this chapter, you learned how to use Server Actions to mutate data. You also learned how to use the `revalidatePath` API to revalidate the Next.js cache and `redirect` to redirect the user to a new page.

You can also read more about [security with Server Actions](https://nextjs.org/blog/security-nextjs-server-components-actions) for additional learning.
