# データベースの設定

ダッシュボードでの作業を続ける前に、いくつかのデータが必要です。この章では、`@vercel/postgres` を使用して PostgreSQL データベースの設定をします。すでに PostgreSQL に慣れていて、独自のプロバイダを使用したい場合は、この章をスキップして自分で設定してもかまいません。それ以外の場合は、続けましょう!

この章で取り上げるトピックは以下のとおりです。

- プロジェクトを GitHub にプッシュします
- Vercel アカウントを設定し、GitHub リポジトリをリンクして、プレビューやデプロイを行えるようにします
- プロジェクトを作成し、Postgres データベースにリンクします
- データベースに初期データをシードします

## GitHub リポジトリの作成

まだリポジトリを Github にプッシュしていない場合は、リポジトリを Github にプッシュしましょう。そうすることで、データベースの設定とデプロイが簡単になります。

リポジトリの設定に関するヘルプが必要な場合は、[GitHub のこのガイド](https://help.github.com/en/github/getting-started-with-github/create-a-repo) を参照してください。

> [!TIP]
>
> - GitLab や Bitbucket などの他の Git プロバイダを使用することもできます
> - GitHub を初めて使用する場合は、開発ワークフローを簡素化できる [GitHub デスクトップアプリ](https://desktop.github.com/) をお勧めします

## Vercel アカウントの作成

[vercel.com/signup](https://vercel.com/signup) にアクセスしてアカウントを作成してください。無料の「hobby」プランを選択してください。[**Continue with GitHub**] を選択して、GitHub アカウントと Vercel アカウントを接続します。

## プロジェクトの接続とデプロイ

次に、先ほど作成した GitHub リポジトリを選択してインポートする画面が表示されます。

![ユーザーの GitHub リポジトリのリストを含むプロジェクトのインポート画面を示す Vercel ダッシュボードのスクリーンショット](/_images/import-git-repo.avif)

プロジェクトに名前を付けて、[**Deploy**] をクリックします。

![プロジェクト名フィールドと Deploy ボタンを示すデプロイ画面](/_images/configure-project.avif)

やりました！ 🎉 これでプロジェクトがデプロイされました。

![プロジェクト名、ドメイン、デプロイ状況を表示するプロジェクト概要画面](/_images/deployed-project.avif)

GitHub リポジトリに接続すると、**main** ブランチに変更をプッシュするたびに、Vercel は設定不要で自動的にアプリケーションを再デプロイします。また、プルリクエストを発行する際 [インスタントプレビュー](https://vercel.com/docs/deployments/preview-deployments#preview-urls) も利用できるので、デプロイエラーを早期に発見したり、チームメンバーとプロジェクトのプレビューを共有してフィードバックを得ることができます。

## Postgres データベースの作成

次に、データベースの設定するために **Continue to Dashboard** をクリックし、プロジェクトダッシュボードから **Storage** タブを選択します。**Connect Store → Create New → Postgres → Continue** を選択します。

![Postgres オプションと KV、Blob、および Edge Config を表示する Connect Store 画面](/_images/create-database.avif)

規約に同意し、データベースに名前を割り当て、データベースリージョンが **Washington D.C (iad1)** に設定されていることを確認してください。これは、すべての新しい Vercel プロジェクトの [デフォルトリージョン](https://vercel.com/docs/functions/serverless-functions/regions#select-a-default-serverless-region) でもあります。データベースを同じリージョンまたはアプリケーションコードの近くに配置することで、データリクエストの [レイテンシ（待ち時間）](https://developer.mozilla.org/en-US/docs/Web/Performance/Understanding_latency) を短縮できます。

![データベース名とリージョンを表示するデータベース作成モーダル](/_images/database-region.avif)

> [!TIP]
> データベース領域は、一度初期化されると変更できません。別の [リージョン](https://vercel.com/docs/storage/vercel-postgres/limits#supported-regions) を使用したい場合は、データベースを作成する前に設定する必要があります。

接続が完了したら `.env.local` タブに移動し、`Show secret` と `Copy Snippet` をクリックします。必ず秘密のキーを表示させてから、コピーするようにしてください。

![`.env.local タブ` で、非表示であるデータベースのシークレットキーを表示させる](/_images/database-dashboard.avif)

コードエディタに移動し、`.env.example` ファイルの名前を `.env` に変更します。 Vercel からコピーした内容を貼り付けます。

> [!IMPORTANT]
> GitHub にプッシュする際、データベースの機密情報である接続キーが公開されるのを防ぐため、`.gitignore` ファイルで、無視されるファイルに `.env` が含まれていることを確認してください。

最後に、ターミナルで `npm i @vercel/postgres` を実行して、[Vercel Postgres SDK](https://vercel.com/docs/storage/vercel-postgres/sdk) をインストールします。

## データベースのシード設定

データベースが作成されたので、初期データをシードしてみましょう。これにより、ダッシュボードを構築するときにいくつかのデータを操作できるようになります。

プロジェクトの `/scripts` フォルダに、`seed.js` というファイルがあります。このスクリプトには、`invoices`、`customers`、`user`、`revenue` テーブルの作成とシードの手順が記述されています。

コードの動作をすべて理解していなくても心配する必要はありません。概要を説明すると、スクリプトは **SQL** を使用してテーブルを作成し、テーブル作成後 `placeholder-data.js` ファイルからデータを取得してテーブル内データの設定を行います。

次に、`package.json` ファイルに、以下のようにスクリプトを追加します。

`/package.json`

```js diff
  "scripts": {
    "build": "next build",
    "dev": "next dev",
    "start": "next start",
+   "seed": "node -r dotenv/config ./scripts/seed.js"
  },
```

これは `seed.js` を実行するためのコマンドになります。

ここで、`npm run seed` を実行してみましょう。ターミナルにいくつかの `console.log` メッセージが表示され、スクリプトが実行中であることがわかります。

> [!IMPORTANT]
> データベースのシークレットキーを `.env` ファイルにコピーする前に、必ず公開してご確認ください
> スクリプトは bcrypt を使用してユーザーのパスワードをハッシュします。`bcrypt` が現在の環境と互換性がない場合は、代わりに [bcryptjs](https://www.npmjs.com/package/bcryptjs) を使用するようにスクリプトを更新できます
> データベースのシード中に問題が発生し、スクリプトを再度実行したい場合は、データベースクエリインターフェイスで `DROP TABLE tablename` を実行して既存のテーブルを削除できます。詳細については、以下の [クエリの実行セクション](https://nextjs.org/learn/dashboard-app/setting-up-your-database#executing-queries) を参照してください。ただし、このコマンドはテーブルとそのすべてのデータを削除することに注意してください。プレースホルダデータを操作しているため、サンプルアプリでこれを実行しても問題ありませんが、運用アプリではこのコマンドを実行しないでください。
> Vercel Postgres データベースのシード中に引き続き問題が発生する場合は、[GitHub でのディスカッション](https://github.com/vercel/next-learn/issues) を開いてください。

## データベースの探索

データベースがどのようなものかを見てみましょう。 Vercel に戻り、サイドナビの **Data** をクリックします。

このセクションには、 users、customers、invoices、revenue の 4 つの新しいテーブルがあります。

![users、customers、invoices、revenue の 4 つのテーブルを含むドロップダウンリストを表示するデータベース画面](/_images/database-tables.avif)

各テーブルを選択すると、そのレコードを表示し、エントリが `placeholder-data.js` ファイルのデータと一致していることを確認できます。

## クエリの実行

「query」タブに切り替えるとデータベースを操作できます。このセクションでは、標準の SQL コマンドをサポートします。たとえば、`DROP TABLE customers` を入力すると `customers` テーブルがそのすべてのデータとともに削除されるため、**注意してください**。

最初のデータベースクエリを実行してみましょう。次の SQL コードを Vercel のインターフェイスに貼り付けて実行してください。

```sql
SELECT invoices.amount, customers.name
FROM invoices
JOIN customers ON invoices.customer_id = customers.id
WHERE invoices.amount = 666;
```
