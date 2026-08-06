<h1 align="center">
  <img src="https://orangefox.download/fox.svg" height="60" style="vertical-align:bottom"/>
  OrangeFox Recovery Builder — Xiaomi onyx
</h1>

<h3 align="center">
  <i>基于 GitHub Actions 的 OrangeFox Recovery 云端构建器，预装中文字体与中文语言支持</i>
</h3>

<p align="center">
  <strong>设备</strong>：Xiaomi Redmi Turbo 4 Pro / POCO F7 (onyx)<br>
  <strong>平台</strong>：Qualcomm SM8735<br>
  <strong>OrangeFox</strong>：R12.0 (Android 14 / SDK 34)<br>
  <strong>语言</strong>：简体中文（默认）/ English
</p>

---

## 项目简介

本仓库是 [OrangeFox-Recovery-Builder](https://github.com/YJDDGT/OrangeFox-Recovery-Builder) 的独立构建仓库，专为 **Xiaomi onyx** 设备定制，并额外集成了 **中文字体与中文语言支持**。

原始 Recovery（`OrangeFox-R12.0-Unofficial-onyx.zip`）功能完整但缺少中文字体，导致界面中文显示为方块（tofu）。本项目通过以下配置解决该问题：

| 配置项 | 作用 |
|--------|------|
| `FOX_USE_MISANS_FONTS=1` | 使用小米 MiSans 字体替换默认 InterDisplay，正确渲染 CJK 字符 |
| `TW_EXTRA_LANGUAGES := true` | 启用包括中文在内的多语言字符串 |
| `TW_DEFAULT_LANGUAGE := zh_CN` | 设置默认语言为简体中文 |

以上修改已提交至设备树仓库 [android_device_xiaomi_onyx_recovery](https://github.com/YJDDGT/android_device_xiaomi_onyx_recovery) 的 `fox_12.0` 分支。

---

## 构建方法

### 方式一：GitHub Actions 网页触发

1. 进入仓库的 **Actions** 标签页
2. 左侧选择 **OrangeFox - 构建** 工作流
3. 点击右上角 **Run workflow**，填写以下参数：

| 参数 | 说明 | 本项目填写的值 |
|------|------|----------------|
| OrangeFox 版本 | OrangeFox manifest 分支 | `14.1` |
| OrangeFox 设备树 | 设备树仓库 URL | `https://github.com/YJDDGT/android_device_xiaomi_onyx_recovery` |
| OrangeFox 设备树分支 | 设备树分支名 | `fox_12.0` |
| 指定设备路径 | 设备在源码树中的路径 | `device/xiaomi/onyx` |
| 指定设备代号 | 设备代号 | `onyx` |
| 构建目标 | 镜像类型（onyx 有独立 recovery 分区） | `recovery` |
| 设备名称 | 显示在 Release 中的名称 | `Xiaomi Redmi Turbo 4 Pro (POCO F7)` |

4. 点击底部绿色 **Run workflow** 开始构建
5. 构建预计耗时 **1–2 小时**（含源码同步与编译）

### 方式二：API 触发

```bash
curl -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer <YOUR_TOKEN>" \
  https://api.github.com/repos/YJDDGT/OrangeFox-Builder-Onyx/actions/workflows/OrangeFox-Compile.yml/dispatches \
  -d '{
    "ref": "main",
    "inputs": {
      "MANIFEST_BRANCH": "14.1",
      "DEVICE_TREE": "https://github.com/YJDDGT/android_device_xiaomi_onyx_recovery",
      "DEVICE_TREE_BRANCH": "fox_12.0",
      "DEVICE_PATH": "device/xiaomi/onyx",
      "DEVICE_NAME": "onyx",
      "BUILD_TARGET": "recovery",
      "DN": "Xiaomi Redmi Turbo 4 Pro (POCO F7)"
    }
  }'
```

---

## 获取构建产物

构建成功后，产物（`recovery.img` 和 OrangeFox 卡刷包 `.zip`）会自动上传到：

> **[Releases 页面](https://github.com/YJDDGT/OrangeFox-Builder-Onyx/releases)**

Release 标题格式为 `onyx // YYYYMMDD`，包含以下信息：
- 构建编号
- 设备名称与代号
- OrangeFox 版本
- 构建日期

---

## 刷入 Recovery

onyx 拥有独立的 recovery 分区，支持直接 fastboot 刷入：

### 方法一：Fastboot 刷入（推荐）

```bash
# 进入 fastboot 模式
adb reboot bootloader

# 刷入 recovery 镜像
fastboot flash recovery recovery.img

# 重启进入 recovery 验证
fastboot reboot recovery
```

### 方法二：在现有 Recovery 中刷入卡刷包

1. 将 `OrangeFox-R12.0-*.zip` 传入手机
2. 进入现有的 OrangeFox / TWRP Recovery
3. 选择 **Install** → 选择 zip 包 → 滑动刷入
4. 刷入完成后重启进入 Recovery

---

## 构建流程

```
GitHub Actions 触发
    │
    ▼
┌─────────────────────────┐
│  构建环境准备             │  安装依赖、配置 Swap 24GB
└───────────┬─────────────┘
            ▼
┌─────────────────────────┐
│  同步 OrangeFox 源码      │  gitlab.com/OrangeFox/sync
│  (fox_14.1 manifest)     │
└───────────┬─────────────┘
            ▼
┌─────────────────────────┐
│  克隆设备树               │  android_device_xiaomi_onyx_recovery
│  (含中文支持配置)          │  fox_12.0 分支
└───────────┬─────────────┘
            ▼
┌─────────────────────────┐
│  编译 Recovery 镜像       │  lunch twrp_onyx-eng && mka recoveryimage
│  (含 MiSans 中文字体)     │
└───────────┬─────────────┘
            ▼
┌─────────────────────────┐
│  整理产物 & 上传 Release  │  recovery.img + OrangeFox.zip
└─────────────────────────┘
```

---

## 设备树中文支持详情

设备树仓库 `fox_12.0` 分支中的关键修改（commit `a3d1586`）：

**`BoardConfig.mk`**
```makefile
# 启用多语言（含中文）
TW_EXTRA_LANGUAGES := true
# 默认语言设为简体中文
TW_DEFAULT_LANGUAGE := zh_CN
```

**`vendorsetup.sh`**
```bash
# 使用 MiSans 字体，正确渲染中文字符
export FOX_USE_MISANS_FONTS=1
```

MiSans 字体由 OrangeFox 官方在 `fox_14.1` 分支中支持，通过该环境变量在构建时自动替换默认的 InterDisplay 字体。

---

## 相关仓库

| 仓库 | 说明 |
|------|------|
| [OrangeFox-Builder-Onyx](https://github.com/YJDDGT/OrangeFox-Builder-Onyx) | 本仓库 — Actions 构建工作流 |
| [android_device_xiaomi_onyx_recovery](https://github.com/YJDDGT/android_device_xiaomi_onyx_recovery) | 设备树（含中文支持配置） |
| [OrangeFox-Recovery-Builder](https://github.com/YJDDGT/OrangeFox-Recovery-Builder) | 原始 Fork 构建仓库 |
| [OrangeFox 官方](https://gitlab.com/OrangeFox) | OrangeFox 源码与同步脚本 |

---

## 致谢

- [OrangeFox Recovery](https://gitlab.com/OrangeFox) — 完整的 Recovery 项目
- [TeamWin / TWRP](https://github.com/TeamWin) — TWRP 核心框架
- [azwhikaru](https://github.com/azwhikaru) — Recovery Builder Template
- 原始 [OrangeFox-Recovery-Builder](https://github.com/YJDDGT/OrangeFox-Recovery-Builder) 构建脚本的作者与贡献者

---

<p align="center">
  <sub>本项目仅供学习和个人使用，刷机有风险，操作需谨慎。</sub>
</p>
