# Out-of-Tree Modules Changelog (`android_kernel_xiaomi_sm8550-modules`)

> **仓库定位**
> - 本仓库（`lineage-24.0` 分支）承载 Xiaomi SM8550（Kalama / 骁龙 8 Gen 2）的 24 套高通/NXP 外置驱动模块源码，与主内核仓库 [`KuryxCherry/android_kernel_xiaomi_sm8550`](https://github.com/KuryxCherry/android_kernel_xiaomi_sm8550) 配套构建。
> - **源码洁净原则**：本仓库不进行私自驱动魔改，仅跟踪 `LineageOS/android_kernel_xiaomi_sm8550-modules` 与高通 CodeLinaro (CLO) `KAILUA` 正式安全标签，确保 ABI 稳定与可长期平滑合并。
> - **构建入口**：模块编译与 `vendor_dlkm.img` / `vendor_boot` 打包统一在主内核仓库的 `Build Full Xiaomi SM8550 Kernel + Modules` 工作流中完成，本仓库仅保留上游驱动同步工作流 `sync_clo_drivers.yml`。

---

## 一、当前已同步 CLO KAILUA 驱动基线

| 子系统驱动 | 子树路径 | 已同步 Tag | 状态 |
| :--- | :--- | :--- | :--- |
| **Graphics (KGSL)** | `qcom/opensource/graphics-kernel` | `GRAPHICS.LA.1.0.r1-10600-KAILUA.0` | ✅ 已合入 |
| **Display (SDE/DRM)** | `qcom/opensource/display-drivers` | `DISPLAY.LA.3.0.r1-14200-KAILUA.0` | ✅ 基线已包含 |
| **Camera** | `qcom/opensource/camera-kernel` | `CAMERA.LA.3.0.r1-11600-KAILUA.0` | ✅ 已合入 |
| **Video (Iris/Venus)** | `qcom/opensource/video-driver` | `VIDEO.LA.3.0.r1-10700-KAILUA.0` | ✅ 已合入 |
| **EVA (CV)** | `qcom/opensource/eva-kernel` | `CV.LA.1.0.r1-06500-KAILUA.0` | ✅ 已合入 |
| **DataIPA** | `qcom/opensource/dataipa` | `LA.VENDOR.13.2.0.r1-28400-KAILUA.QSSI16.0` | ✅ 已合入 |
| **WLAN Trio (`fw-api` / `cmn` / `qcacld-3.0`)** | `qcom/opensource/wlan/*` | `LA.VENDOR.13.2.0.r1-28400-KAILUA.QSSI16.0` | ✅ 原子锁定同步 |
| **SecureMSM** | `qcom/opensource/securemsm-kernel` | 保持 LineageOS 基线 | ⚠️ 上游 `smcinvoke/compat/trace_smcinvoke.h` 存在 LineageOS 兼容补丁冲突，自动跳过以防破坏编译 |

---

## 二、同步工作流说明（`.github/workflows/sync_clo_drivers.yml`）

- **动态探测最新 Tag**：执行 `batch-kailua` 或单项驱动且 `clo_tag` 留空时，自动通过 `git ls-remote --tags` 筛选最新 `KAILUA` Android LA Tag（不再硬编码历史版本）。
- **WLAN 三件套原子锁**：`wlan-fw-api`、`wlan-cmn`、`wlan-qcacld-3.0` 自动取三仓公共最新 Tag 并作为原子事务合并；任一子仓冲突即整体回滚，杜绝 Wi-Fi 驱动 ABI 失配。
- **默认演练保护**：`dry_run` 默认为 `true`，先查看 GitHub Step Summary 表格确认无冲突后再设 `false` 正式推送；推送完成后请前往主内核仓库触发一次 `Build Full Xiaomi SM8550 Kernel + Modules` 验证 24 套模块全部编通。

---

## 三、维护日志

### 2026-11-26
- **CI 规范整理**：
  - 移除无效的 1 字节空文件 `.github/workflows/build.yml`（消除每次推送必红报错）；
  - 重构 `.github/workflows/sync_clo_drivers.yml`：支持动态探测最新 CLO `KAILUA` LA Tag、默认开启 `dry_run` 保护、补充 `concurrency` 并发锁、容错拉取与 `$GITHUB_STEP_SUMMARY` 汇总表，并新增独立的 `wlan-trio` 选项；
  - 新增本文档 `CHANGELOG_MODULES.md`。

### 2026-11-24
- 批量合入 7 项最新 CLO `KAILUA` 安全与稳定性标签（`graphics-kernel`、`camera-kernel`、`video-driver`、`eva-kernel`、`dataipa`、`qca-wifi-host-cmn`、`qcacld-3.0`）。
