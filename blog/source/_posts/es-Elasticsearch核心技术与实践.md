---
title: Elasticsearch核心技术与实践
date: 2020-1-11 19:45:12
tags: es
---
### Es相关的软件
- logStash:日志收集管道
- Kibana:可视化分析利器
- X-pack:商业化软件，6.3版本也开源了
### 关键字
- 索引(Index):作为名词，是指文档的容器，是一类文档的结合。当作为动词的时候代表把一个文档写入到索引的过程。
- 文档(Document):所有可以被搜索的数据的最小单元，在es中文档的数据会被格式化成json存储
- 节点():
- 分片(shard):索引中的数据会分散在分片中


### 数据处理
Es是根据analyze(分词器)，对文本进行分词，可以指定不同的分词器，也可以自定义分词器。当搜索的时根据输入的关键字，倒排索引到对应的doc(文档)

- CharActer Filter：Tokenizer之前对文本进行处理，例如进行删除及替换字符，可以配置多个CharActer Filter，会影响Tokenizer的Position和offset信息，例如下面一Es些自带的CharActer Filter。
1. HTML strip -去除html标签
2. Mapping - 字符串替换 
3. Pattern replace - 正则匹配替换

- Tokenizer：按照一定的规则，将文本切分成词（term（词） or token(词汇单元)），下面是一些Es内置的Tokenizers
1. whitespace(空字符串、空格、换行、tabs)
2. standard（标准分词器）
3. ual_url_email(邮件)
4. pattern(自定义正则表达式分词)
5. keyword(关键字分词,也就是不分词，会把传入的字符串当成一个term)
6. path hierarchy(文件路径分词器)
7. 
- Token Filter:将Tokenizer输出的单词（term）,进行增加，修改，删除。下面是一些自带的Token Filters
1. Lowercase(将term小写)
2. stop(停用词)
3. synonym(近义词)




### 运用场景
场景1:数据库数据同步到es用于搜索查询
场景2：大量日志数据通过消息管道、redis缓冲层到达











GET crm_opportunity_open_sea_0/_search