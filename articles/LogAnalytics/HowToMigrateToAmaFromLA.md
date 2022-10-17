---
title: Log Analytics エージェントから Azure Monitor エージェントへの移行に関するよくあるご質問集
date: 2022-10-18 00:00:00
tags:
  - How-To
  - FAQ
  - Log Analytics
  - Azure Monitor
  - 移行
---

こんにちは、Azure Monitoring サポート チームの北村です。

Log Analytics エージェントが 2024 年 8 月に廃止することに伴い、Azure Monitor エージェントへの移行に関するお問い合わせをよくいただいております。
そこで、本記事では Azure Monitor エージェントへの移行に関するよくあるご質問を Q & A 形式でおまとめいたしました。
これから Azure Monitor エージェントへ移行される方、Azure Monitor エージェントの導入をご検討されている方の一助となれば幸いです。

<br>

<!-- more -->

## Q & A タイトル
- [Q1. Log Analytics エージェントは 2024 年 8 月に廃止されますが、Log Analytics ワークスペースも廃止されますか。](#Q1-Log-Analytics-エージェントは-2024-年-8-月に廃止されますが、Log-Analytics-ワークスペースも廃止されますか。)
- [Q2. Azure Monitor エージェントへ移行した後も、既存のアラート ルールを利用することはできますか。](#Q2-Azure-Monitor-エージェントへ移行した後も、既存のアラート-ルールを利用することはできますか。)
- [Q3. 同一のマシン上で Log Analytics エージェントと Azure Monitor エージェントを稼働させることは可能ですか。](#Q3-同一のマシン上で-Log-Analytics-エージェントと-Azure-Monitor-エージェントを稼働させることは可能ですか。)
- [Q4. Azure Monitor エージェントでは、Log Analytics エージェントと同等の機能が提供されていますか。](#Q4-Azure-Monitor-エージェントでは、Log-Analytics-エージェントと同等の機能が提供されていますか。)
- [Q5. Azure Montior エージェントを利用する上での前提条件とサポートしている OS を教えてください。](#Q5-Azure-Montior-エージェントを利用する上での前提条件とサポートしている-OS-を教えてください。)
- [Q6. Azure Monitor エージェントと Log Analytics エージェントの通信要件は同じですか。](#Q6-Azure-Monitor-エージェントと-Log-Analytics-エージェントの通信要件は同じですか。)
- [Q7. Log Analytics エージェントと Azure Monitor エージェントを同時に稼働しています。Azure Monitor エージェントでログが収集されていることを確認する方法を教えてください。](#Q7-Log-Analytics-エージェントと-Azure-Monitor-エージェントを同時に稼働しています。Azure-Monitor-エージェントでログが収集されていることを確認する方法を教えてください。)
- [Q8. 仮想マシンに Log Analytics エージェントがインストールされています。Azure Monitor エージェントへの移行手順を教えてください。](#Q8-仮想マシンに-Log-Analytics-エージェントがインストールされています。Azure-Monitor-エージェントへの移行手順を教えてください。)
- [Q9. データ収集ルールを作成し、Azure Monitor エージェントがインストールされ、ログが収集されていることを確認しました。しかし、Log Analytics ワークスペースの [ワークスペースのデータ ソース] - [仮想マシン] を見ると、”接続されていません” と表示されます](#Q9-データ収集ルールを作成し、Azure-Monitor-エージェントがインストールされ、ログが収集されていることを確認しました。しかし、Log-Analytics-ワークスペースの-ワークスペースのデータ-ソース-仮想マシン-を見ると、”接続されていません”-と表示されます。)
- [Q10. 仮想マシンを作成したとき、 Azure Monitor エージェントのインストール、および既存のデータ収集ルールとの関連付けを自動で実施する方法はありますか。](#Q10-仮想マシンを作成したとき、-Azure-Monitor-エージェントのインストール、および既存のデータ収集ルールとの関連付けを自動で実施する方法はありますか。)
- [Q11. 閉じたネットワーク環境でコンピューターから Log Analytics ワークスペースへログを送信したいです。Azure Monitor エージェントでこのような事が実現可能ですか。](#Q11-閉じたネットワーク環境でコンピューターから-Log-Analytics-ワークスペースへログを送信したいです。Azure-Monitor-エージェントでこのような事が実現可能ですか。)

<br>

### Q1. Log Analytics エージェントは 2024 年 8 月に廃止されますが、Log Analytics ワークスペースも廃止されますか。
いいえ、Log Analytics エージェントは廃止されますが、Log Analytics ワークスペースは廃止されません。

<br>

### Q2. Azure Monitor エージェントへ移行した後も、既存のアラート ルールを利用することはできますか。
はい、Azure Monitor エージェントへ移行した後も既存のアラート ルールをご利用いただけます。
ログ収集先の Log Analytics ワークスペースを変更する場合はアラート ルールの再作成が必要ですが、
同一のワークスペースへログを収集する場合はそのままご利用いただけます。

<br>

### Q3. 同一のマシン上で Log Analytics エージェントと Azure Monitor エージェントを稼働させることは可能ですか。
はい、可能です。
ただし、データが重複して収集される可能性もございますので、その点ご留意ください。
詳細は [Log Analytics エージェントから Azure Monitor エージェントへの移行](https://learn.microsoft.com/ja-jp/azure/azure-monitor/agents/azure-monitor-agent-migration#migration-plan-considerations) の [移行計画に関する考慮事項] をご覧ください。

<br>

### Q4. Azure Monitor エージェントでは、Log Analytics エージェントと同等の機能が提供されていますか。
いいえ、現時点では Log Analytics エージェントの全ての機能を Azure Monitor エージェントで提供しておりません。
最終的には、Azure Monitor エージェントは Log Analytics エージェント、Azure Diagnostics 拡張機能、および Telegraf エージェントの機能を提供する予定でございます。
Azure Monitor エージェントと Log Analytics エージェントにおける機能の違いにつきましては、[Azure Monitor エージェントの概要](https://learn.microsoft.com/ja-jp/azure/azure-monitor/agents/agents-overview#compare-to-legacy-agents) の [レガシ エージェントとの比較] をご覧ください。

<br>

### Q5. Azure Montior エージェントを利用する上での前提条件とサポートしている OS を教えてください。
Azure Montior エージェントをご利用する上での前提条件は [Azure Monitor エージェントを管理する](https://learn.microsoft.com/ja-jp/azure/azure-monitor/agents/azure-monitor-agent-manage?tabs=ARMAgentPowerShell%2CPowerShellWindows%2CPowerShellWindowsArc%2CCLIWindows%2CCLIWindowsArc#prerequisites) の [前提条件] をご覧ください。
また、当該エージェントがサポートしている OS は [Azure Monitor エージェントの概要](https://learn.microsoft.com/ja-jp/azure/azure-monitor/agents/agents-overview#supported-operating-systems) の [サポートされるオペレーティング システム] の表に記載されている OS かつ、対象列に "X" と記載されているものでございます。

<br>

### Q6. Azure Monitor エージェントと Log Analytics エージェントの通信要件は同じですか。
いいえ、Log Analytics エージェントの通信要件と Azure Monitor エージェントの通信要件は異なります。
Azure Monitor エージェントからワークスペースへデータを送信するには、下記 URL に対して TCP 443 ポートへのアクセスが確保されている必要がございます。

- global.handler.control.monitor.azure.com 
- &lt;virtual-machine-region-name&gt;.handler.control.monitor.azure.com (例: japaneast.handler.control.azure.com)
- &lt;log-analytics-workspace-id&gt;.ods.opinsights.azure.com (例: 12345a01-b1cd-1234-e1f2-1234567g8h99.ods.opsinsights.azure.com)


また、NSG を利用する場合には、AzureMonitor と AzureResourceManager のサービス タグに対して、送信方向にアクセスを許可していただく必要がございます。詳細は [Azure Monitor エージェントのネットワーク設定を定義する](https://learn.microsoft.com/ja-jp/azure/azure-monitor/agents/azure-monitor-agent-data-collection-endpoint?tabs=PowerShellWindows) をご覧ください。

一方で、カスタム ログを収集する場合や [Azure Monitor Private Link Scope (AMPLS)](https://learn.microsoft.com/ja-jp/azure/azure-monitor/logs/private-link-security) を用いて閉域網にてログを収集する場合、データ収集エンドポイント (DCE) を別途ご構築いただく必要がございます。
その場合、DCE をご利用いただくための通信要件を別途考慮いただく必要がございます。
DCE をご利用いただく場合は、下記 URL に対して TCP 443 ポートへのアクセスが確保されている必要がある点について、予めご留意ください。

- &lt;unique-dce-identifier&gt;.&lt;regionname&gt;.handler.control.monitor.azure.com
- &lt;unique-dce-identifier&gt;.&lt;regionname&gt;.ingest.monitor.azure.com

DCE に関する通信要件につきましては、[データ収集エンドポイントのコンポーネント](https://learn.microsoft.com/ja-jp/azure/azure-monitor/essentials/data-collection-endpoint-overview?tabs=portal#components-of-a-data-collection-endpoint) をご確認ください。


<br>


### Q7. Log Analytics エージェントと Azure Monitor エージェントを同時に稼働しています。Azure Monitor エージェントでログが収集されていることを確認する方法を教えてください。
対象のワークスペースで下記クエリを実行してください。
データ収集ルールを作成し、Azure Monitor エージェントがインストールされますと、Heartbeat が既定で収集されます。
Azure Monitor エージェントで収集される Heartbeat の Category は "Azure Monitor Agent" です。

```
Heartbeat
| where Computer == 'computer-name'
| where Category == 'Azure Monitor Agent'
```

<br>

### Q8. 仮想マシンに Log Analytics エージェントがインストールされています。Azure Monitor エージェントへの移行手順を教えてください。
移行の一例を示します。
以下例では、Azure Monitor エージェントでログが収集されていることを確認し、Log Analytics エージェントをアンインストールしています。移行手順につきましては、[Log Analytics エージェントから Azure Monitor エージェントへの移行](https://learn.microsoft.com/ja-jp/azure/azure-monitor/agents/azure-monitor-agent-migration#migration-testing) もあわせてご覧ください。

1. [データ収集ルールを作成し](https://learn.microsoft.com/ja-jp/azure/azure-monitor/essentials/data-collection-rule-overview#create-a-data-collection-rule)、Azure Monitor エージェントをインストールする
2. Azure Monitor エージェントで任意のログが収集できていることを確認する
3. Log Analytics エージェントのアンインストール

> [!WARNING]
> 上記手順は、Log Analytics エージェント以外のエージェント (Dependency エージェント) や他製品で Log Analytics エージェントを使用していない場合の手順でございます。Microsoft Defender for Cloud や Microsoft Sentinel 等、他製品で Log Analytics エージェントを利用されている場合は、移行前に [Azure Monitor エージェントでサポートされているサービスと機能](https://learn.microsoft.com/ja-jp/azure/azure-monitor/agents/agents-overview#supported-services-and-features) で対象製品が Azure Monitor エージェントに対応していることをご確認ください。また、Dependency エージェントをご利用されている場合は、[VM insights の有効化の概要](https://learn.microsoft.com/ja-jp/azure/azure-monitor/vm/vminsights-enable-overview) や、[Azure portal で VM insights を有効にする](https://learn.microsoft.com/ja-jp/azure/azure-monitor/vm/vminsights-enable-portal) をご確認いただき、移行を実施してください。

<br>

### Q9. データ収集ルールを作成し、Azure Monitor エージェントがインストールされ、ログが収集されていることを確認しました。しかし、Log Analytics ワークスペースの [ワークスペースのデータ ソース] - [仮想マシン] を見ると、"接続されていません" と表示されます。
Log Analytics ワークスペースの [ワークスペースのデータ ソース] - [仮想マシン] に表示される "Log Analytics 接続状況" は、
Log Analytics エージェントがワークスペースに接続されていることを意味します。
Azure Monitor エージェントが対象のワークスペースと接続している状況であることを意味するものではございません。

<br>

### Q10. 仮想マシンを作成したとき、 Azure Monitor エージェントのインストール、および既存のデータ収集ルールとの関連付けを自動で実施する方法はありますか。
はい、Azure Policy の機能を利用することで実現できます。
設定手順につきましては、弊社サポート ブログ [仮想マシン作成時に Azure Monitor エージェントのインストールとデータ収集ルールへの関連付けを自動で行う方法](https://jpazmon-integ.github.io/blog/LogAnalytics/automatically_install_ama/) をご覧ください。

<br>


### Q11. 閉じたネットワーク環境でコンピューターから Log Analytics ワークスペースへログを送信したいです。Azure Monitor エージェントでこのような事が実現可能ですか。
はい、Log Analytics エージェントと同様に、[Azure Monitor Private Link Scope (AMPLS)](https://learn.microsoft.com/ja-jp/azure/azure-monitor/logs/private-link-security) をご利用いただくことで実現可能です。
AMPLS 環境下で Azure Monitor エージェントを用いてログを Log Analytics ワークスペースへ送信するためには、別途データ収集エンドポイント (DCE) のご構築が必要となります。
AMPLS を用いたログの収集方法につきましては、[Azure Monitor エージェントのネットワークの分離を有効にする](https://learn.microsoft.com/ja-jp/azure/azure-monitor/agents/azure-monitor-agent-data-collection-endpoint?tabs=PowerShellWindows#enable-network-isolation-for-the-azure-monitor-agent) の公開情報をご参考ください。

<br>

上記の内容以外でご不明な点や疑問点などございましたら、弊社サポート サービスまでお問い合わせください。
最後までお読みいただきありがとうございました！