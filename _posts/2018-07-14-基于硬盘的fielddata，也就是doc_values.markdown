---
layout: post
title: doc_values，基于硬盘的fielddata
date: 2018-07-14 22:39:20 +0800
description: 简介Elasticsearch 中的fielddata数据结构
img: i-rest.jpg # Add image post (optional)
tags: [Elasticsearch]
---
fielddata在Lucene中又叫做 uninverted index，uninvrted index 是为了加速聚合，排序计算的一种索引数据结构。fielddata默认在在内存中，但是由于需要对大量的文档进行聚合，排序等操作，所以将索引全部保存到内存中有点不切实际。于是es引入了一种新特性：doc_values。

## doc_values

所谓doc_values，就是在es将数据写入索引（这里指的是es中的索引数据库）的时候，提前将fielddata的内容生成放到硬盘上。因为fileddata的数据是顺序读写的，所以即使在硬盘上，**通过文件系统层面的缓存**，也可以获得相当不错的性能。
doc_values在硬盘中使用列式存储，例如：
我有以下三条记录


```
[ {Field1: 1, Field2:42}, {Field1: 5, Field2:12}, {Field1: 5, Field2:6} ]

# 则列式存储的方式则是

{ A: [1, 5, 5], B: [42, 12, 6]}
```

表格中则可以表示为：

|      | A | B  |
|------|---|----|
| Doc1 | 1 | 42 |
| Doc2 | 5 | 12 |
| Doc3 | 5 | 6  |



这样做的好处除了易于排序以外，还有利于压缩和充分利用CPU，文件系统缓存。坏处就是与行式存储对比起来，在查找一个文档的所有字段时会很慢。

## 什么时候需要doc_values：

- 排序操作
- 当执行聚合计算的时候，需要某一个字段的具体值进行统计，Max，Min计算。


## 在es中开启字段的doc_value属性
template mapping
```json
"myfieldname": {
	"type": "string",
	"index": "not analyzed",	# 可索引，但不进行分词
	"doc_values" : true
}
# 在es5.0之后，加入了新的数据类型：keyword，等同于上边的表达式。
```




