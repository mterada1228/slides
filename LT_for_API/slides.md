---
theme: seriph
background: https://source.unsplash.com/collection/94734566/1920x1080
class: text-center
highlighter: shiki
lineNumbers: false
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
drawings:
  persist: false
---

## Web API 設計のベストプラクティス

---

# Web API 設計のベストプラクティス

<div>
  [Enchant](https://www.enchant.com/) の開発者、

  [Vinay Sahni](https://twitter.com/veesahni) さんが書いた記事。

  2015 年の記事なのでちょっと古めですが、

  明確なベストプラクティスがない API 設計において一度は読んでおきたい素晴らしい記事。
</div>

---

# RESTful な URL にする

<div>

- CRUD 操作

  ```
  GET /tickets # チケットの一覧を取得
  GET /tickets/12 # 指定したチケットを取得
  POST /tickets # チケットを作成
  PUT /tickets/12 # 指定したチケットを更新する
  DELETE /tickets/12 # 指定したチケットを削除する
  ```

- 関連モデルを操作

  ```
  GET /tickets/12/messages
  ```

</div>

<!-- 
リソースに対して HTTP メソッドを使って操作しましょう。
単独のエンドポイントに対して、主要な API を定義することができます。

また、エンドポイントは難しく考えず、一貫して複数形を用いるのが良いです。

関連モデルに対する操作はネストされた URL にすることで実現できます。
しかし、常に関連モデルが付随するような場合は、レスポンスに含めてしまうのもありです。
-->

---

# RESTful な URL にする

<div>

- CRUD 操作にマッチしない操作

  - 例1: リソースに対してアクティベーションさせる操作
    ```
    PUT /gists/:id/star
    DELETE /gists/:id/star
    ```

  - 例2: 複数のリソースを横断するようなアクション
    ```
    GET /serach
    ```

</div>

<!-- 
CRUD 操作にマッチしない場合もあります。
その場合は状況に応じて設計を考える必要があります。

例1はリソースに対してアクティベーションさせる操作です。
この場合は boolean カラムをサブリソースをして捉えて、
URL にネストさせることで実現できます。

例2 では複数のリソースを横断するような操作になります。

これは REST ful の原則から逸脱するので、
API 仕様書などで、利用者に対して必ずフォローすることが大事です。
-->

---

# SSL 通信を用いる

<div>

- NG

  ```
  http://hoge
  ```

- OK

  ```
  https://hoge
  ```

</div>

<!-- 
公衆 Wifi などは必ずしもセキュアでないので、
非SSL通信を使うと簡単になりすましができます。
-->

---

# API 仕様書を作る

<div>

### 著名な API の仕様書

<br />

- [Github API](http://developer.github.com/v3/gists/#list-gists)
- [Stripe](https://stripe.com/docs/api)

<br />

### 良い API 仕様書のポイント

<br />

- 誰でもアクセスできるところにある。
- リクエストとレスポンスの例が記されている。
- リクエストはコピーできるようになっている。 
- API の破壊的な変更のスケジュールなどをお知らせできる。

</div>

---

# バージョンは URL に含める

- NG
  - バージョン指定をヘッダに含める
    ```
    # endpoint
    /users

    # Request Header
    Accept: application/json; version=v1
    ```

- OK
  - バージョン指定を URL に含める
    ```
    # endpoint
    /v1/users
    ```

---

# リクエストパラメータを活用する

### フィルタ

<br />

```
GET /tickets?state=open
```

<br />

### ソート

<br />

```
GET /tickets?sort=-priority,created_at
```

<br />

### 検索

<br />

```
GET /tickets?q=return
```

<br />

### よく使うパラメータがあるならエイリアスを作る

<br />

```
GET /tickets/recently_closed
```

<!-- 
URL はシンプルにしたいので、フィルタ、ソート、
検索などはパラメータでできるようにしましょう。

まず、フィルタリングはこの例だと、 
tickets のうち、state が open のものだけを抽出しています。

このように項目と値を設定するようにします。

ソートは sort パラメータに値を受け取る形で設定します。
例のように昇順/降順をネガ/ポジで指定できると便利です。
またカンマ区切りで、ソートキーを複数指定できるとなお良いです。

検索はqパラメータに文字列を渡すのがいいです。
全文検索サーバの多くがこの形式を採用しているので、
API から検索サーバにそのまま渡すことができます。

最後によく使用される条件はエイリアスにして、パスを用意してしまうのも手です。
-->

---

# レスポンスのフィールドを絞れるようにする

```
GET /tickets

# Got response
{
  id: 1,
  subject: "Sample Title",
  coutomer_name: "John Doe",
  created_at: "2022-04-29",
  updated_at: "2022-04-30",
  ...
}

GET /tickets?fields=id,subject

# Got resoponse
{
  id: 1,
  subject: "Sample Title"
}
```

<!--
API 利用者は常にリソースの全項目を必要としていません。
利用者のネットワーク負荷を下げるためにもフィールドの絞り込み機能は用意しましょう。
-->

---

# 作成/更新後はリソースをフルで返す

```
POST /tickets

# Got response
{
  id: 2,
  subject: "Created ticket",
  created_at: "2022-04-30",
  updated_at: "2022-04-30
}
```

<!--
auto increment された id や作成日、更新日の情報はサーバサイドで作成され、
API の利用者は知る由がありません。

レスポンスにリソースの情報がなければ、
利用者は再度 GET の API を叩くことになります。

これを避けるためにも作成/更新後のレスポンスにはリソースの情報をフルで返しましょう。
-->

---

# フィールドの命名規則を考える

### API は **スネークケース** を使う。

<!--
JavaScript の流儀に従えばキャメルケースとなる気もしますが、
API のスタンダードはスネークケースです。

研究によるとキャメルケースよりもスネークケースの方が、
20% も視認性が高いようです。
-->

---

# JSON はデフォルトで整形する

<div>

- NG
  ```
  {id:1,subject:"Sample Title",coutomer_name:"John Doe",created_at:"2022-04-29",updated_at:"2022-04-30"}
  ```

- OK
  ```
  {
    id: 1,
    subject: "Sample Title",
    coutomer_name: "John Doe",
    created_at: "2022-04-29",
    updated_at: "2022-04-30"
  }
  ```

</div>

<!--
データ転送量の問題で余分なデータを省く考えもありますが、
空白の有無でそこまで変化はありません。

データ転送量がネックになるような状況があったら、
レスポンスを gzip 圧縮するなどの方針を取りましょう。
-->

---

# 要素はラップしない

<div>

- NG
  ```
  {
    "data" : {
      "id" : 123,
      "name" : "John"
    }
  }
  ```

- OK
  ```
  {
    "id" : 123,
    "name" : "John"
  }
  ```

</div>

<!-- 
要素をなぜラップするか？
それはレスポンスにメタデータなどを埋め込みたいからです。

一昔前には JSONP というクロスドメイン制約を解消する仕組みが一般的で、
この JSONP がヘッダ情報にアクセスできないため、
レスポンスのメタデータを参照していました。

今は JSONP は完全に CORS にその立場を奪われました。

レスポンスにメタデータを埋め込まなければならない状況はそうありません。
-->

---

# 追加/更新のリクエストボディも JSON を使う

<div>

- NG
  ```
  POST /tickets
  Content-Type: application/x-www-form-urlencoded
  subject=sample&customer_name=john
  ```

- OK
  ```
  POST /tickets
  Content-Type: application/json
  {
    subject: "sample",
    customer_name: "john"
  }
  ```

</div>

<!--
JSON を用いることで、データ型の概念を取り入れられます。
なのでサーバサイドで、文字列から、数値、Boolean などに変換する必要がありません。
-->

---

# ページング情報はレスポンスヘッダに入れる

<div>

- NG
  ```
  {
    data: {
      tickets: [
        {
          id: 1,
          subject: "sample"
        }
      ]
    },
    page: 2,
    per_page: 100,
    total_count: 5000
  }
  ```

</div>

---

# ページング情報はレスポンスヘッダに入れよう

<div>

- OK
  ```
  Link: <https://tickets?page=3&per_page=100>; rel="next", <https://tickets?page=50&per_page=100>; rel="last"
  {
    tickets: [
      {
        id: 1,
        subject: "sample"
      }
    ]
  }
  ```

</div>

<!--
ページング情報をレスポンスに含めると、ラップする必要が出てきます。
これを避けるためにもページング情報はヘッダに含めましょう。

またヘッダにはすぐ叩ける状態の URL を含めることで、
API 利用者が自分で URL を組み立てる必要がなくなります。

保守性を高めるためにもこのアプローチは有効です。
-->

---

# 関連データを埋め込む手段を作る

```
GET /tickets/12?enbed=customer.name,assign_user

# Got response
{
  id: 12,
  subject: "sample",
  customer: {
    name: "John"
  },
  assign_user: {
    id: 1,
    name: "Jane",
    ...
  }
}
```

<!--
関連データを取得するために何度も API を叩かせるのは良いとは言えません。
例のように embed パラメータを使って関連データを埋め込む手段を作りましょう。

embed パラメータはドット記法を使って、
field の絞り込みができるようになっていると良いです。

関連データを埋め込む際には、
N+1 問題には注意しましょう。
-->

---

# HTTP メソッドを上書き可能にする

<div>

- 更新
  ```
  POST /tickets/12
  X-HTTP-Method-Override: PUT
  ```

- 削除
  ```
  POST /tickets/12
  X-HTTP-Method-Override: DELETE
  ```

</div>

<!-- 
一部の HTTP クライアントは GET と POST しか扱えません。
X-HTTP-Method-Override でメソッドを上書きできるようにしておきましょう。
(Rails では標準で可能です。)
-->

---

# リクエスト制限情報をレスポンスヘッダに入れる

<div>

- Twitter の API で採用されている制限情報

  ```
  X-Rate-Limit-Limit: 100 # 一定期間内にリクエストできる回数
  X-Rate-Limit-Remaining: 99 # 次の期間までにリクエストできる残り回数
  X-Rate-Limit-Remaining: 600 # 次の期間が来るまでの秒数
  ```

<!--
サーバサイドの負荷を避けるために、リクエストの制限をかけるのは必須です。
その時 API 利用者がリクエスト制限をされたら何をすれば良いかわかるように、
リクエスト制限情報をレスポンスヘッダに含めましょう。
-->

</div>

---

# トークン認証を使う


<div>

- Basic 認証
  ```
  Authorization: Basic dXNlcjpwYXNzd29yZA==
  ```

</div>

<!--
RESTful API はステートレスなので、セッション、cookie を使うことはできません。
なので、認証にはトークン認証を用いる必要があります。

Basic 認証は最も簡単に実装できるトークン認証の方式ですが、
トークンを平文で送る必要があるため、SSL通信を利用して暗号化することが必須です。
-->

---

# キャッシュの情報をレスポンスヘッダに入れる

<div>

レスポンスヘッダにリソース情報を変換したものを含めることで変更を検知する。

- ETag を用いる方法
  - リソースのハッシュか、チェックサムを含める
    ```
    ETag: "qa3311fa"
    ```

- Last-Modified を用いる方法
  - リソースの最終更新日時を含める
    ```
    Last-Modified: Sat, 30 Apr 2022 03:21:05 GMT
    ```

<!--
API レスポンスはキャッシュすることで高速になります。

変化していないリソースを毎回取得するのではなく、
変化があったときのみサーバから取得するようにします。

このリソースの状態をレスポンスヘッダに保持します。

代表的なものに、ETag と Last-Modified があり、
リソースのハッシュを含めるか、更新日時を含めるかといった違いがあります。
-->

</div>

---

# エラーメッセージはちゃんと返す

<div>

- NG
  ```
  HTTP/1.1 422 Unprocessable entry
  {}
  ```

</div>

---

# エラーメッセージはちゃんと返す

<div>

- OK
  ```
  HTTP/1.1 422 Unprocessable entry
  {
    "code" : 1024,
    "message" : "Validation Failed",
    "errors" : [
      {
        "code" : 5432,
        "field" : "first_name",
        "message" : "First name cannot have fancy characters"
      }
    ]
  }
  ```

- 良いエラーメッセージのポイント
  - ユニークなエラーコード
  - 原因の説明
  - (バリデーションエラーであれば)問題のあるフィールドとその理由

</div>

<!--
Web サイトのエラーページのように、
API でも有益なメッセージをちゃんと返すようにしましょう。

エラーメッセージには、
ユニークなエラーコード、原因の説明があるといいです。

また、バリデーションエラーであれば、
さらに、問題のあるフィールドと、その原因が含まれている良いです。
-->

---

# HTTP ステータスコードを活用する

<div>

- HTTP レスポンスステータスコード
  - https://developer.mozilla.org/ja/docs/Web/HTTP/Status

</div>

<!-- 
200, 400, 500 といった大雑把なステータスコードだけでなく、
目的に応じた適切なステータスコードを返しましょう。

このことによりリクエストの結果、
API 使用者が次に何をすべきか明確に分かります。
-->

---

# 実際にやってみた

<div>

- Github
  - https://github.com/Horse-race-common-info-api/api

</div>