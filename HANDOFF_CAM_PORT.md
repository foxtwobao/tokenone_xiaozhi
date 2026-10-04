# CAM 开发交接：以 tokenone_xiaozhi 当前项目为基线

更新日期：2026-10-04。用户已确认：基线以本项目为准。

## 目标

在 `/Users/hulei/Projects/tokenone_xiaozhi` 当前项目上，为手头的面包板 ESP32-S3-CAM 扩展板补齐本地固件需要的功能。本项目是唯一开发基线；官方上游和厂商 2.2.6 工程仅作为参考，不切回官方 v2.5.0，也不以旧工程覆盖本项目。尽量把设备差异限制在板级目录和板级配置中，降低以后合并上游更新的成本。

已完成代码及图纸盘点，并按用户指令建立 `codex/cam-v2-port` 开发分支。
98 项主机测试通过，ESP-IDF 6.1 原版 CAM 基线及第一批启动诊断补丁均已构建成功。
已读取实机芯片信息和现有固件启动日志，尚未烧录或验收本分支固件；
接手后先复核 Git 状态和提交，保留用户后续改动。

## 用户确认的目标硬件（2026-10-04）

- 摄像头：**OV3660**。用户提供的实物型号已通过现有固件启动日志交叉确认：
  `PID=0x3660`，驱动识别为 OV3660。当前构建已启用 `CONFIG_OV3660_SUPPORT=y`，
  驱动会自动探测，不需要因型号确认而替换公共相机实现。
- 屏幕：**2 寸 240x320**。分辨率与现有 CAM 变体一致，保留当前 ST7789
  配置；用户尚未确认控制器和 IPS/非 IPS，仍须验证显示范围、颜色及方向。
- 相机使用 RGB565 VGA、水平镜像关闭；实测画面上下颠倒，已将当前 CAM
  变体的垂直翻转设为开启，待刷写后复测。
- 核心板已读到 **ESP32-S3 revision v0.2、16 MB Flash、8 MB PSRAM**。
  PCB 版本/丝印、供电控制及充电/电量电路仍待核对。

## 源码基线与参考位置

- 开发目录：`/Users/hulei/Projects/tokenone_xiaozhi/`。
- 核查时分支：`main`；提交：`0d576d3d4c049c6f55eaf879725dc23e516511b4`。该提交是本次盘点快照，不是要求后续重置到此提交。
- 本项目 `CMakeLists.txt` 标记版本：`2.5.1`；不能据此将当前检出称为官方 v2.5.0 或已验证的官方 v2.5.1 tag。
- Git remote：`https://github.com/foxtwobao/tokenone_xiaozhi.git`。盘点时已跟踪文件无修改，`HANDOFF_CAM_PORT.md` 为未跟踪文件。
- SDK：本项目要求 ESP-IDF **>= 6.0.1**，优先 **6.1**，不支持 5.x；以本项目 `AGENTS.md`、`README.md`、`main/idf_component.yml` 和 CI 为准。盘点时当前 shell 尚未加载 `idf.py`，不代表本机未安装 SDK。
- 本地厂商 2.2.6 解压工程：`/Users/hulei/Projects/xiaozhi/src/xiaozhi-esp32-main226/xiaozhi-esp32-main/`
- 厂商工程版本：`2.2.6`，`sdkconfig` 记录 ESP-IDF `5.5.3`。
- 厂商 CAM 板级目录：`main/boards/bread-compact-wifi-s3cam/`
- 官方仓库：<https://github.com/78/xiaozhi-esp32>

本项目已经包含 `bread-compact-wifi-s3cam` 板型，目标芯片为 `esp32s3`，现有发布变体同名，默认屏幕为 ST7789 240x320。重点是筛选厂商包相对本项目板级实现的必要差异。若实际硬件 IO 不同，必须按 `docs/custom-board.md` 新增有唯一身份的板型或发布变体，不能直接改现有板型引脚；核对 OTA 身份及完整构建链，每次构建只保留一个 `DECLARE_BOARD(...)`。

## 本地 2.2.6 CAM 源码和资料

当前已解压、可直接阅读的 v2.2.6 工程在：

`/Users/hulei/Projects/xiaozhi/src/xiaozhi-esp32-main226/xiaozhi-esp32-main/`

重点文件：

