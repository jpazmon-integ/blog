---
title: Application Insights API キーの廃止と移行について
date: 2026-04-24 00:00:00
tags:
  - HowTo
---

[更新履歴]  
- 2026/04/24 ブログ公開  


こんにちは、Azure Monitoring & Integration サポート チームの佐々木です。
近日発表された以下の API キーの廃止と移行について Q&A 形式でご紹介させていただきます。

https://azure.microsoft.com/ja-jp/updates?id=transition-to-azure-ad-to-query-data-from-azure-monitor-application-insights-by-31-march-2026

<!-- more -->

# 目次
- [API キーの廃止について](#api-キーの廃止について)
- [Q&A](#qa)
  - [インストルメンテーションキー(InstrumentationKey または ikey)や接続文字列を使っています。移行の必要がありますか?](#インストルメンテーションキーinstrumentationkey-または-ikeyや接続文字列を使っています移行の必要がありますか)
  - [API キーを使用しているか確認するにはどうすればいいですか?](#api-キーを使用しているか確認するにはどうすればいいですか)
  - [API キーから Microsoft Entra ID 認証へ移行するにはどの様にすればいいですか?](#api-キーから-microsoft-entra-id-認証へ移行するにはどの様にすればいいですか)

# API キーの廃止について

以下ページにて Application Insights の API キーの廃止と、Entra ID 認証への移行が案内されています。

https://azure.microsoft.com/ja-jp/updates?id=transition-to-azure-ad-to-query-data-from-azure-monitor-application-insights-by-31-march-2026

サービス正常性やメールにて、過去にも同様の廃止通知を受け取っていたお客様もいるかと思います。

# Q&A

## インストルメンテーションキー(InstrumentationKey または ikey)や接続文字列を使っています。移行の必要がありますか?

ikey(および接続文字列) と 廃止通知が行われた API Key は異なりますので、それぞれ以下の様にご理解いただけますと幸いです。

|区分|用途|今回の記事の対象か|
|:-:|:-:|:-:|
|Instrumentation Key / Connection String|アプリから Application Insights へテレメトリを送信するための識別・取り込み設定|対象ではない|
|Application Insights API key|api.applicationinsights.io などで Application Insights のデータをクエリ取得するための API key|対象|
|Microsoft Entra ID 認証|上記 API key の代替として、クエリ API 呼び出し時に使う認証方式|**API Key を使用している場合の移行先**|

より簡潔にしますと、以下の様に使用方法も異なる別の機能であると言えます。

- ikeyおよび接続文字列 … ログを送信する際に使用されます。
- API keyおよびMicrosoft Entra ID … ログを検索する際に使用されます。

また、これらは使用方法が異なるため、Application Insights へログを送信するアプリケーション(ikeyおよび接続文字列を使用)とは別のアプリケーションで使用されている事があります。

複数のアプリケーションを実装・管理されている場合にはご留意ください。

## API キーを使用しているか確認するにはどうすればいいですか?

Azure ポータル > Application Insights > API アクセス に API キーが表示されるかご確認ください。

- API キーが作成されていない場合の表示例
<img width="761" height="752" alt="image" src="https://github.com/user-attachments/assets/977965f0-00c9-4113-bb0b-881d9a21f7fe" />

API キーが表示される場合には、お客様のアプリケーションが API キーを利用して、ログ検索を行っている可能性がありますので、
お客様がお持ちのアプリケーションソースコードで API キー 認証によるクエリ等が実装されていないかをご確認ください。

コードは開発言語、フレームワーク等により全く異なります。また、Azure および Azure サポートでは Azure VM や App Service/Function App 等に配置された任意のファイルの読み取りができません。

原則として、該当するアプリケーションの有無はお客様にてご確認いただく必要がある事をご理解いただけますと幸いです。

## API キーから Microsoft Entra ID 認証へ移行するにはどの様にすればいいですか?

例えば、API キー による認証から Microsoft Entra ID 認証に移行する場合は以下の様な PowerShell へ置き換えいただく事も可能です。

この様な Connect-AzAccount + Get-AzAccessToken により取得されたトークンによるクエリ実行は Microsoft Entra ID 認証です。

ご利用される場合には冒頭の以下の値を書き換えいただく必要があります。

- $tenantId
- $appId
- $query

```
# Install-Module Az.Accounts -Scope CurrentUser
$tenantId = "<Tenant ID>"
$appId = "<Application Insights の Application ID>" # API Access ブレードの Application ID
$query    = @"
requests
| where timestamp >= ago(12h) //過去12時間のログを探します。
| order by timestamp desc
"@

Connect-AzAccount -Tenant $tenantId
# Application Insights API 用のアクセストークンを取得
$tokenObject = Get-AzAccessToken -ResourceUrl "https://api.applicationinsights.io"
# Az.Accounts のバージョンにより Token が SecureString の場合があるため平文化
if ($tokenObject.Token -is [System.Security.SecureString]) {
   $accessToken = [System.Net.NetworkCredential]::new("", $tokenObject.Token).Password
} else {
   $accessToken = $tokenObject.Token
}
$uri = "https://api.applicationinsights.io/v1/apps/$appId/query"
$headers = @{
   Authorization  = "Bearer $accessToken"
   "Content-Type" = "application/json"
}
$body = @{
   query    = $query
} | ConvertTo-Json -Depth 5
$response = Invoke-RestMethod `
   -Method Post `
   -Uri $uri `
   -Headers $headers `
   -Body $body
# 結果を PowerShell オブジェクトに変換して表示
$table = $response.tables[0]
$columns = $table.columns.name
$results = foreach ($row in $table.rows) {
   $obj = [ordered]@{}
   for ($i = 0; $i -lt $columns.Count; $i++) {
       $obj[$columns[$i]] = $row[$i]
   }
   [pscustomobject]$obj
}
$results | Format-Table -AutoSize
```

このスクリプトの $tenantId と $appId のみを書き換えた実行結果サンプルは以下の様になります。

(保管されているログの内容により表示結果は異なりますので、ご留意ください。)

```
省略
> $results | Format-Table -AutoSize

timestamp                    id               source name                  url                                                                  success resultCode   duration performanceBucket itemType
---------                    --               ------ ----                  ---                                                                  ------- ----------   -------- ----------------- --------
2026-04-24T05:54:56.975431Z  beabe5acf40edd43        GET /                 https://dasasaki-appservice-aienable.azurewebsites.net/              True    200            4.0036 <250ms            request
2026-04-24T05:54:56.975431Z  beabe5acf40edd43        GET /                 https://dasasaki-appservice-aienable.azurewebsites.net/              True    200            4.0036 <250ms            request
2026-04-24T05:54:56.975431Z  beabe5acf40edd43        GET /                 https://dasasaki-appservice-aienable.azurewebsites.net/              True    200            4.0036 <250ms            request
2026-04-24T05:54:00.7867403Z 650e9970e17fd645        GET /                 https://dasasaki-appservice-aienable.azurewebsites.net/              True    200           10.3806 <250ms            request
2026-04-24T05:54:00.7867403Z 650e9970e17fd645        GET /                 https://dasasaki-appservice-aienable.azurewebsites.net/              True    200           10.3806 <250ms            request
2026-04-24T05:54:00.7867403Z 650e9970e17fd645        GET /                 https://dasasaki-appservice-aienable.azurewebsites.net/              True    200           10.3806 <250ms            request
2026-04-24T05:52:16.6158311Z 1ec0c6f4d80ec740        GET /                 http://app-codex-profiler-jpe-20260424-b.azurewebsites.net/          True    200           20.2825 <250ms            request
2026-04-24T05:52:16.6158311Z 1ec0c6f4d80ec740        GET /                 http://app-codex-profiler-jpe-20260424-b.azurewebsites.net/          True    200           20.2825 <250ms            request
2026-04-24T05:52:16.6158311Z 1ec0c6f4d80ec740        GET /                 http://app-codex-profiler-jpe-20260424-b.azurewebsites.net/          True    200           20.2825 <250ms            request
2026-04-24T05:51:49.215188Z  db3281a511fc7a4f        GET /                 https://dasasaki-appservice-aienable.azurewebsites.net/              True    200            6.7112 <250ms            request
2026-04-24T05:51:49.215188Z  db3281a511fc7a4f        GET /                 https://dasasaki-appservice-aienable.azurewebsites.net/              True    200            6.7112 <250ms            request
2026-04-24T05:51:49.215188Z  db3281a511fc7a4f        GET /                 https://dasasaki-appservice-aienable.azurewebsites.net/              True    200            6.7112 <250ms            request
2026-04-24T05:51:47.7188839Z cff6ba3ad9d44a72        GET /                 http://app-codex-profiler-jpe-20260424.azurewebsites.net/            True    200          151.2283 <250ms            request
2026-04-24T05:51:47.7188839Z cff6ba3ad9d44a72        GET /                 http://app-codex-profiler-jpe-20260424.azurewebsites.net/            True    200          151.2283 <250ms            request
2026-04-24T05:51:47.7188839Z cff6ba3ad9d44a72        GET /                 http://app-codex-profiler-jpe-20260424.azurewebsites.net/            True    200          151.2283 <250ms            request
```
