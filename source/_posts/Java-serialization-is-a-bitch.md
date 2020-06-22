---
title: Java serialization is a bitch!
date: 2020-06-21 21:54:05
tags: JAVA, serialization
categories: Effective Java reading notes
description: There is no reason to use Java Serialization anymore.
---

## Concept

**Object Serialization** is the process of converting an object into a stream of bytes to store or transmit the object between machines. 

The reverse process is called **deserialization** to use the byte stream to recreate the object.

## Issue for Java Serialization

The main concern to use Java Serialization is security issue. There is a so called **Java deserialization vulnerability** affect all apps that receives serialized Java objects which can be used by attackers to gain complete remote control of an app service. Also, the attack surface is so big and even if you adhere to all best practice, your app is still be vulnerable.

What's the vulnerability?

Many apps that accept serialized bytes stream do not **validate** or **check** untrusted input before deserialization. The attackers can insert a malicious code into the bytes stream and have it execute on the app. They can easily mount a *denial-of-service* attack by causing the deserialization takes forever, which is called *deserialization bomb*. 

## Solutions

The best way to avoid Java serialization vulnerability is **never** to use Java serialization!

There are other mechanisms to store and transmit between objects and bytes sequences which avoid Java serialization vulnerability, such as *JSON* and *Protocol Buffers(Protobuf)*.

## JSON vs Protocol Buffers

I summarize the differences between them:

|                  |                                                           |
| ---------------- | --------------------------------------------------------- |
| JSON             | Protocol Buffers                                          |
| human-readable   | not human-readable, but it provide pbtxt for readability. |
| text-based       | binary                                                    |
| no schema needed | offer schemas to enforce appropriate usage                |
|                  | simple, faster, smaller in size                           |

But they are both good serialization mechanisms: 

- They are simpler than Java serialization. 
- They don't support auto serialization or deserialization
- They only support a few primitive and array data types to avoid deserialization issue.

## Reference

Effective Java, Third Edition

https://www.darkreading.com/informationweek-home/why-the-java-deserialization-bug-is-a-big-deal/d/d-id/1323237

