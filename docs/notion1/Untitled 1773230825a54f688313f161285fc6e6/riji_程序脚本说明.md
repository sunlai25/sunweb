# 程序脚本说明

本文档包含项目中所有程序脚本的简要介绍和代码实现。

## 1. Ollama文本处理器 (ollama_text_processor.py)

这是一个用于调用Ollama API进行文本处理的Python脚本。它支持加载提示词模板、聊天记录和文本信息，构建完整的提示词，并调用Ollama模型生成内容。

```python
import requests
import json
import os
from datetime import datetime
from pathlib import Path

class OllamaTextProcessor:
    def __init__(self, model_name="qwen3", ollama_url="http://localhost:11434"):
        """
        初始化Ollama文本处理器
        
        Args:
            model_name: Ollama模型名称
            ollama_url: Ollama服务地址
        """
        self.model_name = model_name
        self.ollama_url = ollama_url
        self.api_endpoint = f"{ollama_url}/api/generate"
    
    def load_prompt_template(self, prompt_file="prompt.md"):
        """
        加载提示词模板
        
        Args:
            prompt_file: 提示词文件路径
            
        Returns:
            str: 提示词内容
        """
        try:
            with open(prompt_file, 'r', encoding='utf-8') as f:
                return f.read()
        except FileNotFoundError:
            print(f"提示词文件 {prompt_file} 未找到")
            return ""
    
    def load_chat_records(self, chat_file):
        """
        加载聊天记录
        
        Args:
            chat_file: 聊天记录文件路径
            
        Returns:
            str: 聊天记录内容
        """
        try:
            with open(chat_file, 'r', encoding='utf-8') as f:
                return f.read()
        except FileNotFoundError:
            print(f"聊天记录文件 {chat_file} 未找到")
            return ""
    
    def load_text_info(self, text_file):
        """
        加载文本信息
        
        Args:
            text_file: 文本信息文件路径
            
        Returns:
            str: 文本信息内容
        """
        try:
            with open(text_file, 'r', encoding='utf-8') as f:
                return f.read()
        except FileNotFoundError:
            print(f"文本信息文件 {text_file} 未找到")
            return ""
    
    def build_prompt(self, template, chat_records, text_info):
        """
        构建完整的提示词
        
        Args:
            template: 提示词模板
            chat_records: 聊天记录
            text_info: 文本信息
            
        Returns:
            str: 完整的提示词
        """
        # 可以根据模板中的占位符进行替换
        # 例如：{chat_records} 和 {text_info}
        prompt = template.replace("{chat_records}", chat_records)
        prompt = prompt.replace("{text_info}", text_info)
        
        return prompt
    
    def call_ollama(self, prompt):
        """
        调用Ollama API生成内容
        
        Args:
            prompt: 完整的提示词
            
        Returns:
            str: 生成的内容
        """
        payload = {
            "model": self.model_name,
            "prompt": prompt,
            "stream": False
        }
        
        try:
            response = requests.post(self.api_endpoint, json=payload)
            response.raise_for_status()
            
            result = response.json()
            return result.get("response", "")
            
        except requests.exceptions.RequestException as e:
            print(f"调用Ollama API失败: {e}")
            return ""
    
    def save_result(self, content, output_file=None):
        """
        保存生成的内容到文件
        
        Args:
            content: 生成的内容
            output_file: 输出文件路径，如果为None则自动生成
        """
        if output_file is None:
            timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
            output_file = f"generated_content_{timestamp}.txt"
        
        try:
            with open(output_file, 'w', encoding='utf-8') as f:
                f.write(content)
            print(f"内容已保存到: {output_file}")
        except Exception as e:
            print(f"保存文件失败: {e}")
    
    def process(self, chat_file, text_file, prompt_file="prompt.md", output_file=None):
        """
        主处理流程
        
        Args:
            chat_file: 聊天记录文件路径
            text_file: 文本信息文件路径
            prompt_file: 提示词文件路径
            output_file: 输出文件路径
        """
        print("开始处理...")
        
        # 1. 加载所有输入文件
        print("加载提示词模板...")
        prompt_template = self.load_prompt_template(prompt_file)
        
        print("加载聊天记录...")
        chat_records = self.load_chat_records(chat_file)
        
        print("加载文本信息...")
        text_info = self.load_text_info(text_file)
        
        if not prompt_template:
            print("提示词模板为空，无法继续处理")
            return
        
        # 2. 构建完整提示词
        print("构建提示词...")
        full_prompt = self.build_prompt(prompt_template, chat_records, text_info)
        
        # 3. 调用Ollama生成内容
        print("调用Ollama生成内容...")
        generated_content = self.call_ollama(full_prompt)
        
        if generated_content:
            # 4. 保存结果
            print("保存生成的内容...")
            self.save_result(generated_content, output_file)
            print("处理完成!")
        else:
            print("生成内容失败")

def main():
    """
    主函数示例
    """
    # 创建处理器实例
    processor = OllamaTextProcessor(model_name="qwen3")
    
    # 示例用法
    today_str = datetime.today().strftime("%Y%m%d")
    chat_file = "chat.txt"      # 聊天记录文件
    text_file = f"text_info_{today_str}.txt"     # 自动匹配的文本信息文件
    prompt_file = "prompt.md"           # 提示词文件
    output_file = "result.md"          # 输出文件
    
    # 执行处理
    processor.process(chat_file, text_file, prompt_file, output_file)

if __name__ == "__main__":
    main()
```

