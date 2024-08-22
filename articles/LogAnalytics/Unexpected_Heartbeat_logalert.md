---
title: マシンが起動中にも関わらず Heartbeat ログ アラートが発報された場合の調査方法
date: 2024-08-29 10:00:00
tags:
  - Azure Monitor Agent
  - Log Analytics
---

こんにちは、Azure Monitoring チームの佐藤です！<br>
今回は、Heartbeat ログ アラート ルールが発報したが、マシンは稼働しているという状況、
特に Heartbeat ログの遅延により期待しないアラートの発報が発生した場合の調査方法についてご紹介いたします。<br>

<!-- more -->

# 目次
1. マシンが稼働しているにもかかわらず Heartbeat ログ アラート ルールが発生するシナリオ
2. Heartbeat ログの収集状況を確認する方法
4. Heartbeat ログの遅延が確認されたら

# 1. マシンが稼働しているにもかかわらず Heartbeat ログ アラート ルールが発生するシナリオ
マシンが稼働しているにもかかわらず Heartbeat ログ アラート ルールが発生する原因として、
主に以下がございます。<br>
主な要因 1 : Heartbeat ログの欠損<br>
-> 何らかの原因によりマシンから Heartbeat ログが届かなくなり、それによりアラートの発報条件を満たし発報する。<br>

主な要因 2 : Heartbeat ログの遅延<br>
-> Heartbeat ログの収集に遅延発生したことで、アラートの発報条件を満たし発報する。<br>

主な要因 3 : サービスに問題が発生している状況<br>

今回はこのなかでも、主な要因 2 (遅延) にスポットライトを当ててみます。<br>

# 2. Heartbeat ログの収集遅延状況を確認する方法
以下のクエリにより、まず Heartbeat ログの遅延状況をご確認いただくことが可能です。<br>
(※ たとえば、ログが 30 分遅延して収集されてしまった環境の場合、10:30 に収集されるべきログが Log Analytics ワークスペースに到着するのは 11:00 でございます。<br>
    10:30 ~ 11:00 の間に 10:30 に収集されたログを確認した場合、クエリに該当するレコードがない旨表示されることにご留意ください。)<br>

Log Analytics ワークスペースの各テーブルに存在する以下の 3 列を使用して、収集遅延状況を確認することが可能です。<br>
各列の名称およびどのような値を格納しているかについては以下の通りでございます。​<br>
(列名 : どのような値を格納しているか の順で記載いたします。)<br>

TimeGenerated 列 : エージェントでデータを検知した時刻<br>
_TimeReceived 列 : データが Azure に届いた時刻<br>
ingestion_time() 列 : Log Analytics ワークスペースに取り込まれ、クエリが可能になった時刻<br>

// 参考情報<br>
- インジェスト時間のチェック<br>
https://learn.microsoft.com/ja-jp/azure/azure-monitor/logs/data-ingestion-time#check-ingestion-time<br>

Heartbeat<br>
| where _ResourceId contains "<仮想マシン名>" //<仮想マシン名> は、遅延状況を確認したいログの収集元マシン名に置き換えます<br>
| order by TimeGenerated asc​<br>
| extend E2EIngestionLatency = ingestion_time() - TimeGenerated​ // ①<br>
| extend AgentLatency = _TimeReceived - TimeGenerated​ // ②<br>
| project TimeGenerated, Computer, E2EIngestionLatency, AgentLatency, _TimeReceived, ingestion_time(), Category​<br>
| order by E2EIngestionLatency<br>
<br>
① : エージェントがデータを検知してから、データがクエリ可能になるまでの所要時間を計算します<br>
② : エージェントがデータを検知してから、Azure に届くまでの所要時間を計算します<br>
<br>
上述のクエリを実行いただくことで、① として計算した値が大きい順に以下の表なテーブル形式でデータが出力されます。

エージェントがデータを検知してからログがクエリ可能な状態に処理されるまで、おおよそ 3 分ほどの時間を要します。<br>
E2EIngestionLatency 列の値をご確認いただき、遅延の有無をご確認ください。<br>

