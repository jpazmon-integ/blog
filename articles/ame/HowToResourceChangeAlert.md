---
title: リソース変更を検知するアラートを作成する
date: 2023-11-09 00:00:00
tags:
  - How-To
  - Tips
  - Azure Monitor Essentials
---

こんにちは、Azure Monitoring チームの佐々木です。  
今回はリソース変更を検知するアラートが作成できる様になりましたので、その設定方法やサンプルクエリを紹介させていただきます。

<!-- more -->

## 目次
- Log Analytics から Azure Resource Graph へクエリ出来る様になりました
- Azure Resource Graph にはリソース変更を記録するテーブルがあります
- サンプルクエリ
- 設定手順
- より複雑なサンプルクエリ
  - 操作者を確認したい

## Log Analytics から Azure Resource Graph へクエリ出来る様になりました

Log Analytics より Azure Resource Graph (ARG) へクエリが出来る様になりました。

このアップデートにより、Log Analytics 内に記録されたログだけでなく、ARG へのクエリ結果を アラート ルールにて監視できる様になりました。

[Azure Resource Graph のデータに対してクエリを実行する (プレビュー)](https://learn.microsoft.com/ja-jp/azure/azure-monitor/logs/azure-monitor-data-explorer-proxy#query-data-in-azure-resource-graph-preview)

※ 2023/11/9 時点でプレビューでの公開となります。

## Azure Resource Graph にはリソース変更を記録するテーブルがあります

resourcechanges というテーブルに ARM のリソース変更履歴が記録されています。

例えば、properties 列を見る事で以下が確認可能です。

changeTypeがUpdate  -> 更新されたという事になります。

changes -> 変更があったプロパティと新しい値(newValue)、過去の値(previousValue)が入ります。

また、このテーブルの保持期間は14日で変更できないため、長期的な保管は行われません。

これを踏まえ、以下の properties 列のサンプルでは、microsoft.network/networkinterfaces リソースに MAC アドレスのプロパティが追加された事がわかります。

```json
{
    "targetResourceType": "microsoft.network/networkinterfaces",
    "targetResourceId": "/subscriptions/9835bbb5-ef97-4776-84b5-412b51f0ea8d/resourceGroups/1627/providers/Microsoft.Network/networkInterfaces/1627vm837",
    "changeAttributes": {
        "previousResourceSnapshotId": "08585022745207585807_a9667ff8-fb74-82a5-ffd1-419287f1063f_3620098351_1699332364",
        "newResourceSnapshotId": "08585022744595915807_7aa3a796-054b-ebe0-df8e-673414ebfde7_82482185_1699332425",
        "correlationId": "43c5d84a-c3bb-47c7-bdc5-7ecf69209fc2",
        "changesCount": 3,
        "timestamp": "2023-11-07T04:47:05.8860000Z",
        "changedByType": "User",
        "clientType": "Azure Portal",
        "operation": "Microsoft.Network/networkInterfaces/write",
        "changedBy": "skoyama@microsoft.com"
    },
    "changeType": "Update",
    "changes": {
        "properties.virtualMachine.id": {
            "propertyChangeType": "Insert",
            "changeCategory": "User",
            "previousValue": null,
            "newValue": "/subscriptions/9835bbb5-ef97-4776-84b5-412b51f0ea8d/resourceGroups/1627/providers/Microsoft.Compute/virtualMachines/1627vm"
        },
        "properties.primary": {
            "propertyChangeType": "Insert",
            "changeCategory": "User",
            "previousValue": null,
            "newValue": "True"
        },
        "properties.macAddress": {
            "propertyChangeType": "Insert",
            "changeCategory": "User",
            "previousValue": null,
            "newValue": "60-45-BD-66-0D-B8"
        }
    }
}
```

この様に changeType が Update のログが新しく記録された場合、何か設定値が変更されたという事がわかります。

詳しくは以下ドキュメントをご確認ください。

- https://learn.microsoft.com/ja-jp/azure/governance/resource-graph/how-to/get-resource-changes?tabs=azure-cli

## サンプルクエリ

```Kusto
arg("").resourcechanges
| extend timestamp = todatetime(properties["changeAttributes"]["timestamp"])
| extend changes = properties["changes"]
| extend ResourceId = tostring(properties["targetResourceId"])
| extend CorrelationId = tostring(properties["changeAttributes"]["correlationId"]) // AzureActivity (アクティビティログ) と結合可能
| where timestamp >= ago(5min) // arg("") で検索するテーブルにはアラートルールの粒度や期間指定が機能しないため、クエリで期間を指定します。
```

![image](https://github.com/sakkuntyo/cssblog/assets/20591351/01da721e-2011-4df9-ab3a-1feed3008c21)

## 設定手順

以下に、変更履歴を検知するアラートルールの作成方法を記載させていただきます。

この操作を行うアカウントでは、サブスクリプションスコープ の 閲覧者権限 と 閲覧者 ロールが必要になります。

[閲覧者](https://learn.microsoft.com/ja-jp/azure/role-based-access-control/built-in-roles#reader)
[ロールベースのアクセス制御管理者](https://learn.microsoft.com/ja-jp/azure/role-based-access-control/built-in-roles#role-based-access-control-administrator-preview)

1. Azureポータルにログインし、何れかの Log Analytics > ログ ブレードを開きます。
2. 例えばサンプルのクエリを指定し、一度検索します。

```
arg("").resourcechanges
| extend timestamp = todatetime(properties["changeAttributes"]["timestamp"])
| extend changes = properties["changes"]
| extend ResourceId = tostring(properties["targetResourceId"])
| extend CorrelationId = tostring(properties["changeAttributes"]["correlationId"]) // AzureActivity (アクティビティログ) と結合可能
| where timestamp >= ago(5min) // arg("") で検索するテーブルにはアラートルールの粒度や期間指定が機能しないため、クエリで期間を指定します。
```

![image](https://github.com/sakkuntyo/cssblog/assets/20591351/01da721e-2011-4df9-ab3a-1feed3008c21)

3. "新しいアラート ルール" より アラートルールの作成画面へ移動します。

![image](https://github.com/sakkuntyo/cssblog/assets/20591351/58c560b1-c619-414d-9403-3185e507ae5f)

4. 条件 セクションでは例えば以下の設定値とします。

![image](https://github.com/sakkuntyo/cssblog/assets/20591351/f90210e5-94b1-4ab2-931f-1671ca13b631)

- 測定
  - メジャー: テーブルの行
  - 集計の種類: カウント
  - 集計の粒度: 5分
- ディメンションで分割する
  - リソースID 列: ResourceId (これによりリソース毎にアラートが発生する様になります。)
  - ディメンション: 指定無し
- アラート ロジック
  - 演算子: 次の値より大きい
  - しきい値: 0
  - 評価の頻度: 5分

5. アクション セクションでは必要に応じてアクショングループを指定します。

6. 詳細 セクションにて、 System assigned managed Identity を指定し、作成します。

ARG へのクエリにはサブスクリプションスコープの閲覧者ロールが必要ですが、Default を選択した場合、アラートルールは ARG を閲覧できず検知されません。

![image](https://github.com/sakkuntyo/cssblog/assets/20591351/086a3aca-12de-417b-8095-0f9910dd2cea)

7. アラートルールの マネージドID の権限設定

モニター > アラート > アラート ルール > ID (プレビュー) を確認します。
この時、もしもシステム割り当て済 が オフ となっている場合にはオンにします。

![image](https://github.com/sakkuntyo/cssblog/assets/20591351/5989c4b1-1194-461b-806a-d2d9ff0db5af)

Azure ロールの割り当て へ進み、サブスクリプションスコープの閲覧者権限を付与します。

以上で設定は完了です。

### より複雑なサンプルクエリ

#### 操作者を確認したい

操作者はアクティビティログの Caller プロパティに記載されています。

診断設定で アクティビティログ を Log Analytics に送ると、AzureActivity テーブル上にアクティビティログが記録されますが、

AzureActivity テーブルは resourcechanges テーブルと CorrelationId が同じになるため、結合可能です。

そのため、以下のクエリで結合し、検索する事ができます。このクエリをアラートルールで指定し、検知する事も可能です。

```Kusto
arg("").resourcechanges
| extend timestamp = todatetime(properties["changeAttributes"]["timestamp"])
| extend changes = properties["changes"]
| extend ResourceId = tostring(properties["targetResourceId"])
| extend CorrelationId = tostring(properties["changeAttributes"]["correlationId"]) // AzureActivity (アクティビティログ) と結合可能
| where timestamp >= ago(14d) // arg("") で検索するテーブルにはアラートルールの粒度や期間指定が機能しないため、クエリで期間を指定します。
| join kind=leftouter (
  AzureActivity // Administrative を送る診断設定が必要です。
  | where CategoryValue =~ "Administrative"
  | where ActivityStatusValue =~ "Start" // 一度の操作 でStart,Success 等複数のログが出るため、Start のログのみ対象とします。
)
on CorrelationId 
| order by TimeGenerated
```



