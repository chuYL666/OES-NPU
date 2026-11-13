# OES-NPU

OES（Amlogic A311D）硬件加速 fpk —— 适配 fnOS 内核 **6.18.18.c1191-trim**。

原包内核模块锁定在内核 `6.18.18.c1151-trim`，直接安装会被 fpk 的 ABI 预检拦截。本仓库提供重新适配后的版本：有源码的模块用 c1191 headers 真实编译，闭源模块做等长 vermagic 重标记，重打包保持原包成员权限（644/755 + root:root）。

## 下载

| 版本 | 文件 | 大小 | SHA256 |
|---|---|---|---|
| v5.9.20 | [trim.hwaccel-a311d.v5.9.20-c1191.fpk](https://github.com/chuYL666/OES-NPU/releases/download/v5.9.20-c1191/trim.hwaccel-a311d.v5.9.20-c1191.fpk) | 640,727,763 B | `9457a18f4a4baf2fa5263f2cdcdf9fc4a4e1b4e2b16d6a916e523ded24533262` |
| v5.9.18 | [NPUtrim.hwaccel-a311d.v5.9.18-c1191.fpk](https://github.com/chuYL666/OES-NPU/releases/download/v5.9.18-c1191/NPUtrim.hwaccel-a311d.v5.9.18-c1191.fpk) | 624,502,593 B | `f9609975c4eb5f0b3cac8f6d870d3ba9703975354e135335540ea3f30c83f20e` |

v5.9.20 比 v5.9.18 多 3 个 HEVC 上电模块（`oes_fclk_on` / `oes_wave_on2` / `oes_vcopy2`），共 16 个 `.ko`。

## 安装注意

1. 应用中心手动安装升级，脚本末尾会**自动重启**，约 4 分钟；前端可能一直显示“安装中”，以日志 `/var/log/apps/trim.hwaccel-a311d.log` 是否出现“安装完成”为准。
2. 若流程被打断（日志停在 GPU 段、没有“安装完成”），用下面命令续装，脚本幂等可重入：
   ```bash
   sudo env OES_AUTO_REBOOT=0 bash /var/apps/trim.hwaccel-a311d/cmd/install_callback
   ```
3. 包内验收脚本对 VPU 的判据较旧（要求 `lsmod` 到 `oes_codec_io`），而 c1191 内核已自带 `codec_io_probe`，会误报 `VPU_DOWN`；以 `sudo /usr/local/bin/oes-hwaccel verify` 的结果为准。
4. **换机器安装前先确认 `/lib/firmware/video/video_ucode.bin` 存在**（约 1.6 MB）。缺失会导致 HEVC 硬解 `vdec_if_init -19` 失败，且补文件后**必须重启**才生效（firmware.ko 只在开机 probe 时填清单）。
5. `oes-hwaccel verify` 涉及 `/dev/video*`，普通用户跑会 `Permission denied` 误判，必须 `sudo`。
6. 每次 fnOS OTA 换内核版本号，本包都需要重新适配一次。

## 已验证环境

- 设备：OES 盒子（A311D），fnOS 1.2.1010
- 内核：6.18.18.c1191-trim
- 结果：NPU `NPU_READY`、GPU `GL_RENDER_OK`（Mali-G52 / panfrost）、硬编 `oes_venc_m2m` 在线、硬解 `VPU_READY`（HEVC 10 帧 122880 B/帧，hang=0）
- 已知遗留：VC-1 硬解不支持（`oes_vc1_cavfix` 加载失败）；fnOS 面板 GPU 栏不显示（`HWINFO_GPU_MISSING`，仅显示层问题）
