# 0x00 背景

## 1、前言

在工作中很多时候，都会遇到对文件进行处理。每次都是搜索python文件处理相关的库，或者拿来主义，直接在别人的代码基础上直接改造，周而复始，每次都浪费大把时间。因此在这里大概总结一下文件相关的一些操作。



## 2、常见的文件操作



* 文件读写
* 获取文件属性：创建时间、大小、文件类型、最后修改时间等
* 创建、删除、移动、复制等
* 遍历目录： 遍历根目录下的所有文件、所有文件夹、判断文件是文件夹还是文件
* 文件名模式匹配
* 归档



# 0x01  文件操作相关的库

## 1、python 文件操作库

| 模块名称  | 模块概要                                                     |
| --------- | ------------------------------------------------------------ |
| os        | python  内置的函数                                           |
| sys       |                                                              |
| pathlib   |                                                              |
| SFlock    |                                                              |
| datetime  |                                                              |
| fnmatch   | 有对于模式匹配有更先进的函数和方法                           |
| glob      |                                                              |
| tempfile  | 模块来便捷的创建临时文件和目录                               |
| shutil    | `shutil` 是shell实用程序的缩写。 它为文件提供了许多高级操作，来支持文件和目录的复制，归档和删除 |
| zipfile   | 是一个底层模块，是Python标准库的一部分。可以轻松打开和提取ZIP文件的函数 |
| tarfile   | `TarFile` 类允许读取和写入TAR存档                            |
| fileinput | 从多个输入流或文件列表中读取数据                             |

## 2、模块详细介绍



### 1、os库

### 2、pathlib库

### 3、SFlock库

### 4、datetime库

### 5、fnmatch 库

### 6、glob 库

### 7、tempfile 模块

# 0x02 具体案例

## 0、文件读写

```python 
with open() as f:
	f.read()
	f.write()
```



## 1、 获取文件属性

## 2、目录遍历

**使用OS模块遍历文件：**

```python 
sub_dirs = os.walk(file_path)
for root, dirs,  files in sun_dirs:  
	# root: 根目录（字符串）  dirs: 子目录（数组）  files： 文件名（字符串）
    # 输出子目录列表
    print（dirs） 
    for f in files:
        # 输出文件名
    	print(files)  
        # 输出全路径 
    	print(os.path.join(root, files))  
```



## 3、文件移动、符指、重命名



## 4、文件名模式匹配

使用上述方法之一获取目录中的文件列表后，你可能希望搜索和特定的模式匹配的文件。

- `endswith()` 和 `startswith()` 字符串方法
- `fnmatch.fnmatch()`
- `glob.glob()`
- `pathlib.Path.glob()`