- 板级实现：`main/boards/bread-compact-wifi-s3cam/compact_wifi_board_s3cam.cc`
- 引脚、屏幕和相机配置：`main/boards/bread-compact-wifi-s3cam/config.h`
- 厂商板级说明：`main/boards/bread-compact-wifi-s3cam/README.md`
- 电池/充电检测：`main/boards/bread-compact-wifi-s3cam/power_manager.h`
- 项目版本：`CMakeLists.txt`
- 厂商构建配置：`sdkconfig`

这一工程来自较新的 CAM 扩展板教程资料包中的压缩源码：

`/Users/hulei/Projects/xiaozhi/refer/CAM智能扩展板新版本打字V2.0版本教程/固件跟源码/xiaozhi-esp32-main226固件源码.zip`

同一压缩包也在较早的面包板 DIY 资料包中重复存放：

`/Users/hulei/Projects/xiaozhi/refer/面包板DIY教程 ESP32-S3-CAM摄像头小智资料包/固件跟固件源码/xiaozhi-esp32-main226固件源码.zip`

两套资料的根目录如下，文件名包含空格和中文，查找时请使用完整路径：

- **当前 CAM 智能扩展板 V2.0 教程（优先参考）**：`/Users/hulei/Projects/xiaozhi/refer/CAM智能扩展板新版本打字V2.0版本教程/`
- **面包板 DIY ESP32-S3-CAM 资料包（较早/补充参考）**：`/Users/hulei/Projects/xiaozhi/refer/面包板DIY教程 ESP32-S3-CAM摄像头小智资料包/`

优先参考的 V2.0 教程中，硬件和操作资料包括：

- 板卡硬件资料：`CAM智能扩展板尺寸图.png`、`CAM智能扩展板介绍.jpg`、`ESP32S3-CAM-KZB-V2_0.pdf`
- macOS 烧录说明：`macOS烧录教程.md`
- 烧录/配网资料：`配网说明文档.docx`、`小智AI烧录固件视频.mp4`、`CAM扩展板套件安装视频.mp4`、`CAM扩展板聊天演示视频.mp4`
- 烧录界面参考：`软件烧录画面.png`、`2个接口都可以烧录，只是打开软件选择不一样.png`、`2次开发参数选择.png`
- 旧源码和固件：`固件跟源码/`，其中有 `V223_CAM固件源码.zip`、`xiaozhi-esp32-2.2.4_CAMDL固件源码.zip`、`xiaozhi-esp32-CAM203DL固件源码.rar`，以及 2.2.3/2.2.4/2.2.6 的 DL、NODL `.bin` 固件。

较早的面包板 DIY 资料包包含另一组接线参考：`CAM开发板引脚图片1.png`、`CAM开发接线图2.jpg`、`CAM开发板接线图2.jpg`，以及配网/烧录资料。其 `固件跟固件源码/` 目录也有上述多个版本的源码包和固件，另有 `merged-binaryOV2640.bin`。这些 `.bin` 是预编译固件，用于辨认历史版本或烧录对照，不是应复制进本项目的源码。

资料包里有多份不同屏幕尺寸、摄像头型号和 DL/NODL 固件。不要仅根据文件名判定用户当前板卡配置；应以实物 PCB 版本、相机传感器识别结果及屏幕实测为准。

## 已找到并核对的 V2.0 图纸

2026-10-04 已实际打开 PDF、提取文字并逐页渲染核对，同时查看了配套介绍图和尺寸图。文件保留在原资料目录，以下链接可直接定位；本交接文档已更新在本项目根目录。

- [ESP32S3-CAM-KZB-V2_0.pdf](/Users/hulei/Projects/xiaozhi/refer/CAM智能扩展板新版本打字V2.0版本教程/ESP32S3-CAM-KZB-V2_0.pdf)：共 2 页，第 1 页为扩展板原理图，第 2 页为空白。图框日期为 **2025-10-21**，源文件标注 `ESP32S3-CAM-KZB-V2_0.SchDoc`。
- [CAM智能扩展板介绍.jpg](/Users/hulei/Projects/xiaozhi/refer/CAM智能扩展板新版本打字V2.0版本教程/CAM智能扩展板介绍.jpg)：可见屏幕接口、麦克风、功放、喇叭接口、电池接口、机械开关和两个按键。
- [CAM智能扩展板尺寸图.png](/Users/hulei/Projects/xiaozhi/refer/CAM智能扩展板新版本打字V2.0版本教程/CAM智能扩展板尺寸图.png)：图示外形约 **64.5 x 37.6 mm**，排针间距 2.54 mm，两侧排针标注距离 25.4 mm，丝印可见 `S3-CAM-DevPro`。

