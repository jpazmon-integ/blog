---
title: 実行アカウントとマネージド ID の違いについて
date: 2022-06-10 00:00:00
tags:
  - Automation
  - RunAs Account
  - マネージド ID
---

こんにちは、Azure Monitoring サポート チームの六浦です。
今回は Automation アカウントの実行アカウントとマネージド ID の違いについてご紹介いたします。

<!-- more -->

## 目次
- [目次](#目次)
- [実行アカウントについて (2023 年 9 月 30 日に廃止)](#実行アカウントについて-2023-年-9-月-30-日に廃止)
  - [参考情報](#参考情報)
- [マネージド ID について](#マネージド-id-について)
  - [参考情報](#参考情報-1)
- [実行アカウントとマネージド ID の共通点](#実行アカウントとマネージド-id-の共通点)
- [実行アカウントとマネージド ID の相違点](#実行アカウントとマネージド-id-の相違点)
- [まとめ](#まとめ)

## 実行アカウントについて (2023 年 9 月 30 日に廃止)
実行アカウントは、Automation アカウントに利用される Microsoft Entra ID (旧 Azure AD) のサービス プリンシパルです。
通常、以下のコマンドで実行アカウントを使って Azure へ認証するように Runbook を構成できます。

```
#Get the connection "AzureRunAsConnection"
$Conn=Get-AutomationConnection -Name $connectionName         

"Logging in to Azure..."
Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint
```

### 参考情報
[既存の実行アカウントからマネージド ID に移行する](https://learn.microsoft.com/ja-jp/azure/automation/migrate-run-as-accounts-managed-identity?tabs=sa-managed-identity)


## マネージド ID について
マネージド ID は、Microsoft Entra ID (旧 Azure AD) 認証をサポートするリソースに接続するときに使用する ID をアプリケーションに提供します。
アプリケーションは、マネージド ID を使用して Microsoft Entra ID トークン (旧 Azure AD トークン) を取得できます。
マネージド ID にはシステム割り当てマネージド ID とユーザー割り当てマネージド ID があります。

以下のコマンドでシステム割り当てマネージド ID を使って Azure へ認証するように Runbook を構成します。

```
# Ensures you do not inherit an AzContext in your runbook
Disable-AzContextAutosave -Scope Process

# Connect to Azure with system-assigned managed identity
$AzureContext = (Connect-AzAccount -Identity).context

# set and store context
$AzureContext = Set-AzContext -SubscriptionName $AzureContext.Subscription -DefaultProfile $AzureContext
```


また、ユーザー割り当てマネージド ID では、以下のコマンドとなります。
```
# Ensures you do not inherit an AzContext in your runbook
Disable-AzContextAutosave -Scope Process

# Connect to Azure with user-assigned managed identity
$AzureContext = (Connect-AzAccount -Identity -AccountId <user-assigned-identity-ClientId>).context

# set and store context
$AzureContext = Set-AzContext -SubscriptionName $AzureContext.Subscription -DefaultProfile $AzureContext
```

マネージド ID は、Runbook で認証を行うための推奨される方法であり、現時点で Automation アカウントの既定の認証方法です。

### 参考情報
[Azure Automation アカウントの認証の概要 -マネージド ID | Microsoft Docs](https://docs.microsoft.com/ja-jp/azure/automation/automation-security-overview#managed-identities)
[Azure Automation アカウントのシステム割り当てマネージド ID を使用する](https://docs.microsoft.com/ja-jp/azure/automation/enable-managed-identity-for-automation)
[Azure Automation アカウントのユーザー割り当てマネージド ID を使用する](https://docs.microsoft.com/ja-jp/azure/automation/add-user-assigned-identity)


## 実行アカウントとマネージド ID の共通点
- Azure へ認証する役割を果たします。
- Runbook に上述の認証用コマンドを記載する必要があります。
- Microsoft Entra ID (旧 Azure AD) にアプリケーションとして登録されています。


## 実行アカウントとマネージド ID の相違点
実行アカウントの実体はサービス プリンシパルでございます。
そのため、サービス プリンシパルとマネージド ID の違いから、実行アカウントとマネージド ID の相違点をより深く理解できます。

サービス プリンシパルとマネージド ID の大きな違いは、マネージド ID は「割り当てられたリソースからしか使用できない」という点でございます。
サービス プリンシパルは、クライアント シークレットや証明書などを任意のクライアントに格納して使用できますが、マネージド ID は割り当てられたリソースからしか使用できません。つまり、マネージド ID はクライアント シークレットや証明書の流出による不正アクセスのリスクがございません。

この 2 つは以下のように使い分けいただけます。
- 任意のクライアント (Azure 上に存在しないものを含む) で使用したい場合はサービス プリンシパルを使用
- Azure 上に存在する特定のリソースでのみ使用したい場合はマネージド IDを使用

Azure 上に存在する Automation アカウントでは、マネージド ID のご使用を推奨しております。


最後に、実行アカウントとマネージド ID の相違点をまとめると以下になります。これらの相違点より、マネージド ID が推奨される認証方法です。
- 実行アカウントは、証明書を更新する必要があります。
- マネージド ID は、証明書を更新する必要がないです。
- 特定のリソース (Autoamtion アカウント) からアクセスできるという観点から、実行アカウントより、マネージド ID がセキュアです。


## まとめ
本記事では、実行アカウントとマネージド ID の共通点と相違点ついてご案内いたしましたが、ご理解いただけましたでしょうか。

本記事が少しでもお役に立ちましたら幸いです。
最後までお読みいただきありがとうございました！