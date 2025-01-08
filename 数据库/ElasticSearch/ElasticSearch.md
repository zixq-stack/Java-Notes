# 基础理论和DSL语法

## 准备工作

1、下载ElasticSearch的window版

linux版后续说明

自行百度 [Elasticsearch](https://www.elastic.co/downloads/past-releases#elasticsearch) 进行下载，我的版本是：7.8.0



2、下载postman：自行百度进行下载



### ElasticSearch中的目录解读

> 进到bin目录下，点击 elasticsearch.bat 文件即可启动 ES 服务

| 目录    | 含义           |
| ------- | -------------- |
| bin     | 可执行脚本目录 |
| config  | 配置目录       |
| jdk     | 内置 JDK 目录  |
| lib     | 类库           |
| logs    | 日志目录       |
| modules | 模块目录       |
| plugins | 插件目录       |



### ELK技术是什么意思？

![image-20250101103432196](https://img2023.cnblogs.com/blog/2421736/202501/2421736-20250101103431245-1437515488.png)





### 注意事项

> 保证自己的JDK是1.8或以上，最低要求1.8





### ES非关系型和关系型数据库对应关系

> **注意**：ES 7.x之后，type已经被淘汰了，其他的没变
>
> 只要玩ES，那么这个图就要牢牢地记在自己脑海里，后续的名词解释不再过多说明，就是操作这幅图中的东西

![img](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230621222902701-1239094255.png) 



## 基础理论

### 正向索引和倒排索引

![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230627225152906-1397781389.png)

elasticsearch中使用的就是倒排索引



倒排索引中又有3个小东西：

1. **词条**：**是指索引中的最小存储或查询单元**。这个其实很好理解，白话文来讲就是：字或者词组，英文就是一个单词，中文就是字或词组嘛，比如：你要查询的内容中具备含义的某一个字或词组，这就是词条呗，如：我是中国人，就可以分为：我、是、中国人、中国、国人这样的几个词条。但是数据千千万万，一般的数据结构能够存的下吗？不可能的，所以这里做了文章，采用的是B+树和hash存储(如：hashmap)
2. **词典**：就是词条的集合嘛。**字或者词组组成的内容呗**
3. **倒排表**：**就是指 关键字 / 关键词 在索引中的位置。** 有点类似于数组，你查询数组中某个元素的位置，但是区别很大啊，我只是为了好理解，所以才这么举例子的



### type 类型

> 这玩意儿就相当于关系型数据库中的表，注意啊：关系型中表是在数据库下，那么ES中也相应的 类型是在索引之下建立的

表是个什么玩意呢？行和列嘛，这行和列有多少？N多行和N多列嘛，所以：ES中的类型也一样，可以定义N种类型。
同时：每张表要存储的数据都不一样吧，所以表是用来干嘛的？分类 / 分区嘛，所以ES中的类型的作用也来了：就是为了分类嘛。
另外：关系型中可以定义N张表，那么在ES中，也可以定义N种类型

**因此：ES中的类型类似于关系型中的表，作用：为了分类 / 分区，同时：可以定义N种类型，但是：类型必须是在索引之下建立的（ 是索引的逻辑体现嘛 ）**

**但是：不同版本的ES，类型也发生了变化，上面的解读不是全通用的**

![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230627225152905-1909735633.png)



### field 字段

> 这也就类似于关系型中的列。 对文档数据根据不同属性（列字段）进行的分类标识



字段常见的简单类型：

> **注意**：id的类型在ES中id是字符串，这点需要注意

1. 字符串：text（可分词的文本）、keyword（精确值，例如：品牌、国家、ip地址）。text和keyword的区别如下；

- text类型支持全文检索和完全查询，即：我搜索时只用字符串中的一个字符照样得到结果。**原理：text使用了分词，就是把字符串拆分为单个字符了**
- keyword类型支持完全查询，即：精确查询，**前提：index不是false**。**原理：keyword不支持分词，所以：查询时必须是完全查询（ 所有字符匹配上才可以 ）**

2. 数值：byte、short、integer、long、float、double

3. 布尔：boolean

4. 日期：date

5. 对象：object

6. 地图类型：geo_point 和 geo_shape

- geo_point：有纬度(latitude) 和经度(longitude)确定的一个点，如：“32.54325453, 120.453254”
- geo_shape：有多个geo_point组成的复杂集合图形，如一条直线 “LINESTRING (-77.03653 38.897676, -77.009051 38.889939)”

7. 自动补全类型：completion



> **注意：**没有数组类型，但是可以实现出数组，因为每种类型可以有“多个值”，即可实现出类似于数组类型，例如下面的格式：

```json
{
    "age": 21,	// Integer类型
    "weight": 52.1,		// float类型
    "isMarried": false,		// boolean类型
    "info": "这就是一个屌丝女",		// 字符串类型 可能为test，也可能为keyword 需要看mapping定义时对文档的约束时什么
    "email": "zixq8@slafjkl.com",	// 字符串类型 可能为test，也可能为keyword 需要看mapping定义时对文档的约束时什么
    "score": [99.1, 99.5, 98.9],	// 类似数组	就是利用了一个类型可以有多个值
    "name": {		// object对象类型
        "firstName": "紫",
        "lastName": "邪情"
    }
}
```

谈到数组就得提到一个**扁平化处理 nested**，直接看官网 [Nested datatype](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/nested.html)



> **还有一个字段的拷贝 copy_to：** 可以使用copy_to属性将当前字段拷贝到指定字段
>
> **使用场景：** 多个字段放在一起搜索的时候
>
> **注意：** 定义的要拷贝的那个字段在ES中看不到，但是确实是存在的，就像个虚拟的一样

```json
// 定义了一个字段
"all": {
    "type": "text",
    "analyzer": "ik_max_word"
}


"name": {
    "type": "text",
    "analyzer": "ik_max_word",
    "copy_to": "all"		// 将当前字段 name 拷贝到 all字段中去
}
```



### document 文档

> 这玩意儿类似于关系型中的行。 一个文档是一个可被索引的基础信息单元，也就是一条数据

即：用来搜索的数据，其中的每一条数据就是一个文档。例如一个网页、一个商品信息



**新增文档：**

```json
// 这是kibana中进行的操作，要是使用如postman风格的东西发请求，则在 /索引库名/_doc/文档id 前加上es主机地址即可
POST /索引库名/_doc/文档id		// 指定了文档id，若不指定则es自动创建
{
    "字段1": "值1",
    "字段2": "值2",
    "字段3": {
        "子属性1": "值3",
        "子属性2": "值4"
    },
    // ...
}
```

**查看指定文档id的文档：**

```json
GET /{索引库名称}/_doc/{id}
```

**删除指定文档id的文档：**

```json
DELETE /{索引库名}/_doc/id值
```

**修改文档：**有两种方式

- **全量修改**：直接覆盖原来的文档。其本质是：
  - 根据指定的id删除文档
  - 新增一个相同id的文档
  - **注意**：如果根据id删除时，id不存在，第二步的新增也会执行，也就从修改变成了新增操作了

```json
// 语法格式
PUT /{索引库名}/_doc/文档id
{
    "字段1": "值1",
    "字段2": "值2",
    // ... 略
}
```

- **增量/局部修改**：是只修改指定id匹配的文档中的部分字段

```json
// 语法格式
POST /{索引库名}/_update/文档id
{
    "doc": {
         "字段名": "新的值",
    }
}
```



### mapping 映射

> 指的就是：结构信息 / 限制条件

还是对照关系型来看，在关系型中表有哪些字段、该字段是否为null、默认值是什么........诸如此的限制条件，所以**ES中的映射就是：数据的使用规则设置**



mapping是对索引库中文档的约束，常见的mapping属性包括：

- index：是否创建索引，默认为true
- analyzer：使用哪种分词器
- properties：该字段的子字段

更多类型去官网查看：https://www.elastic.co/guide/en/elasticsearch/reference/8.8/mapping-params.html



创建索引库，最关键的是mapping映射，而mapping映射要考虑的信息包括：

- 字段名
- 字段数据类型
- 是否参与搜索
- 是否需要分词
- 如果分词，分词器是什么？

其中：

- 字段名、字段数据类型，可以参考数据表结构的名称和类型
- 是否参与搜索要分析业务来判断，例如图片地址，就无需参与搜索
- 是否分词呢要看内容，内容如果是一个整体就无需分词，反之则要分词
- 分词器，我们可以统一使用ik_max_word



```json
{
  "mappings": {
    "properties": {		// 子字段
      "字段名1":{		// 定义字段名
        "type": "text",		// 该字段的类型
        "analyzer": "ik_smart"		// 该字段采用的分词器类型 这是ik分词器中的，一种为ik_smart 一种为ik_max_word，具体看一开始给的系列知识链接
      },
      "字段名2":{
        "type": "keyword",
        "index": "false"		// 该字段是否可以被索引，默认值为trus，即：不想被搜索的字段就可以显示声明为false
      },
      "字段名3":{
        "properties": {
          "子字段": {
            "type": "keyword"
          }
        }
      },
      // ...略
    }
  }
}
```

**创建索引库的同时，创建数据结构约束：**

```json
// 格式
PUT /索引库名称				// 创建索引库
{						// 同时创建数据结构约束信息
  "mappings": {
    "properties": {
      "字段名":{
        "type": "text",
        "analyzer": "ik_smart"
      },
      "字段名2":{
        "type": "keyword",
        "index": "false"
      },
      "字段名3":{
        "properties": {
          "子字段": {
            "type": "keyword"
          }
        }
      },
      // ...略
    }
  }
}



// 示例
PUT /user
{
  "mappings": {
    "properties": {
      "info":{
        "type": "text",
        "analyzer": "ik_smart"
      },
      "email":{
        "type": "keyword",
        "index": "falsae"
      },
      "name":{
        "properties": {
          "firstName": {
            "type": "keyword"
          },
		 "lastName": {
			"type": "keyword"
          }
        }
      },
      // ... 略
    }
  }
}
```





### index 索引库

> 所谓索引：类似于关系型数据库中的数据库

但是索引这个东西在ES中又有点东西，它的作用和关系型数据库中的索引是一样的，相当于门牌号，一个标识，旨在：提高查询效率，当然，不是说只针对查询，CRUD都可以弄索引，所以这么一说ES中的索引和关系型数据库中的索引是一样的，就不太类似于关系型中的数据库了，此言差矣！在关系型中有了数据库，才有表结构（ 行、列、类型...... ）

而在ES中就是有了索引，才有doc、field.....，因此：这就类似于关系型中的数据库，只是作用和关系型中的索引一样罢了

**因此：ES中索引类似于关系型中的数据库，作用：类似于关系型中的索引，旨在：提高查询效率，当然：在一个集群中可以定义N多个索引，同时：索引名字必须采用全小写字母**

当然：也别忘了有一个倒排索引

- 关系型数据库通过增加一个**B+树索引**到指定的列上，以便提升数据检索速度。而ElasticSearch 使用了一个叫做 `倒排索引` 的结构来达到相同的目的



**创建索引：** 相当于在创建数据库

```json
# 在kibana中进行的操作
PUT /索引库名称

# 在postman之类的地方创建
http://ip:port/indexName     如：http://127.0.0.1:9200/createIndex    	请求方式：put
```

**注：put请求具有幂等性**，幂等性指的是： 不管进行多少次重复操作，都是实现相同的结果。可以采用把上面的请求多执行几次，然后：观察返回的结果

**具有幂等性的有：put、delete、get**



**查看索引库：**

```json
# 查看指定的索引库
GET /索引库名

# 查看所有的索引库
GET /_cat/indices?v 
```



**修改索引库：**

- 倒排索引结构虽然不复杂，但是一旦数据结构改变（比如改变了分词器），就需要重新创建倒排索引，这简直是灾难。因此索引库**一旦创建，无法修改mapping**。



虽然无法修改mapping中已有的字段，但是却允许添加新的字段到mapping中，因为不会对倒排索引产生影响。

**语法说明**：

```json
PUT /索引库名/_mapping
{
  "properties": {
    "新字段名":{
      "type": "integer"
        // ............
    }
  }
}
```



**删除索引库：**

```json
DELETE /索引库名
```











## 文档_doc

### 使用post创建doc

> 这种方式：是采用ES随机生成id时使用的请求方式
>
> 注：需要先创建索引，因为：这就类似于关系型数据库中在数据库的表中 创建数据

**语法：**

 ```txt
 http://ip:port/indexName/_doc     如： http://ip:9200/createIndex/_doc    请求方式：post
 ```

![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230621222903828-1974717579.png)




### 使用put创建doc-转幂等性-自定义id

在路径后面加一个要创建的id值即可

![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230621222907182-1936688817.png)




### 查询文档_doc - 重点

#### id查询单条_doc

**语法：**

 ```txt
 http://ip:port/indexName/_doc/id      如： http://ip:9200/createIndex/_doc/100001     请求方式：get
 ```

![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230621222907206-597552959.png)




#### 查询ES中索引下的全部_doc

**语法：**

```txt
http://ip:port/indexName/_search    如： http://ip:9200/createIndex/_search     请求方式：get
```



**注意：别再body中携带数据了，不然就会报：**

Unknown key for a VALUE_STRING in [title]

![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230621222908057-722361093.png)




返回的结果：

```json
{
    "took": 69,   // 查询花费的时间  毫秒值
    "timed_out": false,     // 是否超时
    "_shards": {    // 分片  还没学，先不看
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 3,
            "relation": "eq"
        },
        "max_score": 1.0,
        "hits": [   // 查询出来的 当前索引下的所有_doc文档
            // .............................
        ]
    }
}
```






### 文档_doc的修改

#### 全量修改

> 原理：利用内容覆盖，重新发一份文档罢了



**语法：**

```txt
http://ip:port/indexName/_doc/id      如： http://ip:9200/createIndex/_doc/100001     请求方式：post
```

![](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230621222904962-525012672.png)





#### 局部修改

**语法：**

```txt
http://ip:port/indexName/_update/id   如： http://ip:9200/createIndex/_update/100001    请求方式：post
```

![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230621222906923-142580147.png)





### 文档_doc的删除

使用delete请求即可





### 文档DSL查询

elasticsearch的查询依然是基于JSON风格的DSL来实现的



#### DSL查询分类

ElasticSearch提供了基于JSON的DSL（[Domain Specific Language](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)）来定义查询。常见的查询类型包括：

1. **查询所有**：查询出所有数据，一般测试用。例如：match_all

2. **全文检索(full text)查询**：利用分词器对用户输入内容分词，然后去倒排索引库中匹配。例如：

- match_query
- multi_match_query

3. **精确查询**：根据精确词条值查找数据，一般是查找keyword、数值、日期、boolean等类型字段，所以**不会**对搜索条件分词。例如：

- ids
- range
- term

4. **地理（geo）查询**：根据经纬度查询。例如：

- geo_distance
- geo_bounding_box

5. **复合（compound）查询**：复合查询可以将上述各种查询条件组合起来，合并查询条件。例如：

- bool
- function_score

6. **聚合(aggregations)查询：** 可以让我们极其方便的实现对数据的统计、分析、运算，例如：

- **桶（Bucket）**聚合：用来对文档做分组
- **度量（Metric）**聚合：用以计算一些值，比如：最大值、最小值、平均值等
- **管道（pipeline）**聚合：其它聚合的结果为基础做聚合




查询的语法基本一致：除了聚合查询

```json
GET /indexName/_search
{
  "query": {
    "查询类型": {
      "查询条件": "条件值"
    }
  }
}



// 例如：查询所有
GET /indexName/_search
{
  "query": {
    "match_all": {		// 查询类型为match_all
    }				  // 没有查询条件
  }
}
```

其它查询无非就是**查询类型**、**查询条件**的变化







#### 全文检索查询

> **定义：** 利用分词器对用户输入内容分词，然后去倒排索引库中匹配



全文检索查询的基本流程如下：

1. 对用户搜索的内容做分词，得到词条
2. 根据词条去倒排索引库中匹配，得到文档id
3. 根据文档id找到文档，返回给用户



**使用场景：** 搜索框搜索内容，如百度输入框搜索、google搜索框搜索……….



**注意：** 因为是拿着词条去匹配，因此**参与搜索的字段必须是可分词的text类型的字段**



常见的全文检索查询包括：

- match查询：单字段查询
- multi_match查询：多字段查询，任意一个字段符合条件就算符合查询条件

match查询语法如下：

```json
GET /indexName/_search
{
  "query": {
    "match": {
      "field": "搜索的文本内容text"
    }
  }
}


// 例如：
GET /indexName/_search
{
  "query": {
    "match": {
      "name": "紫邪情"
    }
  }
}
```

mulit_match语法如下：

```json
GET /indexName/_search
{
  "query": {
    "multi_match": {
      "query": "搜索的文本内容text",
      "fields": ["field1", "field2"]
    }
  }
}


// 例如：
GET /indexName/_search
{
  "query": {
    "multi_match": {
      "query": "Java",
      "fields": ["username","title", "context"]
    }
  }
}
```

**注意：** 搜索字段越多，对查询性能影响越大，因此建议采用copy_to，然后使用单字段查询的方式(即：match查询)







#### 精准查询

> **定义：** 根据精确词条值查找数据，一般是查找keyword、数值、日期、boolean等类型字段，所以**不会**对搜索条件分词



常见的精准查询有：

- term：根据词条精确值查询
- range：根据值的范围查询





##### term查询/精确查询

因为精确查询的字段搜是不分词的字段，因此查询的条件也必须是**不分词**的词条。查询时，用户输入的内容跟自动值完全匹配时才认为符合条件。如果用户输入的内容过多，反而搜索不到数据



语法说明：

```json
// term查询
GET /indexName/_search
{
  "query": {
    "term": {
      "field": {
        "value": "要精确查询的内容"
      }
    }
  }
}


// 例如：
GET /indexName/_search
{
  "query": {
    "term": {
      "field": {
        "value": "遥远的救世主"
      }
    }
  }
}
```





##### range查询/范围查询

> 范围查询，一般应用在对数值类型做范围过滤的时候。比如做价格范围过滤





基本语法：

```json
// range查询
GET /indexName/_search
{
  "query": {
    "range": {
      "FIELD": {
        "gte": 10, // gte代表大于等于，gt则代表大于
        "lte": 20 // lte代表小于等于，lt则代表小于
      }
    }
  }
}

// 例如：
GET /indexName/_search
{
  "query": {
    "range": {
      "price": {
        "gte": 10000,
        "lte": 20000
      }
    }
  }
}
```







#### 地理坐标查询

> 所谓的地理坐标查询，其实就是根据经纬度查询，官方文档：https://www.elastic.co/guide/en/elasticsearch/reference/current/geo-queries.html



常见的使用场景包括：

- 携程：搜索我附近的酒店
- 滴滴：搜索我附近的出租车
- 微信：搜索我附近的人





##### 矩形范围查询

矩形范围查询，也就是geo_bounding_box查询，查询坐标落在某个矩形范围的所有文档

查询时，需要指定矩形的**左上**、**右下**两个点的坐标，然后画出一个矩形(就是对两个点画“十”字，中间交汇的部分就是要的矩形)，落在该矩形内的都是符合条件的点，比如下图

![DKV9HZbVS6](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230622193521922-1031103897.gif)





语法如下：

```json
// geo_bounding_box查询
GET /indexName/_search
{
  "query": {
    "geo_bounding_box": {
      "FIELD": {
        "top_left": { // 左上点
          "lat": 31.1,	// 这个点的经度
          "lon": 121.5	// 这个点的纬度
        },
        "bottom_right": { // 右下点
          "lat": 30.9,
          "lon": 121.7
        }
      }
    }
  }
}
```







##### 附近查询/距离查询

附近查询，也叫做距离查询（geo_distance）：查询到指定中心点小于某个距离值的所有文档

换句话来说，在地图上找一个点作为圆心，以指定距离为半径，画一个圆，落在圆内的坐标都算符合条件，如下

![vZrdKAh19C](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230622194102711-1944482080.gif)





语法说明：

```json
// geo_distance 查询
GET /indexName/_search
{
  "query": {
    "geo_distance": {
      "distance": "距离", // 半径
      "field": "经度,纬度" // 圆心
    }
  }
}



// 例如：在经纬度为 31.21,121.5 的方圆15km的附近
GET /indexName/_search
{
  "query": {
    "geo_distance": {
      "distance": "15km", // 半径
      "location": "31.21,121.5" // 圆心
    }
  }
}
```







#### 复合查询

> 复合查询可以将其它简单查询组合起来，实现更复杂的搜索逻辑



常见的复合查询有两种：

- fuction score：算分函数查询，可以控制文档相关性算分，控制文档排名
- bool query：布尔查询，利用逻辑关系组合多个其它的查询，实现复杂搜索





##### 相关性算分算法

当我们利用match查询时，文档结果会根据与搜索词条的关联度打分（_score），返回结果时按照分值降序排列

在elasticsearch中，早期使用的打分算法是TF-IDF算法，公式如下：

![image-20210721190152134](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230622194749139-1479435428.png)



在后来的5.1版本升级中，elasticsearch将算法改进为BM25算法，公式如下：

![image-20210721190416214](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230622194749113-1396433065.png)





TF-IDF算法有一各缺陷，就是词条频率越高，文档得分也会越高，单个词条对文档影响较大。而BM25则会让单个词条的算分有一个上限，曲线更加平滑：

![image-20210721190907320](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230622194749495-591958408.png)







##### function_score 算分函数查询

> 算分函数查询可以控制文档相关性算分，控制文档排名



以百度为例，你搜索的结果中，并不是相关度越高排名越靠前，而是谁掏的钱多排名就越靠前

要想人为控制相关性算分，就需要利用elasticsearch中的function score 查询了





**语法格式说明：**

![image-20210721191544750](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230622195143155-1162661874.png)



function score 查询中包含四部分内容：

- **原始查询**条件：query部分，基于这个条件搜索文档，并且基于BM25算法给文档打分，**原始算分**（query score)
- **过滤条件**：filter部分，符合该条件的文档才会重新算分
- **算分函数**：符合filter条件的文档要根据这个函数做运算，得到的**函数算分**（function score），有四种函数
  1. weight：函数结果是常量
  2. field_value_factor：以文档中的某个字段值作为函数结果
  3. random_score：以随机数作为函数结果
  4. script_score：自定义算分函数算法
- **运算模式**：算分函数的结果、原始查询的相关性算分，两者之间的运算方式，包括：
  1. multiply：相乘
  2. replace：用function score替换query score
  3. 其它，例如：sum、avg、max、min



function score的运行流程如下：

1. 根据**原始条件**查询搜索文档，并且计算相关性算分，称为**原始算分**（query score）
2. 根据**过滤条件**，过滤文档
3. 符合**过滤条件**的文档，基于**算分函数**运算，得到**函数算分**（function score）
4. 将**原始算分**（query score）和**函数算分**（function score）基于**运算模式**做运算，得到最终结果，作为相关性算分。



因此，其中的关键点是：

- 过滤条件：决定哪些文档的算分被修改
- 算分函数：决定函数算分的算法
- 运算模式：决定最终算分结果







##### bool 布尔查询

布尔查询是一个或多个查询子句的组合，每一个子句就是一个**子查询**。子查询的组合方式有：

- must：必须匹配每个子查询，类似“与”
- should：选择性匹配子查询，类似“或”
- must_not：必须不匹配，**不参与算分**，类似“非”
- filter：必须匹配，**不参与算分**



**注意：** 搜索时，参与**打分的字段越多，查询的性能也越差**。因此这种多条件查询时，建议这样做：

- 搜索框的关键字搜索，是全文检索查询，使用must查询，参与算分
- 其它过滤条件，采用filter查询。不参与算分



示例：

```json
GET /indexName/_search
{
  "query": {
    "bool": {
      "must": [
        {"term": {"city": "上海" }}
      ],
      "should": [
        {"term": {"brand": "皇冠假日" }},
        {"term": {"brand": "华美达" }}
      ],
      "must_not": [
        { "range": { "price": { "lte": 500 } }}
      ],
      "filter": [
        { "range": {"score": { "gte": 45 } }}
      ]
    }
  }
}
```







#### 排序查询

elasticsearch默认是根据相关度算分（_score）来排序，但是也支持自定义方式对搜索[结果排序](https://www.elastic.co/guide/en/elasticsearch/reference/current/sort-search-results.html)。可以排序字段类型有：keyword类型、数值类型、地理坐标类型、日期类型等



keyword、数值、日期类型排序的语法基本一致

**语法**：

```json
GET /indexName/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "FIELD": "desc"  // 排序字段、排序方式ASC、DESC
    }
    // 多个字段排序就继续写
  ]
}
```

排序条件是一个数组，也就是可以写多个排序条件。按照声明的顺序，当第一个条件相等时，再按照第二个条件排序，以此类推



地理坐标排序略有不同

提示：获取你的位置的经纬度的方式：https://lbs.amap.com/demo/jsapi-v2/example/map/click-to-get-lnglat/

**语法说明**：

```json
GET /indexName/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "_geo_distance" : {
          "FIELD" : "纬度，经度", // 文档中geo_point类型的字段名、目标坐标点
          "order" : "asc", // 排序方式
          "unit" : "km" // 排序的距离单位
      }
    }
  ]
}
```

这个查询的含义是：

- 指定一个坐标，作为目标点
- 计算每一个文档中，指定字段（必须是geo_point类型）的坐标 到目标点的距离是多少
- 根据距离排序





#### 分页查询

elasticsearch 默认情况下只返回top10的数据。而如果要查询更多数据就需要修改分页参数了。elasticsearch中通过修改from、size参数来控制要返回的分页结果：

- from：从第几个文档开始
- size：总共查询几个文档

类似于mysql中的`limit ?, ?`



##### 基本分页

分页的基本语法如下：

```json
GET /indexName/_search
{
  "query": {
    "match_all": {}
  },
  "from": 0, // 分页开始的位置，默认为0
  "size": 10, // 期望获取的文档总数
  "sort": [
    {"price": "asc"}
  ]
}
```

- 优点：支持随机翻页
- 缺点：深度分页问题，默认查询上限（from + size）是10000
- 场景：百度、京东、谷歌、淘宝这样的随机翻页搜索





##### 深度分页问题

现在，我要查询990~1000的数据，查询逻辑要这么写：

```json
GET /indexName/_search
{
  "query": {
    "match_all": {}
  },
  "from": 990, // 分页开始的位置，默认为0
  "size": 10, // 期望获取的文档总数
  "sort": [
    {"price": "asc"}
  ]
}
```

这里是查询990开始的数据，也就是 第990~第1000条 数据

不过，elasticsearch内部分页时，必须先查询 0~1000条，然后截取其中的990 ~ 1000的这10条：

![image-20210721200643029](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230623115445291-121516878.png)



查询TOP1000，如果es是单点模式，这并无太大影响

但是elasticsearch将来一定是集群，例如我集群有5个节点，我要查询TOP1000的数据，并不是每个节点查询200条就可以了

因为节点A的TOP200，在另一个节点可能排到10000名以外了

因此要想获取整个集群的TOP1000，必须先查询出每个节点的TOP1000，汇总结果后，重新排名，重新截取TOP1000

![image-20210721201003229](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230623115445422-276452929.png)



那如果我要查询9900~10000的数据呢？是不是要先查询TOP10000呢？那每个节点都要查询10000条？汇总到内存中？



当查询分页深度较大时，汇总数据过多，对内存和CPU会产生非常大的压力，因此elasticsearch会禁止from+ size 超过10000的请求



针对深度分页，ES提供了两种解决方案，[官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/paginate-search-results.html)：

- search after：分页时需要排序，原理是从上一次的排序值开始，查询下一页数据。官方推荐使用的方式
  - 优点：没有查询上限（单次查询的size不超过10000）
  - 缺点：只能向后逐页查询，不支持随机翻页
  - 场景：没有随机翻页需求的搜索，例如手机向下滚动翻页
- scroll：原理将排序后的文档id形成快照，保存在内存。官方已经不推荐使用
  - 优点：没有查询上限（单次查询的size不超过10000）
  - 缺点：会有额外内存消耗，并且搜索结果是非实时的
  - 场景：海量数据的获取和迁移。从ES7.1开始不推荐，建议用 search after方案





#### 高亮查询

高亮显示的实现分为两步：

1. 给文档中的所有关键字都添加一个标签，例如`<em>`标签
2. 页面给`<em>`标签编写CSS样式





**高亮的语法**：

```json
GET /indexName/_search
{
  "query": {
    "match": {
      "field": "TEXT" // 查询条件，高亮一定要使用全文检索查询
    }
  },
  "highlight": {
    "fields": {
      "FIELD": { // 指定要高亮的字段
        "pre_tags": "<em>",  // 用来标记高亮字段的前置标签，es默认添加的标签就是em
        "post_tags": "</em>" // 用来标记高亮字段的后置标签
      }
    }
  }
}
```

**注意：**

- 高亮是对关键字高亮，因此**搜索条件必须带有关键字**，而不能是范围这样的查询。
- 默认情况下，**高亮的字段，必须与搜索指定的字段一致**，否则无法高亮
- 如果要对非搜索字段高亮，则需要添加一个属性：required_field_match=false，可以解决的场景：要高亮的字段和搜索指定字段不一致。如：



```json
GET /indexName/_search
{
  "query": {
    "match": {
      "name": "紫邪情" // 查询条件，高亮一定要使用全文检索查询
    }
  },
  "highlight": {
    "fields": {
      "all": { // 假如这里的all字段是利用copy_to将其他很多字段copy进来的，就造成上面搜索字段name与这里要高亮得到字段不一致
        "pre_tags": "<em>",
        "post_tags": "</em>",
        "require_field_match": "false"		// 是否要求字段匹配，即：要高亮字段和搜索字段是否匹配，默认是true
      }
    }
  }
}
```





#### 聚合查询/数据聚合

**[聚合（](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html)[aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html)[）](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html)**可以让我们极其方便的实现对数据的统计、分析、运算。例如：

- 什么品牌的手机最受欢迎？
- 这些手机的平均价格、最高价格、最低价格？
- 这些手机每月的销售情况如何？

实现这些统计功能的比数据库的sql要方便的多，而且查询速度非常快，可以实现近实时搜索效果





##### 聚合的分类

聚合常见的有三类：

- **桶（Bucket）**聚合：用来对文档做分组
  - TermAggregation：按照文档字段值分组，例如按照品牌值分组、按照国家分组
  - Date Histogram：按照日期阶梯分组，例如一周为一组，或者一月为一组

- **度量（Metric）**聚合：用以计算一些值，比如：最大值、最小值、平均值等
  - Avg：求平均值
  - Max：求最大值
  - Min：求最小值
  - Stats：同时求max、min、avg、sum等
- **管道（pipeline）**聚合：其它聚合的结果为基础做聚合



> **注意：**参加聚合的字段必须是keyword、日期、数值、布尔类型，即：只要不是 text 类型即可，因为text类型会进行分词，而聚合不能进行分词





##### Bucket 桶聚合

**桶（Bucket）**聚合：用来对文档做分组

- TermAggregation：按照文档字段值分组，例如按照品牌值分组、按照国家分组
- Date Histogram：按照日期阶梯分组，例如一周为一组，或者一月为一组



语法如下：

```json
GET hhtp://ip:port/indexName/_search
{
  "query": {	// 加入基础查询，从而限定聚合范围，不然默认是将es中的文档全部查出来再聚合
    "查询类型": {
      "查询条件": "条件值"
    }
  },
  "size": 0,  // 设置size为0，结果中不包含文档，只包含聚合结果	即：去掉结果hits中的hits数组的数据
  "aggs": { // 定义聚合
    "AggName": { //给聚合起个名字
      "aggType": { // 聚合的类型，跟多类型去官网
        "field": "value", // 参与聚合的字段
        "size": 20, // 希望获取的聚合结果数量	默认是10
		"order": {	// 改变聚合的排序规则，默认是 desc 降序
			"_key": "asc" // 按照什么关键字以什么类型排列
        }
      }
    }
  }
}
```

例如：

```json
// 数据聚合
GET /indexName/_search
{
  "query": {
    "range": {
      "price": {
        "lte": 200
      }
    }
  }, 
  "size": 0, 
  "aggs": {
    "brandAgg": {
      "terms": {
        "field": "brand",
        "size": 15,
        "order": {
          "_count": "asc"
        }
      }
    }
  }
}
```

![image-20230627113244929](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230627113248432-979326214.png)



##### Metric 度量聚合

**度量（Metric）**聚合：用以计算一些值，比如：最大值、最小值、平均值等

- Avg：求平均值
- Max：求最大值
- Min：求最小值
- Stats：同时求max、min、avg、sum等



语法如下：

```json
GET /indexName/_search
{
  "size": 0, 
  "aggs": {
    "aggName": { 
      "aggType": { 
        "field": "value", 
        "size": 20,
        "order": {
            "_key": "orderType"
        }
      },
      "aggs": { // brands聚合的子聚合，也就是分组后对每组分别计算
        "aggName": { // 聚合名称
          "aggType": { // 聚合类型，这里stats可以计算min、max、avg等
            "field": "value" // 聚合字段
          }
        }
      }
    }
  }
}


// 例如：
GET /indexName/_search
{
  "size": 0, 
  "aggs": {
    "brandAgg": { 
      "terms": { 
        "field": "brand", 
        "size": 20,
        "order": {
            "scoreAgg.avg": "asc"	// 注意：若是要使用子聚合来对每个桶进行排序，则这里的写法有点区别
        }
      },
      "aggs": {
        "scoreAgg": {
          "stats": {
            "field": "score"
          }
        }
      }
    }
  }
}
```



![image-20230627114819466](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230627114820906-657603913.png)







#### 自动补全查询 completion

elasticsearch提供了[Completion Suggester](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/search-suggesters.html)查询来实现自动补全功能。**这个查询会匹配以用户输入内容开头的词条并返回**。为了提高补全查询的效率，对于文档中字段的类型有一些约束：

- 参与补全查询的字段**必须**是completion类型

- 字段的内容一般是用来补全的多个词条形成的数组



**场景：** 搜索框输入关键字，搜索框下面就会弹出很多相应的内容出来

![image-20230627230906831](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230627230908175-277805668.png)



比如，一个这样的索引库：

```json
// 创建索引库
PUT test
{
  "mappings": {
    "properties": {
      "title":{
        "type": "completion"	// 指定字段类型为 completion
      }
    }
  }
}
```

然后插入下面的数据：

```json
// 示例数据
POST test/_doc
{
  "title": ["Sony", "WH-1000XM3"]	// 字段内容为多个词条组成的数组
}
POST test/_doc
{
  "title": ["SK-II", "PITERA"]
}
POST test/_doc
{
  "title": ["Nintendo", "switch"]
}
```

查询的DSL语句如下：

```json
// 自动补全查询
GET /test/_search
{
  "suggest": {
    "title_suggest": {	// 起个名字
      "text": "s", // 关键字
      "completion": {
        "field": "title", // 补全查询的字段
        "skip_duplicates": true, // 跳过重复的
        "size": 10 // 获取前10条结果
      }
    }
  }
}
```







# Java操作ES篇 - 重点

## 摸索Java链接ES的流程

自行创建一个maven项目



### 父项目依赖管理

```xml
<properties>
    <ES-version>7.8.0</ES-version>
    <log4j-version>1.2.17</log4j-version>
    <junit-version>4.13.2</junit-version>
    <jackson-version>2.13.0</jackson-version>
    <fastjson.version>1.2.83</fastjson.version>
</properties>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.elasticsearch</groupId>
            <artifactId>elasticsearch</artifactId>
            <!-- 注意：这里的版本问题，要和下载的window的ES版本一致，甚至后续用linux搭建也是一样的
                          到时用linux时，ES、kibana的版本都有这样的限定
                -->
            <version>${ES-version}</version>
        </dependency>

        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <!-- 注意：这里别搞成了elasticsearch-client
                    这个东西在7.x已经不推荐使用了，而到了8.0之后，这个elasticsearch-client已经完全被废弃了
                 -->
            <artifactId>elasticsearch-rest-high-level-client</artifactId>
            <!-- 同样的，注意版本问题 -->
            <version>${ES-version}</version>
        </dependency>

        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>${log4j-version}</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit-version}</version>
        </dependency>

        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>${jackson-version}</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>${fastjson.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```





### 摸索链接流程

获取父项目中的依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.elasticsearch</groupId>
        <artifactId>elasticsearch</artifactId>
    </dependency>

    <dependency>
        <groupId>org.elasticsearch.client</groupId>
        <artifactId>elasticsearch-rest-high-level-client</artifactId>
    </dependency>

    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
    </dependency>
</dependencies>
```



代码编写：

```java
import org.apache.http.HttpHost;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestClientBuilder;
import org.elasticsearch.client.RestHighLevelClient;
import org.junit.Test;

import Java.io.IOException;


public class ConnectionTest {
    /**
     * 倒着看逻辑即可
     */
    @Test
    public void test() throws IOException {

        // 3、创建HttpHost
        HttpHost host = new HttpHost("127.0.0.1", 9200);	// 需要：String hostname, int port
      // 当然：这个方法重载中有一个参数scheme  这个是：访问方式 根据需求用http / https都可以  这里想传的话用：http就可以了

        // 2、创建RestClientBuilder
        RestClientBuilder clientBuilder = RestClient.builder(host);
        // 发现1、有重载；2、重载之中有几个参数，而HttpHost... hosts 这个参数貌似贴近我们想要的东西了，所以建一个HttpHost


        // 1、要链接client，那肯定需要一个client咯，正好：导入得有high-level-client
        RestHighLevelClient esClient = new RestHighLevelClient(clientBuilder);
        // 发现需要RestClientBuilder restClientBuilder，那就建

        // 4、释放资源
        esClient.close();
    }
}


```







## Java中操作ES索引

向父项目获取自己要的依赖

```xml
<dependency>
    <groupId>org.elasticsearch</groupId>
    <artifactId>elasticsearch</artifactId>
</dependency>

<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
</dependency>

<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <scope>test</scope>
</dependency>
```





### 封装链接对象

```java
import org.apache.http.HttpHost;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;

/**
 * @ClassName ESClientUtil
 * @Author ZiXieQing
 * @Date 2021/12/14
 * Version 1.0
 **/
public class ESClientUtil {

    private static final String HOST = "127.0.0.1";
    private static final Integer PORT = 9200;

    public static RestHighLevelClient getESClient() {
        return new RestHighLevelClient(RestClient.builder(new HttpHost(HOST, PORT)));
        // 还有一种方式
        // return new RestHighLevelClient(RestClient.builder(HttpHost.create("http://ip:9200")));
    }
}
```





### 操作索引

```java
import org.apache.http.HttpHost;
import org.elasticsearch.action.admin.indices.delete.DeleteIndexRequest;
import org.elasticsearch.action.admin.indices.flush.FlushRequest;
import org.elasticsearch.action.admin.indices.flush.FlushResponse;
import org.elasticsearch.action.support.master.AcknowledgedResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.client.indices.CreateIndexRequest;
import org.elasticsearch.client.indices.CreateIndexResponse;
import org.elasticsearch.client.indices.GetIndexRequest;
import org.elasticsearch.client.indices.GetIndexResponse;
import org.elasticsearch.common.xcontent.XContentType;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;

import static com.zixieqing.hotel.constant.MappingConstant.mappingContext;

/**
 * elasticsearch的索引库测试
 * 规律：esClient.indices().xxx(xxxIndexRequest(IndexName), RequestOptions.DEFAULT)
 *      其中 xxx 表示要对索引进行得的操作，如：create、delete、get、flush、exists.............
 *
 * <p>@author       : ZiXieqing</p>
 */

@SpringBootTest(classes = HotelApp.class)
public class o1IndexTest {
    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        this.client = new RestHighLevelClient(RestClient.builder(HttpHost.create("http://ip:9200")));
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }

    /**
     * 创建索引 并 创建字段的mapping映射关系
     */
    @Test
    void createIndexAndMapping() throws IOException {
        // 1、创建索引
        CreateIndexRequest request = new CreateIndexRequest("person");
        // 2、创建字段的mapping映射关系   参数1：编写的mapping json字符串  参数2：采用的文本类型
        request.source(mappingContext, XContentType.JSON);
        // 3、发送请求 正式创建索引库与mapping映射关系
        CreateIndexResponse response = client.indices().create(request, RequestOptions.DEFAULT);
        // 查看是否创建成功
        System.out.println("response.isAcknowledged() = " + response.isAcknowledged());
        // 判断指定索引库是否存在
        boolean result = client.indices().exists(new GetIndexRequest("person"), RequestOptions.DEFAULT);
        System.out.println(result ? "hotel索引库存在" : "hotel索引库不存在");
    }

    /**
     * 删除指定索引库
     */
    @Test
    void deleteIndexTest() throws IOException {
        // 删除指定的索引库
        AcknowledgedResponse response = client.indices()
                .delete(new DeleteIndexRequest("person"), RequestOptions.DEFAULT);
        // 查看是否成功
        System.out.println("response.isAcknowledged() = " + response.isAcknowledged());
    }

    // 索引库一旦创建，则不可修改，但可以添加mapping映射

    /**
     * 获取指定索引库
     */
    @Test
    void getIndexTest() throws IOException {
        // 获取指定索引
        GetIndexResponse response = client.indices()
                .get(new GetIndexRequest("person"), RequestOptions.DEFAULT);
    }

    /**
     * 刷新索引库
     */
    @Test
    void flushIndexTest() throws IOException {
        // 刷新索引库
        FlushResponse response = client.indices().flush(new FlushRequest("person"), RequestOptions.DEFAULT);
        // 检查是否成功
        System.out.println("response.getStatus() = " + response.getStatus());
    }
}
```



















## Java操作ES中的文档_doc - 重点

这里还需要json依赖，使用jackson或fastjson均可

同时：为了偷懒，所以把lombok也一起导入了



### 基本的文档CRUD

```java
import com.alibaba.fastjson.JSON;
import com.zixieqing.hotel.pojo.Hotel;
import com.zixieqing.hotel.pojo.HotelDoc;
import com.zixieqing.hotel.service.IHotelService;
import org.apache.http.HttpHost;
import org.elasticsearch.action.delete.DeleteRequest;
import org.elasticsearch.action.delete.DeleteResponse;
import org.elasticsearch.action.get.GetRequest;
import org.elasticsearch.action.get.GetResponse;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.action.index.IndexResponse;
import org.elasticsearch.action.update.UpdateRequest;
import org.elasticsearch.action.update.UpdateResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.common.xcontent.XContentType;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;

/**
 * elasticsearch的文档测试
 * 规律：esClient.xxx(xxxRequest(IndexName, docId), RequestOptions.DEFAULT)
 *      其中 xxx 表示要进行的文档操作，如：
 *          index   新增文档
 *          delete  删除指定id文档
 *          get     获取指定id文档
 *          update  修改指定id文档的局部数据
 *
 * <p>@author       : ZiXieqing</p>
 */

@SpringBootTest(classes = HotelApp.class)
public class o2DocumentTest {
    @Autowired
    private IHotelService service;

    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        this.client = new RestHighLevelClient(
                RestClient.builder(HttpHost.create("http://ip:9200"))
        );
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }

    /**
     * 添加文档
     */
    @Test
    void addDocumentTest() throws IOException {

        // 1、准备要添加的文档json数据
        // 通过id去数据库获取数据
        Hotel hotel = service.getById(36934L);
        // 当数据库中定义的表结构和es中定义的字段mapping映射不一致时：将从数据库中获取的数据转成 es 中定义的mapping映射关系对象
        HotelDoc hotelDoc = new HotelDoc(hotel);

        // 2、准备request对象    指定 indexName+文档id
        IndexRequest request = new IndexRequest("hotel").id(hotel.getId().toString());

        // 3、把数据转成json
        request.source(JSON.toJSONString(hotelDoc), XContentType.JSON);

        // 4、发起请求，正式在ES中添加文档    就是根据数据建立倒排索引，所以这里调研了index()
        IndexResponse response = client.index(request, RequestOptions.DEFAULT);

        // 5、检查是否成功     使用下列任何一个API均可   若成功二者返回的结果均是 CREATED
        System.out.println("response.getResult() = " + response.getResult());
        System.out.println("response.status() = " + response.status());
    }

    /**
     * 根据id删除指定文档
     */
    @Test
    void deleteDocumentTest() throws IOException {
        // 1、准备request对象
        DeleteRequest request = new DeleteRequest("indexName", "docId");

        // 2、发起请求
        DeleteResponse response = client.delete(request, RequestOptions.DEFAULT);
        // 查看是否成功   成功则返回 OK
        System.out.println("response.status() = " + response.status());
    }

    /**
     * 获取指定id的文档
     */
    @Test
    void getDocumentTest() throws IOException {
        // 1、获取request
        GetRequest request = new GetRequest"indexName", "docId");

        // 2、发起请求，获取响应对象
        GetResponse response = client.get(request, RequestOptions.DEFAULT);

        // 3、解析结果
        HotelDoc hotelDoc = JSON.parseObject(response.getSourceAsString(), HotelDoc.class);
        System.out.println("hotelDoc = " + hotelDoc);
    }

    /**
     * 修改指定索引库 和 文档id的局部字段数据
     * 全量修改是直接删除指定索引库下的指定id文档，然后重新添加相同文档id的文档即可
     */
    @Test
    void updateDocumentTest() throws IOException {
        // 1、准备request对象
        UpdateRequest request = new UpdateRequest("indexName", "docId");

        // 2、要修改那个字段和值      注：参数是 key, value 形式 中间是 逗号
        request.doc(
                "price",500
        );

        // 3、发起请求
        UpdateResponse response = client.update(request, RequestOptions.DEFAULT);
        // 查看结果 成功则返回 OK
        System.out.println("response.status() = " + response.status());
    }
}
```





### 批量操作文档

> 本质：把请求封装了而已，从而让这个请求可以传递各种类型参数，如：删除的、修改的、新增的，这样就可以搭配for循环



```java
package com.zixieqing.hotel;

