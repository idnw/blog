# 显示技术入门教程：从零基础到理解 Linux 显示系统

---

## 第一章：像素 — 一切的起点

### 1.1 什么是像素

屏幕上的画面由无数个 **像素（Pixel）** 组成，每个像素是一个 **极小的彩色方块**。

```
放大来看，屏幕上一个"A"字：

  · · ■ ■ · ·
  · ■ · · ■ ·
  · ■ ■ ■ ■ ·
  · ■ · · ■ ·
  · ■ · · ■ ·

每个 ■ 和 · 就是一个像素
```

### 1.2 分辨率

分辨率 = 屏幕水平像素数 × 垂直像素数

```
常见分辨率：
  720p  =  1280 × 720   = 约 92 万像素
  1080p =  1920 × 1080  = 约 207 万像素
  4K    =  3840 × 2160  = 约 829 万像素
```

分辨率越高，像素越多，画面越细腻。

### 1.3 像素的颜色

每个像素的颜色由 **红（R）、绿（G）、蓝（B）** 三种光混合而成：

```
红 + 绿     = 黄
红 + 蓝     = 紫
绿 + 蓝     = 青
红 + 绿 + 蓝 = 白
全部关闭     = 黑
```

每个通道用一个数值表示亮度（通常 0~255）：

```
(255, 0,   0  ) = 纯红
(0,   255, 0  ) = 纯绿
(0,   0,   255) = 纯蓝
(255, 255, 255) = 白
(0,   0,   0  ) = 黑
(255, 165, 0  ) = 橙色
```

### 1.4 像素格式与色深

```
格式           每像素位数    说明
─────────────────────────────────────────
RGB565         16 bit      R5位 G6位 B5位，省内存，色彩较粗糙
RGB888         24 bit      R8 G8 B8，1677万色，"真彩色"
XRGB8888       32 bit      多8位填充，对齐到4字节，CPU处理更快
ARGB8888       32 bit      多8位Alpha（透明度），支持半透明效果
```

**色深**：每个通道的位数。8bit = 256 级亮度，10bit = 1024 级亮度（更平滑的渐变）。

---

## 第二章：屏幕是怎么亮起来的

### 2.1 物理屏幕的工作原理

**LCD（液晶）**：

```
背光灯（白光）
    │
    ▼
偏光片 → 液晶层 → 彩色滤光片 → 偏光片 → 你的眼睛
              │
        控制光的通过量
        （电压改变液晶排列）

每个像素有 R/G/B 三个子像素，各自独立控制亮度
```

**OLED（有机发光）**：

```
无需背光，每个像素自己发光

  [R发光体] [G发光体] [B发光体]  ← 一个像素
       │         │        │
    各自独立发光，可完全关闭 → 纯黑
```

### 2.2 屏幕怎么知道显示什么

屏幕本身 **不做任何计算**。它只做一件事：

> 按照收到的像素数据，依次点亮每个像素。

```
主机/SoC                              屏幕
┌──────────┐    像素数据 + 时序信号    ┌──────────┐
│ 显示控制器 │ ──────────────────────→ │ 面板驱动IC │ → 点亮像素
└──────────┘   (HDMI/MIPI DSI/eDP)   └──────────┘
```

所以关键问题变成了：**谁准备像素数据，怎么传给屏幕？**

---

## 第三章：Framebuffer — 像素数据的家

### 3.1 概念

Framebuffer 是 **内存中存放一整屏像素数据的区域**。

```
内存地址:   0x0000    0x0004    0x0008    ...
像素:      [B G R X] [B G R X] [B G R X] ...
屏幕位置:   (0,0)     (1,0)     (2,0)    ...
            ← ← ← 第一行 → → →
            ← ← ← 第二行 → → →
            ...
```

### 3.2 大小计算

```
1920 × 1080 分辨率，XRGB8888（4字节/像素）：

  大小 = 1920 × 1080 × 4 = 8,294,400 字节 ≈ 8 MB
```

### 3.3 如何画一个点

在 framebuffer 上画一个红色像素到坐标 (x, y)：

```
偏移 = y × stride + x × bytes_per_pixel

假设 stride = 1920 × 4 = 7680
画 (100, 200) 处的红点：
  偏移 = 200 × 7680 + 100 × 4 = 1,536,400
  在偏移处写入 [0x00, 0x00, 0xFF, 0x00]  （B=0, G=0, R=255, X=0）
```