## 2. PC音频接收服务器 (pc_音频接收.py)

这是一个基于Flask的简单服务器，用于接收ESP32设备上传的音频文件，并保存到指定目录。

```python
from flask import Flask, request
import os
import time
from datetime import datetime

app = Flask(__name__)
UPLOAD_FOLDER = 'D:\\esp32\\one'  # 替换为你的保存路径
LOG_FILE = 'file_upload.log'  # 日志文件名
os.makedirs(UPLOAD_FOLDER, exist_ok=True)

def log_upload(filename):
    """记录上传日志"""
    timestamp = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
    log_entry = f"{timestamp} - 文件上传成功: {filename}\n"
    
    # 写入日志文件
    with open(LOG_FILE, 'a', encoding='utf-8') as f:
        f.write(log_entry)

@app.route('/upload', methods=['POST'])
def upload_file():
    # 直接获取原始音频数据
    audio_data = request.data
    if not audio_data:
        return 'No data received', 400

    # 生成文件名
    filename = f"audio_{int(time.time())}.wav"
    file_path = os.path.join(UPLOAD_FOLDER, filename)
    
    # 保存文件
    with open(file_path, 'wb') as f:
        f.write(audio_data)
    
    # 记录日志
    log_upload(filename)
    
    return 'File uploaded successfully', 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

## 3. ESP32音频收集固件 (xiao esp32s3 sense 固件已测试_声音收集.cpp)

这是一个为XIAO ESP32S3 Sense开发板编写的Arduino固件，用于实现低功耗音频录制、人声检测、本地存储和网络上传功能。（注意！！arduino IDE工具中要开启PSRAM：OPIpsram ，因为esp32s3带宽比较高）

```cpp
#include <FS.h>
#include <SD.h>
#include <SPI.h>
#include <WiFi.h>
#include <HTTPClient.h>
#include <time.h>
#include <ESP_I2S.h>
#include <arduinoFFT.h>
#include <driver/rtc_io.h>
#include <esp_sleep.h>
#include <esp_wifi.h>
#include <esp_pm.h>

// ===== 硬件配置 =====
#define SD_CS_PIN 21
#define SD_SCK_PIN 7
#define SD_MISO_PIN 8
#define SD_MOSI_PIN 9
// #define SD_POWER_PIN 6  // XIAO ESP32S3 Sense 没有此功能，注释掉

#define I2S_WS 42
#define I2S_SD 41
#define I2S_SCK 1
#define WAKE_UP_PIN 0   // 唤醒引脚（按键或声音检测）

// ===== 功耗优化参数 =====
#define LIGHT_SLEEP_DURATION 5000000    // 5秒轻度睡眠
#define DEEP_SLEEP_DURATION 30000000    // 30秒深度睡眠
#define INACTIVITY_TIMEOUT 60000        // 60秒无活动进入深度睡眠
#define WIFI_CONNECT_TIMEOUT 15000      // WiFi连接超时
#define BATTERY_CHECK_INTERVAL 300000   // 5分钟检查一次电池

