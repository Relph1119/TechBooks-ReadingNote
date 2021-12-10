# Section04 “存储”模块：和文件相关的常用操作

## 1 循环与文件目录管理：如何实现文件的批量重命名？

### 1.1 使用`os`库实现文件批量重命名


```python
import os

def rename_file(file_path, file_suffix, verbose= 0):
    # 取得指定文件夹下的文件列表
    old_names = os.listdir(file_path)
    # 新文件名称从1开始
    new_name = 1

    # 取得所有的文件名
    for old_name in old_names:
        # 根据扩展名，判断文件是否需要改名
        if old_name.endswith(file_suffix):
            # 完整的文件路径
            old_path = os.path.join(file_path, old_name)

            # 新的文件名
            new_path = os.path.join(file_path, str(new_name)+".JPG")

            # 重命名
            os.rename(old_path, new_path)

            # 文件名数字加1
            new_name = int(new_name)+1
    if (verbose == 1):
        # 显示改名后的结果
        print(os.listdir(file_path))
```

### 1.2 增加命令行解析函数


```python
def args_opt(): 
    """获取命令行参数函数""" 
    
    # 定义参数对象 
    parser = argparse.ArgumentParser() 
    
    # 增加参数选项、是否必须、帮助信息 
    parser.add_argument("-p", "--path", required=True, help="path to rename") 
    parser.add_argument("-e", "--ext", required=True, help="files name extension, eg: jpg") 
    
    # 返回取得的所有参数 
    return parser.parse_args()
```

### 1.3 代码重构

1. 通过使用函数增加代码的逻辑性
2. 通过“name”变量增加了程序入口，便于直接找到程序开始执行的位置
3. 通过增加命令行参数，不用修改代码，就能实现函数的参数的修改。

## 2 不同操作系统下，如何通过网络同步文件？

### 2.1 实现文件的浏览和下载

- 通过命令行运行模块：使用`-m`参数，运行Python的模块
- 使用`http.server`模块提供HTTP服务：基于HTTP协议实现的文件浏览和下载功能，由于客户端服务端都采用HTTP协议，那么服务端列出的文件目录会自动被浏览器翻译给客户端的用户，也就能通过浏览器查看到服务器上的文件名称，并把服务器的文件下载到客户端的电脑上

!python3 -m http.server 8080

### 2.2 实现文件的上传

- 两种请求 HTTP 服务器的方式：
  1. GET 方式一般用于获取服务器的信息，类似从服务器上查找数据
  2. POST 方式一般用于向服务器上传信息，类似向服务器写入
- 使用`Flask`库运行服务器，并提供服务请求


```python
import os
from flask import Flask, request

# 初始化
app = Flask("Download_Picture")
# 配置App对象
app.config['UPLOAD_FOLDER'] = os.getcwd()

html = '''
    <!doctype html>
    <title>Upload new File</title>
    <h1>Upload new File</h1>
    <form action="" method=post enctype=multipart/form-data>
      <p><input type=file name=file>
         <input type=submit value=Upload>
    </form>
    '''

# 把HTML的代码传递给游览器
@app.route('/', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        file = request.files['file']
        filename = file.filename
        file.save(os.path.join(app.config['UPLOAD_FOLDER'], filename))
    return html
```


