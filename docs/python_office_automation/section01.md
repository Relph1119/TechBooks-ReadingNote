# Section01 “输入”模块：不同文件类型的批量合并和拆分问题

## 1 拆分与合并：快速地批量处理内容相似的Excel

### 1.1 Excel扩展库

- 支持Excel读取扩展库：`xlrd`库
- 支持Excel写入扩展库：`xlwt`库

### 1.2 Excel合并

- 自动化办公流程：
  1. 找到整个工作过程当中重复操作的部分
  2. 将重复操作的部分中，使用Python编写程序代替手工操作部分
  3. 对重复的部分，使用循环语句进行批量处理

- 文件合并的重复操作事件：
    1. 打开文件
    2. 提取用户填写内容
    3. 粘贴到汇总文件
    4. 关闭文件

**示例：**  

&emsp;&emsp;将员工的调查问卷进行合并，统计每个员工的每个问题的答案，合并文件的表头是员工姓名、第一题、第二题.....


```python
import xlrd
import xlwt
from pathlib import Path, PurePath
import os
```


```python
# 要合并excel文件的路径
src_path = 'data/ch01/调查问卷'
# 合并完成之后的文件路径
dst_file = 'data/ch01/result/结果.xls'

if not os.path.exists(PurePath(dst_file).parent):
    os.makedirs(PurePath(dst_file).parent)

# 获取该目录下所有的xlsx格式文件
p = Path(src_path)
files = [x for x in p.iterdir() if PurePath(x).match('*.xls')]

content = []
```


```python
# 对每个文件进行重复处理
for file in files:
    # 用文件名作为每个用户的标识
    username = file.name.split('.')[0]
    data = xlrd.open_workbook(file)
    table = data.sheets()[0]
    # 取得每一项的结果
    answer1 = table.cell_value(rowx=4, colx=4)
    answer2 = table.cell_value(rowx=10, colx=4)
    temp = f'{username},{answer1},{answer2}'
    # 合并为一行进行存储
    content.append(temp.split(','))
    print(temp)
```

    李雷,D,C
    调查问卷模版,D,C
    韩梅梅,D,B
    


```python
# 写入文件的表头
table_header = ['员工姓名', '第一题', '第二题']

workbook = xlwt.Workbook(encoding='utf-8')
xlsheet = workbook.add_sheet('统计结果')

row = 0
col = 0
for cell_header in table_header:
    xlsheet.write(row, col, cell_header)
    col += 1

# 下移一行
row += 1
# 取出每一行内容
for line in content:
    col = 0
    # 取出每一个单元格内容
    for cell in line:
        # 写入内容
        xlsheet.write(row, col, cell)
        # 向右移动一个单元格
        col += 1
    row += 1

# 保存文件
workbook.save(dst_file)
```

### 1.3 Excel拆分

**示例：**  

&emsp;&emsp;将员工的总工资单进行拆分，以每个员工名字命名Excel文件，并且表头为工资单的表头，即员工编号、姓名、出勤、工资、奖金、职务工资、全勤奖、实发工资。


```python
# 工资单文件
salary_file = 'data/ch01/工资单/工资单.xls'

# 拆分文件保存路径
dst_path = 'data/ch01/工资单'
```


```python
data = xlrd.open_workbook(salary_file)
table = data.sheets()[0]
# 获取表头
salary_header = table.row_values(rowx = 0, start_colx=0, end_colx=None)
```


```python
# 写入文件
def write_to_file(file_name, count):
    workbook = xlwt.Workbook(encoding='utf-8')
    xlsheet = workbook.add_sheet('本月工资')
    
    row = 0
    for line in count:
        col = 0
        for cell in line:
            xlsheet.write(row, col, cell)
            col += 1
        row += 1
        
    workbook.save(PurePath(salary_file).with_name(file_name).with_suffix('.xls'))
```


