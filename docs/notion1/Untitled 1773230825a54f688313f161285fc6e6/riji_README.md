# 语音日志自动分析系统

## 1. 项目简介

本项目是一个全自动的语音日志和分析系统，结合了物联网硬件、Web服务和本地AI模型，实现了从声音采集、数据上传、语音转文字到内容总结的完整自动化流程。

- **前端硬件 (Seeed Studio XIAO ESP32S3 Sense)**：负责实时监听环境声音，通过人声检测算法智能启动录音，并将音频文件存储在SD卡中。设备具备低功耗管理能力，可在无活动时进入睡眠模式以节省电量。
- **数据传输**：ESP32设备通过Wi-Fi，将录制好的音频文件以HTTP POST请求的方式，自动上传到PC端运行的服务器。
- **后端服务 (PC端)**：
    1.  **音频接收服务器**：一个基于Python Flask的轻量级Web服务，用于接收并保存在ESP32上传的音频文件。
    2.  **语音转文本**：使用`yuyin_tq提取.py`脚本，调用OpenAI的Whisper模型，将音频文件批量转换为文本文档。
    3.  **内容分析与总结**：使用`ollama_text_processor.py`脚本，调用在本地运行的Ollama大语言模型（如Qwen3），根据预设的`prompt.md`模板，对转换后的文本进行智能分析和总结，生成最终的报告。

## 2. 系统架构

```mermaid
graph TD
    A[XIAO ESP32S3 Sense] -- 采集人声 --> B(录制为WAV文件);
    B -- 保存到 --> C[SD卡];
    C -- 定时/触发 --> D{通过WiFi上传};
    D -- HTTP POST --> E[PC端Flask服务器];
    E -- 保存到 --> F[音频文件夹];
    G[yuyin_tq提取.py] -- 读取 --> F;
    G -- 调用Whisper模型 --> H{语音识别};
    H -- 生成 --> I[文本文件 (text_info_YYYYMMDD.txt)];
    J[ollama_text_processor.py] -- 读取 --> I;
    J -- 读取 --> K[提示词模板 (prompt.md)];
    J -- 调用Ollama本地大模型 --> L{分析和总结};
    L -- 生成 --> M[最终报告 (result.md)];
```

## 3. 模块介绍

### 3.1. ESP32固件 (`xiao_esp32s3_sense_固件已测试_声音收集.cpp`)

- **核心功能**：低功耗音频采集和上传。
- **硬件平台**：Seeed Studio XIAO ESP32S3 Sense。
- **主要特性**：
    - **智能录音**：基于音量和FFT（快速傅里叶变换）分析，实现人声检测，只在检测到人声时录音，避免录制大量无用静音。
    - **低功耗管理**：集成轻度睡眠和深度睡眠模式，在无活动时自动降低功耗，延长电池续航。
    - **断点续传**：如果短时间内连续检测到语音，会自动追加到上一个音频文件中，避免产生过多碎片文件。
    - **定时上传**：配置为每天中午12点和晚上8点自动连接Wi-Fi，上传所有未上传的音频文件。
    - **健壮性设计**：包含WiFi连接超时、存储空间检查、电池电量监控等机制。

### 3.2. PC端音频接收服务 (`pc_音频接收.py`)

- **核心功能**：接收ESP32上传的音频文件。
- **技术栈**：Python, Flask。
- **使用方法**：
    1.  修改脚本中的`UPLOAD_FOLDER`变量，设置为你希望保存音频文件的路径。
    2.  运行`python pc_音频接收.py`启动服务。服务会监听在`0.0.0.0:5000`。
    3.  确保PC的IP地址和端口与ESP32固件中的`serverUrl`一致。

### 3.3. 语音转文本脚本 (`yuyin_tq提取.py`)

- **核心功能**：将指定文件夹内的音频文件批量转换为文本。
- **依赖**：OpenAI Whisper。
- **使用方法**：
    1.  修改`AUDIO_DIR`变量，指向音频文件所在的文件夹。
    2.  修改`WHISPER_MODEL`选择合适的模型大小（如`base`, `small`, `medium`）。性能越好的电脑可以选择越大的模型，识别效果更佳。
    3.  运行`python yuyin_tq提取.py`。脚本会自动处理所有音频，并将识别结果合并到`text_info_{当天日期}.txt`文件中。

### 3.4. Ollama文本处理脚本 (`ollama_text_processor.py`)

