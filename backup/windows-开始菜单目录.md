删除开始菜单应用列表，找对应文件夹删除。

目录路径

创建或删除程序快捷方式将影响所有用户的开始菜单

```
C:\ProgramData\Microsoft\Windows\Start Menu\Programs
```

创建或删除程序快捷方式只会影响特定用户的开始菜单

```
C:\Users\[你的用户名]\AppData\Roaming\Microsoft\Windows\Start Menu\Programs
```

注册表打开备份
打开注册表编辑器：点击开始菜单，输入“regedit”并打开注册表编辑器。
修改前建议备份注册表：在注册表编辑器顶部菜单中选择“文件” > “导出”。选择一个安全的位置保存整个注册表的备份。
注册表开始菜单路径设置

在这里，你会找到名为 Common Start Menu 的项，它指向所有用户的开始菜单的位置。

```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders
```

在这里，名为 Start Menu 的项指向当前登录用户的开始菜单的位置。

```
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders
```