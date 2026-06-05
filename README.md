# Vibe Coding Tool

一个软硬结合的 **语音 + 键盘 超级输入法** 项目。通过语音识别将语音实时转为文字/代码输入，与物理键盘并行工作，跨平台使用。

## 核心体验

```
说话 → 自动转文字 → 模拟键盘输出 → 任意编辑器/应用
                                    ↑
                              物理键盘照常可用
```

- **语音输入**：说话即输入，解放双手
- **键盘直通**：物理键盘正常使用，两者不冲突
- **模式切换**：快捷键在「语音模式」「纯键盘模式」间切换
- **命令映射**：特定语音短语映射为快捷键组合（如 "运行" → 保存并执行）
- **跨平台**：目标支持 Linux / macOS / Windows

---

## 整体架构

### Phase 1 — 纯软件原型

```
[麦克风] → VAD 语音检测 → STT 语音识别 → OS 输入模拟 → 目标应用
                                             ↑
                                        [物理键盘]
```

| 模块 | 技术选型 |
|------|---------|
| 音频采集 | PyAudio / sounddevice |
| 语音活动检测 (VAD) | Silero VAD |
| 语音识别 (STT) | openai-whisper / faster-whisper |
| 键盘模拟 | Linux: uinput / Mac: CGEvent / Win: SendInput |
| 热词唤醒 | 按键触发（长按/快捷键） |

### Phase 2 — 硬件设备

```
┌─────────────────────────────────────┐
│ 自定义硬件设备 (USB Dongle)          │
│                                     │
│  ┌──────────┐   ┌───────────────┐   │
│  │ MEMS 麦   │ → │ ESP32-S3      │   │
│  │ 克风阵列  │   │               │   │
│  └──────────┘   │ VAD + STT     │   │
│                 │ (本地/流式)    │   │
│  ┌──────────┐   │               │   │
│  │ LED 指示  │ ← │ USB HID 输出  │───┼──→ PC (识别为标准键盘)
│  │ 灯/按键   │   │               │   │
│  └──────────┘   └───────────────┘   │
└─────────────────────────────────────┘
```

| 模块 | 技术选型 |
|------|---------|
| 主控 | ESP32-S3 (USB OTG + WiFi/BLE) |
| 麦克风 | I2S MEMS 数字麦克风 (INMP441 / SPH0645) |
| 语音识别(本地) | whisper.cpp tiny 模型 |
| 语音识别(流式) | 音频流式传 PC，PC 侧推理 |
| USB 接口 | USB HID Keyboard 设备，无需驱动 |
| 供电 | USB 总线供电，< 500mA |

### 电路板设计

硬件 PCB 设计使用 **[KiCad](https://www.kicad.org/)** (开源 EDA 工具)，需安装：

```bash
# Ubuntu/Debian
sudo apt install kicad

# macOS
brew install --cask kicad

# Windows
# 从 kicad.org 下载安装包
```

主要电路模块：
- ESP32-S3 最小系统（晶振、Flash、USB-C 接口）
- I2S MEMS 麦克风电路
- 3.3V LDO 稳压
- USB HID 接口（直接使用 ESP32-S3 内置 USB）
- 状态 LED + 功能按键
- 可选：功率放大 + 扬声器（语音反馈）

---

## 开发环境

### Python 环境 (Phase 1)

```bash
# 系统依赖
sudo apt install portaudio19-dev ffmpeg    # Linux
brew install portaudio ffmpeg               # macOS

# Python 依赖
pip install pyaudio sounddevice pynput openai-whisper numpy
```

### ESP32 开发环境 (Phase 2)

```bash
# ESP-IDF 框架
git clone --recursive https://github.com/espressif/esp-idf.git
cd esp-idf && ./install.sh esp32s3

# 或 Arduino 框架
# Arduino IDE + ESP32 板支持包
```

---

## 项目结构 (规划)

```
vibe-coding-tool/
├── README.md
├── software/                  # Phase 1: 纯软件原型
│   ├── main.py               # 主入口
│   ├── audio_capture.py      # 音频采集 (PyAudio)
│   ├── vad.py                # 语音活动检测
│   ├── stt.py                # 语音识别
│   ├── keyboard_output.py    # 键盘输入模拟
│   └── config.py             # 配置
├── firmware/                  # Phase 2: ESP32 固件
│   ├── main/
│   ├── components/
│   │   ├── audio/
│   │   ├── hid/
│   │   └── wifi_transport/
│   └── CMakeLists.txt
├── hardware/                  # Phase 2: 电路设计
│   ├── schematic/            # KiCad 原理图
│   ├── pcb/                  # KiCad PCB 布局
│   ├── bom/                  # 物料清单
│   └── enclosure/            # 外壳设计 (FreeCAD/OpenSCAD)
└── docs/
    └── architecture.md
```

---

## 开发路线图

- [ ] **Phase 1** — Python 纯软件原型（音频采集 → VAD → STT → 键盘输出）
- [ ] **Phase 2a** — KiCad 电路原理图 + PCB 设计
- [ ] **Phase 2b** — ESP32-S3 固件 (USB HID + 音频采集 + STT)
- [ ] **Phase 2c** — PCB 打样 + 焊接调试
- [ ] **Phase 3** — 流式识别优化 + 低延迟打磨
- [ ] **Phase 4** — macOS / Windows 适配

---

## 许可证

MIT
