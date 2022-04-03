# Elasticsearch

## 概念

### 简介

Elasticsearch 是一个开源的搜索引擎，建立在一个全文搜索引擎库 [Apache Lucene™](https://lucene.apache.org/core/) 基础之上。 Lucene 可以说是当下最先进、高性能、全功能的搜索引擎库。

然而，Elasticsearch 不仅仅是 Lucene，并且也不仅仅只是一个全文搜索引擎。 它可以被下面这样准确的形容：

- 一个分布式的实时文档存储，*每个字段* 可以被索引与搜索
- 一个分布式实时分析搜索引擎
- 能胜任上百个服务节点的扩展，并支持 PB 级别的结构化或者非结构化数据
- Elasticsearch 将所有的功能打包成一个单独的服务，简单的 RESTful API 进行通信。

### 面向文档

在应用程序中对象很少只是一个简单的键和值的列表。通常，它们拥有更复杂的数据结构，可能包括日期、地理信息、其他对象或者数组等。

一般把这些对象存储在数据库中。使用关系型数据库的行和列存储，这相当于是把一个表现力丰富的对象塞到一个非常大的电子表格中：为了适应表结构，你必须设法将这个对象扁平化—通常一个字段对应一列—而且每次查询时又需要将其重新构造为对象。

Elasticsearch 是 *面向文档* 的，意味着它存储整个对象或 *文档*。Elasticsearch 不仅存储文档，而且 *索引* 每个文档的内容，使之可以被检索。在 Elasticsearch 中，我们对文档进行索引、检索、排序和过滤—而不是对行列数据。这是一种完全不同的思考数据的方式，也是 Elasticsearch 能支持复杂全文检索的原因。

Elasticsearch 使用 **JavaScript Object Notation（或者 [*JSON*](http://en.wikipedia.org/wiki/Json)）作为文档的序列化格式**。JSON 序列化为大多数编程语言所支持，并且已经成为 NoSQL 领域的标准格式。 它简单、简洁、易于阅读。



### 分布式特性







## 交互

### 基本

一个 Elasticsearch 请求和任何 HTTP 请求一样由若干相同的部件组成：Restful 格式

```
curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
```

| `VERB`         | 适当的 HTTP *方法* 或 *谓词* : `GET`、 `POST`、 `PUT`、 `HEAD` 或者 `DELETE`。 |
| -------------- | ------------------------------------------------------------ |
| `PROTOCOL`     | `http` 或者 `https`（如果你在 Elasticsearch 前面有一个 `https` 代理） |
| `HOST`         | Elasticsearch 集群中任意节点的主机名，或者用 `localhost` 代表本地机器上的节点。 |
| `PORT`         | 运行 Elasticsearch HTTP 服务的端口号，默认是 `9200` 。       |
| `PATH`         | API 的终端路径（例如 `_count` 将返回集群中文档数量）。Path 可能包含多个组件，例如：`_cluster/stats` 和 `_nodes/stats/jvm` 。 |
| `QUERY_STRING` | 任意可选的查询字符串参数 (例如 `?pretty` 将格式化地输出 JSON 返回值，使其更容易阅读) |
| `BODY`         | 一个 JSON 格式的请求体 (如果请求需要的话)                    |

#### insert

```
# PUT /{index}/{tyep}/{id}
# 如 索引megacorp 类型employee
curl -X PUT "localhost:9200/megacorp/employee/1?pretty" -H 'Content-Type: application/json' -d'
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}

```

##### 只Create 非update

```
PUT /website/blog/123?op_type=create
{ ... }

-- or
PUT /website/blog/123/_create
{ ... }
```

#### delete

```
curl -X DELETE "localhost:9200/website/blog/123?pretty"

```

即使文档不存在（ `Found` 是 `false` ）， `_version` 值仍然会增加。这是 Elasticsearch 内部记录本的一部分，用来确保这些改变在跨多节点时以正确的顺序执行。

#### update

文档是不可变的：他们不能被修改，只能被替换。 `update` API 必须遵循同样的规则。 从外部来看，我们在一个文档的某个位置进行部分更新。然而在内部， `update` API 简单使用与之前描述相同的 *检索-修改-重建索引* 的处理过程。

+ 增加新字段到文档中（doc）

  ```
  # 增加字段 tags 和 views , 合并到原 Id=1 中
  curl -X POST "localhost:9200/website/blog/1/_update?pretty" -H 'Content-Type: application/json' -d'
  {
     "doc" : {
        "tags" : [ "testing" ],
        "views": 0
     }
  }
  '
  ```

+ 使用脚本更新文档

  ```
  # Groovy 脚本
  curl -X POST "localhost:9200/website/blog/1/_update?pretty" -H 'Content-Type: application/json' -d'
  {
     "script" : "ctx._source.views+=1"
  }
  '
  
  ---
  curl -X POST "localhost:9200/website/blog/1/_update?pretty" -H 'Content-Type: application/json' -d'
  {
     "script" : "ctx._source.tags+=new_tag",
     "params" : {
        "new_tag" : "search"
     }
  }
  '
  ```

+ 更新冲突重试

  为了避免数据丢失， `update` API 在 *检索* 步骤时检索得到文档当前的 `_version` 号，并传递版本号到 *重建索引* 步骤的 `index` 请求。 如果另一个进程修改了处于检索和重新索引步骤之间的文档，那么 `_version` 号将不匹配，更新请求将会失败。

  这可以通过设置参数 `retry_on_conflict` 来自动完成， 这个参数规定了失败之前 `update` 应该重试的次数，它的默认值为 `0` 。

  ```
  curl -X POST "localhost:9200/website/pageviews/1/_update?retry_on_conflict=5&pretty" -H 'Content-Type: application/json' -d'
  {
     "script" : "ctx._source.views+=1",
     "upsert": {
         "views": 0
     }
  }
  '
  ```

+ 33

+ 44



#### query

##### 轻量搜索

**单主键搜索**

```
# GET /{index}/{type}/{id}

如 GET /megacorp/employee/1

# 指定_source 中的指定字段 或 仅_source
curl -X GET "localhost:9200/website/blog/123?_source=title,text&pretty"
curl -X GET "localhost:9200/website/blog/123/_source?pretty"

```

**全量搜索（_search）**

```
# GET /{index}/{type}/_search

如 GET /megacorp/employee/_search
```

**Query-string 搜索（_search?q={filed}:{value}）**

```
# GET /{index}/{type}/_search?q={filed}:{value}
curl -X GET "localhost:9200/megacorp/employee/_search?q=last_name:Smith"
```



##### 查询表达式搜索

Elasticsearch 提供一个丰富灵活的查询语言叫做 *查询表达式* ， 它支持构建更加复杂和健壮的查询。*领域特定语言* （DSL）， 使用 JSON 构造了一个请求。

**过滤单字段，与Query-string 类似**

```
curl -X GET "localhost:9200/megacorp/employee/_search?pretty" -H 'Content-Type: application/json' -d'
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}
'
```



**过滤器 filter**

```
# 查询 last_name=smith、age > 30 的数据
curl -X GET "localhost:9200/megacorp/employee/_search?pretty" -H 'Content-Type: application/json' -d'
{
    "query" : {
        "bool": {
            "must": {
                "match" : {
                    "last_name" : "smith" 
                }
            },
            "filter": {
                "range" : {
                    "age" : { "gt" : 30 } 
                }
            }
        }
    }
}
'
```

这部分是一个 `range` *过滤器* ， 它能找到年龄大于 30 的文档，其中 `gt` 表示_大于_(*great than*)。

- lt：less than 小于
- le：less than or equal to 小于等于
- eq：equal to 等于
- ne：not equal to 不等于
- ge：greater than or equal to 大于等于
- gt：greater than 大于



##### 全文搜索

查询匹配程度，按得分排序

```
# 搜索下所有喜欢攀岩（rock climbing）的员工：
curl -X GET "localhost:9200/megacorp/employee/_search?pretty" -H 'Content-Type: application/json' -d'
{
    "query" : {
        "match" : {
            "about" : "rock climbing"
        }
    }
}
'

# 响应
{
   ...
   "hits": {
      "total":      2,
      "max_score":  0.16273327,
      "hits": [
         {
            ...
            "_score":         0.16273327,  # 得分
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         },
         {
            ...
            "_score":         0.016878016,  # 得分
            "_source": {
               "first_name":  "Jane",
               "last_name":   "Smith",
               "age":         32,
               "about":       "I like to collect rock albums",
               "interests": [ "music" ]
            }
         }
      ]
   }
}
```

Elasticsearch 默认按照相关性得分排序，即每个文档跟查询的匹配程度。第一个最高得分的结果很明显：John Smith 的 `about` 属性清楚地写着 “rock climbing” 。

但为什么 Jane Smith 也作为结果返回了呢？原因是她的 `about` 属性里提到了 “rock” 。因为只有 “rock” 而没有 “climbing” ，所以她的相关性得分低于 John 的。



##### 短语搜索

精确匹配短语搜索(match_phrase)

```
curl -X GET "localhost:9200/megacorp/employee/_search?pretty" -H 'Content-Type: application/json' -d'
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    }
}
'

-- 响应
{
   ...
   "hits": {
      "total":      1,
      "max_score":  0.23013961,
      "hits": [
         {
            ...
            "_score":         0.23013961,
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         }
      ]
   }
}
```

##### 高亮搜索

```
curl -X GET "localhost:9200/megacorp/employee/_search?pretty" -H 'Content-Type: application/json' -d'
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    },
    "highlight": {
        "fields" : {
            "about" : {}
        }
    }
}
'

-- 响应
{
   ...
   "hits": {
      "total":      1,
      "max_score":  0.23013961,
      "hits": [
         {
            ...
            "_score":         0.23013961,
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            },
            "highlight": {
               "about": [
                  "I love to go <em>rock</em> <em>climbing</em>" 
               ]
            }
         }
      ]
   }
}
```



##### 分析

挖掘出员工中最受欢迎的兴趣爱好

```
curl -X GET "localhost:9200/megacorp/employee/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "aggs": {
    "all_interests": {
      "terms": { "field": "interests" }
    }
  }
}
'

-- 响应
{
   ...
   "hits": { ... },
   "aggregations": {
      "all_interests": {
         "buckets": [
            {
               "key":       "music",
               "doc_count": 2
            },
            {
               "key":       "forestry",
               "doc_count": 1
            },
            {
               "key":       "sports",
               "doc_count": 1
            }
         ]
      }
   }
}
```

如果想知道叫 Smith 的员工中最受欢迎的兴趣爱好，可以直接构造一个组合查询：

```
curl -X GET "localhost:9200/megacorp/employee/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match": {
      "last_name": "smith"
    }
  },
  "aggs": {
    "all_interests": {
      "terms": {
        "field": "interests"
      }
    }
  }
}
'

-- 响应
...
  "all_interests": {
     "buckets": [
        {
           "key": "music",
           "doc_count": 2
        },
        {
           "key": "sports",
           "doc_count": 1
        }
     ]
  }
```

聚合还支持分级汇总 。比如，查询特定兴趣爱好员工的平均年龄：

```
curl -X GET "localhost:9200/megacorp/employee/_search?pretty" -H 'Content-Type: application/json' -d'
{
    "aggs" : {
        "all_interests" : {
            "terms" : { "field" : "interests" },
            "aggs" : {
                "avg_age" : {
                    "avg" : { "field" : "age" }
                }
            }
        }
    }
}
'

-- 响应
  ...
  "all_interests": {
     "buckets": [
        {
           "key": "music",
           "doc_count": 2,
           "avg_age": {
              "value": 28.5
           }
        },
        {
           "key": "forestry",
           "doc_count": 1,
           "avg_age": {
              "value": 35
           }
        },
        {
           "key": "sports",
           "doc_count": 1,
           "avg_age": {
              "value": 25
           }
        }
     ]
  }
```

##### 多索引搜索

+ 搜索多个索引中文档

`mget` API 要求有一个 `docs` 数组作为参数，每个元素包含需要检索文档的元数据， 包括 `_index` 、 `_type` 和 `_id` 。如果你想检索一个或者多个特定的字段，那么你可以通过 `_source` 参数来指定这些字段的名字：

```
curl -X GET "localhost:9200/_mget?pretty" -H 'Content-Type: application/json' -d'
{
   "docs" : [
      {
         "_index" : "website",
         "_type" :  "blog",
         "_id" :    2
      },
      {
         "_index" : "website",
         "_type" :  "pageviews",
         "_id" :    1,
         "_source": "views"
      }
   ]
}
'
--- 响应
{
   "docs" : [
      {
         "_index" :   "website",
         "_id" :      "2",
         "_type" :    "blog",
         "found" :    true,
         "_source" : {
            "text" :  "This is a piece of cake...",
            "title" : "My first external blog entry"
         },
         "_version" : 10
      },
      {
         "_index" :   "website",
         "_id" :      "1",
         "_type" :    "pageviews",
         "found" :    true,
         "_version" : 2,
         "_source" : {
            "views" : 2
         }
      }
   ]
}
```

+ 同index、同type 

文档的 `_index` 和 `_type` 都是相同的，你可以只传一个 `ids` 数组，而不是整个 `docs` 数组：

```
GET /website/blog/_mget
{
   "ids" : [ "2", "1" ]
}

---
{
  "docs" : [
    {
      "_index" :   "website",
      "_type" :    "blog",
      "_id" :      "2",
      "_version" : 10,
      "found" :    true,
      "_source" : {
        "title":   "My first external blog entry",
        "text":    "This is a piece of cake..."
      }
    },
    {
      "_index" :   "website",
      "_type" :    "blog",
      "_id" :      "1",
      "found" :    false  # id 1 的数据不存在
    }
  ]
}
```

+ 根据url 多索引搜索

```
#在所有的索引中搜索所有的类型
/_search

#在 gb 索引中搜索所有的类型
/gb/_search

#在 gb 和 us 索引中搜索所有的文档
/gb,us/_search

#在任何以 g 或者 u 开头的索引中搜索所有的类型
/g*,u*/_search

#在 gb 索引中搜索 user 类型
/gb/user/_search

#在 gb 和 us 索引中搜索 user 和 tweet 类型
/gb,us/user,tweet/_search

# 在所有的索引中搜索 user 和 tweet 类型
/_all/user,tweet/_search

```

当在单一的索引下进行搜索的时候，Elasticsearch 转发请求到索引的每个分片中，可以是主分片也可以是副本分片，然后从每个分片中收集结果。多索引搜索恰好也是用相同的方式工作的—只是会涉及到更多的分片。

+ 1

##### 分页

和 SQL 使用 `LIMIT` 关键字返回单个 `page` 结果的方法相同，Elasticsearch 接受 `from` 和 `size` 参数：

- **`size`**

  显示应该返回的结果数量，默认是 `10`

- **`from`**

  显示应该跳过的初始结果数量，默认是 `0`

```
GET /_search?size=5
GET /_search?size=5&from=5
GET /_search?size=5&from=10
```

假设在一个有 5 个主分片的索引中搜索。 当我们请求结果的第一页（结果从 1 到 10 ），每一个分片产生前 10 的结果，并且返回给 *协调节点* ，协调节点对 50 个结果排序得到全部结果的前 10 个。

在分布式系统中，**对结果排序的成本随分页的深度成指数上升。**这就是 web 搜索引擎对任何查询都不要返回超过 1000 个结果的原因。

#### 排序

+ 默认排序

在 Elasticsearch 中， *相关性得分* 由一个浮点数进行表示，并在搜索结果中通过 `_score` 参数返回， 默认排序是 `_score` 降序。

+ 字段值排序 `sort`

  通过时间来对 tweets 进行排序是有意义的，最新的 tweets 排在最前。 我们可以使用 `sort` 参数进行实现：

  ```
  curl -X GET "localhost:9200/_search?pretty" -H 'Content-Type: application/json' -d'
  {
      "query" : {
          "bool" : {
              "filter" : { "term" : { "user_id" : 1 }}
          }
      },
      "sort": { "date": { "order": "desc" }}
  }
  '
  
  ```

  

+ 多级排序

  结合使用 `date` 和 `_score` 进行查询，并且匹配的结果首先按照日期排序，然后按照相关性排序：

  ```
  curl -X GET "localhost:9200/_search?pretty" -H 'Content-Type: application/json' -d'
  {
      "query" : {
          "bool" : {
              "must":   { "match": { "tweet": "manage text search" }},
              "filter" : { "term" : { "user_id" : 2 }}
          }
      },
      "sort": [
          { "date":   { "order": "desc" }},
          { "_score": { "order": "desc" }}
      ]
  }
  '
  
  ```

  注：排序条件的顺序是很重要的。结果首先按第一个条件排序，仅当结果集的第一个 `sort` 值完全相同时才会按照第二个条件进行排序，以此类推。

+ 多值字段排序

  一种情形是字段有多个值的排序、可以通过使用 `min` 、 `max` 、 `avg` 或是 `sum` *排序模式* 。

  ```
  "sort": {
      "dates": {
          "order": "asc",
          "mode":  "min"
      }
  }
  ```

+ 1





#### 过滤与查询

+ 过滤 filter  boolean 是否

  *必须* 匹配，但它以不评分、过滤模式来进行。这些语句对评分没有贡献，只是根据过滤标准来排除或包含文档。

+ 查询 一个“评分”的查询

```
POST / book / _search {
	"query": {
		"bool": {
			"must": {
				"match_all": {}
			},
			"filter": {
				"range": {
					"price": {
						"gte": 200,
						"lte": 1000
					}
				}
			}
		}
	}
}
```



#### 批量操作

 `bulk` API 允许在单个步骤中进行多次 `create` 、 `index` 、 `update` 或 `delete` 请求。 如果你需要索引一个数据流比如日志事件，它可以排队和索引数百或数千批次。

请求格式如下

```
{ action: { metadata }}\n
{ request body        }\n
{ action: { metadata }}\n
{ request body        }\n
...
```

这种格式类似一个有效的单行 JSON 文档 *流* ，它通过换行符(`\n`)连接到一起。注意两个要点：

- 每行一定要以换行符(`\n`)结尾， *包括最后一行* 。这些换行符被用作一个标记，可以有效分隔行。
- 这些行不能包含未转义的换行符，因为他们将会对解析造成干扰。这意味着这个 JSON *不* 能使用 pretty 参数打印。

`action/metadata` 行指定 *哪一个文档* 做 *什么操作* 。

action 必须为以下选项中

+ create

  如果文档不存在，那么就创建它。

+ index

  创建一个新文档或者替换一个现有的文档。

+ update

  部分更新一个文档。

+ delete

  删除一个文档。

+ 1

`metadata` 应该指定被索引、创建、更新或者删除的文档的 `_index` 、 `_type` 和 `_id` 。

`request body` 行由文档的 `_source` 本身组成—文档包含的字段和值。它是 `index` 和 `create` 操作所必需的，这是有道理的：你必须提供文档以索引。

```
curl -X POST "localhost:9200/_bulk?pretty" -H 'Content-Type: application/json' -d'
{ "delete": { "_index": "website", "_type": "blog", "_id": "123" }} 
{ "create": { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "My first blog post" }
{ "index":  { "_index": "website", "_type": "blog" }}
{ "title":    "My second blog post" }
{ "update": { "_index": "website", "_type": "blog", "_id": "123", "_retry_on_conflict" : 3} }
{ "doc" : {"title" : "My updated blog post"} }
'

```

**注：谨记最后一个换行符不要落下。**

每个子请求都是独立执行，因此某个子请求的失败不会对其他子请求的成功与否造成影响。 如果其中任何子请求失败，最顶层的 `error` 标志被设置为 `true` ，并且在相应的请求报告出错误明细。**非事务性**



### 并发

#### 乐观并发控制

Elasticsearch 是分布式的。当文档创建、更新或删除时， 新版本的文档必须复制到集群中的其他节点。Elasticsearch 也是异步和并发的，这意味着这些复制请求被并行发送，并且到达目的地时也许 *顺序是乱的* 。 Elasticsearch 需要一种方法确保文档的旧版本不会覆盖新的版本。

每个文档都有一个 `_version` （版本）号，当文档被修改时版本号递增。 Elasticsearch 使用这个 `_version` 号来确保变更以正确顺序得到执行。如果旧版本的文档在新版本之后到达，它可以被简单的忽略。



指定 `version` 为我们的修改会被应用的版本，索引中的文档只有现在的 `_version` 为 `1` 时，本次更新才能成功。通过乐观锁的方式控制并发。

```
curl -X PUT "localhost:9200/website/blog/1?version=1&pretty" -H 'Content-Type: application/json' -d'
{
  "title": "My first blog entry",
  "text":  "Starting to get the hang of this..."
}
'
```



## DSL语句

Elasticsearch提供了基于JSON的完整查询DSL（Domain Specific Language 特定域的语言）来定义查
询。将查询DSL视为查询的AST（抽象语法树），它由两种子句组成：

+ 叶子查询子句
  叶子查询子句 在特定域中寻找特定的值，如 match，term或 range查询。
+ 复合查询子句
  复合查询子句包装其他叶子查询或复合查询，并用于以逻辑方式组合多个查询（例如 bool或
  dis_max查询），或更改其行为（例如 constant_score查询）。 

### 基本语法

```
POST /索引库名/_search
{
  "query":{
     "查询类型":{
       "查询条件":"查询条件值"
     }
  }
}
```

返回结果示例

+ took：查询花费时间，单位是毫秒
+ time_out：是否超时
+ shards：分片信息
+ hits：搜索结果总览对象
+ total：搜索到的总条数
+ max_score：所有结果中文档得分的最高分
+ hits：搜索结果的文档对象数组，每个元素是一条搜索到的文档信息
  _index：索引库
  _type：文档类型
  _id：文档id
  _score：文档得分
  _source：文档的源数据 



### 全文搜索

全文搜索能够搜索已分析的文本字段，如电子邮件正文，商品描述等 。

#### 匹配搜索  match 

全文查询的标准查询，它可以对一个字段进行模糊、短语查询。 match queries 接收 text/numerics/dates, 对它们进行分词分析, 再组织成一个boolean查询。

可通过operator 指定bool组合操作（or、and ）， 默认 or

```
POST /lagou-property/_search
{
  "query":{
     "match":{
        "title":"小米电视4A"
     }
  }
}
```

 match 类型查询，会把查询条件进行分词，然后进行查询,多个词条之间是or的关系。上述将  "小米电视4A" 语句分为 “小米”，“电视”，“4A” 进行词条搜索。以 or 展示结果集，即把上述词条的都搜索出来。

```
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 1.2044649,
    "hits" : [
      {
        "_index" : "lagou-property",
        "_type" : "_doc",
        "_id" : "N4BmKn4Bz2uxrezJ8i2B",
        "_score" : 1.2044649,
        "_source" : {
          "title" : "小米电视4A",
          "images" : "http://image.lagou.com/12479122.jpg",
          "price" : 4288
        }
      },
      {
        "_index" : "lagou-property",
        "_type" : "_doc",
        "_id" : "OIBnKn4Bz2uxrezJAi3X",
        "_score" : 0.52354836,
        "_source" : {
          "title" : "小米手机",
          "images" : "http://image.lagou.com/12479622.jpg",
          "price" : 2699
        }
      }
    ]
  }
}
```

**指定为 and 查找**

```
POST /lagou-property/_search
{
  "query":{
     "match":{
        "title": {"query":"小米电视4A", "operator": "and"}
     }
  }
}
```

上述 只有同时包含 小米 和 电视 的词条才会被搜索到。 

```
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.2044649,
    "hits" : [
      {
        "_index" : "lagou-property",
        "_type" : "_doc",
        "_id" : "N4BmKn4Bz2uxrezJ8i2B",
        "_score" : 1.2044649,
        "_source" : {
          "title" : "小米电视4A",
          "images" : "http://image.lagou.com/12479122.jpg",
          "price" : 4288
        }
      }
    ]
  }
}
```



#### 短语搜索 match_phrase 

match_phrase 查询用来对一个字段进行短语查询，可以指定 slop移动因子 

```
GET /lagou-property/_search
{
  "query": {
    "match_phrase": {
      "title": "小米电视4A"
    }
  }
}

```

结果为完全短语 等于

```
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 2.8330116,
    "hits" : [
      {
        "_index" : "lagou-property",
        "_type" : "_doc",
        "_id" : "N4BmKn4Bz2uxrezJ8i2B",
        "_score" : 2.8330116,
        "_source" : {
          "title" : "小米电视4A",
          "images" : "http://image.lagou.com/12479122.jpg",
          "price" : 4288
        }
      }
    ]
  }
}
```



使用移动字段 slop, 即词汇短语中间可移动忽略几个词 进行匹配。

+ 移动 0 次，查询不到

```
GET /lagou-property/_search
{
  "query": {
    "match_phrase": {
      "title": {
        "query": "小米 4A",
        "slop": 0
      }
    }
  }
}

--- 移动0次 ---
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  }
}
```

+ 移动1次 ，可查询到

```
GET /lagou-property/_search
{
  "query": {
    "match_phrase": {
      "title": {
        "query": "小米 4A",
        "slop": 1
      }
    }
  }
}

--- 移动1次 ---
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.2441062,
    "hits" : [
      {
        "_index" : "lagou-property",
        "_type" : "_doc",
        "_id" : "N4BmKn4Bz2uxrezJ8i2B",
        "_score" : 1.2441062,
        "_source" : {
          "title" : "小米电视4A",
          "images" : "http://image.lagou.com/12479122.jpg",
          "price" : 4288
        }
      }
    ]
  }
}
```



#### query_string 查询

query_string Query提供了无需指定某字段而对文档全文进行匹配查询的一个高级查询,同时可以指定在
哪些字段上进行匹配。 

```
GET /lagou-property/_search
{
  "query": {
    "query_string": {
      "query": "2699"
    }
  }
}

--- 未指定 title，直接查询内容 ---

{
  "took" : 33,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "lagou-property",
        "_type" : "_doc",
        "_id" : "OIBnKn4Bz2uxrezJAi3X",
        "_score" : 1.0,
        "_source" : {
          "title" : "小米手机",
          "images" : "http://image.lagou.com/12479622.jpg",
          "price" : 2699
        }
      }
    ]
  }
}


--- 逻辑查询 ---
# 逻辑查询 OR 两词或 
GET /lagou-property/_search
{
  "query": {
    "query_string": {
      "query": "手机 OR 小米",
      "default_field": "title"
    }
  }
}

# 逻辑查询 AND 两词与
GET /lagou-property/_search
{
  "query": {
    "query_string": {
      "query": "手机 AND 小米",
      "default_field": "title"
    }
  }
}

# ~数字：模糊查询可修正几个数字
GET /lagou-property/_search
{
  "query": {
    "query_string": {
      "query": "大米~1",
      "default_field": "title"
    }
  }
}


#多字段支持
GET /lagou-property/_search
{
"query": {
"query_string" : {
"query":"2699",
"fields": [ "title","price"]
}
}
}

```

### 

## Java Client

+ 单独引用

```
        <dependency>
            <groupId>io.searchbox</groupId>
            <artifactId>jest</artifactId>
            <version>6.3.1</version>
        </dependency>
        <dependency>
            <groupId>org.elasticsearch</groupId>
            <artifactId>elasticsearch</artifactId>
        </dependency>
```

```
//初始化工厂类
         JestClientFactory jestClientFactory = new JestClientFactory();
          HttpClientConfig httpClientConfig = null;
          if(StringUtils.isNotBlank(username) && StringUtils.isNotBlank(password)){
              httpClientConfig = new HttpClientConfig.Builder(serverUris).defaultCredentials(username, password).readTimeout(10000).multiThreaded(true).build();
          }else{
              httpClientConfig = new HttpClientConfig.Builder(serverUris).readTimeout(10000).multiThreaded(true).build();
          }
         jestClientFactory.setHttpClientConfig(httpClientConfig);
         //获取客户端
         return  jestClientFactory.getObject();
```

+ SpringBoot 引用 RestClient

```
public RestHighLevelClient client() {
        Assert.requireNonEmpty(this.hosts, "无效的es连接");
        return new RestHighLevelClient(
                RestClient.builder(this.hosts).build()
        );
    }
    
    
```



## 集群

一个运行中的 Elasticsearch 实例称为一个节点，而集群是由一个或者多个拥有相同 `cluster.name` 配置的节点组成， 它们共同承担数据和负载的压力。当有节点加入集群中或者从集群中移除节点时，集群将会重新平均分布所有的数据。

当一个节点被选举成为 *主* 节点时， 它将负责管理集群范围内的所有变更，例如增加、删除索引，或者增加、删除节点等。 而主节点并不需要涉及到文档级别的变更和搜索等操作，所以当集群只拥有一个主节点的情况下，即使流量的增加它也不会成为瓶颈。 任何节点都可以成为主节点。我们的示例集群就只有一个节点，所以它同时也成为了主节点。

### 集群健康

Elasticsearch 的集群监控信息中包含了许多的统计数据，其中最为重要的一项就是 *集群健康* ， 它在 `status` 字段中展示为 `green` 、 `yellow` 或者 `red` 。

```
curl -X GET "localhost:9200/_cluster/health?pretty"

{
    "cluster_name": "elasticsearch",
    "status": "yellow",
    "timed_out": false,
    "number_of_nodes": 1,
    "number_of_data_nodes": 1,
    "active_primary_shards": 2,
    "active_shards": 2,
    "relocating_shards": 0,
    "initializing_shards": 0,
    "unassigned_shards": 1,
    "delayed_unassigned_shards": 0,
    "number_of_pending_tasks": 0,
    "number_of_in_flight_fetch": 0,
    "task_max_waiting_in_queue_millis": 0,
    "active_shards_percent_as_number": 66.66666666666666
}
```

`status` 字段指示着当前集群在总体上是否工作正常。它的三种颜色含义如下：

- **`green`**

  所有的主分片和副本分片都正常运行。

- **`yellow`**

  所有的主分片都正常运行，但不是所有的副本分片都正常运行。

- **`red`**

  有主分片没能正常运行。



### 索引

索引实际上是指向一个或者多个物理 *分片* 的 *逻辑命名空间* 。一个 *分片* 是一个底层的 *工作单元* ，它仅保存了全部数据中的一部分。

索引内任意一个文档都归属于一个主分片，所以主分片的数目决定着索引能够保存的最大数据量。文档保存在分片内，分片又被分配到集群内的各个节点里。一个分片可以是 *主* 分片或者 *副本* 分片，默认情况下会被分配5个主分片。

添加一个 3 个主分片 一个副本分片的索引（每个主分片拥有一个副本分片）

```
curl -X PUT "localhost:9200/blogs?pretty" -H 'Content-Type: application/json' -d'
{
   "settings" : {
      "number_of_shards" : 3,
      "number_of_replicas" : 1
   }
}
'
```

### 倒排索引

Elasticsearch 使用一种称为 *倒排索引* 的结构，它适用于快速的全文搜索。一个倒排索引由文档中所有不重复词的列表构成，对于其中每个词，有一个包含它的文档列表。

#### 分析

- 首先，将一块文本分成适合于倒排索引的独立的 *词条* ，
- 之后，将这些词条统一化为标准格式以提高它们的“可搜索性”，或者 *recall*

#### 分析器

+ 字符过滤器

  字符串按顺序通过每个 *字符过滤器* 。他们的任务是在分词前整理字符串。一个字符过滤器可以用来去掉HTML，或者将 `&` 转化成 `and`。

+ 分词器

  字符串被 *分词器* 分为单个的词条。一个简单的分词器遇到空格和标点的时候，可能会将文本拆分成词条。

+ Token过滤器

  词条按顺序通过每个 *token 过滤器* 。这个过程可能会改变词条（例如，小写化 `Quick` ），删除词条（例如， 像 `a`， `and`， `the` 等无用词），或者增加词条（例如，像 `jump` 和 `leap` 这种同义词）。

+ 1

Elasticsearch提供了开箱即用的字符过滤器、分词器和token 过滤器。 这些可以组合起来形成自定义的分析器以用于不同的目的。

**Es内置分析器**

```
Set the shape to semi-transparent by calling set_trans(5)
```

+ 标准分析器

  标准分析器是Elasticsearch默认使用的分析器。它是分析各种语言文本最常用的选择。它根据 [Unicode 联盟](http://www.unicode.org/reports/tr29/) 定义的 *单词边界* 划分文本。删除绝大部分标点。最后，将词条小写。

  ```
  set, the, shape, to, semi, transparent, by, calling, set_trans, 5
  ```

+ 简单分析器

  简单分析器在任何不是字母的地方分隔文本，将词条小写。

  ```
  set, the, shape, to, semi, transparent, by, calling, set, trans
  ```

+ 空格分析器

  空格分析器在空格的地方划分文本。

  ```
  Set, the, shape, to, semi-transparent, by, calling, set_trans(5)
  ```

+ 语言分析器

  特定语言分析器可用于 [很多语言](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-lang-analyzer.html)。它们可以考虑指定语言的特点。例如， `英语` 分析器附带了一组英语无用词（常用单词，例如 `and` 或者 `the` ，它们对相关性没有多少影响），它们会被删除。 由于理解英语语法的规则，这个分词器可以提取英语单词的 *词干* 。

  ```
  set, shape, semi, transpar, call, set_tran, 5
  ```

  注意看 `transparent`、 `calling` 和 `set_trans` 已经变为词根格式。

+ 1

分析器使用

- 当你查询一个 *全文* 域时， 会对查询字符串应用相同的分析器，以产生正确的搜索词条列表。

  ```
  # 在 _all 域查询 2014-09-15，它首先分析查询字符串，产生匹配 2014， 09， 或 15 中 任意 词条的查询。
  GET /_search?q=2014-09-15
  ```

- 当你查询一个 *精确值* 域时，不会分析查询字符串，而是搜索你指定的精确值。

  ```
  # date 域查询 2014-09-15，它寻找 精确 日期
  GET /_search?q=date:2014-09-15
  ```

- 1



### **索引操作**

+ 查看状态

  ```
  GET /_cat/indices?v
  ```

  绿色：索引的所有分片都正常分配。
  黄色：至少有一个副本没有得到正确的分配。
  红色：至少有一个主分片没有得到正确的分配。 

+ 打开索引

  ```
  POST /索引名称/_open
  ```

+ 关闭索引

  ```
  POST /索引名称/_close
  ```

+ 删除索引

  ```
  DELETE /索引名称1,索引名称2,索引名称3...
  ```

+ 1

### 映射 Mapping

索引创建之后，等于有了关系型数据库中的database。Elasticsearch7.x取消了索引type类型的设置，
不允许指定类型，默认为_doc，但字段仍然是有的，我们需要设置字段的约束信息，叫做字段映射
（mapping）
字段的约束包括但不限于：

+ type：类型，可以是text、long、short、date、integer、object等
+ index：是否索引，默认为true
+ store：是否存储，默认为false
+ analyzer：指定分词器 

```
PUT /索引库名/_mapping
{
"properties": {
"字段名": {
"type": "类型",
"index": true， 
"store": true，
"analyzer": "分词器"
}
}
}
```

索引类型

**1、String类型，又分两种：**

+ text：可分词，不可参与聚合
+ keyword：不可分词，数据会作为完整字段进行匹配，可以参与聚合
+ Numerical：数值类型，分两类
  基本数据类型：long、interger、short、byte、double、float、half_float
  浮点数的高精度类型：scaled_float
  需要指定一个精度因子，比如10或100。elasticsearch会把真实值乘以这个因子后存
  储，取出时再
  原。
+ Date：日期类型
  elasticsearch可以对日期格式化为字符串存储，但是建议我们存储为毫秒值，存储为long，节省
  空间。
+ Array：数组类型
  进行匹配时，任意一个元素满足，都认为满足
  排序时，如果升序则用数组中的最小值来排序，如果降序则用数组中的最大值来排序
+ Object：对象
  如果存储到索引库的是对象类型，例如上面的girl，会把girl变成两个字段：girl.name和girl.age

**2、index ，影响字段的索引情况：**

+ true：字段会被索引，则可以用来进行搜索。默认值就是true
+ false：字段不会被索引，不能用来搜索
  index的默认值就是true，也就是说你不进行任何配置，所有字段都会被索引。
  但是有些字段是我们不希望被索引的，比如企业的logo图片地址，就需要手动设置index为false。、

**3、store 是否将数据进行独立存储。**
原始的文本会存储在 _source 里面，默认情况下其他提取出来的字段都不是独立存储的，是从
_source 里面提取出来的。当然你也可以独立的存储某个字段，只要设置store:true即可，获取独立存
储的字段要比从_source中解析快得多，但是也会占用更多的空间，所以要根据实际业务需求来设置，
默认为false。

**4、analyzer：指定分词器**
一般我们处理中文会选择ik分词器 ik_max_word ik_smart 



## 数据存储

### 文档元数据

一个文档不仅仅包含它的数据 ，也包含 *元数据* —— *有关* 文档的信息。 三个必须的元数据元素如下：

- **`_index`**

  文档在哪存放。一个 *索引* 应该是因共同的特性被分组到一起的文档集合。

  实际上，在 Elasticsearch 中，我们的数据是被存储和索引在 *分片* 中，而一个索引仅仅是逻辑上的命名空间， 这个命名空间由一个或者多个分片组合在一起。 然而，这是一个内部细节，我们的应用程序根本不应该关心分片，对于应用程序而言，只需知道文档位于一个 *索引* 内。 Elasticsearch 会处理所有的细节。

  **注：索引名，这个名字必须小写，不能以下划线开头，不能包含逗号**

  

- **`_type`**

  文档表示的对象类别。数据可能在索引中只是松散的组合在一起，但是通常明确定义一些数据中的子分区是很有用的。

  Elasticsearch 公开了一个称为 *types* （类型）的特性，它允许您在索引中对数据进行逻辑分区。不同 types 的文档可能有不同的字段，但最好能够非常相似。

  **注：一个 `_type` 命名可以是大写或者小写，但是不能以下划线或者句号开头，不应该包含逗号， 并且长度限制为256个字符.**

- **`_id`**

  文档唯一标识。*ID* 是一个字符串，当它和 `_index` 以及 `_type` 组合就可以唯一确定 Elasticsearch 中的一个文档。 当你创建一个新的文档，要么提供自己的 `_id` ，要么让 Elasticsearch 帮你生成。

### ID主键

#### 自定义ID

```
# PUT 方式自定义ID
PUT /{index}/{type}/{id}
{
  "field": "value",
  ...
}

---
{
   "_index":    "website",
   "_type":     "blog",
   "_id":       "123",
   "_version":  1,
   "created":   true
}
```

该响应表明文档已经成功创建，该索引包括 `_index` 、 `_type` 和 `_id` 元数据， 以及一个新元素： `_version` 。

版本号：当每次对文档进行修改时（包括删除）， `_version` 的值会递增。

#### Autogenerating ID

如果你的数据没有自然的 ID， Elasticsearch 可以帮我们自动生成 ID 。 请求的结构调整为： 不再使用 `PUT` 谓词(“使用这个 URL 存储这个文档”)， 而是使用 `POST` 谓词(“存储文档在这个 URL 命名空间下”)。

现在该 URL 只需包含 `_index` 和 `_type` :

```
curl -X POST "localhost:9200/website/blog/?pretty" -H 'Content-Type: application/json' -d'
{
  "title": "My second blog entry",
  "text":  "Still trying this out...",
  "date":  "2014/01/01"
}
'
```

### 分布式存储

#### 路由一个文档到分片中

```
shard = hash(routing) % number_of_primary_shards
```

`routing` 是一个可变值，默认是文档的 `_id` ，也可以设置成一个自定义的值。 `routing` 通过 hash 函数生成一个数字，然后这个数字再除以 `number_of_primary_shards` （主分片的数量 **确定好不能改变**）后得到 **余数** 。这个分布在 `0` 到 `number_of_primary_shards-1` 之间的余数，就是我们所寻求的文档所在分片的位置。

所有的文档 API（ `get` 、 `index` 、 `delete` 、 `bulk` 、 `update` 以及 `mget` ）都接受一个叫做 `routing` 的路由参数 ，通过这个参数我们可以自定义文档到分片的映射。一个自定义的路由参数可以用来确保所有相关的文档——例如所有属于同一个用户的文档——都被存储到同一个分片中。



新建、索引和删除 请求都是  `写`  操作， 必须在主分片上面完成之后才能被复制到相关的副本分片

![](Elasticsearch.assets/elas_0402.png)

以下是在主副分片和任何副本分片上面 成功新建，索引和删除文档所需要的步骤顺序：

1. 客户端向 `Node 1` 发送新建、索引或者删除请求。
2. 节点使用文档的 `_id` 确定文档属于分片 0 。请求会被转发到 `Node 3`，因为分片 0 的主分片目前被分配在 `Node 3` 上。
3. `Node 3` 在主分片上面执行请求。如果成功了，它将请求并行转发到 `Node 1` 和 `Node 2` 的副本分片上。一旦所有的副本分片都报告成功, `Node 3` 将向协调节点报告成功，协调节点向客户端报告成功。

在客户端收到成功响应时，文档变更已经在主分片和所有副本分片执行完成，变更是安全的。

#### 取回一个文档

![](Elasticsearch.assets/elas_0403.png)

以下是从主分片或者副本分片检索文档的步骤顺序：

1、客户端向 `Node 1` 发送获取请求。

2、节点使用文档的 `_id` 来确定文档属于分片 `0` 。分片 `0` 的副本分片存在于所有的三个节点上。 在这种情况下，它将请求转发到 `Node 2` 。

3、`Node 2` 将文档返回给 `Node 1` ，然后将文档返回给客户端。

在处理读取请求时，协调结点在每次请求的时候都会通过轮询所有的副本分片来达到负载均衡。

在文档被检索时，**已经被索引的文档可能已经存在于主分片上但是还没有复制到副本分片。 在这种情况下，副本分片可能会报告文档不存在**，但是主分片可能成功返回文档。 一旦索引请求成功返回给用户，文档在主分片和副本分片都是可用的。

#### 局部更新文档

![](Elasticsearch.assets/elas_0404.png)

1. 客户端向 `Node 1` 发送更新请求。
2. 它将请求转发到主分片所在的 `Node 3` 。
3. `Node 3` 从主分片检索文档，修改 `_source` 字段中的 JSON ，并且尝试重新索引主分片的文档。 如果文档已经被另一个进程修改，它会**重试**步骤 3 ，超过 `retry_on_conflict` 次后放弃。
4. 如果 `Node 3` 成功地更新文档，它将新版本的文档并行转发到 `Node 1` 和 `Node 2` 上的副本分片，重新建立索引。 一旦所有副本分片都返回成功， `Node 3` 向协调节点也返回成功，协调节点向客户端返回成功。

`update` API 还接受在 [新建、索引和删除文档](https://www.elastic.co/guide/cn/elasticsearch/guide/current/distrib-write.html) 章节中介绍的 `routing` 、 `replication` 、 `consistency` 和 `timeout` 参数。

**当主分片把更改转发到副本分片时， 它不会转发更新请求。 相反，它转发完整文档的新版本。**

#### 批量文档 bulk

![](Elasticsearch.assets/elas_0406.png)

`bulk` API 按如下步骤顺序执行：

1. 客户端向 `Node 1` 发送 `bulk` 请求。
2. `Node 1` 为每个节点创建一个批量请求，**并将这些请求并行转发到每个包含主分片的节点主机。**
3. **主分片一个接一个按顺序执行每个操作**。当每个操作成功时，主分片并行转发新文档（或删除）到副本分片，然后执行下一个操作。 一旦所有的副本分片报告所有操作成功，该节点将向协调节点报告成功，协调节点将这些响应收集整理并返回给客户端。

`bulk` API 还可以在整个批量请求的最顶层使用 `consistency` 参数，以及在每个请求中的元数据中使用 `routing` 参数。







## 系统配置

+ **consistency**

  consistency，即一致性。在默认设置下，即使仅仅是在试图执行一个_写_操作之前，主分片都会要求 必须要有 *规定数量(quorum)*（或者换种说法，也即必须要有大多数）的分片副本处于活跃可用状态，才会去执行_写_操作(其中分片副本可以是主分片或者副本分片)。这是为了避免在发生网络分区故障（network partition）的时候进行_写_操作，进而导致数据不一致。_规定数量_即：

  ```
  int( (primary + number_of_replicas) / 2 ) + 1
  ```

  `consistency` 参数的值可以设为 `one` （只要主分片状态 ok 就允许执行_写_操作）,`all`（必须要主分片和所有副本分片的状态没问题才允许执行_写_操作）, 或 `quorum` 。默认值为 `quorum` , 即大多数的分片副本状态没问题就允许执行_写_操作。

  处于活跃状态的分片副本数量达不到规定数量，将无法索引和删除任何文档。

  **注：只有当 `number_of_replicas` 大于1的时候，规定数量才会执行。单节点的情况**

+ **timeout**

  如果没有足够的副本分片会发生什么？ Elasticsearch会等待，希望更多的分片出现。默认情况下，它最多等待1分钟。 如果你需要，你可以使用 `timeout` 参数 使它更早终止： `100` 100毫秒，`30s` 是30秒。

+ 3

+ 4