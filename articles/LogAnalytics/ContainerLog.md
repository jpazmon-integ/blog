
---
title: Container Insights を使用したログ収集の概要、Tips のご紹介
date: 2024-05-10 00:00:00
tags:
 - How-To
 - Log Analytics
 - Container Insights
---

こんにちは！Azure Monitoring チームの加治屋です。
この記事では、Container Insights のログ収集の概要、および Tips をご紹介いたします。

<!-- more -->

## Container Insights とは？
AKS クラスターをはじめとした コンテナー環境へデプロイすることで、コンテナー環境の情報を Log Analytics ワークスペースに収集可能になる製品です。
Log Analytics ワークスペースへ収集した情報は通常の VM などから収集したログと同じように、KQL を使用した情報の分析やログ アラート ルールをはじめとした処理に使用することができます。

Kubernetes 監視用の Azure Monitor の機能
https://learn.microsoft.com/ja-jp/azure/azure-monitor/containers/container-insights-overview 

この機能をデプロイすると、コンテナー環境へ Azure Monitor エージェント Pod がデプロイされます。

Kubernetes 監視用の Azure Monitor の機能 - エージェント
https://learn.microsoft.com/ja-jp/azure/azure-monitor/containers/container-insights-overview#agent 

## Container Insights のログ監視とは？
Container Insights では、コンテナー環境内の Pod 内で出力されたログの内容を収集することが可能です。
ログ収集対象となるログは、標準出力 (stdout) または 標準エラー出力 (stderr) のどちらか、もしくはその両方に出力されたログとなります。

なお、コンテナー ログの収集について、ログ インジェストの形式により V1 と V2 のどちらかの形式を選択するように構成が行われます。
どちらの形式を使用するかにより、ログが収集されるテーブル名も異なります。
V1 の場合は ContainerLog テーブル、V2 の場合は ContainerLogV2 テーブルとなり、現在のデフォルト設定は V2 でございます。
なお、ContainerLog テーブルは、現時点では 2026 年 9 月 30 日にサポート終了予定となっております。ご利用されている場合はお早めに移行をいただけますと幸いです。

Container Insights のログ スキーマ
https://learn.microsoft.com/ja-jp/azure/azure-monitor/containers/container-insights-logs-schema#table-comparison 

## Container Insights のログ収集設定
V1 を使用する場合は ConfigMap、V2 を使用する場合は ConfigMap および データ収集ルールにてログ収集設定が可能です。
それぞれで設定できる項目は異なりますので、お客様のご要件に応じてご設定ください。

ConfigMap を使用して Container Insights でデータ収集を構成する
https://learn.microsoft.com/ja-jp/azure/azure-monitor/containers/container-insights-data-collection-configmap#configure-and-deploy-configmap

データ収集ルールを使用して Container Insights でデータ収集とコスト最適化を構成する
https://learn.microsoft.com/ja-jp/azure/azure-monitor/containers/container-insights-data-collection-dcr?tabs=portal

## ログ収集を行う前に ...
以下のコマンドをご実行いただき、Pod の stdout もしくは stderr にて収集されたいログが出力されておりますことをご確認ください。
$ kubectl logs <pod-name> -c <container-name>

## Tips
ここからは、Container Inisghts のログ監視について Tips をご案内いたします。

### Pod 内のファイルシステム上に書き込んでいるファイルの内容を stdout に出力してログ収集が行われるようにする
Pod をデプロイする際の yaml ファイルを修正し、サイドカーを利用した仕組みとすることで、ある程度実現することが可能です。

以下のサンプル yaml ファイルをデプロイすると、ログ ファイルへ出力された内容をサイドカーを利用して stdout に出力いたします。
/var/log/*.log に出力されたログを サイドカーの tail コマンドにて stdout に出力、この結果が Container Insights により Log Analytics ワークスペースの ContainerLog テーブル (ContainerLogV2 テーブル) に収集される仕組みでございます。

<サンプル yaml ファイル>
```
apiVersion: v1
kind: Pod
metadata:
  name: two-files-logging-counter
spec:
  containers:
  - name: two-files-logging-count
    image: busybox
    args:
   - /bin/sh
   - -c
   - >
     i=0;
     while true;
     do
       echo "$i: $(date)" >> /var/log/1.log;
       echo "$(date) INFO $i" >> /var/log/2.log;
       i=$((i+1));
       sleep 1;
     done
   volumeMounts:
   - name: varlog
     mountPath: /var/log
 - name: count-log-1
   image: busybox
   args: [/bin/sh, -c, 'tail -n+1 -F /var/log/*.log | grep -v -e "^\s*$" -e "^=="']
   volumeMounts:
   - name: varlog
     mountPath: /var/log
 volumes:
 - name: varlog
   emptyDir: {}
```

### Pod 内のファイル システムに新規作成されるログ ファイルの中身を ContainerLog (ContainerLogV2) に送信する
上記についても yaml ファイルを修正し、サイドカーを利用する仕組みとすることである程度の実現が可能です。

サイドカー側でファイルのリストを生成し、定期的に新しいファイルのチェックを行います。
リストに載っているファイルをログの収集対象として tail コマンドを実行し stdout にログを出力、Log Analytics ワークスペースの ContainerLog テーブル (ContainerLogV2 テーブル) に収集される仕組みでございます。

```
apiVersion: v1
kind: Pod
metadata:
  name: datetime-logging-counter
spec:
  containers:
  - name: datetime-logging-count
    image: busybox
    args:
    - /bin/sh
    - -c
    - >
      while true; do
        date_time=$(date "+%Y%m%d");
        echo "$i: $date_time" >> /var/log/$date_time.log;
        echo "$date_time INFO $i" >> /var/log/$date_time.log;
        i=$((i+1));
        sleep 1;
      done
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  - name: read-log
    image: busybox
    args:
    - /bin/sh
    - -c
    - >
      log_position_file="/var/log/last_position.txt"
 
      if [ ! -f "$log_position_file" ]; then
        echo "0" > "$log_position_file"
      fi
 
      while true; do
        date_time=$(date "+%Y%m%d");
        log_file_name="/var/log/$date_time.log"
        last_position=$(expr $(cat "$log_position_file") + 1)
 
        if [ -f "$log_file_name" ]; then
            new_log=$(tail -c +$last_position "$log_file_name")
 
            if [ -n "$new_log" ]; then
                echo "reading $log_file_name ...:"
                echo "$new_log"
 
                new_position=$(wc -c < "$log_file_name")
                echo "$new_position" > "$log_position_file"
            fi
        else
            echo "Log file $log_file_name does not exist."
        fi
 
        sleep 5
      done
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  volumes:
  - name: varlog
    emptyDir: {}
```

もし、Container Insights にてお困りごとがございましたら、遠慮なく弊社サポートまでお問い合わせください。

## 参考情報
- Kubernetes 監視用の Azure Monitor の機能
https://learn.microsoft.com/ja-jp/azure/azure-monitor/containers/container-insights-overview

- Container Insights のログ スキーマ
https://learn.microsoft.com/ja-jp/azure/azure-monitor/containers/container-insights-logs-schema

