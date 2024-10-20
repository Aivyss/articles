# objectフィルド
elasticsearch(opensearch)ではフィルドにobject型のデータを登録することができる。

## mapping
```json
PUT movie
{
  "mappings": {
    "properties": {
      "characters": {
        "properties": {
          "name": {
            "type": "text"
          },
          "age": {
            "type": "byte"
          },
          "side": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```
## データの保存
```json
PUT movie/_doc/1
{
  "characters": {
    "name": "Iron Man",
    "age": 46,
    "side": "superhero"
  }
}
```
## 検索クエリ
```json
GET movie/_search
{
  "query": {
    "match": {
      "characters.name": "Iron Man"
    }
  }
}
```
フィルドに`.`をつけて、検索できる。


ここから新しく、2件のデータを追加し、3件のdocumentを作る。
```json
PUT movie/_doc/2
{
  "title": "The Avengers",
  "characters": [
    {
      "name": "Iron Man",
      "side": "superhero"
    },
    {
      "name": "Loki",
      "side": "villain"
    }
  ]
}

PUT movie/_doc/3
{
  "title": "Avengers: Infinity War",
  "characters": [
    {
      "name": "Loki",
      "side": "superhero"
    },
    {
      "name": "Thanos",
      "side": "villain"
    }
  ]
}
```

そして、名前が`"Loki"`でsideが`"villain"`であるmovieのドキュメントを取得しようとすると、`_id=2`と`_id=3`の結果が出力される。
```json
GET movie/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "characters.name": "Loki"
          }
        },
        {
          "match": {
            "characters.side": "villain"
          }
        }
      ]
    }
  }
}
```
```json
// 結果
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
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 1.0611372,
    "hits" : [
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : 1.0611372,
        "_source" : {
          "title" : "Avengers: Infinity War",
          "characters" : [
            {
              "name" : "Loki",
              "side" : "superhero"
            },
            {
              "name" : "Thanos",
              "side" : "villain"
            }
          ]
        }
      },
      {
        "_index" : "movie",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 0.9827781,
        "_source" : {
          "title" : "The Avengers",
          "characters" : [
            {
              "name" : "Iron Man",
              "side" : "superhero"
            },
            {
              "name" : "Loki",
              "side" : "villain"
            }
          ]
        }
      }
    ]
  }
}
```

## Flatten
上記の結果はelasticsearch(opensearch)の検索ロジックである'inverted index'が原因である。

| term (characters.name) | id   | term (characters.side) | id   |
| ---------------------- | ---- | ---------------------- | ---- |
| iron                   | 2    | superhero              | 2, 3 |
| man                    | 2    | villain                | 2, 3 |
| loki                   | 2, 3 |                        |      |
| thanos                 | 3    |                        |      |

apache luceneで処理されたinverted indexは上記の表である
(inverted indexは生成時にフィルドごとに区別して作る。)

しかし、配列のobjectの場合でも全てのobjectの各フィルドを一つのデータとして扱い(flatten)、inverted indexを作成するため、上記の表のように生成される。

# nestedフィルド
## mapping
objectたちが異なるinverted indexの構造にするためにはmappingを`"nested"`に指定する必要がある。
```json
PUT movie
{
  "mappings": {
    "properties": {
      "characters": {
        "type": "nested",
        "properties": {
          "name": {
            "type": "text"
          },
          "side": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

## 検索
nestedクエリを使う。
```json
GET movie/_search
{
  "query": {
    "nested": {
      "path": "characters",
      "query": {
        "bool": {
          "must": [
            {
              "match": {
                "characters.name": "Loki"
              }
            },
            {
              "match": {
                "characters.side": "villain"
              }
            }
          ]
        }
      }
    }
  }
}
```

![image.png](/attachment/670ff1a746a58fc932eb86e0)
`"nested"`の下にまた別のクエリがあるが、これはnestedフィルドが別の「ドキュメント」として動作することを意味する。実際elasticsearchのcore libraryであるapache luceneはnestedフィルドを別のsegmentとして管理している。

# nestedフィルドはできれば、定義しない・使わない方がいい
## IOの観点
![image.png](/attachment/670ff6d446a58fc932eb914b)
Apache luceneのsegment(≒elasticsearchのdocumentの集合体)は「修正不可能（Immutability）」である。そのため、elasticsearchのドキュメントも「修正不可能」である（それにも関わらず、削除とアップデートができる理由はelasticsearchでflagとバージョンによりドキュメントを管理しているため）。

その為、nestedフィルドがある親ドキュメントの削除・更新が行われた場合、子ドキュメントであるnestedフィルドにあるドキュメントも論理的には修正がなくても、同じく削除・更新が行われてしまう。その為、nestedフィルドがないドキュメントよりIOが頻繁に行われ、IOパフォーマンスが低下する。

- references
    - https://www.elastic.co/guide/en/elasticsearch/reference/current/nested.html
    - https://engineering.mercari.com/blog/entry/20220311-97aec2a2f8/

## 検索の観点
子ドキュメントであるnestedフィルドのための別のクエリが走り、その結果を集計して親ドキュメントのクエリを実行するため、異なるドキュメントたちを集計するにコストがかかる。その為、objectや一般的なフィルドより使うリソースが多い。

またApache luceneでは segment mergeを行い、検索性能の向上ができるが、更新・削除によりnestedフィルドも別の小さなsegmentになってしまい、segment mergeによる性能の向上が期待できない。

