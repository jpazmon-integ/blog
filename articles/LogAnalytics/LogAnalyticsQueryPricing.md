---
title: Log Analytics のクエリの実行コストについて
date: 2025-09-12 00:00:00
tags:
  - How-To
  - クエリ
  - コスト
  - Log Analytics
---

こんにちは、Azure Monitoring サポート チームの六浦です。

今回は Log Analytics でクエリを実行する際のコストについてご説明します。

<!-- more -->

## 目次
- [1. クエリ実行のコストに関わる設定の確認](#1-クエリ実行のコストに関わる設定の確認)
- [2. Analytics のリテンション期間のログに対してクエリを実行する場合](#2-Analytics-のリテンション期間のログに対してクエリを実行する場合)
- [3. 長期保持 (アーカイブ) 期間のログに対してクエリを実行する場合](#3-長期保持-(アーカイブ)-期間のログに対してクエリを実行する場合)
- [4. まとめ](#4-まとめ)

<br>

## 1. クエリ実行のコストに関わる設定の確認
Azure ポータルの Log Analytics ワークスペースの [設定] > [テーブル] より、クエリを実行するテーブルの以下の設定を確認します。

| 設定 | 設定可能な値 |
| ---- | ---- |
| プラン | Analytics、基本、補助 のいずれかです。 |
| Analytics のリテンション期間 | 最大 2 年まで設定できます。 |
| 保有期間の合計 | 最大 12 年まで設定できます。 |


これらの設定によって、クエリ実行時のコストが決まります。


<br>

## 2. Analytics のリテンション期間のログに対してクエリを実行する場合
1 で確認した [プラン] によってクエリの実行コストは異なります。

<br>

### 2.1 Analytics テーブル プランの場合
**無料**でクエリを実行できます。
Analytics テーブル プランはデフォルトのテーブル プランのため、多くのテーブルに該当します。

<br>

### 2.2 基本 (Basic) テーブル プランと補助 (Auxiliary) テーブル プランの場合
クエリ実行時にスキャンしたデータ量に応じたコストが発生します。
料金は[価格についての公開情報](https://azure.microsoft.com/ja-jp/pricing/details/monitor/)の [クエリ] からご確認ください。
なお、基本および補助テーブルのログに対して実行できるクエリには制限があります。
制限については、[こちら](https://learn.microsoft.com/ja-jp/azure/azure-monitor/logs/basic-logs-query?tabs=portal-1)をご確認ください。


> [!NOTE]
> Basic テーブル プランは一部のテーブルで利用できます。対象のテーブルは[こちら](https://learn.microsoft.com/ja-jp/azure/azure-monitor/logs/basic-logs-azure-tables)を確認ください。
> Auxiliary テーブル プランは DCR ベースのカスタム テーブルのみで利用できます。
> テーブル プランの詳細については[こちら](https://learn.microsoft.com/ja-jp/azure/azure-monitor/logs/data-platform-logs#table-plans)の公開情報もご確認ください。

<br>

## 3. 長期保持 (アーカイブ) 期間のログに対してクエリを実行する場合
長期保持期間は、1 で確認した [保有期間の合計] から [Analytics のリテンション期間] を引いた期間です。
長期保持期間のログに対するクエリ実行には、テーブル プランに関わらず、[検索ジョブ]((https://learn.microsoft.com/ja-jp/azure/azure-monitor/logs/search-jobs?tabs=portal-1%2Cportal-2))の実行、または[ログの復元](https://learn.microsoft.com/ja-jp/azure/azure-monitor/logs/restore?tabs=api-1)のコストがかかります。
なお、検索ジョブで作成される結果テーブルのログと復元されたログは、Analytics テーブル プランのログのため、クエリ実行による追加のコストは発生しません。

検索ジョブと復元のコストは、[価格についての公開情報](https://azure.microsoft.com/ja-jp/pricing/details/monitor/)の [ジョブの検索] と [復元] をご確認ください。


> [!NOTE]
> 現在 Auxiliary テーブル プランは復元をサポートしません。Auxiliary テーブルから長期保持されているデータを取得するには、検索ジョブをご利用ください。

<br>

## 4. まとめ
本記事では、クエリ実行時のコストについてご案内いたしましたが、いかがでしたでしょうか。
Log Analytics ワークスペースのその他のコストについてはこれらの公開情報にてご案内しておりますので、合わせてご覧ください。
[Azure Monitor ログのコストの計算とオプション](https://learn.microsoft.com/ja-jp/azure/azure-monitor/logs/cost-logs)
[Log Analytics ワークスペースのコスト増加の原因を分析しコスト削減を検討する](https://jpazmon-integ.github.io/blog/LogAnalytics/HowToManageLogAnalyticsBilling/)


最後までお読みいただきありがとうございました！
