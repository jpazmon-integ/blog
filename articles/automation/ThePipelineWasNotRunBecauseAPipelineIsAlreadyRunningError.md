---
title: "The pipeline was not run because a pipeline is already running" エラー発生時の対処方法
date: 2021-8-20 12:00:00
tags:
  - Automation
  - Job
  - Error
  - StartAzureV2VMs
  - StopAzureV2VMs
  - Troubleshooting
  - FAQ
---
こんにちは、Azure Monitoring & Integration サポート チームの趙です。  
本日は、Azure Automation の Runbook 実行時、"The pipeline was not run because a pipeline is already running" のエラーでジョブが失敗する場合の対処方法をご紹介していきます。

<!-- more -->

## 目次
- 事象
- 原因
- 対策

## 事象
### この事象はどういう事象ですか？
以下エラーで、ジョブが失敗する事象です。

      The pipeline was not run because a pipeline is already running. Pipelines cannot be run concurrently.
### このエラーはいつ発生しますか？
**同時に複数のジョブを実行**する際に、タイミングによっては、当該エラーが表示され、ジョブが失敗する場合がございます。

> [!NOTE]
> 特にグラフィック Runbook **Start Azure V2 VMs**、**Stop Azure V2 VMs** を利用しているお客様から当該エラーのお問い合わせを複数いただいております。  
> こちらの Runbook をご利用されている方は、後述する対策をご検討いただけますと幸いです。  

> Runbook **Start Azure V2 VMs** と **Stop Azure V2 VMs**  
> ![](./ThePipelineWasNotRunBecauseAPipelineIsAlreadyRunningError/startazurev2vmsstopazurev2vms.png)

## 原因
以下 Automation ジョブの動作により、発生する事象です。
### Automation ジョブの動作
Automation では、Runbook を実行すると、 Azure 実行基盤上にワーカーというプロセスが作成されます。  
このワーカーに Runbook のジョブが割り当てられ (ジョブ 1 : ワーカー 1) 、実行されます。  
一方で、Runbook による複数のジョブが同一のワーカーにて (ジョブ N : ワーカー 1) 実行されてしまう場合があります。  
このため、タイミングによっては、該当のエラーメッセージが表示され、ジョブが失敗することがございます。  

<!--reviewrequired
[!NOTE]
複数のお客様から、この動作が修正される予定があるかとのお問い合わせを複数いただいておりますが、  
こちらの動作は、より多くのお客様が Automation サービスをご利用できるように  
> Azure 実行基盤上のリソースを効率良く割り当てるための仕組みであり、  
> 今後、修正される見込みはないこと、ご了承いただければ幸いです。
-->

### 図解
![](./ThePipelineWasNotRunBecauseAPipelineIsAlreadyRunningError/pipeline2.png)


## 対策

以下３つの対策の中でいずれかの対策のご利用をご検討いただけますと幸いです。

1. 起動する VM ごとに、 Automation アカウントを分ける
2. スケジュールの時間を数分ずらして実行する
3. Start/Stop VMs during off-hours ソリューションを利用する

### 1. 起動する VM ごとに、 Automation アカウントを分ける
ジョブを実行した Automation アカウントが同一の場合にのみ、  
Runbook のジョブを実行するワーカーが重複する可能性があります。  
回避策として、起動する VM ごとに、Automation アカウントを分ける対応をご検討いただければ幸いです。

例えば、VM を起動する Runbook にて、当該エラーが発生した場合、
以下例のように VM ごとに Automation アカウントを分けて Runbook を実行する案をご検討いただけますと幸いです。 

例)
| 対象 VM | Automation アカウント |
| ---- | ---- |
| VM01 | AutomationAccount-A |
| VM02 | AutomationAccount-B |
| VM03 | AutomationAccount-C |
| VM04 | AutomationAccount-D |

### 2. スケジュールの時間を数分ずらして実行する

同時に複数のジョブを実行する場合、  
Runbook のジョブを実行するワーカーが重複する可能性があります。  
回避策として、ジョブの同時実行を回避する対応をご検討いただければ幸いです。  

例えば、VM を起動する Runbook にて、当該エラーが発生した場合、  
以下例のようにスケジュールの時間をずらして、VM を起動する案をご検討いただけますと幸いです。  
ジョブ間の間隔を 2 ~ 3 分程度、空けていただくことを目安と設定いただければ幸いです。

例)
| 対象 VM | ジョブを実行する時刻 (スケジュール) | Automation アカウント |
| ---- | ---- |---- |
| VM01 | 2021-08-20 08:00 | AutomationAccount-A
| VM02 | 2021-08-20 08:02 | AutomationAccount-A
| VM03 | 2021-08-20 08:05 | AutomationAccount-A
| VM04 | 2021-08-20 08:07 | AutomationAccount-A

### 3. Start/Stop VMs during off-hours ソリューションを利用する

[Start/Stop VMs during off-hours ソリューション](https://docs.microsoft.com/ja-jp/azure/automation/automation-solution-vm-management)では、上記のような動作 (一つのワーカーに複数のジョブがアサインされる動作) が回避できる仕組みがスクリプト レベルで実装されております。  
なお、Start/Stop VMs during off-hours ソリューションをご利用いただくことで、VM の開始および停止をスケジュールで実行することができます。  
回避策として、Start/Stop VMs during off-hours ソリューションを利用する案をご検討いただければ幸いです。

[Start/Stop VMs during off-hours ソリューション v1](https://docs.microsoft.com/ja-jp/azure/automation/automation-solution-vm-management)は、2022 年 5 月 21 日に非推奨となる予定です。  
そのため、今後は、[Start/Stop VMs during off-hours ソリューション v2](https://docs.microsoft.com/ja-jp/azure/azure-functions/start-stop-vms/overview) のご利用をご検討いただけますと幸いです。
