---
title: Azure Defender for Key Vault just released!
date: 2020-09-22 17:30:26
tags: [Azure, Security, Key Vault, Machine Learning]
categories: Azure Defender for Key Vault
description: As the service owner, I am super excited to share that Azure Defender for Key Vault is now generally available! It is really One Microsoft experience to work closely with Azure Security Center and Azure Key Vault team to launch this service. Also personally, I grew up a lot after going through the Machine Learning algorithm improvement, infrastructure refactoring, BCDR and privacy policy compliance, cost reduce, monthly business review(MBR), customer feedback investigation. It is indeed a challenging and inspiring work to wake me up every day.
---

As the service owner, I am super excited to share that Azure Defender for Key Vault is now generally available! 

It is really One Microsoft experience to work closely with Azure Security Center and Azure Key Vault team to launch this service. Also personally, I grew up a lot after going through the Machine Learning algorithm improvement, infrastructure refactoring, BCDR and privacy policy compliance, cost reduce, monthly business review(MBR), customer feedback investigation. 

It is indeed a challenging and inspiring work to wake me up every day.

## What is Azure Defender for Key Vault

https://docs.microsoft.com/en-us/azure/security-center/defender-for-key-vault-introduction

Customers are using Azure Key Vault to store the most sensitive information in their Azure environment: keys, passwords, secrets and certificates for all of their Azure resources. By achieving this data, attackers may be able to perform lateral movement and breach other resources in the customers Azure environment.

Azure Defender for Key Vault is a cloud-native, breadth threat protection suite – gives customers additional layer of protection for the precious secretes stored in the Key Vault by helping the SOC team to detect suspicious activities in their Key Vaults and protect the entire Azure environment.

## How to Enable Azure Defender for Key Vault

1. Enable it from Azure Key Vault

   In Key Vault's **Security** page, click "try it for the first 30 days"

   <img src="https://raw.githubusercontent.com/xinyeah/xinyeah.github.io/master/images/image-20200922165536008.png" alt="image-20200922165536008" style="zoom:50%;" />

2. Enable it from Azure Security Center

   https://docs.microsoft.com/en-us/azure/security-center/security-center-pricing#enable-azure-defender

   1. From Security Center's main menu, select **Pricing & settings**.
   2. Select the subscription that you want to upgrade.
   3. Select **Azure Defender on** to upgrade.
   4. Select **Save**.

   Below is the pricing page for an example subscription. You'll notice that each plan in Azure Defender is priced separately and can be individually set to on or off. Make sure it is on for Azure Key Vault.

   ![image-20200922165912513](https://raw.githubusercontent.com/xinyeah/xinyeah.github.io/master/images/image-20200922165912513.png)

## Azure Defender for Key Vault Alerts

https://review.docs.microsoft.com/en-us/azure/security-center/alerts-reference?branch=master#alerts-azurekv

![image-20200922164528455](https://raw.githubusercontent.com/xinyeah/xinyeah.github.io/master/images/image-20200922164528455.png)

## Current Status

We just releasing to GA and we already have: 

- 30G  Azure Key Vault logs processed per month
- 1.2M   Azure Key Vaults protected
- 63K    Azure subscriptions protected

And expecting these numbers to raise dramatically in the current months.

## General Availability Announcement at Ignite 2020

Azure Defender for Key Vault is generally available: https://docs.microsoft.com/en-us/azure/security-center/release-notes#azure-defender-for-key-vault-is-generally-available

What's new in Azure Key Vault: https://techcommunity.microsoft.com/t5/video-hub/azure-key-vault-what-s-new/m-p/1698834

<img src="https://raw.githubusercontent.com/xinyeah/xinyeah.github.io/master/images/image-20200922135102750.png" alt="image-20200922135102750" style="zoom:50%;" />

Introducing Azure Defender: https://myignite.microsoft.com/sessions/764ff397-97ff-4841-ad62-493f1da51d40

<img src="https://raw.githubusercontent.com/xinyeah/xinyeah.github.io/master/images/image-20200922133555333.png" alt="image-20200922133555333" style="zoom:50%;" />

What's new in Azure Security Center: https://myignite.microsoft.com/sessions/d40bd0a5-485e-455d-ac28-882b85de8dfb

<img src="https://raw.githubusercontent.com/xinyeah/xinyeah.github.io/master/images/image-20200922134003684.png" alt="image-20200922134003684" style="zoom:50%;" />