### 原理图上的连接（尚非实物验收）

| 功能 | 图纸连接/标注 | 对开发的影响 |
| --- | --- | --- |
| 充电检测 | TP4054 的 CHRG 信号经 `CHGR` 网络接 GPIO3 | 与厂商传入 `PowerManager(GPIO_NUM_3)` 一致；实测有效电平、充满及无电池状态 |
| 电池采样 | `VCC_BAT` 经 R5 1.5 kΩ 和 R6 4.7 kΩ 分压，`ADC` 接 GPIO14 | 名义分压关系为 `V_ADC = V_BAT * 4.7 / (1.5 + 4.7)`；需实测阻值、校准和采样范围，不能直接照搬旧 ADC-电量表 |
| 麦克风 | WS=GPIO1、SCK=GPIO2、DIN=GPIO42 | 与当前板级 simplex I2S 配置一致 |
| 扬声器音频 | DOUT=GPIO39、BCLK=GPIO40、LRCK=GPIO41 | 与当前板级 simplex I2S 配置一致 |
| 屏幕接口 | CLK=GPIO19、MOSI=GPIO20、RST=GPIO21、DC=GPIO47、CS=GPIO45、BL=GPIO38 | 与当前板级屏幕引脚一致；需要结合实物确认烧录接口 |
| 按键 | 唤醒/打断/配网按键=GPIO0，预留按键=GPIO46，按下接地 | 当前固件使用 GPIO0；是否启用预留按键另行决定 |
| 相机接口网络 | D0..D7=GPIO11/9/8/10/12/18/17/16，SIOD=GPIO4、SIOC=GPIO5、VSYNC=GPIO6、HREF=GPIO7、XCLK=GPIO15、PCLK=GPIO13 | 与当前板级定义一致；该扩展板图没有确认实际相机传感器型号 |
| GPIO48 | 图中仅在核心板接口和扩展排针引出，未见接入电源开关控制 | 不能以此图支持厂商“拉低 GPIO48 即断电”的实现；核心板内部情况仍待核对 |

还需注意两个源码/图纸差异：

- 原理图 P6 标注 **ST7789 240x240 / ST7735S 128x160**，介绍图写“2.0寸 TFT 屏接口”，而当前及厂商固件默认 **ST7789 240x320**。用户现已确认实际屏幕为 **2 寸 240x320**，不按 P6 标注切换分辨率；控制器及显示效果仍待实测。
- 当前 `config.h` 仍有 `LAMP_GPIO=GPIO14` 测试宏，但本次检查的板级 `.cc` 未实例化 `LampController`。此图的 GPIO14 用于电池 ADC，后续不得未经核对启用同脚灯控制。

**确认边界：** 已找到并核对的是 V2.0 资料包中的扩展板原理图，不是 ESP32-S3-CAM 核心板完整原理图或 PCB/Gerber 工程；尚未核实用户手头实物的版本/丝印。只有实物与此图对应后，才能把上述连接当作目标硬件依据。

## 已确认的本地差异

2026-10-04 已将本项目与本地厂商 2.2.6 的同路径文件逐一比较。CAM 板级文件为 `main/boards/bread-compact-wifi-s3cam/compact_wifi_board_s3cam.cc`，已确认：

- 本项目使用 `FRAMESIZE_VGA`；厂商使用 `FRAMESIZE_QVGA`，另调用 `SetHMirror(false)`、`SetVFlip(1)`。本项目已依据实测将当前 CAM 变体设为 `camera_vflip=true`，水平镜像保持关闭。
- 本项目已提供 `camera_hmirror`、`camera_vflip` 构建选项。`scripts/build.py` 会生成对应 Kconfig 设置，公共相机实现负责应用；方向调整优先复用这条链路。
- 厂商添加 `PowerManager`、电量/充电检测、背光休眠和深睡眠；本项目的 CAM 板级尚未接入这些功能，但已有公共 `PowerSaveTimer` 可供复用。
- 两份 `config.h` 的差异只有空行，源码中的基础引脚定义一致；实物 PCB 匹配仍待验证。
- 板级文件另有 include 和格式差异，不整体移植这些差异。

