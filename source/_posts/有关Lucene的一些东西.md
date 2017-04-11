---
title: 有关Lucene的一些东西
date: 2017-04-11 14:56:50
tags: Lucene
categories: 工作
---

因为项目中用到了Lucene,这里简单的说一下关于Lucene的一些东西.
## 大概介绍
Lucene 是一个基于 Java 的全文信息检索工具包，它不是一个完整的搜索应用程序，而是为你的应用程序提供索引和搜索功能。Lucene 目前是 Apache Jakarta 家族中的一个开源项目。也是较为流行的基于 Java 开源全文检索工具包。
图 1 表示了搜索应用程序和 Lucene 之间的关系，也反映了利用 Lucene 构建搜索应用程序的流程：
图 1. 搜索应用程序和 Lucene 之间的关系
![搜索应用程序和 Lucene 之间的关系](/images/有关Lucene的一些东西001.jpg)
## 索引
举个例子,索引有点类似目录,就是什么内容,会告诉你在第几页，这样就能帮助读者比较快地找到相关内容的页码。
而数据库索引能够大大提高查询的速度原理也是一样，通过目录查找的速度要比一页一页地翻内容高多少倍……当然他还有另外一个原因是它是排好序的。对于检索系统来说核心是一个排序问题。
而数据库索引不是为全文索引设计的，因此，使用like数据库索引不是为全文索引设计的，因此，使用like "%keyword%"时，数据库索引是不起作用的，
在使用like查询时，搜索过程又变成类似于一页页翻书的遍历过程了，所以对于含有模糊查询的数据库服务来说，LIKE对性能的危害是极大的。
如果是需要对多个关键词进行模糊匹配：like"%keyword1%" and like "%keyword2%" ...其效率也就可想而知了。

所以建立一个高效检索系统的关键是建立一个类似于科技索引一样的反向索引机制，将数据源（比如多篇文章）排序顺序存储的同时，有另外一个排好序的关键词列表，
用于存储关键词==>文章映射关系，利用这样的映射关系索引：[关键词==>出现关键词的文章编号，出现次数（甚至包括位置：起始偏移量，结束偏移量），出现频率]，
检索过程就是把模糊查询变成多个可以利用索引的精确查询的逻辑组合的过程。从而大大提高了多关键词查询的效率，所以，全文检索问题归结到最后是一个排序问题。

由此可以看出模糊查询相对数据库的精确查询是一个非常不确定的问题，这也是大部分数据库对全文检索支持有限的原因。
Lucene最核心的特征是通过特殊的索引结构实现了传统数据库不擅长的全文索引机制，并提供了扩展接口，以方便针对不同应用的定制。

## lucene的一些包
org.apache.lucene.document:  存储结构
这个包提供了一些为封装要索引的文档所需要的类，比如 Document, Field。这样，每一个文档最终被封装成了一个 Document 对象。
org.apache.lucene.analysis: 语言分析器
这个包主要功能是对文档进行分词，因为文档在建立索引之前必须要进行分词，所以这个包的作用可以看成是为建立索引做准备工作。
org.apache.lucene.index: 索引入口
这个包提供了一些类来协助创建索引以及对创建好的索引进行更新。这里面有两个基础的类：IndexWriter 和 IndexReader，其中 IndexWriter 是用来创建索引并添加文档到索引中的，IndexReader 是用来删除索引中的文档的。
org.apache.lucene.search: 搜索入口
这个包提供了对在建立好的索引上进行搜索所需要的类。比如 IndexSearcher 和 Hits, IndexSearcher 定义了在指定的索引上进行搜索的方法，Hits 用来保存搜索得到的结果。
org.apache.Lucene.queryParser:	查询分析器
org.apache.Lucene.store: 底层IO/存储结构
org.apache.Lucene.util: 一些公用的数据结构
...


