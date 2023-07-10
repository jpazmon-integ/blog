[更新履歴]
7/10 10:00 ブログ公開

こんにちは、Azure Monitoring サポート チームの横山です。  
7/7 に発生した障害 XMGF-5Z0 について記載いたします。
この度は弊社 Azure の障害によりご不便ご迷惑をおかけし、大変申し訳ございませんでした。

# 目次
[発生した事象と時間帯](#summary)  
[事象の発生したリージョン](#region)  
[直接影響を受けたサービス](#impact1)  
[副次的に影響を受けたサービス](#impact2)  
[影響を受けなかったサービス](#impact3)  
[よくあるご質問](#FAQ)  
[最後に](#atLast)

<a id="summary"></a>
# 発生した事象と時間帯
日本時間 7/7 8:15 - 18:00 の間、Azure Monitor 関連サービスにおけるデータの取り込みに欠損ならびに遅延が発生いたしました。  
具体的には、問題のある構成により過剰な数の処理が発生しいたしました。これにより、ログの取り込みを行う Azure 基盤側のサービスが高負荷状態に陥り、内部で利用する認証情報の取得に失敗する事象が発生いたしました。  

Azure 基盤側のサーバー増強を行い、滞留していた処理と新規の処理を適切に処理できるよう対処いたしました。  

根本原因ならびに今後の恒久対策については現在も弊社開発にて調査、検討中です。(7/10 10:00 現在)  
なお、最新の状況は以下よりご確認いただけます。  
https://app.azure.com/h/XMGF-5Z0/ (Azure ポータル)  
https://azure.status.microsoft/ja-jp/status/history/ (Azure の状態ページ)

<a id="region"></a>
# 発生したリージョン
東日本、西日本を含む以下のリージョンで事象が発生いたしました。  
Global; Australia Central; Australia Central 2; Australia East; Australia Southeast; Brazil South; Brazil Southeast; Canada Central; Canada East; Central India; Central US; Central US EUAP; East Asia; East US; East US 2; East US 2 EUAP; France Central; France South; Germany North; Germany West Central; Japan East; Japan West; Jio India Central; Jio India West; Korea Central; Korea South; North Central US; North Europe; Norway East; Norway West; Qatar Central; South Africa North; South Africa West; South Central US; South India; Southeast Asia; Sweden Central; Sweden South; Switzerland North; Switzerland West; UK South; UK West; West Central US; West Europe; West India; West US; West US 2; West US 3

<a id="impact1"></a>
# 直接影響を受けたサービス
- Log Analytics エージェント、Azure Monitor エージェント、Container Insights、データ コレクタ API、ログ インジェスト API を利用した Log Analytics ワークスペースへのデータ取り込み
- Microsoft Sentinel コネクタを利用した Log Log Analytics ワークスペースへのデータ取り込み
- Application Insights (ワークスペース ベース) へのデータ取り込み
- リソースの診断設定を利用した Log Analytics ワークスペース、ストレージ アカウント、イベント ハブへのデータ取り込み

<a id="impact2"></a>
# 副次的に影響を受けたサービス
- Log Analytics ワークスペースへのクエリを利用したログ アラート ルール
- Log Analytics ワークスペースをスコープとしたメトリック アラート ルール
- Log Analytics ワークスペースからストレージ アカウント、イベント ハブへのデータ エクスポート

<a id="impact3"></a>
# 影響を受けなかったサービス
- Log Analytics ワークスペース、Sentinel、Application Insights (ワークスペース ベース) に取り込まれたデータの参照
- Application Insights (クラシック) へのデータ取り込み
- Windows Azure Diagnostics (WAD)、Linux Azure Diagnostics (LAD) を利用したストレージ アカウントへのデータ取り込み
- 各種リソースのプラットフォーム メトリック
- プラットフォーム メトリックを利用したメトリック アラート ルール

<a id="FAQ"></a>
# よくあるご質問
Q. 影響を受けたサブスクリプションの確認方法はあるか？  
A. 以下に追跡 ID: XMGF-5Z0 が表示される場合、影響を受けております。  
[Azure ポータル] - [サービス正常性] - [サービス正常性の履歴]

Q. データの欠損はあるか？  
A. はい、今回の障害によりデータの欠損は発生していた可能性がございます。  

Q. データの欠損を確認する方法はあるか？  
A. もし、取り込み元のログ (イベント ログや Syslog) をお客様にて確認できる場合は、対象の取り込み先のデータと比較することで可能です。残念ながら、マイクロソフトから欠損したデータを確認することはできません。

Q. 欠損したデータの復旧は可能か？  
A. いいえ、欠損したデータの復旧を行うことはできません。

Q. 今回の障害による返金はあるか？  
A. いいえ、Azure Monitor 各種サービスへのデータ取り込みに関しては SLA (Service Level Agreements) が定められていないため、今回の障害による返金はございません。

<a id="atlast"></a>
# 最後に
もし、本ブログに記載の内容以外について追加のご質問やご不明点がございましたら、大変お手数ではございますが、事象を受けたサブスクリプションからお問い合わせをご発行ください。  
なお、以下についてはお問い合わせを新規ご発行いただいても回答いたしかねますので、予めご了承ください。  
* XMGF-5Z0 に記載されている障害の詳細、ならびに、Azure Monitor に関する内部実装  
* 根本原因や恒久対策について (XMGF-5Z0 を介して開発からアップデートがございますので、そちらをお待ち下さい。弊ブログも開発からの情報公開後、アップデートいたします。)

改めまして、この度は弊社 Azure 障害によりご不便ご迷惑をおかけし、大変申し訳ございませんでした。