---
title: Azure Automation にて特定期間のジョブの実行時間を取得する
date: 2019-4-3 02:15:00
tags:
  - Automation
  - Howto
  - Runbook
  - Job
---

こんにちは Azure サポート チームの村田です。  
Azure Automation においては昨年価格体系が変更されました

具体的には以下のように以前の Free、Basic という概念がなくなり、以下のように統合された価格体系となっております。

- 以前の価格料金
  - Free - 月あたり 500 分の無料時間が付与され、超過した場合はジョブが実行されなくなります。
  - Basic - 従量課金として、0.23円 / 分 で課金が発生します。
- 変更後の価格体系
  - 無償の500分のジョブ実行時間が付与され、超過した分に関しましては、0.304円 / 分 (2025-11-25時点価格) で課金が発生します。

## 参照情報

- Automation の価格
  - https://azure.microsoft.com/ja-jp/pricing/details/automation/

以前は Basic プランの場合、最初の1分から課金対象となっていましたが、新しいプランでは、必ず 月あたり 500 分の無料ジョブ実行時間がつくのでプラン的にはお得になってますね。

しかし、この価格体系の変更に伴い、Automation アカウントの画面内から、月のジョブの実行時間の累計を表示する項目が消えてしまいました。

したがいまして、今どの程度ジョブの実行時間を消費しているのかわからないとご質問をいただく場合がございます。

そんな場合に備え、今回はローカルから PowerShell を実行する事で、特定の期間のジョブの実行時間を取得するスクリプトをご案内します。

日別にジョブ実行時間が取得できるため、特定の日時のみジョブの実行時間がとても長い際などには、トラブルシューティングにも便利ですので是非ご利用ください。

### スクリプトの実行環境要件
Windows 10またはWindows Server 2016でコンピュータを実行している場合は、すでにPowerShell 5以降がインストールされているため、この手順は必要ありません。

今回の場合、PowerShell のバージョンは 3.0 とのことですので、PowerShellを含むWindows Management Framework（WMF）の最新バージョンに更新する必要がございます。

- WMFの最新バージョンを入手するには、次のURLにアクセスしてください。
  - https://www.microsoft.com/en-us/download/details.aspx?id=54616
- WMFとパッケージ管理へのリンクを含むPowerShell Galleryの使用に関する詳細は、下記を参照してください。
  - https://www.powershellgallery.com/


## Azure RM モジュールのインストール
AzureからPowerShell経由で情報を照会できるようにするには、Azure Resource Management（AzureRM）モジュールをインストールする必要があります。

先に進む前に、お使いのコンピュータがパッケージ管理の要件を満たしていることを確認してください。

AzureRMモジュールをインストールするには、以下の手順に従います。

1. [管理者として実行]オプションを使用してPowerShellセッションを開きます。
1. Install-Module -Name AzureRM -Force -Verboseと入力し、Enterキーを押します。
1. インストールが完了するまで待ち、PowerShellのプロンプトに戻ります。
1. モジュールがインストールされたことを確認するために、Get-Module -Name AzureRM -ListAvailableと入力し、Enterキーを押します。

### 実行スクリプト
PowerShell ISE を起動し、以下のスクリプトを貼り付け、期間を指定して実行します。  
※ブロックが複数にわかれておりますが、すべて一つのスクリプトに記載してください。

```ps1
Add-AzureRMAccount | Write-Verbose
#ジョブの実行時間を保存するディレクトリを記載します。
$CSVFile = "c:\temp\automation.csv"
#ジョブの実行時間を取得する期間の開始日時を指定します
$StartTime = "12/01/2017"
#ジョブの実行時間を取得する期間の終了日時を指定します
$EndTime = "12/08/2017"

#ジョブの実行時間を取得するサブスクリプションID を記載します
$SubscriptionID = 'Subscription ID'
Set-AzureRmContext -SubscriptionId $SubscriptionID | Write-Verbose

$UsageInfo = Get-UsageAggregates -ReportedStartTime $StartTime -ReportedEndTime $EndTime -AggregationGranularity Daily -ShowDetails $true

 Do { 
    $UsageInfo.UsageAggregations.Properties | Where-Object {($_.MeterCategory -eq 'Automation') -and ($_.MeterName -eq 'Basic Runtime')} | `
            Select-Object `
            UsageStartTime, `
            UsageEndTime, `
            @{n='SubscriptionId';e={$SubscriptionId}}, `
            MeterCategory, `
            MeterId, `
            MeterName, `
            MeterSubCategory, `
            MeterRegion, `
            Unit, `
            Quantity, `
            @{n='Project';e={$_.InfoFields.Project}}, `
            InstanceData `
            | Export-Csv -Append:$true -NoTypeInformation:$true -Path $CSVFile
    if ($UsageInfo.NextLink) 
    {
        $ContinuationToken = [System.Web.HttpUtility]::UrlDecode($UsageInfo.NextLink.Split("=")[-1])
        $UsageInfo = Get-UsageAggregates -ReportedStartTime $StartTime -ReportedEndTime $EndTime `
                            -AggregationGranularity Daily -ShowDetails $true -ContinuationToken $ContinuationToken
    }
    else
    {
       $ContinuationToken = "" 
   }
 } Until (!$ContinuationToken)  
```


なお実行時に下図のエラーが発生する場合は System.Web アセンブリがよみこまれていない可能性が高いため、以下のコマンドをローカルの環境にてご実施いただければと存じます。

![エラー画面](./gettingExeTimeOfJobForSpecificPeriodAutomation/ErrorMessage.jpg "エラー画面")

**実行していただきたいコマンド**

```ps1
PS C:\> Add-Type -AssemblyName System.Web
```

結果として下図のように日時別のジョブの実行時間が CSV で取得できます。

![](./gettingExeTimeOfJobForSpecificPeriodAutomation/CsvCreation.png)


”Quantity” 欄にて分単位でジョブの実行時間が表示されるのが確認できます。  
この結果では 9 月で約67分のジョブ実行時間なので無償の範囲内ですね。  
月別に出力して、管理することで、変遷を確認するなど運用にお役に立てていただければ幸いです。