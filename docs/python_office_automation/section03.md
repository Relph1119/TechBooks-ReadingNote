# Section03 “控制”模块：增强办公软件及周边软硬件的交互能力

## 1 文本处理函数：三招解决数据对齐问题

### 1.1 数据对齐的思想

- 对齐方式：右对齐、左对齐和居中对齐
  1. 数值型数据会自动靠右对齐，比如日期、时间、数字
  2. 文本型数据会自动靠左对齐，比如汉字、字母、英文、引号开头的数字

- 应对场景：
  1. 采用`format()`函数，实现数字对齐
  2. 使用`split()`和`join()`函数，实现日期右侧对齐
  3. 使用`strip()`函数，实现文本型数据的左侧对齐

### 1.2 使用`format()`函数，实现数字对齐

- 处理思路：使用`format()`函数进行格式处理后，保留小数点后的位数，再粘贴到Excel中，可以实现自动右侧对齐

- 格式要求：
    1. 符号和空格：`+`表示在正数前显示`+`，`-`表示负数前显示`-`，（空格）表示在正数前加空格
    2. 保留小数点后的位数：`^`、`<`、`>`分别是居中、左对齐、右对齐
    3. 类型：用`b`、`d`、`f`分别表示二进制、十进制和浮点数


```python
# 右对齐，6个字符宽度
print("{:>6d}".format(100))
```

       100
    

### 1.3 使用字符串函数，实现日期右侧对齐

处理思路：先拆分日期、再调整格式、最后合并日期


```python
date_demo = [
    "2021-03-18",
    "21-3-18",
    "2021-12-21",
    "2021-3-8",
    "21-3-1"]

for dd in date_demo:
    # 拆分日期
    year, month, day = dd.split('-')
    # 调整格式
    if len(year)== 2 :
        new_year = 2021
    else:
        new_year = year
    
    # 对月份进行调整
    month = "{:>02d}".format(int(month))
    # 对日期进行调整
    day = "{:>02d}".format(int(day))
    # 合并日期
    new_date = [str(new_year), str(month), str(day)]
    new_date = "-".join(new_date)

    print(new_date)
```

    2021-03-18
    2021-03-18
    2021-12-21
    2021-03-08
    2021-03-01
    

### 1.4 使用字符串函数，实现文本型数据的对齐

实现思路：使用字符串自带的`strip()`函数，删除文字前后空格


```python
string = "   广东省广州市   "
newstring = string.strip()
print(f"|{newstring}|")
```

    |广东省广州市|
    

## 2 Excel插件：如何扩展Excel的基本功能？

### 2.1 Power Query插件

- 主要用途：在查询方面对Excel进行优化
- 优化步骤：获取数据、转换和加载
  1. 获取数据：切换到“数据”选项卡。接着，单击“获取数据”按钮，在下拉菜单中选择“自文件”命令，继续在下一级菜单中选择“从文件夹”命令，通过弹出的“文件夹”对话框，就可以加载文件夹中的所有 Excel 文件
  2. 转换：把加载到`Power Query`中的数据从文件到`sheet`，再从`sheet`到行列依次处理，对合并好的数据再进行添加、删除列、筛选、添加自定义计算等操作
  3. 加载：把已经清理和转换的数据返回到 Excel 中

### 2.2 把多个文件合并成一个Excel

1. 获取数据：通过Excel的“数据选项卡”-“获取数据”-“自文件”-“从文件夹”命令，选中需要合并的文件夹，这样该文件夹内的所有文件都会被识别出来
2. 把工作簿中的数据从`Content`列中解析出来，并且添加在现有内容的右侧，切换到“添加列”选项卡，单击“自定义列”按钮，然后在弹出的对话框中输入公式`Excel.Workbook([Content],true) `
3. 调整每个 sheet 的每一行，点击第二步添加“自定义”列右侧的数据展开按钮，然后取消勾选“使用原始列名作为前缀”复选框，并点击确定
4. 在调整完行之后，你还需要调整每个文件中要查询的`sheet`和列
5. 使用主页选项卡的“删除列”下拉列表，选择“删除其他列”
6. 修改类型：点击列标题前的类型图标，通过弹出的下拉列表，选择指定的类型
7. 把编辑器的数据保存回Excel中，点击“主页”选项卡上的关闭并下载

