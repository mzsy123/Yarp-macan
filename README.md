# YARP TWRP for OnePlus Ace 6T (macan)

警告⚠️免责声明
本项目产物暂未在真机上开机、触摸与解密。刷入 recovery 可能变砖或丢失数据，请自行备份并自担风险。

在 GitHub Actions 云端构建 OnePlus Ace 6T（macan）的 TWRP recovery，界面使用 TWRP-Test `twrp-16.0` 分支内置的 **YARP（GUI2，基于 LVGL）**。全程云端编译，本地无需安装任何工具链。

> 本项目 fork 自 [MDXRosmontis/yarp-sm8850](https://github.com/MDXRosmontis/yarp-sm8850)，原版面向小米 SM8850，本仓库改为面向 OnePlus Ace 6T。

## 设备

| 项目 | 值 |
| --- | --- |
| 代号 | macan（内核目录名 macanc） |
| 型号 | PLR110（Ace 6T 国行）/ CPH2767 等（15R） |
| SoC | Snapdragon 8 Gen 5（SM8845） |
| 平台 | canoe |
| Recovery | TWRP-Test `twrp-16.0`（默认 YARP GUI2） |

## 构建（全程 GitHub，本地无需任何环境）

1. Fork 本仓库，以及配套设备树 [android_device_oneplus_macan](https://github.com/mzsy123/android_device_oneplus_macan)。
2. 打开本仓库 → **Actions** → **Build YARP Macan (ccache)** → **Run workflow**。
3. 构建约 3～6 小时。完成后在本次运行的 **Artifacts** 下载产物（保留 14 天）。

> `Build YARP SM8850` 是不带缓存的原始工作流；`(ccache)` 版本带编译缓存，第二次起明显更快，推荐用后者。

## 产物

| 文件 | 说明 |
| --- | --- |
| `YARP-macan-recovery.img` | 可刷入的 recovery 镜像（ramdisk-only，v4 boot header） |
| `SHA256SUMS.txt` | 镜像校验和 |
| `verification.json` | 静态校验报告（header、YARP 资源、GUI2 字体等） |
| `source-manifest.xml` | 解析后的源码清单 |
| `build.log` / `sync.log` 等 | 诊断日志（在 `-logs` 这个 artifact 里） |

## 目录结构
.
├── .github/workflows/
│ ├── build.yml # 原始工作流（无缓存）
│ └── build-ccache.yml # 带 ccache 缓存加速
├── scripts/
│ ├── build-yarp.sh # sync / build / package 主脚本
│ ├── source-lock.py # 按 sources.json 固定源码版本
│ ├── verify-image.py # 产物静态校验
│ ├── prepare-runner.sh # 清理 runner 磁盘 + 16 GiB swap
│ └── resource-report.sh # 构建期资源监控
├── sources.json # 源码 pin 清单（manifest / TWRP 项目 / 设备树）
└── README.md
代码

## 加速说明

`build-ccache.yml` 使用 ccache 缓存编译中间产物，并通过 GitHub Actions cache 持久化（上限 10 GB，7 天未使用自动失效）。首次构建不受益，第二次起编译阶段可显著缩短。

## 刷入（在 Ace 6T 上，风险自负）

```bash
# 先临时启动试水（不落盘，重启即回正常系统）
adb reboot bootloader
fastboot boot YARP-macan-recovery.img

# 确认可用后再正式刷入
adb shell su -c 'dd if=/dev/block/by-name/recovery_a of=/sdcard/recovery_a_stock.img'  # 备份
fastboot flash recovery_a YARP-macan-recovery.img
fastboot reboot recovery
致谢
•
TWRP-Test（twrp-16.0 / YARP GUI2）
•
MDXRosmontis/yarp-sm8845（构建脚手架）
•
OnePlus-SM8845-Development（内核与模块）
