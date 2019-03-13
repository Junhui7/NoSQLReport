# Elasticsearch Database Report #

Junhui LI, Ge QIU, Zhiheng WU, Yue JI

Group : A4 IBO4
    
#### Dataset : 
  Stock exchange
  
#### Abstract : 
  Our group uses the data set named stock exchange, each item contains attributes such as id, compamy, price, date, etc. We design related queries based on these attributes.

#### 1.   Introduction 

Elasticsearch is an open source search engine highly scalable. It allows you to keep and analyse a great volume of information practically in real time.

Elasticsearch works with JSON documents files. Using an internal structure, it can parse your data in almost real time to search for the information you need.


#### 2.   Data prepossessing 

Elasticsearch has features like *index* and *_type*. So we need to process the data before inserting the data.
We add types and indexes in bulk in the dataset. 
Here is the program python for adding.

```
import json
import os

DATASET_FILE = 'stocks.json'
BASE_FILE = 'movielens_usersRating.json'
TYPENAME = 'stock'
INDEXNAME = 'stocks'

if not os.path.isfile(DATASET_FILE):
    print("[#] starting json modification...")
    with open(BASE_FILE, "r") as f:
        with open(DATASET_FILE, "a+") as f2:
            for line in f:
                data = json.loads(line.strip('\n'))
                print data
                
                index = {"index":{"_index":INDEXNAME, "_type":TYPENAME, "_id":data["_id"]}}
                data.pop("_id")
                fields={}
                fields["fields"] = data
                # data["field"] = data["_id"]["$oid"]
                f2.write(json.dumps(index)+'\n')
                f2.write(json.dumps(fields)+'\n')
    print("[#] modification done")

```

#### 3.   Import dataset with cURL

```
curl -XPUT localhost:9200/_bulk -H"Content-Type: application/json" --data-binary @stock.json
```

#### 4.   Queries 

- Simple queries:
    
    *1. find the data of company name which contains "Trust"(simple match)*

```
GET _search
{
  "query": {
    "match" : {
      "fields.Company": "Trust"
    }
  }
}
```

![Simple 1.png](https://upload-images.jianshu.io/upload_images/2381215-83450b31140beb9e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 *2. match the company contains ”Trust” and Country contains "USA"(simple should)*
   
    
```
GET _search
{
  "query": {
    "bool": {
      "should":[{
          "match":{ 
            "fields.Company": "Trust"
          }},{
          "match":{ 
            "fields.description.Country": "USA"
          }}
        ]
    }
  }
}
```
![Simple 2.png](https://upload-images.jianshu.io/upload_images/2381215-856b8cfc6479cc97.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 *3.	find that the Sector must be Services and Company should contains "America"*
    
```
GET _search
{
  "query": {
    "bool": {
      "should":{
        "match":{
          "fields.Company": "America"
          }
        },
      "must":{
        "match":{
          "fields.description.Sector": "Services"
        }
      }
    }
  }
}
```

![Simple 3.png](https://upload-images.jianshu.io/upload_images/2381215-ae3eeb5bfe5fdbfb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*4.    find the Industry named "Management Services"(simple match_phrase)*
    
```
GET _search
{
  "query": {
    "match_phrase": {
      "fields.description.Industry": "Management Services"
    }
  }
}
```

![Simple 4.png](https://upload-images.jianshu.io/upload_images/2381215-259afc756359c004.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*5.	find the ROI greater than 2(simple range)*
    
```
GET _search
{
  "query": {
    "bool":{
      "must":{
        "range":{
          "fields.ROI":{
            "gte": 2
          }
        }
      }
    }
  }
}
```

![Simple 5.png](https://upload-images.jianshu.io/upload_images/2381215-21c778b9101d2fc1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*6.	find the Earning Date that between 2014-01-01 and 2015-01-01(simple date)*
    
```
GET _search
{
  "query": {
    "bool":{
      "must":{
        "range":{
          "fields.Earnings Date.$date":{
            "from": "2014-01-01",
            "to": "2015-01-01"
          }
        }
      }
    }
  }
}
```

![Simple 6.png](https://upload-images.jianshu.io/upload_images/2381215-9315688d85ddbc06.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- Complex queries:
    
    *1.	show the year performance aggregation in desc order(simple terms)*
    
```
GET _search
{
  "aggs": {
    "years": {
      "terms": {
        "field": "fields.performance.Year",
        "size": 10,
        "order": {
          "_count": "desc"
        }
      }
    }
  }
}
```

![Complex 1.png](https://upload-images.jianshu.io/upload_images/2381215-4792287bc8579dcd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*2.	Group by performance.year and find their average ROI(composite)*
    
```
GET _search
{
  "aggs": {
    "group_by_year": {
      "terms": {
        "field": "fields.performance.Year"
      },
      "aggs": {
        "avg_by_ROI": {
          "avg": {
            "field": "fields.ROI"
          }
        }
      }
    }
  }
}
```

![Complex 2.png](https://upload-images.jianshu.io/upload_images/2381215-911ffcc9cfe2b2e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    
*3.	number of distinct industry(cardinality)*
    
![Complex 3.png](https://upload-images.jianshu.io/upload_images/2381215-510643e628de0596.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

So first put the industry as fielddata.

```
GET _search
{
  "aggs": {
    "distinct_count": {
      "cardinality": {
        "field": "fields.description.Industry"
      }
    }
  }
}
PUT /movies/movie/_mappings
{ "properties": { "fields.description.Industry": { "type": "text", "fielddata": true } } }
```
![Complex 3.png](https://upload-images.jianshu.io/upload_images/2381215-6d33a06e2ebb0b4b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- Hard query:
    
    *filter the average volume that greater than 5000, then group by the performance of the year, finally calculate their average ROI*
    
```
GET _search
{
    "query" : {
        "bool" : {
          "must" : {
            "range" : {
              "fields.Average Volume":{
                "gte" : 5000
              }
            }
          }
        }
    },
    "aggs": {
      "group_by_year": {
        "terms": {
          "field": "fields.description.Country",
          "size": 5,
          "order": {
            "_count": "desc"
          }
        },
        "aggs": {
          "average_of_ROI": {
            "avg": {
              "field": "fields.ROI"
            }
          }
        }
      }
    }
}
PUT /movies/movie/_mappings
{ "properties": { "fields.description.Country": { "type": "text", "fielddata": true } } }
```
![Hard.png](https://upload-images.jianshu.io/upload_images/2381215-89919dfe4e91d53c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 5.   Disscussion and Conclusion 

-   Problems： Group by range

![Problem.png](https://upload-images.jianshu.io/upload_images/2381215-bb96b412c2dc0486.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Cannot find proper position for the ranges within a single aggs.
But we can do it by mapping or composite aggregations. 

-   Conclusion

This project, given the many difficulties encountered, really allowed us to understand the basics of Elasticsearch. We realized that an organization, clarity and good structure in the Elasticsearch is paramount so as not to fall into the mistakes and pitfalls of such language. 

Finally, this project allowed us to concretely practice the Elasticsearch and the data structure, which may later in our journey help us better understand the logic of NoSQL.