import com.alibaba.fastjson.JSON;
import com.zixieqing.hotel.pojo.Hotel;
import com.zixieqing.hotel.pojo.HotelDoc;
import com.zixieqing.hotel.service.IHotelService;
import org.apache.http.HttpHost;
import org.elasticsearch.action.bulk.BulkRequest;
import org.elasticsearch.action.bulk.BulkResponse;
import org.elasticsearch.action.delete.DeleteRequest;
import org.elasticsearch.action.get.GetResponse;
import org.elasticsearch.action.get.MultiGetItemResponse;
import org.elasticsearch.action.get.MultiGetRequest;
import org.elasticsearch.action.get.MultiGetResponse;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.common.xcontent.XContentType;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;
import java.util.List;

/**
 * elasticsearch 批量操作文档测试
 * 规律：EsClient.bulk(new BulkRequest()
 *                    .add(xxxRequest("indexName").id().source())
 *                    , RequestOptions.DEFAULT)
 * 其中：xxx 表示要进行的操作，如
 *      index   添加
 *      delete  删除
 *      get     查询
 *      update  修改
 *
 * <p>@author       : ZiXieqing</p>
 */

@SpringBootTest(classes = HotelApp.class)
public class o3BulkDocumentTest {
    @Autowired
    private IHotelService service;

    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        this.client = new RestHighLevelClient(
                RestClient.builder(HttpHost.create("http://ip:9200"))
        );
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }

    /**
     * 批量添加文档数据到es中
     */
    @Test
    void bulkAddDocumentTest() throws IOException {
        // 1、去数据库批量查询数据
        List<Hotel> hotels = service.list();

        // 2、将数据库中查询的数据转成 es 的mapping需要的对象
        BulkRequest request = new BulkRequest();
        for (Hotel hotel : hotels) {
            HotelDoc hotelDoc = new HotelDoc(hotel);
            // 批量添加文档数据到es中
            request.add(new IndexRequest("hotel")
                    .id(hotelDoc.getId().toString())
                    .source(JSON.toJSONString(hotelDoc), XContentType.JSON));
        }

        // 3、发起请求
        BulkResponse response = client.bulk(request, RequestOptions.DEFAULT);
        // 检查是否成功   成功则返回OK
        System.out.println("response.status() = " + response.status());
    }

    /**
     * 批量删除es中的文档数据
     */
    @Test
    void bulkDeleteDocumentTest() throws IOException {
        // 1、准备要删除数据的id
        List<Hotel> hotels = service.list();

        // 2、准备request对象
        BulkRequest request = new BulkRequest();
        for (Hotel hotel : hotels) {
            // 根据批量数据id 批量删除es中的文档
            request.add(new DeleteRequest("hotel").id(hotel.getId().toString()));
        }

        // 3、发起请求
        BulkResponse response = client.bulk(request, RequestOptions.DEFAULT);
        // 检查是否成功       成功则返回 OK
        System.out.println("response.status() = " + response.status());
    }

    
    // 批量获取和批量修改是同样的套路  批量获取还可以使用 mget 这个API


    /**
     * mget批量获取
     */
    @Test
    void mgetTest() throws IOException {
        List<Hotel> hotels = service.list();

        // 1、准备request对象
        MultiGetRequest request = new MultiGetRequest();
        for (Hotel hotel : hotels) {
            // 添加get数据    必须指定index 和 文档id，可以根据不同index查询
            request.add("hotel", hotel.getId().toString());
        }

        // 2、发起请求，获取响应
        MultiGetResponse responses = client.mget(request, RequestOptions.DEFAULT);
        for (MultiGetItemResponse response : responses) {
            GetResponse resp = response.getResponse();
            // 如果存在则打印响应信息
            if (resp.isExists()) {
                System.out.println("获取到的数据= " +resp.getSourceAsString());
            }
        }
    }
}
```







### Java进行DSL文档查询

其实这种查询都是套路而已，一看前面玩的DSL查询的json形式是怎么写的，二看你要做的是什么查询，然后就是用 queryBuilds  将对应的查询构建出来，其他都是相同套路了



#### 查询所有 match all

> match all：查询出所有数据



```java
package com.zixieqing.hotel.dsl_query_document;