// ===== 音频参数（优化后） =====
#define SAMPLE_RATE 8000               // 降低到8kHz，人声足够
#define SAMPLE_BUFFER_SIZE 256         // 减小缓冲区
#define I2S_PORT I2S_NUM_0

// ===== WiFi 配置 =====
const char* ssid = "[WIFI_SSID]";  // 隐藏WiFi名称
const char* password = "[WIFI_PASSWORD]";  // 隐藏WiFi密码

// ===== 服务器配置 =====
const char* serverUrl = "http://[SERVER_IP]:5000/upload";  // 隐藏服务器地址

// ===== 存储配置 =====
#define MIN_RECORDING_SPACE 50 * 1024 * 1024
#define AUDIO_CACHE_SIZE 32768         // 32KB音频缓存

// ===== 人声检测参数（优化后） =====
#define VOLUME_THRESHOLD 80            // 稍微降低阈值
#define FFT_THRESHOLD 150              // 降低FFT阈值 
#define SILENCE_TIMEOUT 3000           // 修正：延长静音超时到3秒，使录音更自然
#define MIN_VOICE_DURATION 300         // 最小人声持续时间（ms）
#define RECORDING_COOLDOWN_PERIOD 10000 // 新增：10秒冷却时间，用于合并录音
#define MAX_CONTINUOUS_RECORDING 300000 // 最大连续录音5分钟

// ===== 工作模式枚举 =====
enum PowerMode {
  POWER_ACTIVE,          // 活跃模式
  POWER_LIGHT_SLEEP,     // 轻度睡眠
  POWER_DEEP_SLEEP,      // 深度睡眠
  POWER_WIFI_OFF         // WiFi关闭模式
};

// ===== 全局变量 =====
bool isVoiceDetected = false;
bool isRecording = false;
bool wifiConnected = false;
bool sdCardPowered = true;
unsigned long lastVoiceTime = 0;
unsigned long lastActivityTime = 0;
unsigned long recordStartTime = 0;
unsigned long historicStartTime = 0;
PowerMode currentPowerMode = POWER_ACTIVE;
String lastFilename = "";              // 新增：记录上一个文件名
unsigned long lastRecordingStopTime = 0; // 新增：记录上次录音停止时间

File audioFile;
double vReal[64];                      // 减小FFT大小
double vImag[64];
ArduinoFFT FFT = ArduinoFFT(vReal, vImag, 64, (double)SAMPLE_RATE);
I2SClass I2S;
int16_t audioBuffer[SAMPLE_BUFFER_SIZE];
uint8_t audioCache[AUDIO_CACHE_SIZE];  // 音频缓存
uint32_t cacheIndex = 0;

// ===== 电源管理配置 =====
void setupPowerManagement() {
  // 配置CPU频率管理
  esp_pm_config_esp32s3_t pm_config = {
    .max_freq_mhz = 80,      // 最大频率80MHz
    .min_freq_mhz = 10,      // 最小频率10MHz
    .light_sleep_enable = true
  };
  esp_pm_configure(&pm_config);
  
  // 配置GPIO唤醒
  esp_sleep_enable_ext0_wakeup(GPIO_NUM_0, 0);
  
  // 配置定时唤醒
  esp_sleep_enable_timer_wakeup(LIGHT_SLEEP_DURATION);
  
  Serial.println("电源管理配置完成");
}

// ===== CPU频率动态调节 =====
void setCPUFrequency(uint32_t freq) {
  static uint32_t currentFreq = 80;
  if (currentFreq != freq) {
    setCpuFrequencyMhz(freq);
    currentFreq = freq;
    Serial.printf("CPU频率调整为: %dMHz\n", freq);
  }
}

