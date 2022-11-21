# Dissector: input validation for deep learning applications by crossing-layer dissection

## MetaData

* Tags: #deep-learning, #fault-tolerance, #input-validation
* Date: [[2020-06-27]]
* Authors: [[Huiyan Wang]], [[Jingwei Xu]], [[Chang Xu]], [[Xiaoxing Ma]], [[Jian Lu]]
* URL: [https://dl.acm.org/doi/10.1145/3377811.3380379](https://dl.acm.org/doi/10.1145/3377811.3380379)

## Overview

```ad-quote
title: Abstract
## Abstract

Deep learning (DL) applications are becoming increasingly popular. Their reliabilities largely depend on the performance of DL models integrated in these applications as a central classifying module. Traditional techniques need to retrain the models or rebuild and redeploy the applications for coping with unexpected conditions beyond the models’ handling capabilities. In this paper, we take a fault tolerance approach, Dissector, to distinguishing those inputs that represent unexpected conditions (beyond-inputs) from normal inputs that are still within the models’ handling capabilities (within-inputs), thus keeping the applications still function with expected reliabilities. The key insight of Dissector is that a DL model should interpret a within-input with increasing confidence, while a beyond-input would probably cause confused guesses in the prediction process. Dissector works in an application-specific way, adaptive to DL models used in applications, and extremely efficiently, scalable to large-size datasets from complex scenarios. The experimental evaluation shows that Dissector outperformed state-of-the-art techniques in the effectiveness (AUC: avg. 0.8935 and up to 0.9894) and efficiency (runtime overhead: only 3.3–5.8 milliseconds). Besides, it also exhibited encouraging usefulness in

```

### Background

### Problem

### Method

## Zotero links

* [Local library](zotero://select/items/1_537C4M7K)
* PDF Attachments
	- [Wang et al. - 2020 - Dissector input validation for deep learning appl.pdf](zotero://open-pdf/library/items/EWPJBYEH)
* DOI: [10.1145/3377811.3380379](https://doi.org/10.1145/3377811.3380379)
* Cite key: wangDissectorInputValidation2020

## Notes











***

