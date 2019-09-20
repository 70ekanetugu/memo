# OAuth1.0について
# 目次
[用語](#用語)
[例(RFC5849)](#RFC5849)

※参照(RFC5849)：[https://tools.ietf.org/html/rfc5849#section-2.1](https://tools.ietf.org/html/rfc5849#section-2.1)

# 用語
- client(consumer) : OAuth認証リクエストを投げるHTTPクライアントの事。
- server(Service Provider) : OAuth認証リクエストを受け取るHTTPサーバの事。
- protected resource : OAuth認証要求でサーバから得られる制限されたリソースの事
- credentials : 共有秘密鍵に対応する一意な識別子の事。OAuthでは3つのクレデンシャルがある。
    - client : リクエストを行うclient認証
    - temporary : リクエスト認可
    - token : アクセス許可
- token : サーバーによって発行される一意な識別子。
    - 認証リクエストをリソースと関連づける為にclientで使われる。
    - クライアントがトークン所有権を確立する為に使用する共有秘密鍵を含む。
    - リソースへのアクセス許可証になる。(アクセストークン)
- User : リソースの所有者
- Consumer key and Secret : clientクレデンシャル。
- Request Token and Secret : tempクレデンシャル。
- Access Token and Secret : tokenクレデンシャル。

# 例(RFC5849)
ユーザーは写真をアップロードできるサーバを利用している。  
別の印刷サイトを用いてユーザーは写真を印刷したいとする。
- "user"はブラウザ等のUA。
- "client"は印刷サイト
- "resource_server"は、写真がアップされているサーバ。

[OAuth1.0フロー]
1. ユーザーがclientへ写真印刷のリクエストを投げる。
2. clientが認証処理開始を要求する為に、resource_serverへ一時トークン情報のリクエストを投げる。(事前登録したclient_tokenを付加する)
3. resource_serverでclient_tokenを検証し、問題なければtemp_tokenを返す。(認可処理要求のためのトークン)
4. clientは認可処理を開始する為にtemp_tokenをつけて、userをresource_serverへリダイレクトさせる。
5. resource_serverがtemp_tokenを確認し、問題なければuserへ認可リクエストを投げる。(承認/ログイン画面)
6. ユーザーが認可レスポンスを返す。(承認・サインイン)
7. 認証がOKであれば、tokenについていたcallback_URLへuserをリダイレクトさせる。
8. clientはHttpヘッダの"Authorization"フィールドにtemp_tokenをつけて再度userをresource_serverへリダイレクトさせる。
9. Httpヘッダを検証し、問題なければaccess_tokenを発行し、レスポンスで返す。
10. 以降clientがresource_serverのリソースを使う際は、Authorizationフィールドにaccess_tokenを付加してリクエストする。
11. clientはuserからリクエストされたHttpヘッダにaccess_token有った場合、resource_serverへトークンと一緒にリクエストを投げて、リソース(写真)をもらう。

![](./pic/OAuth1/OAuth1_0.png)

# リダイレクトベース認可
### Temporary Credentials
Clientは、ServerへのHttpリクエスト(POST)によって、一時的な認可トークンを取得する。  
このリクエスト構築の際、Clientは必須パラメータである"oauth_callback"(認可成功時のClientへのリダイレクト先URL)を追加する。  
例は以下の通り。(改行は見やすくするためにやってるだけ。以降同様)
```Http
POST /request_temp_credentials HTTP/1.1
Host: server.example.com
Authorization: OAuth realm="Example",
        oauth_consumer_key="jd83jd92dhsh93js",
        oauth_signature_method="PLAINTEXT",
        oauth_callback="http%3A%2F%2Fclient.example.net%2Fcb%3Fx%3D1",
        oauth_signature="ja893SD9%26"
```
サーバーはこのリクエストを検証(署名)し、問題なければtemp_tokenをレスポンスとして返す。レスポンスには以下のパラメータを含む。
```Http
HTTP/1.1 200 OK
Content-Type: application/x-www-form-urlencoded
     oauth_token=hdk48Djdsa&oauth_token_secret=xyz4992k83j47x0b&
     oauth_callback_confirmed=true
```
### Resource Owner Authorization
Clientはサーバーからtoken_credentials(access_token)の要求をする前に、リクエスト認証のためにサーバーにuserを送る必要がある。  
clientは取得した一時トークンをクエリパラメータとして含んだURIを構築し、リソース所有者認可を行うリクエストをServerへ送る。
```http
GET /authorize_access?oauth_token=hdk48Djdsa HTTP/1.1
Host: server.example.com
```
このリクエストを受け取ったサーバーはトークンを検証して、問題なければ、userに認可リクエスト(ログイン画面みたいな)を投げ、ユーザーは認可レスポンスとして、認証情報(ID/password)をサーバーに投げる。  
サーバーにて、認証が通れば以下のリダイレクトでClientへ認可がOKだったことを通知する。なお、この時点ではまだ一時トークンである。
```http
GET /cb?x=1&oauth_token=hdk48Djdsa&oauth_verifier=473f82d3 HTTP/1.1
     Host: client.example.net
```

### Token Credentials
認可が通った場合、clientはaccess_tokenの要求リクエストをserverへ向けたリダイレクトで行う。  
この時、前のステップで受け取った`oauth_verifier`をAuthorizationヘッダに付加する。
```http
POST /request_token HTTP/1.1
Host: server.example.com
Authorization: OAuth realm="Example",
        oauth_consumer_key="jd83jd92dhsh93js",
        oauth_token="hdk48Djdsa",
        oauth_signature_method="PLAINTEXT",
        oauth_verifier="473f82d3",
        oauth_signature="ja893SD9%26xyz4992k83j47x0b"

```
serverはこれをチェックし、問題なければ以下のレスポンスをユーザーに返す。これでようやくアクセストークンが入手できる。
```http
HTTP/1.1 200 OK
Content-Type: application/x-www-form-urlencoded
     oauth_token=j49ddk933skd9dks&oauth_token_secret=ll399dj47dskfjdk
```

# 認可後リクエスト
### Making Requests
認可後のリクエストは以下のようになる。
```http
POST /request?b5=%3D%253D&a3=a&c%40=&a2=r%20b HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded
Authorization: OAuth realm="Example",
                    oauth_consumer_key="9djdj82h48djs9d2",
                    oauth_token="kkk9d7dh3k39sjv7",
                    oauth_signature_method="HMAC-SHA1",
                    oauth_timestamp="137131201",
                    oauth_nonce="7d8f3e4a",
                    oauth_signature="bYT5CMsGcbgUdFHObYMEfcx6bsw%3D"

```


### Verifying Requests(認可後リクエストの検証)
認可後パラメータを含んだリクエストを受け取ったサーバは、以下の検証を行う必要がある。
- 署名に「HMAC-SHA1」又は「RSA-SHA1」が使用されている場合、nonce/timestamp/tokenの組み合わせが以前に使用されていないことを確認する。
- トークンが存在する場合、トークンに含まれるクライアント認証のスコープとステータスを検証する。(サーバがトークンを使用するクライアントを発行元のクライアントに制限する)
- oauth_version = 1.0　である事をチェック。
以上の検証を行い、サーバは適切なレスポンスを返す。
ex).
- 400 (Bad Request) 
    - サポートしていないパラメータ、署名など
    - パラメーターの欠落、重複など
- 401 (Unauthorized) 
    - 無効または期限切れクレデンシャル
    - 無効な署名など


### Nonce and Timestamp
- Timestamp : 正の整数でなければならない。GMT秒数。
- Nonce : クライアントが一意に生成するランダム文字列。リプライ攻撃を防ぐためにある。

### Signature
OAuthで認証されたリクエストには、2組の資格情報を含めることができる。  
- oauth_cosumer_key
- oauth_token  
これらの資格情報の所有者である事を証明するために、署名を行う。OAuthでは署名方法として「HMAC-SHA1」「RSA-SHA1」、「PLAINTEXT」を提供している。ただ、実装には各自の要件がある為、署名方法を強制してはいない。  
クライアントは"oauth_signature_method"パラメータで使用する署名方式を宣言する。  
"oauth_signature"には生成した署名を入れる。
