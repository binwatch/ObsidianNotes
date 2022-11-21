# GEAS: Generic Adaptive Scheduling for High-Efficiency Context Inconsistency Detection

## MetaData

* Tags: #context-inconsistency, #scheduling-strategy, #suspicious-pair
* Date: [[9/2017]]
* Authors: [[Bingying Guo]], [[Huiyan Wang]], [[Chang Xu]], [[Jian Lu]]
* URL: [http://ieeexplore.ieee.org/document/8094416/](http://ieeexplore.ieee.org/document/8094416/)

## Overview

```ad-quote
title: Abstract
## Abstract

Context-aware applications adapt their behavior based on collected contexts. However, contexts can be inaccurate due to sensing noise, which might cause applications to misbehave. One promising approach is to check contexts against consistency constraints at runtime, so as to detect context inconsistencies for applications and resolve them in time. The checking is typically immediate upon each collected context change. Such a scheduling strategy is intuitive for avoiding missing context inconsistencies in the detection, but may cause low-efﬁciency problems for heavy-workload checking scenarios, even if equipped with existing incremental or parallel constraint checking techniques. One may choose to check contexts in a batch way to increase the efﬁciency by reducing the number of constraint checking. However, this can easily cause missed context inconsistencies, denying the purpose of inconsistency detection. In this paper, we propose a novel scheduling strategy GEAS of two nice properties: (1) adaptively tuning the batch window to avoid missing any context inconsistency; (2) generic to checking techniques with no or little adjustment. We experimentally evaluated GEAS against the immediate strategy with existing constraint checking techniques. The experimental results show that GEAS achieved 143–645% efﬁciency improvement without missing any context inconsistency, while alternatives caused 39.2–65.3% loss of detected context inconsistencies.

```

### Background

### Problem

### Method

## Zotero links

* [Local library](zotero://select/items/1_24KMJEYH)
* PDF Attachments
	- [Guo et al. - 2017 - GEAS Generic Adaptive Scheduling for High-Efficie.pdf](zotero://open-pdf/library/items/7K2I57DU)
* DOI: [10.1109/ICSME.2017.10](https://doi.org/10.1109/ICSME.2017.10)
* Cite key: guoGEASGenericAdaptive2017

## Notes











***

