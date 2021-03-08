# 怎么删除右键出现的Open Folder as Intellij IDEA Project

首先，Win+R 输入 regedit （注册表编辑器），然后按Enter。

1.右键单击文件夹背景时（出现的情况）：

- 在注册表中打开“ 计算机 \ HKEY_CLASSES_ROOT \ Directory \ Background \ shell \ IDEA Community ”。
- 删除整个“IDEA Community”文件夹。

2.右键单击文件图标时（出现的情况）：

- 在注册表中打开“ 计算机 \ HKEY_CLASSES_ROOT \ * \ shell \ Open with IDEA Community ”。
- 删除整个“Open with IDEA Community”文件夹。

3.右键单击文件夹图标时（出现的情况）：

- 在注册表中打开“ 计算机 \ HKEY_CLASSES_ROOT \ Directory \ shell \ IDEA Community ”。
- 删除整个“IDEA Community”文件夹。