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
   - ![1769680885592](image/README/1769680885592.png)

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
