---
title: サービス正常性アラートの設定手順と推奨設定について
date: 2021-07-31 00:00:00
tags:
  - How-To
  - Tips
  - Azure Monitor Essentials
---

[更新履歴]  
-2021/07/31 ブログ公開  
-2025/12/19 GUI 変更に伴う更新  

こんにちは、Azure Monitoring & Integration サポート チームの堀です。  
今回はサービス正常性アラートの設定手順と推奨設定についてご案内いたします。

<!-- more -->

## 目次
- サービスの正常性アラートの概要
- サービス正常性アラートの設定手順
- サービス正常性アラートの推奨設定

## サービスの正常性アラートの概要
サービス正常性アラートでは、お客様環境にてご利用いただいている Azure サービスおよびリージョンを監視対象とし、Azure サービス自体の正常性を監視することが可能です。Azure サービスの障害や、計画メンテナンスの情報を監視でき、イベントが発生した際に、メールや SMS 等の方法で管理者にアラート通知できます。

サービス正常性アラートにて検知できるイベントの詳細は、下記弊社公開情報をご覧ください。  
-- Service Health の概要    
https://learn.microsoft.com/ja-jp/azure/service-health/service-health-portal-update#service-health-events  
※ [Service Health のイベント] をご覧ください。

## サービス正常性アラートの設定手順
サービス正常性アラートを設定する方法については以下公開情報をご参照ください。

-- Azure portal で Service Health アラートを作成する  
https://learn.microsoft.com/ja-jp/azure/service-health/alerts-activity-log-service-notifications-portal#alert-and-new-action-group-using-azure-portal

## サービス正常性アラートの推奨設定
サービス正常性アラート設定時の "サービス" や "リージョン" はすべて選択いただくことを推奨しております。  
サービス正常性アラートは、ご利用いただいているサービスのご利用いただいているリージョンを対象としたイベントが発生した場合にのみ通知が行われる仕組みです。  
このため、サブスクリプション上のすべてのサービス、リージョンおよびイベントの種類を選択した場合にも、利用していないリソースに対してアラートは発生いたしません。  
