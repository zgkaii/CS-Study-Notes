## SpringBoot整合ElasticSearch
### 1、导入依赖
新建mall-search检索服务模块。  
这里的版本要和所按照的ELK版本匹配。
```xml
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
    <version>7.6.2</version>
</dependency>
```
在spring-boot-dependencies中所依赖的ELK版本位6.8.7
```
<elasticsearch.version>6.8.7</elasticsearch.version>
```
![](https://img-blog.csdnimg.cn/20200909232520794.png)  需要在项目中将它改为7.6.2
```xml
<properties>
        ...
        <elasticsearch.version>7.6.2</elasticsearch.version>
    </properties>
```
### 2、编写测试类
#### 1）测试保存数据
[https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-document-index.html](https:_www.elastic.co_guide_en_elasticsearch_client_java-rest_current_java-rest-high-document-index)
```java
@Test
    public void indexData() throws IOException {
        IndexRequest indexRequest = new IndexRequest ("users");
        User user = new User();
        user.setUserName("张三");
        user.setAge(20);
        user.setGender("男");
        String jsonString = JSON.toJSONString(user);
        //设置要保存的内容
        indexRequest.source(jsonString, XContentType.JSON);
        //执行创建索引和保存数据
        IndexResponse index = client.index(indexRequest, GulimallElasticSearchConfig.COMMON_OPTIONS);
        System.out.println(index);
    }
```
测试前：
![](https://cdn.nlark.com/yuque/0/2020/png/512093/1591341332477-ab36083f-84d2-40d2-bef6-ee0f9fe58a9b.png#align=left&display=inline&height=473&margin=%5Bobject%20Object%5D&originHeight=473&originWidth=1405&status=done&style=none&width=1405)
测试后：
![](https://cdn.nlark.com/yuque/0/2020/png/512093/1591341332575-495d410f-0883-424c-a36e-4dfd4234ec6a.png#align=left&display=inline&height=546&margin=%5Bobject%20Object%5D&originHeight=546&originWidth=1397&status=done&style=none&width=1397)
#### 2）测试获取数据
[https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-search.html](https:_www.elastic.co_guide_en_elasticsearch_client_java-rest_current_java-rest-high-search)
```
@Test
    public void searchData() throws IOException {
        GetRequest getRequest = new GetRequest(
                "users",
                "_-2vAHIB0nzmLJLkxKWk");
        GetResponse getResponse = client.get(getRequest, RequestOptions.DEFAULT);
        System.out.println(getResponse);
        String index = getResponse.getIndex();
        System.out.println(index);
        String id = getResponse.getId();
        System.out.println(id);
        if (getResponse.isExists()) {
            long version = getResponse.getVersion();
            System.out.println(version);
            String sourceAsString = getResponse.getSourceAsString();
            System.out.println(sourceAsString);
            Map<String, Object> sourceAsMap = getResponse.getSourceAsMap();
            System.out.println(sourceAsMap);
            byte[] sourceAsBytes = getResponse.getSourceAsBytes();
        } else {
        }
    }
```
查询state="AK"的文档：
```json
{
	"took": 1,
	"timed_out": false,
	"_shards": {
		"total": 1,
		"successful": 1,
		"skipped": 0,
		"failed": 0
	},
	"hits": {
		"total": {
			"value": 22,   //匹配到了22条
			"relation": "eq"
		},
		"max_score": 3.7952394,
		"hits": [{
			"_index": "bank",
			"_type": "account",
			"_id": "210",
			"_score": 3.7952394,
			"_source": {
				"account_number": 210,
				"balance": 33946,
				"firstname": "Cherry",
				"lastname": "Carey",
				"age": 24,
				"gender": "M",
				"address": "539 Tiffany Place",
				"employer": "Martgo",
				"email": "cherrycarey@martgo.com",
				"city": "Fairacres",
				"state": "AK"
			}
		},
           ....//省略其他
          ]
	}
}
```
**搜索address中包含mill的所有人的年龄分布以及平均年龄，平均薪资**
```json
GET bank/_search
{
  "query": {
    "match": {
      "address": "mill"
    }
  },
  "aggs": {
    "ageAgg": {
      "terms": {
        "field": "age",
        "size": 10
      }
    },
    "ageAvg": {
      "avg": {
        "field": "age"
      }
    },
    "balanceAvg": {
      "avg": {
        "field": "balance"
      }
    }
  }
}
```
java实现
```java
/**
     * 复杂检索:在bank中搜索address中包含mill的所有人的年龄分布以及平均年龄，平均薪资
     * @throws IOException
     */
    @Test
    public void searchData() throws IOException {
        //1. 创建检索请求
        SearchRequest searchRequest = new SearchRequest();
        //1.1）指定索引
        searchRequest.indices("bank");
        //1.2）构造检索条件
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        sourceBuilder.query(QueryBuilders.matchQuery("address","Mill"));
        //1.2.1)按照年龄分布进行聚合
        TermsAggregationBuilder ageAgg=AggregationBuilders.terms("ageAgg").field("age").size(10);
        sourceBuilder.aggregation(ageAgg);
        //1.2.2)计算平均年龄
        AvgAggregationBuilder ageAvg = AggregationBuilders.avg("ageAvg").field("age");
        sourceBuilder.aggregation(ageAvg);
        //1.2.3)计算平均薪资
        AvgAggregationBuilder balanceAvg = AggregationBuilders.avg("balanceAvg").field("balance");
        sourceBuilder.aggregation(balanceAvg);
        System.out.println("检索条件："+sourceBuilder);
        searchRequest.source(sourceBuilder);
        //2. 执行检索
        SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);
        System.out.println("检索结果："+searchResponse);
        //3. 将检索结果封装为Bean
        SearchHits hits = searchResponse.getHits();
        SearchHit[] searchHits = hits.getHits();
        for (SearchHit searchHit : searchHits) {
            String sourceAsString = searchHit.getSourceAsString();
            Account account = JSON.parseObject(sourceAsString, Account.class);
            System.out.println(account);
        }
        //4. 获取聚合信息
        Aggregations aggregations = searchResponse.getAggregations();
        Terms ageAgg1 = aggregations.get("ageAgg");
        for (Terms.Bucket bucket : ageAgg1.getBuckets()) {
            String keyAsString = bucket.getKeyAsString();
            System.out.println("年龄："+keyAsString+" ==> "+bucket.getDocCount());
        }
        Avg ageAvg1 = aggregations.get("ageAvg");
        System.out.println("平均年龄："+ageAvg1.getValue());
        Avg balanceAvg1 = aggregations.get("balanceAvg");
        System.out.println("平均薪资："+balanceAvg1.getValue());
    }
```
可以尝试对比打印的条件和执行结果，和前面的ElasticSearch的检索语句和检索结果进行比较；
```shell script
年龄：38 ==> 2
年龄：28 ==> 1
年龄：32 ==> 1
平均年龄：34.0
平均薪资：25208.0
```

## 其他
### 1. kibana控制台命令
ctrl+home：回到文档首部；  
ctril+end：回到文档尾部。