// ===== SD卡电源控制 =====
void powerSD(bool enable) {
  // XIAO ESP32S3 Sense 没有硬件支持，此功能无效
  // #ifdef SD_POWER_PIN
  // if (enable && !sdCardPowered) {
  //   pinMode(SD_POWER_PIN, OUTPUT);
  //   digitalWrite(SD_POWER_PIN, HIGH);
  //   delay(100);  // 等待SD卡上电稳定
  //   if (initSDCard()) {
  //     sdCardPowered = true;
  //     Serial.println("SD卡已上电");
  //   }
  // } else if (!enable && sdCardPowered) {
  //   SD.end();
  //   digitalWrite(SD_POWER_PIN, LOW);
  //   sdCardPowered = false;
  //   Serial.println("SD卡已断电");
  // }
  // #endif
}

// ===== WiFi智能连接 =====
bool connectWiFiSmart() {
  if (WiFi.status() == WL_CONNECTED) {
    return true;
  }
  
  setCPUFrequency(160);  // 提高频率以快速连接
  
  // 使用快速连接模式
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);
  
  unsigned long startTime = millis();
  while (WiFi.status() != WL_CONNECTED && millis() - startTime < WIFI_CONNECT_TIMEOUT) {
    delay(100);
  }
  
  if (WiFi.status() == WL_CONNECTED) {
    wifiConnected = true;
    Serial.println("WiFi快速连接成功");
    return true;
  } else {
    WiFi.disconnect();
    WiFi.mode(WIFI_OFF);
    wifiConnected = false;
    Serial.println("WiFi连接失败，进入离线模式");
    return false;
  }
}

// ===== WiFi断开连接 =====
void disconnectWiFi() {
  if (wifiConnected) {
    WiFi.disconnect();
    WiFi.mode(WIFI_OFF);
    wifiConnected = false;
    Serial.println("WiFi已断开");
  }
}

// ===== 进入睡眠模式 =====
void enterSleepMode(PowerMode mode) {
  if (isRecording) {
    return;  // 录音时不能睡眠
  }
  
  Serial.printf("进入睡眠模式: %d\n", mode);
  
  switch (mode) {
    case POWER_LIGHT_SLEEP:
      // 致命问题：Light Sleep会暂停CPU，导致I2S数据丢失，设备变"聋"。
      // 暂时禁用此功能，以保证基本功能可用。
      // setCPUFrequency(10);
      // esp_light_sleep_start();
      // setCPUFrequency(80);
      return; // 直接返回，不进入Light Sleep
      
    case POWER_DEEP_SLEEP:
      // 深度睡眠：关闭所有外设
      stopI2S();
      disconnectWiFi();
      powerSD(false);
      esp_deep_sleep_start();
      break;
      
    case POWER_WIFI_OFF:
      // WiFi关闭模式：保持音频功能
      disconnectWiFi();
      setCPUFrequency(80); // 修正：维持80MHz以确保音频处理正常，40MHz过低。
      break;
  }
  
  currentPowerMode = mode;
}

// ===== 检查是否需要睡眠 =====
void checkSleepConditions() {
  unsigned long currentTime = millis();
  
  // 检查是否长时间无活动
  if (currentTime - lastActivityTime > INACTIVITY_TIMEOUT) {
    if (!isRecording) {
      enterSleepMode(POWER_DEEP_SLEEP);
    }
  }
  // 暂时禁用Light Sleep，因为它会导致音频数据丢失
  // else if (currentTime - lastVoiceTime > 10000 && !isRecording) {
  //   enterSleepMode(POWER_LIGHT_SLEEP);
  // }
  // 检查是否可以关闭WiFi
  else if (currentTime - lastVoiceTime > 30000 && wifiConnected) {
    enterSleepMode(POWER_WIFI_OFF);
  }
}

// ===== I2S配置（优化版） =====
void setupI2S() {
  I2S.setPinsPdmRx(I2S_WS, I2S_SD);
  if (!I2S.begin(I2S_MODE_PDM_RX, SAMPLE_RATE, I2S_DATA_BIT_WIDTH_16BIT, I2S_SLOT_MODE_MONO, I2S_PORT)) {
    Serial.println("Failed to initialize I2S!");
    while (1);
  }
  Serial.println("I2S麦克风初始化完成（8kHz）");
}

void stopI2S() {
  I2S.end();
  Serial.println("I2S已停止");
}

