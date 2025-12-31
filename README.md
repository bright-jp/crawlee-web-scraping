# CrawleeによるWebスクレイピング

[![Promo](https://github.com/bright-jp/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.jp/) 

効率的な [web scraping with Node.js](https://brightdata.jp/blog/how-tos/web-scraping-with-node-js) のためにCrawleeを使用する方法を学びます。

- [Crawleeによる基本的なWebスクレイピング](#basic-web-scraping-with-crawlee)
- [Crawleeによるプロキシローテーション](#proxy-rotation-with-crawlee)
- [Crawleeによるセッション管理](#sessions-management-with-crawlee)
- [Crawleeによる動的コンテンツの処理](#dynamic-content-handling-with-crawlee)

## 前提条件

開始する前に、以下の前提条件がインストールされていることを確認してください。

* **[Node.js](https://nodejs.org/)。**
* **[npm](https://www.npmjs.com/):** これは通常Node.jsに同梱されています。ターミナルで `node -v` または `npm -v` を実行してインストールを確認できます。
* **任意のコードエディタ:** このチュートリアルでは [Visual Studio Code](https://code.visualstudio.com/) を使用します。

## Basic Web Scraping with Crawlee

まずは [Books to Scrape](https://books.toscrape.com/) のWebサイトをスクレイピングしてみましょう。

ターミナルまたはシェルを開き、Node.jsプロジェクトを初期化します。

```bash
mkdir crawlee-tutorial
cd crawlee-tutorial
npm init -y
```

Crawleeライブラリをインストールします。

```bash
npm install crawlee
```

データを効果的にスクレイピングするには、対象WebサイトのHTML構造を確認します。ブラウザでサイトを開き、ページ上の任意の場所を右クリックし、**Developer Tools** で **Inspect** または **Inspect Element** を選択します。

![Inspect HTML element](https://github.com/bright-jp/crawlee-web-scraping/blob/main/images/Inspect-HTML-element-1024x540.png)

**Developer Tools** の **Elements** タブには、ページのHTMLレイアウトが表示されます。この例では次のとおりです。  

- 各書籍は、class が `product_pod` の `article` タグ内にあります。  
- 書籍タイトルは `h3` タグ内にあり、実際のタイトルはネストされた `a` タグの `title` 属性に格納されています。  
- 書籍価格は、class が `price_color` の `p` タグ内にあります。  

![Inspect the HTML elements on the Books to Scrape website](https://github.com/bright-jp/crawlee-web-scraping/blob/main/images/Inspect-the-HTML-elements-on-the-Books-to-Scrape-website-1024x522.png)

プロジェクトのルートディレクトリ配下に `scrape.js` という名前のファイルを作成し、以下のコードを追加します。

```js
const { CheerioCrawler } = require('crawlee');

const crawler = new CheerioCrawler({
    async requestHandler({ request, $ }) {
        const books = [];
        $('article.product_pod').each((index, element) => {
            const title = $(element).find('h3 a').attr('title');
            const price = $(element).find('.price_color').text();
            books.push({ title, price });
        });
        console.log(books);
    },
});

crawler.run(['https://books.toscrape.com/']);
```

このコードは `crawlee` の `CheerioCrawler` を使用して、`https://books.toscrape.com/` から書籍タイトルと価格を抽出します。HTMLを取得し、jQueryライクな構文で `<article class="product_pod">` 要素を選択し、結果をコンソールに出力します。

`scrape.js` ファイルにコードを追加したら、次のコマンドで実行します。

```bash
node scrape.js
```

書籍タイトルと価格の配列がターミナルに表示されるはずです。

```
…output omitted…
  {
    title: 'The Boys in the Boat: Nine Americans and Their Epic Quest for Gold at the 1936 Berlin Olympics',
    price: '£22.60'
  },
  { title: 'The Black Maria', price: '£52.15' },
  {
    title: 'Starving Hearts (Triangular Trade Trilogy, #1)',
    price: '£13.99'
  },
  { title: "Shakespeare's Sonnets", price: '£20.66' },
  { title: 'Set Me Free', price: '£17.46' },
  {
    title: "Scott Pilgrim's Precious Little Life (Scott Pilgrim #1)",
    price: '£52.29'
  },
…output omitted…
```

## Proxy Rotation with Crawlee

プロキシは、あなたのコンピュータとインターネットの間に入る仲介役として機能し、IPアドレスを隠しながらWebリクエストを転送します。これにより、レート制限やIPアドレスのBANを防ぐのに役立ちます。

Crawleeは、リトライ、エラー、ローテーティングプロキシを組み込みで扱えるため、プロキシ実装を簡単にします。

次に、プロキシを設定し、有効なプロキシアドレスを取得し、リクエストがそれを経由していることを確認します。

無料プロキシは遅く、安全性が低く、機密性の高いWebタスクでは信頼性に欠けることが多いため、セキュアで安定した信頼性の高いプロキシを提供するBright Dataのような信頼できるサービスの利用を検討してください。また、無料トライアルも提供されているため、契約前にサービスをテストできます。 

Bright Dataを利用するには、[home page](https://brightdata.jp/) の **Start free trial** ボタンをクリックし、必要事項を入力してアカウントを作成します。

アカウント作成後、Bright Dataのダッシュボードにログインし、**Proxies & Scraping Infrastructure** に移動して、**[Residential Proxies](/proxy-types/residential-proxies)** を選択して新しいプロキシを追加します。

![Add a residential proxy](https://github.com/bright-jp/crawlee-web-scraping/blob/main/images/Add-a-residential-proxy-1024x574.png)

デフォルト設定を維持したまま、**Add** をクリックしてレジデンシャルプロキシの作成を完了します。

証明書のインストールを求められた場合は、**Proceed without certificate** を選択できます。ただし、本番環境および実運用のユースケースでは、プロキシ情報が漏えいした場合の不正利用を防ぐために証明書を設定するべきです。

作成後、ホスト、ポート、ユーザー名、パスワードを含むプロキシ認証情報を控えてください。次のステップで必要になります。

![Bright Data proxy credentials](https://github.com/bright-jp/crawlee-web-scraping/blob/main/images/Bright-Data-proxy-credentials-1024x557.png)

プロジェクトのルートディレクトリ配下で、次のコマンドを実行して [axios](https://www.npmjs.com/package/axios) ライブラリをインストールします。

```bash
npm install axios
```

`axios` ライブラリは `http://lumtest.com/myip.json` にGETリクエストを送るために使用され、使用中のプロキシに関する詳細を返します。

これを実装するには、プロジェクトのルートディレクトリに `scrapeWithProxy.js` という名前のファイルを作成し、以下のコードを追加します。

```js
const { CheerioCrawler } = require("crawlee");
const { ProxyConfiguration } = require("crawlee");
const axios = require("axios");

const proxyConfiguration = new ProxyConfiguration({
  proxyUrls: ["http://USERNAME:PASSWORD@HOST:PORT"],
});

const crawler = new CheerioCrawler({
  proxyConfiguration,
  async requestHandler({ request, $, response, proxies }) {
    // Make a GET request to the proxy information URL
    try {
      const proxyInfo = await axios.get("http://lumtest.com/myip.json", {
        proxy: {
          host: "HOST",
          port: PORT,
          auth: {
            username: "USERNAME",
            password: "PASSWORD",
          },
        },
      });
      console.log("Proxy Information:", proxyInfo.data);
    } catch (error) {
      console.error("Error fetching proxy information:", error.message);
    }

    const books = [];
    $("article.product_pod").each((index, element) => {
      const title = $(element).find("h3 a").attr("title");
      const price = $(element).find(".price_color").text();
      books.push({ title, price });
    });
    console.log(books);
  },
});

crawler.run(["https://books.toscrape.com/"]);
```

> **Note:**
> 
> `HOST`、`PORT`、`USERNAME`、`PASSWORD` は必ずご自身の認証情報に置き換えてください。

このコードは `crawlee` の `CheerioCrawler` を使用し、指定したプロキシを経由して `https://books.toscrape.com/` からデータをスクレイピングします。  

- プロキシは `ProxyConfiguration` を使用して設定されます。  
- `http://lumtest.com/myip.json` へのGETリクエストにより、プロキシ詳細を取得してログに出力します。  
- CheerioのjQueryライクな構文を使用して書籍タイトルと価格を抽出し、コンソールにログ出力します。  

コードを実行してプロキシ設定をテストし、機能していることを確認します。

```bash
node scrapeWithProxy.js
```

以前と同様の結果が表示されますが、今回はリクエストがBright Dataプロキシを経由します。また、コンソールにプロキシの詳細もログ出力されるはずです。

```js
Proxy Information: {
  country: 'US',
  asn: { asnum: 21928, org_name: 'T-MOBILE-AS21928' },
  geo: {
    city: 'El Paso',
    region: 'TX',
    region_name: 'Texas',
    postal_code: '79925',
    latitude: 31.7899,
    longitude: -106.3658,
    tz: 'America/Denver',
    lum_city: 'elpaso',
    lum_region: 'tx'
  }
}
[
  { title: 'A Light in the Attic', price: '£51.77' },
  { title: 'Tipping the Velvet', price: '£53.74' },
  { title: 'Soumission', price: '£50.10' },
  { title: 'Sharp Objects', price: '£47.82' },
  { title: 'Sapiens: A Brief History of Humankind', price: '£54.23' },
  { title: 'The Requiem Red', price: '£22.65' },
…output omitted..
```

`node scrapingWithBrightData.js` でスクリプトを実行すると、毎回異なるIPアドレスが表示されるはずです。これにより、Bright DataがロケーションとIPを自動的にローテーションしていることが確認できます。このローテーションは、Webサイトをスクレイピングする際のブロックやIPアドレスのBANを防ぐのに役立ちます。

> **Note:**
> 
> `proxyConfiguration` では異なるプロキシIPを渡すこともできますが、Bright Dataがそれを実施してくれるため、IPを指定する必要はありません。

## Sessions Management with Crawlee

セッションは、複数のリクエスト間で状態を維持するのに役立ちます。特にCookieやログインセッションを使用するサイトで有効です。  

セッション管理を実装するには、プロジェクトのルートディレクトリに `scrapeWithSessions.js` という名前のファイルを作成し、以下のコードを追加します。

```js
const { CheerioCrawler, SessionPool } = require("crawlee");

(async () => {
  // Open a session pool
  const sessionPool = await SessionPool.open();

  // Ensure there is a session in the pool
  let session = await sessionPool.getSession();
  if (!session) {
    session = await sessionPool.createSession();
  }

  const crawler = new CheerioCrawler({
    useSessionPool: true, // Enable session pool
    async requestHandler({ request, $, response, session }) {
      // Log the session information
      console.log(`Using session: ${session.id}`);

      // Extract book data and log it (for demonstration)
      const books = [];
      $("article.product_pod").each((index, element) => {
        const title = $(element).find("h3 a").attr("title");
        const price = $(element).find(".price_color").text();
        books.push({ title, price });
      });
      console.log(books);
    },
  });

  // First run
  await crawler.run(["https://books.toscrape.com/"]);
  console.log("First run completed.");

  // Second run
  await crawler.run(["https://books.toscrape.com/"]);
  console.log("Second run completed.");
})();
```

このコードは `crawlee` の `CheerioCrawler` と `SessionPool` を使用して、`https://books.toscrape.com/` からデータをスクレイピングします。  

- セッションプールを初期化し、クローラーに割り当てます。  
- `requestHandler` はセッションの詳細をログ出力し、Cheerioセレクタで書籍タイトルと価格を抽出します。  
- スクリプトは連続して2回スクレイピングを実行し、そのたびにセッションIDをログ出力します。  

コードを実行して、異なるセッションが使用されていることを確認してください。

```bash
node scrapeWithSessions.js
```

以前と同様の結果が表示されますが、今回は各実行でセッションIDも表示されます。

```
Using session: session_GmKuZ2TnVX
[
  { title: 'A Light in the Attic', price: '£51.77' },
  { title: 'Tipping the Velvet', price: '£53.74' },
…output omitted…
Using session: session_lNRxE89hXu
[
  { title: 'A Light in the Attic', price: '£51.77' },
  { title: 'Tipping the Velvet', price: '£53.74' },
…output omitted…
```

コードをもう一度実行すると、別のセッションIDが使用されていることが確認できるはずです。

## Dynamic Content Handling with Crawlee

**動的Webサイト**（JavaScriptでコンテンツを読み込むサイト）をスクレイピングするのは、レンダリング後にのみデータが利用可能になるため難しい場合があります。  

これに対応するために、CrawleeはJavaScriptをレンダリングし、人間のようにWebページと対話できるヘッドレスブラウザである [Puppeteer](https://pptr.dev/) と統合されています。  

デモとして、[this YouTube page](https://www.youtube.com/watch?v=wZ6cST5pexo) からコンテンツをスクレイピングします。**スクレイピングの前に、必ずサイトのルールおよび利用規約を確認してください。**  

利用規約を確認したら、プロジェクトのルートディレクトリに `scrapeDynamicContent.js` という名前のファイルを作成し、以下のコードを追加します。

```js
const { PuppeteerCrawler } = require("crawlee");

async function scrapeYouTube() {
  const crawler = new PuppeteerCrawler({
    async requestHandler({ page, request, enqueueLinks, log }) {
      const { url } = request;
      await page.goto(url, { waitUntil: "networkidle2" });

      // Scraping first 10 comments
      const comments = await page.evaluate(() => {
        return Array.from(document.querySelectorAll("#comments #content-text"))
          .slice(0, 10)
          .map((el) => el.innerText);
      });

      log.info(`Comments: ${comments.join("\n")}`);
    },

    launchContext: {
      launchOptions: {
        headless: true,
      },
    },
  });

  // Add the URL of the YouTube video you want to scrape
  await crawler.run(["https://www.youtube.com/watch?v=wZ6cST5pexo"]);
}

scrapeYouTube();
```

次に、以下のコマンドでコードを実行します。

```bash
node scrapeDynamicContent.js
```

このコードはCrawleeの `PuppeteerCrawler` を使用して、YouTube動画のコメントをスクレイピングします。  

- クローラーは特定のYouTube動画URLへ移動し、ページが完全に読み込まれるのを待ちます。  
- CSSセレクタ `#comments #content-text` を使用して最初の10件のコメントを選択します。  
- 抽出したコメントをコンソールにログ出力します。  

実行すると、選択した動画から最初の10件のコメントが出力されます。

```
INFO  PuppeteerCrawler: Starting the crawler.
INFO  PuppeteerCrawler: Comments: Who are you rooting for?? US Marines or Ex Cons 
Bro Mateo is a beast, no lifting straps, close stance.
ex convict doing the pushups is a monster.
I love how quick this video was, without nonsense talk and long intros or outros
"They Both have combat experience" is wicked 
That military guy doing that deadlift is really no joke.. ...
One lives to fight and the other fights to live.
Finally something that would test the real outcome on which narrative is true on movies
I like the comradery between all of them. Especially on the Bench Press ... Both team members quickly helped out on the spotting to protect from injury. Well done.
I like this style, no youtube funny business. Just straight to the lifts
…output omitted…
```

このチュートリアルで使用したすべてのコードは [GitHub](https://github.com/See4Devs/crawlee-web-scraping) で確認できます。

## Conclusion

Crawleeは、Webスクレイピングプロジェクトの効率と信頼性を向上させるのに役立ちます。プロ仕様のデータ、ツール、プロキシでWebスクレイピングプロジェクトを次のレベルへ引き上げる準備はできていますか。 [ready-to-use datasets](https://brightdata.jp/products/datasets) と [advanced proxy services](https://brightdata.jp/proxy-types) を提供し、データ収集の取り組みを効率化するBright Dataの包括的なWebスクレイピングプラットフォームをご覧ください。

今すぐ登録して、無料トライアルを開始しましょう！