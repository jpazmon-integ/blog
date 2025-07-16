---
title: "Azure Montior のアラート ルールが発報した原因調査に関するお問い合わせを発行いただく前の確認ポイント"
date: 2025-08-15 00:00:00
tags:
  - Azure Monitor Essential
  - Service Health
  - Resource Health
  - Metric Alert
  - Activity Log Alert
  - Log Alert
  - How-To
---

こんにちは、Azure Monitoring サポート チームの北村、徳田です。
今回は [Azure Monitor のアラート ルール](https://learn.microsoft.com/ja-jp/azure/azure-monitor/alerts/alerts-overview)が発報した原因の調査をご依頼いただく際に、お問い合わせを起票される前にご確認いただきたいポイントをご紹介します。本記事の内容をご確認いただくことで、よりスムーズに調査を実施することが可能となりますので、ご参考になれば幸いです。

<br>


<!-- more -->
## 目次
- []()
- []()
- []()

<br>



## 1. はじめに
「Azure Monitor のアラート ルールが発報した原因を調査してほしい」というお問い合わせをよくいただきますが、
**アラート ルールの閾値を満たして発報したのか**、それとも**閾値を満たしていないにもかかわらず発報されたのか**によって、調査の観点が異なります。

| アラート ルールが発報した経緯                                          | 調査観点            |
| --------------------------------------------------------------- | ------------------- |
| アラート ルールが閾値を満たして発報した場合                     | Azure リソース観点        |
| アラート ルールが閾値を満たしていないものの発報した場合 | アラート ルール観点 |

**閾値を満たして発報した場合は、監視対象である Azure リソースに問題がある可能性があるため、Azure リソース観点での調査が必要です。一方で、閾値を満たしていないにもかかわらず発報した場合は、アラート ルールの動作に問題がある可能性があるため、アラート ルール観点（Azure Monitor 製品観点）で調査を実施する必要がございます。**

つまり、アラートが発報した状況によって、[お問い合わせの起票時に選択いただく製品カテゴリ](https://learn.microsoft.com/ja-jp/azure/azure-portal/supportability/how-to-create-azure-support-request#create-a-support-request:~:text=%E5%95%8F%E9%A1%8C%E3%81%AB%E9%96%A2%E9%80%A3%E3%81%97%E3%81%A6%E3%81%84%E3%82%8B%E3%82%B5%E3%83%BC%E3%83%93%E3%82%B9%E3%82%92%E6%8C%87%E5%AE%9A%E3%81%97%E3%81%BE%E3%81%99%E3%80%82) (監視している Azure リソース or アラート ルール) が変わってきます。この切り分けのために **「閾値を満たして発報したかどうか」**を事前にご確認いただくことで、弊社での調査がよりスムーズに進みます。

本記事では、アラートの発報条件を確認する方法や、実際に閾値を満たしていたかどうかを確認する方法などをご紹介します。



<br>
<br>



## 2. メトリック アラート ルール

### 2-1. 確認するポイント
**[メトリック アラート ルール](https://learn.microsoft.com/ja-jp/azure/azure-monitor/alerts/alerts-types#metric-alerts)の場合は、アラート ルールが実行された時にメトリックの値が閾値を満たしていたかどうかを確認します。**メトリック アラート ルールは、Azure プラットフォームから既定で収集される[プラットフォーム メトリックやカスタム メトリック](https://learn.microsoft.com/ja-jp/azure/azure-monitor/metrics/data-platform-metrics#types-of-metrics)を、一定の間隔で監視します。アラート ルールの閾値を満たして発報したかどうかを確認するためには、アラートが評価した期間と、当該期間におけるメトリックの値を確認する必要があります。

<br>

### 2-2. 閾値を満たしていたかどうかを確認する方法
まずは、アラート ルールが監視している Azure リソースとアラートの評価期間を確認します。
次に、メトリック エクスプローラーで評価期間におけるメトリックの値を確認します。


1. Azure ポータル > モニター > [アラート] を開き、発報したアラートを選択します。
![](./HowToCreateAlertRuleSR/image-metric-alert-01.png)


2. 監視しているリソース、評価された値、アラートの評価期間を確認します。
[メトリック アラートの詳細] 画面が開きますので、下表の <該当する項目> に表示されている内容をご確認ください。
このアラート ルールでは、Azure VM の [VmAvailabilityMetric](https://learn.microsoft.com/ja-jp/azure/virtual-machines/monitor-vm-reference#:~:text=VmAvailabilityMetric) (可用性メトリック) を監視しており、 評価期間 5 分以内の平均値が 0.8 を下回った場合に発報します。


| 確認する項目                | 画面上の該当箇所                                                                                                      | 例（下図参照）                                                        |
| --------------------------- | --------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| 監視対象のリソース | 影響を受けるリソース (赤枠部分)                                                                                      | Azure VM (win2022-jpe)                                           |
| 評価された値 | このアラートが発生した理由 (青枠部分)                                                                               | 0.5  |     | 
| アラートの評価期間  | Evaluation window start time, <br>Evaluation window end time (緑枠部分) | 2025/7/3 16:58 ～ 2025/7/3 17:03                                 |


![](./HowToCreateAlertRuleSR/image-metric-alert-02.png)

<br>

3. [影響を受けるリソース] のリンクを押下し、左ペインの [監視] > [メトリック] から [メトリック エクスプローラー](https://learn.microsoft.com/ja-jp/azure/azure-monitor/metrics/analyze-metrics)を開きます。

4. [スコープ] に監視している Azure リソースが表示されていることを確認します（赤枠部分）。
このアラート ルールは VmAvailabilityMetric の平均を監視しているので、[メトリック] で **VmAvailabilityMetric**、[集計の粒度] は **平均** を選択します（青枠部分）。画面右上の [現地時刻] のボタンを押下し、2. で確認したアラートの評価期間 (2025/7/3 16:58 ～ 17:03 JST) を指定します（緑枠部分）。今回は、評価期間内のメトリックの推移を最小単位 (1 分) で確認したかったため、[時間の粒度] は **1 分** を指定していますが、7/3 16:59 以降はメトリックが 1 を下回っていることが分かります。
このような場合は閾値を満たして発報しているため、発報した原因の調査をご希望される場合には、**アラート ルール観点ではなく、監視しているリソース (Azure VM) 観点で調査が必要となります。**
![](./HowToCreateAlertRuleSR/image-metric-alert-03.png)

<br>
<br>

### 2-3. メトリックの値を確認するときにご留意いただきたいこと
メトリック アラート ルール編集画面の [プレビュー] や、アラートの詳細画面で表示されるグラフは、実際のアラート ルールの評価期間に基づいた集計の粒度で表示されていない場合がございます。アラートの評価期間に記録されたメトリックの値を確認する際には、[メトリック エクスプローラー](https://learn.microsoft.com/ja-jp/azure/azure-monitor/metrics/analyze-metrics)からご確認いただきますようお願いいたします。

- メトリック アラート ルール編集画面
![](./HowToCreateAlertRuleSR/image-metric-alert-04.png)

- アラートの詳細画面
![](./HowToCreateAlertRuleSR/image-metric-alert-05.png)


<br>
<br>



## 3. ログ アラート ルール
<!-- 3.1 閾値を満たして発報したかどうかを確認する -->
<!-- https://jpazmon-integ.github.io/blog/LogAnalytics/Unexpected_Heartbeat_logalert/ が参考になりそうか-->


<br>
<br>



## 4. アクティビティ ログ アラート
[アクティビティ ログ アラート](https://learn.microsoft.com/ja-jp/azure/azure-monitor/alerts/alerts-types#activity-log-alerts)は、条件に一致するアクティビティ ログが出力されたかどうかを監視します。

[アクティビティ ログ](https://learn.microsoft.com/ja-jp/azure/azure-monitor/platform/activity-log-schema#categories)は、サブスクリプション レベルのイベントが記録されます。例えば、リソースが作成された、変更された、削除された、Azure VM が起動された、といったログが出力されます。アクティビティ ログは、操作が実行されるリソースのリソース プロバイダーによって提供されるため、リソース プロバイダー側でアクティビティ ログの出力条件が決定されております。
このため、**アラートの条件を満たして発報し、アクティビティ ログが記録された原因の調査をご希望の場合は、監視対象のリソース観点でお問い合わせをご起票ください。**

本記事では、アクティビティ ログ アラートの発報条件と、アクティビティ ログ アラートで検知した内容を確認する手順をご紹介します。なお、**アラートの条件を満たしていないにもかかわらず発報したと疑われる場合は、アラート ルールの動作に関する調査が必要となるため、アラート ルール観点 (Azure Monitor 製品) でお問い合わせをご起票ください。**

<br>


### 4-1. アクティビティ ログ アラートの発報条件を確認する
1. Azure ポータル > モニター > [アラート] を開き、画面上部の [アラート ルール] を選択します。

2. アクティビティ ログ アラートを選択し、[概要] ページを開きます。赤枠部分が発報条件、青枠部分がアラートのスコープです。
下記の設定の場合は、指定した仮想マシンで、下記のアクティビティ ログが出力された場合に発報します。アクティビティ ログ アラートの条件につきましては、[公開情報](https://learn.microsoft.com/ja-jp/azure/azure-monitor/alerts/alerts-create-activity-log-alert-rule?tabs=activity-log#:~:text=%5B%E3%82%A2%E3%83%A9%E3%83%BC%E3%83%88%20%E3%83%AD%E3%82%B8%E3%83%83%E3%82%AF%5D%20%E3%82%BB%E3%82%AF%E3%82%B7%E3%83%A7%E3%83%B3%E3%81%A7%E3%80%81%E3%81%93%E3%82%8C%E3%82%89%E3%81%AE%E5%90%84%E3%83%95%E3%82%A3%E3%83%BC%E3%83%AB%E3%83%89%E3%81%AE%E5%80%A4%E3%82%92%E9%81%B8%E3%81%B3%E3%81%BE%E3%81%99%E3%80%82)をご覧ください。

- ([カテゴリ](https://learn.microsoft.com/en-us/azure/azure-monitor/platform/activity-log-schema#categories)が Administrative) かつ (操作名が Microsoft.Compute/virtualMachines/start/action) かつ ([レベル](https://learn.microsoft.com/en-us/azure/azure-monitor/platform/activity-log-schema#severity-level)が Informational) かつ (状態= Started)

![](./HowToCreateAlertRuleSR/image-activity-log-alert-01.png)


※ 下図は、[概要] ページ画面上部の [JSON ビュー] を選択した画面です。
![](./HowToCreateAlertRuleSR/image-activity-log-alert-02.png)


<br>


### 4-2. 閾値を満たして発報したかどうかを確認する
下記の手順でアクティビティ ログ アラートが検知した内容を確認します。

1. Azure ポータル > モニター > [アラート] を開き、発報したアラートを選択します。
![](./HowToCreateAlertRuleSR/image-activity-log-alert-03.png)

2. 発報したリソースと検知されたアクティビティ ログの内容を確認します。
[アクティビティ ログ アラートの詳細] 画面が開きますので、下表の <画面上の該当箇所> の内容をご確認ください。
このアクティビティ ログ アラートは **(カテゴリが Administrative) かつ (操作名が Microsoft.Compute/virtualMachines/start/action) かつ (レベルが Informational) かつ (状態= Started)** のときに発報する条件のため、想定どおりアラートが発報したことが分かります。

| 確認する項目                                        | 画面上の該当箇所                        | 例（下図参照）                                                                                                  | 
| --------------------------------------------------- | --------------------------------------- | --------------------------------------------------------------------------------------------------------------- | 
| 監視対象のリソース                                  | 影響を受けるリソース (赤枠部分)         | Azure VM (win2022-jpe)                                                                                          | 
| アクティビティ ログの操作名、<br> カテゴリ、レベル       | このアラートが発生した理由 (青枠部分)   | 操作名 : Microsoft.Compute/virtualMachines/start/action<br>カテゴリ : Administrative<br>レベル :  Informational | 
| アクティビティ ログの状態と<br>ログが出力された日時 | Status, Submission timestamp (緑枠部分) | 状態 : Started<br>ログが出力された日時 : 2025/7/16 14:48                                                        | 


<br>

### 4-3. 監視しているアクティビティ ログを確認する
アクティビティ ログの条件に一致するログは、以下の手順にてご確認いただくことが可能です。

1. Azure ポータル > モニター > [アラート] を開き、画面上部の [アラート ルール] を選択します。

2. アクティビティ ログ アラートを選択し、[概要] ページを開きます。

3. [条件] の項目に表示されている "Azure Monitor でイベントを表示する - アクティビティ ログ" のリンクを押下します。
![](./HowToCreateAlertRuleSR/image-activity-log-alert-05.png)

3. アラートの条件に一致するアクティビティ ログが表示されます。
※ 同じ操作名のアクティビティ ログが 3 件ありますが、状態が「開始済み (Started)」のログが該当します。
![](./HowToCreateAlertRuleSR/image-activity-log-alert-06.png)

4. [JSON] を選択すると、アクティビティ ログの内容が表示されます。
![](./HowToCreateAlertRuleSR/image-activity-log-alert-07.png)

> [!NOTE]
> アクティビティ ログのカテゴリやレベルなどの値は、[こちら](https://learn.microsoft.com/ja-jp/azure/azure-monitor/platform/activity-log-schema)の公開情報でご確認ください。


<br>
<br>



## 5. リソース正常性アラート
[リソース正常性アラート](https://learn.microsoft.com/ja-jp/azure/service-health/resource-health-overview)は、リソースの正常性に変化が生じた際に通知する機能です。
リソースの正常性に変化が生じると、リソース プロバイダーによってリソース正常性に関するイベントがアクティビティ ログに書き込まれます。出力されたアクティビティ ログがアラートで指定した条件を満たした場合に発報します。
そのため、**アラートの条件を満たして発報し、リソース正常性イベントが発生した原因の調査をご希望の場合は、監視対象のリソース観点でお問い合わせをご起票ください。**

本記事では、リソース正常性アラートの発報条件と、リソース正常性アラートで検知した内容を確認する手順をご紹介します。
なお、**アラートの条件を満たしていないにもかかわらず発報したと疑われる場合は、アラート ルールの動作に関する調査が必要となるため、アラート ルール観点 (Azure Monitor 製品) でお問い合わせをご起票ください。**

<br>


### 5-1. リソース正常性アラートの発報条件を確認する
1. Azure ポータル > モニター > [アラート] を開き、画面上部の [アラート ルール] を選択します。

2. リソース正常性アラートを選択し、[概要] ページを開きます。赤枠部分が発報条件、青枠部分がアラートのスコープです。
下記の設定の場合は、指定したリソース グループの仮想マシン (microsoft.compute/virtualmachines のリソース) で、下記の条件を満たした場合に発報します。リソース正常性アラートの条件と設定値につきましては、[リソース正常性アラートに関するよくあるご質問](https://jpazmon-integ.github.io/blog/AzureMonitorEssential/ResourceHealthAlert/)のブログをご覧ください。


- (イベントの状態が Active) かつ (現在のリソースの状態が Degraded または Unavailable) かつ (以前のリソースの状態が Available) かつ (理由の種類が Platform Initiated または Unknown または User Initiated)


※ 下図は、[概要] ページ画面上部の [JSON ビュー] を選択した画面です。
![](./HowToCreateAlertRuleSR/image-resource-health-01.png)

<br>


### 5-2. 閾値を満たして発報したかどうかを確認する
下記の手順でリソース正常性アラートが検知した内容を確認します。

1. Azure ポータル > モニター > [アラート] を開き、発報したアラートを選択します。
![](./HowToCreateAlertRuleSR/image-resource-health-04.png)

2. 発報したリソースと検知されたアクティビティ ログの内容を確認します。
[リソースの正常性 アラートの詳細] 画面が開きますので、下表の <画面上の該当箇所> の内容をご確認ください。
このリソース正常性アラートは **(イベントの状態が Active) かつ (現在のリソースの状態が Degraded または Unavailable) かつ (以前のリソースの状態が Available) かつ (理由の種類が Platform Initiated または Unknown または User Initiated)** のときに発報する条件のため、想定どおりアラートが発報したことが分かります。

| 確認する項目                | 画面上の該当箇所                                                                                                      | 例（下図参照）                                                        |
| -------------------------------------------- | ------------------------------------- | ---------------------------------------------------------------------------------- |
| 監視対象のリソース                           | 影響を受けるリソース (赤枠部分)       | Azure VM (win2022-jpe)                                                             |
| リソース正常性の状態、 <br>イベントが発生した理由 | このアラートが発生した理由 (青枠部分) | リソース正常性の状態 : Available から Unavailable <br>理由の種類 : UserInitiated |
| イベントの状態、 <br>イベントが発生した日時       | Status, Event timestamp (緑枠部分)    | イベントの状態が : Active<br>イベントが発生した日時 : 2025/7/14 17:16 JST          |


![](./HowToCreateAlertRuleSR/image-resource-health-05.png)

<br>


### 5-3. リソース正常性のアクティビティ ログを確認する
リソース正常性のアクティビティ ログは、以下の手順にてご確認いただくことが可能です。

1. Azure ポータル > モニター > [アクティビティ ログ] を開き、[フィルターの追加] を選択します。
![](./HowToCreateAlertRuleSR/image-resource-health-02.png)

2. フィルタ―の条件として **イベントのカテゴリ** を選択し、**リソース正常性** カテゴリを指定します。
![](./HowToCreateAlertRuleSR/image-resource-health-03.png)

3. [リソースの正常性 アラートの詳細] 画面で確認した Event timestamp (2025/7/14 17:16) のアクティビティ ログを選択します。
※ 同じ日時に記録されたアクティビティ ログが 2 件ありますが、状態が「アクティブ (Active)」のログが該当します。
![](./HowToCreateAlertRuleSR/image-resource-health-06.png)

4. [JSON] を選択すると、アクティビティ ログの内容が表示されます。
![](./HowToCreateAlertRuleSR/image-resource-health-07.png)

> [!NOTE]
> リソース正常性アラートの条件と設定値につきましては、[リソース正常性アラートに関するよくあるご質問](https://jpazmon-integ.github.io/blog/AzureMonitorEssential/ResourceHealthAlert/)のブログをご覧ください。また、アクティビティ ログのスキーマは、[こちら](https://learn.microsoft.com/ja-jp/azure/azure-monitor/platform/activity-log-schema#resource-health-category)の公開情報でご確認ください。

<br>
<br>



## 6. サービス正常性アラート
[サービス正常性アラート](https://learn.microsoft.com/ja-jp/azure/service-health/overview)は、お客様環境にてご利用いただいている Azure サービスおよびリージョンを監視対象とし、Azure サービス自体の正常性を監視します。例えば、以下のような情報が通知されます：

- Azure サービスの障害
- 計画メンテナンス
- Azure サービスの仕様変更や廃止
- セキュリティ関連の重要情報（可用性に影響する可能性があるもの）

<!-- サービス正常性アラートの仕組みを追記する -->

[サービス正常性アラートで通知された内容についてご不明な点がある場合は、基本的に該当するサービス観点でお問い合わせをご起票くださいますようお願いいたします](https://jpazmon-integ.github.io/blog/AzureMonitorEssential/MonitorAlertFAQ/#Q-%E3%82%B5%E3%83%BC%E3%83%93%E3%82%B9%E6%AD%A3%E5%B8%B8%E6%80%A7%E3%82%A2%E3%83%A9%E3%83%BC%E3%83%88%E3%81%A7%E9%80%9A%E7%9F%A5%E3%81%95%E3%82%8C%E3%81%9F%E5%86%85%E5%AE%B9%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6%E7%A2%BA%E8%AA%8D%E3%81%97%E3%81%9F%E3%81%84%E3%81%A7%E3%81%99%E3%80%82)。サービス正常性の通知メールの下部には、対象となるサービス名が記載されておりますので、該当サービスのカテゴリを選択してお問い合わせいただきますようお願いいたします。

※ 下記例の場合は Azure Monitor 観点でお問い合わせをご起票ください。
<img width="750" src="../HowToCreateAlertRuleSR/image-service-health-01.png">

<br>
<br>



## 7. その他
稀に、Azure Monitor 製品以外の方法で Azure リソースを監視されているお客様から、検知されたメッセージの意味や発報した原因について調査のご依頼をいただくことがございます。大変恐縮ではございますが、**サードパーティー製の監視製品をご利用されている場合、弊社サポートではその監視の仕組みや発報条件などの詳細を確認することはできません。**

そのため、お問い合わせをご起票いただく前に、お客様ご自身でご利用中の監視ツールの仕組みや発報条件をご確認いただきますようお願いいたします。調査いただいた結果、Azure リソース側に問題がある可能性があると判断された場合には、調査いただいた内容をもとに、Azure リソース観点でお問い合わせをご起票ください。その際、監視ツールの仕組みや発報条件に関する情報もご共有いただけますと、弊社での調査がよりスムーズに進みますので、ご協力のほどよろしくお願いいたします。


<br>