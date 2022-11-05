---
typora-copy-images-to: ..\images
---

# 1 csv文件的概念

csv是“Comma-Separated Values(逗号分割的值)”的首字母缩写，它其实和txt文件一样，都是纯文本文件。但csv文件可以显示为电子表格的样式，所以我们也可以把csv文件视为一种简化版的电子表格。



csv是纯文本文件，它只能存储数据。它有多种打开方式，如果你使用文本编辑器（如记事本）打开它，它就会以文本形式显示；但如果你使用常用的office软件打开，csv就会直接显示为表格样式。

![](https://upload-images.jianshu.io/upload_images/15336533-72329da02999ca26.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

如果只是记录数据，不对数据进行操作的话，相较于Excel表格，csv文件会更加简洁轻便。使用python来读写csv文件是非常容易的，因为实现csv的读取和写入功能的csv模块，是一个内置模块，我们可以直接使用。

# 2 python中创建csv文件

## 2.1 使用csv.writer()创建

```python
import csv

headers = ['学号', '姓名', '分数']
rows = [('202001', '张三', '98'), ('202002', '李四', '95'), ('202003', '王五', '92')]

with open('score.csv', 'w', encoding='utf-8', newline='') as f:
	writer = csv.writer(f)
	writer.writerow(headers)
	writer.writerows(rows)
```

## 2.2 使用csv.Dictwriter()创建

```python
import csv

headers = ['学号', '姓名', '分数']
rows = [{'学号':'202001', '姓名':'张三', '分数':'98'}, ('学号':'202002', '姓名':'李四', '分数':'95'), ('学号':'202003', '姓名':'王五', '分数':'92')]

with open("score.csv", "w", encoding='utf-8',newline='') as f:
	writer = csv.DictWriter(f, headers)
	writer.writeheader()
	writer.writerows(rows)

```

## 2.3 使用writeline()创建

```python
import csv

headers = ['学号,姓名,分数', '\n']
csv = ['202001,张三,98', '\n',
'202002,李四,95', '\n'
'202003,王五,92']

with open('score.csv', 'w', encoding='utf-8',newline='') as f:
	f.writelines(headers)
	f.writelines(csv)
```

# 3 python中读取csv文件

## 3.1 使用pandas.read_csv()读取

```python
import pandas as pd

my_matrix = pd.read_csv('score.csv') # header=None,index_col=None
"""
header:指定行数用来作为列名，数据开始行数。如果文件中没有列名，则默认为0
index_col：用来行索引的列编号或者列名
"""
print(my_matrix)
print(my_matrix.shape)

# 为了后续分析，可以将数据类型改为np.array型
 my_matrix = np.array(my_matrix)

```

## 3.2 使用csv.reader()进行读取

```python
import csv

with open('score.csv', "r", encoding="utf-8",newline='') as f:
	reader = csv.reader(f)
	for row in reader:
		print(row)
```

## 3.3 使用csv.DictReader()进行读取

```python
import csv

with open("score.csv", "r", encoding='utf-8', newline='') as f:
	reader = csv.DictReader(f)
	for row in reader:
		print(row)
```