```python
# 获取员工数量
employee_number = table.nrows
# 获取每一行，并用第二个单元格（即员工姓名）作为新的文件名
for line in range(1, employee_number):
    content = table.row_values(rowx=line, start_colx=0, end_colx=None)
    # 将表头和员工工资信息重新组成一个新的文件
    new_content = []
    new_content.append(salary_header)
    new_content.append(content)
    write_to_file(file_name=content[1], count=new_content)
```

## 2 善用Python扩展库：批量合并多个文档

### 2.1 Word扩展库

- Word处理：python-docx库，专门用来编辑Word文档
- 优点：可以像操作`.txt`文本一样直接打开、修改和保存关闭文件

### 2.2 Word合并

**示例**  
&emsp;&emsp;将两个Word文档（绩效考核管理制度1、绩效考核管理制度2）进行合并


```python
from docx import Document
```


```python
# word文件所在路径
word_files_path = 'data/ch02/word样例文件'

# 获取该目录下所有的docx格式文件
p = Path(word_files_path)
files = [x for x in p.iterdir() if PurePath(x).match('*.docx')]
```


```python
# 实例化文档对象
doc = Document()
```


```python
# 保留文件原有格式的合并
def merge_files(docx_files: list):
    # 循环处理每个文件
    for file in sorted(docx_files):
        temp_doc = Document(file)
        # 遍历每一页
        for word_body in temp_doc.element.body:
            # 合并内容到新的word文档中
            doc.element.body.append(word_body)

    doc.save(Path(word_files_path, '绩效考核管理制度.docx'))
```


```python
merge_files(files)
```

### 2.3 将纯文本和Word文件合并

**注意：**  
&emsp;&emsp;Word 文件支持的格式丰富程度远远高于 Txt 文件，所以当这两种格式丰富程度不一致的文件进行合并时，要么向下兼容，去掉 Txt 不支持的格式；要么向上兼容，对 Txt 进行格式再调整。


```python
def add_content_mode1(content):
    '''
    增加内容
    '''
    para = doc.add_paragraph().add_run(content)
    # 设置字体格式
    para.font.name = '仿宋'
    # 设置下划线
    para.font.underline = True
    # 设置颜色
    para.font.color.rgb = RGBColor(255,128,128)  
```

### 2.4 将Excel和Word文件合并

**示例：**  
&emsp;&emsp;有一个邀请函的标准公文格式的Word文档，其中的被邀请人、性别（先生、女士）以及发出邀请的时间，分别用“< 姓名 >”“< 性别 >”“< 时间 >”替代。另一个Excel文件中，已有被邀请人的姓名、性别信息。将以上两个文件合并，为每个被邀请人自动生成一个Word格式的邀请函。


```python
import xlrd
from docx import Document
from pathlib import Path, PurePath
import datetime
import os
```


```python
today = datetime.date.today().strftime('%Y-%m-%d')

# 客户信息文件
customer = 'data/ch02/邀请函样例文件/客户信息.xls'

# 邀请函模版
invitation = 'data/ch02/邀请函样例文件/邀请函模版.docx'

# 邀请函路径
invitation_path = 'data/ch02/邀请函文件'

if not os.path.exists(invitation_path):
    os.makedirs(invitation_path)

# 替换内容
replace_content = {
    '<姓名>' : 'no_name',
    '<性别>' : 'm_f',
    '<今天日期>' : today
}
```


```python
# 生成邀请函文件
def generate_invitation():
    doc = Document(invitation)
    # 取出每一段
    for para in doc.paragraphs:
        for key, value in replace_content.items():
            if key in para.text:
                para.text = para.text.replace(key, value)
                
    file_name = PurePath(invitation_path, replace_content['<姓名>']).with_suffix('.docx')
    doc.save(file_name) 
```


```python
# 获取邀请函信息
def get_customer(customer_file: Path):
    # 从第一个sheet中获取客户信息
    data = xlrd.open_workbook(customer_file)
    table = data.sheets()[0]
    
    # 获取客户数量
    customer_number = table.nrows
    
    for line in range(1, customer_number):
        content = table.row_values(rowx = line, start_colx = 0, end_colx = None)
        replace_content['<姓名>'] = content[0]
        replace_content['<性别>'] = content[1]
        generate_invitation()
```


