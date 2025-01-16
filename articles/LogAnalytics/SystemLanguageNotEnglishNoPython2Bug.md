---
title: システム言語が日本語の Linux マシンにて発生する Log Analytics エージェントの不具合について
date: 2022-02-28 00:00:00
tags:
  - Log Analytics
  - FAQ
  - Troubleshooting
---

こんにちは。Azure Monitoring & Integration チームの Austin です。

最近よくお問い合わせをいただいております、Log Analytics エージェントの既知の不具合の影響・原因・回避策についてご紹介いたします。

<!-- more -->

## 本不具合の内容およびその影響
- Log Analytics エージェントのインストールに失敗する
- Log Analytics エージェントからデータが正常に収集されない
- Log Analytics ワークスペースの「エージェント構成」画面より収集設定を変更したが、変更が適用されない
＊ 例えば収集する Syslog のファシリティを追加して、OS 上にログが出力されていることを確認したのに、Log Analytics ワークスペースへ収集されない

## 不具合が発生する原因
以下 2 つの条件で不具合が発生します。
1. システム言語 (ロケール) が英語以外に設定されている
＊ 「localectl」または「echo $LANG」で現在のロケールを確認できます。
2. python2 がインストールされていない
＊ RHEL 8 や CentOS 8、Ubuntu 20.04 等のマシンには python2 が規定では入っていないため、本不具合の影響を受けることが多い

以下の Log Analytics の公開情報にインストール時にシステム言語が英語でないとインストールが失敗する問題について記載しておりますが、**一度システム言語が英語の状態でインストールをして、その後システム言語を日本語等に変更した場合にも不具合が発生する事例を確認しております。そのため、新規 VM に Log Analytics エージェントを正常にインストールできた場合にも、その後システム言語を変更すると不具合が発生します。**

[問題: Python 3 を使用しているのに、Python 2 では Ctype をサポートしていない、というエラーが出てインストールに失敗する](https://docs.microsoft.com/ja-jp/azure/azure-monitor/agents/agent-linux-troubleshoot#issue-installation-is-failing-saying-python2-cannot-support-ctypes-even-though-python3-is-being-used)

## 回避策について
本不具合が発生する条件をいずれか修正することで不具合を回避いただけます。
1.  システム言語を英語に戻す
＊ RHEL 8 の場合は以下のコマンドで設定できます。
`$ sudo localectl set-locale en_US.UTF-8`

2. python2 をインストールする
＊ RHEL 8 の場合は以下のコマンドで設定できます。
`$ sudo dnf install python2 -y`
`$ sudo alternatives --set python /usr/bin/python2`


以上、Log Analytics エージェントで発生している既知の問題と回避策をご案内いたしました。
弊社製品の不具合によりご不便ご迷惑おかけし申し訳ございません。
本不具合については、弊社開発部門にて修正作業を進めております。
本不具合の修正がリリースされた際には本記事を更新いたします。
2022 年 2 月 28 日の最新の情報です。
