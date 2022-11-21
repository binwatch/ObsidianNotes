# Minimizing Link Generation in Constraint Checking for Context Inconsistency Detection

## MetaData

* Tags: #constraint-checking, #context-inconsistency, #link-redundancy
* Authors: [[Chuyang Chen]], [[Huiyan Wang]], [[Lingyu Zhang]], [[Chang Xu]], [[Ping Yu]]

## Overview

```ad-quote
title: Abstract
## Abstract

Adaptive applications rely on conditions about their environments (or contexts) to deliver smart services, e.g., locationaware services. Due to inherent noises in environmental sensing and interpretation, there is an increasing demand for guarding the consistency of contexts to avoid application misbehavior, and at the same time minimizing the guarding cost. Existing work has tried to reduce the cost by speeding up the kernel constraint checking module inside the consistency guarding process. Most efforts have been spent on reusing previous checking results or checking constraints in parallel, while **leaving untouched one central problem of link generation**, the step that consumes a substantially large part of the total time cost for explaining why constraints have been violated. **In this paper, we propose a novel technique, MG, to automatically identify and remove redundant link generation, without harming any checking result.** We show that MG is **sound** (always checking correctly) and **complete** (removing all redundancy). Our experiments with synthesized and real-world consistency constraints reported that compared with existing work, MG achieved signiﬁcant efﬁciency improvements on the link generation (tens to hundreds times speedup), and could reduce the total constraint checking time up to 45.4%.

```

### Background

### Problem

### Method

## Zotero links

* [Local library](zotero://select/items/1_4SQA89E5)
* PDF Attachments
	- [Chen et al. - Minimizing Link Generation in Constraint Checking .pdf](zotero://open-pdf/library/items/G5NXTQYN)
* Cite key: chenMinimizingLinkGeneration

## Notes











***