厂商工程内相关板级配置和说明在（`power_manager.h` 目前仅在厂商 CAM 目录中）：

- `main/boards/bread-compact-wifi-s3cam/config.h`
- `main/boards/bread-compact-wifi-s3cam/README.md`
- `main/boards/bread-compact-wifi-s3cam/power_manager.h`

厂商 `sdkconfig` 选择了 `BREAD_COMPACT_WIFI_CAM`、ST7789 240x320、16MB Flash、`partitions/v2/16m.csv`、Octal PSRAM、AFE 唤醒词、唤醒词音频上送和音频处理；OTA URL 是 `https://api.tenclass.net/xiaozhi/ota/`。这些是厂商构建记录，不是已确认的实物参数，也不应直接覆盖本项目配置。

### 厂商电源代码已发现的问题

- `InitializeLcdDisplay()` 创建的是局部 `panel`，关机回调却使用未赋值、仍为 `nullptr` 的成员 `panel_`。
- GPIO48 在 `config.h` 中定义为板载 LED，厂商同时用它做所谓关机控制。须核对原理图、实际接线和目标芯片对相关 GPIO/RTC API 的支持。
- `PowerManager` 在 ADC 初始化完成前启动定时器；同时板级先初始化电源管理，再初始化其回调会访问的省电定时器。移植时调整初始化和启用顺序。
- 厂商使用 `ADC_UNIT_2` / `ADC_CHANNEL_3` 和硬编码 ADC-电量表；图纸已经给出 GPIO14 和分压电阻，实际焊装、电压校准、目标 SDK 通道映射及 Wi-Fi 并行采样行为仍需要验证。
- 电源回调来自 `ESP_TIMER_TASK`。应用及界面变化按本项目线程规则调度，使用 `Application::Schedule()` 或事件位，并检查显示/背光等可选能力。

## 型号依据及尚待验证的硬件细节

- 厂商 README 写的是 OV2640，但板级 `.cc` 中翻转注释写着“OV3360”；已有固件文件名也出现 OV3660。用户确认及现有固件传感器 ID 均为 OV3660，方向、采集分辨率和画质仍待验证。
- 用户已确认 2 寸 240x320，与当前构建分辨率一致；不再将屏幕尺寸/分辨率作为待询问信息。控制器、IPS/非 IPS 和实际显示效果尚待确认。
- 相机/屏幕配置要与手头 PCB 逐项核对，特别是充电、电量、LED/电源控制，以及 GPIO19/20 的显示连接与烧录接口选择。
- 厂商包是 IDF 5.5.3，本项目要求 IDF >= 6.0.1。不要拷入旧工程的 `sdkconfig`、构建目录、依赖锁文件或 `managed_components/`。

## 推荐工作顺序

1. **项目原版构建基线**：复核当前提交和工作区，加载 ESP-IDF 6.1，使用规范构建入口构建现有 CAM 变体。记录 SDK/依赖版本、固件大小、Flash/分区及烧录参数。构建脚本会改变本地 `sdkconfig` 和 `build/` 状态，先保留需要的旧构建资料，不手改生成文件。
2. **硬件核对表**：依据 V2.0 图纸、实物 PCB 和启动日志，确认屏幕、相机 ID、Flash/PSRAM、电池/充电和供电控制。明确已确认项、待实测项和配置依据；如新增板型/变体，更新相关 `config.json`、构建选择、Kconfig、CMake 和板级文档，检查唯一 OTA 身份。
3. **相机与基础交互**：先验证屏幕、按键、音频及相机；翻转优先用现有构建配置；比较 VGA/QVGA 的图像、耗时和内存后决定是否改动。验证连续拍照及拍照与语音并行。
4. **电量/充电独立接入**：核对采样和充电电路后实现最小板级适配，修正上述初始化/句柄问题，校准读数并接入 `Board::GetBatteryLevel()`；验证无电池、充电插拔、采样失败和 Wi-Fi/音频并行状态。
5. **息屏优先，深睡眠后置**：复用 `PowerSaveTimer` 和应用休眠条件，先实现空闲调暗、交互恢复，确认聆听/播放/拍照/重连不误休眠。明确供电控制、唤醒来源和实际需求后再加入深睡眠，不照搬厂商超时参数。
6. **自定义 server 独立交付**：确认本项目 OTA 配置、认证、WebSocket 或 MQTT/UDP，以及视觉接口；分别测试配置获取、音频上下行、图像请求和断线重连，不将换 URL 等同于完成接入。
7. **分功能验证与交付**：相机、电量、省电分别组织小补丁和提交；仅格式化触及的 C/C++ 文件。记录构建结果、实机结果和未验证项。

