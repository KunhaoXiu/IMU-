# AMASS: Archive of Motion Capture as Surface Shapes

一句话概括：将大量基于标记的光学人体动作捕捉数据集统一起来，使它们能够在一个共同的框架和参数化体系中被处理。



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

