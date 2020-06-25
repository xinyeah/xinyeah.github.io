---
title: Always flush Application Insights
date: 2020-06-22 21:50:20
tags: Application Insights
categories: Application Insights
description: Databricks docs says, the SDK sends out data at fixed intervals (typically 30 secs) or whenever the buffer is full (typically 500 items). However,  from my personal experience, it won't send the data if you don't flush.
---

## What is Application Insights?

Application Insights is one of Azure Monitoring solutions, it monitors the availability, performance, and usage of your web application. 

## How it works

Before you can use the Application Insights, you need to install an Application Insights SDK (instrumentation package) in your app. This instrumentation monitors your app and sends out the telemetry data  to an Azure Application Insights resource identified by an instrumentation key (a unique GUID).

It support Java, C#, Node.js, python [and so on](https://docs.microsoft.com/en-us/azure/azure-monitor/app/platforms)

## Flush data

The official docs says, the SDK sends out data at fixed intervals (typically 30 secs) or whenever the buffer is full (typically 500 items). 

However,  from my personal experience, **it won't send the data if you don't flush**.

Here is a C# code example to flush the telemetry. 

```c#
// Set up some properties and metrics:
var properties = new Dictionary <string, string>
    {{"game", currentGame.Name}, {"difficulty", currentGame.Difficulty}};
var metrics = new Dictionary <string, double>
    {{"Score", currentGame.Score}, {"Opponents", currentGame.OpponentCount}};

// Send the event:
telemetry.TrackEvent("WinGame", properties, metrics);
// Flush the buffer
telemetry.Flush();
// Allow some time for flushing before shutdown.
System.Threading.Thread.Sleep(5000);
```



## Reference

https://docs.microsoft.com/en-us/azure/azure-monitor/app/api-custom-events-metrics

https://docs.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview