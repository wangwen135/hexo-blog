---
title: ES-使用Lucene-修复分片问题
date: 2020-11-10 23:41
tags: 
  - Elasticsearch
categories:
  - [Elasticsearch]
---


Elasticsearch分片包含一个Lucene索引，因此我们可以使用Lucene的CheckIndex工具，该工具使我们能够扫描并修复有问题的片段，而且通常只会造成最小的数据丢失。

Lucene CheckIndex工具包含在默认的Elasticsearch发行版中，不需要额外的下载。

```
# change this to reflect your shard path, the format is
# {path.data}/{cluster_name}/nodes/{node_id}/indices/{index_name}/{shard_id}/index/

$ export SHARD_PATH=data/elasticsearch/nodes/0/indices/foo/0/index/
$ java -cp lib/elasticsearch-*.jar:lib/*:lib/sigar/* -ea:org.apache.lucene... org.apache.lucene.index.CheckIndex $SHARD_PATH
```

如果CheckIndex检测到问题并且其修复建议看起来很明智，则可以通过添加-fix命令行参数来告诉CheckIndex应用修补程序。

实际是用 **-exorcise** 命令

------
```
 java -cp lib/elasticsearch-*.jar:lib/*:lib/sigar/* -ea:org.apache.lucene... org.apache.lucene.index.CheckIndex

ERROR: index path not specified
Usage: java org.apache.lucene.index.CheckIndex pathToIndex [-exorcise] [-crossCheckTermVectors] [-segment X] [-segment Y] [-dir-impl X]

  -exorcise: actually write a new segments_N file, removing any problematic segments
  -fast: just verify file checksums, omitting logical integrity checks
  -crossCheckTermVectors: verifies that term vectors match postings; THIS IS VERY SLOW!
  -codec X: when exorcising, codec to write the new segments_N file with
  -verbose: print additional details
  -segment X: only check the specified segments.  This can be specified multiple
              times, to check more than one segment, eg '-segment _2 -segment _a'.
              You can't use this with the -exorcise option
  -dir-impl X: use a specific FSDirectory implementation. If no package is specified the org.apache.lucene.store package will be used.

**WARNING**: -exorcise *LOSES DATA*. This should only be used on an emergency basis as it will cause
documents (perhaps many) to be permanently removed from the index.  Always make
a backup copy of your index before running this!  Do not run this tool on an index
that is actively being written to.  You have been warned!

Run without -exorcise, this tool will open the index, report version information
and report any exceptions it hits and what action it would take if -exorcise were
specified.  With -exorcise, this tool will remove any segments that have issues and
write a new segments_N file.  This means all documents contained in the affected
segments will be removed.

This tool exits with exit code 1 if the index cannot be opened or has any
corruption, else 0.

```

# 修复

当出现分片损坏时，应该是分片中的某些 segment 出问题了。

解决办法是删掉这些 segment 用新的代替，但是这些segment中保存的数据会丢失。

处理步骤： 

1. 关闭ES

2. 执行
```
$ export SHARD_PATH=data/elasticsearch/nodes/0/indices/foo/0/index/
$ java -cp lib/elasticsearch-*.jar:lib/*:lib/sigar/* -ea:org.apache.lucene... org.apache.lucene.index.CheckIndex $SHARD_PATH
```
会提示有几个坏的segment，会丢失多少数据

3. 使用 **-exorcise**命令进行数据恢复

```
$ java -cp lib/elasticsearch-*.jar:lib/*:lib/sigar/* -ea:org.apache.lucene... org.apache.lucene.index.CheckIndex $SHARD_PATH -exorcise
```

4. 完了启动es即可，只是会丢失上面所说的X条数据
