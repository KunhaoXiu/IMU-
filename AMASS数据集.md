# AMASS: Archive of Motion Capture as Surface Shapes

概括：将大量基于标记的光学人体动作捕捉数据集统一起来，使它们能够在一个共同的框架和参数化体系中被处理。



SMPL骨架

```markdown
                 15 head
                    |
                 12 neck
                    |
13 l_collar -    9 spine3  -     14 r_collar
      |             |                |
16 l_sho         6 spine2        17 r_sho
      |             |                |
18 l_elb         3 spine1        19 r_elb
      |             |                |
20 l_wri         0 pelvis        21 r_wri
      |          /       \           |
22 l_hand    1 l_hip   2 r_hip   23 r_hand
               |          |
            4 l_knee   5 r_knee
               |          |
            7 l_ank    8 r_ank
               |          |
           10 l_foot  11 r_foot
```
```markdown
0  pelvis
├─ 1  left_hip
│  └─ 4  left_knee
│     └─ 7  left_ankle
│        └─ 10 left_foot
├─ 2  right_hip
│  └─ 5  right_knee
│     └─ 8  right_ankle
│        └─ 11 right_foot
└─ 3  spine1
   └─ 6  spine2
      └─ 9  spine3
         ├─ 12 neck
         │  └─ 15 head
         ├─ 13 left_collar
         │  └─ 16 left_shoulder
         │     └─ 18 left_elbow
         │        └─ 20 left_wrist
         │           └─ 22 left_hand
         └─ 14 right_collar
            └─ 17 right_shoulder
               └─ 19 right_elbow
                  └─ 21 right_wrist
                     └─ 23 right_hand
```



SMPL坐标系：（y up ,右手系。 ）

原始 Xsens 世界系坐标系和smpl坐标系，都是“全局参考系”下的表示，不是局部关节坐标系；在这个项目（dynaip）里，原始 Xsens 全局系和 SMPL 全局系的区别主要就是坐标轴定义不同。



这个仓库里最终用于训练和处理的统一全局坐标约定，基本上是 SMPL 风格的 y-up。
Xsens 来源的数据会显式转换到这套坐标系。
DIP 这类数据不是“先转换”，而是本来就在 SMPL 这套表示里处理。

对于Xsens系统导出的MVNX文件，其**全局坐标系定义非常明确**，是一个**东北天（ENU）**系统：

- **X轴（红色）**：指向**当地磁北**（正方向为北）。
- **Y轴（绿色）**：根据**右手定则确定**，指向**西**。
- **Z轴（蓝色）**：垂直向上。



### 全局朝向和局部朝向的区别

这就是核心区别：

全局朝向看的是“它在世界里朝哪边”
局部朝向看的是“它相对父节点又多转了多少”

所以整个人一起转身时：

骨盆全局朝向变了
胸腔全局朝向变了
肩膀全局朝向变了
上臂全局朝向也变了

但局部朝向上：

胸腔相对骨盆可能几乎不变
肩膀相对胸腔可能几乎不变
上臂相对肩膀也可能几乎不变

再看“转身之后，右手臂向前抬起”。

这时候右上臂不只是跟着身体转，还相对肩膀自己发生了额外转动。于是：

右上臂的全局朝向变了
右上臂的局部朝向也变了

因为它相对全局当然朝向变了；同时它相对父节点也确实多抬了一下。

所以你可以把这两个量理解成：

全局朝向 = 身体这一段在世界里的绝对朝向
局部朝向 = 身体这一段相对上一段的关节动作

如果用一句特别形象的话：

整个人转身，像是把整棵骨架一起端着转了一下
这会改变很多关节的全局朝向，但不一定改变它们的局部朝向

手臂抬起，像是树枝相对树干自己弯了一下
这既会改变这个树枝的全局朝向，也会改变它的局部朝向





segment指的是什么,



在这个项目和 Xsens 这类动作捕捉语境里，segment 中文更接近：

骨段或者身体段，最直白地理解，就是“人体里两关节之间的一段刚体部分”。

比如：

Pelvis：骨盆段
LeftUpperLeg：左大腿段
LeftLowerLeg：左小腿段
T8：胸椎对应的一段躯干段
LeftUpperArm：左上臂段
LeftForeArm：左前臂段

所以 segment 不是“关节点”那个点，而是连接这些点之间的一段身体部分。
关节更像连接处，segment 更像连接处之间的那根“杆”。

你可以这样对应着理解：

joint：
关节、关节点、连接点

segment：
骨段、肢段、身体的一段

为什么这里更适合叫“骨段”？

因为 orientation 朝向这种量，本来就更适合描述一个刚体或一段骨头，而不是一个点。
点只有位置，没有朝向。
所以 Xsens 里的 Segment Orientation，本质上就是“这一段身体在全局系里怎么朝向”。





# AMASS数据集的数据格式

原始的amass数据集以.npz文件的格式存储，里面存储着一个字典，这个字典通常包括6个key：['trans', 'gender', 'mocap_framerate', 'betas', 'dmpls', 'poses']

以ACCAD数据集下的Female1General_c3d\A1 - Stand_poses.npz文件作为示例：

```raw
Total keys: 6
Keys: ['trans', 'gender', 'mocap_framerate', 'betas', 'dmpls', 'poses']

key             | shape      | dtype   | python_type
----------------+------------+---------+------------
trans           | (360, 3)   | float64 | ndarray    
gender          | scalar     | str     | ndarray    
mocap_framerate | scalar     | float   | ndarray    
betas           | (16,)      | float64 | ndarray    
dmpls           | (360, 8)   | float64 | ndarray    
poses           | (360, 156) | float64 | ndarray    
```





poses 的形状是 (360,156)，本质上表示“360 帧动作，每一帧有 156 个姿态参数”。AMASS 这里用的是 SMPL+H 风格的姿态参数，所以每帧可以 reshape 成 (52,3)(52,3)



这里的 52 个关节，每个关节用 3 维轴角表示旋转，所以：156=52×3。更细一点可以拆成：156=3+63+45+45。

对应含义通常是：

1. 前 3 维：根节点，也就是骨盆的全局朝向
2. 接下来 63 维：身体关节姿态，21×321×3
3. 再 45 维：左手，15×315×3
4. 最后 45 维：右手，15×315×3



poses 是人体骨架各关节的局部姿态参数表示；采用轴角形式存储。其余关节是相对父关节的局部旋转，不是直接的世界坐标系旋转。如果要得到每个关节在世界坐标系下的旋转，需要沿运动学链做前向运动学，把父关节旋转逐级累乘。
