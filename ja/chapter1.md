# はじめに

## 新規プロジェクトの作成

Next.js アプリを作成するには、ターミナルを開き、プロジェクトを保存したいフォルダに [cd](https://developer.mozilla.org/en-US/docs/Learn/Tools_and_testing/Understanding_client-side_tools/Command_line#basic_built-in_terminal_commands) で移動し、以下のコマンドを実行します。

```shell
npx create-next-app@latest nextjs-dashboard --use-npm --example "https://github.com/vercel/next-learn/tree/main/dashboard/starter-example"
```

このコマンドは、Next.js アプリケーションをセットアップするコマンドラインインターフェイス (CLI) ツールである `create-next-app` を使用しています。上記のコマンドでは、このコースの [スターターサンプル](https://github.com/vercel/next-learn/tree/main/dashboard/starter-example) を使用するため --example フラグを付加しています。

## プロジェクトの探索

コードを最初から作成するチュートリアルとは異なり、このコースのコードの多くはすでに作成されています。これは、既存のコードベースを使用して作業することになる実際の開発をよりよく反映しています。

私たちの目標は、すべてのアプリケーションコードを記述することなく、Next.js の主な機能の学習に集中できるようにすることです。

インストール後、コードエディターでプロジェクトを開き、`nextjs-dashboard` に移動します。

```shell
cd nextjs-dashboard
```

時間をかけてプロジェクトを探索してみましょう。

### フォルダ構造

このプロジェクトは以下のようなフォルダ構造になっていることがわかります。

![ダッシュボード プロジェクトのフォルダ構造。主要なフォルダとファイル: アプリ、パブリック、構成ファイルが表示されます。]()

* `/app`: アプリケーションのすべてのルート、コンポーネント、ロジックが含まれています。主にここから作業します。
* `/app/lib`: 再利用可能なユーティリティ関数やデータ取得関数など、アプリケーションで使用される関数が含まれています。
* `/app/ui`: カード、テーブル、フォームなど、アプリケーションのすべての UI コンポーネントが含まれます。時間を節約するために、これらのコンポーネントは事前にスタイル設定されています。
* `/public`: 画像など、アプリケーションのすべての静的アセットが含まれます。
* `/scripts`: 後の章で使用する、データベースにデータを取り込むためのシードスクリプトが含まれています。
* `Config Files`: アプリケーションのルートに `next.config.js` などの構成ファイルがあることにも気づくでしょう。これらのファイルのほとんどは `create-next-app` を使用して新しいプロジェクトを開始するときに作成され、あらかじめ設定されています。このコースでは、これらのファイルを変更する必要はありません。

これらのフォルダを自由に探索してください。コードの動作をすべて理解していなくても心配する必要はありません。

### プレースホルダデータ

ユーザーインターフェイスを構築するとき、いくつかのプレースホルダデータがあると便利です。データベースや API がまだ利用できない場合は、以下のような方法があります。

* プレースホルダデータは JSON 形式または JavaScript オブジェクトとして使用します。
* [mockAPI](https://mockapi.io/) などのサードパーティサービスを使用します。

このプロジェクトでは、いくつかのプレースホルダデータを `app/lib/placeholder-data.js` に用意しました。ファイル内の各 JavaScript オブジェクトは、データベース内のテーブルを表しています。たとえば、invoices テーブルの場合は以下のようになります。

```js /app/lib/placeholder-data.js
const invoices = [
  {
    customer_id: customers[0].id,
    amount: 15795,
    status: 'pending',
    date: '2022-12-06',
  },
  {
    customer_id: customers[1].id,
    amount: 20348,
    status: 'pending',
    date: '2022-11-14',
  },
  // ...
];
```

データベースのセットアップに関する章では、このデータを使用してデータベースをシード (データベースに初期データを入力) します。

### TypeScript

また、ほとんどのファイルには .ts または .tsx という接尾辞が付いていることに気づくかもしれません。これは、プロジェクトが TypeScript で記述されているためです。私たちは、現代の Web 環境を反映したコースを作成したいと考えました。

TypeScript を知らなくても大丈夫です。必要に応じて TypeScript コードスニペットを提供します。

`/app/lib/settings.ts` ファイルを見てください。ここでは、データベースから返される型を手動で定義します。たとえば、invoices テーブルには以下の型があります。

```ts /app/lib/definitions.ts
export type Invoice = {
  id: string;
  customer_id: string;
  amount: number;
  date: string;
  // In TypeScript, this is called a string union type.
  // It means that the "status" property can only be one of the two strings: 'pending' or 'paid'.
  status: 'pending' | 'paid';
};
```

TypeScript を使用することで、コンポーネントやデータベースに間違ったデータ形式を誤って渡さないようにすることができます。たとえば、請求書の `amount` に `number` ではなく `string` を渡すような場合です。

> TypeScript 開発者の場合:
> * データ型を手動で宣言していますが、型の安全性を高めるために、データベーススキーマに基づいて型を自動的に生成する Prisma をお勧めします。
> * Next.js は、プロジェクトで TypeScript が使用されているかどうかを検出し、必要なパッケージと構成を自動的にインストールします。Next.js には、自動補完（オートコンプリート）と型安全性（タイプセーフティ）を支援する、コードエディタ用の TypeScript プラグインも付属しています。

## 開発サーバーの実行

`npm i` を実行してプロジェクトのパッケージをインストールします。

```shell
npm i
```

続いて `npm run dev` を実行して開発サーバーを起動します。

```shell
npm run dev
```

`npm run dev` は、Next.js の開発サーバーをポート 3000 で起動します。動作確認をしてみましょう。ブラウザで [http://localhost:3000](http://localhost:3000) を開いてください。トップページは以下のようになっているはずです。

![Unstyled page with the title 'Acme', a description, and login link.]()