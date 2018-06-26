### 1 elastic介绍与solr的对比
   
    Elasticsearch是一个实时的分布式搜索和分析引擎，它可以用于全文搜索，结构化搜索以及分析.
    
1. 分布式实时文件存储，并将每一个字段都编入索引，使其可以被搜索
2. 实时分析的分布式搜索引擎。
3. 可以扩展到上百台服务器，处理PB级别的结构化或非结构化数据。


es优点：

1. es本身就是分布式的，安装起来方便，不需要zookeper进行管理
2. es完全支持Apache Lucene 的接近实时的搜索.
3. 处理多租户（multitenancy）不需要特殊配置，而Solr则需要更多的高级设置。
4. Elasticsearch 采用 Gateway 的概念，使得完备份更加简单。
5. 各节点组成对等的网络结构，某些节点出现故障时会自动分配其他节点代替其进行工作。

Solr（读作“solar”）是Apache Lucene项目的开源企业搜索平台。其主要功能包括全文检索、命中标示、分面搜索、动态聚类、数据库集成，以及富文本（如Word、PDF）的处理。Solr是高度可扩展的，并提供了分布式搜索和索引复制。
Solr是用Java编写、运行在Servlet容器（如 Apache Tomcat 或Jetty）的一个独立的全文搜索服务器。
solr 5.+ 可以使用内置的jetty启动。

solr优点：

1. solr的用户，开发者更多，更加的成熟。
2. 支持添加多种格式的索引，如：HTML、PDF、微软 Office 系列软件格式以及 JSON、XML、CSV 等纯文本格式。
3. 不考虑建索引的同时进行搜索，速度更快。


优缺点对比： 

==在数据频繁更新的时候，需要实时建立索引，solr的搜索效率急剧下降。因此新兴的电商产业，对于数据更新比较频繁，都采用es做搜索引擎.==

1. solr需要zookeper进行管理，而es自带分布式管理
2. solr支持多种格式的文档，如json，xml，cvs，而es只支持json格式的文档
3. solr官方提供很多强大的功能，而es只注重本身的核心功能，其他功能由插件提供
4. 搜索效率不同。

### 2 elastic安装

[Windows环境搭建ElasticSearch 5.*并配置head](https://blog.csdn.net/u012270682/article/details/72934270)

[Elasticsearch5.2.1集群搭建，动态加入节点，并添加监控诊断插件](https://blog.csdn.net/gamer_gyt/article/details/59077189)

==注意：es需要java环境1.8 ,solr6.x以下需要1.7以上，6.x以上要1.8以上环境==

### 3 elastic的一些概览

- node与cluster

    Elastic 本质上是一个分布式数据库，允许多台服务器协同工作，每台服务器可以运行多个 Elastic 实例。单个 Elastic 实例称为一个节点（node），一组节点构成一个集群（cluster）。
 
 - index 
    
    Elastic 会索引所有字段，经过处理后写入一个反向索引（Inverted Index）。查找数据的时候，直接查找该索引。这个index就相当于一个数据库（database）

 - Document
    
    Index 里面单条的记录称为 Document（文档）。许多条 Document 构成了一个 Index。

 - type
    
    对Document进行分组，类似关系型数据库中的表，例如电商的数据可以按照地域分组，按照时间分组，按照价格分组，用于过滤document，一个index里面可以运行多个type,另外不同的type应该有相同的schema，不能id在这个type中是int类型，而在另外一个type中是string类型

    6.x 一个index中只允许一个type，7.x将彻底废除type.
    
### 4,es的一些基本java API操作

1. es集群的连接，推荐使用setting连接
    
```
public ElasticUtil(String clusterName,String ip,int port) {
		try {
			Settings settings = Settings.builder()
					.put("cluster.name", clusterName) //设置ES实例的名称
					.put("client.transport.sniff", true) //自动嗅探整个集群的状态，把集群中其他ES节点的ip添加到本地的客户端列表中
					.build(); 
			client = new PreBuiltTransportClient(settings);//初始化client
			client.addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName(ip), port)); //添加一个集群节点，自动添加
		} catch (Exception e) {
			 logger.error("connect elastic cluster faild",e);
		}
		 logger.info(">>> ElasticUtil init successfully");
	}
```

2. es插入文档，支持java bean ，map，json字符串
    
    
```
private IndexResponse insertDocument(String index, String type,
			String id, Object data) {
		IndexResponse response = null;
		try {
			// generate json
			byte[] json = mapper.writeValueAsBytes(data); //序列化一个对象
			response = (id == null) ? client.prepareIndex(index,type).setSource(json).get() : client.prepareIndex(index,type,id).setSource(json).get();
		} catch (Exception e) {
			logger.error("insert data error",e);
		}
		return response; 
	}
```

3. es 通过id查询文档


```
public String getDocumentById(String index ,String type,String id){
		GetResponse response = client.prepareGet(index, type, id)
				.setOperationThreaded(false)    // 线程安全
				.get();
		return response.getSourceAsString();
	}
```
4. es 通过组合条删除文档
    
    
```
public long deleteDocumentByCondition(String index,MatchQueryBuilder builder){
		BulkByScrollResponse response =
				DeleteByQueryAction.INSTANCE.newRequestBuilder(client)
				.filter(builder) //查询条件
				.source(index) //index(索引名)
				.get(); //执行
		long deleted = response.getDeleted(); //删除文档的数量
		return deleted;
	}
```
5. es通过组合条件查询文档


```
public void searchMutil()throws Exception{  
	    SearchRequestBuilder srb=client.prepareSearch("app").setTypes("province");  
	    QueryBuilder queryBuilder=QueryBuilders.matchPhraseQuery("name", "知乎");  
	    QueryBuilder queryBuilder2=QueryBuilders.matchPhraseQuery("adStatus", "Y");  
	    SearchResponse sr=srb.setQuery(QueryBuilders.boolQuery()  
	            .must(queryBuilder)  
	            .must(queryBuilder2))  
	        .execute()  
	        .actionGet();   
	    SearchHits hits=sr.getHits();  
	    for(SearchHit hit:hits){  
	        System.out.println(hit.getSourceAsString());  
	    }  
	}  
```
6. es通过id删除文档


```
public int deleteDocumentById(String index ,String type,String id){
		try {
			DeleteResponse response = client.prepareDelete(index, type ,id)
					.get();
		} catch (Exception e) {
			logger.error("delete data error",e);
			return -1;
		}
		return 0;
	}
```
7. es批量插入文档


```
public boolean bulkInsertDocument(String index ,String type,List<Map<String, Object>> data){
		BulkRequestBuilder bulkRequest = client.prepareBulk();
		for (Map<String, Object> map : data) {
			bulkRequest.add(client.prepareIndex(index, type)
					.setSource(map));
		}
		BulkResponse bulkResponse = bulkRequest.get();
		if (bulkResponse.hasFailures()) {
			logger.error("bulk insert data error{}",bulkResponse.buildFailureMessage());
			return false;
		}
		return true;
	}
```

更多api资料详见

[Elasticsearch Java API 手册](https://github.com/quanke/elasticsearch-java)


    
    
 
    
    

    



    
    