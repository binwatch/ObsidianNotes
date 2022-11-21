![](https://raw.githubusercontent.com/binwatch/images/main/taichi-S1-lecture02.png)

## Metaprogramming

产生最终实际运行程序的程序/编程方式

Metaprogramming in Taichi:

- **Unify** the development of **dimensionality-dependent** code, such as 2D/3D physical simulations
- Improve **run-time performance** by taking run-time costs to compile time

### 维度无关代码

#### Template

`ti.template()`

- The Taichi kernels and functions with **`ti.template()`** **arguments** are **template functions**
- Template functions are **instantiated** when needed. (Depends on the arguments.)

Allows you to pass **"anything"** _supported by Taichi_

- **Pass-by-reference**, use with cautions
    - Computations in the Taichi scope can **NOT** modify Python scope data
    - Computations in the Taichi scope can modify Taichi fields
    - Computations in the Taichi scope can modify Taichi scope data 未使用模板时，由于是 pass-by-value 不会改变传入参数的值
- Taichi kernels are **instantiated whenever seeing a new parameter** (even same typed) 根据变量的地址决定，如果为一个变量重新赋值，或者使用其他相同类型的变量，由于地址不同，Taichi 均会重新实例化模板函数

#### Dimension independent programming

`ti.grouped()`

```Python
@ti.kernel
def copy(x: ti.template(), y: ti.template()):
    for I in ti.grouped(y):
        # I is a vector with dimensionality same to y
        # If y is 0D, then I = ti.Vector([]), which is equivalent to `None` used in x[I]
        # If y is 1D, then I = ti.Vector([i])
        # If y is 2D, then I = ti.Vector([i, j])
        # If y is 3D, then I = ti.Vector([i, j, k])
        # ...
        x[I] = y[I]
```

#### Metadata

- Field 
    - `field.dtype`: type of a field
    - `field.shape`: shape of a field
- Matrix / Vector
    - `matrix.n`: rows of a mat
    - `matrix.m`: cols of a mat / vec

### 运行时性能

#### static

`ti.static()`

- Compile-time branching

```Python
enable_projection = False
x = ti.field(ti.f32, shape=10)
@ti.kernel
def static():
    if ti.static(enable_projection): # No runtime overhead
        x[0] = 1
```

- Forced loop unrolling for performance

```Python
@ti.kernel
def foo():
    for i in ti.static(range(4)):
        print(i)

# is equivalent to:
@ti.kernel
def foo():
    print(0)
    print(1)
    print(2)
    print(3)
```

- Forced loop unrolling for element index access
	- Indices into compound Taichi types must be a compile-time constant

```Python
# Here we declare a field contains 8 vectors. Each vector contains 3 elements.
x = ti.Vector.field(3, ti.f32, shape=(8))
@ti.kernel
def reset():
    for i in x:
        for j in ti.static(range(x.n)):
            # The inner loop must be unrolled since j is an index for accessing a vector
            x[i][j] = 0
```

## Object-oriented programming

Python OOP + Taichi DOP = ODOP

Objective data-oriented programming (ODOP)

```Python
@ti.data_oriented
class TaichiWheel:
    def __init__(self, radius, width, rolling_fric):
        self.radius = radius
        self.width = width
        self.rolling_fric = rolling_fric
        self.pos = ti.Vector.field(3, ti.f32, shape=4)
    @ti.kernel
    def Roll(self):
        ...
    @ti.func
    def foo(self):
        ...
```

`@ti.data_oriented`

Compared with Python, the Taichi classes are more **data_oriented**:

Python class:

```Python
class PythonCelestialObject:
    def __init__(self):
        self.pos = ...
        self.vel = ...
        self.force = ...
```

Taichi class:

```Python
@ti.data_oriented
class CelestialObject:
    def __init__(self):
        self.pos = ti.Vector(2, ti.f32, shape = N)
        self.vel = ti.Vector(2, ti.f32, shape = N)
        self.force = ti.Vector(2, ti.f32, shape = N)
```

Taichi 更多使用面向数据的方法，因为使用数组计算效率更高

- PDOP: Use Python scope variables in Taichi scope with caution
- ODOP: Use Python scope **members** in Taichi scope with caution