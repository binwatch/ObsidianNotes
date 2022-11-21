## Initial Value Problems

**Differential equations** describe the relation between an unknown function and its derivatives. To solve a differential equation is to find a function that satisfies the relation, typically while satisfying some additional conditions as well

In a canonical **initial value problem**, the *behavior of the system* is described by an **ordinary differential equation (ODE)** of the form:

$$
\dot{\boldsymbol{x}} = f(\boldsymbol{x}, t)
$$

where:

- $f$ is a known function
- $\boldsymbol{x}$  is the state of the system
- $\dot{\boldsymbol{x}}$ is $\boldsymbol{x}$ 's time derivative

In an initial value problem:

- **Given** $\boldsymbol{x}(t_{0}) = \boldsymbol{x}_{0}$ at some starting time $t_{0}$
- **Want** to follow $\boldsymbol{x}$ over time thereafter

2D visualization:

![](https://raw.githubusercontent.com/binwatch/images/main/pbm-ode-0.png)

The trajectory swept out by $\boldsymbol{p}$ through $f$  forms an **integral curve** of the vector field.

$f$  may or may not **depend directly** on time $t$. If it does, then not only the point $\boldsymbol{p}$ but the the **vector field itself moves**. So that point's velocity depends not only on **where it is**, but on **when it arrives there**.

## Numerical Solutions

Numerical solutions take **discrete time steps** starting with the initial value $\boldsymbol{x}(t_{0})$.

To take a step:

1. Use the derivative function $f$ to calculate an **approximate change** in $\boldsymbol{x}$: $\Delta{\boldsymbol{x}}$. over a time interval $\Delta{t}$.
2. Increment $\boldsymbol{x}$ by $\Delta{\boldsymbol{x}}$ to obtain the **new value**

In calculating a numerical solution, the derivative function $f$ is regarded as a black box: we provide numerical values for $\boldsymbol{x}$ and $t$, receiving in return a numerical value for $\dot{\boldsymbol{x}}$

**Numerical methods** operate by *performing one or more of these **derivative evaluations** at each time step*.

### Euler’s Method

> The simplest numerical method

#### Method

let $h$ be the **stepsize** parameter, Euler’s method simply computes the next state $\boldsymbol{x}(t +h)$ by taking a step in the derivative direction:

$$
\boldsymbol{x}(t +h) = \boldsymbol{x}(t) + h \dot{\boldsymbol{x}}(t)
$$

Instead of the true integral curve, the approximate solution follows a polygonal path, obtained by evaluating the derivative at the beginning of each leg:

![](https://raw.githubusercontent.com/binwatch/images/main/pbm-ode-1.png)


#### Problems

**\[1\]** 💣 **NOT Accurate**

Consider the case of a 2D function $f$ whose integral curves are concentric circles. A point $\boldsymbol{p}$ governed by $f$ supposed to orbit forever on whichever circle it started on. Instead, with each Euler step, $\boldsymbol{p}$ will move on a straight line to a circle of larger radius, so that its path will follow an **outward spiral**. *Shrinking the stepsize will slow the rate of this outward drift, but NEVER ELIMINATE it*.

**\[2\]** 💣 **Unstable**

Consider a 1D function $f = -kx$ (is satisfied by $x = e^{-kt}$), which should make the point $\boldsymbol{p}$ decay exponentially to zero. For sufficiently small step sizes we get reasonable behavior, but when:

- $h > 1/k$, we have $\lvert \Delta{x} \rvert > \lvert x \rvert$, so the solution *oscillates about zero*
- Beyond $h = 2/k$, the *oscillation diverges*, and the system *blows up*

> 这里应该是假定了 $k > 0$？

![](https://raw.githubusercontent.com/binwatch/images/main/pbm-ode-2.png)


**\[3\]** 💣 **NOT efficient**

Most numerical solution methods spend nearly all their time **performing derivative evaluations**, so the computational cost per step is determined by the number of evaluations per step.

Though Euler’s method **only requires one evaluation per step**, the **real efficiency** of a method **depends on the size of the steps it lets you take**—while **preserving accuracy and stability**—as well as on the cost per step.

More sophisticated methods, even some requiring as many as four or five evaluations per step, can greatly outperform Euler’s method because their ***higher cost per step is more than offset by the larger stepsizes they allow***.

```ad-note
数值方法需要平衡结果（精确性、稳定性）与效率（计算开销）

复杂的数值方法主要计算开销在每一步的 derivative evaluation 上，但是由于它们的复杂性，在同样的精确性、稳定性要求下允许更大的步长，因此需要更少的步数

计算开销（总价）综合了每一步的开销（单位价格）与步数（数量）
``` 

#### Improvement

To understand how we go about improving on Euler’s method, we need to look more closely at the **error** that the method produces. The key to understanding what’s going on is the **Taylor series**: Assuming $\boldsymbol{x}(t)$ is *smooth*, we can express its *value* at the *end* of the step as an **infinite sum** involving the the *value* and *derivatives* at the *beginning*:

$$
\boldsymbol{x}(t + h) = \boldsymbol{x}(t) + h \dot{\boldsymbol{x}}(t) + \frac{h^2}{2!}\ddot{\boldsymbol{x}}(t) + \cdots + \frac{h^n}{n!}\frac{\partial^{n} \boldsymbol{x}(t)}{\partial t^{n}}
$$

We get the Euler update formula by **truncating** the series, which means that *Euler’s method would be correct **only if** all derivatives beyond the first were zero*, i.e. if $\boldsymbol{x}(t)$ were **linear**.

The **error term**, the *difference between the Euler step and the full, untruncated Taylor series*, is **dominated by the leading term** $\frac{h^2}{2!}\ddot{\boldsymbol{x}}(t)$. Consequently, we can describe the error as $O(h^2)$ (read “Order h squared”).

Suppose that we chop our stepsize in half. Although this produces only about one fourth the error we got with a stepsize of $h$, we have to take twice as many steps over any given interval. That means that the **error we accumulate** over an interval $[t_0,  t_1]$ depends **linearly** upon $h$. Theoretically, using Euler’s method we can numerically compute with as little error as we want, by choosing a suitably small stepsize. In practice, a great many timesteps might be required, depending on the error and the function $f$.

### The Midpoint Method

If we were able to evaluate $\ddot{\boldsymbol{x}}$ as well ass $\dot{\boldsymbol{x}}$, we could acheive $O(h^3)$ accuracy instead of $O(h^2)$:

$$
\boldsymbol{x}(t + \Delta t) = \boldsymbol{x}(t) +  h \dot{\boldsymbol{x}}(t) + \frac{h^2}{2!}\ddot{\boldsymbol{x}}(t) + O(h^{3})
$$

Recall that the time derivative $\dot{\boldsymbol{x}}$ is given by a function $f(\boldsymbol{x}(t), t)$. For simplicity in what follows, we will **assume** that the derivative function $f$ does **depends on time only indirectly through** $\boldsymbol{x}$, so that $\dot{\boldsymbol{x}} = f(\boldsymbol{x}(t))$. The**chain rule** then gives:

$$
\ddot{\boldsymbol{x}} = \frac{\partial{f}}{\partial{\boldsymbol{x}}}\dot{\boldsymbol{x}} = f^{\prime}f
$$

```ad-note
title: 详细推导

$$
\ddot{\boldsymbol{x}} = \frac{\mathrm{d}\dot{\boldsymbol{x}}}{\mathrm{d}t}  = \frac{\mathrm{d}f}{\mathrm{d}t} =  \frac{\partial{f}}{\partial{\boldsymbol{x}}}\frac{\mathrm{d}\boldsymbol{x}}{\mathrm{d}t} = \frac{\partial{f}}{\partial{\boldsymbol{x}}}\dot{\boldsymbol{x}} = f^{\prime}f
$$

基于假设 $f$ 不直接依赖于 $t$，仅直接依赖于 $\boldsymbol{x}$
```


$f^{\prime}$ would often be *complicated* and *expensive*, we want to **avoid evaluating** it.

An approach is ***approximate** the second-order term just in terms of* $f$, and *substitute* the approximation into the truncated Taylor series of $\boldsymbol{x}$, leaving us with $O(h^3)$ error. 

To do this, we perform **another Taylor expansion** of $f$:

$$
f(\boldsymbol{x} + \Delta \boldsymbol{x}) = f(\boldsymbol{x}) +  \Delta \boldsymbol{x} f^{\prime}(\boldsymbol{x}) + O(\Delta \boldsymbol{x}^{2})
$$

We first introduce $\ddot{\boldsymbol{x}}$ into this expression by choosing non-fixed step $\Delta{\boldsymbol{x}}$ at time $t$, that is, varying with $\boldsymbol{x}(t)$:

$$
\Delta \boldsymbol{x} =  \frac{h}{2}f(\boldsymbol{x})
$$

so that:

$$
f(\boldsymbol{x} + \frac{h}{2}f(\boldsymbol{x})) = f(\boldsymbol{x}) +  \frac{h}{2}f(\boldsymbol{x}) f^{\prime}(\boldsymbol{x}) + O(h^{2}) =  f(\boldsymbol{x}) +  \frac{h}{2}\ddot{\boldsymbol{x}}(t) + O(h^{2})
$$

We can now multiply both sides by $h$ (turning the $O(h^2)$ term into $O(h^3)$) and rearrange, yielding:

$$
\frac{h^{2}}{2} \ddot{\boldsymbol{x}}(t) + O(h^{3}) = h\{f[\boldsymbol{x}(t) + \frac{h}{2}f[\boldsymbol{x}(t)] - f[\boldsymbol{x}(t)]\}
$$

Substituting the right hand side into the truncated Taylor series of $\boldsymbol{x}$ gives the update formula (notice that $f[\boldsymbol{x}(t)] = \dot{\boldsymbol{x}}(t)$):

$$
\boldsymbol{x}(t + \Delta t) = \boldsymbol{x}(t) +  h\{f[\boldsymbol{x}(t) + \frac{h}{2}f[\boldsymbol{x}(t)]\}
$$

```ad-question
title: **\[Doubt\]** 🤔 可以直接把 $O(h^{3})$ 替代掉？或者这里只是给出近似公式？
```

This formula:

1. evaluates an **Euler step**
2. performs a **second derivative evaluation** at the **midpoint** of the step
3. using the midpoint evaluation to update $\boldsymbol{x}$

Hence the name midpoint method. The midpoint method is **correct to within** $O(h^3)$, but requires **two evaluations** of $f$.

![](https://raw.githubusercontent.com/binwatch/images/main/pbm-ode-3.png)


> 图里应该是 $\Delta{t}f(\boldsymbol{x})$ 和 $f(\boldsymbol{x} + \Delta{\boldsymbol{x}}/2)$?

We don’t have to stop with an error of $O(h^3)$. By evaluating $f$ **a few more times**, we can **eliminate higher and higher orders of derivatives**. The most popular procedure for doing this is a method called **Runge-Kutta of order 4** and has an error per step of $O(h^5)$ (The Midpoint method could be called Runge-Kutta of order 2). We won’t derive the fourth order Runge-Kutta method, but the formula for computing $\boldsymbol{x}(t + h)$ is listed below:

$$
\begin{align*}
 k_{1} &= hf(\boldsymbol{x}, t) \\
 k_{2} &= hf(\boldsymbol{x} + k_{1}/2, t + h/2) \\  k_{3} &= hf(\boldsymbol{x} + k_{2}/2, t + h/2) \\  k_{4} &= hf(\boldsymbol{x} + k_{3}, t + h) \\ \boldsymbol{x}(t + h) &= \boldsymbol{x} + \frac{1}{6}k_{1} + \frac{1}{3}k_{2}  + \frac{1}{3}k_{3} + \frac{1}{6}k_{4}
\end{align*}
$$

## Adaptive Stepsizes

Whatever the underlying method, a **major problem** lies in **determing a good stepsize**.

Ideally, we want to choose $h$ **as large as possible**—but not so large as to give us an unreasonable amount of error, or worse still, to induce instability.

```ad-note
理想的步长尽可能大（快），但不会产生不可接受的错误与不稳定性（精确性、稳定性）
```

If we choose a **fixed stepsize**, we can only proceed as fast as the “worst” sections of $\boldsymbol{x}(t)$ will alllow.

What we would like to do is to **vary** $h$ **as we march forward in time**. This is the idea of **adaptive** **stepsizing**: varying $h$ over the course of solving the ODE.

Here we’ll be present adaptive stepsizing for Euler’s method. The basic idea is as follows. Lets assume we have a given stepsize $h$, and we want to know how much we can consider changing it.

Suppose we compute two estimates for $\boldsymbol{x}(t + h)$. We compute an estimate $\boldsymbol{x}_{a}$ by taking an Euler step of size $h$. We also compute an estimate $\boldsymbol{x}_{b}$ by taking two Euler steps of size $h/2$.

Both are differ from the true value of $\boldsymbol{x}(t + h)$ by $O(h^{2})$. That means that they *differ from each other by* $O(h^{2})$. As a result, we can write that a measure of the current error $e$ is:

$$
e = \lvert \boldsymbol{x}_{a} - \boldsymbol{x}_{b} \rvert
$$

This gives us a convenient **estimate to the error** in *taking an Euler step* of size $h$.

```ad-example
Suppose that we are willing to have an error of as much as $10^{−4}$ per step, and that the current error is only $10^{−8}$. Since the error goes up as $h^{2}$, we can increase the stepsize to:

$$
(\frac{10^{-4}}{10^{-8}})^{\frac{1}{2}}h = 100h
$$

Conversely, if we currently had an error of $10^{−3}$, and could only tolerate an error of $10^{−4}$, we would have to decrease the stepsize to:

$$
(\frac{10^{-4}}{10^{-3}})^{\frac{1}{2}}h = .316h
$$
```

```ad-question
title: **\[Doubt\]** 🤔 但是如何知道我们什么时候需要什么级别的误差？
```

## Implementation

The ODEs we will want to solve may represent many things—for instance, a collection of masses and springs, some rigid bodies, or a deformable object.

We want to implement **ODE solvers** and the models on which they operate in a way that **isolate**s each from the internal details of the other. This will make it possible to *change solvers easily*, and also make the solver code *reusable*.

Fortunately, this kind of **modularity** is not difficult to acheive, since all solvers can *be expressed in terms of a small, stereotyped set of operations*. Presumably, the system of ODE-governed objects will be embodied in a structure of some kind. The approach is to write type-specific code that operates on this structure to perform the standard operations, then to implement solvers in terms of these generic operations.

From the solver’s viewpoint, the *system* on which it operates is a **black-box function** $f(\boldsymbol{x}, t)$. The solver needs to be able to **evaluate** $f$, as required, at any values of $\boldsymbol{x}$ and $t$, and then to install the **updated**  $\boldsymbol{x}$ and $t$ when *a time step* is taken. To support these operations, the object that represents the ODE being solved must be able to handle these requests from the solver:

- `Return`  $dim(\boldsymbol{x})$: Since $\boldsymbol{x}$ and $\dot{\boldsymbol{x}}$ may be vectors, the solver must know their length, to allocate storage, perform vector arithmetic ops, etc.
- `Get`, `Set` $\boldsymbol{x}$ and $t$: The solver must be able to install new values at the end of a step. In addition, a multi-step method must set them to intermediate values in the course of performing derivative evaulations.
- `Evaluate` $f$ at the current $\boldsymbol{x}$ and $t$

>- In an object-oriented language, these operations would naturally be implemented as generic functions that are handled in a type-specific way.
>- In a non-object-oriented language generic functions would be faked by installing pointers to type-specific functions in structure slots, or simply by passing the function pointers as arguments to the solver.

Later on we will consider in detail how these operations are to be implemented for specific models such as particle-and-spring systems