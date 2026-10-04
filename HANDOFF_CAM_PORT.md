# CAM 开发交接：以 tokenone_xiaozhi 当前项目为基线

更新日期：2026-10-04。用户已确认：基线以本项目为准。

## 目标

在 `/Users/hulei/Projects/tokenone_xiaozhi` 当前项目上，为手头的面包板 ESP32-S3-CAM 扩展板补齐本地固件需要的功能。本项目是唯一开发基线；官方上游和厂商 2.2.6 工程仅作为参考，不切回官方 v2.5.0，也不以旧工程覆盖本项目。尽量把设备差异限制在板级目录和板级配置中，降低以后合并上游更新的成本。

已完成代码及图纸盘点，并按用户指令建立 `codex/cam-v2-port` 开发分支。
98 项主机测试通过，ESP-IDF 6.1 原版 CAM 基线及第一批启动诊断补丁均已构建成功。
尚未实机烧录或测试；接手后先复核 Git 状态和提交，保留用户后续改动。

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

- 原理图 P6 标注 **ST7789 240x240 / ST7735S 128x160**，介绍图写“2.0寸 TFT 屏接口”，而当前及厂商固件默认 **ST7789 240x320**。接口标注不等于实际装屏型号，分辨率以实物和显示测试为准。
- 当前 `config.h` 仍有 `LAMP_GPIO=GPIO14` 测试宏，但本次检查的板级 `.cc` 未实例化 `LampController`。此图的 GPIO14 用于电池 ADC，后续不得未经核对启用同脚灯控制。

**确认边界：** 已找到并核对的是 V2.0 资料包中的扩展板原理图，不是 ESP32-S3-CAM 核心板完整原理图或 PCB/Gerber 工程；尚未核实用户手头实物的版本/丝印。只有实物与此图对应后，才能把上述连接当作目标硬件依据。

## 已确认的本地差异

2026-10-04 已将本项目与本地厂商 2.2.6 的同路径文件逐一比较。CAM 板级文件为 `main/boards/bread-compact-wifi-s3cam/compact_wifi_board_s3cam.cc`，已确认：

- 本项目使用 `FRAMESIZE_VGA`；厂商使用 `FRAMESIZE_QVGA`，另调用 `SetHMirror(false)`、`SetVFlip(1)`。是否改分辨率和方向必须依据实测。
- 本项目已提供 `camera_hmirror`、`camera_vflip` 构建选项，当前 CAM 变体均为 `false`。`scripts/build.py` 会生成对应 Kconfig 设置，公共相机实现负责应用；方向调整优先复用这条链路。
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

## 需要先澄清的硬件细节

- 厂商 README 写的是 OV2640，但板级 `.cc` 中翻转注释写着“OV3360”；已有固件文件名也出现 OV3660。必须以实际摄像头模组/传感器 ID 和实机画面为准，核实传感器型号、方向、分辨率和画质，再决定是否移植参数。
- 本地源码配置为 ST7789 240x320，但 V2.0 原理图的屏幕接口标注为 ST7789 240x240 / ST7735S 128x160，资料目录也有 240x240 屏固件。以实物屏幕及实测为准，不能只按文件名或接口旁标注决定。
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
- 当前未发现连接的 ESP USB 串口设备，实机烧录与验收尚未执行。

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
AFE 唤醒和音频处理，相机水平镜像/垂直翻转均关闭。
生成的 `flasher_args.json` 指定烧录参数为 **DIO / 80 MHz / 16 MB**，
与 SDK 用于运行时初始化的 `CONFIG_ESPTOOLPY_FLASHMODE_QIO=y` 区分。
烧录布局为 bootloader `0x0`、分区表 `0x8000`、OTA 初始数据 `0xd000`、
应用 `0x20000`、资源 `0x800000`；合并固件从 `0x0` 写入。
这些参数尚未在用户实物上确认，不代表已经可以对任意 CAM 核心板烧录。

- 原版日志：`/tmp/tokenone-cam-baseline-build.log`。
- 诊断补丁日志：`/tmp/tokenone-cam-diagnostics-build.log`。
- 原版产物、烧录参数及依赖锁文件的临时备份：`/tmp/tokenone-cam-baseline.IKIGI5/`。
- 原版应用 SHA-256：
  `45bbacf46c25da396047caefae3d24a66624097999670842c08240032ec68a34`。
- 原版合并固件 SHA-256：
  `fa3076340c6121275ebb4fddc5dafaafe7ce85475c8a92571a906f20f06e27c0`。
- 当前诊断固件：`/Users/hulei/Projects/tokenone_xiaozhi/build/merged-binary.bin`。

临时目录及 `build/` 会被清理或后续构建覆盖，持续交接应以源码和构建命令为准。
下一步需要用户提供实物 PCB 版本、屏幕型号/分辨率及摄像头型号，或正反面照片；
连接独立 USB 转串口接口后再开展烧录、启动日志及屏幕/音频/拍照实测。

## 验收清单

- 本项目当前基线的原版 CAM 配置可以干净构建，版本、依赖和构建参数有记录。
- 屏幕显示、触摸/按键（若有）、摄像头采集和画面方向均在目标硬件上验证。
- 唤醒和连续对话回归测试通过；如启用电源管理，确认它不会打断聆听或导致误入睡眠。
- 电量/充电读数和状态经过实测，空闲息屏可恢复；如启用深睡眠，必须验证唤醒路径。
- 自定义 server 单独验证 OTA 配置获取、认证、音频上下行和断线重连。
- 变更保持局部、独立提交，并注明未实机验证的部分。

## 给接手者的边界

以本项目和项目 `AGENTS.md` 为准，不回退至交接旧稿中的 v2.5.0。先复核当前状态，保留已有工作，不把整份厂商 2.2.6 合并进来。未经核实，不更改公共音频管线、唤醒算法、MQTT/WebSocket 协议实现，也不将旧 `sdkconfig`、`build/`、`managed_components/` 或依赖锁文件作为移植内容。