### 构建与检查入口

先加载本机实际的 ESP-IDF 6.1 环境，再执行：

```sh
idf.py --version
python3 scripts/build.py --list-boards
python3 -m unittest discover -s scripts/tests -v
python3 scripts/build.py bread-compact-wifi-s3cam --name bread-compact-wifi-s3cam
```

板级修改构建受影响变体；若涉及公共层、Kconfig、CMake 或协议，则补充代表性芯片/网络路径验证，共享协议修改覆盖 WebSocket 与 MQTT/UDP。成功构建不能替代实机验证。

### 开工记录（2026-10-04）

- 从当前 `main` 创建并切换到 `codex/cam-v2-port`，未重置或覆盖已有工作。
- 本机原有 SDK 为 `/Users/hulei/esp/esp-idf-v5.5.3`。已另行安装官方 `v6.1`
  到 `/Users/hulei/esp/esp-idf-v6.1`，保留原 SDK。`idf.py --version` 为
  `ESP-IDF v6.1`，SDK 提交为 `fff9895c82d744c7237be8847347bdd1b07c6643`；
  Python 为 3.14.8，Xtensa 工具链为 `esp-15.2.0_20251204`。
- 主机测试：`python3 -m unittest discover -s scripts/tests -v`，**98 项通过**。
  首轮 6 个错误由 CI 测试子进程找不到 `python` 命令导致；在已有 Python 虚拟环境的
  `bin` 加入本次命令 PATH 后通过，没有为此修改项目代码。
- 主机测试日志：`/tmp/tokenone-cam-host-tests.log`；SDK 安装日志：
  `/tmp/tokenone-cam-idf-install.log`。
- 已补齐 CAM 板级 README 的规范构建入口、引脚/USB 接口说明和实机核对项。
- 已在 CAM 板级初始化中增加屏幕配置、相机 PID/VER、镜像/翻转及实际采集尺寸日志。
  屏幕日志仅报告构建配置；相机初始化失败时检查空指针，尺寸查表前检查枚举范围。
  未更改引脚、OTA 身份、相机分辨率或方向，也未接入未经实测的电量/关机逻辑。
- 修改的 C++ 文件已使用项目 `.clang-format` 格式化并通过
  `clang-format --dry-run -Werror`；`git diff --check` 通过。
- 首轮未发现连接的 ESP USB 串口设备；后续已成功连接，见下方实机读取记录。
- 用户说明设备已接 USB 且已有固件。扫描 macOS `/dev/cu.*` 及 USB 注册表，
  仅发现系统/蓝牙串口和 USB2.0 Hub，未发现 ESP 或 USB-UART。
  GPIO19/20 在当前板型用于显示，运行固件可能使原生 USB 无法枚举；
  外置 USB-UART 的枚举通常独立于 ESP 应用固件。需确认 USB 插口接线，
  用 BOOT+RESET 进入下载模式再扫描，或连接独立 UART0 接口。

### 实机读取记录（2026-10-04）

- 用户调整连接后，发现 `/dev/cu.usbserial-10`，USB 描述为 `USB Serial`，
  VID/PID 为 `1A86:7523`（CH340/CH341 系列 USB-UART）。此前未枚举不是
  “已有固件就默认断开”的必然结果；本次可以经 USB-UART 读取信息和运行日志。
- `esptool v5.4.0` 成功读取芯片及 Flash：ESP32-S3 (QFN56) revision v0.2，
  40 MHz 晶振，内置 8 MB PSRAM (`AP_3v3`)，Flash 为 16 MB，
  厂商 ID `0x68`、设备 ID `0x4018`；与当前 CAM 构建容量设置一致。
- 115200 波特率启动日志确认当前运行 `xiaozhi 2.2.6`，编译时间为
  `2026-04-29 12:13:09`，SDK 为 `v5.5.3-dirty`，板型
  `bread-compact-wifi-s3cam`。这是设备原有固件，不能作为本项目 IDF 6.1
  诊断固件的实机验证结果。
