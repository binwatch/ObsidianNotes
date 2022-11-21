# Introduction to Character Animation

## What is Character Animation

-   动画电影（计算机角色动画的一个重要应用领域 motivation，但目前占比很小）
-   游戏（交互，直接推动计算机角色动画的发展）
-   虚拟角色、虚拟主播、数字人
-   VR
-   大场景人群仿真（粗粒度动作）

### 3D Computer Graphics

-   Geometry
-   **Animation**
-   Rendering

建模与渲染是静态的，考虑对一个固定的帧如何展现外观
**动画**解决的问题：下一帧如何生成——建模**时序**上的规律

### (Character) Animation

Animation 可以分为两个方向:
-   **Simulation** 一些客观的物理现象 (Phenomenon) rigid bodies, deformable solids, ropes, thin shells, cloths, fluid, smoke, sound... 数学建模 可描述
-   **Character Animation** 行为上的建模 (Behavior) humans, animals, virual creatures, hands, robots, crowds 大量观察，统计上的建模

角色动画与仿真之间：
-   联系：**Simulation +** _**Control**_ **= Character Animation** 控制带来的是主观意愿的一些内容
-   最大不同点：角色动画关注的对象是**角色**

## Why Do We Study Character Animation

将劳动密集型工作转化为计算密集型！
- A character typically has 20+ joints, or 50-100+ parameters
    - It is not super high-dimensional, so most animation can be created manually, by posing the character at keyframes
    - 劳动密集型 Labor-intensive, 不适合交互式应用 not for interactive applications
- Character animation techniques
    - 理解做出这些动作时发生了什么 Understanding the mechanism behind motions and behaviors
    - Smart editing / Reuse animation / Generate new animation
    - 计算密集型 "Compute-intensive"

