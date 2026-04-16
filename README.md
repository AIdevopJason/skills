# skills
Python 3.12 / 3.11 x64 作为分发构建首选，避免用过新解释器拖累兼容性
PyInstaller 默认 onedir（exe + _internal），不用 onefile 作默认
关闭 UPX、work/dist 放 %TEMP%，减少杀毒误报和 OneDrive 锁文件
启动 .bat 用 ASCII + CRLF，并校验 _internal，避免你同事那种「命令被拆碎」的 cmd 解析问题
整夹 zip 分发 + README 里 VC++ / SmartScreen / 杀毒 说明
文末有 Agent 执行清单，以后你对别的项目说「按同样方式打包给同事」时，Agent 可以按条核对