```python
# 设置应用端口为8090
app.run(host='0.0.0.0', port=8090)
```

     * Serving Flask app '__main__' (lazy loading)
     * Environment: production
    [31m   WARNING: This is a development server. Do not use it in a production deployment.[0m
    [2m   Use a production WSGI server instead.[0m
     * Debug mode: off
    

     * Running on all addresses.
       WARNING: This is a development server. Do not use it in a production deployment.
     * Running on http://127.0.0.1:8090/ (Press CTRL+C to quit)
    

## 3 http库：如何批量下载在线内容，解放鼠标（上）？

### 3.1 实现批量下载图片

1. 访问 HTTP 服务器，得到搜索结果的整个网页
2. 在访问服务器之后下载一张图片
3. 找到多张图片的相似地址
4. 提取相似地址，下载多张图片

### 3.2 访问HTTP服务端的资源

使用`requests-html`库实现访问HTTP服务器：
1. 设置请求的URL
2. 启动一个会话
3. 发送`GET`请求
4. 得到返回结果


```python
from requests_html import HTMLSession

# 需要访问的服务器URL
name = "猫"
url = f"https://unsplash.com/s/photos/{name}"

# 启动会话
session = HTMLSession()

# 向upsplash网站提供的HTTPS服务器发起“GET”请求
result = session.get(url)

# 得到返回状态码
print(result.status_code)
# 得到返回的网页源代码
# print(result.html.html)
```

    200
    

### 3.2 使用`requests-html`下载一张图片


```python
from requests_html import HTMLSession
import os

# URL
url = "https://unsplash.com/photos/NLzaiXOELFY/download"

# 启动
session = HTMLSession()

# GET请求
result = session.get(url)

# 结果
print(result.status_code)

if not os.path.exists("data/ch18"):
    os.makedirs("data/ch18")

# 保存图片
with open("data/ch18/one.jpg", "wb") as f:
    # 使用二进制方式获取图片内容
    f.write(result.content)
```

    200
    

### 3.3 批量下载图片

**实现思路：**  
通过上述请求网页和下载图片的两个功能，组合两个功能函数，找到多张图片之间的`HTML`代码的规律，实现批量下载。


```python
# 需要访问的服务器URL
name = "猫"
url = f"https://unsplash.com/s/photos/{name}"

# 启动会话
session = HTMLSession()

# 向upsplash网站提供的HTTPS服务器发起“GET”请求
result = session.get(url)
```


```python
# 使用 Xpath 匹配多张图片的下载地址
down_list = result.html.xpath('//figure[@itemprop="image"]//a[@rel="nofollow"]/@href')
```


```python
# 得到图片ID
def get_picID_from_url(url):
    return url.split('/')[4] + ".jpg"
```


```python
# 下载图片
def down_one_pic(url):
    result = session.get(url)
    filename = get_picID_from_url(url)
    with open("data/ch18/" + filename, "wb") as f:
        f.write(result.content)
```


```python
for one_url in down_list:
    down_one_pic(one_url) 
```

## 4 http库：如何批量下载在线内容，解放鼠标（下）？

### 4.1 selenium的适用场景

- 适用场景：解决无法通过工具下载资源、需要通过JavaScript脚本语言才能获得服务器数据
- 实现原理：通过`WebDriver`组件，把Python与浏览器连接起来，让Python来控制浏览器的行为，向浏览器发送各种模拟用户操作的指令


```python
from selenium import webdriver
import time

# 浏览器初始化
browser =  webdriver.Chrome()

# 控制浏览器行为
browser.get("http://www.jd.com")

# 获取网页的源代码
content = browser.page_source
time.sleep(10)
browser.quit()
```

### 4.2 实现京东自动签到

- 原理：通过对浏览器的功能拆解，把浏览器的交互行为，一一对应到“selenium”的非交互命令，之后就能实现自动签到
- 实现思路：
    1. 打开登录页面
    2. 切换到用户密码登录选项卡
    3. 点击登录按钮

### 4.3 使用“selenium”模拟浏览器，实现自动登录


```python
from selenium import webdriver
import time

browser =  webdriver.Chrome()

# 访问主页
browser.get("http://www.jd.com")
time.sleep(2)

# 访问登录页
browser.get("https://passport.jd.com/new/login.aspx?ReturnUrl=https%3A%2F%2Fwww.jd.com%2F")
time.sleep(2)

# 切换为用户密码登录
r = browser.find_element_by_xpath(
    '//div[@class="login-tab login-tab-r"]')
browser.execute_script('arguments[0].click()', r)
time.sleep(2)

# 发送要输入的用户名和密码
browser.find_element_by_xpath(
    "//input[@id='loginname']").send_keys("username")
time.sleep(1)
for i in "password":
    browser.find_element_by_xpath(
         "//input[@id='nloginpwd']").send_keys(i)
    time.sleep(1)

# 点击登录按钮
browser.find_element_by_xpath(
    '//div[@class="login-btn"]/a').click()
time.sleep(2)
```

### 4.4 利用“selenium”，实现自动签到


```python
# 访问签到页面
browser.get("https://mall.jd.com/index-1000002826.html")
time.sleep(2)

# 签到并领金豆
browser.find_element_by_xpath('//div[@class="jSign"]/a').click()
time.sleep(10)

# 退出浏览器
browser.quit()
```

## 5 不同文件混在一起，怎么快速分类？

### 5.1 设计合理的数据类型

- 分类规则：将扩展名和要移动的目录建立对应关系


```python
# 适用字典定义文件类型和它的扩展名
file_type = {
    "music": ("mp3", "wav"),
    "movie": ("mp4", "rmvb", "rm", "avi"),
    "execute": ("exe", "bat")
}
```

### 5.2 设计生产者消费者模式

- 概念：有两个进程共用一个缓冲区，两个进程分别是生产数据和消费数据的。而缓冲区，用于存放生产进程产生的数据，并让消费进程从缓冲区读取数据进行消费。

- 文件分类思路：把读取当前文件名称和路径函数作为生产者，把分类和移动文件的逻辑作为消费者。在生产者消费者中间，使用队列作为它们中间的缓冲区

- 好处：
    1. 如果生产者比消费者快，可以把多余的生产数据放在缓冲区中，确保生产者可以继续生产数据。
    2. 如果生产者比消费者慢，消费者处理完缓冲区中所有数据后，会自动进入到阻塞状态，等待继续处理任务。
    3. 缓冲区会被设置为一定的大小，当生产者的速度远远超过消费者，生产者数据填满缓冲区后，生产者也会进入到阻塞状态，直到缓冲区中的数据被消费后，生产者才可以继续写入。而当消费性能不足时，可以等待消费者运行，减少生产者和消费者在进度上相互依赖的情况。

### 5.3 分类实现

1. 创建分类需要的文件夹
2. 遍历目录并写入队列
3. 分类并移动到新的文件夹


```python
import os

# 定义文件类型和它的扩展名
file_type = {
    "music": ("mp3", "wav"),
    "movie": ("mp4", "rmvb", "rm", "avi"),
    "execute": ("exe", "bat")
}

source_dir = "data/ch19/files"

# 创建分类需要的文件夹
def make_new_dir(dir, type_dir):
    for td in type_dir:
        new_td = os.path.join(dir, td)
        if not os.path.isdir(new_td):
            os.makedirs(new_td)

# 建立新的文件夹
make_new_dir(source_dir, file_type)
```


```python
from queue import Queue

# 遍历目录并存入队列
def write_to_q(path_to_write, q: Queue):
    for full_path, dirs, files in os.walk(path_to_write):
        # 如果目录下没有文件，就跳过该目录
        if not files:
            continue
        else:
            # 将文件的完整路径和该路径下的文件列表放到缓冲区中
            q.put(f"{full_path}::{files}")

source_dir = "data/ch19/files"

# 定义一个用于记录扩展名放在指定目录的队列
filename_q = Queue()

# 遍历目录并存入队列
write_to_q(source_dir, filename_q)
```


```python
import shutil

# 移动文件到新的目录
def move_to_newdir(filename_withext, file_in_path, type_to_newpath):
    # 取得文件的扩展名
    filename_withext = filename_withext.strip(" \'")
    ext = filename_withext.split(".")[1]

    for new_path in type_to_newpath:
        if ext in type_to_newpath[new_path]:
            oldfile = os.path.join(file_in_path, filename_withext)
            newfile = os.path.join(source_dir, new_path, filename_withext)
            shutil.move(oldfile, newfile)

# 将队列的文件名分类并写入新的文件夹
def classify_from_q(q: Queue, type_to_classify):
    while not q.empty():
        # 从队列里取目录和文件名
        item = q.get()

        # 将路径和文件分开
        filepath, files = item.split("::")
        
        # 剔除文件名字符串出现的"[" "]",并用"，"做分隔转换为列表
        files = files.strip("[]").split(",")
        # 对每个文件进行处理
        for filename in files:
            # 将文件移动到新的目录
            move_to_newdir(filename, filepath, type_to_classify)
```


```python
# 将队列的文件名分类并写入新的文件夹 
classify_from_q(filename_q, file_type)
```

## 6 SQLite文本数据库：如何进行数据管理（上）？

### 6.1 SQLite介绍

- 特性：具有大型数据库的稳定、高效、支持 SQL 语言的特性，比大型数据库学习起来更加简单
- 优势：
    1. 数据查询速度快
    2. 存放数据的空间占用少
    3. 实现了一般数据库能够支持的（基于SQL语言的）增删改查

### 6.2 SQLite 建立数据表的一般流程

1. 连接数据库文件
2. 创建游标：又称为操作行指针，表示当前选中的行
3. 执行SQL语句
4. 关闭游标和连接


```python
import sqlite3
import pathlib
import os

# 数据库文件的路径和文件名称
dst_file = 'data/ch21/contents.db'

if not os.path.exists(pathlib.PurePath(dst_file).parent):
    os.makedirs(pathlib.PurePath(dst_file).parent)

db = pathlib.PurePath(dst_file)

# 创建连接
conn = sqlite3.connect(db)

# 创建游标
cur = conn.cursor()
```


```python
# 定义要执行的SQL语句
sql = '''CREATE TABLE address_book(
        id INT PRIMARY KEY NOT NULL,
        name TEXT NOT NULL,
        phone INT NOT NULL
       )'''

# 执行SQL
try:
    cur.execute(sql)
    print("创建成功")
except Exception as e:
    print("创建失败")
    print(f"失败原因是：{e}")
```

    创建成功
    

### 6.3 数据的写入

示例：为通讯录添加一个联系人`Tom`


```python
# 定义要执行的SQL语句
sql1 = '''INSERT INTO address_book VALUES (?, ?, ?)'''
v = (1, "Tom", 12377778888)

# 执行SQL
try: 
    cur.execute(sql1, v) 
    conn.commit()
except Exception as e: 
    print(f"失败原因是：{e}")
```

### 6.4 数据的查询


```python
# 定义要执行的SQL语句
sql2 = '''SELECT phone FROM address_book WHERE name = "Tom" ''' 
# 执行SQL
try: 
    result = cur.execute(sql2) 
    print(result.fetchone())
except Exception as e: 
    print(f"失败原因是：{e}")
```

    (12377778888,)
    


```python
# 关闭游标 
cur.close() 
# 关闭连接 
conn.close()
```

## 7 SQLite文本数据库：如何进行数据管理（下）？

### 7.1 使用类实现`SQLite`的读写

**实现思路：**  
使用“类”对 SQLite 的读写 SQL 操作进行封装，并将类进行实例化以后进行调用，得到 SQLite 中的通讯录数据


```python
import sqlite3
import pathlib

class OptSqlite(object):
    def __init__(self, dbname = "new.db"):
        """
        :param dbname  数据库名称
        """
        self.dir = "data/ch21"
        self.db = pathlib.PurePath(self.dir, dbname)
        self.conn = sqlite3.connect(self.db)
        self.cur = self.conn.cursor()

    def close(self):
        """
        关闭连接
        """
        self.cur.close()
        self.conn.close()

    def get_one_phone(self, username):
        """
        获取一个联系人的电话
        """

        self.get_user_phone_sql = f"""
            SELECT phone FROM address_book WHERE name = "{username}" """
        try:
            self.result = self.cur.execute(self.get_user_phone_sql)
            return self.result.fetchone()
        except Exception as e:
            print(f"失败原因是：{e}")

    def set_one_phone(self, name, phone):
        """
        增加一个联系人
        """
        self.set_user_phone_sql = '''INSERT INTO address_book
          VALUES (?, ?, ?)'''
        self.v =  (2, str(name), int(phone))
        try:
            self.cur.execute(self.set_user_phone_sql, self.v)
            self.conn.commit()
        except Exception as e:
            print(f"失败原因是：{e}")
```


```python
my_query = OptSqlite("contents.db")

my_query.set_one_phone("Jerry","12344445555")

phone = my_query.get_one_phone("Tom")
phone2 = my_query.get_one_phone("Jerry")    

my_query.close()

print(phone)
print(phone2)
```

    (12377778888,)
    (12344445555,)
    

### 7.2 类和自定义函数的区别

1. 对代码的封装方式上不同：编写自定义函数，实现思路是通过函数去描述程序运行的过程；编写基于类的程序，实现思路要针对相同的一类数据，具有的属性和相同的动作
2. 语法结构不同：函数是通过`def`关键字定义的，而类是通过`class`关键字定义的
3. 调用方式不同：主要表现在各自成员能否被访问和运行方式

### 7.3 实现增删改查的类


```python
import sqlite3
import pathlib

class OptSqlite(object):
    def __init__(self, dbname = "new.db"):
        """
        :param dbname  数据库名称
        """
        self.dir = "data/ch21"
        self.db = pathlib.PurePath(self.dir, dbname)
        self.conn = sqlite3.connect(self.db)
        self.cur = self.conn.cursor()

    def close(self):
        """
        关闭连接
        """
        self.cur.close()
        self.conn.close()

    def new_table(self, table_name):
        """
        新建联系人表
        """
        sql = f'''CREATE TABLE {table_name}(
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            phone INT NOT NULL
            )'''

        try:
            self.cur.execute(sql)
            print("创建表成功")
        except Exception as e:
            print("创建表失败")
            print(f"失败原因是：{e}")

    def get_one_phone(self, username):
        """
        获取一个联系人的电话
        """
        self.get_user_phone_sql = f"""
            SELECT phone FROM address_book WHERE name = "{username}" """
        try:
            self.result = self.cur.execute(self.get_user_phone_sql)
            return self.result.fetchone()
        except Exception as e:
            print(f"失败原因是：{e}")

    def get_all_contents(self):
        """
        取得所有的联系人
        """
        try:
            self.result = self.cur.execute("SELECT * FROM address_book")
            return self.result.fetchall()
        except Exception as e:
            print(f"失败原因是：{e}")

    def set_one_phone(self, name, phone):
        """
        增加或修改一个联系人的电话
        """
        if self.get_one_phone(name):
            self.set_user_phone_sql = '''UPDATE address_book 
            SET phone= ? WHERE name=?'''
            self.v =  (int(phone), str(name))
        else:
            self.set_user_phone_sql = '''INSERT INTO address_book
            VALUES (?, ?, ?)'''
            self.v =  (None, str(name), int(phone))
        try:
            self.cur.execute(self.set_user_phone_sql, self.v)
            self.conn.commit()
        except Exception as e:
            print(f"失败原因是：{e}")

    def delete_one_content(self, name):
        """
        删除一个联系人的电话
        """
        self.delete_user_sql = f'''DELETE FROM address_book 
                WHERE name="{name}"'''

        try:
            self.cur.execute(self.delete_user_sql)
            self.conn.commit()
        except Exception as e:
            print(f"删除失败原因是：{e}")
```


```python
# 实例化
my_query = OptSqlite("contents.db")

# 创建一张表
# my_query.new_table("address_book")

# 增加或修改一个联系人的电话
my_query.set_one_phone("Jerry","12344445556")

# 查询一个联系人的电话
phone = my_query.get_one_phone("Jerry")    
print(phone)

# 查询所有人的电话
contents = my_query.get_all_contents()
print(contents)

# 删除一个联系人
my_query.delete_one_content("Jerry")

contents = my_query.get_all_contents()
print(contents)   

# 关闭连接
my_query.close()
```

    (12344445556,)
    [(1, 'Tom', 12377778888), (2, 'Jerry', 12344445556)]
    [(1, 'Tom', 12377778888)]
    

## 8 总结

&emsp;&emsp;本篇章主要介绍了文件的批量重命名、通过网络同步文件、实现批量下载在线文件、文件分类、操作`SQLite`数据库进行数据管理；
1. 通过`os.rename()`函数，实现文件重命名，并通过遍历文件夹的文件列表，实现文件的批量重命名
2. 使用`Flask`库，创建运行程序，并使用`GET`和`POST`请求，完成文件的上传功能
3. 利用`requests-html`和`XPath`进行网页内容搜索，实现批量下载图片功能；使用`Webdriver`和`selenium`，结合浏览器模拟，把浏览器的点击链接、用户登录、切换标签等常用功能转换为用Python可以控制的操作
4. 通过字典类型设计数据结构，并通过函数设计合理的功能封装，使用生产者消费者模式，实现文件分类功能
5. 使用`sqlite3`库，通过连接数据库文件、创建游标、执行SQL语句、关闭游标和连接等步骤，完成对通讯录数据的管理