### 3.4 Stride（行跨度）

```
实际每行字节数 ≥ width × bpp

为什么？内存对齐。硬件要求每行起始地址对齐到 64/128 字节。

  有效像素          padding
├──────────────────┤───┤
[pixel][pixel]...[pixel][pad]   ← 一行
[pixel][pixel]...[pixel][pad]   ← 下一行

stride = 有效像素字节 + padding
```

---

## 第四章：显示控制器 — 从内存到屏幕

### 4.1 显示控制器是什么

显示控制器（Display Controller）是 SoC/GPU 中的 **硬件模块**，负责：

1. 从内存中 **DMA 读取** framebuffer 数据
2. 按照屏幕要求的 **时序** 输出像素

```
      内存
  ┌──────────┐
  │Framebuffer│
  └────┬─────┘
       │ DMA 读取（硬件自动搬运，不需要 CPU 参与）
       ▼
  ┌──────────┐
  │ 显示控制器 │  ← 硬件，如 Rockchip VOP、Intel Display Engine
  └────┬─────┘
       │ 像素流 + 时序信号
       ▼
     屏幕
```

### 4.2 时序（Timing）— 最重要的概念之一

屏幕刷新画面的方式继承自 CRT 时代的 **逐行扫描**：

```
→ → → → → → → →  ─┐
→ → → → → → → →   │ 有效显示区域（Active Area）
→ → → → → → → →   │
→ → → → → → → →  ─┘
─ ─ ─ ─ ─ ─ ─ ─  ← 垂直消隐（Vertical Blanking）
                     这段时间不显示像素
```

一帧图像的时序参数：

```
水平方向（一行）：
  ┌──────────┬────────┬──────┬────────┐
  │ 有效像素  │ 前肩   │ 同步 │ 后肩   │
  │ (Active) │(Front  │(Sync)│(Back   │
  │ 1920 px  │ Porch) │      │ Porch) │
  └──────────┴────────┴──────┴────────┘

垂直方向（一帧）：
  ┌──────────┐
  │ 有效行    │ 1080 lines
  ├──────────┤
  │ 前肩      │ 4 lines
  ├──────────┤
  │ 同步脉冲  │ 5 lines
  ├──────────┤
  │ 后肩      │ 36 lines
  └──────────┘
```

这些参数合称 **mode / modeline**，决定了分辨率和刷新率。

### 4.3 刷新率

```
60Hz = 每秒刷新 60 帧
     = 每帧约 16.67 毫秒

像素时钟 = 总像素(含消隐) × 刷新率
1080p@60Hz ≈ 2200 × 1125 × 60 ≈ 148.5 MHz
```

### 4.4 VSync 与撕裂

```
撕裂（Tearing）：
  显示控制器读到一半时，CPU/GPU 更新了 framebuffer
  上半屏是旧画面，下半屏是新画面 → 画面撕裂

  ┌──────────────┐
  │  旧画面上半   │ ← 控制器已经读过
  ├──── 撕裂线 ──┤
  │  新画面下半   │ ← CPU 刚写入的新数据
  └──────────────┘

解决：VSync（垂直同步）
  在垂直消隐期间（屏幕不读数据时）切换 framebuffer
  通常配合双缓冲使用
```

### 4.5 双缓冲与 Page Flip

```
双缓冲：

  Front Buffer ← 显示控制器正在读
  Back Buffer  ← CPU/GPU 正在写下一帧

  VSync 到来时：
    交换两个 buffer 的角色（只改指针，不拷贝数据）
    这就是 Page Flip

  ┌─────────┐    ┌─────────┐
  │ Front   │    │ Back    │
  │ (显示中) │    │ (绘制中)│
  └────┬────┘    └────┬────┘
       │              │
       └──── swap ────┘  ← VSync 时刻
```

---

## 第五章：显示接口 — 像素怎么传到屏幕

### 5.1 接口分类

```
                    ┌─── HDMI    (消费电子，电视/显示器)
        ┌─ 数字 ───┼─── DisplayPort / eDP  (笔记本内屏)
        │           ├─── MIPI DSI (手机/嵌入式小屏)
信号 ───┤           └─── LVDS    (工控/车载屏)
        │
        └─ 模拟 ─── VGA     (老式显示器，基本淘汰)
```

### 5.2 各接口对比

