---
title: Azure Monitor エージェントにより収集される Heartbeat ログを使用した死活監視方法
date: 2025-01-31 00:00:00
tags:
  - Log Analytics
  - Azure Monitor Essentials
  - Azure Monitor Agent
  - How-To
---

こんにちは、Azure Monitoring サポート チームの北村、佐藤です。<br>
Azure Monitor エージェントを使用した死活監視方法について本ブログで紹介します。<br>
本ブログは、以下の過去の記事 (Log Analytics エージェントを使用した死活監視) を Azure Monitor エージェント版に更新したものとなります。<br>
-- 死活監視のクエリについて<br>
https://jpazmon-integ.github.io/blog/LogAnalytics/MonitorVM/

<br>

## 目次
- Heartbeat を使用した死活監視を行うために
- Heartbeat を使用した死活監視
- Heartbeat の収集が遅延していることを確認する
- まとめ

## Heartbeat を使用した死活監視を行うために
Azure Monitor エージェントを使用して死活監視をするためには、Heartbeat ログ (マシンが実行中の間、1 分間に 1 度収集されるログ) を収集する必要がございます。

まず、Azure Monitor エージェントがログを収集するよう構成する方法からご説明します。<br>
Azure Monitor エージェントは、データ収集ルールと呼ばれるリソースに仮想マシンを関連付けることで、インストールおよび必要なデータ収集設定の連携が行われます。<br>
データ収集ルールは、Azure Monitor エージェントを用いてどのようなログを収集するかを Azure Monitor エージェントに伝える指示書の役割を果たします。<br>

データ収集ルールでデータ ソース (どのようなログを取得するか) として設定可能なログ種は以下の通りです。<br>

■ Windows OS
- イベント ログ
- パフォーマンス
- テキスト ログ
- IIS ログ

■ Linux OS
- syslog
- パフォーマンス
- テキスト ログ

データ収集ルールは、上述のいずれかのデータ ソースを収集するよう指定する必要があります。マシンをデータ収集ルールと関連付け、上述のデータ ソースのログ収集を開始すると、Heartbeat ログも併せて収集されるようになります。<br>
すなわち、Azure Monitor エージェントをご利用いただく場合、基本的に Heartbeat ログを単体で収集することはできかねます。<br>

では、以下に Azure Monitor エージェントを使用したログ収集を行うための基本的な流れを簡単にご説明します。<br>
1. Log Analytics ワークスペースの作成
2. データ収集ルールの作成とデータ収集ルールへの関連付け
3. アラート設定

### 1. Log Analytics ワークスペースの作成
ログの送信先となる Log Analytics ワークスペースを作成します。<br>
-- Create a Log Analytics workspace<br>
https://learn.microsoft.com/ja-jp/azure/azure-monitor/logs/quick-create-workspace?tabs=azure-portal

### 2. データ収集ルールの作成とデータ収集ルールへの関連付け
1 で作成した Log Analytics ワークスペースにログを収集するよう、データ収集ルールを構成します。<br>
-- Collect data with Azure Monitor Agent<br>
https://learn.microsoft.com/ja-jp/azure/azure-monitor/agents/azure-monitor-agent-data-collection