import com.zixieqing.hotel.HotelApp;
import org.apache.http.HttpHost;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.SearchHits;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;

/**
 * es的dsl文档查询之match all查询所有，也可以称之为 全量查询
 *
 * <p>@author       : ZiXieqing</p>
 */

@SpringBootTest
public class o1MatchAll {
    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        this.client = new RestHighLevelClient(
                RestClient.builder(HttpHost.create("http://ip:9200"))
        );
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }


    /**
     * 全量查询：查询所有数据
     */
    @Test
    void matchAllTest() throws IOException {
        // 1、准备request
        SearchRequest request = new SearchRequest("indexName");
        // 2、指定哪种查询/构建DSL语句
        request.source().query(QueryBuilders.matchAllQuery());
        // 3、发起请求 获取响应对象
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);
        // 4、处理响应结果
        // 4.1、获取结果中的Hits
        SearchHits searchHits = response.getHits();
        // 4.2、获取Hits中的total
        long total = searchHits.getTotalHits().value;
        System.out.println("总共获取了 " + total + " 条数据");
        // 4.3、获取Hits中的hits
        SearchHit[] hits = searchHits.getHits();
        for (SearchHit hit : hits) {
            // 4.3.1、获取hits中的source 也就是真正的数据，获取到之后就可以用来处理自己要的逻辑了
            String source = hit.getSourceAsString();
            System.out.println("source = " + source);
        }
    }
}
```

Java代码和前面玩的DSL语法的对应情况：

![image-20230623213506444](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230623213508446-24732815.png)





#### 全文检索查询

##### match 单字段查询 与 multi match多字段查询

下面的代码根据情境需要，可以自行将响应结果处理进行抽取

```java
package com.zixieqing.hotel.dsl_query_document;

import com.zixieqing.hotel.HotelApp;
import org.apache.http.HttpHost;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.SearchHits;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;

/**
 * DLS之全文检索查询：利用分词器对用户输入内容分词，然后去倒排索引库中匹配
 * match_query 单字段查询 和 multi_match_query 多字段查询
 *
 * <p>@author       : ZiXieqing</p>
 */


