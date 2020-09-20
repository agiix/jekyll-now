---
layout: post
title: Honeynet Project - GSoC 2020
---

**Student**: Armin Huremagic  
**Mentor**:  Ricardo van Zutphen, Jurriaan Bremer  
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
The analytical approach was chosen because we want to show which specific Indicator of Compromise (or in this case which TTP, but more about that later) caused the decision about categorization to be made. Additionally, the project aimed to create new signatures for detecting malicious behavior, starting with Ransomware and then focusing on Trojan-like behavior, specifically for the Emotet family.

## So, what have I been doing all Summer?

#### Malware Categories

The start of the project involved a lot of research into different malware types, their specific behavior and key attributes that can be used to distinguish malware categories from each other and build signatures upon that information to support Cuckoo in detecting malicious behavior and classifying analyzed samples.
Together with my mentor we decided to start digging into Ransomware families first.
My key research findings can be found [here](https://github.com/agiix/analytical-malware-classification/blob/master/malware/ransomware/classification.md) and additionally a statistical summary of each analyzed ransomware sample can be found [here](https://github.com/agiix/analytical-malware-classification/tree/master/malware/ransomware/stats). 

The PoC tool I was working on relied on Cuckoo's analysis result outputted into the .pb file (more about that in the next section) and therefore some behaviors could not be implemented into signatures, since information about API-calls or the static analysis were missing.

After the research on Ransomware completed our focus shifted towards Trojan like malware categories, and we decided to take a look on the Emotet family. 
I summarized my research findings [here](https://github.com/agiix/analytical-malware-classification/tree/master/malware/trojan/emotet) and documented the analysis of all Emotet samples [here](https://github.com/agiix/analytical-malware-classification/tree/master/malware/trojan/emotet/stats).
Generating useful signatures became a lot more challenging, because of Emotet's stealthiness. 

The signatures were partially self developed during the research phase and partially taken from Cuckoo's signature database and converted to be able to work with the PoC's structure.

#### TTPS

Most of the signatures were assigned to one or more TTP numbers, whereas TTP stands for Tactics, Techniques and Procedures and describes patterns of activities or methods associated with a specific threat. Each TTP number matches a certain threat activity / behavior, which can be looked up at [MITRE](https://attack.mitre.org). Here a snippet of all TTPs MITRE assigned to the WannaCry malware:

![MITRE WannaCry]({{ site.baseurl }}/images/WannaCry.png "MITRE WannaCry")

#### Threemon

While researching upon malware categories and their behavior was a big part of the project, it also involved getting familiar with the new Cuckoo architecture. During the sandbox analysis, Cuckoo 3 uses a kernel based monitor called 'threemon', which for example, logs events like:

* Process creation, termination, injection etc.
* File creation, modification, deletion etc.
* Mutants
* Networkflows
* Registry modifications
* Predefined suspicious behavior

and output the logged events into a pb file, which is based on [Google’s protocol buffer language](https://developers.google.com/protocol-buffers/docs/overview), which makes it possible to stream events directly from file.

#### The PoC Tool

The PoC tool (cuckoopbanalyzer) takes the pb file as input, reads every event, tries to match the events to malicious behavior and classifies the analyzed sample to one or more categories. While the first version during the project required the user to convert the pb file to a json formatted output for further analysis, the second version is able to parse the events directly from the pb file. A Cuckoo 3 instance was setup during the project to generate the neccessary pb files for testing.
The tool takes one required argument as input which is a Cuckoo pb file. This file will be generated once the malware analysis of the Cuckoo sandbox finishes.
To test the analyzer take the pb file from the test folder, which was generated from analyzing a WannaCry sample.
```
cuckoopbanalyzer test/wannacry.pb
```

The following snippet shows the displayed terminal output after the PoC tool analyzed the pb file:

![Terminal output]({{ site.baseurl }}/images/terminal.png "Terminal output")


As shown above, it not only displays which malware category and which signature have been matched to the sample, but also a description of each category, signature and TTP number, to give the user a better understanding of the results. 

Additionally, support for creating a detailed pdf report and a json dump of all captured events was added, that included every matched signature and the events it was triggered by. Here a snippet of the same sample's json dump:

```json
"Categories": {
  "Ransomware": "Uses encryption to disable a target\u2019s access to its data until a ransom i\ns paid. The victim organization is rendered partially or totally unable \nto operate until it pays, but there is no guarantee that payment will re\nsult in the necessary decryption key or that the decryption key provided\n will function properly."
},
"TTPS": {
  "T1005" : "Sensitive data can be collected from local system sources, such as the file system or databases of information residing on the system prior to Exfiltration."
},
"Signatures": {
  "UsesWindowsUtilities": {
        "Process":{
           "Command":[
              "attrib +h .",
              "cmd.exe /c vssadmin delete shadows /all /quiet & wmic shadowcopy delete & bcdedit /set {default} bootstatuspolicy ignoreallfailures & bcdedit /set {default} recoveryenabled no & wbadmin delete catalog -quiet",
              "wmic  shadowcopy delete ",
              "cmd.exe /c reg add HKLM\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Run /v \"biiywaghgcpcp303\" /t REG_SZ /d \"\\\"C:\\Users\\Admin\\AppData\\Local\\Temp\\tasksche.exe\\\"\" /f",
              "reg  add HKLM\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Run /v \"biiywaghgcpcp303\" /t REG_SZ /d \"\\\"C:\\Users\\Admin\\AppData\\Local\\Temp\\tasksche.exe\\\"\" /f"
         ],
         "Image":[
            "C:\\Windows\\SysWOW64\\attrib.exe",
            "C:\\Windows\\SysWOW64\\cmd.exe",
            "C:\\Windows\\SysWOW64\\Wbem\\WMIC.exe",
            "C:\\Windows\\SysWOW64\\cmd.exe",
            "C:\\Windows\\SysWOW64\\reg.exe"
         ],
         "Status":[
            "Create",
            "Create",
            "Create",
            "Create",
            "Create"
         ],
      },
    },
  -truncated-
 }
```

## What needs to be done in the future

* The malware categorization is currently done by simply matching each TTP to a particular malware category, which is prone to false positives. Signature like [ADS](https://github.com/agiix/analytical-malware-classification/blob/master/sigs/persistence_ads.py), [ExecBitsAdmin](https://github.com/agiix/analytical-malware-classification/blob/master/sigs/exec_bitsadmin.py), [CreatesExe](https://github.com/agiix/analytical-malware-classification/blob/master/sigs/creates_exe.py) or [DeletesExecutedFiles](https://github.com/agiix/analytical-malware-classification/blob/master/sigs/deletes_executed.py) are too generic to attribute to a specific malware category, it is therefore important to create a more precise approach.
* Creating more and updating already existing signatures. Signatures like [BrowserStealer](https://github.com/agiix/analytical-malware-classification/blob/master/sigs/infostealer_browser_modifications.py) check if certain files, which are known to hold sensible information collected by the browser, where read from. Those file locations tend to change over time and therefore some research is needed to update the file paths accordingly.
* A scoring system is missing, that gives each analyzed sample a score, for example from 1 to 10, to indicate how malicious the captured events are. One possible approach might be to assign every signature with a severity score. 

## Conclusion

Currently the PoC takes a rather simple approach to categorize malware samples, by matching certain TTP's to one or more malware categories. This works fine for malware categories like Ransomware, where there are characteristic behaviors that are unique for this category, but categories within the Trojan families are harder to distinguish by only matching for TTP's.  
Some of the adapted signatures remain valid, but certain signatures need to be further updated, since they check for file locations within common applications that might have changed over time.  
Overall, the taken approach works primarily well for testing old and new signatures, but needs more research and development to improve the challenge of malware categorization.

## Attribution

I want to thank the Honeynet Project and the awesome team at Hatching for this unique and awesome opportunity. During the project I was able to get an impression of the effort that goes into developing a sandbox system and I am deeply impressed by the knowledge you guys posses. 

My biggest thanks goes to my mentor Ricardo, for the great time while working on the project. You are clearly an expert in what you are doing and I learned so much from you. I couldn't have asked for a better mentor, keep up the great work!

As for me personally, I had to chance to dive deeper into the field of malware analysis with people that share the same passion. I am grateful to have had the chance to participate in this years GSoC and learn a lot about the development of open source projects. I am looking forward to keep contributing to this great community and project. 
