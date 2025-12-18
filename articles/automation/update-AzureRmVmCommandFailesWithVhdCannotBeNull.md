---
title: 【非推奨】 Runbook の Update-AzureRmVM コマンドが 'Vhd' cannot be null. で失敗する
date: 2019-4-3 02:26:00
tags:
  - Automation
  - Troubleshooting
  - Runbook
---

[更新履歴]
- 2019/04/03 ブログ公開
- 2021/05/24 最新情報に更新
- 2025/12/18 非推奨について追記

>[!IMPORTANT]
>AzureRM PowerShell モジュールは、[2024 年 2 月 29 日の時点で正式に非推奨](https://learn.microsoft.com/ja-jp/powershell/azure/azurerm-retirement-overview?view=azps-15.1.0)になりました。 
>引き続きサポートと更新を行うために、AzureRM から Az PowerShell モジュールに移行することをお勧めします。
>AzureRM モジュールの機能は今後も使用できますが、メンテナンスやサポートは行われないため、引き続きの使用はユーザーの判断に委ねられ、リスクが発生することがあります。 
>Az モジュールへの移行に関するガイダンスについては、[移行リソース](https://learn.microsoft.com/ja-jp/powershell/azure/migrate-from-azurerm-to-az?view=azps-15.1.0)を参照してください。

こんにちは、Windows プラットフォーム サポートの世古です。  
今回は Azure  Automation の Runbook 実行でよくお問い合わせいただく内容についてご紹介させていただきます。

Runbook の実行で Update-AzureRmVM が 'Vhd' cannot be null. で失敗する事象がございます。  
この問題は以下の条件で発生します。

- 対象の VM が管理ディスクを持つ
- モジュールのバージョンが最新でない

この問題はモジュールのバージョンを最新に更新いただく事で回避されますので、以下の公開情報の手順を実施下さい。

- [Azure Automation の Azure PowerShell モジュールを更新する方法](https://docs.microsoft.com/ja-jp/azure/automation/automation-update-azure-modules)

以前の Powershell モジュールにおいては、管理ディスクを操作可能な実装が含まれていなかった為、VM の情報を正常に取得出来なかった為、本問題が発生いたします。その為、モジュールのアップデートをご検討ください。