---
title: Automation アカウントの実行アカウントを作成するスクリプトにてエラーが発生して失敗する
date: 2022-6-xx xx:xx:xx
tags:
  - Automation
  - RunAs Account
---

こんにちは、Azure Monitor サポートの三輪です。
今回は Automation アカウントに対して実行アカウントを作成するスクリプト "Create-RunAsAccount.ps1" にて、エラーが発生する問題についてご案内させていただきます。

- 実行アカウントを作成する PowerShell スクリプト
https://docs.microsoft.com/ja-JP/azure/automation/create-run-as-account#powershell-script-to-create-a-run-as-account


実行アカウントを作成するためにスクリプト "Create-RunAsAccount.ps1" を実行すると、以下の様なエラーが発生して失敗する場合があります。

```
PS C:\temp> .\Create-RunAsAccount.ps1 -ResourceGroup #### `
>> -AutomationAccountName #### `
>> -SubscriptionId #### `
>> -ApplicationDisplayName #### `
>> -SelfSignedCertPlainPassword #### `
>> -CreateClassicRunAsAccount $false

Account              SubscriptionName            TenantId                        Environment
-------              ----------------            --------                        -----------
####@contoso.com     #######                     XXXX-XXXX-XXXX-XXXX-XXXX        AzureCloud

New-AzADApplication_CreateExpanded: C:\Program Files\WindowsPowerShell\Modules\Az.Resources\5.4.0\MSGraph.Autorest\custom\New-AzADApplication.ps1:702
Line |
 702 |      $app = Az.MSGraph.internal\New-AzADApplication @PSBoundParameters
     |      ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
     | Values of identifierUris property must use a verified domain of the organization or its subdomain:
     | 'http://XXXX-XXXX-XXXX-XXXX-XXXX'

New-AzADAppCredential: C:\temp\Create-RunAsAccount.ps1:42
Line |
  42 |  … w-AzADAppCredential -ApplicationId $Application.ApplicationId -CertVa …
     |                                       ~~~~~~~~~~~~~~~~~~~~~~~~~~
     | Cannot process argument transformation on parameter 'ApplicationId'. Cannot convert null to type
     | "System.Guid".

New-AzADServicePrincipal: C:\temp\Create-RunAsAccount.ps1:44
Line |
  44 |  …  = New-AzADServicePrincipal -ApplicationId $Application.ApplicationId
     |                                               ~~~~~~~~~~~~~~~~~~~~~~~~~~
     | Cannot process argument transformation on parameter 'ApplicationId'. Cannot convert null to type
     | "System.Guid".

Get-AzADServicePrincipal: C:\temp\Create-RunAsAccount.ps1:45
Line |
  45 |  … cePrincipal = Get-AzADServicePrincipal -ObjectId $ServicePrincipal.Id
     |                                                     ~~~~~~~~~~~~~~~~~~~~
     | Cannot bind argument to parameter 'ObjectId' because it is an empty string.
:
:
```


上記エラーの原因は、Create-RunAsAccount.ps1 にて利用している New-AzADApplication コマンドが 古い identifierUris 形式になっているためです。
2021 年 10 月 15 日より、Azure AD に登録されるシングルテナント アプリケーションの AppId URI (identifierUris) には、
デフォルトのスキーム (api://{appId}) または検証済みドメインのいずれかが要求されるようになりましたが、現在公開されている Create-RunAsAccount.ps1 にはまだその内容が反映されておりません。

※ Azure AD 側の変更につきまして、以下の公開情報をご参照ください。
- Values of identifierUris property must use a verified domain of the organization or its subdomain エラーの対処法
https://jpazureid.github.io/blog/azure-active-directory/aad-changes-impacting-azurecli-azureps/


また、スクリプトでは New-AzADApplication にて AAD アプリを作成し、そのアプリのアプリケーション ID を $Application.ApplicationId にて取得しております。
これも誤った記述となっており、アプリケーション ID を取得するためには、$Application.AppId と記述する必要があります。
この記述の間違いにより、New-AzADAppCredential 等のコマンドにおいてエラーが発生している状況です。

これらの問題に対応するため、変更を行ったスクリプトを以下にご案内いたします。
お手数をおかけしますが、上記エラーが発生する場合は、以下のスクリプトに置き換えて実行いただければと存じます。

[Create-RunAsAccount_new.zip](./Create-RunAsAccount_new.zip)

なお、現時点では Automation アカウントの認証では実行アカウントではなく、マネージド ID を推奨しております。
マネージド ID の利点としては、実行アカウントの様に証明書を更新する必要がないこと、実行アカウントよりセキュアな認証であることがあげられます。

マネージド ID については、以下の Blog にてご案内しております。こちらも是非ご覧下さい。

- 実行アカウントとマネージド ID の違いについて
https://jpazmon-integ.github.io/blog/automation/DifferenceRunAsAccountAndManagedID/