![](./Unexpected_Heartbeat_logalert/image001.png)
例えば、上述のハイライトの場合ですと、以下のようになっております。<br>
<br>
E2EIngestionLatency : 18 分 8 秒<br>
AgentLatency : 18 分 2 秒
<br>
<br>
この場合、AgentLatency が大きいために E2EIngestionLatency の値が大きくなっています。
すなわち、エージェントがデータを検知してから Azure に届くまでの過程で時間を要していることがわかります。

# 3. Heartbeat ログの遅延が確認されたら
2 に記載のクエリにより Heartbeat ログの遅延が確認された場合、以下 2 パターンの遅延の仕方がございます。<br>
パターン 1 : E2EIngestionLatency が大きく、かつ AgentLatency も大きい<br>
パターン 2 : E2EIngestionLatency が大きいが、 AgentLatency は小さい<br>

遅延箇所により遅延の発生原因が異なります。<br>
それぞれのパターンについて、以下に詳述します。<br>

## パターン 1 : E2EIngestionLatency が大きく、かつ AgentLatency も大きい
この場合、エージェントがデータを検知してから Azure に届くまでに時間を要しています。<br>
考えられる主な原因は以下の 2 つでございます。<br>
- ご利用いただいているネットワークの問題<br>
- エージェントで発生している問題<br>

エージェントで問題が発生しているかどうかは、以下の手順でエージェントのログを取得いただくことでご確認いただけます。
### Windows OS の場合
1. 問題が発生した Windows サーバーへ管理者権限にてログインします。<br>
2. PowerShell ウインドウを管理者権限にて開きます。<br>
3. 以下のコマンドを実行します。<br>
cd "C:\Packages\Plugins\Microsoft.Azure.Monitor.AzureMonitorWindowsAgent\1.X.X.X\Troubleshooter"<br>
(X にはバージョン番号が入ります。環境に応じて変更下さい。)<br>
    .\AgentTroubleshooter.exe --ama -v<br>
4. ログ採取が行われますのでそのまま待機します。<br>
5. "C:\Packages\Plugins\Microsoft.Azure.Monitor.AzureMonitorWindowsAgent\1.X.X.X\Troubleshooter" フォルダ内 AgentTroubleshooterOutput.zip ファイルが作成されます。<br>
こちらが Azure Monitor エージェントのログでございます。<br>

### Linux OS の場合
対象の Linux OS 上で作業を実施してください。
以下のコマンドを任意のディレクトリ上で実行します。
<br>
$ mkdir ama_tst<br>
$ cd ama_tst<br>
$ wget https://github.com/Azure/azure-linux-extensions/raw/master/AzureMonitorAgent/ama_tst/ama_tst.tgz<br>
$ tar -xzvf ama_tst.tgz<br>
$ sudo sh ama_troubleshooter.sh<br>
"Please select an option:" と表示されますので、"L" を入力し、エンターキーを押します。<br>
"Output Directory:" と表示されますので、"." を入力し、エンターキーを押します。<br>
<br>
シェルが終了しますと、"amalogs" から始まるディレクトリ、またはファイルがカレントディレクトリに出力されます。<br>
こちらが Azure Monitor エージェントのログでございます。<br>
<br>

事象発生直後のほうが、ログに残存している情報が多いため、
弊社サポート窓口にお問い合わせいただく場合でも事象発生後なるべくお早めにログを取得いただくことをお勧めいたします。<br>
ログ ファイルがディレクトリとなっている場合には、弊社にご提供いただきます際には圧縮いただきますようお願いいたします。<br>

また、エージェントのバージョンが最新でない場合には、
過去のエージェント特有の不具合として事象が発生する可能性もございます。<br>
エージェントは不具合の修正などを目的としたアップグレードが行われております。<br>
そのため、エージェントは最新バージョンをご利用いただくことを合わせてご検討ください。<br>

## パターン 2 : E2EIngestionLatency が大きいが、 AgentLatency は小さい
この場合、AgentLatency が小さいことからエージェントがデータを検知してから Azure に届くまでの経路に問題はありません。<br>
一方で、Azure に届いてからクエリ可能になるまでに時間を要しているため E2EIngestionLatency の値が大きくなっています。<br>
この場合、Azure 内部の処理 (受信したデータをクエリ可能となるよう行う処理) に時間がかかっております。
<br>
なぜ Azure 内部の処理に時間がかかったか詳細な原因を調査されたい場合には、サポート リクエストを発行いただくことで調査が可能です。

