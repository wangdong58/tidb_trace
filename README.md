# Proposal: TiDB全量SQL采集及性能分析
- Author(s): 王栋 徐云磊 台枫
- Last updated: 2021/01/07

## 概要
采集并记录访问TiDB的SQL语句，以及对应SQL语句的性能指标，用来更细粒度的分析语句级的性能问题。

## 背景
对于记录到慢查询的语句，我们可以根据对应的性能指标来分析语句问题。
如果一个应用下的语句执行时间都没有达到慢查询的阀值（默认300 毫秒），想优化这个应用的执行时间，就需要所有执行语句的性能汇总数据来分析。

## 方案及原理：
- 采集程序数据源基于information_schema.cluster_statements_summary表，在TiDB Server节点上间隔10ms获取数据，在内存中比较前后两次间隔获取的查询结果集的差异值记录下来。对于执行时间小于间隔时间的SQL语句，我们可以近似的获取到这条SQL语句执行对应的性能指标。
- 采集程序输出文件到tmpfs，基于内存的文件系统中。
- 导入程序，通过访问tmpfs中的trace临时文件，拼接成批量的SQL，直接insert到clickhouse数据库中。
- 需要修改源码扩展一些功能：
1、QUERY_SAMPLE_TEXT：现在记录的是原 SQL 语句，一个SQL语句带参数的样本，可以考虑修改为上次执行的SQL语句明细。
2、现在表中没有记录客户端IP，可以考虑增加客户端IP。
3、将LAST_SEEN字段的时间精度提高到毫秒级等。