// ===== WAV文件头生成（8kHz版本） =====
void writeWavHeader(File file) {
  const int byteRate = SAMPLE_RATE * 2;
  byte header[44] = {
    'R','I','F','F',0,0,0,0,'W','A','V','E',
    'f','m','t',' ',16,0,0,0,1,0,1,0,
    (byte)(SAMPLE_RATE & 0xFF), (byte)((SAMPLE_RATE >> 8) & 0xFF),
    (byte)((SAMPLE_RATE >> 16) & 0xFF), (byte)((SAMPLE_RATE >> 24) & 0xFF),
    (byte)(byteRate & 0xFF), (byte)((byteRate >> 8) & 0xFF),
    (byte)((byteRate >> 16) & 0xFF), (byte)((byteRate >> 24) & 0xFF),
    2,0,16,0,'d','a','t','a',0,0,0,0
  };
  file.write(header, 44);
}

// ===== 更新WAV文件头 =====
void updateWavHeader(File file) {
  if (!file) return;
  int dataSize = file.size() - 44;
  file.seek(4);
  uint32_t fileSize = file.size() - 8;
  file.write((uint8_t*)&fileSize, 4);
  file.seek(40);
  file.write((uint8_t*)&dataSize, 4);
}

// ===== 初始化SD卡 =====
bool initSDCard() {
  SPI.begin(SD_SCK_PIN, SD_MISO_PIN, SD_MOSI_PIN, SD_CS_PIN);
  if (!SD.begin(SD_CS_PIN)) {
    Serial.println("SD卡初始化失败!");
    return false;
  }
  uint64_t cardSize = SD.cardSize() / (1024 * 1024);
  Serial.printf("SD卡就绪，容量: %lluMB\n", cardSize);
  return true;
}

// ===== 检查存储空间 =====
bool checkStorageSpace() {
  if (!sdCardPowered) {
    powerSD(true);
  }
  
  uint64_t freeSpace = SD.totalBytes() - SD.usedBytes();
  if (freeSpace < MIN_RECORDING_SPACE) {
    Serial.printf("存储空间不足! 剩余: %.2fMB\n", freeSpace / (1024.0 * 1024));
    return false;
  }
  return true;
}

// ===== 生成文件名 =====
String generateFilename() {
  static uint32_t fileIndex = 0;
  char filename[30];
  sprintf(filename, "/audio_%lu_%u.wav", millis() / 1000, fileIndex++);
  return String(filename);
}

// ===== 缓存音频数据 =====
void cacheAudioData(int16_t* data, int samples) {
  int bytesToWrite = samples * sizeof(int16_t);
  
  if (cacheIndex + bytesToWrite > AUDIO_CACHE_SIZE) {
    // 缓存满了，写入文件
    if (audioFile) {
      audioFile.write(audioCache, cacheIndex);
    }
    cacheIndex = 0;
  }
  
  memcpy(audioCache + cacheIndex, data, bytesToWrite);
  cacheIndex += bytesToWrite;
}

// ===== 刷新缓存到文件 =====
void flushAudioCache() {
  if (cacheIndex > 0 && audioFile) {
    audioFile.write(audioCache, cacheIndex);
    cacheIndex = 0;
  }
}

// ===== 开始录音（优化版） =====
void startRecording() {
  if (!checkStorageSpace()) return;
  
  powerSD(true);  // 确保SD卡供电
  setCPUFrequency(80);  // 提高频率以处理录音
  
  // 检查是否在冷却期内，如果是，则追加到上一个文件
  if (millis() - lastRecordingStopTime < RECORDING_COOLDOWN_PERIOD && lastFilename != "") {
    audioFile = SD.open(lastFilename.c_str(), FILE_APPEND);
    Serial.printf("在冷却期内，追加到文件: %s\n", lastFilename.c_str());
  } else {
    // 否则，创建新文件
    String filename = generateFilename();
    int tryCount = 0;
    while (SD.exists(filename.c_str()) && tryCount < 1000) {
      delay(2);
      filename = generateFilename();
      tryCount++;
    }
    audioFile = SD.open(filename.c_str(), FILE_WRITE);
    if (audioFile) {
      writeWavHeader(audioFile);
      lastFilename = filename; // 记录新文件名
      Serial.printf("开始录音 (新文件): %s\n", filename.c_str());
    }
  }

  if (!audioFile) {
    Serial.println("创建文件失败!");
    return;
  }
  isRecording = true;
  recordStartTime = millis();
  cacheIndex = 0;
}

