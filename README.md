# Whisper-Ollama-Translator
Real-time Japanese/English to Chinese subtitle translator using Faster-Whisper and Ollama (Hy-mt1.5). (基于 Whisper 和 Ollama 混元 7b 的实时日文中文字幕翻译器。
## ✨ 特性 (Features)

- **实时系统音频捕获**: 直接通过声卡回环捕获电脑上播放的任何声音（视频播放器、浏览器等）。
- **高性能语音识别**: 使用 `faster-whisper` 模型进行快速、准确的日文语音转写。
- **实时翻译**: 将识别到的日文实时翻译为中文。
- **美观的悬浮字幕UI**:
  - 现代化设计，半透明磨砂背景。
  - 始终置顶 (Always-on-top)。
  - 鼠标穿透 (可选) 或可拖拽调整位置。
  - 智能断句与滚动显示。

## 🛠️ 技术栈 (Tech Stack)

- **语言**: Python 3.10+
- **GUI**: PyQt6 (用于构建现代、平滑的悬浮窗口)
- **音频捕获**: PyAudioWPatch (支持 Windows WASAPI Loopback 音频捕获)
- **语音识别**: Faster-Whisper (推荐使用 `medium` 模型)
- **翻译**: Ollama (本地 LLM, 推荐 `hy-mt1.5:7b`) / Google Translate (Fallback)

## 📋 安装指南 (Installation)

1. 安装依赖:

   ```bash
   pip install -r requirements.txt
   ```
2. 安装 CUDA (可选, 但推荐):
   如果使用 NVIDIA 显卡，请确保安装了 CUDA 11.x 或 12.x 以及对应的 cuDNN，能大幅降低延迟。
3. **安装 Ollama**:

   - 请访问 [ollama.com](https://ollama.com) 下载并安装 Ollama。
   - 拉取推荐的翻译模型:
     ```bash
     ollama pull hy-mt1.5:7b
     ```
   - (可选) 3080实测可以跑到200ms内，如果需要更快的速度，可以拉取 `hy-mt1.5:1.8b`。
   - 我采集了日志中的 200+ 个数据点，计算出的 算术平均值 (Avg) 和 中位数 (Median) 如下：

### 📊 翻译模型耗时统计 (Time to First Token + Gen)

基于日志中的 200+ 个数据点采集，计算出的 **算术平均值 (Avg)** 和 **中位数 (Median)** 如下：

| 翻译模型 | 搭配 Whisper Medium (平均/中位数) | 搭配 Whisper Small (平均/中位数) | 模型特性评价 |
| :--- | :--- | :--- | :--- |
| ***混元 1.8b** | **139ms / 105ms** | 326ms / 307ms ⚠️ | **极速**。在 Medium 下快得惊人；在 Small 下反而变慢（原因见下）。 |
| **Gemma 4b** | 358ms / 344ms | 315ms / 298ms | **均衡**。无论输入好坏，通过时间比较稳定，约 0.35秒。 |
| **混元 7b** | 270ms / 262ms | **118ms / 101ms** ❓ | **稳重**。数据出现了反直觉的倒挂（见下方异常分析）。 |

---

### 🧐 数据异常与深度分析

您可能会注意到两个违背直觉的数据现象，这揭示了模型背后的运行逻辑：

#### 1. 为什么 Qwen 1.8b 在 Whisper Small 下反而变慢了？(139ms -> 326ms)
*   **输入质量影响输出长度**：在 Medium 下，输入准确简短，1.8b 迅速翻译完。但在 Small 下，输入全是“富士山、藤井、前世”等乱七八糟的词，1.8b 试图去解释这些乱码，生成了更长、更啰嗦的句子（例如试图把“富士山”合理化），导致生成耗时增加。

#### 2. 为什么 混元 7b 在 Whisper Small 下快得离谱？(270ms -> 118ms)
*   **拒绝机制**：查看日志发现，当 Whisper Small 输出离谱的 `嫌だって` (No/Hate) 或 `私` (I) 等极短碎片时，混元 7b 倾向于简短回答或直接输出原本的短语，触发了它的“短序列偏好”。它没有像 Gemma 那样去编故事，而是快速丢出了结果。这反而让它在平均数上显得“快”，但这是**无效的快**。

## 🚀 使用方法 (Usage)

1. 运行主程序:

   ```bash
   python main.py
   ```
2. **初始化**:

   - 程序首次启动时会自动下载 Whisper `medium` 模型 (约 1.5GB) 到 `models/` 目录。
   - 自动扫描并连接系统默认音频输出设备 (Loopback)。
3. **操作**:

   - 字幕悬浮窗会自动置顶显示。
   - **右键菜单/设置按钮**:
     - **🤖 模型设置**: 可以在此处切换 Whisper 模型或 Ollama 翻译模型 (支持自动扫描本地模型)。
     - **🎨 外观设置**: 调整背景颜色、透明度等。
   - 拖拽窗口边缘可调整大小，拖拽空白处可移动位置。
4. **开始体验**: 播放任意英文/日文视频（其他语言需要改prompt），实时中文字幕将立即呈现。

## 📦 打包与发布 (Build & Release)

本项目提供了一键打包脚本，可生成 Windows 可执行文件 (`.exe`)。

1. **运行打包脚本**:

   ```bash
   python build.py
   ```
2. **输出产物**:

   - 打包完成后，文件位于 `dist/LiveSubtitleTranslator/` 目录。
   - 双击 `LiveSubtitleTranslator.exe` 即可运行。
3. **关于模型文件**:

   - 默认打包不包含 `models/` 文件夹中的大模型（为了减小体积）。
   - **发布时**：您可以将本地下载好的 `models/medium` 文件夹复制到 `dist/LiveSubtitleTranslator/models/` 下，或者让用户首次运行时自动下载。

## 📂 项目结构

- `main.py`: 程序入口与协调逻辑。
- `core/audio_capture.py`: 音频录制与预处理。
- `core/stt_engine.py`: 语音识别引擎封装。
- `core/translator.py`: 翻译逻辑。
- `ui/overlay.py`: 字幕悬浮窗界面。
