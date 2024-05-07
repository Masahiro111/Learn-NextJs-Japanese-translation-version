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
> ただし、React では、`action` 属性は特別なプロップとみなされます。つまり、React はその上に構築され、アクションを呼び出すことができるようになります。
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

型の検証を行うには、いくつかの方法があります。手作業で型を検証することもできますが、型検証ライブラリを使用すると時間と労力を節約できます。例として、このタスクを簡素化できる TypeScript ファーストの検証ライブラリである [Zod](https://zod.dev/) を使用します。

`actions.ts` ファイルで Zod をインポートし、フォームオブジェクトの形状に一致するスキーマを定義します。このスキーマは、データベースに保存する前に `formData` を検証します。

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

`amount` フィールドは、文字列から数値に強制的に変更すると同時に、その型を検証するように設定されています。

次に、`rawFormData` を `CreateInvoice` に渡して型を検証します。

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

#### セント単位で値を保存

JavaScript の浮動小数点エラーを排除し、より高い精度を確保するために、通常、データベースに通貨値をセント単位で保存することをお勧めします。

金額をセントに変換しましょう。

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

#### 新しい日付の作成

最後に、請求書の作成日として「YYYY-MM-DD」の形式で新しい日付を作成しましょう。

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

5. データベースにデータを挿入

データベースに必要な値がすべて揃いました。新しい請求書をデータベースに挿入するために、SQL クエリの作成とクエリに必要な変数を渡します。

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

今はまだエラーを処理していません。次の章で処理を行います。とりあえず、次のステップに進みましょう。

### 6. 再検証とリダイレクト

Next.js には、ユーザーのブラウザにルートセグメントを一時的に保存する [クライアントサイドルーターキャッシュ](https://nextjs.org/docs/app/building-your-application/caching#router-cache) があります。[プリフェッチ](https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating#1-prefetching) と合わせて、このキャッシュはサーバーへのリクエスト数を減らしながら、ユーザーがルート間をすばやく移動できるようにします。

請求書ルートに表示されるデータを更新するので、このキャッシュをクリアして、サーバーへの新しいリクエストをトリガーしたいと思います。これは、Next.js の [revalidatePath](https://nextjs.org/docs/app/api-reference/functions/revalidatePath) 関数を使用して実行できます。

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

データベースが更新されると、`/dashboard/invoices` パスが再検証され、新しいデータがサーバーから取得されます。

この時点で、ユーザーを `/dashboard/invoices` ページにリダイレクトします。これは、Next.js の [redirect](https://nextjs.org/docs/app/api-reference/functions/redirect) 関数を使用して実行できます。

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

おめでとうございます！これで、最初のサーバーアクションが実装されました。新しい請求書を追加してテストしてみてください。

1. 送信すると、`/dashboard/invoices` ルートにリダイレクトされます
1. 表の一番上に新しい請求書が表示されます

## 請求書の更新

請求書の更新フォームは請求書の作成フォームに似ていますが、データベースのレコードを更新するために請求書の `id` を渡す必要がある点が異なります。請求書の `id` を取得して渡し方を見てみましょう。

請求書を更新する手順は以下の通りです。

1. 請求書の `id` を使用して新しい動的ルートセグメントを作成します
1. ページパラメータから請求書の `id` を読み取ります
1. データベースから指定の請求書を取得します
1. 請求書のデータをフォームへ事前に入力します
1. データベースの請求書データを更新します

### 1. 請求書 `id` を使用した動的ルートセグメントの作成

Next.js では、正確なセグメント名がわからず、データに基づいてルートを作成したい場合に、[ダイナミックルートセグメント](https://nextjs.org/docs/app/building-your-application/routing/dynamic-routes)を作成することができます。例えば、ブログ記事のタイトルや商品ページなどです。フォルダの名前を角括弧で囲むことで、動的なルートセグメントを作成できます。たとえば `[id]`、`[post]`、`[slug]`などです。

`/invoices` フォルダに、`[id]` という名前の新しい動的ルートを作成し、次に `page.tsx` ファイルを含む `edit` という名前の新しいルートを作成します。ファイル構造は次のようになります。

![ネストされた [id] フォルダーとその中に edit フォルダがある invoices フォルダ]()

`<Table>` コンポーネントには、テーブルレコードから請求書の `id` を受け取る `<UpdateInvoice />` ボタンがあることに注目してください。

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

`<UpdateInvoice />` コンポーネントに移動し、`id` プロップを受け入れるように `Link` の `href` を更新します。動的ルートセグメントにリンクするためにテンプレートリテラルを使用します。

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

2. Page コンポーネントの `params` から請求書 `id` を読み込む

`<Page>` コンポーネントに戻り、次のコードを貼り付けます。

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

これは `/create` 請求書ページと似ていますが、（`edit-form.tsx` ファイルから）別のフォームをインポートしていることに注意してください。このフォームには、顧客の名前、請求金額、ステータスの `defaultValue` を **事前に入力** しておく必要があります。フォームフィールドに事前に入力するには、`id` を使用して指定の請求書を取得する必要があります。

`searchParams` に加えて、ページコンポーネントは `id` にアクセスするために使用できる `params` というプロップも受け取ります。`<Page>` コンポーネントを更新してプロップを受け取ってみてください。

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

### 3. 指定の請求書を取得

そして

- `fetchInvoiceById` という新しい関数をインポートし、引数として `id` を渡します
- `fetchCustomers` をインポートして、ドロップダウン用の顧客名を取得します

`Promise.all` を使用すると、請求書と顧客の両方を並行して取得することができます。

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

`invoice` が `undefined` である可能性があるため、ターミナルで `invoice` プロップの一時的な TS エラーが表示されます。次の章でエラー処理を追加するときに解決するので、今は気にしないでください。

素晴らしい！それでは、すべてが正しく記述されているかテストしてみましょう。http://localhost:3000/dashboard/invoices にアクセスし、鉛筆のアイコンをクリックして請求書を編集します。ナビゲーションが完了すると、請求書の詳細が事前に入力されたフォームが表示されます。

![パンくずリストとフォームを含む請求書ページの編集]()

URL も次のような `id` で更新されるはずです。http://localhost:3000/dashboard/invoice/uuid/edit

> [!note]
>
> **UUID と自動インクリメントキー**
>
> インクリメントキー（1、2、3 など）の代わりに UUID を使用しています。これは URL が長くなりますが、UUID は ID 衝突のリスクを排除し、グローバルに一意であり、列挙型攻撃のリスクを軽減するため、大規模なデータベースに最適です。
>
> ただし、よりクリーンな URL を好む場合は、自動インクリメントキーを使用することをお勧めします。

### 4. id をサーバーアクションに渡す

最後に、データベースの適切なレコードを更新できるように、`id` をサーバーアクションに渡します。次のように `id` を引数として渡すことは **できません**。

`/app/ui/invoices/edit-form.tsx`

```tsx
// Passing an id as argument won't work
<form action={updateInvoice(id)}>
```

代わりに、JS の `bind` を使用して `id` をサーバーアクションに渡すことができます。こうすることで、サーバーアクションに渡される値がエンコードされるようになります。

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
> フォームに隠し入力フィールドを使うこともできます（例：`<input type="hidden" name="id" value={invoice.id} />`）。ただし、値は HTML ソースにフルテキストとして表示されるため、ID などの機密データには適していません。

次に、`actions.ts` ファイルに新しいアクション `updateInvoice` を作成します。

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

`createInvoice` アクションと同様に、次のようになります。

- `formData` からデータを抽出します
- Zod を使用して型を検証します
- 金額をセントに変換します
- SQL クエリに変数を渡します
- `revalidatePath` を呼び出してクライアントキャッシュをクリアし、新しいサーバーリクエストを作成します
- `redirect` を呼び出して、ユーザーを請求書のページにリダイレクトします

請求書を編集してテストしてみましょう。フォームを送信すると、請求書ページにリダイレクトされ、請求書が更新されるはずです。

## 請求書の削除

サーバーアクションを使用して請求書を削除するには、削除ボタンを `<form>` 要素でラップし、`bind` を使用して `id` をサーバーアクションに渡します。

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

`actions.ts` ファイルに、`deleteInvoice` という新しいアクションを作成します。

`/app/lib/actions.ts`

```ts
export async function deleteInvoice(id: string) {
  await sql`DELETE FROM invoices WHERE id = ${id}`;
  revalidatePath("/dashboard/invoices");
}
```

このアクションは `/dashboard/invoices` パスで呼び出されるので、`redirect` を呼び出す必要はありません。`revalidatePath` を呼び出すと、新しいサーバーリクエストが発生し、テーブルが再レンダリングされます。

## 参考文献

この章では、サーバーアクションを使用してデータを変更する方法を学習しました。また、`revalidatePath` API を使用して Next.js のキャッシュを再検証し、`redirect` を使用してユーザーを新しいページにリダイレクトする方法も学びました。

さらに、[サーバーアクションによるセキュリティ](https://nextjs.org/blog/security-nextjs-server-components-actions) についてもお読みください。