@SpringBootTest
public class o2FullTextTest {
    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        this.client = new RestHighLevelClient(
                RestClient.builder(HttpHost.create("http://ip:9200"))
        );
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }

    /**
     * match_query  单字段查询
     */
    @Test
    void matchQueryTest() throws IOException {
        // 1、准备request
        SearchRequest request = new SearchRequest("indexName");
        // 2、准备DSL
        request.source().query(QueryBuilders.matchQuery("city", "上海"));
        // 3、发送请求，获取响应对象
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);
        // 处理响应结果，后面都是一样的流程 都是解析json结果而已
        SearchHits searchHits = response.getHits();
        long total = searchHits.getTotalHits().value;
        System.out.println("获取了 " + total + " 条数据");
        for (SearchHit hit : searchHits.getHits()) {
            String dataJson = hit.getSourceAsString();
            System.out.println("dataJson = " + dataJson);
        }
    }

    /**
     * multi match 多字段查询 任意一个字段符合条件就算符合查询条件
     */
    @Test
    void multiMatchTest() throws IOException {
        SearchRequest request = new SearchRequest("indexName");
        request.source().query(QueryBuilders.multiMatchQuery("成人用品", "name", "business"));
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);

        // 处理响应结果，后面都是一样的流程 都是解析json结果而已
        SearchHits searchHits = response.getHits();
        long total = searchHits.getTotalHits().value;
        System.out.println("获取了 " + total + " 条数据");
        for (SearchHit hit : searchHits.getHits()) {
            String dataJson = hit.getSourceAsString();
            System.out.println("dataJson = " + dataJson);
        }
    }
}
```





#### 精确查询

> **精确查询**：根据精确词条值查找数据，一般是查找keyword、数值、日期、boolean等类型字段，所以**不会**对搜索条件分词



##### range 范围查询 和 term精准查询

> term：根据词条精确值查询
>
> range：根据值的范围查询



```java
package com.zixieqing.hotel.dsl_query_document;

import com.zixieqing.hotel.HotelApp;
import org.apache.http.HttpHost;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.SearchHits;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;

/**
 * DSL之精确查询：根据精确词条值查找数据，一般是查找keyword、数值、日期、boolean等类型字段，所以 不会 对搜索条件分词
 * range 范围查询 和 term 精准查询
 *
 * <p>@author       : ZiXieqing</p>
 */

@SpringBootTest
public class o3ExactTest {
    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        this.client = new RestHighLevelClient(
                RestClient.builder(HttpHost.create("http://ip:9200"))
        );
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }

    /**
     * term 精准查询 根据词条精确值查询
     * 和 match 单字段查询有区别，term要求内容完全匹配
     */
    @Test
    void termTest() throws IOException {
        SearchRequest request = new SearchRequest("indexName");
        request.source().query(QueryBuilders.termQuery("city", "深圳"));
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);

        // 处理响应结果，后面都是一样的流程 都是解析json结果而已
        SearchHits searchHits = response.getHits();
        long total = searchHits.getTotalHits().value;
        System.out.println("获取了 " + total + " 条数据");
        for (SearchHit hit : searchHits.getHits()) {
            String dataJson = hit.getSourceAsString();
            System.out.println("dataJson = " + dataJson);
        }
    }

    /**
     * range 范围查询
     */
    @Test
    void rangeTest() throws IOException {
        SearchRequest request = new SearchRequest("indexName");
        request.source().query(QueryBuilders.rangeQuery("price").lte(250));
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);

        // 处理响应结果，后面都是一样的流程 都是解析json结果而已
        SearchHits searchHits = response.getHits();
        long total = searchHits.getTotalHits().value;
        System.out.println("获取了 " + total + " 条数据");
        for (SearchHit hit : searchHits.getHits()) {
            String dataJson = hit.getSourceAsString();
            System.out.println("dataJson = " + dataJson);
        }
    }
}
```





#### 地理坐标查询

##### geo_distance 附近查询

```java
package com.zixieqing.hotel.dsl_query_document;

import com.zixieqing.hotel.HotelApp;
import org.apache.http.HttpHost;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.SearchHits;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;

/**
 * DSL之地理位置查询
 * geo_bounding_box 矩形范围查询 和 geo_distance 附近查询
 *
 * <p>@author       : ZiXieqing</p>
 */

@SpringBootTest
public class o4GeoTest {
    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        this.client = new RestHighLevelClient(
                RestClient.builder(HttpHost.create("http://ip:9200"))
        );
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }

    /**
     * geo_distance 附近查询
     */
    @Test
    void geoDistanceTest() throws IOException {
        SearchRequest request = new SearchRequest("indexName");
        request.source()
                .query(QueryBuilders.geoDistanceQuery("location")
                        .distance("15km").point(31.21,121.5));
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);

        // 处理响应结果，后面都是一样的流程 都是解析json结果而已
        SearchHits searchHits = response.getHits();
        long total = searchHits.getTotalHits().value;
        System.out.println("获取了 " + total + " 条数据");
        for (SearchHit hit : searchHits.getHits()) {
            String dataJson = hit.getSourceAsString();
            System.out.println("dataJson = " + dataJson);
        }
    }
}
```





#### 复合查询

function_score 算分函数查询 是差不多的道理



##### bool 布尔查询之must、should、must not、filter查询

布尔查询是一个或多个查询子句的组合，每一个子句就是一个**子查询**。子查询的组合方式有：

- must：必须匹配每个子查询，类似“与”
- should：选择性匹配子查询，类似“或”
- must_not：必须不匹配，**不参与算分**，类似“非”
- filter：必须匹配，**不参与算分**

**注意：** 搜索时，参与**打分的字段越多，查询的性能也越差**。因此这种多条件查询时，建议这样做：

- 搜索框的关键字搜索，是全文检索查询，使用must查询，参与算分
- 其它过滤条件，采用filter查询。不参与算分



```java
package com.zixieqing.hotel.dsl_query_document;

import com.zixieqing.hotel.HotelApp;
import org.apache.http.HttpHost;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.index.query.BoolQueryBuilder;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.SearchHits;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;

/**
 * DSL之复合查询：基础DSL查询进行组合，从而得到实现更复杂逻辑的复合查询
 * function_score 算分函数查询
 *
 * bool布尔查询
 *  must     必须匹配每个子查询   即：and “与”   参与score算分
 *  should   选择性匹配子查询    即：or “或”    参与score算分
 *  must not 必须不匹配         即：“非"       不参与score算分
 *  filter   必须匹配           即：过滤        不参与score算分
 *
 * <p>@author       : ZiXieqing</p>
 */

@SpringBootTest
public class o5Compound {
    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        this.client = new RestHighLevelClient(
                RestClient.builder(HttpHost.create("http://ip:9200"))
        );
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }


    /**
     * bool布尔查询
     *  must     必须匹配每个子查询   即：and “与”   参与score算分
     *  should   选择性匹配子查询    即：or “或”    参与score算分
     *  must not 必须不匹配         即：“非"       不参与score算分
     *  filter   必须匹配           即：过滤        不参与score算分
     */
    @Test
    void boolTest() throws IOException {
        SearchRequest request = new SearchRequest("indexName");
        BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
        // 构建must   即：and 与
        boolQueryBuilder.must(QueryBuilders.termQuery("city", "北京"));
        // 构建should   即：or 或
        boolQueryBuilder.should(QueryBuilders.multiMatchQuery("速8", "brand", "name"));
        // 构建must not   即：非
        boolQueryBuilder.mustNot(QueryBuilders.rangeQuery("price").gte(250));
        // 构建filter   即：过滤
        boolQueryBuilder.filter(QueryBuilders.termQuery("starName", "二钻"));

        request.source().query(boolQueryBuilder);
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);

        // 处理响应结果，后面都是一样的流程 都是解析json结果而已
        SearchHits searchHits = response.getHits();
        long total = searchHits.getTotalHits().value;
        System.out.println("获取了 " + total + " 条数据");
        for (SearchHit hit : searchHits.getHits()) {
            String dataJson = hit.getSourceAsString();
            System.out.println("dataJson = " + dataJson);
        }
    }
}
```

Java代码和前面玩的DSL语法对应关系：

![image-20230624131548461](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230624131550457-305085644.png)





#### fuzzy 模糊查询

```java
package com.zixieqing.hotel.dsl_query_document;

import com.zixieqing.hotel.HotelApp;
import org.apache.http.HttpHost;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.common.unit.Fuzziness;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.SearchHits;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;

/**
 * DSL之模糊查询
 *
 * <p>@author       : ZiXieqing</p>
 */

@SpringBootTest
public class o6FuzzyTest {
    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        this.client = new RestHighLevelClient(
                RestClient.builder(HttpHost.create("http://ip:9200"))
        );
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }

	/**
     * 模糊查询
     */
    @Test
    void fuzzyTest() throws IOException {
        SearchRequest request = new SearchRequest("indexName");
        // fuzziness(Fuzziness.ONE)     表示的是：字符误差数  取值有：zero、one、two、auto
        // 误差数  指的是：fuzzyQuery("name","深圳")这里面匹配的字符的误差    可以有几个字符不一样，多/少几个字符？
        request.source().query(QueryBuilders.fuzzyQuery("name", "深圳").fuzziness(Fuzziness.ONE));
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);

        // 处理响应结果，后面都是一样的流程 都是解析json结果而已
        SearchHits searchHits = response.getHits();
        long total = searchHits.getTotalHits().value;
        System.out.println("获取了 " + total + " 条数据");
        for (SearchHit hit : searchHits.getHits()) {
            String dataJson = hit.getSourceAsString();
            System.out.println("dataJson = " + dataJson);
        }
    }
}
```







#### 排序和分页查询

```java
package com.zixieqing.hotel.dsl_query_document;

import com.zixieqing.hotel.HotelApp;
import org.apache.http.HttpHost;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.SearchHits;
import org.elasticsearch.search.sort.SortOrder;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;

/**
 * DSL之排序和分页
 *
 * <p>@author       : ZiXieqing</p>
 */


@SpringBootTest
public class o7SortAndPageTest {
    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        this.client = new RestHighLevelClient(
                RestClient.builder(HttpHost.create("http://ip:9200"))
        );
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }

    /**
     * sort 排序查询
     */
    @Test
    void sortTest() throws IOException {
        SearchRequest request = new SearchRequest("indexName");
        request.source()
                .query(QueryBuilders.matchAllQuery())
                .sort("price", SortOrder.ASC);
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);

        // 处理响应结果，后面都是一样的流程 都是解析json结果而已
        SearchHits searchHits = response.getHits();
        long total = searchHits.getTotalHits().value;
        System.out.println("获取了 " + total + " 条数据");
        for (SearchHit hit : searchHits.getHits()) {
            String dataJson = hit.getSourceAsString();
            System.out.println("dataJson = " + dataJson);
        }
    }

    /**
     * page 分页查询
     */
    @Test
    void pageTest() throws IOException {
        int page = 2, size = 20;
        SearchRequest request = new SearchRequest("indexName");
        request.source()
                .query(QueryBuilders.matchAllQuery())
                .from((page - 1) * size).size(size);

        SearchResponse response = client.search(request, RequestOptions.DEFAULT);
        // 处理响应结果，后面都是一样的流程 都是解析json结果而已
        SearchHits searchHits = response.getHits();
        long total = searchHits.getTotalHits().value;
        System.out.println("获取了 " + total + " 条数据");
        for (SearchHit hit : searchHits.getHits()) {
            String dataJson = hit.getSourceAsString();
            System.out.println("dataJson = " + dataJson);
        }
    }
}
```







#### 高亮查询

返回结果处理的逻辑有点区别，但思路都是一样的



```java
package com.zixieqing.hotel.dsl_query_document;

import com.alibaba.fastjson.JSON;
import com.zixieqing.hotel.HotelApp;
import com.zixieqing.hotel.pojo.HotelDoc;
import org.apache.http.HttpHost;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.builder.SearchSourceBuilder;
import org.elasticsearch.search.fetch.subphase.highlight.HighlightField;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.util.CollectionUtils;

import java.io.IOException;
import java.util.Map;

/**
 * DSL之高亮查询
 *
 * <p>@author       : ZiXieqing</p>
 */

@SpringBootTest(classes = HotelApp.class)
public class o8HighLightTest {
    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        this.client = new RestHighLevelClient(
                RestClient.builder(HttpHost.create("http://ip:9200"))
        );
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }

    /**
     * 高亮查询
     * 返回结果处理不太一样
     */
    @Test
    void highLightTest() throws IOException {
        SearchRequest request = new SearchRequest("hotel");
        request.source()
                .query(QueryBuilders.matchQuery("city", "北京"))
                .highlighter(SearchSourceBuilder.highlight()
                        .field("name")  // 要高亮的字段
                        .preTags("<em>")    // 前置HTML标签 默认就是em
                        .postTags("</em>")  // 后置标签
                        .requireFieldMatch(false));     // 是否进行查询字段和高亮字段匹配

        // 发起请求，获取响应对象
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);
        // 处理响应结果
        for (SearchHit hit : response.getHits()) {
            String originalData = hit.getSourceAsString();
            HotelDoc hotelDoc = JSON.parseObject(originalData, HotelDoc.class);
            System.out.println("原始数据为：" + originalData);

            // 获取高亮之后的结果
            // key 为要进行高亮的字段，如上为field("name")   value 为添加了标签之后的高亮内容
            Map<String, HighlightField> highlightFields = hit.getHighlightFields();
            if (!CollectionUtils.isEmpty(highlightFields)) {
                // 根据高亮字段，获取对应的高亮内容
                HighlightField name = highlightFields.get("name");
                if (name != null) {
                    // 获取高亮内容   是一个数组
                    String highLightStr = name.getFragments()[0].string();
                    hotelDoc.setName(highLightStr);
                }
            }

            System.out.println("hotelDoc = " + hotelDoc);
        }
    }
}
```

代码和DSL语法对应关系： request.source()  获取到的就是返回结果的整个json文档

![image-20230624175348848](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230624175351527-407883617.png)





#### 聚合查询

**[聚合（](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html)[aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html)[）](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html)**可以让我们极其方便的实现对数据的统计、分析、运算



聚合常见的有三类：

- **桶（Bucket）**聚合：用来对文档做分组
  - TermAggregation：按照文档字段值分组，例如按照品牌值分组、按照国家分组
  - Date Histogram：按照日期阶梯分组，例如一周为一组，或者一月为一组

- **度量（Metric）**聚合：用以计算一些值，比如：最大值、最小值、平均值等
  - Avg：求平均值
  - Max：求最大值
  - Min：求最小值
  - Stats：同时求max、min、avg、sum等
- **管道（pipeline）**聚合：其它聚合的结果为基础做聚合



> **注意：**参加聚合的字段必须是keyword、日期、数值、布尔类型，即：只要不是 text 类型即可，因为text类型会进行分词，而聚合不能进行分词





```java
package com.zixieqing.hotel.dsl_query_document;

import com.zixieqing.hotel.HotelApp;
import org.apache.http.HttpHost;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.aggregations.AggregationBuilders;
import org.elasticsearch.search.aggregations.Aggregations;
import org.elasticsearch.search.aggregations.BucketOrder;
import org.elasticsearch.search.aggregations.bucket.terms.Terms;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;
import java.util.List;

/**
 * 数据聚合 aggregation 可以让我们极其方便的实现对数据的统计、分析、运算
 * 桶（Bucket）聚合：用来对文档做分组
 *      TermAggregation：按照文档字段值分组，例如按照品牌值分组、按照国家分组
 *      Date Histogram：按照日期阶梯分组，例如一周为一组，或者一月为一组
 *
 *  度量（Metric）聚合：用以计算一些值，比如：最大值、最小值、平均值等
 *      Avg：求平均值
 *      Max：求最大值
 *      Min：求最小值
 *      Stats：同时求max、min、avg、sum等
 *
 *  管道（pipeline）聚合：其它聚合的结果为基础做聚合
 *
 * <p>@author       : ZiXieqing</p>
 */

@SpringBootTest(classes = HotelApp.class)
public class o9AggregationTest {
    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        this.client = new RestHighLevelClient(
                RestClient.builder(HttpHost.create("http://ip:9200"))
        );
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }

    @Test
    void aggregationTest() throws IOException {
        // 获取request
        SearchRequest request = new SearchRequest("indexName");
        // 组装DSL
        request.source()
                .size(0)
                .query(QueryBuilders
                        .rangeQuery("price")
                        .lte(250)
                )
                .aggregation(AggregationBuilders
                        .terms("brandAgg")
                        .field("brand")
                        .order(BucketOrder.aggregation("scoreAgg.avg",true))
                        .subAggregation(AggregationBuilders
                                .stats("scoreAgg")
                                .field("score")
                        )
                );

        // 发送请求，获取响应
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);
        // 处理响应结果
        System.out.println("response = " + response);
        // 获取全部聚合结果对象 getAggregations
        Aggregations aggregations = response.getAggregations();
        // 根据聚合名 获取其聚合对象
        Terms brandAgg = aggregations.get("brandAgg");
        // 根据聚合类型 获取对应聚合对象
        List<? extends Terms.Bucket> buckets = brandAgg.getBuckets();
        for (Terms.Bucket bucket : buckets) {
            // 根据key获取其value
            String value = bucket.getKeyAsString();
            // 将value根据需求做处理
            System.out.println("value = " + value);
        }
    }
}
```

请求组装对应关系：

![image-20230627140843561](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230627140846640-42593182.png)

响应结果对应关系：

![image-20230627141303392](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230627141304881-71420632.png)







#### 自动补全查询

```java
package com.zixieqing.hotel.dsl_query_document;