- 日志确认 8 MB PSRAM、80 MHz；相机 `PID=0x3660`，识别 OV3660，
  初始化及帧缓冲分配成功；Wi-Fi、MQTT 连接成功，唤醒引擎启动。
  屏幕控制器、实际显示范围/颜色/方向、音频及照片质量仍需实物验收。
- 观察到三条 `RTCIO number error`，分别来自 `rtc_gpio_init`、
  `rtc_gpio_set_direction` 和 `rtc_gpio_set_level`。括号中的数字是 SDK
  日志位置，不能解读成 GPIO 号；本地厂商源码在对应初始化中对 GPIO48
  调用了 RTC API，需进一步核对运行固件与源码是否一致。
- 观察到一次 `cam_hal: EV-EOF-OVF`。暂不判定为持续故障，后续通过连续拍照
  和拍照/语音并行验证其是否重现。
- 原始启动日志暂存 `/tmp/tokenone-cam-existing-boot.log`（含网络及设备标识，
  不提交仓库）。该阶段仅执行识别和串口读取；后续刷写结果见下文。

### 原固件备份（2026-10-04）

- 用户指示继续开工后，已完整备份设备 16 MB Flash 到：
  `/Users/hulei/Projects/xiaozhi/device-backups/2026-10-04-cam/original-flash-16mb.bin`。
  文件包含原固件、资源、NVS 和设备身份，不提交仓库；备份目录权限为 700，
  二进制备份权限为 600。
- 备份文件大小为 16,777,216 字节，设备整片 Flash MD5 与备份 MD5 一致：
  `f846de4409aaf4288b604870e3ff4a6b`。SHA-256：
  `9dd992ba6cc95bedf4966a4ea5ad745df24ae61628407d8b44c57fa250968e04`。
- 连续读取在 460800/230400 波特率下丢包，115200 下也偶有丢包。
  最终使用本地只读工具以 64 KB 分块、逐块 MD5 核验和重试；
  空白块经设备 MD5 确认为全 `0xff`，最终再核对整片 MD5。
  工具和 `backup-manifest.json` 保存在同一备份目录。
- 原分区表与新构建分区表逐字节一致。烧录采用 `flasher_args.json` 中的
  独立分区文件，不写入 NVS `0x9000..0xcfff` 和 PHY `0xf000..0xffff`，
  不用会跨越这些区域的合并固件覆盖配网及身份数据。
- 回退时可在加载 ESP-IDF 环境后执行下列命令；该操作恢复备份时的完整状态：

```sh
python -m esptool --chip esp32s3 --port /dev/cu.usbserial-10 --baud 115200 \
  write-flash --flash-mode keep --flash-freq keep --flash-size keep \
  0x0 /Users/hulei/Projects/xiaozhi/device-backups/2026-10-04-cam/original-flash-16mb.bin
```

### 本分支实机刷写与启动（2026-10-04）

- 备份完成后，按 `build/flasher_args.json` 分区写入本分支固件：
  bootloader `0x0`、分区表 `0x8000`、OTA 初始数据 `0xd000`、
  应用 `0x20000`、资源 `0x800000`。NVS 和 PHY 区域未写入。
- esptool 对每个写入文件均报告 `Hash of data verified`；刷写后设备正常复位。
  启动日志暂存 `/tmp/tokenone-cam-new-boot.log`，不提交仓库。
- 新固件确认运行 `xiaozhi 2.5.1`、`ESP-IDF v6.1`，板级初始化打印：
  `ST7789 240x320`，偏移 `0,0`，镜像和交换坐标关闭。
- 相机再次确认 `PID=0x3660`（OV3660），板级日志报告 `hmirror=0`、
  `vflip=0`、采集尺寸 **640x480**、RGB565、XCLK 20 MHz；相机初始化成功。
  后续实拍发现画面上下颠倒，已将当前变体的 `camera_vflip` 改为 `true`，
  需要重新刷写后复测。
- 显示 LVGL、2 MB PSRAM 图像缓存、背光 75%、simplex 音频通道和
  `self.camera.take_photo` MCP 工具均初始化成功。该串口日志证明驱动初始化，
  不能替代肉眼确认屏幕颜色、范围和方向，也不能替代实际拍照上传质量测试。