```python
get_customer(customer)
```

## 3 图片转文字：提高识别准确率

### 3.1 图片转文字的识别方式

- 在线识别方式：在初次进行文字识别的时候，准确率非常高；识别文字的过程需要在公有云的服务器上完成，并且信息泄露的风险比较大
- 离线识别方式：在识别过程中不需要连接网络，节省了在线传输图片的时间，适合那些对实时性要求比较高或网络信号比较差的场景

|     | 初次识别准确率 | 安全性 | 是否需要人工训练模型 | 实时性 |
| :- | :-: | :-: | :-: | :-: |
| 在线识别 | 高 | 一般 | 不需要 | 一般 |
| 离线识别 | 一般 | 高 | 需要 | 高 |

### 3.2 在线识别

- 百度Ai在线文字识别：使用baidu-api包，提供AipOcr函数实现用户验证、client.basicGeneral函数实现文字识别功能
- 使用步骤：
  1. 安装SDK
  2. 注册用户
  3. 申请应用

### 3.3 离线识别import


```python
import pytesseract
from PIL import Image
```


```python
pytesseract.pytesseract.tesseract_cmd = r"D:/Tesseract-OCR/tesseract.exe"
```


```python
# 打开图片
image = Image.open('data/ch03/example.png')

# 转为灰度图片
imgry = image.convert('L')
```


```python
# 二值化，采用阈值分割算法，threshold为分割点，根据图片质量调节
threshold = 150
table = []
for j in range(256):
    if j < threshold:
        table.append(0)
    else:
        table.append(1)

temp = imgry.point(table, '1')

# OCR识别：lang指定中文,--psm 6 表示按行识别，有助于提升识别准确率
text = pytesseract.image_to_string(temp, lang="chi_sim+eng", config='--psm 6')

# 打印识别后的文本
print(text)
```

    枪 a 发 咤
    心 RR 闹 发 索 ac
    er 0000000000 oe No 0123456789 。
    aes aa soos
    E 奂 怀 扬 55 ,
    ae fe BU US ann ath cn 一 < 1 2018400 川 0011
    白 % 一 标 ALMEIDA So ORI AFIPS 21 FAHNSKTD 2/09
    吴 硕 林人 技 吊 孔 ; 0000000000on00 万 137+ocg>esogggdggy72172“7-
    * Babe We 东 N 华n03 0000-0006600 团 31533333
    於 江 文 胡 芒 佳 如 ooooooesoooooso {9G B8OFA 70192 2 3062261006289
    洁 汀 李 RRR aS wa ake 国 E 0 绍 东
    国 医 I Aooaog 《 0 coooo 060.00 0 aon
    a yooo.00 a voo0
    中 kph 技 eves
    , “ 2 根 根 椿 有 匹 。
    yO YOO IRL Aad ‘a *an
    切 ARLASOSF: 000000000000000 nn
    东 EE OR EE MM LOR OS MR 000ocooooo0 REEMH
    兰 AR Sp ALM AULA T AUR kh GunGK:00D0000 PN
    FOAL TRG Lie mA 仪 WEE
    
    

### 3.4 对图像进行文字识别处理

- 对图像进行文字识别的一般过程：
    1. 图像输入
    2. 前期处理，比如二值化，图像降噪，倾斜纠正
    3. 文字检测，比如版面分析，字符分割
    4. 文本识别，比如字符识别，后期矫正
    5. 输出文本。

- 文字识别精确过程：
    1. 人工观察
    2. 对比原始图像
    3. 把错误的文字手工纠正为正确的汉字

## 4 总结

&emsp;&emsp;本篇章主要介绍了Excel合并与拆分、Word文件的合并、不同文件之间的合并处理、图片的文字识别；批量合并与拆分主要是将操作重复的部分用Python实现自动化操作，再通过脚本实现批量操作。通过使用`xlrd`、`xlwt`、`python-docx`、`pytesseract`等扩展库，对各类文档进行自动化处理，达到批量合并与拆分的目的。