import com.zixieqing.hotel.HotelApp;
import org.apache.http.HttpHost;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.search.suggest.Suggest;
import org.elasticsearch.search.suggest.SuggestBuilder;
import org.elasticsearch.search.suggest.SuggestBuilders;
import org.elasticsearch.search.suggest.completion.CompletionSuggestion;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;

/**
 * 自动补全 completion类型： 这个查询会匹配以用户输入内容开头的词条并返回
 *  参与补全查询的字段 必须 是completion类型
 *  字段的内容一般是用来补全的多个词条形成的数组
 *
 * <p>@author       : ZiXieqing</p>
 */

@SpringBootTest(classes = HotelApp.class)
public class o10Suggest {
    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        this.client = new RestHighLevelClient(
                RestClient.builder(HttpHost.create("http://ip:9200"))
        );
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }

    @Test
    void completionTest() throws IOException {
        // 准备request
        SearchRequest request = new SearchRequest("hotel");
        // 构建DSL
        request.source()
                .suggest(new SuggestBuilder()
                        .addSuggestion(
                                "title_suggest",
                                SuggestBuilders.completionSuggestion("title")
                                        .prefix("s")
                                        .skipDuplicates(true)
                                        .size(10)
                        ));

        // 发起请求，获取响应对象
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);
        // 解析响应结果
        // 获取整个suggest对象
        Suggest suggest = response.getSuggest();
        // 通过指定的suggest名字，获取其对象
        CompletionSuggestion titleSuggest = suggest.getSuggestion("title_suggest");
        for (CompletionSuggestion.Entry options : titleSuggest) {
            // 获取每一个options中的test内容
            String context = options.getText().string();
            // 按需求对内容进行处理
            System.out.println("context = " + context);
        }
    }
}
```

代码与DSL、响应结果对应关系：

![image-20230627235426570](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230627235428071-2109239701.png)







#### ES与MySQL数据同步

这里的同步指的是：MySQL发生变化，则elasticsearch索引库也需要跟着发生变化



数据同步一般有三种方式：同步调用方式、异步通知方式、监听MySQL的binlog方式





**1、同步调用：**

- 优点：实现简单，粗暴
- 缺点：业务耦合度高

![image-20230628155716064](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230628155716992-1015419003.png)



**2、异步通知：**

- 优点：低耦合，实现难度一般
- 缺点：依赖mq的可靠性



![image-20230628160432048](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230628160433144-390172066.png)



**3、监听MySQL的binlog文件：**

- 优点：完全解除服务间耦合
- 缺点：开启binlog增加数据库负担、实现复杂度高



![image-20230628160321828](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230628160321783-1630441014.png)







# ES高级篇



## Docker带密码部署

1. 拉取镜像

```bash
docker pull docker.elastic.co/elasticsearch/elasticsearch:7.6.2
```

2. 创建elasticsearch.yml文件。注意：创建此文件的路径在第3步中有用

```yaml
cluster.name: "docker-cluster"
network.host: 0.0.0.0
# 这一步是开启x-pack插件	配置密码就需要它
xpack.security.enabled: true
```

3. 创建容器并启动

```bash
docker run -d -it \
--restart=always \
--privileged=true \
--name=es7 \
-p 9200:9200 \
-p 9300:9300 \
-e "discovery.type=single-node" \
# /opt/elasticsearch.yml 就是第2步的路径
-v /opt/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-e ES_JAVA_OPTS="-Xms256m -Xmx256m" 镜像id


# 获取镜像ID方式
docker images
```

4. 进入容器内容

```bash
# 1
docker exec -it es7 /bin/bash

# 2
cd bin
```

5. 手动设置密码

```bash
elasticsearch-setup-passwords interactive
```

![image-20240307204034870](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240307203946751-1508909511.png)



6. 退出容器，重启ES

```bash
# 退出容器
exit

# 重启ES
docker restart es7
```

测试：ip:9200访问，输入密码页面能返回JSON数据即成功。





## 集群部署

> 集群的意思：就是将多个节点归为一体罢了，这个整体就有一个指定的名字了



### window中部署集群 - 了解

把下载好的window版的ES中的data文件夹、logs文件夹下的所有的文件删掉，然后拷贝成三份，对文件重命名

![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000228562-880977819.png)





**1、修改node-1001节点的config/elasticsearch.yml配置文件。**这个配置文件里面有原生的配置信息，感兴趣的可以查看，因为现在要做的配置信息都在原生的配置信息里，只是被注释掉了而已，当然：没兴趣的，直接全选删掉，然后做如下配置

```yml
# ---------------------------------- Cluster -----------------------------------
#
# Use a descriptive name for your cluster:
# 集群名称  注意：是把多个节点归为一个整体，所以这个集群名字就是各节点归为一体之后的名字
# 因此：各个节点中这个集群名字也就要一样了
cluster.name: es-colony
#
# ------------------------------------ Node ------------------------------------
#
# Use a descriptive name for the node:
# 节点名称  在一个集群中，这个名字要全局唯一
node.name: node-1001
# 是否有资格成为主机节点
node.master: true
# 是否是数据节点
node.data: true
#
# ---------------------------------- Network -----------------------------------
#
# Set the bind address to a specific IP (IPv4 or IPv6):
# 当前节点的ip地址
network.host: 127.0.0.1
#
# Set a custom port for HTTP:
# 当前节点的端口号
http.port: 1001
# 当前节点的通讯端口（ 监听端口 ）
transport.tcp.port: 9301
# 跨域配置
http.cors.enabled: true
http.cors.allow-origin: "*"
```



**2、修改node-1002节点的config/elasticsearch.yml配置文件**

```yml
# ---------------------------------- Cluster -----------------------------------
#
# Use a descriptive name for your cluster:
# 集群名称  注意：是把多个节点归为一个整体，所以这个集群名字就是各节点归为一体之后的名字
# 因此：各个节点中这个集群名字也就要一样了
cluster.name: es-colony
#
# ------------------------------------ Node ------------------------------------
#
# Use a descriptive name for the node:
# 节点名称  在一个集群中，这个名字要全局唯一
node.name: node-1002
# 是否是主机节点
node.master: true
# 是否是数据节点
node.data: true
#
# ---------------------------------- Network -----------------------------------
#
# Set the bind address to a specific IP (IPv4 or IPv6):
# 当前节点的ip地址
network.host: 127.0.0.1
#
# Set a custom port for HTTP:
# 当前节点的端口号
http.port: 1002
# 当前节点的通讯端口（ 监听端口 ）
transport.tcp.port: 9302
# 当前节点不知道集群中另外节点是哪些，所以配置，让当前节点能够找到其他节点
discovery.seed_hosts: ["127.0.0.1:9301"]
# ping请求调用超时时间，但同时也是选主节点的delay time 延迟时间
discovery.zen.fd.ping_timeout: 1m
# 重试次数，防止GC[ 垃圾回收 ]节点不响应被剔除
discovery.zen.fd.ping_retries: 5
# 跨域配置
http.cors.enabled: true
http.cors.allow-origin: "*"
```



**3、修改node-1003节点的config/elasticsearch.yml配置文件**

```yml
# ---------------------------------- Cluster -----------------------------------
#
# Use a descriptive name for your cluster:
# 集群名称  注意：是把多个节点归为一个整体，所以这个集群名字就是各节点归为一体之后的名字
# 因此：各个节点中这个集群名字也就要一样了
cluster.name: es-colony
#
# ------------------------------------ Node ------------------------------------
#
# Use a descriptive name for the node:
# 节点名称  在一个集群中，这个名字要全局唯一
node.name: node-1003
# 是否是主机节点
node.master: true
# 是否是数据节点
node.data: true
#
# ---------------------------------- Network -----------------------------------
#
# Set the bind address to a specific IP (IPv4 or IPv6):
# 当前节点的ip地址
network.host: 127.0.0.1
#
# Set a custom port for HTTP:
# 当前节点的端口号
http.port: 1003
# 当前节点的通讯端口（ 监听端口 ）
transport.tcp.port: 9303
# 当前节点不知道集群中另外节点是哪些，所以配置，让当前节点能够找到其他节点
discovery.seed_hosts: ["127.0.0.1:9301","127.0.0.1:9302"]
# ping请求调用超时时间，但同时也是选主节点的delay time
discovery.zen.fd.ping_timeout: 1m
# 重试次数，防止GC[ 垃圾回收 ]节点不响应被剔除
discovery.zen.fd.ping_retries: 5
# 跨域配置
http.cors.enabled: true
http.cors.allow-origin: "*"
```



依次启动1、2、3节点的bin/elasticsearch.bat即可启动集群



用postman测试集群

```json
GET http://localhost:1001/_cluster/health

// 响应内容
{
    "cluster_name": "es-colony",
    "status": "green",  // 重点查看位置 状态颜色
    "timed_out": false,
    "number_of_nodes": 3,	// 重点查看位置	集群中的节点数量
    "number_of_data_nodes": 3,		// 重点查看位置	集群中的数据节点数量
    "active_primary_shards": 0,
    "active_shards": 0,
    "relocating_shards": 0,
    "initializing_shards": 0,
    "unassigned_shards": 0,
    "delayed_unassigned_shards": 0,
    "number_of_pending_tasks": 0,
    "number_of_in_flight_fetch": 0,
    "task_max_waiting_in_queue_millis": 0,
    "active_shards_percent_as_number": 100.0
}
```

**status字段颜色表示：**当前集群在总体上是否工作正常。它的三种颜色含义如下：

1. **green：** 所有的主分片和副本分片都正常运行
2. **yellow：** 所有的主分片都正常运行，但不是所有的副本分片都正常运行
3. **red：** 有主分片没能正常运行



**附加内容：一些配置说明，下面的一些配置目前有些人可能并没有遇到，但是在这里留个印象吧，知道个大概和怎么去找就行了**

官网地址： https://www.elastic.co/guide/en/elasticsearch/reference/current/modules.html



**主节点 [ host区域 ] 和 数据节点 [ stale区域 ]：**

```yaml
cluster.name: elastics   # 定义集群名称所有节点统一配置
node.name: es-0   # 节点名称自定义
node.master: true  # 主节点,数据节点设置为 false
node.data: false   # 数据节点设置为true
path.data: /home/es/data	# 存储目录，可配置多个磁盘
path.logs: /home/es/logs	# 日志文件路径
bootstrap.mlockall: true        # 启动时锁定内存
network.publish_host: es-0	# 绑定网卡
network.bind_host: es-0		# 绑定网卡
http.port: 9200		# http端口
discovery.zen.ping.multicast.enabled: false	# 禁用多播，跨网段不能用多播
discovery.zen.ping_timeout: 120s
discovery.zen.minimum_master_nodes: 2 # 至少要发现集群可做master的节点数，
client.transport.ping_timeout: 60s
discovery.zen.ping.unicast.hosts: ["es-0","es-1", "es-2","es-7","es-8","es-4","es-5","es-6"]	# 集群自动发现

# fd 是 fault detection 
# discovery.zen.ping_timeout 仅在加入或者选举 master 主节点的时候才起作用；
# discovery.zen.fd.ping_timeout 在稳定运行的集群中，master检测所有节点，以及节点检测 master是否畅通时长期有用
discovery.zen.fd.ping_timeout: 120s		 # 超时时间(根据实际情况调整)
discovery.zen.fd.ping_retries: 6		 # 重试次数，防止GC[垃圾回收]节点不响应被剔除
discovery.zen.fd.ping_interval: 30s		 # 运行间隔

# 控制磁盘使用的低水位。默认为85%，意味着如果节点磁盘使用超过85%，则ES不允许在分配新的分片。当配置具体的大小如100MB时，表示如果磁盘空间小于100MB不允许分配分片
cluster.routing.allocation.disk.watermark.low: 100GB	# 磁盘限额

# 控制磁盘使用的高水位。默认为90%，意味着如果磁盘空间使用高于90%时，ES将尝试分配分片到其他节点。上述两个配置可以使用API动态更新，ES每隔30s获取一次磁盘的使用信息，该值可以通过cluster.info.update.interval来设置
cluster.routing.allocation.disk.watermark.high: 50GB	# 磁盘最低限额

node.zone: hot	# 磁盘区域，分为hot和stale，做冷热分离
script.inline: true	# 支持脚本
script.indexed: true 
cluster.routing.allocation.same_shard.host: true	# 一台机器部署多个节点时防止一个分配到一台机器上，宕机导致丢失数据

# 以下6行为设置thread_pool
threadpool.bulk.type: fixed
threadpool.bulk.size: 32 
threadpool.bulk.queue_size: 100
threadpool.search.type: fixed  
threadpool.search.size: 49 
threadpool.search.queue_size: 10000

script.engine.groovy.inline.aggs: on

# 以下为配置慢查询和慢索引的时间
index.search.slowlog.threshold.query.warn: 20s
index.search.slowlog.threshold.query.info: 10s
index.search.slowlog.threshold.query.debug: 4s
index.search.slowlog.threshold.query.trace: 1s
index.search.slowlog.threshold.fetch.warn: 2s
index.search.slowlog.threshold.fetch.info: 1600ms
index.search.slowlog.threshold.fetch.debug: 500ms
index.search.slowlog.threshold.fetch.trace: 200ms
index.indexing.slowlog.threshold.index.warn: 20s
index.indexing.slowlog.threshold.index.info: 10s
index.indexing.slowlog.threshold.index.debug: 4s
index.indexing.slowlog.threshold.index.trace: 1s

# 索引库设置
indices.fielddata.cache.size: 20%	# 索引库缓存时占用大小
indices.fielddata.cache.expire: "48h"	# 索引库缓存的有效期
indices.cache.filter.size: 10%	# 索引库缓存过滤占用大小
index.search.slowlog.level: WARN	# 索引库搜索慢日志级别
```







### Linux中部署ES

#### 部署单机ES

**1、准备工作**

1. 下载linux版的ES，自行百度进行下载，老规矩，我的版本是：7.8.0
2. 将下载好的linux版ES放到自己的服务器中去 



**2、解压文件：**

```shell
# 命令
tar -zxvf elasticsearch-7.8.0-linux-x86_64.tar.gz
```

 

**3、对文件重命名：**

```shell
# 命令
mv elasticsearch-7.8.0 es 
```



**4、创建用户**：因为安全问题， Elasticsearch 不允许 root 用户直接运行，所以要创建新用户，在 root 用户中创建新用户

```shell
useradd es		# 新增 es 用户
passwd es		# 为 es 用户设置密码，输入此命令后，输入自己想设置的ES密码即可
userdel -r es	# 如果错了，可以把用户删除了再重新加
chown -R es:es /opt/install/es		# 文件授权		注意：/opt/install/es 改成自己的ES存放路径即可
```



**5、修改 config/elasticsearch.yml 配置文件**

```yml
# 在elasticsearch.yml文件末尾加入如下配置
cluster.name: elasticsearch
node.name: node-1
network.host: 0.0.0.0
http.port: 9200
cluster.initial_master_nodes: ["node-1"] 
```



**6、修改 /etc/security/limits.conf 文件**

```shell
# 命令
vim /etc/security/limits.conf

# 在文件末尾中增加下面内容 这个配置是：每个进程可以打开的文件数的限制
es soft nofile 65536
es hard nofile 65536
```



**7、修改 /etc/security/limits.d/20-nproc.conf 文件**

```shell
# 命令
vim /etc/security/limits.d/20-nproc.conf


# 在文件末尾中增加下面内容
# 每个进程可以打开的文件数的限制
es soft nofile 65536
es hard nofile 65536
# 操作系统级别对每个用户创建的进程数的限制
* hard nproc 4096
# 注： * 表示 Linux 所有用户名称
```



**8、修改 /etc/sysctl.conf 文件**

```shell
# 在文件中增加下面内容
# 一个进程可以拥有的 VMA(虚拟内存区域)的数量,默认值为 65536
vm.max_map_count=655360
```



**9、重新加载文件**

```shell
# 命令
sysctl -p
```



**10、启动程序 : 准备进入坑中**

```shell
cd /opt/install/es/
# 启动
bin/elasticsearch
# 后台启动
bin/elasticsearch -d 
```

![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000229055-69575572.png) 



这个错误告知：不能用root用户，所以：**切换到刚刚创建的es用户**

```shell
# 命令
su es
```

然后再次启动程序，**进入下一个坑**

 ![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000229218-1478021295.png) 



这个错误是因为：启动程序的时候会动态生成一些文件，这和ES没得关系，所以：需要切回到root用户，然后把文件权限再次刷新一下

```shell
# 切换到root用户
su root

# 切换到root用户之后，执行此命令即可
chown -R es:es /opt/install/es

# 再切换到es用户
su es
```

![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000228601-598751838.png) 



**11、再次启动程序**

 ![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000230019-351307939.png) 

吃鸡，这样linux中单机ES就部署成功了



不过啊，前面这种方式都是low的方式，有更简单的方式，就是使用docker容器来进行配置，简直不要太简单，虽然：使用docker容器来启动程序有弊端，如：MySQL就不建议放在docker容器中，因为：MySQL是不断地进行io操作，放到docker容器中，就会让io操作效率降低，而ES放到docker中也是同样的道理，但是：可以玩，因为：有些公司其实并没有在意docker的弊端，管他三七二十一扔到docker中



**如果想要用docker容器进行ES配置，编写如下的docker-compose.yml文件**

```yml
version: "3.1"
services:
  elasticsearch:
