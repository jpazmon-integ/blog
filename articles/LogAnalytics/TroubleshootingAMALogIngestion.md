---
title: Azure Monitor エージェントを使用したログ収集が行われない場合のトラブルシューティングについて
date: 2024-10-25 00:00:00
tags:
 - Tips
 - Log Analytics
---

こんにちは、Azure Monitoring チームの徳田です。

Azure Monitor エージェントを使用したログ収集を行っている場合、Log Analytics ワークスペースには、Heartbeat と呼ばれるログが 1 分毎に収集されます。
この Heartbeat はマシンの死活監視にも使用されることがありますが、何らかの理由でこの Heartbeat が収集されないことがあります。
Heartbeat が収集できていない場合は、ログ収集設定以前もところで問題が発生していることも多く、その場合は、Heartbeat 以外のログ収集も停止してしまいます。

今回は、Heartbeat をはじめとしたログが収集されないときの、はじめのトラブルシューティング方法をご紹介します。

## 目次
- [はじめに](#はじめに)
- [サポート対象 OS の確認](#サポート対象-os-の確認)
- [マシンの起動状態の確認](#マシンの起動状態の確認)
    - [確認方法](#確認方法)
    - [マシンが起動していないときの対処方法](#マシンが起動していないときの対処方法)
- [Azure Monitor エージェントの状態の確認](#azure-monitor-エージェントの状態の確認)
    - [確認方法](#確認方法-1)
    - [Azure Monitor エージェントが正常に稼働していない場合の対処方法](#azure-monitor-エージェントが正常に稼働していない場合の対処方法)
- [通信要件の確認](#通信要件の確認)
    - [確認方法](#確認方法-2)
    - [エンドポイントへ接続できていない場合の対処方法](#エンドポイントへ接続できていない場合の対処方法)
- [データ収集ルールの設定の確認](#データ収集ルールの設定の確認)
    - [確認方法](#確認方法-3)
    - [マシンとデータ収集ルールを関連付いていない場合の対処方法](#マシンとデータ収集ルールを関連付いていない場合の対処方法)
    - [ログ送信先となる Log Analytics ワークスペースを設定/変更する方法](#ログ送信先となる-log-analytics-ワークスペースを設定変更する方法)
- [マネージド ID の設定の確認](#マネージド-id-の設定の確認)
    - [確認方法](#確認方法-4)
    - [システム割り当てマネージド ID が割り当てられていないときの対処方法](#システム割り当てマネージド-id-が割り当てられていないときの対処方法)
- [おわりに](#おわりに)

## はじめに
Azure Monitor を使用したログ収集が行われない場合の要因は多岐にわたります。　
以下では、その中でもよくある原因の特定とその対処方法をご案内します。

## サポート対象 OS の確認
マシンの OS が、Azure Monitor エージェントにサポートされていない場合、当仮想マシン上でのエージェントの正常な動作は保証されておりません。  
そのため、まずはじめに、使用している OS が Azure Monitor エージェントのサポート対象であるかをご確認ください。  

Azure Monitor エージェントがサポートする OS は、以下公開情報にてご確認いただけます。  

[Azure Monitor エージェントでサポートされるオペレーティング システムと環境](https://learn.microsoft.com/ja-jp/azure/azure-monitor/agents/azure-monitor-agent-supported-operating-systems)

## マシンの起動状態の確認
サポート対象であることが分かったのち、次に確認すべきことは、マシンが正常に起動した状態であるかです。
マシンが起動していない場合は、ログ収集が行われません。  

### 確認方法
Azure portal では、対象のマシンの [概要] ページ内の "基本" > "状態" からご確認いただけます。

### マシンが起動していないときの対処方法
マシンが起動されていない場合 ("状態" が "停止済み" となっている場合) は、対象のマシンの [概要] ページ内の [開始] ボタンより起動させ、ログ収集が再開するかどうかをご確認ください。

## Azure Monitor エージェントの状態の確認
マシンが問題なく起動していても、Azure Monitor エージェントが正常に稼働しておらず、ログ収集が行われない場合があります。  

### 確認方法
Azure portal で対象のマシンを開き、左ペインの [設定] > [拡張機能とアプリケーション] を開きます。   
Linux の場合は "AzureMonitorLinuxAgent"、Windows の場合は "AzureMonitorWindowsAgent" の "状態" が "Provisioning Succeeded" となっている場合は、Azure Monitor エージェントが正常に稼働していることを意味します。

### Azure Monitor エージェントが正常に稼働していない場合の対処方法
- Azure Monitor エージェントの状態が "Provisioning Succeeded" になっていない場合は、一度マシン自体を再起動し、しばらく待って "Provisioning Succeeded" になるかをご確認ください。  
- それでも "Provisioning Succeeded" とならない場合は、下記の手順で、エージェントの再インストール をお試しください。これにより事象が解決する場合があります。

■ Azure Monitor エージェントの再インストール手順
1. Azure portal で対象のマシンを開き、左ペインの [拡張機能とアプリケーション] を開きます。
2. "種類" に "AzureMonitorLinuxAgent" (Linux の場合)、または "AzureMonitorWindowsAgent" (Windows の場合) が含まれるものを押下し、右ペインの [アンインストール] を押下します。
3. 2 のアンインストールの完了が確認出来たら、以下のコマンドを実行します。
    * <> の箇所はお客様の環境に合わせてご変更ください。実行時、<> は不要です
    * 自動アップグレード機能を有効化したくない場合は、"-EnableAutomaticUpgrade $true" を削除して実行してください。  

    (Linux の場合)
   ```
   Set-AzVMExtension -Name AzureMonitorLinuxAgent -ExtensionType AzureMonitorLinuxAgent -Publisher Microsoft.Azure.Monitor -ResourceGroupName <resourcegroup-name> -VMName <vm-name> -Location <region-name> -HandlerVersion <version> -DisableAutoUpgradeMinorVersion -EnableAutomaticUpgrade $true
   ```

   (Windows の場合)
   ```
   Set-AzVMExtension -Name AzureMonitorWindowsAgent -ExtensionType AzureMonitorWindowsAgent -Publisher Microsoft.Azure.Monitor -ResourceGroupName <resourcegroup-name> -VMName <vm-name> -Location <region-name> -HandlerVersion <version> -DisableAutoUpgradeMinorVersion -EnableAutomaticUpgrade $true
   ```
4. 1 の  [拡張機能とアプリケーション]  にて、Azure Monitor エージェントの "状態" が "Provisioning Succeeded" となれば、完了です。

## 通信要件の確認
マシンにインストールされた Azure Monitor エージェントがマシンのログを Log Analytics ワークスペースに送信したい場合は、マシン (Azure Monitor エージェント) が必要なエンドポイントへ接続できるよう設定する必要があります。  

接続できる必要のあるエンドポイントは以下の通りです。  

`global.handler.control.monitor.azure.com`  

`<virtual-machine-region-name>.handler.control.monitor.azure.com` 

`<log-analytics-workspace-id>.ods.opinsights.azure.com`  

Log Analytics ワークスペース内のカスタム ログ テーブルにデータを送信する場合は、以下のエンドポイントへの接続も必要です。  
`<data-collection-endpoint>.<virtual-machine-region-name>.ingest.monitor.azure.com`  
＊ 実際の値は、使用しているデータ収集ルールに設定しているデータ収集エンドポイントを Azure portal で開き、[概要] の "ログ インジェスト" を確認してください。  
   こちらに記載されている値から "https://" を除いたものがエンドポイントです。  
   ![](./TroubleshootingAMALogIngestion/dce-logingest.png)

### 確認方法
上記エンドポイントへ接続できているかどうかは、以下の方法で確認可能です。  

**■ Windows の場合**  
以下のコマンドを実行し、出力結果に `TcpTestSucceeded : True` が記載されていれば、エンドポイントに接続できています。  
( `<endpoint>` の箇所は、上記のエンドポイントに置き換えてください)
```
Test-NetConnection <endpoint> -port 443
```  
(結果例)  
![](./TroubleshootingAMALogIngestion/result-tnc.png)

**■ Linux の場合**  
以下のコマンドを実行し、出力結果に `CONNECTED` が記載されていれば、エンドポイントに接続できています。  

```
Test-NetConnection <endpoint> -port 443
```  
(結果例)  
![](./TroubleshootingAMALogIngestion/result-echo.png)

### エンドポイントへ接続できていない場合の対処方法
ネットワーク構成を確認し、エンドポイントへの接続を拒否するような設定が適用されていないかを確認してください。  
プロキシ サーバーやロード バランサーなどを介してログを送信するよう構成している場合は、プロキシ サーバーやロード バランサーのネットワーク設定も確認してください。  

上記内容を含む、Azure Monitor エージェントを使用する上でのネットワーク要件については、以下の弊社公開情報もご参照ください。  
[Azure Monitor エージェントのネットワーク構成](https://learn.microsoft.com/ja-jp/azure/azure-monitor/agents/azure-monitor-agent-network-configuration?tabs=PowerShellWindows)


## データ収集ルールの設定の確認
Azure Monitor エージェントを使用してログ収集を行う場合、データ収集ルールを使用して、対象のマシンと、ログ収集先の Log Analytics ワークスペースを指定します。  
確認対象のマシンが、データ収集ルールに関連付けられているか、ログ収集を確認している Log Analytics ワークスペースが合っているかを、確認してください。

### 確認方法
**■ マシンとデータ収集ルールの関連付け**  
1. Azure portal でデータ収集ルールのリソース ページを開きます。  
2. 左ペインの [構成] > [リソース] を開きます。  
   ここに、確認対象のマシンが表示されていることを確認してください。

**■ ログ収集先の Log Analytics ワークスペース**  
1. Azure portal でデータ収集ルールのリソース ページを開きます。  
2. 左ペインの [構成] > [データ ソース] を開きます。  
3. 表示されているデータ ソースを押下し、右ペインの [ターゲット] タブを開きます。  
   ここに表示される Log Analytics ワークスペースに、ログが収集されます。  
   ![](./TroubleshootingAMALogIngestion/check-dcr-target.png)

### マシンとデータ収集ルールを関連付いていない場合の対処方法
以下の手順でマシンとデータ収集ルールを関連付けます。
1. Azure portal でデータ収集ルールのリソース ページを開きます。  
2. 左ペインの [構成] > [リソース] を開きます。 
   [追加] を押下し、関連付けたいマシンを選択します。 
3. [適用] を押下し、完了です。

### ログ送信先となる Log Analytics ワークスペースを設定/変更する方法
ログ送信先となる Log Analytics ワークスペースを設定/変更したい場合は、以下の手順を実施してください。  
1. Azure portal でデータ収集ルールのリソース ページを開きます。  
2. 左ペインの [構成] > [データ ソース] を開きます。
3. 設定対象のデータ ソースを押下し、右ペイン内の [ターゲット] タブを開きます。
4. 現在設定されているターゲット (ワークスペース) を変更したい場合は、既存のターゲットのゴミ箱アイコンを押下し、削除します。
5. [ターゲットの追加] を押下し、設定したい Log Analytics ワークスペースを選択します。
6. [保存] を押下し、完了です。

## マネージド ID の設定の確認
Azure Monitor エージェントを使用する場合は、マシンにシステム割り当てマネージド ID または、ユーザー マネージド ID が割り当てられている必要があります。  

### 確認方法
1. Azure portal で確認対象のマシンを開きます。  
2. (システム割り当てマネージド ID の場合)   
   左ペインの [セキュリティ] > [ID] を押下し、[システム割り当て済み] タブを開きます。  
   "状態" が "オン" になっていれば、そのマシンにはシステム割り当てマネージド ID が割り当てられています。  
   ![](./TroubleshootingAMALogIngestion/check-systemassignedId.png)  
   (ユーザー割り当てマネージド ID の場合)  
   左ペインの [セキュリティ] > [ID] を押下し、[ユーザー割り当て済み] タブを開きます。  
   ユーザー割り当てマネージド ID が表示されていれば、そのマシンにはユーザー割り当てマネージド ID が割り当てられています。  

### マネージド ID が割り当てられていないときの対処方法  
以下の手順で、システム割り当てマネージド ID を割り当ててください。  

** ■ システム割り当てマネージド ID の割り当て手順 **  
1. Azure portal で確認対象のマシンを開きます。  
2. 左ペインの [セキュリティ] > [ID] を押下し、[システム割り当て済み] タブを開きます。  
3. "状態" の "オン" を押下し、しばらく待つと、システム割り当てマネージド ID が割り当てられます。  

** ■ ユーザー割り当てマネージド ID の割り当て手順 ** 
1. ユーザー割り当てマネージド ID を作成していない場合は、ユーザー割り当てマネージド ID を作成します。  
   [参考：ユーザー割り当てマネージド ID を作成する](https://learn.microsoft.com/ja-jp/entra/identity/managed-identities-azure-resources/how-manage-user-assigned-managed-identities?pivots=identity-mi-methods-azp#create-a-user-assigned-managed-identity)
2. Azure portal で確認対象のマシンを開き、左ペインの [セキュリティ] > [ID] を押下し、[ユーザー割り当て済み] タブを開きます。  
3. [追加] を押下し、右ペインで使用したいユーザー割り当てマネージド ID を選択し、[追加] を押下し、完了です。

なお、マネージド ID については、以下弊社サイトでもご案内しています。  
- [Azure Monitor エージェントの要件 | アクセス許可](https://learn.microsoft.com/ja-jp/azure/azure-monitor/agents/azure-monitor-agent-requirements#permissions)
- [Azure リソース用マネージド ID とは | マネージド ID の種類](https://learn.microsoft.com/ja-jp/entra/identity/managed-identities-azure-resources/overview#managed-identity-types)

## おわりに
今回は、Azure Monitor エージェントを使用したログ収集が行われない場合の、初期トラブルシューティング方法についてご案内しました。  
この情報が、事象解決の一助となれば幸いです。