// ===== 停止录音（优化版） =====
void stopRecording() {
  if (!isRecording) return;
  
  flushAudioCache();  // 刷新缓存
  
  if (audioFile) {
    updateWavHeader(audioFile);
    audioFile.close();
  }
  
  isRecording = false;
  lastRecordingStopTime = millis(); // 记录停止时间
  Serial.println("停止录音");
  
  // 录音结束后可以降低功耗
  setCPUFrequency(80); // 修正：应恢复到80MHz以进行下一次监听，40MHz不足以处理音频。
}

// ===== HTTP 上传文件（优化版） =====
void uploadFile(const char* filename) {
  // 优化：移除独立的WiFi连接/断开逻辑，由调用者（如uploadAllFiles）管理
  powerSD(true);  // 确保SD卡供电
  setCPUFrequency(240);  // 提高频率以快速上传
  
  String path = String("/") + filename;
  File file = SD.open(path.c_str(), FILE_READ);
  if (!file) {
    Serial.printf("文件打开失败: %s\n", filename);
    return;
  }

  if(file.size() == 0) {
    Serial.printf("文件 %s 为空，跳过上传并直接删除。\n", filename);
    file.close();
    SD.remove(path.c_str());
    return;
  }

  Serial.printf("准备上传文件: %s, 大小: %d bytes\n", filename, file.size());

  HTTPClient http;
  http.begin(serverUrl);
  http.setTimeout(90000);
  http.addHeader("Content-Type", "application/octet-stream");
  
  int httpResponseCode = http.sendRequest("POST", &file, file.size());
  file.close();

  if (httpResponseCode == 200) {
    Serial.printf("文件 %s 上传成功，准备删除...\n", filename);
    if (SD.remove(path.c_str())) {
      Serial.printf("文件 %s 已从SD卡删除。\n", filename);
    } else {
      Serial.printf("从SD卡删除文件 %s 失败！\n", filename);
    }
  } else {
    Serial.printf("文件 %s 上传失败！将保留在SD卡中。错误: %s\n", filename, http.errorToString(httpResponseCode).c_str());
  }
  http.end();
}

// ===== 遍历并上传所有文件（优化版） =====
void uploadAllFiles() {
  if (!connectWiFiSmart()) {
    Serial.println("无网络连接，跳过上传");
    return;
  }
  
  Serial.println("正在扫描SD卡根目录...");
  powerSD(true);
  
  File root = SD.open("/");
  if(!root){
    Serial.println("打开根目录失败！");
    disconnectWiFi(); // 在异常退出时也要断开WiFi
    return;
  }

  while(true){
    File file = root.openNextFile();
    if (!file) {
      break;
    }

    if(file.isDirectory()){
      Serial.printf("跳过目录: %s\n", file.name());
      continue;
    }
    uploadFile(file.name());
    file.close();
  }
  root.close();
  
  // 上传完成后断开WiFi
  disconnectWiFi();
}

// ===== 同步网络时间（优化版） =====
void syncTime() {
  if (!connectWiFiSmart()) return;
  
  configTime(8 * 3600, 0, "pool.ntp.org", "time.nist.gov");
  Serial.println("等待时间同步...");
  time_t now = time(nullptr);
  unsigned long start = millis();
  while (now < 8 * 3600 * 2 && millis() - start < 10000) {
    delay(500);
    Serial.print(".");
    now = time(nullptr);
  }
  if (now > 8 * 3600 * 2) Serial.println("\n时间同步完成");
  else Serial.println("\n时间同步超时");
  
  // 同步完成后可以断开WiFi
  disconnectWiFi();
}

