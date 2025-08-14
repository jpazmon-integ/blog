---
title: Azure Update Manager に関するよくあるご質問
date: 2025-08-14 16:00:00
tags:
  - How-To
  - FAQ
  - Azure Update Manager
---

こんにちは、Azure Monitoring サポート チームの中条です。
今回は Azure Update Manager に関するよくあるご質問をご紹介します。
[Update Manager の基本概念、アーキテクチャ、機能、設定手順等](https://jpazmon-integ.github.io/blog/UpdateManager/AboutUpdateManager/) については、別途ブログを投稿しておりますので併せてご覧いただけますと幸いです！

<br>


<!-- more -->
## 目次

- [Q. パッチ オーケストレーションの変更方法を教えてください。](#Q.-パッチ-オーケストレーションの変更方法を教えてください。)
- [Q. Azure Update Manager でスケジュールで更新プログラムの適用を実施したいのに、Windows Update にて自動で更新プログラムが適用されました。](#Q.-Azure-Update-Manager-でスケジュールで更新プログラムの適用を実施したいのに、Windows-Update-にて自動で更新プログラムが適用されました。)
- [Q. パッチ オーケストレーション設定がグレー アウトしていて変更できません。](#Q.-パッチ-オーケストレーション設定がグレー-アウトしていて変更できません。)
- [Q. WSUS サポート終了に伴い、Azure Update Manager で更新プログラムを適用したいです。](#Q.-WSUS-サポート終了に伴い、Azure-Update-Manager-で更新プログラムを適用したいです。)
- [Q. SQL サーバーの更新プログラムが適用対象になりません。](#Q.-SQL-サーバーの更新プログラムが適用対象になりません。)
- [Q. Azure Update Manager で適用する更新プログラムを事前にダウンロードしておいて、後からインストールできますか。](#Q.-Azure-Update-Manager-で適用する更新プログラムを事前にダウンロードしておいて、後からインストールできますか。)
- [Q. Azure Update Manager のネットワーク要件を教えてください。](#Q.-Azure-Update-Manager-のネットワーク要件を教えてください。)

<br>
<br>
<br>



### Q. パッチ オーケストレーションの変更方法を教えてください。
[Azure Portal 上でパッチ オーケストレーションの設定を変更する方法](https://learn.microsoft.com/ja-jp/azure/update-manager/manage-update-settings?tabs=manage-single-overview%2Cmanage-scale-overview) は、下図のように Azure VM のパッチ オーケストレーションの設定を変更し、保存します。
<br>
![](Update_Manager_FAQ/Update_Manager_FAQ1.png)

<br>

[PowerShell 等のコマンドでパッチ オーケストレーションの設定を変更する方法](https://learn.microsoft.com/ja-jp/azure/update-manager/prerequsite-for-schedule-patching?tabs=new-prereq-powershell%2Cauto-portal#enable-scheduled-patching-on-azure-vms) もございます。
<br>
<br>
<br>

### Q. Azure Update Manager でスケジュールで更新プログラムの適用を実施したいのに、Windows Update にて自動で更新プログラムが適用されました。

以下の 2 点を確認します。
<details><summary><b>1. パッチ オーケストレーション設定が [Customer Managed Schedule] になっていること</b></summary>
パッチ オーケストレーション設定が [Windows 自動更新] が設定されている場合、OS 側で更新プログラムの適用が実施されます。

なお、パッチ オーケストレーションの設定を [Windows 自動更新] から [Customer Managed Schedule] へ変更いただいた後、更新プログラムの適用もしくは評価が実行されるまでは、後述の NoAutoUpdate = 1 が即時反映されません。

Azure Update Manger による更新プログラムの適用もしくは評価が実行されたタイミングで、対象マシンに Microsoft.CPlat.Core.WindowsPatchExtension 拡張機能がインストールされ、この拡張機能により、レジストリ キー NoAutoUpdate = 1 が変更されるという振る舞いです。


</details>

<br>

<details><summary><b>2. レジストリ キー NoAutoUpdate = 1 になっていること</b></summary>
以下のレジストリ キー内の NoAutoUpdate = 0 になっている場合、OS 側で更新プログラムの適用が実施されます。
HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU

レジストリ キーの詳細は [こちら](https://learn.microsoft.com/ja-jp/windows/deployment/update/waas-wu-settings#configuring-automatic-updates-by-editing-the-registry) をご参照ください。

レジストリ キー NoAutoUpdate を即時 0 から 1 に変更する方法としては、1 回限りの更新プログラム適用を実施します。

なお、Windows OS 側で設定いただく [自動更新のグループ ポリシー設定](https://learn.microsoft.com/ja-jp/windows-server/administration/windows-server-update-services/deploy/4-configure-group-policy-settings-for-automatic-updates##configure-automatic-updates) の有効化により NoAutoUpdate = 0 に書き換わる可能性があります。

</details>

<br>
<br>
<br>

 

### Q. パッチ オーケストレーション設定がグレー アウトしていて変更できません。
例えば、Windows OS の Azure VM のパッチ オーケストレーション設定は [Windowsの自動更新から手動更新に変更できません](https://learn.microsoft.com/ja-jp/azure/update-manager/troubleshoot?tabs=azure-machines#unable-to-change-the-patch-orchestration-option-to-manual-updates-from-automatic-updates)

Windows OS では VM 作成時に指定する[プロパティ osProfile.windowsConfiguration.enableAutomaticUpdates の値により、指定可能なパッチ オーケストレーション設定が決まります。](https://learn.microsoft.com/ja-jp/azure/virtual-machines/automatic-vm-guest-patching#patch-orchestration-modes)

 

|     osProfile.windowsConfiguration.enableAutomaticUpdates        |                            指定可能なパッチ オーケストレーション設定                                                         |
| ---------- | ----------------------------------------------------------------------------------- |
| true | カスタマー マネージド スケジュール (Customer Managed Schedule)、Azure マネージド - 安全なデプロイ (Azure Managed - Safe Deployment)、手動更新 (Manual)  |
| false | カスタマー マネージド スケジュール (Customer Managed Schedule)、Azure マネージド - 安全なデプロイ (Azure Managed - Safe Deployment)、Windows 自動更新 (AutomaticByOS) |


また、ホットパッチを有効化から無効化いただいた場合、選択可能なパッチ オーケストレーション設定は以下の 2 つのみです。

・カスタマー マネージド スケジュール (Customer Managed Schedule)

・Azure マネージド - 安全なデプロイ (Azure Managed - Safe Deployment)

<br>

![](Update_Manager_FAQ/Update_Manager_FAQ2.png)

<br>
<br>
<br>
<br>

### Q. WSUS サポート終了に伴い、Azure Update Manager で更新プログラムを適用したいです。
Azure Update Manager で更新プログラムの適用がサポートされているオペレーティング システムにつきましては[こちら](https://learn.microsoft.com/ja-jp/azure/update-manager/support-matrix-updates?tabs=ci-win&pivots=azure-vm) をご確認ください。

なお、Windows 10 や Windows 11 などのクライアント OS はサポートされていません。
Microsoft Intune での更新プログラムの管理をおすすめしております。

Azure Update Manager でサポートされていないワークロードにつきましては [こちら](https://learn.microsoft.com/ja-jp/azure/update-manager/unsupported-workloads) をご確認ください。

<br>
<br>
<br>

### Q. SQL サーバーの更新プログラムが適用対象になりません。
SQL サーバーの更新プログラムをインストールいただくためには、Windows OS の設定にて、[Windows の更新時に他の Microsoft 製品の更新プログラムも入手します] 設定を有効にする必要があります。詳細は[こちら](https://learn.microsoft.com/ja-jp/azure/update-manager/configure-wu-agent#enable-updates-for-other-microsoft-products)をご確認ください。
<br>
![](Update_Manager_FAQ/Update_Manager_FAQ3.png)
<br>

<br>
<br>
<br>

### Q. Azure Update Manager で適用する更新プログラムを事前にダウンロードしておいて、後からインストールできますか。
[Azure Update Manager では更新プログラムの事前ダウンロードはサポートされていません。](https://learn.microsoft.com/ja-jp/azure/update-manager/configure-wu-agent#pre-download-updates)


<br>
<br>
<br>

### Q. Azure Update Manager のネットワーク要件を教えてください。

Azure Update Manager をご利用いただく際の[ネットワーク要件](https://learn.microsoft.com/ja-jp/azure/update-manager/prerequisites#network-planning)について、OS の種類ごとに説明します。


<details><summary><b>1. Windows OS の場合</b></summary>
Windows OS のマシンの場合は、Windows Update エージェントで必要とされるすべてのエンドポイントへのトラフィックを許可する必要があります。

Windows Update エージェントで必要なエンドポイントの一覧は[こちら](
https://learn.microsoft.com/ja-jp/troubleshoot/windows-client/installing-updates-features-roles/windows-update-issues-troubleshooting?toc=%2Fwindows%2Fdeployment%2Ftoc.json&bc=%2Fwindows%2Fdeployment%2Fbreadcrumb%2Ftoc.json#issues-related-to-httpproxy) で確認できます。 


</details>

<br>

<details><summary><b>2. Linux OS の場合</b></summary>
Linux OS のマシンの場合は、Linux パッケージの配布先のエンドポイントへの通信を許可する必要があります。

例えば、Red Hat Linux マシンの場合の Linux パッケージの配布先のエンドポイントは[こちら](https://learn.microsoft.com/ja-jp/azure/virtual-machines/workloads/redhat/redhat-rhui#the-ips-for-the-rhui-content-delivery-servers)で確認できます。

他の Linux ディストリビューションについては、各プロバイダーのドキュメントをご覧ください。

</details>

<br>
<br>
<br>


上記の内容以外でご不明な点や疑問点などございましたら、弊社サポート サービスまでお問い合わせください。
最後までお読みいただきありがとうございました！

※本情報の内容（添付文書、リンク先などを含む）は、作成日時点でのものであり、予告なく変更される場合があります。


