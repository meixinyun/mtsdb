# Influxdb 概览

Influxdb 作为时间序列数据库，当前业界的领军作品，在数据模型，存储读写优化，以及索引上都做出了
卓越贡献。深入的学习和了解它，能快速提升我们对时间序列数据库的认知。

# Influxdb 的历史

### 1.0 之前

### 1.0 阶段 

#### 增加对高基时间序列的限制，v1.1.0 [2016-11-14].

max-values-per-tag : 选项加入，超过100000的高基序列将会被丢弃，partial write error 会返回给调用者。这个限制可以用来阻止某个Measurement的基数过高的tag values 序列写入。

#### 50%以上的多核系统的写入性能提升，v1.2.0 [2017-01-24]

#### 增加时间序列索引v1.3.0 [2017-06-21]

这是个大版本更新，首次引入了 TSI， TSM也是这次引入。[解决的主要问题是高基序列问题。](https://www.influxdata.com/blog/path-1-billion-time-series-influxdb-high-cardinality-indexing-ready-testing/) 已有序列数据的加载启动时间缩小到可以忽略不计。


增加了 持续查询统计功能，即预计算能力，（类似Prometheus的Recording Rule）
功能，

### 1.5.0 [2018-03-06]

提升了 高基序列内存索引启动性能
TSI增加了 支持流失和复制数据分片功能。
 增加了对 Prometheus 格式的监控 支持/metrics

### v1.7.8 [2019-08-20] 
不支持非UTF-8的字符编码格式

### v1.8.0 [2020-4-13]
Flux v0.65 可以上生产环境了


### 2.0 阶段

###  v2.0.0-alpha.1 [2019-01-23]



# Influxdb 设计理念及权衡


## 重复数据多次写

优势：简化的冲突解决提高了写入性能

劣势：无法存储重复数据，在极少数情况下可能会覆盖数据。

## 删除很少发生，若真的需要删除，总是针对大范围的旧数据删除

优势：限制对删除的访问可以提高查询和写入性能

劣势：删除功能受到很大限制

## 假设数据更新几乎不会发生

优势：限制更新访问提高读写性能

劣势：更新功能受到很大限制

## 绝大部分的数据写入是针对时间戳最近的热点数据，并且是按照时间升序加入

优势：按照时间升序写入数据显著提高性能

劣势：随机时间时间点的写入，或不按照时间升序写入性能很差

## 扩展性至关重要，数据库必须能处理大量的读写操作。

优势：数据库能处理海量的读写

劣势：Influxdb 团队被迫作出权衡来提高性能



## 能读写大量数据比拥有强一致的数据试图更重要

优势：海量的读写可以通过多客户端和高负载完成

劣势：**如果数据库的负载非常高，可能不会返回最新的数据点**


# 存储引擎设计亮点

* TSM 针对时序数据在内存以及文件格式上做了针对性的优化，优雅地实现了时序数据的高效率写入以及高压缩率存储，同时文件级别的B+树索引可以有效提高时序数据根据SeriesKey查询时间序列的性能

* 实现了内存以及文件级别的倒排索引，有效实现了根据给定维度fieldKey查询对应SeriesKey的功能，这样再根据SeriesKey、fieldKey和时间间隔就可以在文件中查找到对应的时序数据集合






