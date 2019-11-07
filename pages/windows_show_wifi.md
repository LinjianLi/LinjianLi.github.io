# Windows 系统查看系统里保存的 WiFi 信息和密码

## 查看保存的所有 WiFi 的名字

打开 `cmd` 或 `Powershell` ，执行以下指令
```
netsh wlan show profiles
```

## 查看某个 WiFi 的信息

执行以下指令
```
netsh wlan show profile name="WiFi_name" key=clear
```
其中 `WiFi_name` 要替换成想要查看的 WiFi 的名字。在输出内容中，`关键内容`对应的就是 WiFi 的密码。 `key=clear` 是为了让密码显示出来，如果没有这个选项，输出内容中不会显示 WiFi 密码。

## 导出 WiFi 信息

执行以下指令
```
netsh wlan export profile folder="folder_path" key=clear
```
可以导出所有 WiFi 的信息到 XML 文件中，每个 WiFi 对应一个文件。其中， `folder_path` 要替换成想要导出的位置。 XML 文件中 `keyMaterial` 元素里面的内容就是密码。 `key=clear` 是为了让密码以明文形式导出，如果没有这个选项，导出的 XML 文件中 `keyMaterial` 元素里面的内容是被加密过的字符串。 如果只想导出某一个 WiFi 信息，可以执行以下指令，即加上 `name="WiFi_name` 参数。
```
netsh wlan export profile name="WiFi_name" folder="folder_path" key=clear
```