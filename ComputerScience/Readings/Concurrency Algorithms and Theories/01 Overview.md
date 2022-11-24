## What is Concurrency

- Programmers' View: `Thread`, `join()`, output, ...
- More Abstract View:
	- [Parallel composition](https://www.sciencedirect.com/topics/computer-science/parallel-composition)
	- Shared memory & interleaving semantics

## Problems with Concurrency

**Nondeterministic !**

- Recall: interleaving semantics
- <font color="ff3333">Difficult to find a bug</font>
- <font color="ff3333">Difficult to reproduce a bug</font>

C++ example

```CPP
#include <iostream>
#include <thread>

void thread_function() {
    for (int i = -100; i < 0; i++)
        std::cout << "thread function: " << i << "\n";
}

int main() {
	std::thread t(thread_function);
	for (int i = 0; i < 100; i++)
	    std::cout << "main thread: " << i << "\n";
	t.join();
	return 0;
}
```

```ad-question
title: **\[Doubt\]** 🤔 

我一开始以为是 `i` 的问题，但 `i` 不是共享变量

这里的共享变量应该是 `std::cout`，导致了输出乱序的问题
```

Sloutions? Using **Lock**s!

More Abstract View:

- Synchronization operations: lock/unlock, acg/rel

But...

- How are `lock()` and `unlock()` implemented?
- Are there interleavings between the threads when executing the the implementations of `lock()`/`unlock()`?

Programmers' View:

- A concurrent program = concurrent objects + their clients
- ![](https://raw.githubusercontent.com/binwatch/images/main/concurrency2021-01-0.png)
## Memory Models

```ad-quote
title: [Wikipedia: Memory model](https://en.wikipedia.org/wiki/Memory_model_(programming))

In computing, a **memory model** describes the interactions of [threads](https://en.wikipedia.org/wiki/Thread_(computer_science) "Thread (computer science)") through [memory](https://en.wikipedia.org/wiki/Memory_(computing) "Memory (computing)") and their shared use of the [data](https://en.wikipedia.org/wiki/Data_(computing) "Data (computing)").
```

- Interleaving semantics
	- Sequential Consistency (SC) model

- (Weak/Relaxed) Memory Models
	- Java
	- C/C++ 11
	- x86-TSO
	- ARMv8
	- ...

Two C++ examples: **Will the assertion fail?**

```CPP
#include <thread>
#include <assert.h>

int x = 0;

void foo() {
    while (true) {
        x = 1;
        x = 0;
        assert(x == 0);
    }
}

int main() {
    std::thread t(foo);
    std::thread t2(foo);

    t.join();
    t2.join();
}
```

```CPP
#include <thread>
#include <assert.h>

int x = 0;

void foo() {
    for ( int i = 0; i < 10000000; i++ ) {
       x = x + 1; 
    }
}

int main() {
    std::thread t1(foo);
    std::thread t2(foo);
    t1.join();
    t2.join();
    assert(x == 20000000);
}
```

```ad-question
title: **\[Doubt\]** 🤔 

取决于指令的执行顺序？如果用编译优化可能改变语句/指令顺序？
```

## Course Overview

### Goals of the Course

- Understanding concurrent programs
	- Interleaving semantics
	- Some feelings of weak memory models
- Understanding concurrent objects (algorithms)
	- Correctness criteria
	- Design ideas
	- Why they are correct
	- Some impossibility results

### Preliminary Syllabus

- Weak memory models
	- Java, C11
- Locks IMplemented by Read/Write
	- Peterson Lock, Filter Lock, Lamport's Bakery Lock
- Linearizability
- Synchronization Operations
- Practical Locks
	- TAS Lock, Queue Locks
- List-Based Sets
- Concurrent Queues and Stacks
- Snapshot