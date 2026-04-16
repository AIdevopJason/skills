---
name: windows-pyinstaller-distribute
description: >-
  Packages a Python CLI or desktop tool for Windows colleagues using PyInstaller
  (prefer onedir), stable batch launchers, and a zip-ready release folder. Use when
  the user wants to ship a .exe without requiring Python on target PCs, mentions
  PyInstaller, onefile/onedir, colleague distribution, SmartScreen, antivirus,
  OneDrive build failures, or batch files that break on some machines.
---

# Windows：PyInstaller 打包与同事分发

面向「目标机不装 Python、双击可用、尽量少踩坑」的发布流程。默认 **Windows 64 位**。

## 何时采用本流程

- 需要把 Python 脚本交给同事，对方不应安装 Python / pip。
- 遇到过：**部分人能用、部分人闪退**；**bat 乱报 `'xxx' is not recognized`**；**OneDrive 下构建报拒绝访问**；**杀毒拦截**。

## 核心原则（按优先级）

1. **解释器版本**：分发用构建机优先 **Python 3.12 或 3.11（64 位）**。避免用最新主线（如 3.14）打给全公司，旧系统/策略环境更容易出兼容问题。
2. **打包形态**：默认 **onedir**（`exe` + `_internal`），**不要默认 onefile**。onefile 解压到临时目录，更容易被企业杀毒误杀或启动失败。
3. **UPX**：构建时 **关闭 UPX**（`--noupx` / spec 里 `upx=False`），降低误报。
4. **构建输出路径**：`--workpath` 与 **最终 `dist` 输出**尽量放在 **`%TEMP%`**，避免项目盘在 **OneDrive/同步盘** 上导致 `PermissionError` 清理失败。
5. **分发物**：只发 **`release/...` 整夹**（含 `_internal`），压缩成 zip；**禁止**只发单个 `exe` 或单个 `bat`。
6. **启动器 `.bat`**：
   - 内容用 **纯 ASCII**（提示语也用英文），避免部分 `cmd` 在编码/换行异常时把中文行拆碎执行。
   - 文件必须是 **`CRLF` 换行**。
   - 若存在 `exe`，必须 **同目录存在 `_internal`**，否则明确报错并 `pause`。
7. **运行时诊断**：主程序在 `frozen` 下把致命错误写入 **`exe` 同目录的 `last_error.log`**，必要时 Windows 弹窗（便于非技术同事反馈）。
8. **文档**：README 写明 **VC++ 2015–2022 x64**、SmartScreen、杀毒白名单/隔离恢复、必须整夹解压。

## PyInstaller spec 注意点

- `SPECPATH` 在 spec 内表示 **含有该 `.spec` 文件的目录**，脚本路径应写 `Path(SPECPATH) / "entry.py"`，**不要再 `.parent`**，否则会把入口指到错误目录。
- onedir：`EXE(..., exclude_binaries=True, ...)` + `COLLECT(...)`；`onefile` 与此不同，不要混用模板。

## 推荐发布目录结构（示例）

```text
RELEASE_NAME/
  run_xxx.bat              # ASCII + CRLF 启动器
  app.exe                  # PyInstaller 生成的入口
  _internal/               # 必须与 exe 同级，整夹带走
  input/                     # 若业务需要
  output/
  README.txt
```

## 构建脚本要点（模板级）

- 选择解释器顺序示例：`py -3.12` → `py -3.11` → 再回退其它。
- `pip install` 与 `PyInstaller` **必须用同一个** `PY_CMD`（同一解释器）。
- 构建前清理：`%TEMP%\app_build`、`%TEMP%\app_dist`（或等价目录）。
- 用 `robocopy` 把 onedir 产物同步到 `release/...`（成功码 0–7；`>=8` 才算失败）。
- 构建结束后 **复制** 最新 `run_*.bat` 与 `README.txt` 进 `release/...`。

## 给同事的排查顺序（可贴进 README）

1. 是否 **完整解压 zip**，且文件夹里同时有 **`exe` 与 `_internal`**。
2. 是否被杀软隔离；企业环境是否需 **加白名单**。
3. 是否缺少 **VC++ x64 运行库**（官方 `vc_redist.x64.exe`）。
4. 查看 **`last_error.log`**（若生成）。

## 常见症状对照

| 现象 | 常见原因 |
|------|----------|
| bat 报 `'cho'`、`'PDF'`、`'SCRIPT'` 等碎片命令 | bat **编码/换行**异常或非 ASCII 内容；改为 **ASCII + CRLF** |
| 构建时 `PermissionError` 删不掉 `dist` | 输出目录在同步盘或被锁定；改 **`%TEMP%` 输出** |
| 双击 exe 无反应 | 被杀软拦截；或 **缺 `_internal`** |
| 仅部分人失败 | 组合因素：缺 VC++、策略、**解释器版本过新**；用 **3.12/3.11** 重打 |

## Agent 执行清单（接到「要打包给同事」类需求时）

- [ ] 确认构建机 Python：**优先 3.12 x64**（或 3.11）。
- [ ] 采用 **onedir + noupx**；work/dist 放 **TEMP**。
- [ ] spec 中 `SPECPATH` 用法正确。
- [ ] 启动 bat：**ASCII + CRLF**，检查 `_internal`。
- [ ] 准备 **`release/...` 整夹** + README；提醒用户 **zip 整夹发送**。
- [ ] 可选：主程序写 **`last_error.log`** + 错误弹窗。

## 与本仓库的对应关系（参考）

若项目已存在 `build_*_exe.bat`、`*.spec`、`release/...`，优先 **沿用同一模式** 改入口名与目录名即可，不要另起一套矛盾流程。
