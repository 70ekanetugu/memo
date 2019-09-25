# IMS Security Framework
# 目次  
[1.概要](#1.概要)
[2.用語](#2.用語)
[3.アーキテクチャ](#アーキテクチャ)
[4.Webサービスの保護](#4.Webサービスの保護)
[5.メッセージセキュリティ・署名](#5.メッセージセキュリティ・署名)
[6.キー管理](#6.キー管理)
[7.ベストプラクティス](#7.ベストプラクティス)

# 1. 概要
IMS仕様の採用者がセキュリティアプローチに関して参照するドキュメント。  
全てのIMS仕様で共通するセキュリティフレームワークが記載されている。  

# 2. 用語
##### claim
エンティティに対して渡す情報の一部

##### Consumer
エンドユーザーがプラットフォームを通じてアクセスし得るエンティティ。  
consumerはプラットフォームにサービスを提供したり、反対に使用したりする。

##### ID token  
認証に関するclaimsやそれ以外のclaimsを含んだJWT。

##### Identifier
特定のコンテキスト中エンティティの一意な文字列。

##### Identity
エンティティに関連する属性のセット。

##### Issuer
claimsセットの発行者。情報交換を開始するエンティティであり、Consumerやプラットフォームが成り得る。

##### Message
ConsumerとPlatform間でやり取りされるリクエスト-レスポンスの事。

##### Platform
エンドユーザーが対話し、リモートで起動されたtoolやconsumerにアクセスするためのエンティティ。  
プラットフォームはツールを使用したり、ツールやconsumerにサービスを提供する。

##### Relying Party
OpenId Providerからのclaimsやエンドユーザーの認証を必要とするOAuth2.0クライアントアプリケーション。

##### Subject identifier
ローカル内で一意であり、Issuerで再発行されない識別子。エンドユーザーがConsumerで利用する。

##### Tool
プラットフォームによって起動されるエンティティ。プラットフォームにサービスを提供したり、逆にプラットフォームを使用する。

##### Voluntary Claim
任意のclaim(オプション)。エンドユーザーによって要求された指定タスクで必須では無いが、利便性の為Consumerによって指定されるclaimの事。

##### Client / Client Authentication / Client Identifier and Scope

##### Claim List / Claim Name / Claim Value / 

##### Code Verifier / Code Challenge and Code Challenge Method

##### JOSE Header

##### JWA / JWE / JWK / JWS / JWT
Json Web : アルゴリズム / 暗号 / キー / 署名 / トークン