#	注：此网站版本不全，可以直接用官网 elasticsearch:7.8.0
    image: daocloud.io/library/elasticsearch:7.9.0
    restart: always
    container_name: elasticsearch
    ports:
      - 9200:9200
    environment:
      - Java_OPTS=--Xms256m -Xmx1024m
```

然后启动容器即可

**注：使用docker安装需要保证自己的linux中安装了docker和docker-compose，** 没有安装的话，教程链接：[centos7安装docker和docker-compose](https://www.cnblogs.com/xiegongzi/p/15621992.html) 



**注**：有些人可能还会被防火墙整一下，贫道的防火墙是关了的

```shell
# 暂时关闭防火墙
systemctl stop firewalld
# 永久关闭防火墙
systemctl enable firewalld.service 		# 打开防火墙永久性生效，重启后不会复原
systemctl disable firewalld.service 	# 关闭防火墙，永久性生效，重启后不会复原
```



**12、测试是否成功**

```json
// 在浏览器或postman中输入以下指令均可
GET http://ip:9200/
```

 ![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000228550-1072922586.png) 



浏览器访问不了，看看自己服务器开放9200端口没有，别搞这种扯犊子事啊 









#### 部署集群ES

可以选择和windows版的集群搭建一样

- 解压ES、重命名
- 复制几份ES文件夹
- 修改对应配置



更简单的方式

- 解压linux版的ES，重命名
- 分发节点

```shell
# 分发节点的步骤

xsync es-cluster   # es-cluster为解压之后的es名字

# 注：xsync需要单独配置，配置过程如下：

# 1、安装rsync
yum -y install rsync

# 配置hosts 节点服务配置  此配置是在/etc/hosts中进行的
ip name		# 如：192.168.0.100  hadoop


# 2、编写脚本  看自己的/usr/local/bin中是否有xsync文件  都看这里了，那就是第一次弄，肯定没有，所以
# 在/usr/local/bin中新建一个xsync文件  touch xsync
# 编辑内容如下：
#!/bin/sh
# 获取输入参数个数，如果没有参数，直接退出
pcount=$#
if((pcount==0)); then
        echo no args...;
        exit;
fi

# 获取文件名称
p1=$1
fname=`basename $p1`
echo fname=$fname
# 获取上级目录的绝对路径
pdir=`cd -P $(dirname $p1); pwd`
echo pdir=$pdir
# 获取当前用户名称
user=`whoami`
# 循环
for((host=1; host<=2; host++)); do
        echo $pdir/$fname $user@slave$host:$pdir
        echo ==================slave$host==================
        rsync -rvl $pdir/$fname $user@slave$host:$pdir
done
# Note:这里的slave对应自己主机名，需要做相应修改。另外，for循环中的host的边界值由自己的主机编号决定


# 3、给新建的xsync文件授权
chmod a+x xsync

# 这样就配置成功了
```





**1、同样的，root用户不能直接运行，所以创建用户**

```shell
useradd es 		# 新增 es 用户
passwd es 		# 为 es 用户设置密码
userdel -r es 	# 如果错了，可以删除再加
chown -R es:es /opt/module/es 		# 给文件夹授权
```

**2、编辑 ES文件夹的config/elasticsearch.yml文件，实现集群配置**

```yml
# 加入如下配置，这些配置在网上都有，看不懂的网上比我更详细
# 集群名称
cluster.name: cluster-es
# 节点名称， 每个节点的名称不能重复
node.name: node-1
# ip 地址， 每个节点的地址不能重复
network.host: zixieqing-linux1		# 这个主机名字是在前面使用xsync异步分发时在hosts中配置的名字
# 是不是有资格主节点
node.master: true
node.data: true
http.port: 9200
# head 插件需要这打开这两个配置
http.cors.allow-origin: "*"
http.cors.enabled: true
http.max_content_length: 200mb
# es7.x 之后新增的配置，初始化一个新的集群时需要此配置来选举 master
cluster.initial_master_nodes: ["node-1"]
# es7.x 之后新增的配置，节点发现
discovery.seed_hosts: ["zixieqing-linux1:9300","zixieqing-linux2:9300","zixieqing-linux3:9300"]
gateway.recover_after_nodes: 2
network.tcp.keep_alive: true
network.tcp.no_delay: true
transport.tcp.compress: true
# 集群内同时启动的数据任务个数，默认是 2 个
cluster.routing.allocation.cluster_concurrent_rebalance: 16
# 添加或删除节点及负载均衡时并发恢复的线程个数，默认 4 个
cluster.routing.allocation.node_concurrent_recoveries: 16
# 初始化数据恢复时，并发恢复线程的个数，默认 4 个
cluster.routing.allocation.node_initial_primaries_recoveries: 16
```

**3、修改 etc/security/limits.conf**

```shell
# 在文件末尾中增加下面内容
es soft nofile 65536
es hard nofile 65536
```

**4、修改 /etc/security/limits.d/20-nproc.conf**

```shell
# 在文件末尾中增加下面内容
es soft nofile 65536
es hard nofile 65536
* hard nproc 4096
# 注： * 表示 Linux 所有用户名称
```

**5、修改/etc/sysctl.conf** 

```shell
# 在文件中增加下面内容
vm.max_map_count=655360
```

**6、加载文件**

```shell
sysctl -p
```

**7、启动软件**

```shell
cd /opt/module/es-cluster
# 启动
bin/elasticsearch
# 后台启动
bin/elasticsearch -d
```

防火墙的问题前面已经提到了

**8、集群验证**

 ![img](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000228608-371172604.png) 





## 分片、副本、分配



### 分片 shards - 重要

**这玩意儿就类似于关系型中的分表**

在关系型中如果一个表的数据太大了，查询效率很低、响应很慢，所以就会采用大表拆小表，如：用户表，不可能和用户相关的啥子东西都放在一张表吧，这不是找事吗？因此：需要分表

相应的在ES中，也需要像上面这么干，如：存储100亿文档数据的索引，在单节点中没办法存储这么多的文档数据，所以需要进行切割，就是将这整个100亿文档数据切几刀，然后每一刀切分出来的每份数据就是一个分片 （ 索引 ），然后在切开的每份数据单独放在一个节点中，这样切开的所有文档数据合在一起就是一份完整的100亿数据，因此：这个的作用也是为了提高效率

**创建一个索引的时候，可以指定想要的分片的数量。每个分片本身也是一个功能完善并且独立的“索引”，这个“索引”可以被放置到集群中的任何节点上**



**分片有两方面的原因：**

- 允许水平分割 / 扩展内容容量，水平扩充，负载均衡嘛
- 允许在分片之上进行分布式的、并行的操作，进而提高性能 / 吞吐量



**注意： 当 Elasticsearch 在索引中搜索的时候， 它发送查询到每一个属于索引的分片，然后合并每个分片的结果到一个全局的结果集中**





### 副本 Replicas - 重要

**这不是游戏中的刷副本的那个副本啊。是指：分片的复制品**

失败是常有的事嘛，所以：在ES中也会失败呀，可能因为网络、也可能因此其他鬼原因就导致失败了，此时不就需要一种故障转移机制吗，也就是 **创建分片的一份或多份拷贝，这些拷贝就叫做复制分片( 副本 )** 



**副本（ 复制分片 ）之所以重要，有两个原因：**

- 在分片 / 节点失败的情况下，**提供了高可用性。因为这个原因，复制分片不与原 / 主要（ original / primary ）分片置于同一节点上是非常重要的**
- 扩展搜索量 / 吞吐量，因为搜索可以在所有的副本上并行运行





多说一嘴啊，分片和副本这两个不就是配套了吗，分片是切割数据，放在不同的节点中（ 服务中 ）；副本是以防服务宕掉了，从而丢失数据，进而把分片拷贝了任意份。这个像什么？不就是主备吗（ 我说的是主备，不是主从啊 ，这两个有区别的，主从是主机具有写操作，从机具有读操作；而主备是主机具有读写操作，而备机只有读操作 ，不一样的啊 ）





**有个细节需要注意，在ES中，分片和副本不是在同一台服务器中，是分开的，如：分片P1在节点1中，那么副本R1就不能在节点1中，而是其他服务中，不然服务宕掉了，那数据不就全丢了吗**





### 分配  Allocation

前面讲到了分片和副本，对照Redis中的主备来看了，那么对照Redis的主从来看呢？主机宕掉了怎么重新选一个主机？Redis中是加了一个哨兵模式，从而达到的。那么在ES中哪个是主节点、哪个是从节点、分片怎么去分的？就是利用了分配

**所谓的分配是指： 将分片分配给某个节点的过程，包括分配主分片或者副本。如果是副本，还包含从主分片复制数据的过程。注意：这个过程是由 master 节点完成的，和Redis还是有点不一样的啊**





既然都说了这么多，那就再来一个ES的系统架构吧

![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000229228-1486448986.png)



其中，**P表示分片、R表示副本**

**默认情况下，分片和副本都是1，根据需要可以改变**







## 单节点集群

这里为了方便就使用window版做演示，就不再linux中演示了



**1、打开前面玩的window版集群的1节点**

 ![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000228550-528032722.png) 



**2、创建索引  把这个索引切成3份（ 切片 ）、每份拷贝1份副本**

```json
PUT http://127.0.0.1:1001/users

// 请求体内容
{
    "settings" : {
        "number_of_shards" : 3,
        "number_of_replicas" : 1
    }
}
```



**3、开始安装head插件，这就是一个可视化界面而已，后续还会用Kibana**

还有一种es的集群监控的方式是使用cerebro，官网地址：https://github.com/lmenezes/cerebro 下载解压，运行 bin/cerebro.bat 即可

自行到官网下载elasticsearch-head-master，这是用Vue写的。启动效果如下：

![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000228557-274732068.png) 



访问上图中的地址即可，**但是：这个端口是9100，而我们的ES是9200端口，所以9100访问9200是跨越的，因此：需要对ES设置跨越问题，而这个问题在第一次玩ES集群时就配置了的**

![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000228639-1850737104.png) 





head打开之后就是下图中的样子

![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000229337-492049169.png)




head链接ES之后就是下图的样子

![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000230141-1297348078.png)




三种颜色再巩固一下：

- **green**：所有的主分片和副本分片都正常运行
- **yellow**：所有的主分片都正常运行，但不是所有的副本分片都正常运行
- **red**：有主分片没能正常运行



但是：上述的单节点集群有问题，就是将分片和副本都放在一个节点（ node-1001 ）中了，这样会导致前面说的服务宕掉，数据就没了，做的副本就是无用功。要解决就要引入接下来的内容了









## 故障转移

> 所谓的故障转移指的就是：
>
> 1. 若新开节点，那么ES就会将原有数据重新分配到所有节点上
> 2. 若是节点挂了，那么ES就会将挂了的节点的数据进行拷贝到另外好的节点中。要是挂的正好是master主节点，那么还有多一个选主过程，然后再分配数据————这种情况也可以称之为“应对故障”



**1、新开节点的情况：** 启动node-1002节点

可能由于玩windows版时的一些数据导致node-1002节点启动不了，所以删掉data文件夹和logs文件夹下的东西即可

刷新head可视化页面：

![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000229099-1217753837.png)

恢复正常











## 水平扩容 / 负载均衡

**1、启动node-1003节点**

![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000229213-633979197.png)



刷新head页面

![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000229835-1314285878.png)



对照前面单节点集群来看，数据就被很好的分开了，这样性能不就提上来了吗



**但是：如果相应继续扩容呢？即：超过6份数据（ 6个节点，前面讲到过索引切分之后，每一份又是单独的索引、副本也算节点 ），那怎么办？**

- 首先知道一个点：主分片的数目在索引创建时就已经确定下来了的，这个我们没法改变，这个数目定义了这个索引能够存储的最大数据量（ 实际大小取决于你的数据、硬件和使用场景 ）
- 但是，读操作——搜索和返回数据——可以同时被主分片 或 副本分片所处理，==所以当你拥有越多的副本分片时，也将拥有越高的吞吐量==
- **因此：增加副本分片的数量即可**

```json
put http://127.0.0.1:1001/users/_settings

// 请求体内容
{
    "number_of_replicas": 2
}
```



刷新head页面

![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000229214-1758584411.png)







## 应对故障

应对的是什么故障？前面一直在说：服务宕掉了嘛



**1、关掉node-1001节点（ 主节点 ）**

**2、刷新head页面**

![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000229968-1318340200.png)



**但是注意啊：yellow虽然不正常，但是不影响操作啊，就像你看了yellow之后，影响你正常发挥吗？只是可能有点虚脱而已，所以对于ES来说也是可以正常查询数据的，只是：效率降低了而已嘛（ 主节点和3个分片都在的嘛 ）**



**3、解决这种问题：** 开启新节点（把node-1001节点启动。此时它就不是主节点了 ，当成新节点了)



![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000230069-419905273.png)



这就会报错： unless existing master is discovered 找不到主节点（ 对于启动的集群来说，它现在是新节点]，因此：需要做一下配置修改（ node-1001的 config/ElasticSearch.yml ）

```yml
discovery.seed_hosts: ["127.0.0.1:9302","127.0.0.1:9303"]
```

保存开启node-1001节点即可



**4、刷新head页面**

![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000229886-1031785461.png)



**故障恢复了，所以：这也告知一个问题，配置集群时，最好在每个节点的配置文件中都加上上述的配置，从而节点宕掉之后，重启节点即可（ 不然每次改不得烦死 ），注意：ES版本不一样，这个配置方法不一样的，6.x的版本是用cluster.initial_master_nodes: 来进行配置的**







## 路由计算和分片控制理论

### 路由计算

路由、路由，这个东西太熟悉了，在Vue中就见过路由router了（ 用来转发和重定向的嘛 ）

那在ES中的路由计算又是怎么回事？**这个主要针对的是ES集群中的存数据，试想：你知道你存的数据是在哪个节点 / 哪个主分片中吗（ 副本是拷贝的主分片，所以主分片才是核心 ）？**

- 当然知道啊，就是那几个节点中的任意一个嘛。娘希匹~这样的骚回答好吗？其实这是由一个公式来决定的

```json
shard = hash( routing ) % number_of_primary_shards

routing 是一个任意值，默认是文档的_id，也可以自定义

number_of_primary_shards 表示主分片的数量，如前面切分为了3份

hash() 是一个hash函数
```

这就解释了为什么我们要在创建索引的时候就确定好主分片的数量并且永远不会改变这个数量：因为如果数量变化了，那么之前所有路由的值都会无效，文档也再也找不到了





### 分片控制

既然有了存数据的问题，那当然就有取数据的问题了。

**请问：在ES集群中，取数据时，ES怎么知道去哪个节点中取数据（ 假如在3节点中，你去1节点中，可以取到吗？），因此：来了分片控制**



负载均衡，轮询嘛。所以这里有个小知识点，就是：协调节点 `coordinating node`，**我们可以发送请求到集群中的任一节点，==每个节点都有能力处理任意请求，每个节点都知道集群中任一文档位置==，这就是分片控制，而我们发送请求的那个节点就是：协调节点，它会去帮我们找到我们要的数据在哪里**



综合前面的知识就可以得到：

1. 所谓的分片就是：将索引切分成任意份嘛，然后得到的每一份数据都是一个单独的索引

2. 分片完成后，我们存数据时，存到哪个节点上，就是通过 shard = hash( routing ) % number_of_primary_shards 得到的

3. 而我们查询数据时，ES怎么知道我们要找的数据在哪个节点上，就是通过**协调节点**做到的，它会去找到和数据相关的“所有节点”，从而轮询，然后进行数据整合，通过协调节点返回给客户端。因此最后的结果可能是从主分片上得到的，也可能是从副本上得到的，就看最后轮询到的是哪个节点罢了



## 集群下的数据写流程

新建、删除请求都是写操作， 必须在主分片上面完成之后才能被复制到相关的副本分片

**整个流程也很简单**

1. 客户端请求任意节点（协调节点）
2. 通过路由计算，协调节点把请求转向指定的节点
3. 转向的节点的主分片保存数据
4. 主节点再将数据转发给副本保存
5. 副本给主节点反馈保存结果
6. 主节点给客户端反馈保存结果
7. 客户端收到反馈结果

![image-20230621160556854](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000230017-1073460259.png)

但是：从图中就可以看出来，这套流程完了，才可以做其他事（ 如：才可以去查询数据 ），那我为什么不可以异步呢？就是我只要保证到了哪一个步骤之后，就可以进行数据查询，所以：这里有两个小东西需要了解

在进行写数据时，我们做个小小的配置，这就是接下来的两个小节内容






### 一致性 consistency

这玩意就是为了和读数据搭配起来嘛，写入和读取保证数据的一致性呗

**这玩意儿可以设定的值如下：**

- one ：只要主分片状态 ok 就允许执行读操作，这种写入速度快，但不能保证读到最新的更改
- all：这是强一致性，必须要主分片和所有副本分片的状态没问题才允许执行写操作
- quorum：这是ES的默认值。即大多数的分片副本状态没问题就允许执行写操作。这是折中的方法，write的时候，W>N/2，即参与写入操作的节点数W，必须超过副本节点数N的一半，在这个默认情况下，ES是怎么判定你的分片数量的，就一个公式：

```txt
int((primary + number_of_replicas) / 2) + 1

primary						指的是创建的索引数量

