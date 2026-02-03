---
title: CMA/AMA とエンドポイントの通信経路上に Azure Firewall がある場合の注意事項
date: 2025-01-05 21:00:00
tags:
  - Log Analytics
  - Azure Arc
  - Azure Monitor Agent
  - Arc-enabled servers
  - HowTo
---

[更新履歴]
- 2025/01/05 ブログ公開
- 2025/12/22 最新情報への更新

<!-- more -->
皆様こんにちは、Azure Monitoring チームの佐藤です。
本日は Azure Arc や Azure Monitor サービスを使用いただく際に監視、管理対象マシンにインストールする以下エージェントが Azure のエンドポイント（ログなどの送り先）と通信する際の途中経路に Azure Firewall (以後、AZFW) がある場合、留意すべきポイントについてご紹介します。

| 略称  | エージェント名  | 用途  | 
| ------------ | ------------ | ------------ |
| CMA | Connected Machine Agent  |   Azure Arc サーバーとして管理する。Azure 基盤と通信する |
| AMA | Azure Monitor Agent  | ログ採取、採取したログを Log Analytics ワークスペースなどログ格納先に送る  |

目次


### 対象マシンからエンドポイントへ Test-NetConnection コマンドで疎通確認する場合
以下 blog で当社 AZFW を所管しているチームにて公開しておりますとおりエンドポイントへ疎通できてなくとも "TcpTestSucceeded : True" の応答を得る点にご注意ください。
[Azure Firewall の各ルールの動作について](https://jpaztech.github.io/blog/network/firewall-rules/)

//内容抜粋
HTTP (80), HTTPS (443) の通信において、ネットワークルール、アプリケーションルールで明示的に拒否していない場合、ルールの処理順序に従って、Azure Firewall の既定の動作として、アプリケーションルールで拒否されることが想定される動作となります。
この動作について **Windows の Test-Netconnection** や Linux の curl, nc コマンド等 (以下に記載) で **TCP の 3way handshake による接続確認を実施した場合はアプリケーションルールで許可していないにもかかわらず、3way handshake が確立され、許可されたように見える動作**となります。


上記シナリオに合致する場合、 Test-NetConnection と tls コネクションまで実施する curl コマンド（※）の結果は以下のように違い発生します。

◆Test-NetConnection コマンドの実行例
PS C:\Users\azureuser> Test-NetConnection global.handler.control.monitor.azure.com -port 443                                                                                                                  
ComputerName     : global.handler.control.monitor.azure.com
RemoteAddress    : 20.43.70.102
RemotePort       : 443
InterfaceAlias   : Ethernet 3
SourceAddress    : 10.10.94.69
TcpTestSucceeded : True

◆ curl コマンドの実行例（curl.output.txt より）
Starting external process: C:\Windows\system32\curl.exe -v -s -S -k https://global.handler.control.monitor.azure.com/ping
Some arguments have been redacted for security reasons.
* Host global.handler.control.monitor.azure.com:443 was resolved.
* IPv6: (none)
* IPv4: 20.43.70.102
*   Trying 20.43.70.102:443...
* Connected to global.handler.control.monitor.azure.com (20.43.70.102) port 443
* schannel: disabled automatic use of client certificate
* ALPN: curl offers http/1.1
* schannel: **failed to receive handshake, SSL/TLS connection failed**
* closing connection #0

※今回は AMA の トラブルシューティング ツールで取得した curl コマンド結果を使用いたします。
[Azure Monitor エージェント トラブルシューティング ツールの使用方法](https://learn.microsoft.com/ja-jp/azure/azure-monitor/agents/troubleshooter-ama-windows?tabs=WindowsPowerShell)

Test-NetConnection コマンドで疎通できているように見える場合でも実態としては、AZFW で疎通が許可されていない場合は上記のようになる点ご留意ください。
その他上記の結果は TLS コネクションをはるための暗号化スイートが要件を満たしていない可能性もございますが、既定では暗号化スイートは基本満たしていることがほとんどです。
新しい Windows Server 構築して上記結果になる場合は一度 AZFW の疎通許可を確認いただけますと幸いです。