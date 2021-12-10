# 技术书籍阅读笔记 {docsify-ignore-all}
记录本人的技术书籍阅读笔记，包括笔记、总结和思维导图。

## 在线阅读地址
在线阅读地址：https://relph1119.github.io/TechBooks-ReadingNote

## 目录
- QCon2019（广州站）参会总结
- 《凤凰项目》读书笔记
- 《架构整洁之道》读书笔记
- 《编码:隐匿在计算机软硬件背后的语言》读书笔记
- 《科学家列传》（壹）读书笔记
- 极客时间《Python自动化办公》学习笔记

## 运行环境
### Python版本
Mini-Conda Python 3.8 Windows环境

### Notebook运行环境配置
安装相关的依赖包
```shell
conda install --yes --file requirements.txt
```

### 安装Tesseract（用于离线文字识别）  
可参考博客：https://blog.csdn.net/guliang21/article/details/86735822

### 安装ImageMagick（用于长图拼接）
可参考博客（Windows）：https://blog.csdn.net/qq_37674858/article/details/80361860

### Conda批量导出环境中所有组件
```shell
conda list -e > requirements.txt
```

### 本地启动docsify
```shell
docsify serve ./docs
```

## 学习资料
【1】[Python 自动化办公实战课](https://time.geekbang.org/column/intro/100071001?tab=catalog)  
【2】[QCon2019（广州站）](https://time.geekbang.org/course/intro/100028701?tab=catalog)