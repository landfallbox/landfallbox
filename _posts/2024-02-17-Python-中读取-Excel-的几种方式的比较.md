---
layout: article
title: Python 中读取 Excel 的几种方式的比较
article_header: 
  type: cover
  image:
    src: https://raw.githubusercontent.com/landfallbox/Pictures/master/202402172031868.png
tags: 
    - Java
---

Python 中读取 Excel 一般通过借助一些库实现，比如 Pandas、Tablib、Openpyxl、LibreOffice、DuckDB、Calamine。

# Pandas

```python
from typing import IO, Iterator
import pandas as pd

def iter_excel_pandas(file: IO[bytes]) -> Iterator[dict[str, object]]:
    yield from pd.read_excel(file).to_dict('records')
    
with open('file.xlsx', 'rb') as f:
    rows = iter_excel_pandas(f)
    row = next(rows)
    print(row)
```

使用 Pandas 读取 Excel 文件时，先将文件以二进制格式打开，借助 `pd.read_excel` 读取文件，并用 `to_dict` 将读取到的数据转换为字典格式。`records` 参数表示每行数据都将转换为一个字典，列名作为键，单元格内容作为值。最后借助 `yield from` 返回一个生成器，借此逐行读取数据，而不是一次性加载整个数据集。

值得注意的是，使用 Pandas 读取 Excel 文件时，日期的返回格式是 Pandas 的 `Timestamp` ，而不是 `datetime.date` 。当然也可以手动转换：

```py
def iter_excel_pandas(file: IO[bytes]) -> Iterator[dict[str, object]]:
    yield from pandas.read_excel(file, converters={
        'date': lambda ts: ts.date(),
    }).to_dict('records')
```

一般来说，如果使用 Pandas 读取数据，自然会用 Pandas 完成后续的数据处理，那么即使日期的格式是 Pandas 的 `Timestamp` 也应当是无伤大雅的。

经过测试，对于一个 25 MB 大小有 500K 行数据的 Excel 文件而言，使用 Pandas 读取数据需要大约 32 秒。

# Tablib

```python
def iter_excel_tablib(file: IO[bytes]) -> Iterator[dict[str, object]]:
    yield from tablib.Dataset().load(file).dict
    
with open('file.xlsx', 'rb') as f:
    rows = iter_excel_tablib(f)
    row = next(rows)
    print(row)
```

使用 Tablib 读取 Excel 文件的代码与使用 Pandas 读取 Excel 文件的代码是类似的，不作过多解释。

使用 Tablib 读取 Excel 文件时得到的返回值类型是 `OrderedDict` 而不是 `Dict` ，但 `OrderedDict` 是 `Dict` 的子类，一般而言不影响使用。此外，使用 Tablib 读取 Excel 文件时日期的类型被转换为 `datetime.datetime` 而不是期望的 `datetime.date` 。这是一个很大的问题。

经过测试，对于一个 25 MB 大小有 500K 行数据的 Excel 文件而言，使用 Tablib 读取数据需要大约 28 秒。

# Openpyxl

Openpyxl 是一个专用于读写 Excel 文件的库，并且在底层上无论 Pandas 还是 Tablib 都使用了 Openpyxl。

```python
def iter_excel_openpyxl(file: IO[bytes]) -> Iterator[dict[str, object]]:
    workbook = openpyxl.load_workbook(file)
    rows = workbook.active.rows
    headers = [str(cell.value) for cell in next(rows)]
    for row in rows:
        yield dict(zip(headers, (cell.value for cell in row)))
        
with open('file.xlsx', 'rb') as f:
    rows = iter_excel_openpyxl(f)
    row = next(rows)
    print(row)
```

令人吃惊的是，经过测试，对于一个 25 MB 大小有 500K 行数据的 Excel 文件而言，使用 Openpyxl 读取数据需要大约 35 秒。这甚至超过了 Pandas 和 Tablib 的读取时间，二者一个花费大约 32 秒，一个花费大约 28 秒。

查阅官方文档得知，可以通过将文件指定为以只读或只写的模式打开以提高性能。

```python
def iter_excel_openpyxl(file: IO[bytes]) -> Iterator[dict[str, object]]:
    workbook = openpyxl.load_workbook(file, read_only=True)
    rows = workbook.active.rows
    headers = [str(cell.value) for cell in next(rows)]
    for row in rows:
        yield dict(zip(headers, (cell.value for cell in row)))
        
with open('file.xlsx', 'rb') as f:
    rows = iter_excel_openpyxl(f)
    row = next(rows)
    print(row)
```

经测试，对于一个 25 MB 大小有 500K 行数据的 Excel 文件而言，只读模式下使用 Openpyxl 读取数据需要的时间从大约 35 秒降到了大约 24 秒，实现了比 Pandas 和 Tablib 更高的效率。

# 参考资料

https://hakibenita.com/fast-excel-python