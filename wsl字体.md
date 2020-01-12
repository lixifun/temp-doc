1. regedit 打开注册表
2. HKEY_CURRENT_USER\Console 寻找 wsl 相关，或者新建 %SystemRoot%_System32_wsl.exe 项
3. 新建 DWORD32 类型 CodePage 十六进制 4101b5