| 接口 | 典型场景 | 信号类型 | 最大带宽 | 特点 |
|------|---------|---------|---------|------|
| **MIPI DSI** | 手机、嵌入式 | 差分串行 | ~4 Gbps/lane | 省电、短距离 |
| **LVDS** | 工控屏、车载 | 差分并行 | ~1 Gbps | 成熟稳定 |
| **eDP** | 笔记本内屏 | 串行 | ~32 Gbps | DisplayPort 的嵌入式版本 |
| **HDMI** | 电视、显示器 | TMDS 串行 | ~48 Gbps (2.1) | 支持音频、CEC |
| **DisplayPort** | 显示器 | 串行 | ~80 Gbps (2.0) | 支持 daisy chain |

### 5.3 HDMI 信号细节

```
HDMI 线缆内部：

  ┌─────────────────────────────┐
  │  TMDS Channel 0  (R + 同步) │ ← 差分信号对
  │  TMDS Channel 1  (G)       │
  │  TMDS Channel 2  (B)       │
  │  TMDS Clock                │ ← 像素时钟
  │  DDC (I2C)                 │ ← 读 EDID 用
  │  CEC                       │ ← 设备控制
  │  HPD                       │ ← 热插拔检测
  │  +5V                       │ ← 供电
  └─────────────────────────────┘
```

### 5.4 MIPI DSI 信号细节

```
手机/嵌入式最常用：

  SoC ─── DSI Host                 屏幕 ─── DSI Device
        ┌──────────┐              ┌──────────┐
        │ CLK+/CLK-│─────────────│          │
        │ D0+/D0-  │─────────────│          │
        │ D1+/D1-  │─────────────│  LCD     │
        │ D2+/D2-  │─────────────│  Panel   │
        │ D3+/D3-  │─────────────│          │
        └──────────┘              └──────────┘
        1 clock lane + 1~4 data lanes

  两种模式：
    Command Mode: 像操作内存一样写屏（屏有自己的显存）
    Video Mode:   持续推送像素流（更常见）
```

---

## 第六章：EDID — 屏幕的自我介绍

### 6.1 什么是 EDID

EDID（Extended Display Identification Data）是屏幕内部存储的一段数据，描述自己的能力。

```
显示器插入 HDMI/DP 后：

  主机 ──── DDC (I2C) ────→ 显示器
       ←── 返回 EDID 数据 ──

EDID 内容（128~256 字节）：
  ├── 厂商名称、型号
  ├── 屏幕物理尺寸
  ├── 支持的分辨率列表
  │     ├── 1920×1080 @ 60Hz (preferred)
  │     ├── 1280×720  @ 60Hz
  │     └── 3840×2160 @ 30Hz
  ├── 色彩能力
  └── 音频能力（HDMI）
```

### 6.2 为什么重要

主机 **必须** 读到 EDID 才能知道屏幕支持什么分辨率，否则只能用安全的默认值（如 640×480）。

```
开机流程：
  1. 检测到 HDMI 热插拔 (HPD)
  2. 通过 I2C 读 EDID
  3. 解析得到分辨率列表
  4. 选择最优分辨率（通常是 preferred mode）
  5. 配置时序，开始输出
```

---

## 第七章：渲染 — 像素从哪来

### 7.1 渲染 = 算像素

屏幕只认像素数据，但应用程序描述的是 "窗口"、"按钮"、"文字"、"3D模型"。
**渲染** 就是把这些抽象描述计算成具体的像素。

### 7.2 2D 渲染

```
输入: "在 (100,50) 画一个 200×40 的蓝色矩形，里面白色文字 'Hello'"

过程:
  1. 遍历 (100,50) 到 (300,90) 的所有像素，填蓝色
  2. 用字体库把 "Hello" 光栅化为像素点阵
  3. 在矩形中央覆盖白色文字像素

输出: framebuffer 中对应区域的像素值被更新
```

### 7.3 3D 渲染管线

```
3D 模型顶点                     屏幕上的 2D 像素
─────────                     ──────────────

  三角形顶点坐标
       │
  ┌────▼────┐
  │ 顶点着色器│  坐标变换、投影（3D → 2D）
  └────┬────┘
       │
  ┌────▼────┐
  │ 光栅化   │  确定三角形覆盖了哪些像素
  └────┬────┘
       │
  ┌────▼────┐
  │ 片段着色器│  计算每个像素的颜色（纹理、光照）
  └────┬────┘
       │
       ▼
  写入 Framebuffer
```