// ===== 检查定时上传时间（优化版） =====
void checkUploadTime() {
  static bool uploadedTodayAt12 = false;
  static bool uploadedTodayAt20 = false;
  static unsigned long lastCheck = 0;

  if (millis() - lastCheck < 600000) {  // 10分钟检查一次
    return;
  }
  lastCheck = millis();

  // 需要联网检查时间
  if (!connectWiFiSmart()) {
    return;
  }

  time_t now = time(nullptr);
  struct tm timeinfo;
  localtime_r(&now, &timeinfo);
  int currentHour = timeinfo.tm_hour;

  if (currentHour == 0) {
    uploadedTodayAt12 = false;
    uploadedTodayAt20 = false;
  }

  if (currentHour == 12 && !uploadedTodayAt12) {
    Serial.println("触发12:00定时上传...");
    uploadAllFiles();
    uploadedTodayAt12 = true;
  }

  if (currentHour == 20 && !uploadedTodayAt20) {
    Serial.println("触发20:00定时上传...");
    uploadAllFiles();
    uploadedTodayAt20 = true;
  }
  
  // 检查完成后断开WiFi
  disconnectWiFi();
}

// ===== 电池电量检查 =====
void checkBatteryLevel() {
  static unsigned long lastBatteryCheck = 0;
  if (millis() - lastBatteryCheck < BATTERY_CHECK_INTERVAL) {
    return;
  }
  lastBatteryCheck = millis();
  
  // 读取电池电压（需要根据硬件调整）
  // 修正：XIAO ESP32S3的电池检测引脚是GPIO2，即Arduino的A1
  int batteryReading = analogRead(A1);
  float voltage = (batteryReading * 3.3 * 2) / 4095.0; // XIAO板载分压电阻为100K+100K，需要乘以2
  
  Serial.printf("电池电压: %.2fV\n", voltage);
  
  // 增加一个判断，如果电压读数非常低（< 1.0V），
  // 则很可能没有连接电池，而是通过USB供电。
  // 在这种情况下，不应因"低电量"而进入睡眠。
  if (voltage < 1.0) {
    Serial.println("未检测到电池或电压极低，假设为USB供电，跳过低电量检查。");
    return;
  }

  // 低电量时进入极省电模式
  if (voltage < 3.4) {
    Serial.println("电池电量低，进入极省电模式");
    enterSleepMode(POWER_DEEP_SLEEP);
  }
}

// ===== 主程序 =====
void setup() {
  Serial.begin(115200);
  delay(1000);
  Serial.println("=== XIAO ESP32S3 低功耗音频录制系统 ===");

  // 配置电源管理
  setupPowerManagement();

  // 检查PSRAM
  if(psramFound()){
    Serial.printf("PSRAM 已启用! 大小: %.2fMB\n", ESP.getPsramSize() / 1024.0 / 1024.0);
  } else {
    Serial.println("警告: 未检测到PSRAM");
  }

  // 初始化硬件
  if (!initSDCard()) {
    Serial.println("SD卡初始化失败，程序停止");
    return;
  }
  
  setupI2S();
  
  // 初始连接WiFi同步时间
  if (connectWiFiSmart()) {
    syncTime();
  }
  
  lastActivityTime = millis();
  Serial.println("系统就绪，开始监听...");
}

