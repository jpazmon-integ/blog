こんにちは、Azure Monitoring チームの佐藤です！
今回は、Heartbeat ログ アラート ルールが発報したが、マシンは稼働しているという状況、
特に Heartbeat ログの遅延により期待しないアラートの発報が発生した場合の調査方法についてご紹介いたします。

<!-- more -->

# 目次
1. マシンが稼働しているにもかかわらず Heartbeat ログ アラート ルールが発生するシナリオ
2. Heartbeat ログの収集状況を確認する方法
4. Heartbeat ログの遅延が確認されたら

# 1. マシンが稼働しているにもかかわらず Heartbeat ログ アラート ルールが発生するシナリオ
マシンが稼働しているにもかかわらず Heartbeat ログ アラート ルールが発生する原因として、
主に以下がございます。
主な要因 1 : Heartbeat ログの欠損
-> 何らかの原因によりマシンから Heartbeat ログが届かなくなり、それによりアラートの発報条件を満たし発報する。

主な要因 2 : Heartbeat ログの遅延
-> Heartbeat ログの収集に遅延発生したことで、アラートの発報条件を満たし発報する。

主な要因 3 : サービスに問題が発生している状況

今回はこのなかでも、主な要因 2 (遅延) にスポットライトを当ててみます。

# 2. Heartbeat ログの収集遅延状況を確認する方法
以下のクエリにより、まず Heartbeat ログの遅延状況をご確認いただくことが可能です。
(※ たとえば、ログが 30 分遅延して収集されてしまった環境の場合、10:30 に収集されるべきログが Log Analytics ワークスペースに到着するのは 11:00 でございます。
    10:30 ~ 11:00 の間に 10:30 に収集されたログを確認した場合、クエリに該当するレコードがない旨表示されることにご留意ください。)

Log Analytics ワークスペースの各テーブルに存在する以下の 3 列を使用して、収集遅延状況を確認することが可能です。
各列の名称およびどのような値を格納しているかについては以下の通りでございます。​
(列名 : どのような値を格納しているか の順で記載いたします。)

TimeGenerated 列 : エージェントでデータを検知した時刻
_TimeReceived 列 : データが Azure に届いた時刻
ingestion_time() 列 : Log Analytics ワークスペースに取り込まれ、クエリが可能になった時刻

// 参考情報
- インジェスト時間のチェック
https://learn.microsoft.com/ja-jp/azure/azure-monitor/logs/data-ingestion-time#check-ingestion-time

Heartbeat
| where _ResourceId contains "<仮想マシン名>" //<仮想マシン名> は、遅延状況を確認したいログの収集元マシン名に置き換えます
| order by TimeGenerated asc​
| extend E2EIngestionLatency = ingestion_time() - TimeGenerated​ // ①
| extend AgentLatency = _TimeReceived - TimeGenerated​ // ②
| project TimeGenerated, Computer, E2EIngestionLatency, AgentLatency, _TimeReceived, ingestion_time(), Category​
| order by E2EIngestionLatency

① : エージェントがデータを検知してから、データがクエリ可能になるまでの所要時間を計算します
② : エージェントがデータを検知してから、Azure に届くまでの所要時間を計算します

上述のクエリを実行いただくことで、① として計算した値が大きい順に以下の表なテーブル形式でデータが出力されます。


エージェントがデータを検知してからログがクエリ可能な状態に処理されるまで、おおよそ 3 分ほどの時間を要します。
E2EIngestionLatency 列の値をご確認いただき、遅延の有無をご確認ください。

# 3. Heartbeat ログの遅延が確認されたら
2 に記載のクエリにより Heartbeat ログの遅延が確認された場合、以下 2 パターンの遅延の仕方がございます。
パターン 1 : E2EIngestionLatency が大きく、かつ AgentLatency も大きい
パターン 2 : E2EIngestionLatency が大きいが、 AgentLatency は小さい

遅延箇所により遅延の発生原因が異なります。
それぞれのパターンについて、以下に詳述します。

## パターン 1 : E2EIngestionLatency が大きく、かつ AgentLatency も大きい
この場合、エージェントがデータを検知してから Azure に届くまでに時間を要しています。
考えられる主な原因は以下の 2 つでございます。
- ご利用いただいているネットワークの問題
- エージェントで発生している問題

エージェントで問題が発生しているかどうかは、以下の手順でエージェントのログを取得いただくことでご確認いただけます。
### Windows OS の場合
1. 問題が発生した Windows サーバーへ管理者権限にてログインします。
2. PowerShell ウインドウを管理者権限にて開きます。
3. 以下のコマンドを実行します。
cd "C:\Packages\Plugins\Microsoft.Azure.Monitor.AzureMonitorWindowsAgent\1.X.X.X\Troubleshooter"
(X にはバージョン番号が入ります。環境に応じて変更下さい。)

.\AgentTroubleshooter.exe --ama -v

4. ログ採取が行われますのでそのまま待機します。
5. "C:\Packages\Plugins\Microsoft.Azure.Monitor.AzureMonitorWindowsAgent\1.X.X.X\Troubleshooter" フォルダ内 AgentTroubleshooterOutput.zip ファイルが作成されます。
こちらが Azure Monitor エージェントのログでございます。

### Linux OS の場合
対象の Linux OS 上で作業を実施してください。
以下のコマンドを任意のディレクトリ上で実行します。

$ mkdir ama_tst
$ cd ama_tst
$ wget https://github.com/Azure/azure-linux-extensions/raw/master/AzureMonitorAgent/ama_tst/ama_tst.tgz
$ tar -xzvf ama_tst.tgz
$ sudo sh ama_troubleshooter.sh
"Please select an option:" と表示されますので、"L" を入力し、エンターキーを押します。
"Output Directory:" と表示されますので、"." を入力し、エンターキーを押します。

シェルが終了しますと、"amalogs" から始まるディレクトリ、またはファイルがカレントディレクトリに出力されます。
こちらが Azure Monitor エージェントのログでございます。


事象発生直後のほうが、ログに残存している情報が多いため、
弊社サポート窓口にお問い合わせいただく場合でも事象発生後なるべくお早めにログを取得いただくことをお勧めいたします。
ログ ファイルがディレクトリとなっている場合には、弊社にご提供いただきます際には圧縮いただきますようお願いいたします。

また、エージェントのバージョンが最新でない場合には、
過去のエージェント特有の不具合として事象が発生する可能性もございます。
エージェントは不具合の修正などを目的としたアップグレードが行われております。
そのため、エージェントは最新バージョンをご利用いただくことを合わせてご検討ください。

## パターン 2 : E2EIngestionLatency が大きいが、 AgentLatency は小さい
この場合、AgentLatency が小さいことからエージェントがデータを検知してから Azure に届くまでの経路に問題はありません。
一方で、Azure に届いてからクエリ可能になるまでに時間を要しているため E2EIngestionLatency の値が大きくなっています。
この場合、Azure 内部の処理 (受信したデータをクエリ可能となるよう行う処理) に時間がかかっております。

なぜ Azure 内部の処理に時間がかかったか詳細な原因を調査されたい場合には、サポート リクエストを発行いただくことで調査が可能です。

