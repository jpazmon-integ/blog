---
title: Application Insights クラシック版とワークスペース版との違いについて
date: 2021-07-20 00:00:00
tags:
  - Application Insights
  - Tips
  - FAQ
---

こんにちは、Azure Monitoring & Integration サポート チームの北山です。  
今回の記事では、Application Insights リソースのクラシック版とワークスペース版との違いについて説明します。

## 目次

- [テレメトリ データの保存先が異なる](#テレメトリ-データの保存先が異なる)
- [テレメトリ データのエクスポート方法が異なる](#テレメトリ-データのエクスポート方法が異なる)
  - [連続エクスポートに関する注意点](#連続エクスポートに関する注意点)
- [データ保有期間の設定場所が異なる](#データ保有期間の設定場所が異なる)
- [その他 よくあるご質問](#その他-よくあるご質問)
  - [Q 1 料金は変わりますか](#Q-1-料金は変わりますか)
  - [Q 2 ログ アラート ルールで使用していた今までのクエリは使えなくなりますか](#Q-2-ログ-アラート-ルールで使用していた今までのクエリは使えなくなりますか)
  - [Q 3 どうやってクラシック版かワークスペース版かを確認すればよいですか](#Q-3-どうやってクラシック版かワークスペース版かを確認すればよいですか)
  - [Q 4 ワークスペース版へ移行すると、今まで使えなくなる機能はありますか](#Q-4-ワークスペース版へ移行すると今まで使えなくなる機能はありますか)
  - [Q 5 ワークスペース版への移行作業中は、テレメトリ データは欠落しますか](#Q-5-ワークスペース版への移行作業中はテレメトリ-データは欠落しますか)

## テレメトリ データの保存先が異なる
クラシック版 Application Insights リソースは、下図のようにテレメトリ データを独自のワークスペースに保存していました。  

![](https://user-images.githubusercontent.com/25476639/126289964-fd294432-a722-43f7-91d2-670b2e4a2b1c.png)



しかしワークスペース版では、Log Analytics ワークスペースに対してテレメトリ データを保存しています。  
そのため、ワークスペース版 Application Insights リソースを作成する場合は、必ず 1 つの Log Analytics ワークスペースも一緒に作成する必要があります。  
(既存 Log Analytics ワークスペースを利用することも可能です)

![](https://user-images.githubusercontent.com/25476639/126290035-ee93423f-8418-4722-88fd-ddc6692c5e90.png)



Log Analytics ワークスペースを利用する事で Log Analytics の機能が使えるようになる点が、ワークスペース版のメリットとなります。

- Log Analytics ワークスペースの容量予約レベルが使えるようになり、従量課金制と比べて最大 30 % のコストを抑えることが可能になります。
- カスタマー マネージド キー (CMK) を使って、ユーザー独自の暗号化キーでデータを暗号化が行えます。
- 診断設定を用いて、ストレージ アカウントやイベント ハブへの転送可能となります。


Log Analytics ワークスペースは、他 Azure サービスのリソース ログの分析や Azure VM や オンプレ環境コンピューターのログ分析、Azure Sentinel などでも利用されている、開発が活発なサービスです。  
そのため、クラシック版では体験出来ないような新機能が、これからもご利用いただけます。  
そのポイントが主なメリットであると思っています。

#### 参考資料
- [ワークスペース版の新機能](https://docs.microsoft.com/ja-jp/azure/azure-monitor/app/create-workspace-resource#new-capabilities)



## テレメトリ データのエクスポート方法が異なる
クラシック版とワークスペース版と比較して、下記のような違いがあります。

|                            | クラシック版   | ワークスペース版 |
|-------------------------------|----------|----------|
| エクスポート方法                      | 連続エクスポート | 診断設定     |
| ストレージ アカウントへのエクスポート           | ○        | ○        |
| (別の) Log Analytics ワークスペースへのエクスポート |          | ○        |
| イベント ハブへのエクスポート               |          | ○        |


クラシック版とワークスペース版と比較してエクスポート方法が異なります。  
そのため、クラシック版からワークスペース版へ移行した場合は、連続エクスポートがご利用出来なくなります。  
移行前には必ず連続エクスポートを機能を無効化していただき、移行後に診断設定を構築していただく必要があるため注意が必要です。


### 連続エクスポートに関する注意点
現時点では連続エクスポートは非推奨となっております。  
また、連続エクスポートに対応していないリージョンも存在するため、もしクラシック版で連続エクスポートをご利用いただく場合は、下記のドキュメントをご一読いただくようお願いしています。

- [Application Insights からのテレメトリのエクスポート](https://docs.microsoft.com/ja-jp/azure/azure-monitor/app/export-telemetry)

### 診断設定に関する注意点
ワークスペース版 Application Insights リソースにて、診断設定を利用して他の Log Analytics ワークスペースへエクスポートした場合、当該 Application Insights リソースからログ検索すると**二重でログが取得される**現象が発生します。  
(Application Insights リソースに紐付いている Log Analytics ワークスペースに格納されたログと、診断設定で指定した他の Log Analytics ワークスペースに格納されたログが取得される)  

現時点では仕様となっております。  
そのため、予め注意していただけますと幸いです。

#### 参考資料
- [Application Insights からのテレメトリのエクスポート (連続エクスポート)](https://docs.microsoft.com/ja-jp/azure/azure-monitor/app/export-telemetry)
- [テレメトリのエクスポート (診断設定)](https://docs.microsoft.com/ja-jp/azure/azure-monitor/app/create-workspace-resource#export-telemetry)
- [ワークスペース版への移行の際の、連続エクスポートに関する注意点](https://docs.microsoft.com/ja-jp/azure/azure-monitor/app/convert-classic-resource#continuous-export)



## データ保有期間の設定場所が異なる
前述のとおり、クラシック版とワークスペース版と比較するとデータ保存先が異なります。  
データ保有期間は保存先に依存しておりますので、設定場所が異なる点注意が必要です。

クラシック版の場合は、下図のように Application Insights リソース ページからデータ保有期間の変更が可能です。

![](https://user-images.githubusercontent.com/25476639/126421433-3e4872d9-f9b0-43cb-92a0-8a6650ca72b9.png)


一方でワークスペース版の場合は、Application Insights リソース ページからは変更出来ません。  
Application Insights リソースに紐付いている Log Analytics ワークスペースにて変更する必要があります。

![](https://user-images.githubusercontent.com/25476639/126421328-301f8c10-5039-432d-9fcc-22d148c22b0a.png)





## その他 よくあるご質問
### Q 1 料金は変わりますか
料金は変わりません。  
また、ワークスペース版の場合に、Application Insights リソースと Log Analytics ワークスペースとで 2 重に費用が発生するのでは? というご質問をよくいただきますが、2 重に費用が発生しませんのでご安心ください。

1 つ注意事項といたしましては、連続エクスポートでは発生していなかったコストが、診断設定の場合は追加で費用が発生します。  
ただ現時点では、下記のドキュメントに記載があるとおり、診断設定によるデータのエクスポートに対する費用は発生しません。  
しかし、将来的に費用が発生する点ご留意願います。

![](https://user-images.githubusercontent.com/25476639/126422733-b6eaddf4-f2aa-4a9c-8a9e-281005a0f04f.png)

- [Azure Monitor の価格](https://azure.microsoft.com/ja-jp/pricing/details/monitor/)


### Q 2 ログ アラート ルールで使用していた今までのクエリは使えなくなりますか
ログベースのアラートについては、引き続き完全な下位互換性を提供します。  
そのため、既存のログ アラート ルールに対してクエリの内容をご変更いただく必要はありません。

詳細は下記のドキュメントに記載があります。

- [ログ クエリについて](https://docs.microsoft.com/ja-jp/azure/azure-monitor/app/convert-classic-resource#understanding-log-queries)

### Q 3 どうやってクラシック版かワークスペース版かを確認すればよいですか
Azure potal にて、当該 Application Insights リソース ページへ移動し、概要ページを参照していただければご確認いただけます。

下図のように Log Analytics ワークスペースのリンクが存在する場合は、ワークスペース版 Application Insights リソースです。

![](https://user-images.githubusercontent.com/25476639/126263222-b890c59b-b672-4ff1-9c1e-4e25e1820c4a.png)


下図のようにワークスペースのリンクがない場合は、クラシック版 Application Insights リソースです。

![](https://user-images.githubusercontent.com/25476639/126263264-379cafa3-9ea1-49f5-8e4c-efdf20812b45.png)


大量に Application Insights リソースが存在して Azure potal から確認するのが大変な場合は、下記のような PowerShell スクリプトを実行いただく事でまとめて確認することが可能です。

#### PowerShell スクリプト
```.ps
Connect-AzAccount
Set-AzContext -SubscriptionID "サブスクリプションID"

$Sub = (Get-AzContext).Subscription.Id

# Set required info for REST API
$Token = (Get-AzAccessToken).Token
$AuthHeader = "Bearer " + $Token
$ReqHeader = @{"Authorization" = $AuthHeader; "Accept" = "application/json"}
$ContType = "application/json"

$AIs = Get-AzApplicationInsights
foreach($AI in $AIs){
 $AIName = $AI.Name
 $AIRG = $AI.ResourceGroupName
 $Uri = "https://management.azure.com/subscriptions/" + $Sub + "/resourcegroups/" + $AIRG + "/providers/microsoft.insights/components/" + $AIName + "?api-version=2018-05-01-preview"
 $ReqResult = Invoke-WebRequest -Uri $Uri -Headers $ReqHeader -Method GET -ContentType $ContType
 $Cont = $ReqResult.Content | ConvertFrom-Json
 $Workspace = $Cont.properties.WorkspaceResourceId
 Write-Output("`nAI Name:" + $AI.Name)
 Write-Output("Workspace Id:" + $Workspace)
 $Workspace = $null
}
```

#### 実行結果
Workspace Id: に Log Analytics ワークスペースのリソース ID が記述されている場合は、ワークスペース版 Application Insights リソースとなります。


```
AI Name:dasasaki-appservice-aspnet48-ampls
Workspace Id:/subscriptions/11b4afdb-3329-42f2-b8dc-b26c5ac19146/resourcegroups/defaultresourcegroup-ejp/providers/microsoft.operationalinsights/workspaces/defaultworkspace-11b4afdb-3329-42f2-b8dc-b26c5ac19146-ejp

AI Name:dasasaki-sparkapp
Workspace Id:

AI Name:dasasaki-csnetcore31
Workspace Id:

AI Name:dasasaki-wsai-jpeast
Workspace Id:/subscriptions/11b4afdb-3329-42f2-b8dc-b26c5ac19146/resourcegroups/dasasaki_rg/providers/microsoft.operationalinsights/workspaces/dasasaki-test

AI Name:dasasaki-aiws-jpeast2
Workspace Id:/subscriptions/11b4afdb-3329-42f2-b8dc-b26c5ac19146/resourcegroups/dasasaki_rg/providers/microsoft.operationalinsights/workspaces/dasasaki-ws-jpeast

AI Name:dasaski-nodejs14-basic-auth
Workspace Id:

AI Name:dasasaki-appservice-aspnet48-ampls-linux
Workspace Id:/subscriptions/11b4afdb-3329-42f2-b8dc-b26c5ac19146/resourcegroups/defaultresourcegroup-ejp/providers/microsoft.operationalinsights/workspaces/defaultworkspace-11b4afdb-3329-42f2-b8dc-b26c5ac19146-ejp

AI Name:dasasaki-netsdk
Workspace Id:

...
```

### Q 4 ワークスペース版へ移行すると、使えなくなる機能はありますか
連続エクスポートがご利用いただけなくなります。  
連続エクスポートの代わりに、診断設定を用いてストレージ アカウントへテレメトリ データを転送する事になる点、ご留意願います。

アプリケーション マップや可用性テスト、トランザクションの検索などといったそれ以外の機能に関しましては、問題なくご利用いただけますのでご安心ください。

### Q 5 ワークスペース版への移行作業中は、テレメトリ データは欠落しますか
テレメトリ データは欠落しませんので、ご安心して移行作業を実施してください。  
ただし、その他に注意点がございますので、移行作業は必ず下記のドキュメントを読みながら実施してください。

- [ワークスペースベースの Application Insights リソースに移行する](https://docs.microsoft.com/ja-jp/azure/azure-monitor/app/convert-classic-resource)