- **核心功能**：调用本地大模型，对识别出的文本进行总结。
- **依赖**：Ollama。
- **使用方法**：
    1.  确保你已经在本地安装并运行了Ollama，并且下载了所需模型（如`qwen3`）。
    2.  创建一个`prompt.md`文件，编写你希望AI执行任务的提示词。你可以使用占位符`{text_info}`来代指识别出的文本内容。
    3.  运行`python ollama_text_processor.py`。脚本会自动加载文本文件和提示词，调用Ollama API，并将结果保存到`result.md`中。

## 4. 环境搭建与安装

### 4.1. 硬件端 (ESP32)

1.  **IDE**: 安装 [Arduino IDE](https://www.arduino.cc/en/software) 或 [PlatformIO](https://platformio.org/)。
2.  **开发板支持**: 在Arduino IDE中，通过开发板管理器安装`esp32`支持。并选择`Seeed Studio XIAO ESP32S3`作为目标开发板。
3.  **库依赖**:
    - `WiFi`
    - `HTTPClient`
    - `FS`, `SD`, `SPI`
    - `ESP_I2S`
    - `arduinoFFT`
4.  **配置**:
    - 修改`xiao_esp32s3_sense_固件已测试_声音收集.cpp`文件中的WiFi账号和密码：
      ```cpp
      const char* ssid = "你的WiFi名称";
      const char* password = "你的WiFi密码";
      ```
    - 修改服务器地址，确保IP是你运行`pc_音频接收.py`的电脑的局域网IP：
      ```cpp
      const char* serverUrl = "http://192.168.x.x:5000/upload";
      ```
5.  **编译和上传**: 将代码编译并上传到你的XIAO ESP32S3 Sense开发板。

### 4.2. PC端 (软件)

1.  **Python**: 确保你安装了 Python 3.8 或更高版本。
2.  **依赖库**:
    ```bash
    # 安装Flask用于接收文件
    pip install Flask

    # 安装OpenAI Whisper用于语音识别
    pip install git+https://github.com/openai/whisper.git

    # 如果你的电脑有NVIDIA显卡，强烈建议安装PyTorch的CUDA版本以加速Whisper
    # 访问 https://pytorch.org/ 获取适合你CUDA版本的安装命令

    # 安装requests库用于调用Ollama API
    pip install requests
    ```
    
3.  **Ollama**:
    - 从 [Ollama官网](https://ollama.com/) 下载并安装Ollama。
    - 在命令行运行以下命令，下载你需要的模型：
      ```bash
      ollama pull qwen3
      ```
4.  **.env 文件 (可选)**:
    - 项目中包含一个`.env`文件模板，用于配置环境变量。虽然当前脚本中未直接使用`python-dotenv`库加载，但这是个好习惯。你可以将Ollama的地址等配置信息放在这里。

## 5. 使用流程

1.  **启动PC端服务**：
    - 在PC上打开一个终端，运行音频接收服务：
      ```bash
      python pc_音频接收.py
      ```
2.  **启动硬件**：
    - 给ESP32设备上电。它会自动开始监听声音，录音并保存到SD卡。在预设的时间点，它会自动上传文件到PC。
3.  **执行语音识别**：
    - 当音频文件上传到PC后，打开另一个终端，运行语音识别脚本：
      ```bash
      python yuyin_tq提取.py
      ```
    - 这会生成一个`text_info_YYYYMMDD.txt`文件。
4.  **执行内容总结**：
    - 准备好你的`prompt.md`文件。
    - 运行总结脚本：
      ```bash
      python ollama_text_processor.py
      ```
    - 检查生成的`result.md`文件，查看AI的分析总结。

## 6. 补充说明与自定义

- **提示词工程 (`prompt.md`)**: 这是整个系统的“大脑”。精心设计你的Prompt，可以引导AI完成各种任务，例如：
    - **会议纪要**：`请将以下会议内容整理成纪要，包含议题、决策和待办事项。`
    - **灵感提取**：`从以下语音记录中，提取出所有关于[某个主题]的想法和点子。`
    - **任务列表**：`根据以下内容，生成一个TODO列表。`
- **模型选择**:
    - **Whisper**: `medium`模型在中文识别上效果不错。如果追求更高精度且硬件允许，可以尝试`large`模型。
    - **Ollama**: `qwen3`是个不错的选择。你也可以尝试其他模型，如`llama3`等，只需在`ollama_text_processor.py`中修改`model_name`即可。
- **文件路径**: 请确保所有脚本中的文件路径（如`UPLOAD_FOLDER`, `AUDIO_DIR`）都已根据你的实际环境正确配置。