### 2.3 单元格的拆分

1. 启动Power Query编辑器
2. 拆分：选中“转换”选项卡，通过选中需要拆分的列，使用拆分列下拉列表的“按分隔符拆分”按钮，把分隔符改为空格后，再点击确定
3. 删除原始列，完成拆分功能

## 3 VBA脚本编程：如何扩展Excel，实现文件的批量打印？

### 3.1  宏和VBA脚本的用途

- 通过宏可以记录的内容包括对 Excel 格式和文字的修改
- 使用宏的录制功能，把格式调整、复制粘贴、打印等重复操作记录下来，并保存成一个快捷键
- 利用 VBA 脚本可以扩展宏的功能，把手动操作部分实现自动化

### 3.2 使用宏，实现单个工作表的打印

- 自动打印的步骤：
    1. 录制宏
    2. 手动执行一次操作 
    3. 停止宏录制
    4. 通过快捷键执行宏

### 3.3 使用VBA脚本的循环，打印多个工作表

1. 查看当前宏的VBA脚本：使用视图-宏-查看宏按钮，选中要查看的宏，并点击右侧的编辑按钮
2. 编写VBA脚本
3. 通过VBA脚本增强了默认录制宏的功能，实现了批量打印工作表的功能

## 4 PowerShell脚本：如何实现文件批量处理的自动化？

### 4.1 PowerShell介绍

- PowerShell是开源的终端命令解释器
- 使用场景：所有的GUI操作就完全可以基于PowerShell实现，能用GUI界面实现Windows操作系统中的所有功能

### 4.2 实现文件批量重命名

1. 批量创建文本文件

```powershell
foreach($num in (1..10)) { New-Item $num".txt" -type file }
```

2. 批量修改文件名

```powershell
dir *.txt | foreach { Rename-Item $_ -NewName ("new_"+$_.BaseName+"_new.txt")  }
```

其中`$_`这个内置变量表示当前对象，`$.BaseName`用于获得文件 (不包含扩展名的) 基本名称，以及`$.extension`用于获得扩展名

3. PowerShell中与“Item”相关的Cmdlet

```
PS> Get-Command -Noun Item

CommandType     Name                            Definition
-----------     ----                            ----------
Cmdlet          Clear-Item                      Clear-Item [-Path] <String[]...
Cmdlet          Copy-Item                       Copy-Item [-Path] <String[]>...
Cmdlet          Get-Item                        Get-Item [-Path] <String[]> ...
Cmdlet          Invoke-Item                     Invoke-Item [-Path] <String[...
Cmdlet          Move-Item                       Move-Item [-Path] <String[]>...
Cmdlet          New-Item                        New-Item [-Path] <String[]> ...
Cmdlet          Remove-Item                     Remove-Item [-Path] <String[...
Cmdlet          Rename-Item                     Rename-Item [-Path] <String>...
Cmdlet          Set-Item                        Set-Item [-Path] <String[]> ...
```

### 4.3 按扩展名搜索文件

1. 显示当前目录下所有的文件扩展名

```powershell
dir | foreach{$_.extension} | Get-Unique
```

2. 取出符合扩展名的文件名称

使用`Where-Object`命令，对管道中的每个对象进行筛选，把不符合条件的对象删除

```powershell
dir | Where-Object{ $_.extension -eq ".zip" }
```

同时搜索“.zip”和“.rar”文件
```powershell
dir | Where-Object{ ($_.extension -eq ".zip" ) -or ($_.extension -eq ".rar" ) }
```

## 5 总结

&emsp;&emsp;本篇章主要介绍使用办公软件进行自动化操作。
1. 通过使用python中的`format()`函数对字符串或数字进行格式化，`split()`函数将字符串按照指定的分隔符进行分隔，`join()`函数将列表、元组的每个元素连成字符串，`strip()`函数能够自动去掉字符串前后的空格，完成对文本的对齐
2. 通过Excel的Power Query插件，实现多个文件的合并和单元格的拆分处理，并能完成自动化查询的功能
3. 通过宏和VBA脚本，支持对所有Office办公的自动化操作
4. 通过使用PowerShell脚本，完成批量改名和按多个扩展名搜索文件的功能，使用Cmdlet类型的命令，支持批量操作
