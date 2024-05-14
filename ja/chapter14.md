# アクセシビリティの向上

前の章では、エラー（404 エラーを含む）をキャッチし、ユーザーにフォールバックを表示する方法について説明しました。しかし、パズルのもう 1 つのピースであるフォームバリデーションについてはまだ説明する必要があります。ここでは、Server Actions を使用してサーバーサイドのバリデーションを実装する方法と、`useFormState` フックを使用してフォームエラーを表示する方法について、アクセシビリティを考慮しながら説明します！

この章で取り上げるトピックは以下のとおりです。

- Next.js で `eslint-plugin-jsx-a11y` を使用してアクセシビリティのベストプラクティスを実装する方法
- サーバーサイドのフォームバリデーションの実装方法
- React の `useFormState` フックを使用してフォームエラーを処理し、ユーザーに表示する方法

## アクセシビリティとは？

アクセシビリティとは、障害のある人を含め、誰もが利用できる Web アプリケーションを設計、実装することを指します。キーボードナビゲーション、セマンティック HTML、画像、色、動画など、多くの領域をカバーする広大なトピックです。

このコースでは、アクセシビリティについて深く掘り下げることはしませんが、Next.js で利用可能なアクセシビリティ機能と、アプリケーションをよりアクセシブルにするための一般的な実践方法について説明します。

