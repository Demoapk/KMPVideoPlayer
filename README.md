# KMPVideoPlayer

一个以学习为目的的跨平台视频播放器项目，目标是把 **Kotlin Multiplatform（KMP）**、**跨平台视频播放** 和 **实时端侧 ASR（自动语音识别）** 串成一条完整的工程实践路线。

项目最终希望同时运行在 Android 与 iOS 上：界面与大部分业务逻辑由 Kotlin 共享，播放和语音识别分别调用两端的原生能力。

> 这是一个学习型项目，代码由本人逐步完成。当前仓库仍处于工程起步阶段，下面标注的目标并不代表已经全部实现。

## 项目目标

### 1. 掌握 Kotlin Multiplatform 与 Compose Multiplatform

- 理解 `commonMain`、`androidMain`、`iosMain` 的职责划分与编译关系。
- 能用 Kotlin 协程与 `StateFlow` 组织跨平台状态。
- 掌握 `expect/actual`，把播放器、文件选择、麦克风、端侧推理等平台能力隔离在清晰的接缝后面。
- 使用 Compose Multiplatform 共用一套声明式 UI，同时理解 Android 与 iOS 的宿主工程差异。

### 2. 掌握跨平台视频播放器

- 设计不依赖具体播放器的通用抽象接口，让 UI 层不直接依赖 Media3 或 AVPlayer。
- Android 使用 Media3 / ExoPlayer，iOS 使用 AVFoundation / AVPlayer，实现同一套播放语义。
- 完成播放/暂停、进度拖动、当前时间与总时长、加载态、错误态和资源释放。
- 支持内置示例视频、本地文件选择和网络 URL 三种视频来源。
- 理解平台视图互操作、播放器生命周期以及跨平台状态同步。

### 3. 掌握实时端侧 ASR

- 在设备本地完成语音识别，音频不依赖上传到云端。
- 建立从麦克风采集、VAD（语音活动检测）、流式推理到字幕展示的完整链路。
- 支持实时返回部分识别结果，并在语音结束后给出最终文本。
- 关注端侧实时性指标：首字延迟、推理耗时、断句稳定性、功耗与内存占用。
- 探索可在 Android 与 iOS 复用的端侧推理方案，例如 sherpa-onnx、whisper.cpp 或 ONNX Runtime Mobile，最终选型以真机验证结果为准。
- 通过 `expect/actual` 把录音、模型推理和生命周期管理接到两端平台实现。

## 最终形态

- **跨平台：** Android 与 iOS 共用 Compose UI 和核心业务逻辑。
- **可播放：** 支持内置视频、本地文件和网络 URL，具备完整的播放控制与异常处理。
- **可识别：** 在播放或录制过程中实时输出端侧 ASR 文本，可扩展到字幕与转写记录。
- **可理解：** 架构边界清楚，能够说明每个能力为什么放在共享层或平台层。

## 当前进度

当前仓库还是 Android + Jetpack Compose 的基础工程：

- 已完成：Kotlin / Compose Android 工程骨架、Gradle 构建配置、Git 版本管理。
- 下一步：从 Android 单端模板迁移到 Android + iOS 的 KMP 工程结构。
- 尚未实现：真实视频播放、本地文件与网络来源、实时端侧 ASR。

因此，README 中描述的 KMP、播放器和 ASR 能力属于项目目标与学习路线；实现进度会随着各阶段完成继续更新。

## 目标架构

```text
Compose Multiplatform UI（commonMain）
        |
        +-- Player 抽象接口（commonMain）
        |       +-- Android：Media3 / ExoPlayer
        |       +-- iOS：AVFoundation / AVPlayer
        |
        +-- ASR 抽象接口（commonMain）
                +-- Android：麦克风 + 端侧推理
                +-- iOS：AVAudioEngine + 端侧推理

端侧 ASR 数据流：
麦克风音频 -> 音频预处理 -> VAD -> 流式识别 -> 部分结果 / 最终文本 -> UI 字幕
```

架构约束：

1. UI 不直接依赖平台播放器或推理框架，只依赖共享层定义的接口和状态。
2. 平台差异集中在播放内核、视图容器、文件选择、音频采集和模型推理等少数位置。
3. 状态从平台实现单向流回共享层，由 Compose 根据状态重绘界面。
4. 端侧模型只在本地运行，录音和识别文本不默认上传网络。

## 技术栈

当前已使用：

| 类别 | 技术 |
| --- | --- |
| 语言 | Kotlin 2.2.10 |
| UI | Jetpack Compose / Material 3 |
| 构建 | Gradle + Android Gradle Plugin 9.2.1 |
| Android | minSdk 24 / targetSdk 36 |

计划引入：

| 类别 | 技术 |
| --- | --- |
| 跨平台 UI | Compose Multiplatform |
| Android 播放内核 | Media3 / ExoPlayer |
| iOS 播放内核 | AVFoundation / AVPlayer |
| 异步与状态 | Kotlin 协程、StateFlow |
| 端侧 ASR | sherpa-onnx / whisper.cpp / ONNX Runtime Mobile（候选，待真机评估） |

## 学习路线

| 阶段 | 主题 | 目标产出 |
| --- | --- | --- |
| 1 | 工程启蒙 | Android 与 iOS 都能运行同一份 Compose 界面 |
| 2 | Kotlin 与 Compose | 用状态驱动 UI，完成视频列表页 |
| 3 | 播放内核 | 通过 `expect/actual` 接入 Media3 与 AVPlayer |
| 4 | 播放器界面 | 控制条、进度、时间、加载与错误状态 |
| 5 | 视频来源 | 内置视频、系统文件选择、网络 URL |
| 6 | 实时端侧 ASR | 麦克风采集、VAD、流式识别与字幕展示 |
| 7 | 打磨与总结 | 全屏、生命周期、性能、测试与文档复盘 |

详细任务、自测清单和常见问题见 [KMP音视频播放器自学手册](./KMP音视频播放器自学手册.md)。

## 运行当前工程

环境要求：

- Android Studio（使用其内置 JDK）
- Android SDK 36
- 可用的 Android 模拟器或真机

构建 Debug APK：

```bash
./gradlew :app:assembleDebug
```

当前版本仅包含 Android 工程，iOS 入口会在 KMP 工程迁移阶段加入。

## 说明

- 本仓库只使用公开测试视频与可合法分发的模型资源。
- 端侧 ASR 涉及模型体积、内存占用和真机性能，方案会根据 Android / iOS 实测结果调整。
- 项目以学习和可解释性为先，每个阶段都保留独立提交，便于回顾架构演进过程。