number_of_replicas			是指的在索引设置中设定的副本分片数
							如果你的索引设置中指定了当前索引拥有3个副本分片
							那规定数量的计算结果为：int(1 primary + 3 replicas) / 2) + 1 = 3，
							如果此时你只启动两个节点，那么处于活跃状态的分片副本数量就达不到规定数量，
							也因此你将无法索引和删除任何文档
```

- realtime request：就是从translog里头读，可以保证是最新的。**但是注意：get是最新的，但是检索等其他方法不是( 如果需要搜索出来也是最新的，需要refresh，这个会刷新该shard但不是整个index，因此如果read请求分发到repliac shard，那么可能读到的不是最新的数据，这个时候就需要指定preference=_primar y)**





### 超时 timeout

如果没有足够的副本分片会发生什么？Elasticsearch 会等待，希望更多的分片出现。默认情况下，它最多等待 1 分钟。 如果你需要，你可以使用timeout参数使它更早终止，单位是毫秒，如：100就是100毫秒

新索引默认有1个副本分片，这意味着为满足规定数量应该需要两个活动的分片副本。 但是，这些默认的设置会阻止我们在单一节点上做任何事情。为了避免这个问题，要求只有当number_of_replicas 大于1的时候，规定数量才会执行



上面的理论不理解、或者感觉枯燥也没事儿，后面慢慢的就理解了，这里只是打个预防针、了解理论罢了







## 集群下的数据读流程

有写流程，那肯定也要说一下读流程嘛，其实和写流程很像，只是变了那么一丢丢而已

**流程如下：**

- 客户端发送请求到任意节点（ 协调节点 ）
- 这里不同，此时协调节点会做两件事：1、通过路由计算得到分片位置，2、还会把当前查询的数据所在的另外节点也找到（ 如：副本 ）
- 为了负载均衡（ 可能某个节点中的访问量很大嘛，减少一下压力咯 ），所以就会对查出来的所有节点做轮询操作，从而找到想要的数据（ 因此：你想要的数据在主节点中有、副本中也有，但是：给你的数据可能是主节点中的，也可能是副本中的 ———— 看轮询到的是哪个节点中的 ）
- 节点反馈结果
- 客户端收到反馈结果

![image-20230619202223102](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000229608-2147419270.png)



**这里有个注意点：** 在文档( 数据 ）被检索时，已经被索引的文档可能已经存在于主分片上但是还没有复制到副本分片。 在这种情况下，副本分片可能会报文档不存在，但是主分片可能成功返回文档。 一旦索引请求成功返回给用户，文档在主分片和副本分片都是可用的







## 集群下的更新操作流程

### 更新操作流程

![image-20230619202310833](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000230209-1656037049.png)



1. 客户端向node 1发送更新请求
2. 它将请求转发到主分片所在的node 3 
3. node 3从主分片检索文档，修改_source字段中的JSON，并且尝试重新索引主分片的文档。如果文档已经被另一个进程修改,它会重试步骤3 ,超过retry_on_conflict次后放弃
4. 如果 node 3成功地更新文档，它将新版本的文档并行转发到node 1和 node 2上的副本分片，重新建立索引。一旦所有副本分片都返回成功，node 3向协调节点也返回成功，协调节点向客户端返回成功



当然：上面有个漏洞，就是万一在另一个进程修改之后，当前修改进程又去修改了，那要是把原有的数据修改了呢？这不就成关系型数据库中的“不可重复读”了吗？

- 不会的。因为当主分片把更改转发到副本分片时， 它不会转发更新请求。 相反，它转发完整文档的新版本。注意点：这些更改将会“异步转发”到副本分片，并且不能保证它们以相同的顺序到达。 如果 ES 仅转发更改请求，则可能以错误的顺序应用更改，导致得到的是损坏的文档





### 批量更新操作流程

这个其实更容易理解，单文档更新懂了，那多文档更新就懂了嘛，多文档就请求拆分呗

**所谓的多文档更新就是：将整个多文档请求分解成每个分片的文档请求，并且将这些请求并行转发到每个参与节点。协调节点一旦收到来自每个节点的应答，就将每个节点的响应收集整理成单个响应，返回给客户端**



原理图的话：我就在网上偷一张了
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000229530-995704350.png)



其实mget 和 bulk API的模式就类似于单文档模式。区别在于协调节点知道每个文档存在于哪个分片中



**用单个 mget 请求取回多个文档所需的步骤顺序:**

1. 客户端向 Node 1 发送 mget 请求
2. Node 1为每个分片构建多文档获取请求，然后并行转发这些请求到托管在每个所需的主分片或者副本分片的节点上。一旦收到所有答复，Node 1 构建响应并将其返回给客户端。可以对docs数组中每个文档设置routing参数

- bulk API， 允许在单个批量请求中执行多个创建、索引、删除和更新请求

 ![img](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000230024-1242411568.png) 





**bulk API 按如下步骤顺序执行：**

1. 客户端向Node 1 发送 bulk请求
2. Node 1为每个节点创建一个批量请求，并将这些请求并行转发到每个包含主分片的节点主机
3. 主分片一个接一个按顺序执行每个操作。当每个操作成功时,主分片并行转发新文档（或删除）到副本分片，然后执行下一个操作。一旦所有的副本分片报告所有操作成功，该节点将向协调节点报告成功，协调节点将这些响应收集整理并返回给客户端





## 文档搜索

### 不可变的倒排索引

以前的全文检索是将整个文档集合弄成一个倒排索引，然后存入磁盘中，当要建立新的索引时，只要新的索引准备就绪之后，旧的索引就会被替换掉，这样最近的文档数据变化就可以被检索到

而索引一旦被存入到磁盘就是不可变的（ 永远都可以修改 ），而这样做有如下的好处：

1. 只要索引被读入到内存中了，由于其不变性，所以就会一直留在内存中（ 只要空间足够 ），从而当我们做“读操作”时，请求就会进入内存中去，而不会去磁盘中，这样就减小开销，提高效率了
2. 索引放到内存中之后，是可以进行压缩的，这样做之后，也就可以节约空间了
3. 放到内存中后，是不需要锁的，如果自己的索引是长期不用更新的，那么就不用怕多进程同时修改它的情况了



当然：这种不可变的倒排索引有好处，那就肯定有坏处了

- 不可变，不可修改嘛，这就是最大的坏处，当要重定一个索引能够被检索时，就需要重新把整个索引构建一下，这样的话，就会导致索引的数据量很大（ 数据量大小有限制了 ），同时要更新索引，那么这频率就会降低了
- 这就好比是什么呢？关系型中的表，一张大表检索数据、更新数据效率高不高？肯定不高，所以延伸出了：可变索引



### 可变的倒排索引

**又想保留不可变性，又想能够实现倒排索引的更新，咋办？**

- 就搞出了`补充索引`，**所谓的补充索引：有点类似于日志这个玩意儿，就是重建一个索引，然后用来记录最近指定一段时间内的索引中文档数据的更新。**这样更新的索引数据就记录在补充索引中了，然后检索数据时，直接找补充索引即可，这样检索时不再重写整个倒排索引了，这有点类似于关系型中的拆表，大表拆小表嘛，**但是啊：每一份补充索引都是一份单独的索引啊，这又和分片很像，可是：查询时是对这些补充索引进行轮询，然后再对结果进行合并，从而得到最终的结果，这和前面说过的读流程中说明的协调节点挂上钩了**



**这里还需要了解一个配套的`按段搜索`，玩过 Lucene 的可能听过。按段，每段也就可以理解为：补充索引，它的流程其实也很简单：**

1. 新文档被收集到内存索引缓存
2. 不时地提交缓存
   1. 一个新的段，一个追加的倒排索引，被写入磁盘
   2. 一个新的包含新段名字的提交点被写入磁盘
3. 磁盘进行同步，所有在文件系统缓存中等待的写入都刷新到磁盘，以确保它们被写入物理文件
4. 内存缓存被清空，等待接收新的文档
5. 新的段被开启，让它包含的文档可见，以被搜索





一样的，段在查询的时候，也是轮询的啊，然后把查询结果合并从而得到的最终结果

另外就是涉及到删除的事情，**段本身也是不可变的， 既不能把文档从旧的段中移除，也不能修改旧的段来进行文档的更新，而删除是因为：是段在每个提交点时有一个.del文件，这个文件就是一个删除的标志文件，要删除哪些数据，就对该数据做了一个标记，从而下一次查询的时候就过滤掉被标记的这些段，从而就无法查到了，这叫逻辑删除（ 当然：这就会导致倒排索引越积越多，再查询时。轮询来查数据也会影响效率 ），所以也有物理删除，它是把段进行合并，这样就舍弃掉被删除标记的段了，从而最后刷新到磁盘中去的就是最新的数据（ 就是去掉删除之后的 ，别忘了前面整的段的流程啊，不是白写的 ）**







## 近实时搜索、文档刷新、文档刷写、文档合并

> **ES的最大好处就是实时数据全文检索**
>
> 但是：ES这个玩意儿并不是真的实时的，而是近实时 / 准实时
>
> 原因就是：ES的数据搜索是分段搜索，最新的数据在最新的段中(每一个段又是一个倒排索引)，只有最新的段刷新到磁盘中之后，ES才可以进行数据检索，这样的话，磁盘的IO性能就会极大的影响ES的查询效率，而ES的目的就是为了：快速的、准确的获取到我们想要的数据，因此：降低数据查询处理的延迟就very 重要了，而ES对这方面做了什么操作？
>
> - 就是搞的**一主多副的方式**(一个主分片，多个副本分片)，这虽然就是一句话概括了，但是：里面的门道却不是那么简单的







**首先来看一下主副操作**
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000229794-123533688.png)

但是：这种去找寻节点的过程想都想得到会造成延时，而**延时 = 主分片延时 + 主分片拷贝数据给副本的延时**

而且并不是这样就算完了，前面提到了N多次的分段、刷新到磁盘还没上堂呢，所以接着看
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000229411-445938052.png)



但是：在flush到磁盘中的时候，万一断电了呢？或者其他原因导致出问题了，那最后数据不就没有flush到磁盘吗

因此：其实还有一步操作，把数据保存到另外一个文件中去
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000230033-2132748470.png)



数据放到磁盘中之后，translog中的数据就会清空

同时更新到磁盘之后，用户就可以进行搜索数据了



**注意：**这里要区分一下，数据库中是先更新到log中，然后再更新到内存中，而ES是反着的，是先更新到Segment（ 可以直接认为是内存，因它本身就在内存中 ），再更新到log中



可是啊，还是有问题，flush刷写到磁盘是很耗性能的，假如：不断进行更新呢？这样不断进行IO操作，性能好吗？也不行，因此：继续改造(没有什么是加一层解决不了的，一层不够，那就再来一层)

![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000230213-1598498312.png)



加入了缓存之后，这缓存里面的数据是可以直接用来搜索的，这样就不用等到flush到磁盘之后，才可以搜索了，这大大的提高了性能，而flush到磁盘，只要时间到了，让它自个儿慢慢flush就可以了，**上面这个流程也叫：持久化 / 持久化变更**



**写入和打开一个新段的轻量的过程叫做refresh。默认情况下每个分片会每秒自动刷新一次。这就是为什么我们说 ES是近实时搜索：文档的变化并不是立即对搜索可见，但会在一秒之内变为可见**

刷新是1s以内完成的，这是有时间间隙的，所以会造成：搜索一个文档时，可能并没有搜索到，因此：解决办法就是使用refresh API刷新一下即可

**但是这样也伴随一个问题：虽然这种从内存刷新到缓存中看起来不错，但是还是有性能开销的。并不是所有的情况都需要refresh的，** 假如：是在索引日志文件呢？去refresh干嘛，浪费性能而已，所以此时：你要的是查询速度，而不是近实时搜索，因此：可以通过一个配置来进行改动，从而降低每个索引的刷新频率

```json
http://ip:port/index_name/_settings		// 请求方式：put

// 请求体内容
{
    "settings": {
        "refresh_interval": "60s"
    }
}
```

refresh_interval 可以在既存索引上进行动态更新。在生产环境中，当你正在建立一个大的新索引时，可以先关闭自动刷新，待开始使用该索引时，再把它们调回来。虽然有点麻烦，但是按照ES这个玩意儿来说，确实需要这么做比较好

```json
// 关闭自动刷新
http://ip:port/users/_settings		// 请求方式：put

// 请求体内容
{ 
    "refresh_interval": -1 
}

// 每一秒刷新
http://ip:port/users/_settings		// 请求方式：put
// 请求体内容
{ 
    "refresh_interval": "1s" 
}
```



另外：不断进行更新就会导致很多的段出现（在内存刷写到磁盘那里，会造成很多的磁盘文件），因此：在哪里利用了文档合并的功能(也就是段的能力，合并文档，从而让刷写到磁盘中的文档变成一份)









## 文档分析

试想：我们在浏览器中，输入一条信息，如：搜索“博客园紫邪情”，为什么连“博客园也搜索出来了？我要的是不是这个结果涩”
![image-20230621232021726](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000230158-1026624158.png)



这就是全文检索，就是ES干的事情（ 过滤数据、检索嘛 ），但是：它做了哪些操作呢？



在ES中有一个**文档分析的过程**，文档分析的过程也很简单：

1. **将文本拆成适合于倒排索引的独立的词条，然后把这些词条统一变为一个标准格式，从而使文本具有“可搜索性”。** 而这个文档分析的过程在ES是由一个叫做“分析器 analyzer”的东西来做的，这个分析器里面做了三个步骤
   1. 字符过滤器：就是用来处理一些字符的嘛，像什么将 & 变为 and 啊、去掉HTML元素啊之类的。它是文本字符串在经过分词之前的一个步骤，文本字符串是按文本顺序经过每个字符串过滤器从而处理字符串
   2. 分词器：见名知意，就是用来分词的，也就是将字符串拆分成词条（ 字 / 词组 ），这一步和Java中String的split()一样的，通过指定的要求，把内容进行拆分，如：空格、标点符号
   3. Token过滤器：这个玩意儿的作用就是 词条经过每个Token过滤器，从而对数据再次进行筛选，如：字母大写变小写、去掉一些不重要的词条内容、添加一些词条（ 如：同义词 ）



上述的内容不理解没事，待会儿会用IK中文分词器来演示，从而能够更直观的看到效果



在ES中，有提供好的内置分析器、我们也可以自定义、当然还有就是前面说的IK分词器也可以做到。而这里重点需要了解的就是IK中文分词器



在演示在前，先玩kibana吧，原本打算放在后面的，但是越早熟悉越好嘛，所以先把kibana说明了





### kibana

**1、去Elastic [官网](https://www.elastic.co/cn/downloads/?elektra=home&storm=hero) 下载kibana。** 但是需要注意：==kibana的版本必须和ES的版本一致==

下载好了kibana之后，解压到自己想要的目录（ 注：加压会有点久，因为是用Vue写的，里面有模块module 要是加压快的话，可能还下错了 ），然后点击bin/kibana.bat即可启动kibana、第一次进去会有一个选择页面，add … / explore，选择explore就可以了，进去之后就是如下界面：
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000231356-2124430229.png)



这是英文版，要是没玩过大数据的话，那么里面的一些专业名词根据英文来看根本不知道

所以：**汉化吧。** kibana本身就提供得有汉化的功能，只需要改动一个配置即可。就是一个i1bn配置而已

**进入config/kibana.yml，刷到最底部**

![image-20230619192916919](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000230144-627303359.png)



加上上面的信息，然后重启kibana就可以了

但是：个人建议，先汉化一段时间，等熟悉哪些名词了，然后再转成英文 ，总之最后建议用英文，一是增加英文词汇量，二是熟悉英文专业词



**kibana遵循的是rest风格（ get、put、delete、post..... ），具体用法接下来玩分析器和后面都会慢慢熟悉**





### 内置分析器

#### 标准分析器 standard

**这是根据Unicode定义的单词边界来划分文本，将字母转成小写，去掉大部分的标点符号，从而得到的各种语言的最常用文本选择，==另外：这是ES的默认分析器==。** 接下来演示一下



1、启动ES和kibana，打开控制台
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000231190-580712443.png)



2、编写指令
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000230555-1183470716.png)

```json
GET _analyze
{
    "analyzer": "standard", // analyzer 分析器  standard 标准分析器
    "text": "my name is ZiXieQing" // text 文本标识   my name is ZiXieQing 自定义的文本内容
}


