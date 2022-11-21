## Graphics pipeline

![](https://raw.githubusercontent.com/binwatch/images/main/gpu-arch-0.png)

### Rasterization

- Problem: Turing triangle data into pixels
- Steps:
	1. Geometry processing: *Project* triangles in screen space
	2. Rasterization: Find the pixel *covered* by triangle (or triangle walking)
	3. Pixel processing: Actually *assign* a color to the pixel

- "**Vertex shading**": Transforming and projecting 3d point vertex  into 2d vertex
	- These 2d vertices are then assembled into a **primitive** 
- "**Fragment shading**": Assign a color to each one of "fragments"
	- "**Fragments**": What pixels of the screen the primitive "touches"

### GPU to help

- A modern gpu can *accelerate* all of this
	- All of the previous operation map to several **specific HW block**
	- Some functionality are *programmable* and performed by the **shader cores** (i.e. fragment shader, vertex shader)
	- Other are *fixed* but *parameterizable* (i.e. primitive assembly, blending)

- "*Logical*" pipeline described in OGL/DX specification (abstraction)
- At *physical* level things are different (as long as specs are met there's no problem)

A sligntly close POV:

### Anatomy of a GPU

#### Why GPU?

- Extremely Parallel machine 高并行
	- Thousands of "threads" in flight
	- But:
		- limited flow control 控制流简单
		- Some threads shares program counter 共享程序计数器
		- No inter process communication 无进程间通信
	- Extremely good at doing lots of **independent** operations at the same time 同时进行大量独立操作
- Memory bandwidth is very high 内存：高带宽 高延迟
	- Hundreds of GB/s
	- But:
		- Very high latency
			- Thousand of cycles
		- Latency hiding mechanism necessary

> Graphics pipeline is organized to overcome those constraints

#### Before all that

CPU 向 GPU 发号施令

- CPU issues commands to the GPU
	- e.g.
		- Draw using thoses vertices and indices
		- Set this viewport
		- Changing states
		- May contain constant for shaders
		- Blend everything over using this blending function
- Commands are not executed immediately
	- Typically the CPU prepare command for the next frame while the GPU is rendering the current one
		- **Double buffering**
	- The commands written into a command buffer
	- The GPU parse them

#### Command Buffer Parser

![](https://raw.githubusercontent.com/binwatch/images/main/gpu-arch-1.png)

- Parse the command buffer
- Send commands down the graphics pipeline
- Synchronization point between CPU and GPU
	- Surface synch
		- e.g. wait that all the draw command on that rt finished before binding it as texture
- CPU bound applications: command buffer is not filled fast enough and GPU is idle

#### Geometry Stage

- Lots of sub stages:
	- Input assembly
	- Vertex shading
	- Primitive assembly
- Plus optional stuff:
	- Domain shader
	- Tessellation shader
	- Geometry shader
	- Stream out
	- For simplicity we are skipping those

##### Input Assembly Unit

![](https://raw.githubusercontent.com/binwatch/images/main/gpu-arch-2.png)

- Fetches the indices / vertices from main memory
- Has a vertex **reuse cache**
	- *Triangles share vertices*, so it is likely to have *cache hit*
	- *Cache miss* means the vertex need to be sent to the shader system to transform
- When *enough cache misses* are accumulated a job is sent to the shader core
- Usually there are more than one input assembly unit in a GPU
	- *Work distribution is usually done at drawcall level*
		- e.g. assign 128 indices to a different input assembly unit

##### Vertex Shading

![](https://raw.githubusercontent.com/binwatch/images/main/gpu-arch-3.png)

- Done by **Shader Core**
- First stage that is entirely programmable
- Export position and vertex attribute to forward to pixel shading 导出后续着色需要的顶点位置与属性信息
- Positions are stored in a positional cache, used in primitive assembly / setup 位置信息（在 primitive 装配中使用）
- Attributes are stored in a attributes cache, they are needed only in pixel shaders 属性信息（在 pixel shaders 中使用）

##### Primitive Assembly Unit

> So far we have only point (vertex) transformed

![](https://raw.githubusercontent.com/binwatch/images/main/gpu-arch-4.png)

- Primitive assembly takes the position form the position cache
- Turn them into triangles (primitives) using the *connectivity information* we gave in the API (e.g. `Trilist`)
- At this point triangles need to be: 
	- Discarded if outside the view
	- **Clipped** if partially in view
		- Clipping produces more traingles, expensive so guard band used to minimize

![](https://raw.githubusercontent.com/binwatch/images/main/gpu-arch-5.png)

- Surviving primitive are then *perspective projected* (divided by `w`) and *viewport transformed*
- Backfacing and zero area culling happens here
- Vertices are "snapped" into pixel
- A CPU bounding box culling can avoid primitive assembly unit being overwhelmed