## 建立索引
为了对文档进行索引，Lucene 提供了五个基础的类，他们分别是 Document, Field, IndexWriter, Analyzer, Directory。下面我们分别介绍一下这五个类的用途：
Document
Document 是用来描述文档的，这里的文档可以指一个 HTML 页面，一封电子邮件，或者是一个文本文件。一个 Document 对象由多个 Field 对象组成的。可以把一个 Document 对象想象成数据库中的一个记录，而每个 Field 对象就是记录的一个字段。
Field
Field 对象是用来描述一个文档的某个属性的，比如一封电子邮件的标题和内容可以用两个 Field 对象分别描述。
Analyzer
在一个文档被索引之前，首先需要对文档内容进行分词处理，这部分工作就是由 Analyzer 来做的。Analyzer 类是一个抽象类，它有多个实现。针对不同的语言和应用需要选择适合的 Analyzer。Analyzer 把分词后的内容交给 IndexWriter 来建立索引。
IndexWriter
IndexWriter 是 Lucene 用来创建索引的一个核心的类，他的作用是把一个个的 Document 对象加到索引中来。
Directory
这个类代表了 Lucene 的索引的存储的位置，这是一个抽象类，它目前有两个实现，第一个是 FSDirectory，它表示一个存储在文件系统中的索引的位置。第二个是 RAMDirectory，它表示一个存储在内存当中的索引的位置。
熟悉了建立索引所需要的这些类后，我们就开始对某个目录下面的文本文件建立索引了，清单 1 给出了对某个目录下的文本文件建立索引的源代码。


## 搜索文档
利用 Lucene 进行搜索就像建立索引一样也是非常方便的。在上面一部分中，我们已经为一个目录下的文本文档建立好了索引，现在我们就要在这个索引上进行搜索以找到包含某个关键词或短语的文档。Lucene 提供了几个基础的类来完成这个过程，它们分别是呢 IndexSearcher, Term, Query, TermQuery, Hits. 下面我们分别介绍这几个类的功能。
Query
这是一个抽象类，他有多个实现，比如 TermQuery, BooleanQuery, PrefixQuery. 这个类的目的是把用户输入的查询字符串封装成 Lucene 能够识别的 Query。
Term
Term 是搜索的基本单位，一个 Term 对象有两个 String 类型的域组成。生成一个 Term 对象可以有如下一条语句来完成：Term term = new Term(“fieldName”,”queryWord”); 其中第一个参数代表了要在文档的哪一个 Field 上进行查找，第二个参数代表了要查询的关键词。
TermQuery
TermQuery 是抽象类 Query 的一个子类，它同时也是 Lucene 支持的最为基本的一个查询类。生成一个 TermQuery 对象由如下语句完成： TermQuery termQuery = new TermQuery(new Term(“fieldName”,”queryWord”)); 它的构造函数只接受一个参数，那就是一个 Term 对象。
IndexSearcher
IndexSearcher 是用来在建立好的索引上进行搜索的。它只能以只读的方式打开一个索引，所以可以有多个 IndexSearcher 的实例在一个索引上进行操作。
Hits
Hits 是用来保存搜索的结果的。

## 项目使用总结
现在的项目是这样使用Lucene的:
import jar->用job初始化待查询数据(建立索引)->查询返回(搜索文档),因为Lucene有很简单的api,这里就不细说了,结尾附上大致的代码例子:

## 一个例子
创建索引:
```java	
/**
 * 处理百科信息
 */
public void procAllForTermBaike() throws Exception {
	List<Map<String, String>> baikeList = new ArrayList<Map<String, String>>();
	Query<TermBaike> query = this.createQuery();
	long counts = this.count(query);
	Integer pages = this.luceneService.getPages(Integer.valueOf(String.valueOf(counts)));
	for (int i = 1; i <= pages; i++) {
		// 查询获取数据
		query.offset((i - 1) * 100).limit(100);
		QueryResults<TermBaike> result = this.find(query);
		List<TermBaike> TermBaikeList = result.asList();
		// 封装返回数据
		for (TermBaike termBaike : TermBaikeList) {
			Map<String, String> map = new HashMap<String, String>();
			map.put("id", termBaike.get_id());
			map.put("name", termBaike.getVcName());
			map.put("nameEn", termBaike.getVcNameEn());
			map.put("label", String.valueOf(ServiceTypeEnum.BAIKE_LAB.getValue()));
			baikeList.add(map);
		}
	}
	this.luceneService.createIndex(ServiceTypeEnum.BAIKE.getKey(), baikeList, Boolean.TRUE);
}
```
...job相关的代码就不放上来了,就是用job跑这个procxxx方法



