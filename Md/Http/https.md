# Https
# 目次

# SSL設定
## 自己証明書作成
1. opensslをDL
2. 秘密鍵作成
   - `$openssl genrsa -aes128 2048 > private.pem`
   - ↑はPEM形式の場合。他にも色々な形式がある。
   - パスフレーズはあったほうがいい。SSH用鍵とする場合は無いとダメだった気がする。
3. CSR(公開鍵+申請者情報)の作成
   - `$openssl req -new -key server.pem > server.csr`
   - 組織名やサーバアドレスなどを聞かれるので答える。※Common Name(FQDN)は、IPアドレスではダメ。
   - パスフレーズも必須？
4. CRT(ディジタル証明書)の作成
   - `$openssl x509 -in server.csr -days 365000 -req -signkey server.pem > server.crt `
   - 公開鍵が正しい事を証明するデータ。本来はCAが発行してくれるもの。

## 