// 响应内容
{
    "tokens" : [
        {
            "token" : "my",		// 分词之后的词条
            "start_offset" : 0,
            "end_offset" : 2,		// start和end叫偏移量
            "type" : "<ALPHANUM>",
            "position" : 0	// 当前词条在整个文本中所处的位置
        },
        {
            "token" : "name",
            "start_offset" : 3,
            "end_offset" : 7,
            "type" : "<ALPHANUM>",
            "position" : 1
        },
        {
            "token" : "is",
            "start_offset" : 8,
            "end_offset" : 10,
            "type" : "<ALPHANUM>",
            "position" : 2
        },
        {
            "token" : "zixieqing",
            "start_offset" : 11,
            "end_offset" : 20,
            "type" : "<ALPHANUM>",
            "position" : 3
        }
    ]
}
```



![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000229930-500261460.png)



从上图可以看出：所谓标准分析器是将文本通过标点符号来分词的（ 空格、逗号... ，不信可以自行利用这些标点测试一下，观察右边分词的结果 ），同时大写转小写







#### 简单分析器 simple

**简单分析器是“按非字母的字符分词，例如：数字、标点符号、特殊字符等，会去掉非字母的词，大写字母统一转换成小写”**
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000230910-910185186.png)





#### 空格分析器 whitespace

**是简单按照空格进行分词，相当于按照空格split了一下，大写字母不会转换成小写**
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000230432-1470145706.png)







#### 去词分析器 stop

**会去掉无意义的词（此无意义是指语气助词等修饰性词，补语文：语气词是疑问语气、祈使语气、感叹语气、肯定语气和停顿语气），例如：the、a、an 、this等，大写字母统一转换成小写**
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000230043-81554053.png)





#### 不拆分分析器 keyword

**就是将整个文本当作一个词**
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000230849-306225142.png)





### IK中文分词器

来个实验：
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000231504-376578282.png)



它把我的名字进行拆分了，这不是我想要的，我想要的“紫邪情”应该是一个完整的词，同样道理：**想要特定的词汇，如：ID号、用户名....，这些不应该拆分，而ES内置分析器并不能做到，所以需要IK中文分词器（专门用来处理中文的 ）**





**1、下载IK分词器：** https://github.com/medcl/elasticsearch-analysis-ik/releases/tag/v7.8.0

**注意：版本对应关系，** 还是和ES版本对应，https://github.com/medcl/elasticsearch-analysis-ik 这个链接进去之后有详细的版本对应
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000230743-1315918433.png)









**2、把IK解压到ES/plugins中去。** 如我的：
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000231784-785897373.png)



**3、重启ES即可。**  kibana开着的话，也要关了重启 ，注意观察：重启时会有一个IK加载过程
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000230750-1178181323.png)



经过如上的操作之后，IK中文分词器就配置成功了，接下来就来体验一下（ 启动ES和kibana ），主要是为了了解IK中的另外两种分词方式：**ik_max_word 和 ik_smart**


- **ik_max_word		是细粒度的分词，就是：穷尽词汇的各种组成。** 4个字是一个词，继续看3个字是不是一个词，再看2个字又是不是一个词，以此穷尽..........
  ![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000230977-604585597.png)



- **ik_smart			是粗粒度的分词。** 如：那个叼毛也是一个程序员，就先看整句话是不是一个词(length = 11)，不是的话，就看length-1是不是一个词.....，如果某个长度是一个词了，那么这长度内的内容就不看了，继续看其他的是不是一个词，如“那个"是一个词，那就看后面的内容，继续length、length-1、length-2........
  ![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000231083-702192252.png)





回到前面的问题，“紫邪情”是名字，我不想让它分词，怎么做？上面哪些分词都是在一个“词典”中，所以我们自己搞一个词典即可

**1、创建一个.dic文件  dic就是dictionary词典的简写**
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000230842-616365710.png)



**2、在创建的dic文件中添加不分词的词组，保存**
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000230445-594840499.png)



**3、把自定义的词典放到ik中去，保存**
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000230956-1301907456.png)



**4、重启ES和kibana**

**5、测试**
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000230698-1664061840.png)



可见，现在就把“紫邪情”组成词组不拆分了，前面玩的kibana汉化是怎么做的？和这个的原理差不多









### 多玩几次kibana

在第一篇高级篇中我边说过：kibana重要，只是经过前面这些介绍了使用之后，并不算熟悉，因此：多玩几次吧

另外：就是前面说的kibana遵循rest风格，在ES中是怎么玩的？总结下来其实就下面这些，要上手简单得很，但理论却是一直弄到现在
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000231215-14108994.png)



现在用kibana来演示几个，其他在postman中怎么弄，换一下即可（ 其实不建议用postman测试，专业的人做专业的事，kibana才是我们后端玩的 ）



**1、创建索引**
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000230798-995800117.png)



**2、查看索引**
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000230882-19004852.png)



**3、创建文档（ 随机id值，想要自定义id值，在后面加上即可 ）**

**4、删除索引**
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000231029-1738132378.png)



**5、创建文档（ 自定义id ）**
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000230949-1098773880.png)



**6、查看文档（ 通过id查询 ）**
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000230947-885723701.png)



**7、修改文档（ 局部修改 ）**
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000231071-1840753096.png)



验证一下：
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000231272-1085429581.png)



**8、建字段类型**
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000230948-690072417.png)



其他的也是差不多的玩法，在基础篇中怎么玩，稍微变一下就是kibana的玩法了







### 拼音分词器/自定义分析器

官网：https://github.com/medcl/elasticsearch-analysis-pinyin

安装和IK分词器一样

- 下载
- 上传解压
- 重启es



测试拼音分词器

![image-20230627210119445](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000231116-1544979830.png)

由上可知，伴随2个问题：

1. 只进行了拼音分词，汉字分词不见了
2. 只采用拼音分词会出现一种情况：同音字，如“狮子”，“虱子”，这样的话明明想搜索的是“狮子”，结果“虱子”也出来了，所以这种搜索效果不好



因此：需要定制，让汉字分词出现，同时搜索时使用的汉字是什么就是什么，别弄同音字

要完成上面的需求，就需要结合前面的文档分析的过程

在ES中有一个**文档分析的过程**，文档分析的过程也很简单：

1. **将文本拆成适合于倒排索引的独立的词条，然后把这些词条统一变为一个标准格式，从而使文本具有“可搜索性”。** 而这个文档分析的过程在ES是由一个叫做“分析器 analyzer”的东西来做的，这个分析器里面做了三个步骤
   1. 字符过滤器(character filters)：就是用来处理一些字符的嘛，像什么将 & 变为 and 啊、去掉HTML元素啊之类的。它是文本字符串在经过分词之前的一个步骤，文本字符串是按文本顺序经过每个字符串过滤器从而处理字符串
   2. 分词器(tokenizer)：见名知意，就是用来分词的，也就是将字符串拆分成词条（ 字 / 词组 ），这一步和Java中String的split()一样的，通过指定的要求，把内容进行拆分，如：空格、标点符号
   3. Token过滤器(tokenizer filter)：这个玩意儿的作用就是 词条经过每个Token过滤器，从而对数据再次进行筛选，如：字母大写变小写、去掉一些不重要的词条内容、添加一些词条（ 如：同义词 ）



举例理解：character filters、tokenizer、tokenizer filter)

![image-20210723210427878](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000231009-17318988.png)

因此现在自定义分词器就变成如下的样子：

**注：** 是建立索引时自定义分词器，即自定义的分词器只对当前索引库有效

```json
PUT /test
{
  "settings": {
    "analysis": {
      "analyzer": { // 自定义分词器
        "my_analyzer": {  // 分词器名称
          "tokenizer": "ik_max_word",
          "filter": "py"
        }
      },
      "filter": { // 自定义tokenizer filter
        "py": { // 过滤器名称
          "type": "pinyin", // 过滤器类型，这里是pinyin，这些参数都在 拼音分词器官网有
		  "keep_full_pinyin": false,
          "keep_joined_full_pinyin": true,
          "keep_original": true,
          "limit_first_letter_length": 16,
          "remove_duplicated_term": true,
          "none_chinese_pinyin_tokenize": false
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "name": {
        "type": "text",
        "analyzer": "my_analyzer",	// 指明在索引时使用的分词器
        "search_analyzer": "ik_smart"	// 指明搜索时使用的分词器
      }
    }
  }
}
```



使用自定义分词器：

![image-20230627212610200](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000231208-1118735277.png)

















## 文档控制-了解

**所谓的文档控制就是：不断更新的情况，试想：多进程不断去更新文档，会造成什么情况？会把其他人更新过的文档进行覆盖更新了，而ES是怎么解决这个问题的？**



**就是弄了一个锁来实现的，和Redis一样，也是用的乐观锁来实现的，这个其实没什么好说的，只需要看一下就知道了**
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000231143-1375649794.png)



上图中·的三个字段就和锁挂钩的，version，版本号嘛，每次更新都会有一个版本号，这样就解决了多进程修改从而造成的文档冲突了（ 必须等到一个进程更新完了，另一个进程才可以更新 ），当然：需要注意旧版本的ES在请求中加上version即可，但是新版本的ES需要使用  if"_seq_no" 和"if_primary_term" 来达到version的效果





## ES的优化

**ES的所有索引和文档数据都是存储在本地的磁盘中的，所以：磁盘能处理的吞吐量越大，节点就越稳定**



**要修改的话，是在config/elasticsearch.yml中改动**
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000231135-1505585999.png)





### 硬件方面

1、选用固态硬盘（ 即：SSD ），它比机械硬盘的好是因为：机械硬盘是通过旋转马达的驱动来进行的，所以这就会造成发热、磨损，就会影响ES的效率，而SSD是使用芯片式的闪存来存储数据的，性能比机械硬盘好得多

2、使用RAID 0 （ 独立磁盘冗余阵列 ），它是把连续的数据分散到多个磁盘上存取，这样，系统有数据请求就可以被多个磁盘并行的执行，每个磁盘执行属于它自己的那部分数据请求。这种数据上的并行操作可以充分利用总线的带宽，显著提高磁盘整体存取性能

![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000231399-268504957.png)



3、由上面的RAID 0可以联想到另外一个解决方式：使用多块硬盘，也就可以达到同样的效果了（ 有钱就行 ），是通过path data目录配置把数据条分配到这些磁盘上面

4、不要把ES挂载到远程上去存储







### 分片策略

分片和副本不是乱分配的！分片处在不同节点还可以（ 前提是节点中存的数据多 ），这样就类似于关系型中分表，确实可以算得到优化，但是：如果一个节点中有多个分片了，那么就会分片之间的资源竞争，这就会导致性能降低



所以**分片和副本遵循下面的原则**就可以了

1. 每个分片占用的磁盘容量不得超过ES的JVM的堆空间设置（ 一般最大为32G ），假如：索引容量为1024G，那么节点数量为：1024 / 32 = 32左右
2. 分片数不超过节点数的3倍，就是为了预防一个节点上有多个分片的情况，万一当前节点死了，那么就算做了副本，也很容易导致集群丢失数据
3. 节点数 <= 主节点数 * （ 副本数 + 1 ）
4. 推迟分片分配。有可能一个节点宕掉了，但是后面它又恢复了，而这个节点原有数据是还在的，所以：推迟分片分配，从而减少ES的开销，具体做法如下：

```json
PUT /_all/_settings
{
	"settings": {
		"index.unassigned.node_left.delayed_timeout": "5m"
	}
}
```

![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000231171-1873569998.png)



可以全局修改，也可以在建索引时修改





### 带路由查询

前面说过：路由计算公式 `shard = hash( routing ) % number_of_primary_shards`



而routing默认值就是文档id，所以查询时把文档id带上，如：前面玩kibana做的操作
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000231250-219437370.png)



不带路由就会把分片和副本都查出来，然后进行轮询，这效率想都想得到会慢一点嘛





### 内存优化

**修改es的config/jvm.options**
![image](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629000231213-895296465.png)



把上面的数字改了

**Xms 表示堆的初始大小， Xmx 表示可分配的最大内存，ES默认是1G，这个数字在现实中是远远不够了，改它的目的是：为了能够在 Java 垃圾回收机制清理完堆内存后不需要重新分隔计算堆内存的大小而浪费资源，可以减轻伸缩堆大小带来的压力，但是也需要注意：改这两个数值，需要确保 Xmx 和 Xms 的大小是相同的，另外就是：这两个数值别操作32G啊，前面已经讲过了**







## 附上一些配置说明

| **参数名**                         | **参数值**    | **说明**                                                     |
| ---------------------------------- | ------------- | ------------------------------------------------------------ |
| cluster.name                       | elasticsearch | 配置 ES 的集群名称，默认值是 ES，建议改成与所存数据相关的名称， ES 会自动发现在同一网段下的 集群名称相同的节点 |
| node.name                          | node-1001     | 集群中的节点名，在同一个集群中不能重复。节点 的名称一旦设置，就不能再改变了。当然，也可以 设 置 成 服 务 器 的 主 机 名 称 ， 例 如 node.name: ${hostname} |
| node.master                        | true          | 指定该节点是否有资格被选举成为 Master 节点，默 认是 True，如果被设置为 True，则只是有资格成为 Master 节点，具体能否成为 Master 节点，需要通过选举产生 |
| node.data                          | true          | 指定该节点是否存储索引数据，默认为 True。数据的增、删、改、查都是在 Data 节点完成的 |
| index.number_of_shards             | 1             | 设置索引分片个数，默认是 1 片。也可以在创建索引时设置该值，具体设置为多大值要根据数据量的大小来定。如果数据量不大，则设置成 1 时效率最高 |
| index.number_of_replicas           | 1             | 设置默认的索引副本个数，默认为 1 个。副本数越多，集群的可用性越好，但是写索引时需要同步的数据越多 |
| transport.tcp.compress             | true          | 设置在节点间传输数据时是否压缩，默认为 False                 |
| discovery.zen.minimum_master_nodes | 1             | 设置在选举 Master 节点时需要参与的最少的候选主节点数，默认为 1。如果使用默认值，则当网络不稳定时有可能会出现脑裂。 合理的 数 值 为 ( master_eligible_nodes / 2 )+1 ， 其 中 master_eligible_nodes 表示集群中的候选主节点数 |
| discovery.zen.ping.timeout         | 3s            | 设置在集群中自动发现其他节点时 Ping 连接的超时时间，同时也是选主节点的延迟时间，默认为 3 秒。 在较差的网络环境下需要设置得大一点，防止因误判该节点的存活状态而导致分片的转移 |





## 说一些另外的理论吧

### ES的master主节点选举流程

1. 首先选主是由ZenDiscovery来完成的。ZenDiscovery做了两件事：
   1. 一个是Ping过程，发现节点嘛 
   2. 二是Unicast过程，控制哪些节点需要Ping通
2. 对所有可以成为master的节点(文件中设置的node.master: true)根据nodeId字典排序，每次“选举节点(即：参与投票选举主节点的那个节点)”都把自己知道的节点排一次序，就是把排好序的第一个节点(第0位)认为是主节点(投一票)
3. 当某个节点的投票数达到一个值时，此值为 (可以成为master节点数n / 2 ） + 1，而该节点也投自己，那么这个节点就是master节点，否则重新开始，直到选出master



**另外注意：** master节点的职责主要包括集群、节点和索引的管理，不负责文档级别的管理；data节点可以关闭http功能

ES中的节点职责如下：

| **节点类型**    | **配置参数**                             | **默认值** | **节点职责**                                                 |
| --------------- | ---------------------------------------- | ---------- | ------------------------------------------------------------ |
| master eligible | node.master                              | true       | 备选主节点：主节点可以管理和记录集群状态、决定分片在哪个节点、处理创建和删除索引库的请求 |
| data            | node.data                                | true       | 数据节点：存储数据、搜索、聚合、CRUD                         |
| ingest          | node.ingest                              | true       | 数据存储之前的预处理 但：若是已经使用Java代码进行了预处理，那么此配置就无效了 |
| coordinating    | 上面3个参数都为false则为coordinating节点 | 无         | 协调节点，路由请求到其它节点合并其它节点处理的结果，返回给用户 |

默认情况下，集群中的任何一个节点都同时具备上述四种角色



但是真实的集群一定要将集群职责分离：

- master节点：对CPU要求高，但是内存要求第
- data节点：对CPU和内存要求都高
- coordinating节点：对网络带宽、CPU要求高

职责分离可以让我们根据不同节点的需求分配不同的硬件去部署。而且避免业务之间的互相干扰





### ES的集群脑裂问题

> 所谓的脑裂一句话来概括就是老大(master节点)“没了”，然后小弟(有资格成为主节点的节点)重新选举出老大，结果最后旧老大回来了，从而造成新旧老大整合的数据不一样，最后就摊上事儿了



**导致的原因：** 

1. 网络问题：集群间的网络延迟导致一些候选主节点(文件中设置的node.master: true，即：可以成为主节点的节点)访问不到master, 认为master 挂掉了从而选举出新的master,并对master上的分片和副本标红，分配新的主分片
2. 节点负载：主节点的角色既为master又为data,访问量较大时可能会导致ES停止响应造成大面积延迟，此时其他节点得不到主节点的响应认为主节点挂掉了，会重新选取主节点
3. 内存回收：data 节点上的ES进程占用的内存较大，引发JVM的大规模内存回收，造成ES进程失去响应



**脑裂问题解决方案：**

1. **减少误判：discovery.zen ping_ timeout 节点状态的响应时间，默认为3s**。可以适当调大，如果master在该响应时间的范围内没有做出响应应答，判断该节点已经挂掉了。调大参数（ 如6s，discovery.zen.ping_timeout:6 ），可适当减少误判

2. **选举触发：discovery.zen.minimum_master_nodes:1，该参數是用于控制选举行为发生的最小集群主节点数量**。当备选主节点的个數大于等于该参数的值，且备选主节点中有该参数个节点认为主节点挂了，进行选举。官方建议为(n / 2) +1，n为有资格成为主节点的节点个数）

   1. 多提一嘴：为了避免脑裂，要求选票超过 (n+1) / 2 才能当选为主，n为有资格成为主节点的节点个数(即：候选主节点个数)。因此n候选主节点数最好是奇数，对应配置就是上面的 discovery.zen.minimum_master_nodes 。当然，在es7.0以后，已经成为默认配置，es会自动去计算候选主节点的数量，从而进行配置，所以一般不会发生脑裂

3. **角色分离：即master节点与data节点分离，限制角色**

   - 主节点配置为：node master: true，node data: false

   - 从节点置为：node master: false，node data: true













