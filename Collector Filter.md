# Collector Filter

Collector 过滤：支持从 lucene 的 collector 过虑结果。

solr 中使用：
solrconfig.xml 中加入

```xml
<queryParser name="cf" class="org.vootoo.search.CollectorFilterQParserPlugin"/>
```

如 schema 中有  my_int 的 int 字段。
solr 查询语法：

* in - 像 sql 的 in 操作.
  * fq={!cf name=in}my_int:(2, 3)
  * fq={!cf name=in not=true}my_int:(0,4)
  * fq={!cf name=in}sub(my_int,1):(2, 3)
  * fq={!cf name=in}add(my_int, 1):4
* range - '(' 是开区间; '[' 是闭区间 
  * fq={!cf name=range}my_int:(3 TO 4]
  * fq={!cf name=range not=true}my_int:[2 TO 3]
  * fq={!cf name=range}add(my_int,1):(3 TO 4]
* bit - 支持 bit 位过虑, 含有一个任一bit：```indexValue & queryValue != 0```
  * fq={!cf name=bit}bit_field:(0b01100)
  * fq={!cf name=bit}bit_field:(0xa)
  * fq={!cf name=bit}bit_field:(3)
* cbit - 索引里必需全部同时包含查询的bit：```indexValue & queryValue == queryValue```
  * fq={!cf name=cbit}bit_field:(0b01100)

0.2.0 加入 bit/cbit， 输入参数支持2、10、16进制。

