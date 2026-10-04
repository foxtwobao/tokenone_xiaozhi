# ESP32-S3-CAM 面包板 Wi-Fi 板型

板级实现基于 `bread-compact-wifi-lcd`，引脚以本目录 `config.h` 为准。
历史说明使用 OV2640；实际模组应通过启动日志中的传感器 ID 确认，不能只根据厂商
源码注释或固件文件名判断。

现有发布变体 `bread-compact-wifi-s3cam` 默认使用 ST7789 240x320、
RGB565 VGA 相机采集，不启用水平镜像或垂直翻转。项目默认配置为 16 MB Flash、
`partitions/v2/16m.csv` 分区表和 80 MHz Octal PSRAM；这些是构建配置，
烧录前须确认实际核心板满足要求。

## 构建

使用 ESP-IDF 6.1，最低支持 6.0.1，不支持 5.x。在项目根目录执行：

```bash
source /path/to/esp-idf/export.sh
idf.py --version
python3 scripts/build.py bread-compact-wifi-s3cam --name bread-compact-wifi-s3cam
```

规范构建入口会选择目标芯片、板型及发布身份，生成 `sdkconfig` 并构建合并固件。
不要复制厂商旧版 `sdkconfig` 或直接修改生成文件。若实际硬件引脚不同，
按 `docs/custom-board.md` 新增板型或具有独立 OTA 身份的发布变体。

相机方向通过构建选项 `camera_hmirror`、`camera_vflip` 配置，
由公共相机实现应用；方向和分辨率均须用实机画面验证。

## 引脚与烧录接口

| 功能 | GPIO |
| --- | --- |
| 麦克风 WS / SCK / DIN | 1 / 2 / 42 |
| 扬声器 DOUT / BCLK / LRCK | 39 / 40 / 41 |
| 显示 MOSI / CLK / DC / RST / CS / BL | 20 / 19 / 47 / 21 / 45 / 38 |
| 相机 D0…D7 | 11 / 9 / 8 / 10 / 12 / 18 / 17 / 16 |
| 相机 SIOD / SIOC / VSYNC / HREF / XCLK / PCLK | 4 / 5 / 6 / 7 / 15 / 13 |
| BOOT 按键 / 板载 LED | 0 / 48 |

GPIO19/20 用于显示 SPI，同时也是芯片原生 USB 引脚。按核心板接线选择烧录和
监控接口；显示工作时使用独立 USB 转串口接口，不将原生 USB 与显示同时使用。

默认使用 simplex I2S。`config.h` 中的备用 duplex I2S 引脚 4/5/6/7 与相机冲突，
不能直接切换后继续使用相机。

## V2.0 扩展板适配依据

本地 V2.0 扩展板原理图与本板型的音频、显示和相机接口引脚对应，
但尚需核对实际 PCB 版本。资料位置和逐项核查记录见项目根目录
`HANDOFF_CAM_PORT.md`。

- 原理图 P6 标注 ST7789 240x240 / ST7735S 128x160，与现有默认 240x320 不同；
  实际装屏型号及分辨率仍需确认。
- 图纸上的 GPIO3 为 TP4054 充电状态，GPIO14 为电池分压采样。本板型尚未接入
  电量/充电检测；不得启用同脚 `LAMP_GPIO` 测试宏作为灯控制。
- 图纸未显示 GPIO48 控制电源开关，不能照搬厂商拉低 GPIO48 的关机逻辑。
- 厂商 ADC 原始值表、画面翻转和睡眠参数均未作为本项目硬件实测结果。

## 实机验证

烧录后保存完整启动日志，核对 Flash/PSRAM、显示配置和相机传感器 ID。
显示配置日志描述固件设置，不能作为屏幕型号的自动检测结果。

本板级日志标签为 `CompactWifiBoardS3Cam`：

- `Display configured`：驱动、分辨率、偏移和显示方向配置。
- `Camera sensor`：传感器 PID/VER 和初始化后的镜像/翻转状态。
- `Camera capture`：传感器报告的采集尺寸、像素格式和 XCLK。
- `Camera sensor unavailable`：相机初始化失败；结合前面的驱动错误定位。

依次验证显示范围与颜色、BOOT 配网/对话切换、录音与播放、唤醒及打断、
相机画面方向、连续拍照和拍照与语音并行。电量、充电及省电功能另行验证；
编译成功不等于硬件验收通过。