### 7.4 CPU 渲染 vs GPU 渲染

```
CPU 渲染（软件渲染）：
  通用处理器逐像素计算
  灵活但慢（核心少）
  适合：简单 2D、嵌入式、无 GPU 场景

GPU 渲染（硬件加速）：
  专用处理器，数千个小核心并行
  适合：大量像素的并行计算
  1080p = 200万像素，GPU 可以同时处理大量像素

       CPU                  GPU
  ┌─┬─┬─┬─┐      ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┐
  │█│█│█│█│      │█│█│█│█│█│█│█│█│█│█│  × 很多组
  └─┴─┴─┴─┘      └─┴─┴─┴─┴─┴─┴─┴─┴─┴─┘
  4~16 大核心      数千个小核心
  啥都能做         专精并行计算
```

---

## 第八章：图层与合成

### 8.1 为什么需要图层

桌面上同时有多个窗口，每个窗口独立渲染自己的内容。最终需要把它们 **叠加合成** 为一帧完整画面。

```
图层3（最上层）: 鼠标光标        ·  ↖
图层2:          终端窗口        ┌────────┐
图层1:          浏览器窗口   ┌──┤ 终端   │
图层0（最底层）: 桌面壁纸  ┌──┤ 浏览器  └──┐
                          │  │          │
                          │  └──────────┘
                          └──────────────┘
       │
       ▼ 合成 (compositing)
  ┌──────────────────┐
  │ 最终画面          │ → 写入 framebuffer → 屏幕
  │  壁纸+浏览器+终端+光标│
  └──────────────────┘
```

### 8.2 软件合成 vs 硬件合成

**软件合成**：CPU/GPU 把所有图层混合，写入一个最终 framebuffer。

```
GPU 读取所有图层 → 混合计算 → 写入一个 final FB → 显示控制器读取
```

**硬件合成（Hardware Overlay / Plane）**：显示控制器自身支持多图层叠加。

```
Plane 0 (背景)  ──→ ┌───────────┐
Plane 1 (视频)  ──→ │ 显示控制器  │──→ 输出混合后的像素流
Plane 2 (UI)    ──→ │ (硬件混合) │
Cursor Plane    ──→ └───────────┘

优势：不需要 GPU 参与合成，省电、省带宽
```

这就是 DRM 框架中 **Plane** 的概念。

---

## 第九章：Linux DRM 显示框架

### 9.1 为什么需要 DRM

早期 Linux 用 fbdev（`/dev/fb0`）直接操作 framebuffer，太简陋：
- 不支持多图层
- 不支持硬件加速
- 不支持热插拔
- 多进程同时访问会冲突

DRM（Direct Rendering Manager）解决了这些问题，是现代 Linux 唯一的显示框架。

### 9.2 DRM 核心抽象

```
DRM 用 5 个核心对象建模整个显示管道：

  Framebuffer
       │
       ▼
     Plane          "图层"，一个 plane 绑定一个 framebuffer
       │
       ▼
     CRTC           "扫描输出引擎"，读取 plane 数据，生成时序
       │
       ▼
    Encoder          "信号编码器"，把像素数据编码为接口协议
       │
       ▼
   Connector         "物理接口"，HDMI/DP/DSI 等
       │
       ▼
     屏幕
```

一个实际例子：

```
笔记本电脑：

  FB0 → Primary Plane ──→ CRTC0 ──→ Encoder(eDP) ──→ Connector(eDP) → 内屏
  FB1 → Primary Plane ──→ CRTC1 ──→ Encoder(HDMI) ──→ Connector(HDMI) → 外接显示器
  FB2 → Cursor Plane  ──→ CRTC0     (光标叠加在内屏上)
```

### 9.3 各对象详解

**Framebuffer**

```
struct drm_framebuffer
├── width, height          像素尺寸
├── format                 像素格式 (DRM_FORMAT_XRGB8888)
├── pitches[4]             每个 plane 的 stride
└── GEM object 引用        实际内存 buffer
```

**Plane（图层）**

```
类型：
  Primary  - 主图层（必须有）
  Overlay  - 叠加层（可选，用于视频叠加等）
  Cursor   - 鼠标光标（64×64 小图层）

属性：
  - 位置 (crtc_x, crtc_y)
  - 源矩形 (src_x, src_y, src_w, src_h)  ← 从 FB 中截取
  - 目标矩形 (crtc_w, crtc_h)             ← 在屏幕上的大小
  - rotation, zpos, alpha ...
```

