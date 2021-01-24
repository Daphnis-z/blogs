# 1. 如何清空索引里面的数据

**以使用 head 插件为例：**

http://ip:9200/index-name/

_delete_by_query  POST

```json
{
	"query":{
    "match_all": {}
  }
}
```

**在 Linux 上可以使用 curl 请求：**

```shell
curl -XPOST -H 'Content-Type:application/json' http://ip:9200/index-name/_delete_by_query -d '{"query":{"match_all": {}}}'
eg. 
curl -XPOST -H 'Content-Type: application/json' http://127.0.0.1:9200/demo-data-202008/_delete_by_query -d '{"query":{"match_all": {}}}'
```

# 2. 管理模板

**创建模板：**

http://ip:9200/_template/template-name/

POST

```json
{
  "template": "demo-data-*",
  "order": 1,
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 1
  },
  "mappings": {
    "properties": {
      "goods_id": {
        "type": "keyword"
      }
    }
  },
  "aliases": {
    "demo-data-all": {}
  }
}
```

**删除模板：**

http://ip:9200/_template/template-name/

DELETE

```json
{}
```





