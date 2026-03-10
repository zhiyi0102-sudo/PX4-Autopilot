# PX4-Autopilot 项目分析文档

## 项目概述

PX4-Autopilot 是一个开源的无人机自动驾驶仪软件平台，支持多旋翼（Multicopter）、固定翼（Fixed Wing）、垂直起降（VTOL）等多种飞行器。

---

## 主要目录结构

| 目录 | 说明 |
|------|------|
| `src/modules` | 核心飞行控制模块 |
| `src/drivers` | 硬件驱动 |
| `src/lib` | 公共库 |
| `src/include` | 头文件 |
| `msg` | ROS/MAVLink 消息定义 |
| `boards` | 板级支持包 |
| `platforms` | 平台支持 |

---

## 核心模块

### 1. commander（指挥官）
**职责**：状态机管理、飞行模式切换、安全检查
- **入口**：`Commander.cpp`
- **主要功能**：
  - 飞行状态管理（解锁/上锁）
  - 飞行模式切换
  - 安全检查（Health and Arming Checks）
  - 传感器校准

### 2. navigator（导航器）
**职责**：任务规划、航点导航
- **入口**：`navigator_main.cpp`
- **主要功能**：
  - 任务管理
  - 航点导航
  - 地理围栏
  - 返航点管理

### 3. mc_att_control / mc_pos_control（多旋翼控制）
**职责**：多旋翼姿态和位置控制
- **入口**：`McAttControl.cpp` / `McPosControl.cpp`
- **主要功能**：
  - 姿态控制（PID控制器）
  - 位置控制
  - 速度控制

### 4. ekf2（扩展卡尔曼滤波器）
**职责**：状态估计
- **入口**：`Ekf2.cpp`
- **主要功能**：
  - 位置估计
  - 速度估计
  - 姿态估计
  - 传感器融合

### 5. sensors（传感器管理器）
**职责**：传感器数据处理
- **入口**：`sensors_main.cpp`
- **主要功能**：
  - 传感器数据读取
  - 传感器融合
  - 偏差补偿

### 6. mavlink（通信）
**职责**：MAVLink协议通信
- **入口**：`mavlink_main.cpp`
- **主要功能**：
  - 与地面站通信
  - 参数设置
  - 任务上传/下载

---

## 模块调用关系

```
用户输入 (RC/MAVLink)
    ↓
commander (状态机)
    ↓
mc_att_control (姿态控制)
    ↓
mc_pos_control (位置控制)
    ↓
    ↓
ekf2 (状态估计) ← sensors (传感器数据)
    ↓
mixer (混控) → drivers (电机驱动)
```

---

## 编译入口

- **主 Makefile**：`Makefile`
- **CMake**：`CMakeLists.txt`
- **构建命令**：
  ```bash
  make px4_fmu-v5_default
  ```

---

## 主要消息（MSG）

| 消息名 | 说明 |
|--------|------|
| vehicle_attitude | 无人机姿态 |
| vehicle_local_position | 本地位置 |
| vehicle_global_position | 全球位置 |
| actuator_controls | 控制器输出 |
| vehicle_command | 车辆命令 |
| offboard_control_mode | 外部控制模式 |

---

## 飞行模式

1. **手动模式** (Manual)
2. **自稳模式** (Stabilized)
3. **位置模式** (Position)
4. **自动模式** (Auto)
5. **悬停模式** (Hover)
6. **返航模式** (Return)
7. **着陆模式** (Land)

---

## 重要参数

| 参数 | 说明 | 默认值 |
|------|------|--------|
| MPC_XY_VEL_MAX | 最大水平速度 | 12 m/s |
| MPC_Z_VEL_MAX_UP | 最大上升速度 | 3 m/s |
| MPC_TKO_SPEED | 起飞爬升速度 | 1 m/s |
| COM_RC_LOSS_T | RC丢失超时 | 0.5 s |

---

## 入口函数

典型的模块入口：
```cpp
// commander/Commander.cpp
int Commander::custom_command(int argc, char **argv)

// ekf2/Ekf2.cpp
int ekf2_main(int argc, char **argv)
```

---

*本文档由天赐生成 - 2026-03-10*
