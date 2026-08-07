---
title: ContainerLog テーブルのサポート終了に伴う ContainerLogV2 テーブルへの移行について
date: 2026-08-07 00:00:00

tags:
 - Log Analytics
 - Container Insights
 - Retirement
---

[更新履歴]
- 2026/08/07 ブログ公開

こんにちは！Azure Monitoring チームの佐藤です。

本記事は、2026 年 9 月 30 日にサポートを終了いたします ContainerLog テーブルについて、後継の ContainerLogV2 テーブルへ移行いただく方法について案内しております。

<!-- more -->
## 目次
- [1. 本記事の概要](https://github.com/risaakita/blog-1/blob/main/articles/LogAnalytics/MigrationToContainerLogV2.md#1-%E6%9C%AC%E8%A8%98%E4%BA%8B%E3%81%AE%E6%A6%82%E8%A6%81)
- [2. 移行の前に認証方法をご確認ください](https://github.com/risaakita/blog-1/blob/main/articles/LogAnalytics/MigrationToContainerLogV2.md#2-%E7%A7%BB%E8%A1%8C%E3%81%AE%E5%89%8D%E3%81%AB%E3%81%94%E7%A2%BA%E8%AA%8D%E3%81%8F%E3%81%A0%E3%81%95%E3%81%84)
  - [2-1. レガシ認証の使用有無](https://github.com/risaakita/blog-1/blob/main/articles/LogAnalytics/MigrationToContainerLogV2.md#2-1-%E3%83%AC%E3%82%AC%E3%82%B7%E8%AA%8D%E8%A8%BC%E3%81%AE%E4%BD%BF%E7%94%A8%E6%9C%89%E7%84%A1)
  - [2-2. 該当する移行シナリオはどれか](https://github.com/risaakita/blog-1/blob/main/articles/LogAnalytics/MigrationToContainerLogV2.md#2-2-%E8%A9%B2%E5%BD%93%E3%81%99%E3%82%8B%E7%A7%BB%E8%A1%8C%E3%82%B7%E3%83%8A%E3%83%AA%E3%82%AA%E3%81%AF%E3%81%A9%E3%82%8C%E3%81%8B)
  - [2-3. 移行に伴う注意点](https://github.com/risaakita/blog-1/blob/main/articles/LogAnalytics/MigrationToContainerLogV2.md#2-3-%E7%A7%BB%E8%A1%8C%E3%81%AB%E4%BC%B4%E3%81%86%E6%B3%A8%E6%84%8F%E7%82%B9)
- [3. レガシ認証を使用した Container Insights の場合の移行方法](https://github.com/risaakita/blog-1/blob/main/articles/LogAnalytics/MigrationToContainerLogV2.md#3-%E3%83%AC%E3%82%AC%E3%82%B7%E8%AA%8D%E8%A8%BC%E3%82%92%E4%BD%BF%E7%94%A8%E3%81%97%E3%81%9F-container-insights-%E3%81%AE%E5%A0%B4%E5%90%88%E3%81%AE%E7%A7%BB%E8%A1%8C%E6%96%B9%E6%B3%95)
- [4. マネージド ID 認証を使用した Container Insights の場合の移行方法](https://github.com/risaakita/blog-1/blob/main/articles/LogAnalytics/MigrationToContainerLogV2.md#4-%E3%83%9E%E3%83%8D%E3%83%BC%E3%82%B8%E3%83%89-id-%E8%AA%8D%E8%A8%BC%E3%82%92%E4%BD%BF%E7%94%A8%E3%81%97%E3%81%9F-container-insights-%E3%81%AE%E5%A0%B4%E5%90%88%E3%81%AE%E7%A7%BB%E8%A1%8C%E6%96%B9%E6%B3%95)
  - [4-1. ConfigMap の yaml を編集することにより移行する](https://github.com/risaakita/blog-1/blob/main/articles/LogAnalytics/MigrationToContainerLogV2.md#4-1-configmap-%E3%81%AE-yaml-%E3%82%92%E7%B7%A8%E9%9B%86%E3%81%99%E3%82%8B%E3%81%93%E3%81%A8%E3%81%AB%E3%82%88%E3%82%8A%E7%A7%BB%E8%A1%8C%E3%81%99%E3%82%8B)
  - [4-2. ConfigMap を有効化することにより移行する](https://github.com/risaakita/blog-1/blob/main/articles/LogAnalytics/MigrationToContainerLogV2.md#4-2-configmap-%E3%82%92%E6%9C%89%E5%8A%B9%E5%8C%96%E3%81%99%E3%82%8B%E3%81%93%E3%81%A8%E3%81%AB%E3%82%88%E3%82%8A%E7%A7%BB%E8%A1%8C%E3%81%99%E3%82%8B)
  - [4-3. ログ プロファイルを設定することにより移行する](https://github.com/risaakita/blog-1/blob/main/articles/LogAnalytics/MigrationToContainerLogV2.md#4-3-%E3%83%AD%E3%82%B0-%E3%83%97%E3%83%AD%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E3%82%92%E8%A8%AD%E5%AE%9A%E3%81%99%E3%82%8B%E3%81%93%E3%81%A8%E3%81%AB%E3%82%88%E3%82%8A%E7%A7%BB%E8%A1%8C%E3%81%99%E3%82%8B)
- [5. まとめ](https://github.com/risaakita/blog-1/blob/main/articles/LogAnalytics/MigrationToContainerLogV2.md#5-%E3%81%BE%E3%81%A8%E3%82%81)


<br>

## 1. 本記事の概要
以下の公開情報に記載があります通り、ContainerLog テーブルの使用が 2026 年 9 月 30 日をもってサポートされなくなります。
- Container Insights のログ スキーマ  
https://learn.microsoft.com/ja-jp/azure/azure-monitor/containers/container-insights-logs-schema
![](./MigrationToContainerLogV2/announce01.png)
  
サポート終了に伴い、ContainerLog テーブルをご利用いただいている場合には後継の ContainerLogV2 テーブルへ移行をいただく必要がございます。  
本記事では、Azure Kubernetes Service (AKS) リソースで利用されていることを前提とした移行方法について、いくつか具体的なシナリオを挙げて紹介いたします。

## 2. 移行の前にご確認ください
### 2-1. レガシ認証の使用有無
シナリオごとの移行方法の前に、まずは Container Insights における認証方法として現在何が使用されているかを確認いただく必要がございます。  
Container Insights の認証方法としては、証明書ベースのローカル認証であるレガシ認証と、マネージド ID 認証の 2 種類がございます。  
レガシ認証を使用している場合、レガシ認証も ContainerLog テーブルと同じ 2026 年 9 月 30 日をもってサポートがされなくなります。  
- Azure の更新情報  
https://azure.microsoft.com/ja-jp/updates/?id=500853
![](./MigrationToContainerLogV2/announce02.png)

そのため、レガシ認証を使用している場合には、まずマネージド ID 認証に移行する必要があります。  
(後述いたしますマネージド ID 認証への移行をいただくだけで、ContainerLogV2 テーブルへの移行も完了します。)  

レガシ認証を使用しているクラスターは、Azure Resource Graph のクエリを用いて以下の手順で調べることができます。  

#### 公開情報
- コンテナーの分析情報のレガシ認証 > レガシ認証を使用してクラスターを検索する  
https://learn.microsoft.com/ja-jp/azure/azure-monitor/containers/container-insights-authentication?tabs=cli#find-clusters-using-legacy-authentication


#### レガシ認証を使用しているクラスターの検索手順

1. Azure ポータルの検索バーにて、"Resource Graph エクスプローラー" と検索し、サービス項目にある [Resource Graph エクスプローラー] をクリックします。<br>
![](./MigrationToContainerLogV2/resourcegraph01.png)

2. 以下のクエリを貼り付け、画面上部にあります [実行] をクリックします。
```
resources        
| where type =~ 'Microsoft.ContainerService/managedClusters'       
| project id, name, aksproperties = parse_json(tolower(properties)), location, identity        
| extend isEnabled = aksproperties.addonprofiles.omsagent.enabled         
| extend workspaceResourceId = iif(isEnabled == true, aksproperties.addonprofiles.omsagent.config.loganalyticsworkspaceresourceid, '')        
| extend useAADAuth  = aksproperties.addonprofiles.omsagent.config.useaadauth  
| where isEnabled =~ "true" and useAADAuth != true 
| extend parts = split(tostring(id), "/")
| extend subscriptionId = parts[2], AKSClusterName = parts[-1], resourceGroupName = parts[4] 
| project AKSClusterName, resourceGroupName, subscriptionId, location, AKSClusterId = tolower(id), workspaceResourceId
```
![](./MigrationToContainerLogV2/resourcegraph02.png)

3. 実行結果に表示されているクラスターでレガシ認証が使用されております。


### 2-2. 該当する移行シナリオはどれか
ここから、具体的な移行方法を説明します。  
どのシナリオが該当するかについては、以下のフロー チャートが参考になれば幸いです。
![](./MigrationToContainerLogV2/chart.png)

### 2-3. 移行に伴う注意点
ログの格納先が ContainerLog テーブルから ContainerLogV2 テーブルに変更されても、ログ アラート ルールやダッシュボードなど ContainerLog テーブルのデータを活用するサービスで用いるクエリは自動で変更されません。  
ContainerLog テーブルを検索しているクエリがある場合には、お客様ご自身で ContainerLogV2 テーブルをクエリするよう変更いただく必要がございます。  

例えば、ContainerLog テーブルに対してクエリを行っているアラート ルールが存在するかどうかは、
以下の Azure Resource Graph クエリでご確認いただけます。

Azure Resource Graph クエリの使用方法は、前述の ["2-1. レガシ認証の使用有無 " 内、"レガシ認証を使用しているクラスターの検索手順"](https://github.com/risaakita/blog-1/blob/main/articles/LogAnalytics/MigrationToContainerLogV2.md#2-1-%E3%83%AC%E3%82%AC%E3%82%B7%E8%AA%8D%E8%A8%BC%E3%81%AE%E4%BD%BF%E7%94%A8%E6%9C%89%E7%84%A1) に記載しております。  
```
resources
| where type in~ ('microsoft.insights/scheduledqueryrules') and ['kind'] !in~ ('LogToMetric')
| extend severity = strcat("Sev", properties["severity"])
| extend enabled = tobool(properties["enabled"])
| where enabled in~ ('true')
| where tolower(properties["targetResourceTypes"]) matches regex 'microsoft.operationalinsights/workspaces($|/.*)?' or tolower(properties["targetResourceType"]) matches regex 'microsoft.operationalinsights/workspaces($|/.*)?' or tolower(properties["scopes"]) matches regex 'providers/microsoft.operationalinsights/workspaces($|/.*)?'
| where properties contains "ContainerLog"
| project id,name,type,properties,enabled,severity,subscriptionId
| order by tolower(name) asc
```

- Container Insights のログ スキーマ > ContainerLogV2 スキーマを有効にする
https://learn.microsoft.com/ja-jp/azure/azure-monitor/containers/container-insights-logs-schema#enable-the-containerlogv2-schema

また、移行が完了してから ContainerLogV2 テーブルにログが格納されるようになるまで、10 分程度お時間がかかります。

## 3. レガシ認証を使用した Container Insights の場合の移行方法
レガシ認証からマネージド ID 認証に移行する過程で、ContainerLog テーブルから ContainerLogV2 テーブルへの移行も行われます。  
そのため、レガシ認証からマネージド ID 認証への移行のみを実施いただければ十分でございます。

### 移行手順
Azure CLI (バージョン 2.49.0 以上) で移行いただけます。  
具体的な移行方法について、以下の公開情報に記載しております。
- コンテナーの分析情報のレガシ認証 > マネージド ID 認証に移行する  
https://learn.microsoft.com/ja-jp/azure/azure-monitor/containers/container-insights-authentication?tabs=cli#migrate-to-managed-identity-authentication

マネージド ID 認証に移行した場合に、どのような変更が生じるか、どのような影響があるかなどについては、
以下の公開情報にまとまっております。  
よろしければ併せてご参照ください。  
- コンテナーの分析情報のレガシ認証 > マネージド ID 認証に移行するとどうなりますか?  
https://learn.microsoft.com/ja-jp/azure/azure-monitor/containers/container-insights-authentication?tabs=cli#what-happens-when-i-migrate-to-managed-identity-authentication

## 4. マネージド ID 認証を使用した Container Insights の場合の移行方法
すでに移行対象のクラスターでマネージド ID 認証を使用している場合には、ConfigMap で ContainerLogV2 テーブルにログを送るような設定を行う、
またはログ プロファイルの設定で ContainerLogV2 テーブルにログを送るような設定を行うことで移行がいただけます。(*)

(*)
ContainerLog テーブルにログが格納される条件は、以下 2 つを両方とも満たす場合でございます。
- ConfigMap において ContainerLogV2 へのログ送信が明示的にオフにされている
- ログ プロファイルで ContainerLogV2 へのログ送信が明示的にオフされている

### 4-1. ConfigMap の yaml を編集することにより移行する
ConfigMap では、パラメーター containerlog_schema_version を使用して、ContainerLog テーブルにログを送信する設定 (v1) になっているか、ContainerLogV2 テーブルにログを送る設定 (v2) になっているかを判断しています。  
そのため、使用している yaml 内でこのパラメーターの値を変更いただく必要があります。

(参考) パラメーターの一覧および各パラメーターの説明については、以下をご参照ください。  
- ConfigMap の設定  
https://learn.microsoft.com/ja-jp/azure/azure-monitor/containers/kubernetes-data-collection-configmap#configmap-settings

#### 移行手順
1. まず、ConfigMap が利用されていること、およびパラメーター container_schema_version の値を確認します。
```
kubectl get configmap container-azm-ms-agentconfig -n kube-system -o yaml
```

対象の環境で、ConfigMap の使用がない場合には、以下のような出力となります。
```
Error from server (NotFound): configmaps "container-azm-ms-agentconfig" not found
```
この場合、既存の ConfigMap はありませんので、[4-2. ConfigMap を有効化することにより移行する] または [4-3. ログ プロファイルを設定することにより移行する] のどちらかで移行いただく必要がございます。

2. ConfigMap が存在し、ConfigMap の設定が出力された場合には、パラメーター container_schema_version の値が "v1" になっていることを確認します。
この部分を "v2" に変更いただき、kubectl apply によりデプロイしてください。
```
kubectl apply -f container-azm-ms-agentconfig.yaml
```

### 4-2. ConfigMap を有効化することにより移行する

#### 移行手順
ConfigMap を有効化することで ContainerLogV2 テーブルに移行する場合の手順は、以下の公開情報に記載しております。
- ConfigMap を使用してコンテナー ログ収集を構成する  
https://learn.microsoft.com/ja-jp/azure/azure-monitor/containers/kubernetes-data-collection-configmap

ConfigMap を有効化いただくと、既定で ContainerLogV2 テーブルにログが送られるようになります。


### 4-3. ログ プロファイルを設定することにより移行する
#### 必要な Azure 組み込みロール
少なくともクラスターへの共同作成者権限が必要でございます。
- Azure Kubernetes Service (AKS) クラスターの監視を有効にする > 前提条件  
https://learn.microsoft.com/ja-jp/azure/azure-monitor/containers/kubernetes-monitoring-enable?tabs=azure-cli#prerequisites

#### 移行手順
1. Azure ポータルにて、ContainerLogV2 テーブルへの移行を行う AKS リソースを開きます。
2. 画面左側にある [モニター (分析情報)] をクリックします。
3. 画面上部にある [モニターの設定] をクリックします。<br>
![](./MigrationToContainerLogV2/logprofile01.png)
4. [機能] タブ内、"インフラストラクチャの監視" 項目にある [機能のカスタマイズ] をクリックします。<br>
![](./MigrationToContainerLogV2/logprofile02.png)
5. "構成のカスタマイズ" ウィンドウが開いたら、ログとイベント > ログ プリセット 項目に表示されている [コレクション設定の編集] をクリックします。<br>
![](./MigrationToContainerLogV2/logprofile03.png)
6. [ContainerLogV2 を有効にする] にチェックを入れ、下部にある [保存] をクリックします。  <br>
![](./MigrationToContainerLogV2/logprofile04.png)
<br>設定内容を確認し、もう一度 [保存] をクリックすると、"構成のカスタマイズ" ウィンドウが閉じます。<br>
![](./MigrationToContainerLogV2/logprofile05.png)

7. "モニターの設定" 画面下部にあります [レビューと有効化] をクリックし、設定内容が想定通りである場合には、[有効にする] をクリックします。

(参考)  
- AKS クラスターでコンテナーの分析情報とログ記録を有効にする  
https://learn.microsoft.com/ja-jp/azure/azure-monitor/containers/kubernetes-monitoring-enable?tabs=azure-portal#enable-container-insights-and-logging-on-an-aks-cluster

## 5. まとめ
2026 年 9 月 30 日に ContainerLog テーブルの使用のサポートが終了するに際し、AKS 環境にて後継の ContainerLogV2 テーブルへ移行する方法を紹介いたしました。  
本記事が、少しでも移行を検討される際にお役に立てれば幸いです。