void loop() {
  lastActivityTime = millis();
  
  // 检查是否需要防止过长录音
  if (isRecording && millis() - recordStartTime > MAX_CONTINUOUS_RECORDING) {
    Serial.println("录音时间过长，强制停止");
    stopRecording();
  }
  
  // 采集音频数据
  int samples = SAMPLE_BUFFER_SIZE;
  for (int i = 0; i < samples; i++) {
    audioBuffer[i] = I2S.read();
  }

  // 计算DC偏移
  int64_t sum = 0;
  for (int i = 0; i < samples; i++) {
    sum += audioBuffer[i];
  }
  int64_t dcOffset = sum / samples;

  // 音量检测
  int32_t volumeSum = 0;
  for (int i = 0; i < samples; i++) {
    int32_t centered = (int32_t)audioBuffer[i] - dcOffset;
    volumeSum += abs(centered);
  }
  int avgVolume = volumeSum / samples;

  // FFT分析（降低频率）
  static int frameCount = 0;
  bool voiceDetectedThisFrame = false;
  if (++frameCount >= 32) {  // 降低FFT频率
    frameCount = 0;
    for (int i = 0; i < 64 && i < samples; i++) {
      vReal[i] = (double)audioBuffer[i] - (double)dcOffset;
      vImag[i] = 0;
    }
    FFT.windowing(FFT_WIN_TYP_HAMMING, FFT_FORWARD);
    FFT.compute(FFT_FORWARD);
    FFT.complexToMagnitude();
    
    float maxMagnitude = 0;
    for (int i = 2; i < 16; i++) {  // 人声频率范围
      if (vReal[i] > maxMagnitude) maxMagnitude = vReal[i];
    }
    
    if (maxMagnitude > FFT_THRESHOLD) {
      voiceDetectedThisFrame = true;
    }
  }

  if (avgVolume > VOLUME_THRESHOLD) {
    voiceDetectedThisFrame = true;
  }

  // 人声检测逻辑
  if (voiceDetectedThisFrame) {
    if (!isVoiceDetected) {
      historicStartTime = millis();
      isVoiceDetected = true;
    }
    lastVoiceTime = millis();
  } else {
    isVoiceDetected = false;
  }

  // 录音控制逻辑
  if (isVoiceDetected && !isRecording && (millis() - historicStartTime >= MIN_VOICE_DURATION)) {
    startRecording();
  } else if (isRecording) {
    // 音频增益和缓存
    for (int i = 0; i < samples; i++) {
      audioBuffer[i] = constrain(audioBuffer[i] * 2, -32768, 32767);
    }
    cacheAudioData(audioBuffer, samples);
    
    if (millis() - lastVoiceTime > SILENCE_TIMEOUT) {
      stopRecording();
    }
  }

  // 定期检查
  static unsigned long lastPeriodicCheck = 0;
  if (millis() - lastPeriodicCheck > 60000) {  // 修正：延长检查周期为60秒，减少不必要的WiFi连接尝试
    checkUploadTime();
    checkBatteryLevel();
    lastPeriodicCheck = millis();
  }

  // 检查睡眠条件
  checkSleepConditions();
  
  // 短暂延迟以降低CPU负载
  delay(5);
}
```

## 4. 语音提取脚本 (yuyin_tq提取.py)

这是一个使用Whisper模型进行语音识别的Python脚本。它会遍历指定目录中的音频文件，使用Whisper进行语音转文本，并将结果整理到一个文本文件中。

```python
import os
import subprocess
from datetime import datetime

# 设置存放音频的文件夹路径
AUDIO_DIR = "one"
today_str = datetime.today().strftime("%Y%m%d")
OUTPUT_TEXT = f"text_info_{today_str}.txt"
WHISPER_MODEL = "medium" # Whisper 模型选择（根据电脑性能选择：base / small / medium / large）
LANG = "Chinese"  # Whisper识别的语言，可设置为 "Chinese"、"English"，或设为 None 让其自动判断

# 获取所有音频文件，按文件名排序
audio_files = sorted([
    f for f in os.listdir(AUDIO_DIR)
    if f.lower().endswith((".mp3", ".wav", ".m4a", ".flac"))
])

with open(OUTPUT_TEXT, "w", encoding="utf-8") as out_file:
    for idx, audio_file in enumerate(audio_files, start=1):
        audio_path = os.path.join(AUDIO_DIR, audio_file)
        print(f"[{idx}/{len(audio_files)}] 正在识别 {audio_file} ...")

        # Whisper 会自动生成 {filename}.txt
        cmd = [
            "whisper", audio_path,
            "--model", WHISPER_MODEL,
            "--output_format", "txt",
            "--language", LANG,
            "--fp16", "False",
            "--temperature", "0.3",
            "--output_dir", "."  # 输出到当前目录
        ]

        # 执行 whisper 命令
        subprocess.run(cmd, check=True)

        # 读取识别后的文本文件
        txt_filename = os.path.splitext(audio_file)[0] + ".txt"
        if os.path.exists(txt_filename):
            with open(txt_filename, "r", encoding="utf-8") as f:
                content = f.read().strip()
                out_file.write(content + "\n\n")
            os.remove(txt_filename)  # 删除中间文件
        else:
            print(f"⚠️ 没找到识别结果：{txt_filename}")

    # 添加整理时间戳
    today = datetime.today().strftime("%Y-%m-%d")
    out_file.write(f"\n以上信息由 {today} 日整理。\n")

print("✅ 所有音频识别完成，结果保存在:", OUTPUT_TEXT)
```