(["データ ソース"](https://learn.microsoft.com/ja-jp/azure/azure-monitor/agents/azure-monitor-agent-data-collection#data-sources) 項目にて、各データ ソースをクリックすることでデータ ソースごとの設定方法の公開情報に遷移します。)

データ収集ルールにマシンを関連づけると Azure Monitor エージェントがインストールされます。<br>
(すでにインストール済みの場合には、マシン上に Azure Monitor エージェントが存在することが確認されたのち、インストールがスキップされます。)

### 3. アラート設定
作成した Log Analytics ワークスペースの画面左側にあるメニュー内 [ログ] をクリックします。<br>
[ログ] 検索フィールドでクエリを実行し、アラートの設定を行います。アラートの評価期間や設定したクエリを実行する頻度、
アラートの通知方法等を設定します。<br>
詳細は以下の弊社公開情報をご覧ください。<br>

-- Create or edit a log search alert rule<br>
https://learn.microsoft.com/ja-jp/azure/azure-monitor/alerts/alerts-create-log-alert-rule

-- Action groups<br>
https://learn.microsoft.com/ja-jp/azure/azure-monitor/alerts/action-groups

> [!NOTE]
> (補足)
> 前述の通り、Azure Monitor エージェントをご利用いただく場合、基本的に Heartbeat ログを単体で収集することはできかねます。
> どうしても、Heartbeat ログだけを収集される必要がある場合には、存在しないダミーの情報を収集するようデータ収集ルールを構成することで、
> 実質 Heartbeat ログだけを収集すること自体は可能でございます。
> ~~~~~~~~~~~~~~~~~~~~
> カスタム パス追加手順
> ~~~~~~~~~~~~~~~~~~~~
> 以下すべて [データ ソースの追加] タブでの設定でございます。
> 1. [データ ソースの種類] にて [パフォーマンス カウンター] を選択します。
> 2. [パフォーマンス カウンターを構成する] という項目で [カスタム] タブを選択します。
> 3. [収集するパフォーマンス カウンターと、そのサンプリング頻度を構成します:] に "\dummy" と入力し、[追加] をクリックします。
>  ![](./MonitorVM_AMA/sample_dummypath.png)
> 4. お手数ではございますが、3. で追加した "\dummy" 以外のパスの左側にあるチェックをすべて外します。
>
> なお、パフォーマンス カウンターについて、任意の収集設定を行う方法については以下のブログ記事に記載しております。
> 設定方法の詳細は以下をご確認ください。
> -- 既定で用意されているもの以外のパフォーマンス カウンターを収集する方法<br>
> https://jpazmon-integ.github.io/blog/LogAnalytics/HowToCollectCustomPerfCounter/

<br>

## Heartbeat を使用した死活監視
Heartbeat は Log Analytics エージェントや Azure Monitor エージェントによって Log Analytics ワークスペースに既定で収集されます。<br>
Log Analytics ワークスペースで Heartbeat のレコードを確認すると、1 分間に 1 回レコードが生成されていることがわかります。この Heartbeat を使用した死活監視の例をご紹介します。<br>

### 例 1. あるマシンから Heartbeat が収集されていることを確認する
下記クエリでは、指定した VM の Heartbeat を抽出します。

```
Heartbeat
| where Computer == 'computer-name'
```
![](./MonitorVM_AMA/result_example1.png)

上記クエリを利用したアラート ルールの設定例を示します。まず、アラート ルールの主な設定項目をご説明します。
![](./MonitorVM_AMA/parameters.png)

- 集計の粒度 (赤枠線部分)
アラートの検索クエリが 1 回の評価を行う際に評価の対象とする期間です。<br>
以下のように 5 分と設定した場合は 5 分間隔でグループ化され、検索クエリが実行されます。<br>
 
- 演算子としきい値 (黄色ハイライト部分)
上記クエリは、直近 5 分以内の Heartbeat を返しますので、当該クエリの実行結果が 0 件の場合は直近 5 分間 Heartbeat が収集されていないことを意味します。<br>
そのため、演算子は 「等しい」、しきい値は 「0」 としています。<br>
 
- 評価の頻度 (黄色枠線部分)
アラートの検索クエリが実行される間隔です。この例では、5 分ごとにクエリが実行されます。<br>

- 評価期間 (黒枠線部分)
過去何分間のログを評価対象とするかを設定する項目でございます。<br>

- 違反の数 (緑枠線部分)
評価期間内にある集約ポイントのうち (集約ポイントの個数 = 評価期間 ÷ 集計の粒度) 、指定した回数以上発報条件を満たしたらアラートが発報します。<br>
(連続で発報条件を満たした回数ではございません。)<br>

例えば、以下の設定であったとします。<br>
その場合、具体的には以下の挙動となります。<br>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
集計の粒度 : 5 分
評価頻度 : 10 分
評価期間: 15 分
違反の数 : 2 (評価期間内に 2 回閾値を超えたら発報する構成の場合)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

以下、〇はしきい値を超えること、×はしきい値を超えないことを表します。<br>
10:00 の評価<br>
9:45 ~ 9:50 のログ :  ×<br>
9:50 ~ 9:55 のログ : 〇<br>
9:55 ~ 10:00 のログ : ×<br>
---------> 発報しない<br>
 
10:10 の評価<br>
9:55 ~ 10:00 のログ : ×<br>
10:00 ~ 10:05 のログ :  〇<br>
10:05 ~ 10:10 のログ : 〇<br>
---------> 発報する<br>
 
10:20 の評価<br>
10:05 ~ 10:10 のログ : 〇<br>
10:10 ~ 10:15 のログ :  ×<br>
10:15 ~ 10:20 のログ : 〇<br>
---------> 発報する<br>


<ご参考><br>
アラートの設定項目は下記弊社公開情報にも記載しております。<br>
–- Create or edit a log search alert rule<br>
https://learn.microsoft.com/ja-jp/azure/azure-monitor/alerts/alerts-create-log-alert-rule

### 例 2. 直近 15 分間に Heartbeat が途絶えた VM を確認する
下記クエリでは、直近 15 分間、VM の Heartbeat が送信されなかった場合に検知します。<br>
LastCall は 各コンピューターの Heartbeat が生成された最新の時刻です。<br>
Heartbeat の最新時刻が 15 分より前の場合、実行結果が返されます。<br>

```
Heartbeat
| summarize LastCall = max(TimeGenerated) by Computer 
| where LastCall < ago(15m)
```
![](./MonitorVM_AMA/result_example2.png)

> [!NOTE]
> アラートルールに設定する場合、アラートルール側では 15 分より長い粒度(期間) を指定する必要があります。
> 例えばこのクエリを利用する場合には、以下の様に設定する必要があります。
> メジャー: テーブルの行
> 集計の種類: カウント
> 集計の粒度: 30 分 (クエリで指定した期間より長い期間)
> 演算子: 次の値より大きい
> しきい値: 0
> 頻度: 15分
<br>

### 例 3. 過去 24 時間以内に Heartbeat が収集された VM における最新の Heartbeat を確認する
過去 24 時間以内に Heartbeat が収集された VM における 最新の Heartbeat を返します。<br>
クエリの 2 行目で、Log Analytics ワークスペース上の Heartbeat ログを過去 24 時間以内に収集されたレコードに絞り込み、
3 行目で送信元の Computer ごとに最新の Heartbeat の時刻を検索します。<br>
つまり、下記クエリでは、"過去 24 時間以内における Computer ごとの 最新の Heartbeat の時刻” を検索することが可能です。<br>
 
```
Heartbeat
| where TimeGenerated > ago(24h)
| summarize LastCall = max(TimeGenerated) by Computer
```
![](./MonitorVM_AMA/result_example3.png)

<ご参考><br>
Heartbeat を使用したサンプル クエリにつきましては、下記弊社公開情報にも掲載しておりますのでご確認いただけると幸いです。<br>
-- Azure Monitor での Agent Health ソリューション - サンプル ログ<br>
https://learn.microsoft.com/ja-jp/azure/azure-monitor/agents/solution-agenthealth#sample-log-searches
<br>


## Heartbeat の収集が遅延していることを確認する
死活監視のアラートを検知したにもかかわらず、監視対象の VM は正常に動作している。<br>
このようなときは、Heartbeat の収集遅延が発生している可能性がございます。<br>
 
-- Azure Monitor でのログ データ インジェスト時間<br>
https://docs.microsoft.com/ja-jp/azure/azure-monitor/logs/data-ingestion-time
<br>
※ ログ データを取り込むための一般的な待ち時間は 20 秒から 3 分です。ただしシステムの負荷状況によりさらに遅延が発生する可能性がございます。<br>
 
 
Heartbeat の収集遅延には、様々な理由が考えられますが、以下のクエリを実行していただきますと、
"Log Analytics エージェントがデータを生成してから Azure 側でデータを受信するまで" の間に遅延が発生していたことを確認できます。
 
```
Heartbeat 
| where Computer == "Computer Name"
| extend E2EIngestionLatency = ingestion_time() - TimeGenerated
| project Computer, TimeGenerated, _TimeReceived, ingestion_time(), E2EIngestionLatency
| order by TimeGenerated desc
```

※ TimeGenerated : データ ソースが生成された時刻<br>
※ _TimeReceived : Azure Monitor のインジェスト エンドポイントによって受信される時刻<br>
※ $ingestion_time : Log Analytics ワークスペースに保存され、クエリ検索が可能となる時刻<br>
※ E2EIngestionLatency : ログが生成されてから Log Analytics ワークスペースにログが送信されるまでに要した時間<br>

下記画像は上記クエリを実行した例です。<br>
赤線で囲んだ部分が「ログが生成されてから Log Analytics ワークスペースにログが送信されるまでに要した時間」です。<br>

![](./MonitorVM_AMA/result_latency.png)

Heartbeat ログ アラートが期待されないタイミングで発報した場合の調査方法については、よろしければ以下のブログ記事もご参照ください。<br>

-- マシンが起動中にも関わらず Heartbeat ログ アラートが発報された場合の調査方法<br>
https://jpazmon-integ.github.io/blog/LogAnalytics/Unexpected_Heartbeat_logalert/
<br>

## まとめ
本記事では、Azure Monitor エージェントを使用して Heartbeat ログを収集する方法および、収集された Heartbeat ログを用いて死活監視を行う方法をご案内いたしました。<br>

Heartbeat ログを含め、仮想マシンの死活監視全般については以下のブログ記事でもご案内しております。よろしければこちらもご参照いただけますと幸いです。<br>

-- Azure VM における死活監視の考え方<br>
https://jpazmon-integ.github.io/blog/LogAnalytics/MonitorVM02/
<br>
最後までお読みいただき、ありがとうございました！
