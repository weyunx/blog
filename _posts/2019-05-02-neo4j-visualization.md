---
layout:     post
title:      neo4j 可视化踩坑
subtitle:   
author:     WeYunx
header-style: text
catalog: true
signature: true
tags:
    - neo4j

---

最近接手了一个数据知识图谱展示的项目，其中图数据库用的是 neo4j，前端可视化组件用的是 ECharts 3。上线后用户反应图谱查询较慢，故分析了一下项目的代码，发现了一个编码思路的问题。

一般图的可视化组件，如 ECharts，Cytoscape.js 等，接收的图数据格式是按照节点和关系来分类的，类似：

```javascript
    /*
    nodes:id,label,properties
    edges:id,source,target,title
    */
data = {
            nodes: nodes,
            edges: edges
        };
```

但是我们代码里后台查询出来的结果是按照`节点`的形式，然后再通过节点中遍历出关系信息，组装成上文的 `nodes\edges` 格式的 json 返回到前端。根据一番测试，发现图数据库查询耗时都在毫秒级，瓶颈就在这个 json 组装上，通常耗费2s+。

后来通过查询资料，发现 neo4j 自带提供 `graph format` 格式的返回，专门用于可视化组件。不过此种方式仅在 HTTP API 中提供:

发送请求：

- **POST** http://localhost:7474/db/data/transaction/commit
- **Accept:** application/json; charset=UTF-8
- **Content-Type:** application/json

```json
{
  "statements" : [ {
    "statement" : "CREATE ( bike:Bike { weight: 10 } ) CREATE ( frontWheel:Wheel { spokes: 3 } ) CREATE ( backWheel:Wheel { spokes: 32 } ) CREATE p1 = (bike)-[:HAS { position: 1 } ]->(frontWheel) CREATE p2 = (bike)-[:HAS { position: 2 } ]->(backWheel) RETURN bike, p1, p2",
    "resultDataContents" : [ "row", "graph" ]
  } ]
}
```



返回结果：

- **200:** OK
- **Content-Type:** application/json

```json
{
  "results" : [ {
    "columns" : [ "bike", "p1", "p2" ],
    "data" : [ {
      "row" : [ {
        "weight" : 10
      }, [ {
        "weight" : 10
      }, {
        "position" : 1
      }, {
        "spokes" : 3
      } ], [ {
        "weight" : 10
      }, {
        "position" : 2
      }, {
        "spokes" : 32
      } ] ],
      "meta" : [ {
        "id" : 7,
        "type" : "node",
        "deleted" : false
      }, [ {
        "id" : 7,
        "type" : "node",
        "deleted" : false
      }, {
        "id" : 0,
        "type" : "relationship",
        "deleted" : false
      }, {
        "id" : 8,
        "type" : "node",
        "deleted" : false
      } ], [ {
        "id" : 7,
        "type" : "node",
        "deleted" : false
      }, {
        "id" : 1,
        "type" : "relationship",
        "deleted" : false
      }, {
        "id" : 9,
        "type" : "node",
        "deleted" : false
      } ] ],
      "graph" : {
        "nodes" : [ {
          "id" : "7",
          "labels" : [ "Bike" ],
          "properties" : {
            "weight" : 10
          }
        }, {
          "id" : "8",
          "labels" : [ "Wheel" ],
          "properties" : {
            "spokes" : 3
          }
        }, {
          "id" : "9",
          "labels" : [ "Wheel" ],
          "properties" : {
            "spokes" : 32
          }
        } ],
        "relationships" : [ {
          "id" : "0",
          "type" : "HAS",
          "startNode" : "7",
          "endNode" : "8",
          "properties" : {
            "position" : 1
          }
        }, {
          "id" : "1",
          "type" : "HAS",
          "startNode" : "7",
          "endNode" : "9",
          "properties" : {
            "position" : 2
          }
        } ]
      }
    } ]
  } ],
  "errors" : [ ]
}
```

可以看到 `graph` 节点中的数据格式符合我们可视化组件的要求。

通过这次改造，解决了图谱展示慢的问题，如果大家也有图谱展示的要求，不妨试试这种方式，欢迎交流～



## 参考

- https://neo4j.com/docs/http-api/current/actions/return-results-in-graph-format/