调用的方法:

```java
/**
 * 获取页数
 */
public Integer getPages(Integer totalNum) throws Exception {
	Integer pages = 1;
	if ((totalNum % 100) == 0) {
		pages = totalNum / 100;
	} else {
		pages = (totalNum / 100) + 1;
	}
	return pages;
}
```

```java
/**
 * 创建索引
 */
public void createIndex(Integer serviceType, List<Map<String, String>> list, Boolean isDel) throws Exception {
	IndexWriter indexWriter = this.initIndexWriter(serviceType, isDel);
	try {
		for (Map<String, String> map : list) {
			Document doc = new Document();
			for (Map.Entry<String, String> entry : map.entrySet()) {
				if (null != entry.getKey() && null != entry.getValue()) {
					doc.add(new TextField(entry.getKey(), entry.getValue(), Field.Store.YES));
				}
			}
			indexWriter.addDocument(doc);
		}
	} catch (Exception e) {
		logger.error(e);
		e.printStackTrace();
	} finally {
		if (null != indexWriter) {
			indexWriter.close();
		}
	}
}
```

搜索文档:
```java
/**
 * 百科信息查询
 * 
 * @param key 关键字
 * @param currentPage 页码
 * @return
 * @throws Exception
 */
public List<TermBaikeVo> queryBaike(String key, Integer currentPage) throws Exception {
	List<TermBaikeVo> termBaikeVoList = new ArrayList<TermBaikeVo>();
	Sort sort = new Sort(new SortField("name", SortField.Type.STRING, true));
	List<Document> docList = this.luceneService.query(ServiceTypeEnum.BAIKE.getKey(), key, "name", sort, currentPage, DEF_ENUM.DEF_TWENTY.getnCode());
	for (Document doc : docList) {
		TermBaikeVo termBaikeVo = new TermBaikeVo();
		termBaikeVo.setBaikeId(doc.get("id"));
		termBaikeVo.setVcName(doc.get("name"));
		termBaikeVo.setVcName(doc.get("nameEn"));
		termBaikeVo.setVcLabel(doc.get("label"));
		termBaikeVoList.add(termBaikeVo);
	}
	return termBaikeVoList;
}
```

调用的方法:

```java
/**
* 单条件分页查询
*/
public List<Document> query(Integer serviceType, String key, String field, Sort sort, Integer currentPage, Integer pageSize) throws Exception {
	List<Document> docList = new ArrayList<Document>();
	IndexReader reader = this.initIndexReader(serviceType);
	try {
		Analyzer analyzer = new StandardAnalyzer();
		IndexSearcher indexSearcher = new IndexSearcher(reader);
		QueryParser parser = new QueryParser(field, analyzer);
		org.apache.lucene.search.Query query = parser.createPhraseQuery(field, key);
		TopDocs topDocs = indexSearcher.search(query, currentPage + pageSize, sort);
		ScoreDoc[] scoreDocs = topDocs.scoreDocs;
		if (scoreDocs.length == 0) {
			scoreDocs = this.wildcardQuery(key, field, indexSearcher);
		}
		int begin = pageSize * (currentPage - 1);
		int end = Math.min(begin + pageSize, scoreDocs.length);
		for (int i = begin; i < end; i++) {
			int docID = scoreDocs[i].doc;
			Document doc = indexSearcher.doc(docID);
			docList.add(doc);
		}
	} catch (Exception e) {
		logger.error(e);
	} finally {
		if (null != reader) {
			reader.close();
		}
	}
	return docList;
}
```


