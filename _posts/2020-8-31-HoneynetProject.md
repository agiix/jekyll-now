---
layout: post
title: Honeynet Project - GSoC 2020
---

**Student**: Armin Huremagic  
**Mentor**:  Ricardo van Zutphen  
**Organization**: The Honeynet Project  
**Project**: Analytical Malware Classification  
**Tag**: Information Security  

![Cuckoo Logo]({{ site.baseurl }}/images/cuckoo-invert.png "Cuckoo Logo")

## Introduction

### Cuckoo Sandbox

Cuckoo is an open source automated malware analysis system that started as a Google Summer of Code project in 2010 within The Honeynet Project.
It’s used to automatically run and analyze files and collect comprehensive analysis results that outline what the malware does while running inside an isolated operating system. Cuckoo is currently undergoing a complete redesign, the new version will be released in the coming months. 

### Project Goals

The goal of the project was to build a proof of concept module that uses an analytical approach to classify malware behavior into one or more categories. Additionally the project aimed to create new signatures for detecting malicious behavior, starting with Ransomware and then focusing on Trojan-like behavior, specifically for the Emotet family.
