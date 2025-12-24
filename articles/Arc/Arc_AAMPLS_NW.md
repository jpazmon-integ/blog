---
title: Azure Private Link  を使った Azure Arc への接続 - NW 構成
date: 2024-03-30 17:00:00
tags:
  - Azure Arc
  - HowTo
  - Azure Private Link
---

<!-- more -->
こんにちは、Azure Monitoring チームの 佐藤 です。

今回は、Azure Private Link を使った Azure Arc への接続に関するよくあるお問い合わせ内容とその回答について紹介したいと思います。
Azure Arc にオンボードさせるにはオンプレミスマシン（もしくは AWS のようなクラウド上の VM）には、Azure Connected Machine エージェント （以後、Arc エージェントと呼称します）をインストールする必要があります。

その Arc エージェントを使用して Azure Arc にオンボード（管理状態）にするには以下ネットワーク要件を満たす必要がございます。
[Connected Machine エージェントのネットワーク要件](https://learn.microsoft.com/ja-jp/azure/azure-arc/servers/network-requirements?tabs=azure-cloud)

ここで Azure Private Link を使った Azure Arc への接続については以下ページに基本構成を記載しております。
[Azure Private Link を使用して、サーバーを Azure Arc に安全に接続する](https://learn.microsoft.com/ja-jp/azure/azure-arc/servers/private-link-security)

上記ページに記載しておりますが、Azure Private Link を使った Azure Arc への接続は以下が基本構成となります。
一部情報補記しております
■図1 Azure Private Link を使った Azure Arc 構成イメージ
![](Arc_AAMPLS_NW/01.png)

※1 後述する説明のために ExpressRoute などの Azure 側の終端となる *仮想ネットワーク ゲートウェイ* の情報を記載しております。


## Q1. Microsoft Entra ID と ARM には閉域で接続できるか
まず上図 1 とネットワーク要件に記載されている内容で多くのお問い合わせいただく以下 QA から説明します。

Q1. 
Microsoft Entra ID (旧 Azure Active Directory) および ARM (Azure Resource Manager) に閉域で接続できますか？

A1. 
いいえ。接続できません。

*説明*
上記ネットワーク要件を記載している [ページ](https://learn.microsoft.com/ja-jp/azure/azure-arc/servers/network-requirements?tabs=azure-cloud#urls) に記載しておりますとおり、下図の `プライベート リンク対応` 列に ”パブリック” と表記されている列の宛先 （download.microsoft.com や packages.microsoft.com）はインターネット経由で接続する必要がございます。

■図2. NW 要件
![](Arc_AAMPLS_NW/02.png)

上図 1 で言及しますと "On-premises / Cloud network" から上に Firewall を超え、インターネット経由で Microsoft Entra ID および ARM へ接続されるイメージがわかるかと存じます。


## Q2. Microsoft Entra ID と ARM への通信を Azure network 経由で通信できるか
ここで On-premises / Cloud network から Microsoft Entra ID と ARM への通信も Azure Virtual Network 経由で接続できるかについて説明いたします。

■図3.  通信イメージ
![](Arc_AAMPLS_NW/03.png)

Q2. 
Microsoft Entra ID (旧Azure Active Directory) および ARM (Azure Resource Manager) への通信を Azure Virtual Network 経由で接続できますか？

A2. 
仕組み上可能となりますが、推奨しておりません。
まず仮想ネットワーク ゲートウェイにはインターネットへ接続するための機能は持っておりません。そのため、インターネットへ接続するためのサービスなどをお客様の Azure Virtual Network 環境に構築いただく必要がございますが、追加の機能のデプロイや設定が必要となり 2025 年 12 月時点で当社としては推奨しておりません。


## Q3.  公開情報の『ネットワーク構成』に記載されているオプション 1 はどのようなシナリオですか。
上記に関連して公開情報 の [ネットワーク構成](https://learn.microsoft.com/ja-jp/azure/azure-arc/servers/private-link-security#network-configuration) に記載されている下図のオプション 1 の以下文言について質問を頂戴することがあります。

```
すべてのインターネットにバインドされたトラフィックを Azure VPN または ExpressRoute 回線経由でルーティングするようにネットワークが構成されている場合
```

■図4. オプション 1 の説明箇所
![](Arc_AAMPLS_NW/04.png)


Q3. 
公開情報の『ネットワーク構成』に記載されているオプション 1 はどのようなシナリオですか。

A3.
Azure Virtual Network からインターネット（パブリック エンドポイント）へ通信させる場合に、全てのトラフィックを On-premises / Cloud network へルーティングする場合となります。
上図 1 ですと "Azure Virtual Network" から "On-premises / Cloud network" へルーティングすることを意図しております。
具体的なルーティング設定の例としては "On-premises network" 側のお客様 NW 機器などから BGP にて デフォルトルート （0.0.0.0/0）の広告が仮想ネットワーク ゲートウェイ側に届いている場合のシナリオとなります。


本日のご紹介は以上となります。上記の内容以外でご不明な点や疑問点などございましたら、弊社サポート サービスまでお問い合わせください。
最後までお読みいただきありがとうございました。
