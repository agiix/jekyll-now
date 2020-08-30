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

#### Cuckoo Sandbox

Cuckoo is an open source automated malware analysis system that started as a Google Summer of Code project in 2010 within The Honeynet Project.
It’s used to automatically run and analyze files and collect comprehensive analysis results that outline what the malware does while running inside an isolated operating system. Cuckoo is currently undergoing a complete redesign, the new version will be released in the coming months. 

#### Project Goals

The goal of the project was to build a proof of concept module that uses an analytical approach to classify malware behavior into one or more categories. 
The analytical approach was chosen because we want to show which specific Indicator of Compromise (or in this case which TTP, but more about that later) caused the decision about categorization to be made. Additionally the project aimed to create new signatures for detecting malicious behavior, starting with Ransomware and then focusing on Trojan-like behavior, specifically for the Emotet family.

## So, what have you been doing all Summer?

#### Malware Categories

The start of the project involved a lot of research into different malware types, their specific behavior and key attributes that can be used to distinguish malware categories from each other and build signatures upon that information to support Cuckoo in detecting malicious behavior and classifying analyzed samples.
Together with my mentor we decided to start digging into Ransomware families first.
My key research findings can be found [here](https://github.com/agiix/analytical-malware-classification/blob/master/malware/ransomware/classification.md) and additionally a statistical summary of eacht analyzed ransomware sample can be found [here](https://github.com/agiix/analytical-malware-classification/tree/master/malware/ransomware/stats). 

The PoC tool I was working on relied on Cuckoo's analysis result outputted into the .pb file (more about that in the next section) and therefore some behaviors could not be implemented into signatures, since information about API-calls or the static analysis were missing.

After the research on Ransomware completed our focus shifted towards trojan like malware categories and we decided to take a look on the Emotet family. 
I summarized my research findings [here](https://github.com/agiix/analytical-malware-classification/tree/master/malware/trojan/emotet) and documented the analysis of all Emotet samples [here](https://github.com/agiix/analytical-malware-classification/tree/master/malware/trojan/stats).
Generating useful signatures became a lot more challenging, because of Emotet's stealthiness. 

#### TTPS

Most of the signatures were assigned to one or more TTP numbers, where as TTP stands for Tactics, Techniques and Procedures and describes patterns of activities or methods associated with a specific threat. Each TTP number matches a certain threat activity / behavior, which can be looked up at [MITRE](https://attack.mitre.org). Here a snippet of all TTPs MITRE assigned to the WannaCry malware:

![MITRE WannaCry]({{ site.baseurl }}/images/WannaCry.png "MITRE WannaCry")


#### Threemon

While researching upon malware categories and their behavior was a big part of the project, it also involved getting familiar with the new Cuckoo architecture. During the sandbox analysis, Cuckoo 3 uses a kernel based monitor called 'threemon', which for example, logs events like:

* Process creation, termination, injection etc.
* File creation, modification, deletion etc.
* Mutants
* Networkflows
* Registry modifications
* Predefined suspicious behavior

and output the logged events into a .pb file, which is based on [Google’s protocol buffer language](https://developers.google.com/protocol-buffers/docs/overview), which makes it possible to stream events directly from file.

#### The PoC Tool

The PoC tool takes the .pb file as input, reads every event, tries to match the events to malicious behavior and classifies the analyzed sample to one or more categories. While the first version during the project required the user to convert the pb file to a json formatted output for further analysis, the second version is able to parse the events directly from the pb file. 

![Terminal output]({{ site.baseurl }}/images/terminal.png "Terminal output")

Additionally, support for creating a detailed pdf report was added, that included every matched signature and the events it was triggered by. 
The signatures where partially self developed during the research phase and partially taken from Cuckoo's signature database and converted to be able to work with the PoC's structure.

