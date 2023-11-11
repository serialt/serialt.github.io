+++
title = 'Go leveldb'
date = 2023-11-11T10:24:52+08:00
draft = false

tags = ["Go","leveldb"]
categories = ["Go 库文档"]

+++

## Go leveldb库



github地址：https://github.com/syndtr/goleveldb



安装：

```
go get github.com/syndtr/goleveldb/leveldb
```



### 简介：

levelDB在区块链中比较常用，其是Google开源的持久化单机Key-Value文件数据库，其支持按照文件大小切分文件的功能。levelDB具有很高的随机写，顺序读/写性能，但是随机读的性能很一般，也就是说，**levelDB很适合应用在查询较少，而写很多的场景。**



### LevelDB特点

1）key和value都是任意长度的字节数组；
2）entry（即一条k-v记录）默认是按照key的**字典顺序**存储的，开发者也可以重写这个方法；
3）提供了基本的增删改查接口；
4）支持批量操作以原子操作进行；
5）开源创建数据全景的snapshot（快照），并允许在快照中查询；
6）开源通过向前（后）迭代器遍历数据（迭代器隐含的创建了一个snapshot）；
7）自动使用Snappy压缩数据；
8）可移植性。

### levelDB限制

1）NoSQL，不支持sql语句，也不支持索引；
2）一次只允许一个进程访问一个特定的数据库；
3）没有内置的C/S架构，开发者需要使用levelDB库自己封装一个server；



### 使用：

1）打开、创建数据库

```go
db, err := leveldb.OpenFile("./block.db", nil)
```

2）写入key数据

```go
err = db.Put([]byte("hello"), []byte("world"), nil)
```

3）读取key数据

```go
data, _ := db.Get([]byte("hello"), nil)  
```

4）遍历数据库

```go
iter := db.NewIterator(nil, nil)  
for iter.Next() {  
    logger.Debug(iter.Key() + iter.Value())  
}  
```

5）读取某个前缀的所有KEY数据

读出来的数据会被放进一个Iterator中。加入数据库现在有key-$num为头的数条数据

```go
iter := db.NewIterator(dbUtil.BytesPrefix([]byte("key-")), nil)
```

遍历读取这些数据

```cpp
for iter.Next() {
    logger.Debug(string(iter.Key()) + string(iter.Value()))
}
```

读取最后一条数据

```rust
if iter.Last() {
    logger.Debug(iter.Key() + iter.Value())
}
```

6）删除某个KEY

```
err = db.Delete([]byte("key-3"), nil)
```



7）批量写

```
batch := new(leveldb.Batch)
batch.Put([]byte("foo"), []byte("value"))
batch.Put([]byte("bar"), []byte("another value"))
batch.Delete([]byte("baz"))
err = db.Write(batch, nil)
```

