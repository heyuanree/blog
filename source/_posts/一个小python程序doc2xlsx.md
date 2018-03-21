---
title: 一个小python程序doc2xlsx
tags:
  - python
categories: []
date: 2017-07-06 16:36:43
---

这破玩意因为编码问题坑了我看好久，还有就是dict的深浅拷贝的问题。

<!--more-->

```
import sys, os
import docx
import openpyxl
import chardet           # 检测编码格式用的，蛮好用
import re                     # 我的没用到很多
from win32com import client as wc                # 处理doc转docx
```

python处理doc的包
`pip install python-docx`
pthon处理excel的包
`pip install openpyxl`

在使用python-docx的包必须要将doc转换成docx，用`win32com`即可，这个包属于`pypiwin32`
安装方式`pip install pypiwin32`

使用的时候不知道为什么
```
import win32com
word = win32com.client.Dispatch('Word.Application')
```
会报错`client`找不到

只能够使用`from win32com import client as wc`的方式处理

下面的代码将doc转换成docx
```
def doc2docx(path, docname):
    word = wc.Dispatch('Word.Application')
    doc = word.Documents.Open(ur'{}'.format(os.path.join(path, docname)))
    newfilename = os.path.join(path, docname.split('.')[0] + '.docx')
    doc.SaveAs(ur'{}'.format(newfilename), 16)                  #16表示保存额为docx
    doc.Close()
    word.Quit()
    return newfilename
```

关于字典的深浅拷贝，直接应用别人的一段代码来说明
```
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
dict1 =  {'user':'runoob','num':[1,2,3]}
 
dict2 = dict1          # 浅拷贝: 引用对象
dict3 = dict1.copy()   # 浅拷贝：深拷贝父对象（一级目录），子对象（二级目录）不拷贝，还是引用
 
# 修改 data 数据
dict1['user']='root'
dict1['num'].remove(1)
 
# 输出结果
print(dict1)
print(dict2)
print(dict3)
```

全部代码，虽然只有90行，但是查了我不少资料，果然我python基本功还是不行。。
```
#coding=utf-8

import sys, os
import docx
import openpyxl
import chardet
import re
from win32com import client as wc

def doc2docx(path, docname):
    word = wc.Dispatch('Word.Application')
    doc = word.Documents.Open(ur'{}'.format(os.path.join(path, docname)))
    newfilename = os.path.join(path, docname.split('.')[0] + '.docx')
    doc.SaveAs(ur'{}'.format(newfilename), 16)
    doc.Close()
    word.Quit()
    return newfilename

def wordparse(path, docname):
    document = docx.Document(ur'{}'.format(os.path.join(path, docname)))
    lines = [paragraph.text.encode('utf-8') for paragraph in document.paragraphs ]
    # 图书编号	书名	试题编号	测试题题干	选项1	选项2	选项3	选项4	选项5	答案	选项数量
    col = {'bookno': None, 'bookname': None, 'problemno': None, 'problemcontent': None, 'chose1': None,
           'chose2': None, 'chose3': None, 'chose4': None, 'answer': None, 'answercount': 4}
    result = []
    groupproblem = []
    tmpresult = []
    for line in lines:
        if re.match(r'^\d', line):
            if groupproblem != []:
                tmpresult.append(groupproblem)
                groupproblem = []
                groupproblem.append(line)
            else:
                groupproblem.append(line)
        else:
            if line != '':
                groupproblem.append(line)
    for tmpproblem in tmpresult:
        tmpstr = ''
        tmpcol = col.copy()
        for line in tmpproblem:
            tmpstr += line
        try:
            content = tmpstr.find('、')
            k1 = tmpstr.find('（')
            k2 = tmpstr.find('）')
            A = tmpstr.find('A', k2)
            B = tmpstr.find('B', k2)
            C = tmpstr.find('C', k2)
            D = tmpstr.find('D', k2)
            tmpcol['bookno'] = 'B1-xiaoxue'
            tmpcol['bookname'] = '《传播网络正能量（小学版）》'
            tmpcol['problemno'] = tmpstr[:content]
            tmpcol['problemcontent'] = tmpstr[content+3:k1] + '____' + tmpstr[k2+3:A]
            tmpcol['chose1'], tmpcol['chose2'], tmpcol['chose3'], tmpcol['chose4'] = tmpstr[A:B], tmpstr[B:C], tmpstr[C:D], tmpstr[D:]
            tmpcol['answer'] = tmpstr[k1+3:k2]
            result.append(tmpcol)
        except:
            print tmpstr
            exit()
    return result

def write2xlsx(filename, data):
    workbook = openpyxl.Workbook()
    worksheet = workbook.create_sheet('problem', 0)
    colname = ['bookno', 'bookname', 'problemno', 'problemcontent', 'chose1',
           'chose2', 'chose3', 'chose4', 'answer', 'answercount']
    i, j = 1, 1
    for col in data:
        j = 1
        col['problemno'] = i
        for name in colname:
            try:
                worksheet.cell(row=i, column=j, value=col[name])
            except:
                print col['problemno'], name, col[name]
                exit()
            j += 1
        i += 1
    workbook.save('{}'.format(filename))
    return


if __name__ == '__main__':
    path, filename = os.path.split(sys.argv[1])[0], os.path.split(sys.argv[1])[1]
    suffix = filename.split('.')[1]
    if suffix == 'doc':
        newfilename = doc2docx(path, filename)
    else:
        newfilename = filename

    data = wordparse(path=path, docname=newfilename)
    write2xlsx(sys.argv[2], data)
```