---
title: 【廃止】 実行アカウントからマネージド ID へ移行する方法及び FAQ
date: 2022-10-14 00:00:00
tags:
  - Automation
  - Runbook
  - Run As Account
  - Managed Identity
  - Managed ID
  - Migration
  - 移行
  - 実行アカウント
  - マネージド ID
---

[更新履歴]  
- 2022/10/14 : ブログ公開
- 2025/12/18 : 廃止について追記済み

こんにちは！Azure Monitoring サポート チームの趙です。 
今回は、実行アカウントからマネージド ID へ移行する方法及び FAQ をご案内させていただきます。

<!-- more -->

## 目次
- [目次](#目次)
- [実行アカウントからマネージド ID への移行](#実行アカウントからマネージド-id-への移行)
- [公開情報](#公開情報)
- [FAQ](#faq)
  - [Q1.Automation アカウントと実行アカウントは同じものですか？](#q1automation-アカウントと実行アカウントは同じものですか)
  - [Q2.移行作業を実施すことによる影響はありますか？](#q2移行作業を実施すことによる影響はありますか)
  - [Q3.実行アカウントからマネージド ID への移行とは具体的に何を意味しますか？](#q3実行アカウントからマネージド-id-への移行とは具体的に何を意味しますか)
  - [Q4.実行アカウントが存在するかどうかはどこから確認できますか？](#q4実行アカウントが存在するかどうかはどこから確認できますか)
  - [Q5.実行アカウントの参照、削除ができません。どうすればいいですか？](#q5実行アカウントの参照削除ができませんどうすればいいですか)
  - [Q6.システム割り当てマネージド ID とユーザー割り当てマネージド ID、どちらを使えばいいですか？](#q6システム割り当てマネージド-id-とユーザー割り当てマネージド-idどちらを使えばいいですか)
- [移行作業の順番及び対応例](#移行作業の順番及び対応例)
  - [免責事項](#免責事項)
  - [1.Runbook に実行アカウントを利用しているコマンドが存在するかを確認](#1runbook-に実行アカウントを利用しているコマンドが存在するかを確認)
    - [AzureRM モジュールのコマンドを利用している場合の例](#azurerm-モジュールのコマンドを利用している場合の例)
    - [Az モジュールのコマンドを利用している場合の例](#az-モジュールのコマンドを利用している場合の例)
  - [2.マネージド ID の作成](#2マネージド-id-の作成)
  - [3.マネージド ID へのロールの割り当て](#3マネージド-id-へのロールの割り当て)
  - [4.マネージド ID 用の新規 Runbook を作成、あるいは、既存の Runbook を修正](#4マネージド-id-用の新規-runbook-を作成あるいは既存の-runbook-を修正)
    - [マネージド ID 用の新規 Runbook を作成する場合](#マネージド-id-用の新規-runbook-を作成する場合)
    - [既存の Runbook を修正 (既存の Runbook スクリプトの実行アカウント用の認証コマンドを、マネージド ID 用の認証コマンドに修正) する場合](#既存の-runbook-を修正-既存の-runbook-スクリプトの実行アカウント用の認証コマンドをマネージド-id-用の認証コマンドに修正-する場合)
  - [5.マネージド ID の動作テスト](#5マネージド-id-の動作テスト)
    - [サンプル](#サンプル)
  - [6.マネージド ID 用 Runbook の動作テスト](#6マネージド-id-用-runbook-の動作テスト)
    - [正常動作する場合](#正常動作する場合)
    - [新規作成あるいは修正したマネージド ID 用 Runbook が正常動作しない場合](#新規作成あるいは修正したマネージド-id-用-runbook-が正常動作しない場合)
    - [サンプル Runbook が動作しない場合](#サンプル-runbook-が動作しない場合)
  - [7.実行アカウントの削除](#7実行アカウントの削除)

## 実行アカウントからマネージド ID への移行

- 現在、Runbook の認証手段として実行アカウントをご利用されているお客様は、実行アカウントからマネージド ID への移行をご検討いただければ幸いです。
- 既に Runbook の認証手段としてマネージド ID をご利用されているお客様は、対応不要です。
- 2023 年 9 月 30 日に実行アカウントは、サポート終了 (retire) となります。
- そのため、2023 年 9 月 30 日までには、マネージド ID へ移行する必要があります。
- マネージド ID は、実行アカウントと同じ機能を提供しており、弊社が推奨する認証手段です。

>[!WARNING]
>公開情報には、"Azure Automation実行アカウントは 2023 年 9 月 30 日に廃止され、マネージド ID に置き換えられます。" と記載されておりますが、補足させていただきます。
>弊社側から 2023 年 9 月 30 日までにご利用されている実行アカウントをマネージド ID へ移行する作業・デプロイを行うわけではございません。
>2023 年 9 月 30 日に実行アカウントは、サポート終了 (retire) する意味とご理解いただければ幸いです。
>お客様にて、直接、実行アカウントをマネージド ID へ移行するようにご対応いただく必要がございます。

## 公開情報

弊社にて推奨する移行ガイドラインは、[既存の実行アカウントからマネージド ID に移行する](https://learn.microsoft.com/ja-jp/azure/automation/migrate-run-as-accounts-managed-identity?tabs=run-as-account) に詳細が記載されております。必要なアクション、移行時参考となるサンプル コマンド等は、基本的には、上記公開情報をご参考いただき、ご対応いただければ幸いです。

実行アカウントとマネージド ID そのものの概念及び詳細は、以下公開情報及びブログもご参考いただければ幸いです。

- [Azure Automation アカウントの認証の概要](https://learn.microsoft.com/ja-JP/azure/automation/automation-security-overview) (公開情報)
- [実行アカウントとマネージド ID の違いについて](https://jpazmon-integ.github.io/blog/automation/DifferenceRunAsAccountAndManagedID/) (ブログ)
- [Azure Automation の実行アカウントの管理方法の tips と FAQ](https://jpazmon-integ.github.io/blog/automation/HowToManageRunAsAccount/) (ブログ)

## FAQ

### Q1.Automation アカウントと実行アカウントは同じものですか？

Automation アカウントと実行アカウントは別のものです。

Automation アカウントは、Azure Automation サービスを利用するために必要なリソースを保持するものとご理解いただければ幸いです。Azure Automation サービスを利用するために必要なリソースには、Runbook、スケジュール、モジュール、変数、資格情報、証明書等があります。

実行アカウントは、Azure への認証に利用されるサービス プリンシパルです。
Automation アカウントにある Runbook スクリプトに実行アカウントを利用する認証コマンドを実装いただくことで、Runbook 実行時、Azure への認証が可能です。
実行アカウントの詳細は、[Azure Automation の実行アカウントの管理方法の tips と FAQ](https://jpazmon-integ.github.io/blog/automation/HowToManageRunAsAccount/) をご参考いただければ幸いです。

### Q2.移行作業を実施すことによる影響はありますか？

移行作業時、実行アカウントを先に削除しない限り、
実行アカウントからマネージド ID への移行作業による影響は特にございません。

実行アカウントとマネージド ID は、お互い影響しない別のものであるため、共存 (Automation アカウントに実行アカウントとマネージド IDをどちらも保持すること) ができます。
移行作業時に、既存 Runbook にてご利用されている実行アカウントを削除せず、マネージド ID を新規作成し、
移行作業後、マネージド ID のみ残し、実行アカウントを削除いただければ、
実行アカウントからマネージド ID への移行作業による影響は特にございません。

一方で、現在、Runbook に実行アカウントによる認証コマンドを記載し、その Runbook を実行されており、
マネージド ID を用意せず、実行アカウントを削除してしまった場合は、
対象 Runbook にて、Azure への認証が行われず、対象 Runbook 内後続の処理が失敗する可能性がございます。

移行作業時の順番及び対応例は、後述させていただきます。

### Q3.実行アカウントからマネージド ID への移行とは具体的に何を意味しますか？

以下を意味します。

- Automation アカウントにマネージド ID を作成する
- Runbook スクリプトに記載されている実行アカウントによる認証コマンドをマネージド ID による認証コマンドに書き換える
- Automation アカウントから実行アカウントを削除する


### Q4.実行アカウントが存在するかどうかはどこから確認できますか？

以下操作で確認できます。

```
[Azure ポータル] > [Automation アカウント] > [実行アカウント] > Azure 実行アカウントが存在していれば、実行アカウントが存在
```

### Q5.実行アカウントの参照、削除ができません。どうすればいいですか？

[Azure Automation の実行アカウントの管理方法の tips と FAQ](https://jpazmon-integ.github.io/blog/automation/HowToManageRunAsAccount/) をご参考いただき、操作を行っているユーザー アカウントに必要な権限を付与いただくか、権限があるユーザー アカウントにて操作いただければ幸いです。

### Q6.システム割り当てマネージド ID とユーザー割り当てマネージド ID、どちらを使えばいいですか？

弊社過去事例として、システム割り当てマネージド ID や、ユーザー割り当てマネージド ID の中で、
どちらを利用すればいいかとのご質問をよくいただいておりますが、
弊社から推奨する認証手段は特になく、お客様がご判断いただければ幸いです。

[マネージド ID のベスト プラクティスに関する推奨事項](https://learn.microsoft.com/ja-jp/azure/active-directory/managed-identities-azure-resources/managed-identity-best-practice-recommendations) をご参考いただき、お客様のご要件に応じて、システム割り当てマネージド ID、あるいは、ユーザー割り当てマネージド ID どちらかをご利用いただければ幸いです。

Automation 観点では、システム割り当てマネージド ID が、ユーザー割り当てマネージド ID より、マネージド ID を構成する手間 (操作回数) が少なく、認証に利用するコマンドが短いです。
上記以外の相違点 (メリット、デメリット等)は特にございません。

## 移行作業の順番及び対応例

以下免責事項をご確認いただいたうえで、以下対応例をご参考いただければ幸いです。

### 免責事項

- 以下にご案内する内容は、あくまでも対応例 (ガイドライン) です。お客様の Runbook とスクリプトの記載内容が異なる場合もある点、予めご理解いただければ幸いです。
- 弊社サポートにて、お客様のご要件の Runbook を作成・編集・提供することは、サポートしておりません。
- スクリプトの修正、スクリプトの追記等が必要な際には、お客様のご要件に応じて、お客様にて直接修正・動作確認いただく必要がございます。そのため、大変恐れ入りますが、スクリプトの修正、スクリプトの追記等をお問い合わせいただいても、ご要望を承ることができない点、予めご理解いただければ幸いです。
- 例えば、現在お客様がご利用されている Runbook をご提示いただきながら、サポートにて修正・アップデート・カスタマイズしてくださいとのご要望は、弊社サポートにて承ることができない点、予めご理解いただければ幸いです。

### 1.Runbook に実行アカウントを利用しているコマンドが存在するかを確認

```
[Azure ポータル] > [Automation アカウント] > [Runbook] > 対象 Runbook クリック > [表示] > Runbook スクリプト確認
```

Runbook に実行アカウントを利用しているコマンドがあるかどうかは、
以下のようなサンプルコマンドが Runbook に記載されているかどうかをご確認いただけますと幸いです。
以下例の場合、Azure への認証に実行アカウントを利用しています。

#### AzureRM モジュールのコマンドを利用している場合の例

```PowerShell
$connectionName = "AzureRunAsConnection"
# Get the connection "AzureRunAsConnection"

$servicePrincipalConnection=Get-AutomationConnection -Name $connectionName

"Logging in to Azure..."
Add-AzureRmAccount `
    -ServicePrincipal `
    -TenantId $servicePrincipalConnection.TenantId `
    -ApplicationId $servicePrincipalConnection.ApplicationId `
    -CertificateThumbprint $servicePrincipalConnection.CertificateThumbprint
```

#### Az モジュールのコマンドを利用している場合の例

```PowerShell
# Connect Automation
$connectionName = "AzureRunAsConnection"
try
{
    # Get the connection "AzureRunAsConnection "
    $Conn=Get-AutomationConnection -Name $connectionName         
    Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint　$Conn.CertificateThumbprint

}
catch {
    if (!$Conn)
    {
        $ErrorMessage = "Connection $connectionName not found."
        throw $ErrorMessage
    } else{
        Write-Error -Message $_.Exception
        throw $_.Exception
    }
}
```

上記以外の例は、[既存の実行アカウントからマネージド ID に移行する](https://learn.microsoft.com/ja-jp/azure/automation/migrate-run-as-accounts-managed-identity?tabs=run-as-account) の例もご参考いただければ幸いです。

お客様の Runbook の実行アカウントによる認証コマンドが、上記例とは完全一致しない場合もあると思います。
その際には、スクリプトの Get-AutomationConnection で指定している接続が、"AzureRunAsConnection" と記載されていれば、実行アカウントを認証手段として利用しているとご判断いただければ幸いです。

### 2.マネージド ID の作成

以下公開情報をご参考いただき、システム割り当てマネージド ID 、あるいは、ユーザー割り当てマネージド ID、どちらかを作成いただければ幸いです。

- [Azure Automation アカウントのシステム割り当てマネージド ID を使用する](https://learn.microsoft.com/ja-jp/azure/automation/enable-managed-identity-for-automation)
- [Azure Automation アカウントのユーザー割り当てマネージド ID を使用する](https://learn.microsoft.com/ja-jp/azure/automation/add-user-assigned-identity)

### 3.マネージド ID へのロールの割り当て

以下公開情報をご参考いただき、マネージド ID に必要なロールを割り当てていただければ幸いです。

- [システム マネージド IDへのロールの割り当てを確認する](https://learn.microsoft.com/ja-jp/azure/automation/enable-managed-identity-for-automation#verify-role-assignment-to-a-system-managed-identity)
- [ユーザー マネージド ID へのロールの割り当てを確認する](https://learn.microsoft.com/ja-jp/azure/automation/add-user-assigned-identity#verify-role-assignment-to-a-user-managed-identity)

既存の実行アカウントと同じロールをマネージド ID に割り当てたい場合は、以下をご参考いただければ幸いです。

お客様にて、ポータルで実行アカウントを作成していた場合は、既定では、Azure サブスクリプションの共同作成者ロールが実行アカウントに割り当てられています。この場合は、マネージド ID へサブスクリプションの共同作成者ロールを割り当てていただければ幸いです。

お客様にて、ポータルで実行アカウントを作成していなかた場合、あるいは、ポータルで作成したかどうか忘れた場合は、実行アカウントに割り当てていたロールを確認し、割り当てます。
以下操作で実行アカウントに割り当てられていたロールが確認できます。

```
[Azure ポータル] > [Automation アカウント] > [実行アカウント] > [Azure 実行アカウント] > ロールの配下に記載されている "役割" (ロール) と "割り当て対象" (スコープ) を確認
```

### 4.マネージド ID 用の新規 Runbook を作成、あるいは、既存の Runbook を修正

Runbook にて、認証コマンドで実行アカウントを利用されている場合は、
以下**いずれか** (AND 条件ではなく、OR 条件です) をご対応いただければ幸いです。

- マネージド ID 用の新規 Runbook を作成する
- 既存の Runbook スクリプトの実行アカウント用の認証コマンドを、マネージド ID 用の認証コマンドに修正

お客様のご要件に応じて、Runbook を新規作成するか、既存の Runbook を修正するかをお客様にてご判断いただけますと幸いです。
既存の Runbook をしばらくの間、そのまま使いたいとのご要望の場合は、マネージド ID 用の新規 Runbook を作成する案をご検討いただけますと幸いです。

#### マネージド ID 用の新規 Runbook を作成する場合

以下操作で、Runbook を新規作成できます。

```
[Azure ポータル] > [Automation アカウント] > [Runbook] > [Runbook の作成]
```

以下公開情報をご参考いただき、システム割り当てマネージド ID 用の認証コマンド、あるいは、ユーザー割り当てマネージド ID 用の認証コマンドを Runbook に記載します。

- [サンプルのスクリプト](https://learn.microsoft.com/ja-JP/azure/automation/migrate-run-as-accounts-managed-identity?tabs=sa-managed-identity#sample-scripts)
- [システム割り当てマネージド ID を使用してアクセスを認証する](https://learn.microsoft.com/ja-JP/azure/automation/enable-managed-identity-for-automation#authenticate-access-with-system-assigned-managed-identity)
- [ユーザー割り当てマネージド ID を使用してアクセスを認証する](https://learn.microsoft.com/ja-jp/azure/automation/add-user-assigned-identity#authenticate-access-with-user-assigned-managed-identity)

Runbook スクリプトの最初の処理として、マネージド ID 用の認証コマンドを記載します。
以下のサンプル コマンドのように記載することが可能です。

以下例の場合、Azure への認証にシステム割り当てマネージド ID を利用しています。

```PowerShell
{
    "Logging in to Azure..."
    Connect-AzAccount -Identity
}
catch {
    Write-Error -Message $_.Exception
    throw $_.Exception
}
```

上記のようにマネージド ID の認証コマンドを Runbook の最初に記載した後は、
お客様にて実行したい処理を Runbook に記載いただけますと幸いです。
既存の Runbook に記載されている実行アカウント用の Azure への認証コマンド以外の処理 (例えば、VM を起動・停止するメインの処理等) を
そのまま新規 Runbook に記載いただけますと幸いです。

#### 既存の Runbook を修正 (既存の Runbook スクリプトの実行アカウント用の認証コマンドを、マネージド ID 用の認証コマンドに修正) する場合

以下操作で、Runbook を編集できます。

```
[Azure ポータル] > [Automation アカウント] > [Runbook] > 対象 Runbook クリック > 編集
```

上記 [1.Runbook に実行アカウントを利用しているコマンドが存在するかを確認](#1runbook-に実行アカウントを利用しているコマンドが存在するかを確認) にて、確認した実行アカウント用の Azure への認証コマンドをマネージド ID 用の認証コマンドに置き換えます。

最終的に、Runbook スクリプト内認証コマンドの箇所には、マネージド ID 用の認証コマンドだけ残すようにご対応いただければ幸いです。
以下例の場合、Azure への認証にシステム割り当てマネージド ID を利用しています。

```PowerShell
{
    "Logging in to Azure..."
    Connect-AzAccount -Identity
}
catch {
    Write-Error -Message $_.Exception
    throw $_.Exception
}
```

以下公開情報もご参考いただき、システム割り当てマネージド ID 用の認証コマンド、あるいは、ユーザー割り当てマネージド ID 用の認証コマンドを Runbook に記載します。

- [サンプルのスクリプト](https://learn.microsoft.com/ja-JP/azure/automation/migrate-run-as-accounts-managed-identity?tabs=sa-managed-identity#sample-scripts)
- [システム割り当てマネージド ID を使用してアクセスを認証する](https://learn.microsoft.com/ja-JP/azure/automation/enable-managed-identity-for-automation#authenticate-access-with-system-assigned-managed-identity)
- [ユーザー割り当てマネージド ID を使用してアクセスを認証する](https://learn.microsoft.com/ja-jp/azure/automation/add-user-assigned-identity#authenticate-access-with-user-assigned-managed-identity)

既存の Runbook に記載されている実行アカウント用の Azure への認証コマンド以外の処理 (例えば、VM を起動・停止するメインの処理等) は、そのまま残していただければ幸いです。

### 5.マネージド ID の動作テスト

お客様の Automation アカウントに、チュートリアル用 Runbook "AzureAutomationTutorialWithIdentity" が存在する場合は、以下のように操作することで、マネージド ID が動作するかどうかを確認できます。

```
[Azure ポータル] > [Automation アカウント] > [Runbook] > [AzureAutomationTutorialWithIdentity] クリック > [開始] > ジョブが完了し、エラーが特になければマネージド ID は正常動作
```

お客様の Automation アカウントに、チュートリアル用 Runbook "AzureAutomationTutorialWithIdentity" が存在しない場合は、以下サンプル Runbook を実行することで、マネージド ID が動作するかどうかを確認できます。
以下サンプルが、チュートリアル用 Runbook "AzureAutomationTutorialWithIdentity" のスクリプトです。

#### サンプル 

```PowerShell
<#
    .DESCRIPTION
        An example runbook which gets all the ARM resources using the Managed Identity
    .NOTES
        AUTHOR: Azure Automation Team
        LASTEDIT: Oct 26, 2021
#>
"Please enable appropriate RBAC permissions to the system identity of this automation account. Otherwise, the runbook may fail..."
try
{
    "Logging in to Azure..."
    Connect-AzAccount -Identity
}
catch {
    Write-Error -Message $_.Exception
    throw $_.Exception
}
#Get all ARM resources from all resource groups
$ResourceGroups = Get-AzResourceGroup
foreach ($ResourceGroup in $ResourceGroups)
{    
    Write-Output ("Showing resources in resource group " + $ResourceGroup.ResourceGroupName)
    $Resources = Get-AzResource -ResourceGroupName $ResourceGroup.ResourceGroupName
    foreach ($Resource in $Resources)
    {
        Write-Output ($Resource.Name + " of type " +  $Resource.ResourceType)
    }
    Write-Output ("")
}
```

>[!NOTE]
上記サンプル Runbook は、マネージド ID で Azure へ認証し、接続したサブスクリプションに存在する Azure リソースの一覧を取得 (Get-AzResourceGroup) する処理を実行するだけの Runbook です。
そのため、上記サンプル Runbook を実行しても、お客様のリソースへの影響等はございません。
>

### 6.マネージド ID 用 Runbook の動作テスト


#### 正常動作する場合
上記 [4.マネージド ID 用の新規 Runbook を作成、あるいは、既存の Runbook を修正](#4マネージド-id-用の新規-runbook-を作成あるいは既存の-runbook-を修正) にて作成、あるいは、修正したマネージド ID 用 Runbook を開始し、ジョブが完了し、エラーがなければ正常動作とご理解いただければ幸いです。

#### 新規作成あるいは修正したマネージド ID 用 Runbook が正常動作しない場合
上記 [5.マネージド ID の動作テスト](#5マネージド-id-の動作テスト) のサンプル Runbook は動作するが、[4.マネージド ID 用の新規 Runbook を作成、あるいは、既存の Runbook を修正](#4マネージド-id-用の新規-runbook-を作成あるいは既存の-runbook-を修正) にて作成、あるいは、修正したお客様のマネージド ID 用 Runbook は動作しない場合は、マネージド ID の認証は問題ないが、お客様にて作成あるいは修正した Runbook スクリプト自体に問題があるとご理解いただければ幸いです。
この場合は、お客様のスクリプトを見直していただければ幸いです。

#### サンプル Runbook が動作しない場合
上記 [5.マネージド ID の動作テスト](#5マネージド-id-の動作テスト) のサンプル Runbook が動作しない場合は、マネージド ID が正常に構成されていないとご理解いただければ幸いです。
この場合は、マネージド ID が作成できているか、ロールが割り当てられているかを再度ご確認いただければ幸いです。

### 7.実行アカウントの削除

上記にてマネージド ID 用 Runbook が正常動作することが確認できた場合は、
実行アカウントを削除いただければ幸いです。
今削除したくない場合は、お客様のご要件に応じて、お客様のタイミングにて削除いただければ幸いです。
実行アカウントの削除方法は、以下公開情報をご参考いただければ幸いです。

- [Azure Automation の実行アカウントを削除する](https://learn.microsoft.com/ja-jp/azure/automation/delete-run-as-account)

以上です。