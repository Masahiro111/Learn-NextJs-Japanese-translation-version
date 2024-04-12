# データベースのセットアップ

ダッシュボードでの作業を続ける前に、いくつかのデータが必要です。この章では、`@vercel/postgres` を使用して PostgreSQL データベースをセットアップします。すでに PostgreSQL に慣れていて、独自のプロバイダを使用したい場合は、この章をスキップして自分で設定してもかまいません。それ以外の場合は、続けましょう!

この章で取り上げるトピックは以下のとおりです。

- プロジェクトを GitHub にプッシュします
- Vercel アカウントを設定し、GitHub リポジトリをリンクして、プレビューやデプロイを行えるようにします
- プロジェクトを作成し、Postgres データベースにリンクします
- データベースに初期データをシードします

## GitHub リポジトリの作成

まだリポジトリを Github にプッシュしていない場合は、リポジトリを Github にプッシュしましょう。そうすることで、データベースのセットアップとデプロイが簡単になります。

リポジトリの設定に関するヘルプが必要な場合は、[GitHub のこのガイド](https://help.github.com/en/github/getting-started-with-github/create-a-repo) を参照してください。

> **コラム** :
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

![Project overview screen showing the project name, domain, and deployment status](/_images/deployed-project.avif)

By connecting your GitHub repository, whenever you push changes to your **main** branch, Vercel will automatically redeploy your application with no configuration needed. When opening pull requests, you'll also have [instant previews](https://vercel.com/docs/deployments/preview-deployments#preview-urls) which allow you to catch deployment errors early and share a preview of your project with team members for feedback.

## Create a Postgres database

Next, to set up a database, click **Continue to Dashboard** and select the **Storage** tab from your project dashboard. Select **Connect Store → Create New → Postgres → Continue**.

![Connect Store screen showing the Postgres option along with KV, Blob and Edge Config](/_images/create-database.avif)

Accept the terms, assign a name to your database, and ensure your database region is set to **Washington D.C (iad1)** - this is also the [default region](https://vercel.com/docs/functions/serverless-functions/regions#select-a-default-serverless-region) for all new Vercel projects. By placing your database in the same region or close to your application code, you can reduce [latency](https://developer.mozilla.org/en-US/docs/Web/Performance/Understanding_latency) for data requests.

![Database creation modal showing the database name and region](/_images/database-region.avif)

> **Good to know** :
> You cannot change the database region once it has been initalized. If you wish to use a different [region](https://vercel.com/docs/storage/vercel-postgres/limits#supported-regions), you should set it before creating a database.

Once connected, navigate to the `.env.local` tab, click `Show secret` and `Copy Snippet`. Make sure you reveal the secrets before copying them.

![The .env.local tab showing the hidden database secrets](/_images/database-dashboard.avif)

Navigate to your code editor and rename the `.env.example` file to `.env`. Paste in the copied contents from Vercel.

**Important**: Go to your `.gitignore` file and make sure `.env` is in the ignored files to prevent your database secrets from being exposed when you push to GitHub.

Finally, run `npm i @vercel/postgres` in your terminal to install the [Vercel Postgres SDK](https://vercel.com/docs/storage/vercel-postgres/sdk).

## Seed your database

Now that your database has been created, let's seed it with some initial data. This will allow you to have some data to work with as you build the dashboard.

In the `/scripts` folder of your project, there's a file called `seed.js`. This script contains the instructions for creating and seeding the `invoices`, `customers`, `user`, `revenue` tables.

Don't worry if you don't understand everything the code is doing, but to give you an overview, the script uses **SQL** to create the tables, and the data from `placeholder-data.js` file to populate them after they've been created.

Next, in your `package.json` file, add the following line to your scripts:

`/package.json`

```json diff
  "scripts": {
    "build": "next build",
    "dev": "next dev",
    "start": "next start",
+   "seed": "node -r dotenv/config ./scripts/seed.js"
  },
```

This is the command that will execute `seed.js`.

Now, run `npm run seed`. You should see some `console.log` messages in your terminal to let you know the script is running.

> **Troubleshooting** :
> Make sure to reveal your database secrets before copying it into your `.env` file.
> The script uses bcrypt to hash the user's password, if `bcrypt` isn't compatible with your environment, you can update the script to use [bcryptjs](https://www.npmjs.com/package/bcryptjs) instead.
> If you run into any issues while seeding your database and want to run the script again, you can drop any existing tables by running DROP TABLE tablename in your database query interface. See the [executing queries section](https://nextjs.org/learn/dashboard-app/setting-up-your-database#executing-queries) below for more details. But be careful, this command will delete the tables and all their data. It's ok to do this with your example app since you're working with placeholder data, but you shouldn't run this command in a production app.
> If you continue to experience issues while seeding your Vercel Postgres database, please open a [discussion on GitHub](https://github.com/vercel/next-learn/issues).

## Exploring your database

Let's see what your database looks like. Go back to Vercel, and click Data on the sidenav.

In this section, you'll find the four new tables: users, customers, invoices, and revenue.

![Database screen showing dropdown list with four tables: users, customers, invoices, and revenue](/_images/database-tables.avif)

By selecting each table, you can view its records and ensure the entries align with the data from placeholder-data.js file.

## Executing queries

You can switch to the "query" tab to interact with your database. This section supports standard SQL commands. For instance, inputting DROP TABLE customers will delete "customers" table along with all its data - so be careful!

Let's run your first database query. Paste and run the following SQL code into the Vercel interface:

```sql
SELECT invoices.amount, customers.name
FROM invoices
JOIN customers ON invoices.customer_id = customers.id
WHERE invoices.amount = 666;
```
