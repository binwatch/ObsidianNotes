
## Why Memory Models

![](https://raw.githubusercontent.com/binwatch/images/main/concurrency2021-02-0.png)

## Sequential Consistency (SC) Model

\[Lamport 1979\]

```ad-quote
title: [Wikipedia: Sequential consistency](https://en.wikipedia.org/wiki/Sequential_consistency)

It is the property that "... the result of any execution is the same as if the operations of all the processors were executed in some sequential order, and the operations of each individual processor appear in this sequence in the order specified by its program."

That is, the execution order of a program in the same processor (or thread) is the same as the program order, while the execution order of a program on different processors (or threads) is undefined.

Conceptually, there is single global memory and a "switch" that connects an arbitrary processor to memory at any time step. Each processor issues memory operations in **program order** and the switch provides the global serialization among all memory operations

The sequential consistency is weaker than [strict consistency](https://en.wikipedia.org/wiki/Strict_consistency "Strict consistency"), which requires a read from a location to return the value of the last write to that location; strict consistency demands that operations be seen in the order in which they were actually issued.
```

```ad-question
title: **\[Doubt\]** 🤔 

SC model 允许并发程序的执行顺序不定，但不允许每个程序内部的指令顺序改变？因此禁止编译器的很多优化？
```


## Weak Memory Models

### Why We Need Weak Memory Models?

SC model **prohibits many optimization**:

![](https://raw.githubusercontent.com/binwatch/images/main/concurrency2021-02-1.png)

(`r1 = r2 = 0` ?) Impossible in SC model, but allowed in x86 or Java

**Weak memory model allow more behaviors**

```ad-question
title: **\[Doubt\]** 🤔 

我们需要 Weak Memory Model 是为了允许（主流？）编译器的很多常见优化（指令重排？）？

那么优化后的正确性由谁来保证？模型本身？程序员？
```

### Design Criteria

- Usability: DRF (Data-Race-Freedom) guarantee
	- DRF programs have the same behaviors as in SC model
- Not too strong
	- Allow common optimization techniques
	- In some sense hijacked by the mainstream compiler
- Preserve type-safety and security guarantee
	- Cannot be too weak

***Very challenging to satisfy all the requirements!***

- Data-race: read-write / write-write conflicts
	- It occurs when we have 2 <font color="0088ff">concurrent</font> <font color="ff6600">conflicting</font> operations
	- <font color="ff6600">Conflicting</font>: the 2 operations both access the *same memory location* and *at least one is a write*
	- <font color="0088ff">Concurrent</font>?
		- Differs across memory models
		- Java: the 2 operations are **not ordered by "happens-before"**

### Happens-Before Memory Model (HMM)

- Happens-Before Order: **transitive closure** of po \* sw
	-  ![](https://raw.githubusercontent.com/binwatch/images/main/concurrency2021-02-2.png)
- Happens-Before Memory Model (HMM)
	- Read can see:
		1. The most recent write that happens-before it
		2. A write that has no happens-before relation
	- Relaxed Ordering
	- Out-of-Thin-Air Read
	- See: 
		- [Preventing of Out of Thin Air values with a memory barrier in C++](https://stackoverflow.com/questions/51232730/preventing-of-out-of-thin-air-values-with-a-memory-barrier-in-c)
		- [Treatment of out-of-thin-air values](https://www.hboehm.info/c++mm/thin_air.html)
		- [What is out-of-thin-air safety?](https://stackoverflow.com/questions/42588079/what-is-out-of-thin-air-safety)

HMM does not have DRF-guarantee

### JMM

- Take HMM as the core, and try hard to distinguish good speculation from bad speculation!
- Introduce 9 axioms to constrain causality.
- Very complex, with surprising results and bugs.
	- Surprising Results:
		- Adding more synchronization may increase behaviors!
		- Inline threads may increase behaviors!
		- More:
			- Re-ordering independent operations may change behaviors
			- Adding / removing redundant reads may change behaviors

Reading:
- [The Java Memory Model](http://www.cs.umd.edu/~pugh/java/memoryModel/)
- http://openjdk.java.net/jeps/188

## See More

```ad-quote
title: [Weak vs. Strong Memory Models](https://preshing.com/20120930/weak-vs-strong-memory-models/)

A **memory model** tells you, for a given processor or toolchain, exactly what types of *memory reordering* to expect at runtime relative to a given *source code listing*. 

> **Memory reordering** at *compile time* (toolchain) and at *runtime* (processor), it is invisible to a single-threaded program, and only becomes apparent when lock-free techniques are used -- that is, when shared memory is manipulated without any mutual exclusion between threads. And the effects of processor reordering are only visible in multicore and multiprocessor systems.

![](https://raw.githubusercontent.com/binwatch/images/main/concurrency2021-02-3.png)

A **hardware** memory model tells you what kind of memory ordering to expect at runtime relative to an *assembly (or machine) code listing*.

![](https://raw.githubusercontent.com/binwatch/images/main/concurrency2021-02-4.png)

- Weak Memory Models:
	- In the weakest memory model, it’s possible to experience [all four types of memory reordering](http://preshing.com/20120710/memory-barriers-are-like-source-control-operations) (`#LoadLoad`, `#LoadStore`, `#StoreLoad`, `#StoreStore`).
	- Weak With Data Dependency Ordering:
		- Several modern CPU families:
			- ARM
			- PowerPC
			- Itanium
		- They have memory models which are, in various ways, almost as weak as the DEC Alpha’s, except for one common detail of particular interest to programmers: **they maintain [data dependency ordering](http://www.mjmwired.net/kernel/Documentation/memory-barriers.txt#305). It means that if you write `A->B` in C/C++, you are always guaranteed to load a value of `B` which is at least as new as the value of `A`. The Alpha doesn’t guarantee that**. The [Linux RCU mechanism](http://lwn.net/Articles/262464/) relies on data dependency ordering heavily.
- Strong Memory Models:
	- A **strong hardware memory model** is one in which every machine instruction comes implicitly with [acquire and release semantics](http://preshing.com/20120913/acquire-and-release-semantics). As a result, when one CPU core performs a sequence of writes, every other CPU core sees those values change in the same order that they were written.
	- Sequential Consistency:
		- **No memory reordering**. It’s as if the entire program execution is reduced to a sequential interleaving of instructions from each thread.
		- These days, you won’t easily find a modern multicore device which guarantees sequential consistency at the hardware level.
		- In any case, sequential consistency only really becomes interesting as a **software** memory model
```

- [Lock-Free Programming](https://www.cs.cmu.edu/~410-s05/lectures/L31_LockFree.pdf)

```ad-quote
title: [Wikipedia: Happened-before](https://en.wikipedia.org/wiki/Happened-before)

**Happened-before** [relation](https://en.wikipedia.org/wiki/Binary_relation "Binary relation") (denoted: $\rightarrow$) is a relation between the result of two events, such that if one event should happen before another event, the result must reflect that, even if those events are in reality executed out of order (usually to optimize program flow). This involves [ordering](https://en.wikipedia.org/wiki/Partially_ordered_set "Partially ordered set") events based on the potential [causal relationship](https://en.wikipedia.org/wiki/Causal_relationships "Causal relationships") of pairs of events in a concurrent system, especially asynchronous distributed systems.

Formal definition: the least [strict partial order](https://en.wikipedia.org/wiki/Strict_partial_order "Strict partial order") on events such that:

- If events $a$ and $b$ occur on the same process, $a \rightarrow b$ if the occurrence of event $a$ preceded the occurrence of event $b$.
- If event $a$ is the sending of a message and event $b$ is the reception of the message sent in event $a$, $a \rightarrow b$.

If 2 events happen in different isolated processes (that do not exchange messages directly or indirectly via third-party processes), then the 2 processes are said to be concurrent, that is neither $a \rightarrow b$ nor $b \rightarrow a$ is true.

Like all strict partial orders, the happened-before relation is _[transitive](https://en.wikipedia.org/wiki/Transitive_relation "Transitive relation")_, _[irreflexive](https://en.wikipedia.org/wiki/Irreflexive_relation "Irreflexive relation")_ and _[antisymmetric](https://en.wikipedia.org/wiki/Antisymmetric_relation "Antisymmetric relation")_.
```

```ad-quote
title: [The Happens-Before Relation](https://preshing.com/20130702/the-happens-before-relation/)

Let A and B represent operations performed by a multithreaded process. If A **happens-before** B, then the memory effects of A effectively become visible to the thread performing B before B is performed.

- If operations A and B are performed by the same thread, and A’s statement comes before B’s statement in program order, then A _happens-before_ B.
- But that’s not the only way to achieve a _happens-before_ relation. The C++11 standard states that, among other methods, you also can achieve it between operations in different threads using [acquire and release semantics](http://preshing.com/20120913/acquire-and-release-semantics).

**The _happens-before_ relation, under the definition given above, is not the same as A actually happening before B!** In particular,

1. A _happens-before_ B does not imply A happening before B.
2. A happening before B does not imply A _happens-before_ B.

_happens-before_ is a **formal relation** between operations, defined by a family of language specifications; **it exists independently of the concept of time**.
```

```ad-question
title: **\[Doubt\]** 🤔 

Happens-before 是一种关系，根据定义确定，而与实际程序运行时发生的 "happening-before"（实际执行顺序？） 无关？
```