- 设备连接 Wi-Fi，MQTT 连接并完成激活，状态从 `starting` 进入 `idle`。
  本次新固件日志未出现旧固件的 `RTCIO number error`，也未出现
  `cam_hal: EV-EOF-OVF`。
- 当前仍待做：肉眼屏幕验收、按键/录音/播放、实际拍照与连续拍照、
  拍照和语音并行，以及电池/充电电路验证。刷写和启动验证已完成，
  不能把这些未做的项目标记为通过。

### 相机方向修正（2026-10-04）

- 实机拍照确认画面上下颠倒；根因对应模组安装方向，已在
  `main/boards/bread-compact-wifi-s3cam/config.json` 将
  `camera_vflip` 改为 `true`，`camera_hmirror` 仍为 `false`。
- 已重新构建并仅刷写应用分区；启动日志确认 `Camera sensor` 报告
  `hmirror=0, vflip=1`，相机和联网初始化正常。用户已于 2026-10-04
  完成实拍复测，确认上下方向已经恢复。
- 方向修正后的本地固件：应用 `build/xiaozhi.bin`，合并固件
  `build/merged-binary.bin`。应用 SHA-256：
  `b833eec544da150fa8ccfd3f863b75daff35148b42d51c78ad3bd69909fb2e86`；
  合并固件 SHA-256：
  `7ff408eaf109863da77aea1ec4dbf28761856f0e95e2ded90417bd131942da72`。

### 自建服务端接入（2026-10-04）

- 服务端项目位于 `coding:/home/hulei/projects/tokenone_xiaozhiserver`，
  智控台为 `http://192.168.10.38:8002`，OTA 为
  `http://192.168.10.38:8002/xiaozhi/ota/`，WebSocket 为
  `ws://192.168.10.38:8000/xiaozhi/v1/`。
- 设备原先保存的 `wifi:ota_url` 缺少末尾 `/`。该路径返回 HTTP 200
  但正文为资源不存在的错误 JSON，缺少协议配置，设备因而沿用旧 MQTT 配置。
- 经用户授权通过 USB 修正该 NVS 字符串及对应 CRC，仅重写 `0x9000`
  所在 4 KB 扇区；全 NVS 回读与预期结果完全一致，其他配置保留，未重新烧录应用。
- 修改前配置备份：
  `/Users/hulei/Projects/xiaozhi/device-backups/2026-10-04-cam/nvs-before-custom-ota-fix.bin`。
  修改清单及校验值保存在同目录 `custom-ota-fix-manifest.json`。
- 重启已取得自建 OTA 的 WebSocket 配置并完成激活。服务端日志确认设备
  `1c:29:04:31:16:ec` 的 WebSocket 握手、Opus 参数和五项设备工具注册
  （包括 `self.camera.take_photo`）；实际语音交互及串口回复已观察到。
  自建视觉接口实际拍照请求、断线重连和长时间对话仍需验证。
- 本次改动是设备持久配置；现有本地固件的编译默认 OTA 地址仍为原地址。
  保留 NVS 刷写应用会继续使用自建地址，清空 NVS 后需重新填写。

### 自建服务端响应延迟核查（2026-10-04）

- 用户反馈切换自建服务端后语音响应变慢。本次只读取服务端日志和实际模型配置，
  未修改固件、服务端代码或模型选择。
- 已观察的四轮对话，从 ASR 识别出文字到服务端发送第一段音频分别为约
  6、4、5、4 秒（日志时间精度为秒）；这是服务端阶段的延迟，
  不包括用户说话、静默判定及设备收到音频后的播放延迟。
- ASR 日志中的模型 forward 约为 0.16～0.28 秒。实际配置来自智控台，
  VAD 静默阈值为 700 ms，不能以源码默认配置的 200 ms 代替。
- 实际 LLM 为 `deepseek-v4.1-flash`，通过 `tokenone.work` 接口；
  实际 TTS 为 EdgeTTS，音色 `zh-CN-XiaoxiaoNeural`。虽然模型文本以流式方式
  进入 TTS 队列，当前 EdgeTTS 路径会收齐一段 MP3 后再转码发送，
  各短句也分别发起合成请求，存在首音等待和句间等待。