> アクセシビリティについてさらに詳しく知りたい場合は、[web.dev](https://web.dev/) の [Learn Accessibility](https://web.dev/learn/accessibility/) コースをお勧めします。

## Next.js で ESLint アクセシビリティプラグインを使う

デフォルトでは、Next.js にはアクセシビリティの問題を早期に発見するのに役立つ [eslint-plugin-jsx-a11y](https://www.npmjs.com/package/eslint-plugin-jsx-a11y) プラグインが含まれています。たとえば、画像に `alt` テキストがなかったり、`aria-*` 属性や role 属性が正しく使われていなかったりすると警告が表示されます。

どのように機能するかを見てみましょう！

`package.json` ファイルにスクリプトとして `next lint` を追加します。

`/package.json`

```diff js
  "scripts": {
      "build": "next build",
      "dev": "next dev",
      "seed": "node -r dotenv/config ./scripts/seed.js",
      "start": "next start",
+     "lint": "next lint"
  },
```

次に、ターミナルで `npm run lint` を実行します。

`ターミナル`

```command
npm run lint
```

次のような警告が表示されるはずです。

`ターミナル`

```
✔ No ESLint warnings or errors
```

しかし、もし `alt` テキストがない画像があったらどうなるでしょうか？確認してみましょう！

`/app/ui/invoices/table.tsx` に移動し、画像から `alt` プロパティを削除します。エディタの検索機能を使えば、すぐに `<Image>` を見つけることができます。

`/app/ui/invoices/table.tsx`

```diff tsx
  <Image
    src={invoice.image_url}
    className="rounded-full"
    width={28}
    height={28}
-   alt={`${invoice.name}'s profile picture`} // Delete this line
  />
```

もう一度 `npm run lint` を実行すると、次のような警告が表示されるはずです。

`ターミナル`

```
./app/ui/invoices/table.tsx
45:25  Warning: Image elements must have an alt prop,
either with meaningful text, or an empty string for decorative images. jsx-a11y/alt-text
```

アプリケーションを Vercel にデプロイしようとすると、警告がビルドログにも表示されます。これは、`next lint` がビルドプロセスの一部として実行されるためです。したがって、アプリケーションをデプロイする前にローカルで `lint` を実行してアクセシビリティの問題を見つけることができます。

## フォームのアクセシビリティの向上

フォームのアクセシビリティを向上させるために、私たちがすでに行っていることが 3 つあります。

- **セマンティック HTML** : `<div>` の代わりにセマンティック要素（`<input>`、`<option>` など）を使用します。これにより、支援技術（AT）が入力要素に焦点を当て、適切なコンテキスト情報をユーザーに提供できるようになり、フォームのナビゲーションや理解が容易になります。
- **ラベル付け** : `<label>` と `htmlFor` 属性を含めることで、各フォームフィールドに説明的なテキストラベルが付けられます。これにより、コンテキストが提供されることで AT サポートが向上し、ユーザーがラベルをクリックして対応する入力フィールドにフォーカスできるようにすることでユーザビリティを向上させます。
- **フォーカスアウトライン** : フィールドがフォーカスされているときにアウトラインが表示されるように、フィールドは適切にスタイルされます。これはアクセシビリティにとって重要で、ページ上のアクティブな要素を視覚的に示し、キーボードとスクリーンリーダーの両方のユーザーがフォームのどこにいるかを理解するのに役立ちます。これは、`tab`を押すことで確認することができます。

これらのプラクティスは、フォームを多くのユーザーにとってよりアクセシブルなものにするための良い基礎となります。しかし、`フォームの検証（バリデーション）` や `エラー` には対応していません。

## フォームバリデーション

http://localhost:3000/dashboard/invoices/create に移動し、空のフォームを送信します。何が起こるのですか？

エラーが発生します。これは、空のフォーム値をサーバーアクションに送信しているためです。クライアント側またはサーバー側でフォームのバリデーションを行うことで、これを防ぐことができます。

### クライアントサイドのバリデーション

クライアント側でフォームを検証する方法はいくつかあります。最も簡単な方法は、フォーム内の `<input>` 要素と `<select>` 要素に `required` 属性を追加することで、ブラウザが提供するフォームバリデーションに頼ることができます。たとえば

`/app/ui/invoices/create-form.tsx`

```diff tsx
  <input
    id="amount"
    name="amount"
    type="number"
    placeholder="Enter USD amount"
    className="peer block w-full rounded-md border border-gray-200 py-2 pl-10 text-sm outline-2 placeholder:text-gray-500"
+   required
  />
```

フォームをもう一度送信してみましょう。今度は空の値でフォームを送信するのでブラウザに警告が表示されるはずです。

AT によってはブラウザのバリデーションをサポートしているものもあるので、この方法は一般的に問題ありません。

クライアントサイドのバリデーションに代わるものとして、サーバーサイドのバリデーションがあります。次のセクションで実装方法を見てみましょう。ここでは、`required` 属性を追加した場合は削除してください。

### サーバーサイドのバリデーション

サーバー上でフォームのバリデーションを行うことで、次のことが可能になります。

- データベースにデータを送信する前に、データが期待通りの形式であることを確認します
- 悪意のあるユーザーがクライアント側のバリデーションを回避するリスクを減らします
- 有効なデータであるかどうかの判断基準をひとつにまとめられます

`create-form.tsx` コンポーネントで、`react-dom` から `useFormState` フックをインポートします。`useFormState` はフックであるため、`"use client"` ディレクティブを使用してフォームをクライアントコンポーネントにする必要があります。

`/app/ui/invoices/create-form.tsx`

```diff tsx
+ 'use client';

  // ...
+ import { useFormState } from 'react-dom';
```

フォームコンポーネント内部の `useFormState` フックは

- 2 つの引数を取ります (`action`, `initialState`)
- 2 つの値を返します `[state, dispatch]` - フォームの state とディスパッチ関数（[useReducer](https://react.dev/reference/react/useReducer) と同様）

`useFormState` の引数に `createInvoice` アクションを渡し、`<form action={}>` 属性内で `dispatch` を呼び出します。

`/app/ui/invoices/create-form.tsx`

```diff tsx
  // ...
  import { useFormState } from 'react-dom';

  export default function Form({ customers }: { customers: CustomerField[] }) {
+   const [state, dispatch] = useFormState(createInvoice, initialState);

+   return <form action={dispatch}>...</form>;
  }
```

`initialState` は任意に定義できます。この場合は、2 つの空のキー（`message` と `errors`）を持つオブジェクトを作成します。

`/app/ui/invoices/create-form.tsx`

```diff tsx
  // ...
  import { useFormState } from 'react-dom';

  export default function Form({ customers }: { customers: CustomerField[] }) {
+   const initialState = { message: null, errors: {} };
    const [state, dispatch] = useFormState(createInvoice, initialState);

    return <form action={dispatch}>...</form>;
  }
```

最初は混乱するかもしれませんが、サーバーアクションを更新すれば、より理解できるようになります。それではやってみましょう。

`action.ts` ファイルでは、Zod を使用してフォームデータを検証できます。次のように `FormSchema` を更新します。

`/app/lib/action.ts`

```diff tsx
  const FormSchema = z.object({
    id: z.string(),
    customerId: z.string({
+     invalid_type_error: 'Please select a customer.',
    }),
    amount: z.coerce
      .number()
+     .gt(0, { message: 'Please enter an amount greater than $0.' }),
    status: z.enum(['pending', 'paid'], {
+     invalid_type_error: 'Please select an invoice status.',
    }),
    date: z.string(),
  });
```

- `customerId` - Zod は `string` 型を想定しているため、customer フィールドが空の場合、エラーをスローします。しかし、ユーザーが顧客を選択しなかった場合にフレンドリーなメッセージを追加してみましょう
- `amount` - 金額の型を `string` から `number` に強制しているので、文字列が空の場合はデフォルトで 0 になります。`.gt()` 関数を使用して、常に 0 より大きい金額を行事するよう Zod に伝えましょう
- `status` - Zod は "pending "または "paid "を期待しているので、status フィールドが空だとエラーをスローします。ユーザーがステータスを選択しなかった場合にフレンドリーなメッセージを追加してみましょう

次に、`createInvoice`アクションを更新して、2 つのパラメータを受け取れるようにします。

`/app/lib/actions.ts`

```diff ts
  // This is temporary until @types/react-dom is updated
+ export type State = {
+   errors?: {
+     customerId?: string[];
+     amount?: string[];
+     status?: string[];
+   };
+   message?: string | null;
+ };

+ export async function createInvoice(prevState: State, formData: FormData) {
    // ...
  }
```

- `formData` - 以前と同じです
- `prevState` - `useFormState` フックから渡された state を含みます。今回の例のアクションでは使用しませんが、必須のプロップです

次に、Zod の `parse()` 関数を `safeParse()` に変更します。

`/app/lib/actions.ts`

```diff ts
  export async function createInvoice(prevState: State, formData: FormData) {
    // Validate form fields using Zod
+   const validatedFields = CreateInvoice.safeParse({
      customerId: formData.get('customerId'),
      amount: formData.get('amount'),
      status: formData.get('status'),
    });

    // ...
  }
```

`safeParse()` は、`success` または `error` フィールドのいずれかを含むオブジェクトを返します。これにより、このロジックを `try/catch` ブロック内に記述することなく、バリデーションをより適切に処理できるようになります。

データベースに情報を送信する前に、フォームフィールドが正しくバリデートされたかどうかを条件付きでチェックします。

`/app/lib/actions.ts`

```diff ts
  export async function createInvoice(prevState: State, formData: FormData) {
    // Validate form fields using Zod
    const validatedFields = CreateInvoice.safeParse({
      customerId: formData.get('customerId'),
      amount: formData.get('amount'),
      status: formData.get('status'),
    });

    // If form validation fails, return errors early. Otherwise, continue.
+   if (!validatedFields.success) {
+     return {
+       errors: validatedFields.error.flatten().fieldErrors,
+       message: 'Missing Fields. Failed to Create Invoice.',
+     };
+   }

    // ...
  }
```

`validatedFields` が成功しない場合は、Zod からのエラー メッセージとともに関数を早期に返します。

> [!tip]
>
> console.log `validatedFields` を実行し、空のフォームを送信してその形状を確認します。

最後に、try/catch ブロックの外側でフォームのバリデーションを個別に処理しているため、データベースエラーに対して特定のメッセージを返すことができます。最終的なコードは次のようになります。

`/app/lib/actions.ts`

```ts
export async function createInvoice(prevState: State, formData: FormData) {
  // Validate form using Zod
  const validatedFields = CreateInvoice.safeParse({
    customerId: formData.get("customerId"),
    amount: formData.get("amount"),
    status: formData.get("status"),
  });

  // If form validation fails, return errors early. Otherwise, continue.
  if (!validatedFields.success) {
    return {
      errors: validatedFields.error.flatten().fieldErrors,
      message: "Missing Fields. Failed to Create Invoice.",
    };
  }

  // Prepare data for insertion into the database
  const { customerId, amount, status } = validatedFields.data;
  const amountInCents = amount * 100;
  const date = new Date().toISOString().split("T")[0];

  // Insert data into the database
  try {
    await sql`
      INSERT INTO invoices (customer_id, amount, status, date)
      VALUES (${customerId}, ${amountInCents}, ${status}, ${date})
    `;
  } catch (error) {
    // If a database error occurs, return a more specific error.
    return {
      message: "Database Error: Failed to Create Invoice.",
    };
  }

  // Revalidate the cache for the invoices page and redirect the user.
  revalidatePath("/dashboard/invoices");
  redirect("/dashboard/invoices");
}
```

それでは、フォームコンポーネントにエラーを表示してみましょう。`create-form.tsx` コンポーネントに戻って、フォーム `state` を使用してエラーにアクセスできます。

特定のエラーをそれぞれチェックする **三項演算子** を追加します。たとえば、customer のフィールドの後に次のように追加します。

`/app/ui/invoices/create-form.tsx`

```diff tsx
  <form action={dispatch}>
    <div className="rounded-md bg-gray-50 p-4 md:p-6">
      {/* Customer Name */}
      <div className="mb-4">
        <label htmlFor="customer" className="mb-2 block text-sm font-medium">
          Choose customer
        </label>
        <div className="relative">
          <select
            id="customer"
            name="customerId"
            className="peer block w-full rounded-md border border-gray-200 py-2 pl-10 text-sm outline-2 placeholder:text-gray-500"
            defaultValue=""
+           aria-describedby="customer-error"
          >
            <option value="" disabled>
              Select a customer
            </option>
            {customers.map((name) => (
              <option key={name.id} value={name.id}>
                {name.name}
              </option>
            ))}
          </select>
          <UserCircleIcon className="pointer-events-none absolute left-3 top-1/2 h-[18px] w-[18px] -translate-y-1/2 text-gray-500" />
        </div>
+       <div id="customer-error" aria-live="polite" aria-atomic="true">
+         {state.errors?.customerId &&
+           state.errors.customerId.map((error: string) => (
+             <p className="mt-2 text-sm text-red-500" key={error}>
+               {error}
+             </p>
+           ))}
+       </div>
      </div>
      // ...
    </div>
  </form>
```

> [!tip]
>
> コンポーネントの内部で console.log `state` を実行し、すべてが正しく接続されているかどうかを確認できます。フォームがクライアントコンポーネントになっているので、開発ツールのコンソールをチェックしてください。

上記のコードでは、次の aria ラベルも追加しています。

- `aria-describedby="customer-error"` : これにより、`select` 要素とエラーメッセージコンテナの関係を確立します。これは、`id="customer-error"` のコンテナが `select` 要素を記述していることを示します。ユーザーが `select` ボックスを操作してエラーを通知すると、スクリーンリーダーがこの説明を読み上げます。
- `id="customer-error"` : この `id` 属性は、`select` 入力のエラーメッセージを保持する HTML 要素を一意に識別します。これは、`aria-describedby` が関係を確立するために必要です。
- `aria-live="polite"` : `div` 内のエラーが更新されたとき、スクリーンリーダーはユーザーに丁寧に通知する必要があります。コンテンツが変更されたとき（例えば、ユーザーがエラーを修正したとき）、スクリーンリーダーはこれらの変更をアナウンスしますが、ユーザーの邪魔にならないように、ユーザーがアイドル状態のときにのみアナウンスします。

## 練習：aria ラベルを追加する

上記の例を使用して、残りのフォームフィールドにエラーを追加します。フィールドが不足している場合は、フォームの下部にメッセージを表示する必要もあります。UI は次のようになります。

![各フィールドのエラーメッセージを表示する請求書フォームを作成します。](/_images/form-validation-page.avif)

準備ができたら、`npm run lint` を実行して、aria ラベルが正しく使用されているかチェックしてください。

この章で学んだ知識を活用して、`edit-form.tsx` コンポーネントにフォームバリデーションを追加してみましょう。

そのために

- `edit-form.tsx` コンポーネントに `useFormState` を追加します
- `updateInvoice` アクションを編集して、Zod からのバリデーションエラーを処理します
- コンポーネントにエラーを表示し、アクセシビリティを向上させるために aria ラベルを追加します

準備ができたら、以下のコードスニペットを活用してアプリを作成していきましょう。

**請求書フォームの編集**

`/app/ui/invoices/edit-form.tsx`

```tsx
export default function EditInvoiceForm({
  invoice,
  customers,
}: {
  invoice: InvoiceForm;
  customers: CustomerField[];
}) {
  const initialState = { message: null, errors: {} };
  const updateInvoiceWithId = updateInvoice.bind(null, invoice.id);
  const [state, dispatch] = useFormState(updateInvoiceWithId, initialState);

  return <form action={dispatch}></form>;
}
```

**サーバーアクション**

`/app/lib/actions.ts`

```ts
export async function updateInvoice(
  id: string,
  prevState: State,
  formData: FormData
) {
  const validatedFields = UpdateInvoice.safeParse({
    customerId: formData.get("customerId"),
    amount: formData.get("amount"),
    status: formData.get("status"),
  });

  if (!validatedFields.success) {
    return {
      errors: validatedFields.error.flatten().fieldErrors,
      message: "Missing Fields. Failed to Update Invoice.",
    };
  }

  const { customerId, amount, status } = validatedFields.data;
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
