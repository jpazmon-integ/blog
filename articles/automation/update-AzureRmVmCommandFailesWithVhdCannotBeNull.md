---
title: Runbook の Update-AzureRmVM コマンドが 'Vhd' cannot be null. で失敗する
date: 2019-4-3 02:26:00
tags:
  - Automation
  - Troubleshooting
  - Runbook
---

こんにちは、Windows プラットフォーム サポートの世古です。  
今回は Azure  Automation の Runbook 実行でよくお問い合わせいただく内容についてご紹介させていただきます。

Runbook の実行で Update-AzureRmVM が 'Vhd' cannot be null. で失敗する事象がございます。  
この問題は以下の条件で発生します。

- 対象の VM が管理ディスクを持つ
- モジュールのバージョンが最新でない

この問題はモジュールのバージョンを最新に更新いただく事で回避されますので、以下の公開情報の手順を実施下さい。

- [Azure Automation の Azure PowerShell モジュールを更新する方法](https://docs.microsoft.com/ja-jp/azure/automation/automation-update-azure-modules)

以前の Powershell モジュールにおいては、管理ディスクを操作可能な実装が含まれていなかった為、VM の情報を正常に取得出来なかった為、本問題が発生いたします。その為、モジュールのアップデートをご検討ください。