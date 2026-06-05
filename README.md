# Vibe Coding Tool

一个软硬结合的 **蓝牙键盘 + 蓝牙麦克风 + 语音识别** 项目。

## 核心体验

```
┌─────────────────────────┐
│  自定义蓝牙键盘（带麦克风） │
│                         │
│  物理按键 → BT HID →    │
│  内置麦克风 → BT 音频 →  │
└──────────┬──────────────┘
           │  蓝牙连接
           ▼
      ┌─────────┐
      │   PC    │
      │         │
      │ 按键输入  │──→ 正常打字
      │ 音频流   │──→ STT 识别 ──→ 文字输出
      └─────────┘
```

- **实体蓝牙键盘**：正常打字，蓝牙连接电脑
- **内置麦克风**：键盘上集成麦克风，通过蓝牙传输音频到 PC
- **语音识别软件**：PC 端接收音频流，实时 STT 转为文字输出
- 说话和打字可以同时进行，互不干扰

---

## 架构

### 硬件：蓝牙键盘 + 麦克风

| 模块 | 选型 |
|------|------|
| 主控芯片 | ESP32-S3（BLE + 蓝牙音频 + I2S 麦克风）或 nRF52840（低功耗蓝牙） |
| 键盘矩阵 | 机械轴 + 二极管矩阵，GPIO 扫描 |
| 麦克风 | I2S MEMS 数字麦克风（INMP441） |
| 蓝牙 | BLE HID（键盘）+ A2DP / BLE Audio（音频流） |
| 电源 | 锂电池 + 充电管理（TP4056） |
| 外壳 | 3D 打印 / CNC |

### 软件：PC 端语音识别

| 模块 | 技术选型 |
|------|---------|
| 音频接收 | 系统蓝牙音频输入设备 |
| 语音活动检测 (VAD) | Silero VAD / 能量阈值 |
| 语音识别 (STT) | faster-whisper / whisper.cpp |
| 文字输出 | 模拟键盘输入（xdotool / uinput） |
| 自定义词典 | YAML 配置，术语替换 |

---

## 开发计划

### Phase 1 — 软件原型（先验证可行性）
先用现有蓝牙耳机/麦克风 + PC 端软件跑通：
```
蓝牙麦克风 → PC → VAD → faster-whisper → 键盘输出
```
- [ ] 音频采集（sounddevice / pyaudio）
- [ ] VAD 语音检测
- [ ] faster-whisper STT
- [ ] 键盘输出模拟
- [ ] 快捷键触发（按一个键开始/停止录音）

### Phase 2 — 硬件设计

#### 2a. 电路原理图 + PCB（KiCad）
- [ ] 主控 + 天线匹配电路
- [ ] 键盘矩阵电路（如 6×15 = 90 键）
- [ ] I2S 麦克风电路
- [ ] 锂电池充电 + 电源管理
- [ ] PCB Layout + 打样

#### 2b. 固件（ESP-IDF / Arduino）
- [ ] BLE HID 键盘
- [ ] I2S 麦克风驱动
- [ ] 蓝牙音频传输到 PC
- [ ] 按键扫描 + 去抖

#### 2c. 外壳
- [ ] 3D 建模
- [ ] 3D 打印 / CNC

### Phase 3 — 打磨优化
- [ ] 实时流式识别（低延迟）
- [ ] 自定义词典
- [ ] 跨平台（Linux / macOS / Windows）

---

## 开发环境

### 电路设计
```bash
# 已安装
sudo apt install kicad   # 原理图 + PCB
```

### PC 端软件
```bash
# 已安装
pip install vosk                    # 轻量 STT（备选）
sudo apt install xdotool            # 键盘模拟

# 待安装
pip install faster-whisper          # 高质量 STT（推荐）
pip install sounddevice pynput      # 音频采集 + 键盘控制
```

### 固件开发
```bash
# ESP-IDF
git clone --recursive https://github.com/espressif/esp-idf.git
cd esp-idf && ./install.sh esp32s3
```

---

## 项目结构

```
vibe-coding-tool/
├── README.md
├── software/              # PC 端语音识别软件
│   ├── main.py
│   ├── audio_capture.py
│   ├── vad.py
│   ├── stt.py
│   └── keyboard_output.py
├── firmware/              # 键盘固件（ESP32）
│   ├── main/
│   └── components/
├── hardware/              # KiCad 电路设计
│   ├── schematic/
│   └── pcb/
└── enclosure/             # 外壳 3D 模型
```

---

## 许可证

MIT