![](https://raw.githubusercontent.com/binwatch/images/main/games105-01-0.png)

## Character Animation Pipeline

如何做角色动画？
1. 首先我们要有一个虚拟形象（几何建模的内容）
2. Rigging & Skinning 得到模型后第一件事——将模型**绑定到骨骼**上；涉及两个问题：
	- 怎么**设置骨骼**
	- 怎么计算模型（蒙皮）和骨骼的**相对关系**
3. Skeletal animation **骨骼本身应该怎么运动**（绑定后骨骼带动蒙皮运动）

![](https://raw.githubusercontent.com/binwatch/images/main/games105-01-1.png)

## Where does a Motion Come From?

神经信号 -> 肌肉信号 -> 在肌肉骨骼系统上增加力和力矩 -> 物理机制 -> 姿态

![](https://raw.githubusercontent.com/binwatch/images/main/games105-01-2.png)

**Physics** 部分无法人工干预

## Character Animation Methods

很多计算机角色动画技术可以根据**是否使用物理**来区分

- （工业界很多应用）基于运动学/关键帧的方法——隐含掉物理层及其以上层，直接更新姿态、速度、加速度
	- ![](https://raw.githubusercontent.com/binwatch/images/main/games105-01-3.png)

- （希望能完整复现角色动作的物理过程）基于动力学/物理的方法——完全复现比较困难，大部分情况下会简化；但仍然是通过物理仿真建模生产动作，不能进行干预
	- ![](https://raw.githubusercontent.com/binwatch/images/main/games105-01-4.png)

- low-level control 对每一帧每一个姿态精确控制，但非常低效昂贵
- high-level control 给一个目标，很少的信息实现控制，精确度与信息熵有关

![](https://raw.githubusercontent.com/binwatch/images/main/games105-01-5.png)

### Keyframe/Kinematics Animation

在图形学之前 Disney's 12 Principles of Animation

#### Methods

- **Forward Kinematics**
    - Given rotations of every joints
    - Compute position of end-effectors
- **Inverse Kinematics**
    - Given position of end-effectors
    - Compute rotations of every joints
- **Interpolation**
- **Motion Capture**
- **Motion Retargeting**
    - Given motions of a source character
    - Compute mostions for target characters with
        - different skeletion sizes
        - different number of bones
        - different topologies 比如关节朝向相反
        - ......

捕捉到动作后，还是无法生成动作；游戏中如何使用这些数据？——

- **Motion Grphs / State Machines**
	- 能够重用已有的运动数据，在交互下生产新的运动，完成一些下游任务
	- 但是构造非常复杂

动作图是在一些完整的动作片段之间进行切换；而把动作再切片，切片到每一帧进行控制——

- **Motion Matching**
    - 不是去完整地播放一段动作，而是在更加细粒度（比如每一帧结束的时候）通过最近邻搜索找到一个新的姿态——同时：
        - 满足控制目标
        - 与当前角色状态相差不能太大（保证动作连续性）
    - 很大程度上是一个工程上的实现（距离函数？如何设计动作库？）
    - 简单实用
    - 对游戏来说，阻力是如何集成到原来的 pipeline

希望对角色本身的内在规律进行建模，减少手动生成动作库的工作量——

- **Learning-based Approaches**
	- ![](https://raw.githubusercontent.com/binwatch/images/main/games105-01-6.png)
	- **Motion Generative Models**

希望通过一些高级内容比如语言生成动作——

- **Cross-Modal Motion Synthesis**
    - Audio-driven animation
        - Music to dance
        - Co-speech gesture
        - ......
    - Natural language to animation
        - Descriptions to actions
        - Scripts to performance
        - ![](https://raw.githubusercontent.com/binwatch/images/main/games105-01-7.png)
        - ......

如何建立这种模型？角色动画学到动作的统计学表示，而语言也是一个统计学模型，尝试在两者之间建立模型

总结：

![](https://raw.githubusercontent.com/binwatch/images/main/games105-01-8.png)

未来的方向？
- 给一段剧本，生成一段表演（贴近动画电影）
- ......
- （个人想法）能否考虑不同角色的习惯和语气？

#### Problems

- Physics plausibleness 物理可行（注意准确与可行的区别）  
    - 穿模
    - ......
- Interaction with the environment
- 许多很难采集的动作
    - 基于运动学的方法难以生成没有采集到的动作

### Physics-based/Dynamics Character Animation

由于运动学方法的缺陷，我们需要**回到基于物理仿真的方法**

#### Why

- 物理仿真的方法非常适合 **AR**、 **VR** 这样的游戏，VR 环境的动作和通常的游戏手柄不同，用户可以自由控制动作，因此只有通过物理仿真才能完全还原用户的动作：
	- 如何通过三个点的数据生成下半身的动作？
		- Motion Reconstrcution with Sparse
		- 也可以结合一些新的机器学习方法让动作看起来更加真实
		- 物理仿真让用更少数据信息复现成为可能
- 可以实现一些**非常细节**的动作，比如使用筷子

#### How

![](https://raw.githubusercontent.com/binwatch/images/main/games105-01-9.png)

比运动学方法多了一步——运动模型不直接生成下一个状态，而是**生成**一些**控制量**比如力、力矩，再通过物理仿真去真正改变角色姿态

一种简化的基本模型暂时将 Motion Control 忽略——**Ragdoll Simulation**；把控制关掉，只能生成一些无意识的、短时间内身体无法控制的动作；因此为了实现更有意义的动作，还是需要学习 Motion Control

最基本的问题之一：**如何建模角色？**
- 可以完整地建模神经系统与肌肉，但：
	- 其作用机理学术上并没有完全弄清楚
	- 自由度很高（比如肌肉），导致仿真的效率很低
- 简化：使用**关节力矩**驱动角色动作（肌肉通过肌腱连接到骨骼上，每一块肌肉运动时对骨骼产生一个力，每一个力等价地在关节上产生一个力矩）得到等价的仿真结果
	- ![](https://raw.githubusercontent.com/binwatch/images/main/games105-01-10.png)
	- 一类常用的方法：**Proportional-Derivative (PD) Control**
		- Tracking Controllers 给出目标高度的轨迹，然后使用 PD 控制生成对每个关节的力矩
		- ![](https://raw.githubusercontent.com/binwatch/images/main/games105-01-11.png)
		- Trajectory Crafting: NaturalMotion 公司开发的 Endorphin 用于（为了实现一个动作需要）**设计控制轨迹**（然后再使用 PD 控制）

#### Problems and Solutions

- 但过了将近 30 年仍然没有在游戏中见到这个技术，最大的原因是 **Control is Hard**
- 基于关键帧的方法设计成什么姿势就是什么姿势，但基于物理仿真的方法无法直接设置姿势，设置后只能希望得到差不多的动作，动画师制作困难
- "Keyframe Control"
- Spacetime/Trajectory Optimization
    - 需要解决的是高维非线性的问题，优化非常困难，速度很慢
- Abstract Models
    - 来自于机器人 简化模型
    - Linear feedback control + FSM 非常简化 缺少细节 无法做复杂的动作
- Reinforcement Learning
	- ![](https://raw.githubusercontent.com/binwatch/images/main/games105-01-12.png)
	- Deep Reinforcement Learning
	- DRL-based Tracking Controllers
	- Multi-skill Characters
- Generative Control Policies 和基于运动学的**生成模型**类似，但**生成**的是**控制**而非运动
- ......

总结：

![](https://raw.githubusercontent.com/binwatch/images/main/games105-01-13.png)


## More

多刚体仿真




