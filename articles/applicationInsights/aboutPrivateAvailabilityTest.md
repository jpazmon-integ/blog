---
title: プライベート環境への可用性テスト
date: 2022-05-18 00:00:00
tags:
  - Application Insights
  - 可用性テスト
  - Tips
---

こんにちは、Azure Monitoring サポート チームの六浦です。

今回は、以下のようなパブリックからのアクセスを許可していない環境に対して Application Insights の可用性テストを行う方法をご案内いたします。
- ファイアウォールを設定している
- パブリック IP アドレスが設定されていない etc.

<!-- more -->

## 目次
- [目次](#目次)
- [パブリックからのアクセスを許可していない環境への可用性テスト①](#パブリックからのアクセスを許可していない環境への可用性テスト)
- [パブリックからのアクセスを許可していない環境への可用性テスト②](#パブリックからのアクセスを許可していない環境への可用性テスト-1)
- [まとめ](#まとめ)

## パブリックからのアクセスを許可していない環境への可用性テスト①
パブリック DNS レコードが存在し、ファイアウォールなどを使用してパブリックからのアクセスを制限した環境に対しては、サービス タグまたはテストの送信元のサーバーの IP アドレスを許可してテストを行うことができます。

可用性テストの送信元のサーバーの IP アドレスはサービス タグ "ApplicationInsightsAvailability" にて定義しております。このサービス タグをネットワーク セキュリティ グループなどで受信の許可を設定して、可用性テストを実施できます。
![](./aboutPrivateAvailabilityTest/1.png)



または、リクエストを送信するテストの場所ごとの IP アドレスに対して受信の許可を設定し、テストを行うことも可能でございます。
テストの場所ごとの IP アドレスは以下の弊社公開情報にて提供しております。
[場所ごとにグループ化されたアドレス (Azure パブリック クラウド)](https://docs.microsoft.com/ja-jp/azure/azure-monitor/app/ip-addresses#addresses-grouped-by-location-azure-public-cloud)



[参考情報]
[プライベート可用性テスト - Azure Monitor Application Insights - Azure Monitor | Microsoft Docs](https://docs.microsoft.com/ja-jp/azure/azure-monitor/app/availability-private-test)



## パブリックからのアクセスを許可していない環境への可用性テスト②
パブリック DNS レコードが存在せず仮想ネットワーク内からのみアクセスできる環境やサービス タグを許可できない環境に対しては、Azure Functions を使用して可用性テストを行います。

1. Application Insights を有効化した Azure Functions を用意します。
   Application Insights が有効になっているかは、Azure Functions の以下の画面からご確認いただけます。
![](./aboutPrivateAvailabilityTest/2.png)

    > [!NOTE]
    > 仮想ネットワーク内で可用性テストを行う場合は、Azure Functions をテスト対象のサーバーと同じ仮想ネットワークに対して VNET 統合を設定する必要がございます。
    > Azure Functions で VNET 統合を利用するには、従量課金の代わりに Premium プランを使用します。
    > また、仮想ネットワークを使用して Azure Functions からストレージ アカウントへアクセスする場合は、以下の弊社公開情報を参考に、Azure Functions とストレージ アカウントを構成ください。
    > [仮想ネットワークで Azure Functions を構成する方法](https://docs.microsoft.com/ja-jp/azure/azure-functions/configure-networking-how-to#restrict-your-storage-account-to-a-virtual-network)

2. タイマー トリガー関数を作成します。
   以下の弊社公開情報にて、可用性テストを行うためのソース コードの実装例を提供しております。
    [App Service Editor でコードを追加して編集する](https://docs.microsoft.com/ja-jp/azure/azure-monitor/app/availability-azure-functions#add-and-edit-code-in-the-app-service-editor)

    runAvailabilityTest.csx で、テスト先の URL を指定します。
![](./aboutPrivateAvailabilityTest/3.png)

3. Application Insights の [可用性] を開き、可用性テストが動いていることを確認します。
   Azure Functions で作成した可用性テストは、[CUSTOM] と表示されます。
![](./aboutPrivateAvailabilityTest/4.png)

[参考情報]
[Azure Functions を使用してカスタム可用性テストを作成して実行する - Azure Monitor | Microsoft Docs](https://docs.microsoft.com/ja-jp/azure/azure-monitor/app/availability-azure-functions)

## まとめ
本記事では、パブリックからのアクセスを許可していない環境への可用性テストの方法ついてご案内いたしましたが、ご理解いただけましたでしょうか。

本記事が少しでもお役に立ちましたら幸いです。
最後までお読みいただきありがとうございました！

