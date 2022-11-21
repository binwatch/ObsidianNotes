# Generic Adaptive Scheduling for Efficient Context Inconsistency Detection

## MetaData

* Tags: #consistency-constraint, #context-inconsistency-detection, #scheduling-strategy, #susceptibility/cancellation-condition
* Date: [[2021-3-1]]
* Authors: [[Huiyan Wang]], [[Chang Xu]], [[Bingying Guo]], [[Xiaoxing Ma]], [[Jian Lu]]
* URL: [https://ieeexplore.ieee.org/document/8640044/](https://ieeexplore.ieee.org/document/8640044/)

## Overview

```ad-quote
title: Abstract
## Abstract

Many applications use contexts to understand their environments and make adaptation. However, contexts are often inaccurate or even conﬂicting with each other (a.k.a. context inconsistency). To prevent applications from behaving abnormally or even failing, one promising approach is to deploy constraint checking to detect context inconsistencies. A variety of constraint checking techniques have been proposed, based on different incremental or parallel mechanisms for the efﬁciency. They are commonly deployed with the strategy that schedules constraint checking immediately upon context changes. This assures no missed inconsistency, but also limits the detection efﬁciency. One may break the limit by grouping context changes for checking together, but this can cause severe inconsistency missing problem (up to 79.2%). In this article, we propose a novel strategy GEAS to isolate latent interferences among context changes and schedule constraint checking with adaptive group sizes. This makes GEAS not only improve the detection efﬁciency, but also assure no missed inconsistency with theoretical guarantee. We experimentally evaluated GEAS with large-volume real-world context data. The results show that GEAS achieved signiﬁcant efﬁciency gains for context inconsistency detection by 38.8–566.7% (or 1.4x–6.7x). When enhanced with an extended change-cancellation optimization, the gains were up to 2,755.9% (or 28.6x).

```

### Background

### Problem

### Method

## Zotero links

* [Local library](zotero://select/items/1_CG9F56P4)
* PDF Attachments
	- [Wang et al. - 2021 - Generic Adaptive Scheduling for Efficient Context .pdf](zotero://open-pdf/library/items/JFCSMJ8W)
* DOI: [10.1109/TSE.2019.2898976](https://doi.org/10.1109/TSE.2019.2898976)
* Cite key: wangGenericAdaptiveScheduling2021

## Notes











***

