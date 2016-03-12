# 跨 Core 字段

输出其它core的字段，包括用其它core字段排序。

## 配置方法

### vootoo-core-XXX.jar
把 [vootoo-core-0.4.0.jar](https://oss.sonatype.org/service/local/repositories/releases/content/org/vootoo/vootoo-core/0.4.0/vootoo-core-0.4.0.jar) 放到 *solr_home*/lib 目录里。

### schema
schema.xml 加入字段及类型的声明，如：

```xml
<dynamicField name="_solr_*"  type="spanCoreField"    indexed="true"  stored="true"/>

<fieldType name="spanCoreField" class="org.vootoo.schema.CrossCoreField" />
```

借 dynamicField 的字段名动态的能力，从字段名中解决出其它 core 的名和字段名。格式定义如下：

```
_solr_OtherCoreName.FieldName
```

* ```_solr_``` 是固定的。
* ```OtherCoreName``` 改成你指定的 core 名。
* ```FieldName``` 是 ```OtherCoreName``` core 里的字段。

### 创建 solr core

创建一个 main_core 和 一个 sub_core。直接从 ```configsets/basic_configs``` copy 出来两份一个作为 main_core，另一个作为 sub_core。

只要在 main_core 加入上面的 schema 配置。

## 使用示例

### 写入数据

main_core

```xml
<add>
<doc>
<field name="id">1</field>
<field name="my_i" >100</field>
</doc>
<doc>
<field name="id">2</field>
<field name="my_i" >50</field>
</doc>
</add>
```

sub_core

```xml
<add>
<doc>
<field name="id">1</field>
<field name="other_i" >20</field>
</doc>
<doc>
<field name="id">2</field>
<field name="other_i" >80</field>
</doc>
</add>
```

### 查询

```
http://localhost:8983/solr/main_core/select?q=*:*&wt=xml&indent=true
&sort=mul(_solr_sub_core.other_i,2) desc
&fl=*,other_i:sum(field(_solr_sub_core.other_i),3)
&fsv=true
```

分别从三个使用上输出：

* 排序 - 函数排序 ```mul(_solr_sub_core.other_i,2) desc```
  * 字段排序 ```field(_solr_sub_core.other_i) desc```
* 输出 - 函数输出 ```fl=*,other_i:sum(field(_solr_sub_core.other_i),3)``` 此时也要用 field 函数取出
* 输出排序值 - 用 ```fsv=true``` 参数。

结果如：

```xml
<response>
<lst name="responseHeader">
<int name="status">0</int>
<int name="QTime">1</int>
<lst name="params">
<str name="fl">*,other_i:sum(field(_solr_sub_core.other_i),3)</str>
<str name="sort">mul(_solr_sub_core.other_i,2) desc</str>
<str name="indent">true</str>
<str name="q">*:*</str>
<str name="wt">xml</str>
<str name="fsv">true</str>
</lst>
</lst>
<result name="response" numFound="2" start="0">
<doc>
<str name="id">2</str>
<int name="my_i">50</int>
<long name="_version_">1528513646883241984</long>
<float name="other_i">83.0</float>
</doc>
<doc>
<str name="id">1</str>
<int name="my_i">100</int>
<long name="_version_">1528513646809841664</long>
<float name="other_i">23.0</float>
</doc>
</result>
<lst name="sort_values">
<arr name="product(cross_core_field(sub_core, other_i),const(2))">
<double>160.0</double>
<double>40.0</double>
</arr>
</lst>
</response>
```
