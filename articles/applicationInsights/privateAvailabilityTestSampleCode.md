---
title: Azure Functions を利用したプライベート可用性テストについて
date: 2023-06-08 00:00:00
tags:
  - Application Insights
  - 可用性テスト
  - Tips
---

こんにちは、Azure Monitoring サポート チームの北山です。

Application Insights には可用性テストの機能があります。
可用性テストの機能をご利用いただく事で、監視対象の Web サイトに対して自動的にリクエストを送信することが可能です。  
監視対象の Web サイトからのレスポンスをチェックし、期待したレスポンスが返却されない場合にアラートを通知する事が可能です。

[Application Insights 可用性テスト](https://learn.microsoft.com/ja-jp/azure/azure-monitor/app/availability-overview)

Application Insights のサービスから監視対象の Web サイトに対して定期的にリクエストを送信するため、監視対象の Web サイトはパブリックからの通信を許可いただく必要がございます。  
しかし、お客様の環境によってはパブリックからの通信を許可していない可能性がございます。  
この場合は、標準の可用性テストがご利用いただけません。

この記事では、TimerTrigger 関数で指定された構成に従って定期的に実行される、独自のテスト ロジックが実装された Azure Functions から、Application Insights API の一つである TrackAvailability() を使って可用性テストの結果を送信する方法について説明します。

<!-- more -->

## 目次
- [目次](#目次)
- [Azure Functions を用いたプライベート可用性テスト構築の手順](#azure-functions-を用いたプライベート可用性テスト構築の手順)
  - [1. VNET 統合を実施した Azure Functions を用意します。](#1-vnet-統合を実施した-azure-functions-を用意します)
  - [2. Azure Functions にテスト ロジックを実装します。](#2-azure-functions-にテスト-ロジックを実装します)
- [まとめ](#まとめ)
- [関連する記事](#関連する記事)


## Azure Functions を用いたプライベート可用性テスト構築の手順
### 1. VNET 統合を実施した Azure Functions を用意します。
パブリックからの通信を許可していない Web サイトにアクセスするために、VNET 統合した Azure Functions リソースを準備します。
VNET 統合を実施すると、Azure Functions からの通信が VNET を経由します。  
監視対象の Web サイトにアクセス可能な VNET 上に Azure Functions リソースをデプロイすることで、定期的に監視対象の Web サイトへテストのためのリクエストが可能です。

Azure Functions の VNET 統合の方法につきましては、下記の公開情報をご参考くださいませ。

- [チュートリアル: プライベート エンドポイントを使用して Azure Functions を Azure 仮想ネットワークに統合する](https://learn.microsoft.com/ja-jp/azure/azure-functions/functions-create-vnet)


### 2. Azure Functions にテスト ロジックを実装します。
その 1 でご準備いただいた Functions リソースに対して、タイマー トリガーの関数を作成します。
関数名やスケジュールは、貴社のご要件に合うよう適宜ご指定くださいませ。

![](./privateAvailabilityTestSampleCode/function1.png)


その後、当該 Functions リソースの左側ペインより「App Service Editor (プレビュー)」を開きます。

![](./privateAvailabilityTestSampleCode/function2.png)

下図のように、当該関数に対して右クリックし新しいファイルを作成します。  
作成するファイル名は、「function.proj」です。

![](./privateAvailabilityTestSampleCode/function3.png)

function.proj に対して、下記のコードを貼り付けます。

```xml
<Project Sdk="Microsoft.NET.Sdk"> 
 <PropertyGroup> 
 <TargetFramework>netstandard2.0</TargetFramework> 
 </PropertyGroup> 
 <ItemGroup> 
 <PackageReference Include="Microsoft.ApplicationInsights" Version="2.15.0" /> <!-- Ensure you’re using the latest version --> 
 </ItemGroup> 
</Project>
```
![](./privateAvailabilityTestSampleCode/function4.png)


次に runAvailabilityTest.csx というファイルを作成し、下記のコードを貼り付けます。  
> 下記のコードは、監視対象の Web サイトに対して GET リクエストを送信するコードです。  
> こちらはサンプル コードのため、Bing のサイトにアクセスするよう実装しております。  

```cs
using System.Net.Http; 
 
public async static Task RunAvailabilityTestAsync(ILogger log) 
{ 
 using (var httpClient = new HttpClient()) 
 { 
 // TODO: Replace with your business logic 
 await httpClient.GetStringAsync("https://www.bing.com/"); 
 } 
}
```
![](./privateAvailabilityTestSampleCode/function5.png)


次のコードを run.csx にコピーします。  
これによって既存のコードが全て置き換えられます。
```cs
#load "runAvailabilityTest.csx" 
using System; 
using System.Diagnostics; 
using Microsoft.ApplicationInsights; 
using Microsoft.ApplicationInsights.Channel; 
using Microsoft.ApplicationInsights.DataContracts; 
using Microsoft.ApplicationInsights.Extensibility; 
private static TelemetryClient telemetryClient; 
 
// ============================================================= 
// ****************** DO NOT MODIFY THIS FILE ****************** 
// Business logic must be implemented in RunAvailabilityTestAsync function in runAvailabilityTest.csx 
// If this file does not exist, please add it first 
// ============================================================= 
public async static Task Run(TimerInfo myTimer, ILogger log, ExecutionContext executionContext) 
{ 
 if (telemetryClient == null) 
 { 
 // Initializing a telemetry configuration for Application Insights based on connection string 
 var telemetryConfiguration = new TelemetryConfiguration(); 
 telemetryConfiguration.ConnectionString = Environment.GetEnvironmentVariable("APPLICATIONINSIGHTS_CONNECTION_STRING"); 
 telemetryConfiguration.TelemetryChannel = new InMemoryChannel(); 
 telemetryClient = new TelemetryClient(telemetryConfiguration); 
 } 
 
string testName = executionContext.FunctionName; 
 string location = Environment.GetEnvironmentVariable("REGION_NAME"); 
var availability = new AvailabilityTelemetry 
 { 
 Name = testName, 
 RunLocation = location, 
 Success = false, 
 }; 
 
 availability.Context.Operation.ParentId = Activity.Current.SpanId.ToString(); 
 availability.Context.Operation.Id = Activity.Current.RootId; 
 var stopwatch = new Stopwatch(); 
 stopwatch.Start(); 
 
try 
 { 
 using (var activity = new Activity("AvailabilityContext")) 
 { 
 activity.Start(); 
 availability.Id = Activity.Current.SpanId.ToString(); 
 // Run business logic 
 await RunAvailabilityTestAsync(log); 
 } 
 availability.Success = true; 
 } 
 
 catch (Exception ex) 
 { 
 availability.Message = ex.Message; 
 throw; 
 } 
 
 finally 
 { 
 stopwatch.Stop(); 
 availability.Duration = stopwatch.Elapsed; 
 availability.Timestamp = DateTimeOffset.UtcNow; 
telemetryClient.TrackAvailability(availability); 
 telemetryClient.Flush(); 
 } 
}
```

その後、当該関数が実行されると、テスト コードの結果として Application Insights に可用性テストの結果が記録されます。

![](./privateAvailabilityTestSampleCode/function6.png)



上記サンプル コードは、あくまでサンプルです。  
お客様の環境によっては GET リクエストではなく POST リクエストの実行結果を確認する事も想定されます。  
そのため、お客様の監視要件に合わせてテスト コードを実装いただけますと幸いです。


## まとめ
本記事では、パブリックからのアクセスを許可していない環境への可用性テストの方法ついてご案内いたしましたが、ご理解いただけましたでしょうか。

本記事が少しでもお役に立ちましたら幸いです。
最後までお読みいただきありがとうございました！

## 関連する記事
- [プライベート環境への可用性テスト](https://jpazmon-integ.github.io/blog/applicationInsights/aboutPrivateAvailabilityTest/)