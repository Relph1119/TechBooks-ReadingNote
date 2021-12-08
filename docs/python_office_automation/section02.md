# Section02 “运算”模块：扩展常用的统计、搜索和排序功能

## 1 函数与字典：如何实现多次替换

### 1.1 使用Python实现“替换”功能

使用`replace`函数进行替换


```python
string1 = "aaa新年快乐bbb"
string2 = string1.replace("新年快乐", "恭喜发财")
string2
```




    'aaa恭喜发财bbb'




```python
string3 = "aaa新年快乐bbb新年快乐ccc"
string4 = string3.replace("新年快乐", "恭喜发财", 2)
string4
```




    'aaa恭喜发财bbb恭喜发财ccc'



### 1.2 实现批量替换

使用字典+自定义函数替代`replace()`函数实现批量“一对一”替换


```python
city_dir = { "GUANGDONG":"广东省", 
            "HEBEI":"河北省", 
            "HUNAN":"湖南省", 
            "HANGZHOU":"杭州市" }
```


```python
def replace_city(city_name):
    return city_dir[city_name]

# 通过字典映射关系实现批量替换
def replace_multi(my_citys, replaced_string):
    for pinyin_city in my_citys:
        replaced_string = replaced_string.replace(pinyin_city, replace_city(pinyin_city))
    
    return replaced_string
```


```python
# 需要替换的城市
citys = ("GUANGDONG", "HUNAN")

# 需要替换的字符串
string1 = """GUANGDONG，简称“粤”，中华人民共和国省级行政区，省会广州。
因古地名广信之东，故名“GUANGDONG”。位于南岭以南，南海之滨，
与香港、澳门、广西、HUNAN、江西及福建接壤，与海南隔海相望。"""

string2 = replace_multi(citys, string1)
print(string2)
```

    广东省，简称“粤”，中华人民共和国省级行政区，省会广州。
    因古地名广信之东，故名“广东省”。位于南岭以南，南海之滨，
    与香港、澳门、广西、湖南省、江西及福建接壤，与海南隔海相望。
    

使用逻辑判断+自定义函数替代`replace()`函数实现“多对一”替换


```python
def age_replace(age):
    if 0 < age <= 6:
        return "少年"    
    elif 7 < age <= 18:
        return "青年"
    elif 19 < age <= 65:
        return "中年"
    else:
        return "老年"

print(age_replace(80))
```

    老年
    

## 2 图像处理库：如何实现长图拼接？

### 2.1 调用外部命令

- 使用方式：利用`subprocess`模块实现
- 实现机制：使用`run()`函数，通过指定一个可以运行程序的路径，根据该路径运行可执行文件


```python
from subprocess import run, Popen, PIPE

cmd = ["dir", "."]
return_code = run(cmd, shell=True)

print(return_code)
```

    CompletedProcess(args=['dir', '.'], returncode=0)
    


```python
# 使用Popen获取程序运行结果
with Popen(cmd, shell=True, stdout=PIPE, stderr=PIPE, encoding='gbk') as fs:
    # 如果程序在timeout秒后未执行完成，会抛出TimeoutExpired异常
    fs.wait(2)
    # 从标准输出中读取数据，直到文件结束
    files = fs.communicate()[0]
    
print(files)
```

     驱动器 E 中的卷是 Work
     卷的序列号是 A0E0-05F2
    
     E:\LearningDisk\Learning_Projects\MyOtherProject\TechBooks-ReadingNote\notes\python_office_automation 的目录
    
    2021/12/08  21:23    <DIR>          .
    2021/12/08  21:23    <DIR>          ..
    2021/12/08  08:46    <DIR>          .ipynb_checkpoints
    2021/12/08  18:32    <DIR>          data
    2021/12/07  20:04            19,322 section01.ipynb
    2021/12/08  21:23            41,336 section02.ipynb
    2021/12/08  21:08    <DIR>          __pycache__
                   2 个文件         60,658 字节
                   5 个目录 167,966,035,968 可用字节
    
    

