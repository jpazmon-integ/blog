---
title: Log Analytics ワークスペースのコストを管理する方法
date: 2024-07-10 00:00:00
tags:
 - How-To
 - Log Analytics
---

こんにちは、Azure Monitoring チームの徳田です。

本ブログでは、Log Analytics ワークスペースのコストの管理方法 (原因の調査、コストを抑える方法) についてご説明します。  

  
なお、Azure Monitor のコスト最適化、および Log Analytics ワークスペースの使用量の分析については、以下弊社サイトもご参照ください。  
https://learn.microsoft.com/ja-jp/azure/azure-monitor/best-practices-cost
https://learn.microsoft.com/ja-jp/azure/azure-monitor/logs/analyze-usage
<!-- more -->

## 目次
- はじめに
- コスト増加の原因を分析する
  - データのインジェスト量の推移を確認する方法
  - データの保有量の推移を確認する方法
  - データの保有期間を確認する方法
- コストを抑える
  - データのインジェスト量を減らす
  - データの保有量を調整する
  - データの保有期間を調整する
- まとめ

## はじめに
Log Analytics ワークスペースは、データの収集と分析において非常に強力なツールです。  
しかし、そのコストは収集するデータの量によって大きく変動します。  
本ブログでは、Log Analyticsワークスペースのコスト管理方法について詳しく解説します。  
データインジェストのコストを最適化し、予期しない請求を避けるためのベストプラクティスを紹介します。

## コスト増加の原因を分析する
Log Analytics ワークスペースのコストが増加している場合、まずはじめに行うことはコスト増加の原因を分析することです。
Log Analytics ワークスペースのコストは以下 2 点によって決まります(*1)。
- データのインジェスト量
- データの保有量と保有期間
(*1) 単位あたりの料金はリージョンによって変化します。

したがって、コスト増加の主な原因を分析するためには、以下の情報を得る必要があります。
- データのインジェスト量の推移
- データの保有量の推移
- データの保有期間

それぞれの確認方法について、ご説明します。

### データのインジェスト量の推移を確認する方法
データのインジェスト量の推移を確認する方法は、主に以下 2 つです。
- [使用量と推定コスト] から確認する方法
- クエリを実行して確認する方法

#### [使用量と推定コスト] から確認する方法
- Azure portal で、コストを確認したい Log Analytics ワークスペースを開きます。
- 左ペインの [設定] > [使用量と推定コスト] を押下します。
- ページ右側に "使用状況のグラフ" として、過去 31 日間の、課金対象のテーブルごとのデータ インジェスト量が表示されます。
<<< 画像 >>>

#### クエリを実行して確認する方法
1. Azure portal で、コストを確認したい Log Analytics ワークスペースを開きます。
2. 左ペインの [ログ] を押下し、以下いずれかのクエリを実行します。

(過去 31 日間のログのインジェスト量を日ごと、課金対象のテーブルごとに確認したい場合)
```
let days = 31d;
Usage
| where TimeGenerated >= startofday(ago(days))
| where StartTime >= startofday(ago(days)) and EndTime < startofday(now())
| where IsBillable == true
| make-series TotalIngestedData = sum(Quantity) / 1.0E3 default=0 on TimeGenerated from startofday(ago(days)) to startofday(now()) step 1d by DataType
| project TimeGenerated, DataType, TotalIngestedData
| render columnchart 
```

(過去 3 か月のログのインジェスト量を、31 日ごとに確認したい場合)
```
let days = 93d;
Usage
| where TimeGenerated >= startofday(ago(days))
| where StartTime >= startofday(ago(days))
| where IsBillable
| summarize IngestedGB=sum(Quantity) / 1.0E3 by DataType, bin(StartTime, 31d)
| project StartTime, DataType, IngestedGB
| render columnchart
```

(指定した期間のログのインジェスト量を、日ごと、課金対象のテーブルごとに確認したい場合)
```
let StartDate = datetime(YYYY-MM-DD);
let EndDate = datetime(YYYY-MM-DD);
Usage
| where TimeGenerated >= StartDate and TimeGenerated < EndDate
| where StartTime >= StartDate and EndTime < EndDate
| where IsBillable == true
| make-series TotalIngestedData = sum(Quantity) / 1.0E3 default=0 on TimeGenerated from StartDate to EndDate step 1d by DataType
| project TimeGenerated, DataType, TotalIngestedData
| render columnchart 
```

(指定した期間のログのインジェスト量を、月ごと、課金対象のテーブルごとに確認したい場合)
```
let StartDate = datetime(YYYY-MM-DD);
let EndDate = datetime(YYYY-MM-DD);
Usage
| where TimeGenerated >= StartDate and TimeGenerated < EndDate
| where StartTime >= StartDate and EndTime < EndDate
| where IsBillable == true
| make-series TotalIngestedData = sum(Quantity) / 1.0E3 default=0 on TimeGenerated from StartDate to EndDate step 31d by DataType
| project TimeGenerated, DataType, TotalIngestedData
| render columnchart 
```

3. 結果を確認します。特に以下の項目について確認します。
* 一定期間、ログのインジェスト量が多い状態が続いている場合 → [](#) をご確認ください。
* 特定のテーブルへのログのインジェスト量が多い場合 → [](#) をご確認ください。

### データの保有量の推移を確認する方法
1. Azure portal で、コストを確認したい Log Analytics ワークスペースを開きます。
2. 左ペインの [ログ] を押下し、以下いずれかのクエリを実行します。

(過去 31 日間のデータの保有量の増加傾向を日ごと、課金対象のテーブルごとに確認したい場合)
```
Usage
| where TimeGenerated >= startofday(ago(31d))
| where IsBillable
| summarize DailyIngestedGB=sum(Quantity) / 1.0E3 by DataType, bin(TimeGenerated, 1d)
| sort by TimeGenerated asc
| extend CumulativeIngestedGB = row_cumsum(DailyIngestedGB)
| project TimeGenerated, DataType, CumulativeIngestedGB
| render timechart
```

(指定した期間のデータの保有量の増加傾向を日ごと、課金対象のテーブルごとに確認したい場合)
```
let StartDate = datetime(YYYY-MM-DD);
let EndDate = datetime(YYYY-MM-DD);
Usage
| where TimeGenerated >= StartDate and TimeGenerated < EndDate
| where StartTime >= StartDate and EndTime < EndDate
| where IsBillable == true
| summarize DailyIngestedGB=sum(Quantity) / 1.0E3 by DataType, bin(TimeGenerated, 1d)
| sort by TimeGenerated asc
| extend CumulativeIngestedGB = row_cumsum(DailyIngestedGB)
| project TimeGenerated, DataType, CumulativeIngestedGB
| render timechart
```

### データの保有期間を確認する方法


## コストを抑える
[上記](#コスト増加の原因を分析する) により、コスト増加の原因が明らかになった場合は、その原因に応じて、コストを抑えるための対処法を実行します。

### データのインジェスト量を減らす

### データの保有量を調整する


### データの保有期間を調整する