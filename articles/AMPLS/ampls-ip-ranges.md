---
title: AMPLS で必要な IP アドレスの個数について
date: 2025-12-22 00:00:00
tags:
  - Azure Monitor Private Link Scopes
  - Data Collection Endpoint
  - Application Insights
  - Log Analytics
  - Q&A
---

[更新履歴]  
-2025/12/22 ブログ公開  

---

こんにちは、Azure Monitoring & Integration サポート チームの佐々木です。

今回の記事では、Azure Monitor Private Link Scope 利用の際に必要になる IP レンジについてご説明します。

## 目次

- [目次](#目次)
- [計算方法について](#計算方法について)
- [8以外の各リソース追加によって追加される IP](#8以外の各リソース追加によって追加される-IP)
  - [Log Analytics](#Log-Analytics)
  - [データ収集エンドポイント](#データ収集エンドポイント)
  - [Application Insights](#Application-Insights)
- [Q&A](#Q&A)

## 計算方法について

下記の計算式で算出できます。

```
8 + Application Insights の接続文字列に定義されたエンドポイント数 + データ収集エンドポイント*3 + Log Analytics ワークスペースのリージョン数*3
```

8 個の内訳

1. api.monitor.azure.com
2. global.in.ai.monitor.azure.com
3. profiler.monitor.azure.com
4. live.monitor.azure.com
5. diagservices-query.monitor.azure.com
6. snapshot.monitor.azure.com
7. scadvisorcontentpl.blob.core.windows.net
8. global.handler.control.monitor.azure.com

例えば、AMPLS に 1 つの LA を接続する場合は 最低 11 個の IP アドレスが必要です。

次の項目ではより詳細に、追加される FQDN と IP をご紹介します。

## 8以外の各リソース追加によって追加される IP

以下ドキュメントに概要が記載されています。

[エンドポイントの DNS 設定を確認する](https://learn.microsoft.com/ja-jp/azure/azure-monitor/logs/private-link-configure#privatelink-monitor-azure-com)

### Log Analytics

Log Analytics は AMPLS へ追加された Log Analytics のリージョン1つにつき3つのIPアドレスを消費します。

以下の 3 個のエンドポイントはワークスペース毎に作られますが、リージョンごとに同じ IP が割り当てられます。

1. <ワークスペース ID>.privatelink-oms-opinsights-azure-com
2. <ワークスペース ID>.privatelink-ods-opinsights-azure-com
3. <ワークスペース ID>.privatelink-agentsvc-azure-automation-net

以下の様に同じリージョンのエンドポイントは同じ IP に割り当てられます。

<img width="1314" height="374" alt="image" src="https://github.com/user-attachments/assets/0e736def-83fe-4381-aed4-727a8cfaa91c" />

### データ収集エンドポイント

データ収集エンドポイントは AMPLS へ追加された データ収集エンドポイント 1つにつき3つのIPアドレスを消費します。

以下のエンドポイントにそれぞれ IP アドレスが割り当てられます。

1. <unique-dce-identifier>.<regionname>.handler.control
2. <unique-dce-identifier>.<regionname>.ingest
3. <unique-dce-identifier>.<regionname>.metrics.ingest

ワークスペースとは異なり、同じリージョンの DCE を AMPLS に 2 つ以上を追加した場合、 IP はそれぞれ追加されます。

### Application Insights

Application Insights は AMPLS へ追加された Application Insights のインジェストエンドポイントとライブメトリックエンドポイントのIPアドレスを消費します。

以下のエンドポイントにそれぞれ IP アドレスが割り当てられます。

1. <リージョン>-<ID>.in.ai.monitor.azure.com
2. <リージョン>.livediagnostics.monitor.azure.com

基本的に Application Insights 1 リージョンに対して 2 つの IP アドレスが消費される認識で問題ありませんが、
1 つのリージョンに対して 2 つのインジェスト エンドポイントが定義されるケースもあります。

つまり、同じリージョンの Application Insights が既に追加されていた場合でも 1つ の IP アドレスが追加されたり、されない場合があります。

具体的に以下の様に、同じリージョンであっても <ID> 部分が異なる場合があります。

この2つの Application Insights を AMPLS へ追加した場合、3つの IP が追加されます。

例1: InstrumentationKey=***;IngestionEndpoint=https://`japaneast-0`.in.applicationinsights.azure.com/;LiveEndpoint=https://`japaneast`.livediagnostics.monitor.azure.com/;ApplicationId=***
例2: InstrumentationKey=***;IngestionEndpoint=https://`japaneast-1`.in.applicationinsights.azure.com/;LiveEndpoint=https://`japaneast`.livediagnostics.monitor.azure.com/;ApplicationId=***

接続文字列は Application Insights > 概要 より確認いただけます。
<img width="1220" height="599" alt="image" src="https://github.com/user-attachments/assets/6bb136f8-20e3-4f43-b69e-d7c4b5847ff9" />

そのため、Application Insights では接続文字列内のエンドポイントを確認ください。

## Q&A

### Q. どの程度のIPレンジが必要になりますか？

A. お客様が将来的に AMPLS へ紐づけるリソースの数やそれらのリージョンによって必要な IP レンジは異なります。

追加される IP に対して、割り当て可能な IP レンジが不足した場合には、サブネットの再作成や VNET の再作成が必要になるケースもあります。

そのため冒頭に記載した計算式で計算いただき、その上で余裕のある数の IP レンジをご設定いただく事を推奨します。