### 2.2 长图拼接

- 处理方法：通过外部命令调用，使用图像处理软件进行长图拼接
- 使用软件：[安装ImageMagick](https://blog.csdn.net/qq_37674858/article/details/80361860)，详见README（用于长图拼接）


```python
from pathlib import Path, PurePath 
from subprocess import run

jpg_path = 'data/ch04'
result_path = 'data/ch04/result.jpg'
p = Path(jpg_path)

# 使用命令
cmd = ['magick', 'convert', '-append']

# 增加参数
for x in p.iterdir(): 
    if PurePath(x).match('*.jpg'):
        cmd.append(x)
    
# 增加拼接结果
cmd.append(result_path)

run(cmd, shell=True)
```




    CompletedProcess(args=['magick', 'convert', '-append', WindowsPath('data/ch04/01.jpg'), WindowsPath('data/ch04/02.jpg'), 'data/ch04/result.jpg'], returncode=0)



### 2.3 视频的拆分与合并

- 处理方法：通过外部命令调用，使用视频处理软件进行拆分与合并
- 使用软件：`ffmpeg`命令
- 使用场景：  
&emsp;&emsp;将视频文件切分成多个`ts`格式文件，并引入索引文件`m3u8`格式，该文件是在`udf-8`编码格式下的`m3u`视频索引，播放器通过这个索引文件找到视频下所有的分段，再依次播放视频

## 3 jieba分词：如何基于感情色彩进行单词数量统计？

### 3.1 分词

- 语义分词：对中文句子按照语义进行切割的操作
- 扩展库：`jieba`库
- 语义分割技术：从统计学角度、从词库的角度基于TF-IDF算法实现分词
- jieba分词技术：
  1. 基于词库（也称作字典文件）对文章中所有可能出现的词进行匹配
  2. 匹配之后，会生成句子中的汉字所有可能形成的词
  3. 将这些词构成有向无环图，并采用动态规划算法查找最大概率路径，尽可能不会将一个词拆分成单个汉字
  4. 从字典中，找出基于词频最大切分组合，把这些分词的组合从句子中找出来，形成一个个的词


```python
import jieba

words1="速度快，包装好，看着特别好，喝着肯定不错！价廉物美"

words2 = jieba.cut(words1)
print("/".join(words2))
```

    Building prefix dict from the default dictionary ...
    Loading model from cache C:\Users\hurui\AppData\Local\Temp\jieba.cache
    Loading model cost 0.423 seconds.
    Prefix dict has been built successfully.
    

    速度/快/，/包装/好/，/看着/特别/好/，/喝/着/肯定/不错/！/价廉物美
    

### 3.2 优化分词结果

- 主要优化方向：移除标点符号，删除和情感无关的助词、名词等
- 移除标点符号：删除停止词、根据词性提取关键词


```python
words2 = jieba.cut(words1)
words3 = list(words2)
# 停止词
stop_words = ["，", "！"]
words4 = [x for x in words3 if x not in stop_words]
print(words4)
```

    ['速度', '快', '包装', '好', '看着', '特别', '好', '喝', '着', '肯定', '不错', '价廉物美']
    


```python
# 基于词性移除标点符号
import jieba.posseg as psg

words5 = [(w.word, w.flag) for w in psg.cut(words1)]

# 保留形容词、副词、动词
saved = ['a', 'v', 'd', 'l']
words5 = [x for x in words5 if x[1] in saved]
print(words5)
```

    [('快', 'a'), ('包装', 'v'), ('好', 'a'), ('看着', 'v'), ('特别', 'd'), ('好', 'a'), ('肯定', 'v'), ('不错', 'a'), ('价廉物美', 'l')]
    

### 3.3 语义情感分析

- `snownlp`库：用于统计词的正向、负向情感倾向
- 情感倾向：通过`snownlp`的`Bayes`模型训练方法，读取模块自带的正负样本，使用`Bayes`模型中的`classify()`函数进行分类，得到`sentiments`属性的值。
- 情感倾向是正向的，`sentiments`结果会接近1，如果是负向的，结果接近0


```python
from snownlp import SnowNLP

words6 = [x[0] for x in words5]
s1 = SnowNLP(" ".join(words3))
print(s1.sentiments)
```

    0.99583439264303
    

## 4 快速读写文件：如何实现跨文件的字数统计？

### 4.1 对单个文件进行字数统计

- 操作步骤：
  1. 读取要统计的文件
  2. 对字数数量进行统计，并用变量保存
  3. 将结果写入统计字数专用文件中


```python
import pathlib

file_path = 'data/ch07/e.txt'

with open(file_path, encoding='utf-8') as f:
    content = f.read()
    words = content.rstrip()
    number = len(words)
    print(number)
```

    15
    

### 4.2 对多个文件进行字数统计


```python
src_path = 'data/ch07'
count = 0

p = Path(src_path)
files = [x for x in p.iterdir() if PurePath(x).match('*.txt')]
for file in files:
    with open(file, encoding='utf-8') as f:
        content = f.read()
        words = content.rstrip()
        number = len(words)
        print(file.name, number)
        count += number

print("Total words:", count)
```

    a.txt 627
    b.txt 647
    e.txt 15
    Total words: 1289
    

python支持的5种数据类型及适用场景

| 数据类型 | 特点 |
| :-: | :-: |
| 数字 | 多用于算数和几何计算 |
| 字符串 | 一串字符，多用于字符中间的拆分和合并 |
| 列表 | 存储多个并列数据 |
| 元组 | 存储多个并列数据，不可变 |
| 字典 | 存储映射关系 |

### 4.3 扩展需求：统计中文、英文和标点符号各自的数量

**思路：**
1. 使用字符串类型先对每个文件包含的字符进行中文、英文、数字、空格、特殊字符的划分，并使用数字类型的变量对每种类型的数量进行了记录
2. 使用了字典 + 列表的方式，用列表存储每个文件每一种字符的数量，并把它们统一存储在字典当中


```python
import string

word_count = {"count_en":[],
              "count_dg":[],
              "count_sp":[],
              "count_zh":[],
              "count_pu":[]}
```


```python
def str_count(str):
    '''找出字符串中的中英文、空格、数字、标点符号个数'''
    count_en = count_dg = count_sp = count_zh = count_pu = 0

    for s in str:
        # 英文
        if s in string.ascii_letters:
            count_en += 1
        # 数字
        elif s.isdigit():
            count_dg += 1
        # 空格
        elif s.isspace():
            count_sp += 1
        # 中文
        elif s.isalpha():
            count_zh += 1
        # 特殊字符
        else:
            count_pu += 1
    print('英文字符：', count_en)
    print('数字：', count_dg)
    print('空格：', count_sp)
    print('中文：', count_zh)
    print('特殊字符：', count_pu)
    word_count["count_en"].append(count_en)
    word_count["count_dg"].append(count_dg)
    word_count["count_sp"].append(count_sp)
    word_count["count_zh"].append(count_zh)
    word_count["count_pu"].append(count_pu)
```


```python
str1 = "'中文：', count_zh"
str_count(str1)
str2 = "'特殊字符：', count_pu"
str_count(str2)
for item in word_count:
    print(f"{item} 数量为:{sum(word_count[item])}")
```

    英文字符： 7
    数字： 0
    空格： 1
    中文： 2
    特殊字符： 5
    英文字符： 7
    数字： 0
    空格： 1
    中文： 4
    特殊字符： 5
    count_en 数量为:14
    count_dg 数量为:0
    count_sp 数量为:2
    count_zh 数量为:6
    count_pu 数量为:10
    

## 5 正则表达式：如何提高搜索内容的精确度？

### 5.1 正则表达式的搜索过程

- 使用函数：`re`库的`search()`函数，如果能搜索到匹配模型的结果，则返回匹配字符串出现的文件位置和匹配字符串的内容
- 函数参数说明：
  1. `pattern`：表示要匹配的模式
  2. `string`：表示要匹配的字符串。如果模式能够匹配成功，则会返回一个`re`的对象，`re`对象里存储的是匹配位置和匹配内容；如果匹配失败，就会返回`None`
  3. `flags`：表示`search()`函数在匹配之前可以进行各种特殊处理，比如通过`flag`参数忽略要匹配字符串的大小写功能


```python
import re

re.search("[0-9]{11}","13855556666")
```




    <re.Match object; span=(0, 11), match='13855556666'>



### 5.2 精确匹配

使用`[]`、`{}`元字符进行精确匹配，`{}`字符表示前边的字符出现次数

POSIX 字符组合和 ASCII 字符组合的常用对照表：

| POSIX字符组合 | ASCII字符组合 |
| :-: | :-: |
| \[alnum\] | \[a-zA-Z0-9\] |
| \[alpha\]| \[a-zA-Z\] |
| \[digit\] | \[0-9\] |
| \[lower\] | \[a-z\] |
| \[upper\] | \[A-Z\] |
| \[space\] | \[\\t\\n\\r\\f\\v\] |

### 5.3 模糊匹配

- `+`表示前边的元素出现的是1到无穷多次
- `*`表示前面的元素出现的是0次到无穷多次
- `？`表示前面的元素出现的是0次或一次


```python
# 匹配第一个字母是a，最后一个字母是b，中间是5个任意的字母
re.search("a.{5}b", "aaa13855b5a57890bbb")
```




    <re.Match object; span=(2, 9), match='a13855b'>



### 5.4 提取和替换

- `group(0)`中参数0：表示如果搜索过程中能够匹配成功，会把匹配到的第一个字符串作为执行结果进行返回
- `sub()`：对匹配到的字符串进行替换


```python
import re

re.search("[0-9]{3}-[0-9]{8}", "我的电话号码:010-12345678.").group(0)
```




    '010-12345678'




```python
re.sub("(Y|y)(es)*", "No", "aayesbb")
```




    'aaNobb'



### 5.5 元字符分类总结

1. 匹配单个字符，要使用`[]`和`.`元字符
2. 控制元字符出现次数，要使用`?`、`+`和`*`元字符
3. 控制元字符的顺序和位置，要使用`^`、`$`、`|`和`()`元字符

## 6 扩展搜索：如何快速找到想要的文件？

### 6.1 基础搜索方法：用`pathlib`库搜索文件

- 使用函数：使用`pathlib`库的`glob()`函数和`rglob()`函数
- `glob()`函数：可以实现基于文件名的搜索
- `rglob()`函数：基于扩展名的搜索
- `rglob`函数与`glob`函数的主要区别：`rglob`函数是从文件路径末尾向前进行匹配 

| 符号 | 含义 |
| :-: | :-: |
| * | 匹配0个或多个字符 |
| ? | 匹配一个字符 |
| \[\] | 匹配指定范围内的字符，如\[0-9\]匹配数字 |
| ** | 匹配零个或多个目录及子目录，不包括.以及..开头的目录 |

### 6.2 指定搜索路径

**实现思路：**
1. 生成配置文件，把要搜索的路径写入到配置文件中
2. 读取配置文件和搜索的自定义函数，把配置文件中的路径读取出来，逐个目录搜索
3. 将多个目录的搜索结果合并输出，得到想要的文件


```python
import configparser
import pathlib
from pathlib import Path
import os

def read_dirs(ini_filename, section, arg):
    """
    通过ini文件名,节和参数获取要操作的多个目录
    """
    current_path = pathlib.PurePath(os.getcwd())
    inifile = current_path.joinpath(ini_filename)

    # cf是类ConfigParser的实例
    cf = configparser.ConfigParser()

    # 读取.ini文件
    cf.read(inifile)

    # 读取notes节 和 searchpath参数
    return cf.get(section, arg).split(",")


def locate_file(base_dir, keywords):
    p = Path(base_dir)
    files = p.glob(keywords)
    return list(files)
```


```python
dirs = read_dirs('data/ch09/search.ini', 'notes', 'searchpath')
keywords = '**/*section*'

# 定义存放查找结果的列表
result = []
# 得到项目的根路径
program_dir = pathlib.Path(os.getcwd()).parents[1]
# 从每个文件夹中搜索文件
for dir_item in [str(program_dir) + x for x in dirs]:
    files = locate_file(dir_item, keywords)
    result += files
```


```python
# 将PosixPath转为字符串
result = [str(r) for r in result]
result
```




    ['E:\\LearningDisk\\Learning_Projects\\MyOtherProject\\TechBooks-ReadingNote\\notes\\python_office_automation\\section01.ipynb',
     'E:\\LearningDisk\\Learning_Projects\\MyOtherProject\\TechBooks-ReadingNote\\notes\\python_office_automation\\section02.ipynb',
     'E:\\LearningDisk\\Learning_Projects\\MyOtherProject\\TechBooks-ReadingNote\\notes\\python_office_automation\\.ipynb_checkpoints\\Section01-checkpoint.ipynb',
     'E:\\LearningDisk\\Learning_Projects\\MyOtherProject\\TechBooks-ReadingNote\\notes\\python_office_automation\\.ipynb_checkpoints\\section02-checkpoint.ipynb']



### 6.3 建立索引文件

**实现思路：**
1. 把配置文件目录下所有文件路径的保存方式由列表改为文件
2. 把搜索功能改为从文件搜索。


```python
def locate_file(base_dir, keywords='**/*'):
    """
    迭代目录下所有文件
    """
    p = Path(base_dir)
    return p.glob(keywords)


def write_to_db(file_name, search_list):
    """
    写入索引文件
    """
    current_path = pathlib.PurePath(os.getcwd())
    dbfile = current_path.joinpath(file_name)

    with open(dbfile, 'w', encoding='utf-8') as f:
        for r in search_list:
            f.write(f"{str(r)}\n")
```


```python
# 读取配置文件
dirs = read_dirs('data/ch09/search.ini', 'notes', 'searchpath')
# 遍历目录
result = []
# 得到项目的根路径
program_dir = pathlib.Path(os.getcwd()).parents[1]
# 从每个文件夹中搜索文件
for dir_item in [str(program_dir) + x for x in dirs]:
    for files in locate_file(dir_item):
        result.append(files)

# 将目录写入索引文件
write_to_db("data/ch09/search.db", result)
```


```python
import re

keyword = "section"

# 获取索引文件路径
current_path = pathlib.PurePath(os.getcwd())
dbfile = current_path.joinpath("data/ch09/search.db")

# 在索引文件中搜索关键字
with open(dbfile, encoding='utf-8') as f:
    for line in f.readlines():
        if re.search(keyword, line):
            print(line.rstrip())
```

    E:\LearningDisk\Learning_Projects\MyOtherProject\TechBooks-ReadingNote\notes\python_office_automation\section01.ipynb
    E:\LearningDisk\Learning_Projects\MyOtherProject\TechBooks-ReadingNote\notes\python_office_automation\section02.ipynb
    E:\LearningDisk\Learning_Projects\MyOtherProject\TechBooks-ReadingNote\notes\python_office_automation\.ipynb_checkpoints\section02-checkpoint.ipynb
    

## 7 按指定顺序给词语排序，提高查找效率

### 7.1 使用`sorted()`函数实现排序

- 函数原型：`sorted(iterable, cmp=None, key=None, reverse=False)`
- 默认排序是按照从小到大的顺序进行排序
- 不会对原有的列表进行修改，会把排序好的结果存入到一个新的列表当中

### 7.2 自定义排序

- 获取Top3：先使用`reverse`参数，再切片取前三个元素
- 自定义排序：通过`lambda`表达式定义与调用
- 字典类型排序：可以基于键来排序，也可以基于值来排序


```python
student_dict1 = {'Jerry': '1003',
                 'Tom': '1005',
                 'Beta': '2001',
                 'Shuke': '2003'}

# 输出字典的键和值
print(student_dict1.items())
```

    dict_items([('Jerry', '1003'), ('Tom', '1005'), ('Beta', '2001'), ('Shuke', '2003')])
    


```python
# 按照字典的键排序
print(sorted(student_dict1.items(), key=lambda d: d[0]))
```

    [('Beta', '2001'), ('Jerry', '1003'), ('Shuke', '2003'), ('Tom', '1005')]
    


```python
# 按照字典的值排序
result = sorted(student_dict1.items(), key=lambda d: d[1])
print(result)
```

    [('Jerry', '1003'), ('Tom', '1005'), ('Beta', '2001'), ('Shuke', '2003')]
    


```python
# 将结果转换为字典
print(dict(result))
```

    {'Jerry': '1003', 'Tom': '1005', 'Beta': '2001', 'Shuke': '2003'}
    

### 7.3 排序场景总结

1. 如果能转换成列表，可以采用更改`lambda`下标的方式，实现对指定字段的排序
2. 如果不能转换成列表，可以尝试将复杂的类型中，不需要进行排序的部分进行删减，简化成列表或字典类型

## 8 通过程序并行计算，避免CPU资源浪费

### 8.1 并行计算的必要性

1. 大型运营场景下，python自动化执行需要进行大量的计算，而计算主要是通过CPU来实现的
2. 这些任务往往都需要运行相同的程序，但是程序的参数却需要根据不同的需求进行调整

### 8.2 并行计算的实现

- 实现方式：使用`Pool`包中的`map`函数，创建进程执行任务
- 传入`map`函数的第二个参数必须是可迭代的对象


```python
from multiprocessing import Pool
from functools import partial
import inspect
 
def parallal_task(func, iterable, cpu_count = 4):
 
    with open(f'./tmp_func.py', 'w') as file:
        file.write(inspect.getsource(func).replace(func.__name__, "task"))
 
    from tmp_func import task
 
    if __name__ == '__main__':
        func = partial(task)
        pool = Pool(cpu_count * 2)
        res = pool.map(func, iterable)
        pool.close()
        return res
    else:
        raise "Not in Jupyter Notebook"
```


```python
# 计算平方
def def_f(x):
    return x * x

for res in parallal_task(def_f, range(1, 101)):
    print(f'计算平方的结果是:{res}')
```

    计算平方的结果是:1
    计算平方的结果是:4
    计算平方的结果是:9
    计算平方的结果是:16
    计算平方的结果是:25
    计算平方的结果是:36
    计算平方的结果是:49
    计算平方的结果是:64
    计算平方的结果是:81
    计算平方的结果是:100
    计算平方的结果是:121
    计算平方的结果是:144
    计算平方的结果是:169
    计算平方的结果是:196
    计算平方的结果是:225
    计算平方的结果是:256
    计算平方的结果是:289
    计算平方的结果是:324
    计算平方的结果是:361
    计算平方的结果是:400
    计算平方的结果是:441
    计算平方的结果是:484
    计算平方的结果是:529
    计算平方的结果是:576
    计算平方的结果是:625
    计算平方的结果是:676
    计算平方的结果是:729
    计算平方的结果是:784
    计算平方的结果是:841
    计算平方的结果是:900
    计算平方的结果是:961
    计算平方的结果是:1024
    计算平方的结果是:1089
    计算平方的结果是:1156
    计算平方的结果是:1225
    计算平方的结果是:1296
    计算平方的结果是:1369
    计算平方的结果是:1444
    计算平方的结果是:1521
    计算平方的结果是:1600
    计算平方的结果是:1681
    计算平方的结果是:1764
    计算平方的结果是:1849
    计算平方的结果是:1936
    计算平方的结果是:2025
    计算平方的结果是:2116
    计算平方的结果是:2209
    计算平方的结果是:2304
    计算平方的结果是:2401
    计算平方的结果是:2500
    计算平方的结果是:2601
    计算平方的结果是:2704
    计算平方的结果是:2809
    计算平方的结果是:2916
    计算平方的结果是:3025
    计算平方的结果是:3136
    计算平方的结果是:3249
    计算平方的结果是:3364
    计算平方的结果是:3481
    计算平方的结果是:3600
    计算平方的结果是:3721
    计算平方的结果是:3844
    计算平方的结果是:3969
    计算平方的结果是:4096
    计算平方的结果是:4225
    计算平方的结果是:4356
    计算平方的结果是:4489
    计算平方的结果是:4624
    计算平方的结果是:4761
    计算平方的结果是:4900
    计算平方的结果是:5041
    计算平方的结果是:5184
    计算平方的结果是:5329
    计算平方的结果是:5476
    计算平方的结果是:5625
    计算平方的结果是:5776
    计算平方的结果是:5929
    计算平方的结果是:6084
    计算平方的结果是:6241
    计算平方的结果是:6400
    计算平方的结果是:6561
    计算平方的结果是:6724
    计算平方的结果是:6889
    计算平方的结果是:7056
    计算平方的结果是:7225
    计算平方的结果是:7396
    计算平方的结果是:7569
    计算平方的结果是:7744
    计算平方的结果是:7921
    计算平方的结果是:8100
    计算平方的结果是:8281
    计算平方的结果是:8464
    计算平方的结果是:8649
    计算平方的结果是:8836
    计算平方的结果是:9025
    计算平方的结果是:9216
    计算平方的结果是:9409
    计算平方的结果是:9604
    计算平方的结果是:9801
    计算平方的结果是:10000
    

### 8.3 并行计算提高效率的方法

1. 为并行程序自动指定并行度：根据计算机的CPU资源，设置合理的进程数量


```python
import psutil

# 逻辑cpu个数
count = psutil.cpu_count()
```

2. 统计程序运行的时间


```python
import time

# 并行计算时间统计
time1 = time.time()
for res in parallal_task(def_f, range(1, 10001)):
#     print(f'计算平方的结果是:{res}')
    pass

time2 = time.time()
print("并行计算时间统计:", str(time2 - time1))

# 串行计算时间统计
list1 = []

time1 = time.time()
for i in range(1, 10001):
    list1.append(def_f(i))
time2 = time.time()

print("串行计算时间统计:", str(time2 - time1))
```

    并行计算时间统计: 0.2190546989440918
    串行计算时间统计: 0.002999544143676758
    

## 9 总结

&emsp;&emsp;本篇章主要介绍了使用函数和字典方式进行多次替换、通过使用图像处理库进行长图拼接、使用jieba库进行情感分析、对跨文件进行字数统计、使用正则表达式进行精确搜索、文件搜索、使用排序函数进行排序检索、通过并行计算有效利用CPU资源；
1. 对于替换，可采用字符串的`replace`函数、字典映射、逻辑判断等方式
2. 长图拼接：通过使用ImageMagick图像处理软件和外部命令调用，实现长图拼接功能
3. 感情色彩的单词数量统计：使用`jieba`库，通过数据预处理、词性标注、词频统计、数据再次处理等过程，实现词性统计
4. 跨文件的字数统计：通过文件操作函数，统计多文件中的字数
5. 精确搜索：使用`re`库，配合正则表达式，达到精确搜索的目的
6. 文件搜索：使用`pathlib`库的`glob()`和`rglob()`函数，实现对文件的搜索，可使用指定搜索路径和建立索引的两种方法，提高文件搜索效率
7. 数据排序：根据不同场景，使用不同的数据类型，并通过`sorted`函数进行排序
8. 并行计算：通过`multiprocessing`库的`Pool`包可以实现基于进程的并行计算功能，`Pool`包的 `map()`函数会根据指定的进程数量实现并行运行