**CRTC**

```
核心功能：
  1. 从 Plane 读取像素数据（DMA）
  2. 多个 Plane 硬件叠加混合
  3. 生成时序信号（hsync, vsync, pixel clock）
  4. 管理 VBlank 中断（page flip 的时机）

配置参数：mode（分辨率+时序）
```

**Encoder**

```
将 CRTC 输出的像素流编码为特定接口的电信号：
  - TMDS 编码 → HDMI
  - 8b/10b 编码 → DisplayPort
  - DSI 协议打包 → MIPI DSI

很多驱动中 encoder 是"直通"的，逻辑很薄
```

**Connector**

```
代表物理接口：
  - 类型：HDMI-A, DP, eDP, DSI, LVDS ...
  - 状态：connected / disconnected
  - EDID：连接后可读取
  - 支持的 mode 列表（从 EDID 解析）

热插拔检测（HPD）在 connector 层处理
```

### 9.4 Atomic Modesetting

现代 DRM 使用 **原子提交**：把所有变更打包成一次操作，要么全部成功，要么全部回滚。

```
旧方式（Legacy）：                  新方式（Atomic）：
  设置 CRTC 模式                    打包所有变更为一个 atomic state
  设置 Plane 位置                   ├── CRTC.mode = 1080p
  设置 Connector                    ├── Plane0.FB = fb_id
  每一步独立，可能中间态不一致       ├── Plane0.crtc = crtc_id
                                    ├── Connector.crtc = crtc_id
                                    └── 一次性提交（TEST_ONLY 可先测试）
```

### 9.5 GEM — 内存管理

GEM（Graphics Execution Manager）管理显存/内存 buffer：

```
用户空间                    内核
─────────                  ─────
1. 创建 buffer ──────→  GEM 分配内存（可能是 CMA/IOMMU 映射的物理连续内存）
2. mmap 映射   ──────→  映射到用户空间地址
3. CPU 写入像素
4. 创建 FB     ──────→  drm_framebuffer 引用此 GEM object
5. atomic commit ───→  CRTC 用 DMA 读取这块内存
```

### 9.6 DRM Bridge

当 SoC 通过外部芯片连接屏幕时，Bridge 框架将它们串联：

```
CRTC → Encoder → Bridge1 → Bridge2 → Connector
                    │          │
                 格式转换    HDMI发送器
                 (RGB→LVDS)  (dw-hdmi)
```

---

## 第十章：用户空间与内核的交互

### 10.1 设备节点

```
/dev/dri/
├── card0          ← DRM 设备，mode setting + rendering
├── card1
├── renderD128     ← 仅渲染（无 mode setting 权限）
└── renderD129
```

### 10.2 用户空间工具与库

```
应用层：    Firefox / 游戏 / 桌面环境
                │
图形 API：  OpenGL / Vulkan / EGL
                │
用户空间驱动：Mesa (GPU 用户空间驱动)
                │
显示服务：  Wayland Compositor / X11
                │
用户空间库：libdrm（封装 ioctl）
                │
      ──────────┼───────── 用户/内核边界 ──────────
                │
内核：      DRM Core → 平台驱动 (i915, rockchip, msm ...)
                │
硬件：      显示控制器 → 接口 → 屏幕
```

### 10.3 一次完整的显示流程（以 Wayland 为例）

```
1. 应用程序通过 EGL/Vulkan 渲染一帧到自己的 buffer
        │
2. 应用通过 Wayland 协议把 buffer 提交给 compositor
        │
3. Compositor 收到所有窗口的 buffer
        │
4. Compositor 决定布局（哪个窗口在上面、位置、大小）
        │
5. Compositor 通过 libdrm 配置 DRM：
   ├── 分配 Plane，绑定各窗口的 buffer
   ├── 或者 GPU 合成所有窗口到一个 FB
   └── Atomic commit 提交到内核
        │
6. 内核 DRM 驱动配置显示控制器硬件
        │
7. 显示控制器 DMA 读取 framebuffer
        │
8. 像素经过 Encoder → HDMI/DSI → 屏幕点亮
```

---

## 第十一章：嵌入式显示实战知识

### 11.1 设备树中的显示配置