- 现有日志没有记录模型首个文本块和 TTS 请求开始时间，尚不能精确分摊
  首音等待中的模型、上游网络、合成和转码耗时。未做原服务端同步对照测试，
  也未验证调整参数或替换合成路径能减少多少延迟。

### 已验证的 CAM 构建结果

原版和诊断补丁均使用规范入口：

```sh
source /Users/hulei/esp/esp-idf-v6.1/export.sh
python3 scripts/build.py bread-compact-wifi-s3cam --name bread-compact-wifi-s3cam
```

关键依赖实际解析为 esp32-camera **2.1.8**、ESP-SR **2.4.7**、
LVGL **9.5.0**、esp_lvgl_port **2.9.0**。生成的依赖锁文件保持为构建产物，
没有手工修改或纳入移植源码。

| 产物 | 原版基线 | 诊断补丁 |
| --- | ---: | ---: |
| `xiaozhi.bin` | 2,787,008 字节 | 2,787,600 字节 |
| `generated_assets.bin` | 1,262,657 字节 | 1,262,657 字节 |
| `merged-binary.bin` | 9,651,265 字节 | 9,651,265 字节 |

应用 OTA 分区为 `0x3f0000`（4,128,768 字节），诊断补丁增加 592 字节，
仍有 1,341,168 字节（约 32%）余量。构建确认只导出一个 `create_board()`。

生成配置为 ESP32-S3、16 MB Flash、80 MHz Octal PSRAM、ST7789 240x320、
AFE 唤醒和音频处理，相机水平镜像关闭、垂直翻转开启（依据实测画面）。
生成的 `flasher_args.json` 指定烧录参数为 **DIO / 80 MHz / 16 MB**，
与 SDK 用于运行时初始化的 `CONFIG_ESPTOOLPY_FLASHMODE_QIO=y` 区分。
烧录布局为 bootloader `0x0`、分区表 `0x8000`、OTA 初始数据 `0xd000`、
应用 `0x20000`、资源 `0x800000`；合并固件从 `0x0` 写入。
实物芯片和存储容量已核对；这些构建参数仍不代表可以对任意 CAM 核心板烧录，
本分支固件的启动及硬件行为尚未实测。

- 原版日志：`/tmp/tokenone-cam-baseline-build.log`。
- 诊断补丁日志：`/tmp/tokenone-cam-diagnostics-build.log`。
- 原版产物、烧录参数及依赖锁文件的临时备份：`/tmp/tokenone-cam-baseline.IKIGI5/`。
- 原版应用 SHA-256：
  `45bbacf46c25da396047caefae3d24a66624097999670842c08240032ec68a34`。
- 原版合并固件 SHA-256：
  `fa3076340c6121275ebb4fddc5dafaafe7ce85475c8a92571a906f20f06e27c0`。
- 当前诊断固件：`/Users/hulei/Projects/tokenone_xiaozhi/build/merged-binary.bin`。

临时目录及 `build/` 会被清理或后续构建覆盖，持续交接应以源码和构建命令为准。
用户已确认 OV3660 和 2 寸 240x320，无须重复询问这两项。
USB-UART 接口、芯片/Flash/PSRAM 和现有固件启动日志已读取。
下一步保留可回退的现有固件备份，再开展本分支固件烧录及屏幕/音频/拍照实测；
同时确认实物 PCB 版本及电量/充电、供电控制电路。

## 验收清单

- 本项目当前基线的原版 CAM 配置可以干净构建，版本、依赖和构建参数有记录。
- 屏幕显示、触摸/按键（若有）、摄像头采集和画面方向均在目标硬件上验证。
- 唤醒和连续对话回归测试通过；如启用电源管理，确认它不会打断聆听或导致误入睡眠。
- 电量/充电读数和状态经过实测，空闲息屏可恢复；如启用深睡眠，必须验证唤醒路径。
- 自定义 server 单独验证 OTA 配置获取、认证、音频上下行和断线重连。
- 变更保持局部、独立提交，并注明未实机验证的部分。

## 给接手者的边界

以本项目和项目 `AGENTS.md` 为准，不回退至交接旧稿中的 v2.5.0。先复核当前状态，保留已有工作，不把整份厂商 2.2.6 合并进来。未经核实，不更改公共音频管线、唤醒算法、MQTT/WebSocket 协议实现，也不将旧 `sdkconfig`、`build/`、`managed_components/` 或依赖锁文件作为移植内容。
