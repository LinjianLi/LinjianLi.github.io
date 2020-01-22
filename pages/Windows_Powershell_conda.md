# powershell conda

最近在配置新电脑 (Windows 10) 的时候，发现直接使用 Powershell 无法执行 `conda activate`，即使是我执行了 `conda init powershell` 之后再执行都会提示我没有配置好。直接使用 cmd.exe 的时候就可以。

后来发现每次打开 Powershell 都会有一段红色的文字警告，只是我一直都没有留意而已

```
PowerShell 无法加载文件ps1，因为在此系统中禁止执行脚本
```

因为执行 `conda activate` 的时候是会运行 `conda.bat` 这个脚本的，因为 Powershell 禁止执行脚本，所以就没用了。

网上找到的解决办法就是在 Powershell 执行

```
set-ExecutionPolicy RemoteSigned
```

已解决。