---
title: Azure Monitor Agent for Linux で取得可能なパフォーマンス カウンターの一覧
date: 2024-01-18 00:00:00
tags:
  - Azure Monitor Agent
  - Log Analytics
---

[更新履歴]  
- 2024/1/18 データ収集ルールで指定できるパフォーマンス カウンターの追加に伴い、取得可能なパフォーマンス カウンターの一覧を更新いたしました。 
- 2023/3/21 ブログ公開

こんにちは、Azure Monitor サポートの三輪です。
今回は Azure Monitor エージェント for Linux にて取得可能なパフォーマンス カウンターの一覧をご案内します。


Azure Monitor エージェントでは、Windows/Linux 各 OS のパフォーマンス データを取得することが可能です。

- Azure Monitor エージェントを使用して仮想マシンからイベントとパフォーマンス カウンターを収集する
https://learn.microsoft.com/ja-JP/azure/azure-monitor/agents/data-collection-rule-azure-monitor-agent?tabs=portal

Azure Monitor エージェント for Windows では、Windows OS 上に存在するパフォーマンス カウンターであれば取得が可能です。
一方で、Azure Monitor エージェント for Linux では、特定のパフォーマンス カウンターのみが取得可能となっています。
また、Azure Monitor エージェント for Linux にて取得可能なパフォーマンス カウンターは、以下の Log Analytics エージェントで取得可能であったパフォーマンス カウンターと一部異なる部分がありますため、本記事にてご案内を致します。

- Log Analytics エージェントを使用して Windows と Linux のパフォーマンス データ ソースを収集する
https://learn.microsoft.com/ja-jp/azure/azure-monitor/agents/data-sources-performance-counters


Azure Monitor エージェント for Linux にて取得可能なパフォーマンス カウンター：
--

| ObjectName | CounterName | 
| :------------- | :------------: | 
| Logical Disk | Logical Disk Bytes/sec | 
| Logical Disk | Free Megabytes | 
| Logical Disk | Disk Writes/sec | 
| Logical Disk | Disk Write Bytes/sec | 
| Logical Disk | Disk Transfers/sec | 
| Logical Disk | Disk Reads/sec | 
| Logical Disk | Disk Read Bytes/sec | 
| Logical Disk | % Used Space | 
| Logical Disk | % Used Inodes | 
| Logical Disk | % Free Space | 
| Logical Disk | % Free Inodes | 
| Memory | Used Memory MBytes | 
| Memory | Used MBytes Swap Space | 
| Memory | Pages/sec | 
| Memory | Page Writes/sec | 
| Memory | Page Reads/sec | 
| Memory | Available MBytes Swap | 
| Memory | Available MBytes Memory | 
| Memory | % Used Swap Space | 
| Memory | % Used Memory | 
| Memory | % Available Swap Space | 
| Memory | % Available Memory | 
| Network | Total Tx Errors | 
| Network | Total Rx Errors | 
| Network | Total Packets Transmitted | 
| Network | Total Packets Received | 
| Network | Total Collisions | 
| Network | Total Bytes Transmitted | 
| Network | Total Bytes Received | 
| Network | Total Bytes | 
| Processor | % User Time | 
| Processor | % Processor Time | 
| Processor | % Privileged Time | 
| Processor | % Nice Time | 
| Processor | % IO Wait Time | 
| Processor | % Interrupt Time | 
| Processor | % Idle Time | 
| Process | Pct User Time |
| Process | Pct Privileged Time |
| Process | Used Memory |
| Process | Virtual Shared Memory |
| System | Uptime |
| System | Load1 |
| System | Load5 |
| System | Load15 |
| System | Users |
| System | Unique Users |
| System | CPUs |
