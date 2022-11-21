## Scope and Field

### Scope

- Taichi-scope: Everything decorated by `@ti.kernel` or `@ti.func` is in the Taichi-scope
- Python-scope: Everything in a normal Python script is in the Python-scope

The `@ti.kernel` and `@ti.func` will be compiled JIT. 执行到时编译，而编译时 Python scope 的变量均被视为常量，在 Python scope 中更改 Python scope variable 后再次调用已经编译的 kernel 时不会在 Taichi scope 中作出相应改变——Taichi scope variables cannot see Python scope variables at run time, so **use Python variable as** **constants** **in the Taichi scope**

- Taichi scope 中的赋值是 pass by value
- Python scope 中的赋值是 pass by reference

### Field

那如果我们希望能在 Python scope 中修改 Taichi scope 中使用到的变量怎么办呢？

Taichi 提供了 field (域/场) 的概念，可以做到这一点，将在 [[Graphics/Taichi/太极图形课/Lecture01#Data|Data]] 的 [[Graphics/Taichi/太极图形课/Lecture01#Data#`ti.field`|ti.field]] 节详细介绍。

## Data

### Primitive types

- signed integers: ti.i8, ti.i16, ti.i32, ti.i64
- unsigned integers: ti.u8, ti.u16, ti.u32, ti.u64
- floating points: ti.f32, ti.f64

### Type promotions

Taichi picks the **more precise type** to store the algebraic results, for example:

- i32+f32=f32
- i32+i64=i64

### Type Casts

- Implicit casts: static types within the Taichi scope——Taichi scope 中的变量不能随意改变类型，与 Python scope 不同。改变变量类型会发生隐式转换（如果可以转换）
- Explicit casts: `variable = ti.cast(variable, type)`

### Compound types

- Using `ti.types` to create **compound types**
- `types` including: `vector` / `matrix` / `struct`

```Python
vec3f = ti.types.vector(3, ti.f32)
mat2f = ti.types.matrix(2, 2, ti.f32)
ray = ti.types.struct(ro=vec3f, rd=vec3f, l=ti.f32)
```

`ti.types` 用于创建一种数据“类型”

- Access compound elements using **`[i,j,k,…]`** **indexing**

### `ti.field`

^ba2b97

> “a **global N-d array** of **elements**”

```Python
heat_field = ti.field(dtype=ti.f32, shape=(256, 256))
```

- **global**: can be **read/written** from **both** the Taichi-scope and the Python-scope——“全局”变量，对 Taichi 和 Python 而言
- **N-d**: (Scalar: N=0), (Vector: N=1), (Matrix: N=2), (N = 3, 4, 5, …) ——field “本身”的大小
- **elements**: scalar, vector, matrix, struct——field 中的元素可以是各种类型（但需要相同）
- access elements in a field using `[i,j,k,…]` indexing
	- Special case, access a **zero-d** field using **`[None]`**

## Computation

### Kernels

function decorated by `@ti.kernel`

- **For loops** at the **outermost scope** in a Taichi kernel is *automatically parallelized*

```Python
import taichi as ti
ti.init(arch=ti.cpu)

@ti.kernel
def foo(k: ti.i32):
    for i in range(10): # Parallelized :-)
        if k > 42:
            ...
@ti.kernel
def bar(k: ti.i32):
    if k > 42:
        for i in range(10): # Serial :-(
            ...
```

Design your for loops for best performance

```Python
def my_for_loop():
    for i in range(10): # I don't want to parallelize this for
        for j in range(100): # I want to parallelize this for
            ...
my_for_loop()
```

to:

```Python
def my_for_loop():
    for i in range(10):
        my_taichi_for()
@ti.kernel
def my_taichi_for():
    for j in range(100):
        ...
my_for_loop()
```

- `break` is **NOT** supported in the parallel for-loops

因为并行

```Python
@ti.kernel
def foo():
    for i in range(10):
        ...
        break # Error!
@ti.kernel
def foo():
    for i in range(10):
        for j in range(10):
            ...
            break # OK!
```

- Race condition
	- Taichi uses `+=` as an **atomic** add
	- The compiler **optimizes for unnecessary** atomic operations

```Python
@ti.kernel
def sum():
    for i in range(10):
        # 1. OK
        total[None] += x[i]
        # 2. OK
        ti.atomic_add(total[None], x[i])
        # 3. data race
        total[None] = total[None] + x[i]
```

```ad-question
title: **\[Doubt\]** 🤔 How does compiler know what is unnecessary?
```

- **range-for**: loops over a **range**, identical to Python range-for

```Python
@ti.kernel
def foo():
    for i in range(N):
        x[i] = i
```

- **struct-for**: loops over a `ti.field`, only lives at the **outermost** scope

```Python
@ti.kernel
def foo():
    for i, j in x:
        x[i,j] = ti.Vector([i, j])
```

- Arguments:
	- At most 8 parameters
	- Pass from the Python scope to the Taichi scope
	- Must be **type**-hinted
	- **Scalar** Only
	- Pass by **value**
- Return value:
	- May or may not return
	- Returns one **single scalar** value only
	- Must be **type**-hinted

### Functions

function decorated by `@ti.func`

- Taichi functions can only be **called from the Taichi scope**
- Taichi functions can be **nested**
- Taichi functions are **force-inlined**

即：可以在 Taichi kernels 和 Taichi functions 中调用 Taichi functions，但是不能递归调用 Taichi functions；而 Python scope 中不能调用 Taichi functions

- Arguments and return values:
	- Do NOT need to be type-hinted
	- Arguments pass by value

### Taichi Scope

Anything in `@ti.kernel` and `@ti.func` are in the Taichi scope——和 Python scope 的区别：

- **Static data type** in the Taichi scope

```Python
@ti.kernel
def err_change_types_without_implicit_cast():
    a = 1 # a = 1, initialized as an int
    a = 2.7 # a = 2, because of the implicit cast
    a = ti.Vector([1.0, 0.0])
```

调用这个 kernel 时会报错：

```
err_change_types_without_implicit_cast:
a = ti.Vector([1.0, 0.0])
cannot augassign taichi class <class 'taichi.lang.matrix.Vector'> to scalar expr
``` 

即前面提到的 Taichi scope 中的变量类型是静态的，不能改变，如果可以则会进行隐式类型转换（而其实 Python scope 中的变量是“名字”，每一次改变其值时这个“名字”指向的地址都会改变）

- **Static lexical scope** in the Taichi scope

```Python
@ti.kernel
def err_out_of_scope(x: float):
    if x < 0:
        y = -x
    else:
        y = x
    print(y) # `y` is out of the scope
```

Taichi scope 的作用域和 C 类似，而 Python scope 中（从这个变量名字第一次出现的位置开始）在函数中出现的变量均属于此函数的作用域，在函数外出现的变量均属于全局作用域（除了一些关键字声明作用域的情况），可以参考 [SICP](https://cs61a.org/) 中对 Python 解释执行机制的讲解。

## Remark

- Taichi scope v.s. Python scope
	- Taichi scope is “on device”, “statically-typed”, “strongly-typed” 而 Python 是动态类型、弱类型
- `@ti.kernel`  
    - `__global__` functions in CUDA
    - The outermost scope for-loop in a `@ti.kernel` is parallelized 调用 `@ti.func` 时展开后的最外层 for 循环也属于此类
- `@ti.func`
    - `__device__` functions in CUDA
    - force-inlined

![](https://raw.githubusercontent.com/binwatch/images/main/taichi-S1-lecture01.png)


除了 `ti.field`，在 Python scope 定义的所有变量均是 Python scope variable，即使它是使用 Taichi data type 创建的

## Homework

### # Julia set and Fractal

[Julia集 复变函数的分形之美](https://zhuanlan.zhihu.com/p/359051218)

```Python
iterations = 0
while z.norm() < 20 and iterations < 50:
    z = complex_sqr(z) + c
    iterations += 1
pixels[i, j] = 1 - iterations * 0.02
```

这里的代码计算近似收敛的点（在迭代次数 $< 50$ 内都有 $\lvert z \rvert < 20$）对应像素为黑色（0）