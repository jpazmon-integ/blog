---
title: 2025 年 3 月 31 日までにインストルメンテーション キーから接続文字列によるインジェストへの移行をお願いいたします
date: 2024-01-20 00:00:00
tags:
  - How-To
  - Migration
  - Application Insights
  - インストルメンテーション キー
  - 接続文字列
---

こんにちは、Azure Monitoring サポート チームの佐藤です。
今回は、2025 年 3 月 31 日にサポートが終了いたしますインストルメンテーション キーによるインジェストから接続文字列によるインジェストに移行いただく方法および注意点をご案内いたします。

[Transition to using connection strings for data ingestion by 31 March 2025] :https://azure.microsoft.com/en-us/updates/technical-support-for-instrumentation-key-based-global-ingestion-in-application-insights-will-end-on-31-march-2025/

<br>

<!-- more -->
## 目次
1. 必要な対応について
2. 移行に関する注意点
3. 参考情報

<br>
## 1. 必要な対応について

### 対象となる環境
Application Insights SDK をご利用いただき、Web アプリケーションから Application Insights にログを送信されている環境が対象となります。

### 事前準備
ご利用いただいております Application Insights の SDK が、サポートされているバージョンであるかご確認ください。

- サポートされている SDK バージョン
https://learn.microsoft.com/ja-jp/azure/azure-monitor/app/migrate-from-instrumentation-keys-to-connection-strings#supported-sdk-versions

### 移行方法
1. Application Insights リソースの [概要] ペインにアクセスしてください。

2. 右側に表示されている接続文字列を見つけます。

3. 接続文字列をポイントし、[クリップボードにコピー] アイコンを選択します。

4. 以下の公開情報に従い、接続文字列を使用してインジェストを行うよう Application Insights SDK を構成します。
- 接続文字列を設定する
https://learn.microsoft.com/ja-jp/azure/azure-monitor/app/sdk-connection-string?tabs=dotnet5#set-a-connection-string

※ 接続文字列への移行完了後、インストルメンテーション キーの設定は削除いただくようお願いいたします。

<br>
<br>

## 2. 移行に関する注意点
移行に際しご留意いただきたい点を 2 点、ご紹介いたします。
### 2-1. インストルメンテーション キーと接続文字列は同居できない
監視対象の Web アプリケーションにインストルメンテーション キーと接続文字列の両方を設定した場合、
インストルメンテーション キーによるインジェストが優先されます。
インストルメンテーション キーと接続文字列の同居によりデータの欠落などが生じる可能性があるため、
接続文字列に移行いただきました際にはインストルメンテーション キーの削除をいただけますと幸いです。


### 2-2. 加味される日次上限の設定箇所に変更が生じる
インストルメンテーション キーによるインジェストを使用している場合には、
Application Insights リソースに設定した日次上限はリージョンをまたいで有効にならない可能性があり、
Log Analytics ワークスペース側に設定した日次上限のみが適用されます。

一方で、接続文字列によるインジェストを行う場合には、
Application Insights と Log Analytics ワークスペースの両方の日次上限がリージョンごとに有効となります。
すなわち、Application Insights にて設定した日次上限と Log Analytics ワークスペースの日次上限の両方が適用対象となり、
2 つのうち設定された上限値が小さいほうの値が上限値として使用されます。

<br>
<br>

## 3. 参考情報
- Application Insights インストルメンテーション キーから接続文字列への移行 > 移行
https://learn.microsoft.com/ja-jp/azure/azure-monitor/app/migrate-from-instrumentation-keys-to-connection-strings#migration


<br>
<br>