嵌入式 Linux（ARM SoC）通过设备树（Device Tree）描述显示硬件连接关系：

```dts
/* 显示管道连接关系 */
&display_controller {
    status = "okay";

    port {
        dc_out: endpoint {
            remote-endpoint = <&hdmi_in>;  /* 连接到 HDMI */
        };
    };
};

&hdmi {
    status = "okay";

    port {
        hdmi_in: endpoint {
            remote-endpoint = <&dc_out>;
        };
    };
};

/* MIPI DSI 面板示例 */
&dsi {
    status = "okay";

    panel@0 {
        compatible = "vendor,panel-model";
        reg = <0>;
        reset-gpios = <&gpio1 5 GPIO_ACTIVE_LOW>;
        backlight = <&backlight>;
    };
};

/* 背光 */
backlight: backlight {
    compatible = "pwm-backlight";
    pwms = <&pwm0 0 25000 0>;
    brightness-levels = <0 10 20 ... 255>;
    default-brightness-level = <200>;
};
```

### 11.2 常用调试手段

```bash
# 查看 DRM 设备信息
cat /sys/class/drm/card0-HDMI-A-1/status    # connected / disconnected
cat /sys/class/drm/card0-HDMI-A-1/edid | edid-decode

# 查看当前显示状态
cat /sys/kernel/debug/dri/0/state            # atomic state 全貌

# 用 modetest 测试输出（libdrm 自带工具）
modetest -M rockchip -s 91@68:1920x1080      # connector@crtc:mode

# 用 modeprint 列出所有 DRM 对象
modetest -M rockchip -c    # connectors
modetest -M rockchip -p    # planes
modetest -M rockchip -e    # encoders

# 直接往 framebuffer 写彩条测试
modetest -M rockchip -P 43@68:1920x1080 -Ftiles
```

---

## 第十二章：完整知识图谱

```
                        ┌─────────────────────┐
                        │     应用程序          │
                        │  (浏览器/游戏/终端)   │
                        └──────────┬──────────┘
                                   │
                        ┌──────────▼──────────┐
                        │    图形 API           │
                        │ (OpenGL/Vulkan/EGL)  │
                        └──────────┬──────────┘
                                   │
              ┌────────────────────┼──────────────────────┐
              │                    │                       │
    ┌─────────▼────────┐ ┌────────▼────────┐  ┌──────────▼──────────┐
    │ GPU 渲染          │ │ 合成器           │  │ 2D 渲染             │
    │ (Mesa/GPU驱动)    │ │ (Wayland/X11)   │  │ (Cairo/Skia/CPU)    │
    └─────────┬────────┘ └────────┬────────┘  └──────────┬──────────┘
              │                    │                       │
              └────────────────────┼───────────────────────┘
                                   │ 渲染结果写入 buffer
                        ┌──────────▼──────────┐
                        │   Framebuffer (内存)  │
                        └──────────┬──────────┘
    ── ── ── ── ── ── ── ── ── ── ┼ ── ── ── ── ── ── 用户/内核边界
                        ┌──────────▼──────────┐
                        │ DRM/KMS 内核框架     │
                        │ Plane → CRTC →      │
                        │ Encoder → Connector  │
                        └──────────┬──────────┘
                                   │ 控制硬件
                        ┌──────────▼──────────┐
                        │   显示控制器硬件      │
                        │   (DMA + 时序生成)   │
                        └──────────┬──────────┘
                                   │ 电信号
                        ┌──────────▼──────────┐
                        │    显示接口           │
                        │ (HDMI/DSI/eDP/DP)   │
                        └──────────┬──────────┘
                                   │
                        ┌──────────▼──────────┐
                        │      屏幕            │
                        │   (LCD/OLED)        │
                        └─────────────────────┘
```

---

## 学习路径建议

```
第一步：理解像素和颜色       ← 第1章
    │
第二步：理解 framebuffer     ← 第3章
    │   （试着用 /dev/fb0 画点东西）
    │
第三步：理解时序和刷新       ← 第4章
    │
第四步：理解显示接口         ← 第5章
    │
第五步：理解 DRM 框架        ← 第9章
    │   （用 modetest 实践）
    │
第六步：理解渲染和合成       ← 第7、8章
    │
第七步：阅读一个具体平台驱动  ← 如 drivers/gpu/drm/rockchip/
    │
第八步：阅读 Wayland compositor 代码  ← 如 wlroots
```
