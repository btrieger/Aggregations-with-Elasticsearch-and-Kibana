# Aggregations with Elasticsearch and Kibana

## Resources


[Free Elastic Cloud Trial](https://www.elastic.co/cloud/cloud-trial-overview/30-days?ultron=community-beginners-crash-course-May2022+&hulk=30d)

[Instructions](https://dev.to/lisahjung/beginner-s-guide-to-setting-up-elasticsearch-and-kibana-with-elastic-cloud-1joh) on how to access Elasticsearch and Kibana on Elastic Cloud

[E-commerce Dataset](https://www.kaggle.com/carrie1/ecommerce-data) from Kaggle

## Set up data within Elasticsearch
Often times, the dataset is not optimal for running requests in its original state. 

For example, the type of a field may not be recognized by Elasticsearch or the dataset may contain a value that was accidentally included in the wrong field and etc. 

These are exact problems that I ran into while working with this dataset. The following are the requests that I sent to yield the results shared during the workshop. 

Copy and paste these requests into the Kibana console(Dev Tools) and run these requests in the order shown below. 

**STEP 1: Create a new index(ecommerce_data) with the following mapping.** 
```http
PUT ecommerce_data
{
  "mappings": {
    "properties": {
      "Country": {
        "type": "keyword"
      },
      "CustomerID": {
        "type": "long"
      },
      "Description": {
        "type": "text"
      },
      "InvoiceDate": {
        "type": "date",
        "format": "M/d/yyyy H:m"
      },
      "InvoiceNo": {
        "type": "keyword"
      },
      "Quantity": {
        "type": "long"
      },
      "StockCode": {
        "type": "keyword"
      },
      "UnitPrice": {
        "type": "double"
      }
    }
  }
}
```   
**STEP 2: Reindex the data from the original index(source) to the one you just created(destination).**
```http
POST _reindex
{
  "source": {
    "index": "name of your original index when you added the data to Elasticsearch"
  },
  "dest": {
    "index": "ecommerce_data"
  }
}
```
**STEP 3: Remove the negative values from the field "UnitPrice".**

When you explore the minimum unit price in this dataset, you will see that the minimum unit price value is -11062.06. To keep our data simple, I used the delete_by_query API to remove all unit prices less than 1. 

```http
POST ecommerce_data/_delete_by_query
{
  "query": {
    "range": {
      "UnitPrice": {
        "lte": 1
      }
    }
  }
}
```

**STEP 4: Remove values greater than 500 from the field "UnitPrice".**

When you explore the maximum unit price in this dataset, you will see that the maximum unit price value is 38,970. When the data is manually examined, the majority of the unit prices are less than 500. The max value of 38,970 would skew the average. To simplify our demo, I used the delete_by_query API to remove all unit prices greater than 500.
```http
POST ecommerce_data/_delete_by_query
{
  "query": {
    "range": {
      "UnitPrice": {
        "gte": 500
      }
    }
  }
}
```

### Get information about documents in an index
The following query will retrieve information about documents in the `ecommerce_data` index. This query is a great way to explore the structure and content of your document. 

Syntax: 
```http
GET Enter_name_of_the_index_here/_search
```
Example: 
```http
GET ecommerce_data/_search
```
Expected response from Elasticsearch:

Elasticsearch displays a number of hits(line 12) and a sample of 10 search results by default(lines 16+). 

The first hit(a document) is shown on lines 17-31. The field "source"(line 22) lists all the fields(the content) of the document.

![image](https://user-images.githubusercontent.com/60980933/112375185-9c52d280-8ca8-11eb-9952-16f24171dfbd.png)

## Aggregations Request
Syntax:
```http
GET Enter_name_of_the_index_here/_search
{
  "aggs": {
    "Name your aggregations here": {
      "Specify the aggregation type here": {
        "field": "Name the field you want to aggregate on here"
      }
    }
  }
}
```
### Metric Aggregations 
`Metric`aggregations are used to compute numeric values based on your dataset. It can be used to calculate the values of `sum`,`min`, `max`, `avg`, unique count(`cardinality`) and etc.  

#### Compute the `sum` of all unit prices in the index

Syntax:
```http
GET Enter_name_of_the_index_here/_search
{
  "aggs": {
    "Name your aggregations here": {
      "sum": {
        "field": "Name the field you want to aggregate on here"
      }
    }
  }
}
```
Example:
```http
GET ecommerce_data/_search
{
  "aggs": {
    "sum_unit_price": {
      "sum": {
        "field": "UnitPrice"
      }
    }
  }
}
```
Expected response from Elasticsearch:

By default, Elasticsearch returns top 10 hits(Lines 16+). 

![image](https://user-images.githubusercontent.com/60980933/114756805-5b366700-9d18-11eb-8ea4-1164821e715f.png)

When you minimize hits(red box- line 10), you will see the aggregations results we named sum_unit_price(image below). It displays the sum of all unit prices present in our index. 

![image](https://user-images.githubusercontent.com/60980933/114757167-c122ee80-9d18-11eb-9d7a-3856612c7de0.png)

If the purpose of running an aggregation is solely to get the aggregations results, you can add a size parameter and set it to 0 as shown below. This parameter prevents Elasticsearch from fetching the top 10 hits so that the aggregations results are shown at the top of the response. 

**Using a size parameter**

Example:
```http
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "sum_unit_price": {
      "sum": {
        "field": "UnitPrice"
      }
    }
  }
}
```
Expected response from Elasticsearch:

We no longer need to minimize the hits to get access to the aggregations results!  We will be setting the size parameter to 0 in all aggregations requests from this point on. 

![image](https://user-images.githubusercontent.com/60980933/114758361-1c091580-9d1a-11eb-94df-58afa67e20c4.png)

#### Compute the lowest(`min`) unit price of an item 

Syntax:
```http
GET Enter_name_of_the_index_here/_search
{
  "size": 0,
  "aggs": {
    "Name your aggregations here": {
      "min": {
        "field": "Name the field you want to aggregate on here"
      }
    }
  }
}
```
Example: 
```http
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "lowest_unit_price": {
      "min": {
        "field": "UnitPrice"
      }
    }
  }
}
```
Expected response from Elasticsearch:

The lowest unit price of an item is 1.01. 
![image](https://user-images.githubusercontent.com/60980933/112509885-869be680-8d56-11eb-9c2e-5935ff7437e8.png)

#### Compute the highest(`max`) unit price of an item 

Syntax:
```http
GET Enter_name_of_the_index_here/_search
{
  "size": 0,
  "aggs": {
    "Name your aggregations here": {
      "max": {
        "field": "Name the field you want to aggregate on here"
      }
    }
  }
}
```
Example: 

```http
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "highest_unit_price": {
      "max": {
        "field": "UnitPrice"
      }
    }
  }
}
```
Expected response from Elasticsearch:

The highest unit price of an item is 498.79. 

![image](https://user-images.githubusercontent.com/60980933/112511189-cca57a00-8d57-11eb-9ab3-809b2a410636.png)

#### Compute the `average` unit price of items 

Syntax:
```http
GET Enter_name_of_the_index_here/_search
{
  "size": 0,
  "aggs": {
    "Name your aggregations here": {
      "avg": {
        "field": "Name the field you want to aggregate on here"
      }
    }
  }
}
```
Example: 
```http
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "average_unit_price": {
      "avg": {
        "field": "UnitPrice"
      }
    }
  }
}
```
Expected response from Elasticsearch: 

The average unit price of an item is ~4.39.

![image](https://user-images.githubusercontent.com/60980933/112511759-58b7a180-8d58-11eb-811f-8d6cb852c220.png)

#### `Stats` Aggregation: Compute the count, min, max, avg, sum in one go

Syntax:
```http
GET Enter_name_of_the_index_here/_search
{
  "size": 0,
  "aggs": {
    "Name your aggregations here": {
      "stats": {
        "field": "Name the field you want to aggregate on here"
      }
    }
  }
}
```
Example:
```http
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "all_stats_unit_price": {
      "stats": {
        "field": "UnitPrice"
      }
    }
  }
}
```
Expected Response from Elasticsearch:

The `stats aggregation` will yield the values of `count`(the number of unit prices aggregation was performed on), `min`, `max`, `avg`, and `sum`(sum of all unit prices in the index). 

![image](https://user-images.githubusercontent.com/60980933/114769078-f20a2000-9d26-11eb-9827-e7672cbba158.png)

#### `Cardinality` Aggregation

The `cardinality aggregation` computes the count of unique values for a given field. 

Syntax:
```http
GET Enter_name_of_the_index_here
{
  "size": 0,
  "aggs": {
    "Name your aggregations here": {
      "cardinality": {
        "field": "Name the field you want to aggregate on here"
      }
    }
  }
}
```
Example: 
```http
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "number_unique_customers": {
      "cardinality": {
        "field": "CustomerID"
      }
    }
  }
}
```
Expected response from Elasticsearch: 

Approximately, there are 4325 unique number of customers in our index.  
![image](https://user-images.githubusercontent.com/60980933/114774709-aeff7b00-9d2d-11eb-9da5-8faf0dc87292.png)

#### Limiting the scope of an aggregation

In the previous examples, aggregations were performed on all documents in the ecommerce_data index. What if you want to run an aggregation on a subset of the documents? 

For example, our index contains e-commerce data from multiple countries. Let's say you want to calculate the average unit price of items sold in Germany.

To limit the scope of the aggregation, you can add a query clause to the aggregations request. The query clause defines the subset of documents that aggregations should be performed on.  

The combined query and aggregations look like the following. 

Syntax:
```http
GET Enter_name_of_the_index_here/_search
{
  "size": 0,
  "query": {
    "Enter match or match_phrase here": {
      "Enter the name of the field": "Enter the value you are looking for"
    }
  },
  "aggregations": {
    "Name your aggregations here": {
      "Specify aggregations type here": {
        "field": "Name the field you want to aggregate here"
      }
    }
  }
}
```
Example: 
```http
GET ecommerce_data/_search
{
  "size": 0,
  "query": {
    "match": {
      "Country": "Germany"
    }
  },
  "aggs": {
    "germany_average_unit_price": {
      "avg": {
        "field": "UnitPrice"
      }
    }
  }
}
```
Expected response from Elasticsearch:

The average of unit price of items sold in Germany is ~4.58.
![image](https://user-images.githubusercontent.com/60980933/112534501-c1ab1380-8d70-11eb-9ce7-507953cc26d0.png)

The combination of query and aggregations request allowed us to perform aggregations on a subset of documents. What if we wanted to perform aggregations on several subsets of documents? 

This is where `Bucket Aggregations` come into play! 

###  Bucket Aggregations
When you want to aggregate on several subsets of documents, `bucket aggregations` will come in handy. `Bucket aggregations` group documents into several sets of documents called buckets. All documents in a bucket share a common criteria.  

![image](https://user-images.githubusercontent.com/60980933/115422110-d2a54400-a1b9-11eb-919c-15a88f9d148b.png)

The following are different types of `bucket aggregations`. 

1. Date Histogram Aggregation
2. Histogram Aggregation
3. Range Aggregation
4. Terms aggregation

#### 1. `Date Histogram` Aggregation
When you are looking to group data by time interval, the `date_histogram aggregation` will prove very useful! 

Our ecommerce_data index contains transaction data that has been collected over time(from the year 2010 to 2011). 

If we are looking to get insights about transactions over time, our first instinct should be to run the `date_histogram aggregation`. 

There are two ways to define a time interval with `date_histogram aggregation`. These are `Fixed_interval` and `Calendar_interval`.

**Fixed_interval**
With the `fixed_interval`, the interval is always **constant**.

Example: Create a bucket for every 8 hour interval. 

Syntax:
```http
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "Name your aggregations here": {
      "date_histogram": {
        "field":"Name the field you want to aggregate on here",
        "fixed_interval": "Specify the interval here"
      }
    }
  }
}
```

Example:
```http
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "transactions_by_8_hrs": {
      "date_histogram": {
        "field": "InvoiceDate",
        "fixed_interval": "8h"
      }
    }
  }
}
```
Expected response from Elasticsearch:

Elasticsearch creates a bucket for every 8 hours("key_as_string") and shows the number of documents("doc_count") grouped into each bucket. 

![image](https://user-images.githubusercontent.com/60980933/115752687-927bc800-a357-11eb-8ded-77038d9d11fa.png)

Another way we can define the time interval is through the `calendar_interval`. 

**Calendar_interval**
With the `calendar_interval`, the interval may **vary**.

For example, we could choose a time interval of day, month or year. But daylight savings can change the length of specific days, months can have different number of days, and leap seconds can be tacked onto a particular year. 

So the time interval of day, month, or leap seconds could vary! 

A scenario where you might use the `calendar_interval` is when you want to calculate the monthly revenue. 

Ex. Split data into monthly buckets.

Syntax:
```http
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "Name your aggregations here": {
      "date_histogram": {
        "field":"Name the field you want to aggregate on here",
        "calendar_interval": "Specify the interval here"
      }
    }
  }
}
```
Example: 
```http
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "transactions_by_month": {
      "date_histogram": {
        "field": "InvoiceDate",
        "calendar_interval": "1M"
      }
    }
  }
}
```
Expected response from Elasticsearch:

Elasticsearch creates monthly buckets. Within each bucket, the date(monthly interval) is included in the field "key_as_string". The field "key" shows the same date represented as a timestamp. The field "doc_count" shows the number of documents that fall within the time interval. 

![image](https://user-images.githubusercontent.com/60980933/115707200-b9bca000-a32b-11eb-8d33-f089e5616b90.png)

**Bucket sorting for date histogram aggregation**

By default, the `date_histogram aggregation` sorts buckets based on the "key"
values in ascending order. 

To reverse this order, you can add an `order` parameter to the `aggregations` as shown below. Then, specify that you want to sort buckets based on the "_key" values in descending(desc) order.

Example:
```http
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "transactions_by_month": {
      "date_histogram": {
        "field": "InvoiceDate",
        "calendar_interval": "1M",
        "order": {
          "_key": "desc"
        }
      }
    }
  }
}
```
Expected response from Elasticsearch:

You will see that buckets are now sorted to return the most recent interval first.

![image](https://user-images.githubusercontent.com/60980933/115941140-72383000-a461-11eb-90c2-a86cc1ae8233.png)

#### Histogram Aggregation
With the `date_histogram aggregation`, we were able to create buckets based on time intervals. 

The `histogram aggregation` creates buckets based on any numerical interval. 

Ex. Create a buckets based on price interval that increases in increments of 10. 

Syntax: 
```http
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "Name your aggregations here": {
      "histogram": {
        "field":"Name the field you want to aggregate on here",
        "interval": Specify the interval here
      }
    }
  }
}
```
Example: 
```http
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "transactions_per_price_interval": {
      "histogram": {
        "field": "UnitPrice",
        "interval": 10
      }
    }
  }
}
```

Expected response from Elasticsearch:

Elasticsearch returns a buckets array where each bucket represents a price interval("key"). Each interval increases in increments of 10 in unit price. It also includes the number of documents placed in each bucket("doc_count"). 

![image](https://user-images.githubusercontent.com/60980933/115754790-cf48be80-a359-11eb-82c8-128033e452cf.png)

**Bucket sorting for histogram aggregation**

By default, the `histogram aggregation` sorts buckets based on the `_key`
values in ascending order. To reverse this order, you can add an order parameter to the aggregation. Then, specify that you want to sort buckets based on the `_key` values in descending(desc) order! 

Example:
```http
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "transactions_per_price_interval": {
      "histogram": {
        "field": "UnitPrice",
        "interval": 10,
        "order": {
          "_key": "desc"
        }
      }
    }
  }
}
```
Expected response from Elasticsearch:

You will see that the buckets are now sorted to return the price intervals in descending order.

![image](https://user-images.githubusercontent.com/60980933/116111564-01b52d00-a674-11eb-99a1-3891718d414c.png)g)

#### Range Aggregation

The `range aggregation` is similar to the `histogram aggregation` in that it can create buckets based on any numerical interval. The difference is that the `range aggregation` allows you to define intervals of varying sizes so you can customize it to your use case.  

For example, what if you wanted to know the number of transactions for items from varying price ranges(between 0 and $50, between $50-$200, and between $200 and up)? 

Syntax:
```http
GET Enter_name_of_the_index_here/_search
{
  "size": 0,
  "aggs": {
   "Name your aggregations here": {
      "range": {
        "field": "Name the field you want to aggregate on here",
        "ranges": [
          {
            "to": x
          },
          {
            "from": x,
            "to": y
          },
          {
            "from": z
          }
        ]
      }
    }
  }
}
```
Example: 
```http
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "transactions_per_custom_price_ranges": {
      "range": {
        "field": "UnitPrice",
        "ranges": [
          {
            "to": 50
          },
          {
            "from": 50,
            "to": 200
          },
          {
            "from": 200
          }
        ]
      }
    }
  }
}
```
Expected response from Elasticsearch:

Elasticsearch returns a buckets array where each bucket represents a customized price interval("key"). It also includes the number of documents("doc_count") placed in each bucket.

![image](https://user-images.githubusercontent.com/60980933/114792261-44f2d000-9d45-11eb-9298-6bae6dcf8f06.png)

**Bucket sorting for range aggregation**

The `range aggregation` is sorted based on the input ranges you specify and it cannot be sorted any other way! 

#### Terms Aggregation
The `terms aggregation` creates a new bucket for every unique term it encounters for the specified field. It is often used to find the most frequently found terms in a document. 

For example, let's say you want to identify 5 customers with the highest number of transactions(documents). 

Syntax:
```http
GET Enter_name_of_the_index_here/_search
{
  "aggs": {
    "Name your aggregations here": {
      "terms": {
        "field": "Name the field you want to aggregate on here",
        "size": State how many top results you want returned here
      }
    }
  }
}
```
Example:
```http
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "top_5_customers": {
      "terms": {
        "field": "CustomerID",
        "size": 5
      }
    }
  }
}
```
Expected response from Elasticsearch:

Elasticsearch will return 5 customer IDs("key") with the highest number of transactions("doc_count").  
![image](https://user-images.githubusercontent.com/60980933/114796514-6906df00-9d4e-11eb-862e-ac8eed4a10e2.png)

**Bucket sorting for terms aggregation**

By default, the `terms aggregation` sorts buckets based on the "doc_count"
values in descending order.  To reverse this order, you can add an order parameter to the aggregation. Then, specify that you want to sort buckets based on the `_count` values in ascending(asc) order! 

Example:
```http
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "5_customers_with_lowest_number_of_transactions": {
      "terms": {
        "field": "CustomerID",
        "size": 5,
        "order": {
          "_count": "asc"
        }
      }
    }
  }
}
```
Expected response from Elasticsearch:

You will see that the buckets are now sorted in ascending order of "doc_count", showing buckets with the lowest "doc_count" first. 

![image](https://user-images.githubusercontent.com/60980933/116301597-77e18e80-a75d-11eb-835a-2fda2883bc5e.png)

### Combined Aggregations
So far, we have ran `metric aggregations` or `bucket aggregations` to answer simple questions. 

There will be times when we will ask more complex questions that require running combinations of these aggregations. 

For example, let's say we wanted to know the sum of revenue per day. 

To get the answer, we need to first split our data into daily buckets(`date_histogram aggregation`). 

![image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3mcqa7qs9pi7bosopwjr.png)
 
Within each bucket, we need to perform `metric aggregations` to calculate the daily revenue.

![image](https://user-images.githubusercontent.com/60980933/136267632-7ef1e51b-56f6-4a94-b67d-a44eaabbe1bf.png)

#### Calculate the daily revenue
The combined `aggregations` request looks like the following:
```http
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "transactions_per_day": {
      "date_histogram": {
        "field": "InvoiceDate",
        "calendar_interval": "day"
      },
      "aggs": {
        "daily_revenue": {
          "sum": {
            "script": {
              "source": "doc['UnitPrice'].value * doc['Quantity'].value"
            }
          }
        }
      }
    }
  }
}
```

Expected Response from Elasticsearch:

Elasticsearch returns an array of daily buckets. 

Within each bucket, it shows the number of documents("doc_count") within each bucket as well as the revenue generated from each day("daily_revenue"). 

![image](https://user-images.githubusercontent.com/60980933/115085623-0df8f780-9ec8-11eb-81a5-a2da7d5759c1.png)

#### Calculating multiple metrics per bucket

You can also calculate multiple metrics per bucket. 

For example, let's say you wanted to calculate the daily revenue and the number of unique customers per day in one go. To do this, you can add multiple `metric aggregations` per bucket as shown below!  

Example: 
```http
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "transactions_per_day": {
      "date_histogram": {
        "field": "InvoiceDate",
        "calendar_interval": "day"
      },
      "aggs": {
        "daily_revenue": {
          "sum": {
            "script": {
              "source": "doc['UnitPrice'].value * doc['Quantity'].value"
            }
          }
        },
        "number_of_unique_customers_per_day": {
          "cardinality": {
            "field": "CustomerID"
          }
        }
      }
    }
  }
}
```
Expected Response from Elasticsearch:

Elasticsearch returns an array of daily buckets. 

Within each bucket, you will see that the "number_of_unique_customers_per_day" and the "daily_revenue" have been calculated for each day!

![image](https://user-images.githubusercontent.com/60980933/115086712-08041600-9eca-11eb-9afe-43923477b371.png)

**Sorting by metric value of a sub-aggregation**

You do not always need to sort by time interval, numerical interval, or by doc_count! You can also sort by metric value of `sub-aggregations`. 

Let's take a look at the request below. Within the `sub-aggregation`, metric values "daily_revenue" and "number_of_unique_customers_per_day" are calculated.  

Let's say you wanted to find which day had the highest daily revenue to date! 

All you need to do is to add the "order" parameter( and sort buckets based on the metric value of "daily_revenue" in descending("desc") order! 

Example:
```http
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "transactions_per_day": {
      "date_histogram": {
        "field": "InvoiceDate",
        "calendar_interval": "day",
        "order": {
          "daily_revenue": "desc"
        }
      },
      "aggs": {
        "daily_revenue": {
          "sum": {
            "script": {
              "source": "doc['UnitPrice'].value * doc['Quantity'].value"
            }
          }
        },
        "number_of_unique_customers_per_day": {
          "cardinality": {
            "field": "CustomerID"
          }
        }
      }
    }
  }
}
```

Expected response from Elasticsearch:

You will see that the response is no longer sorted by date. The buckets are now sorted to return the highest daily revenue first! 

![image](https://user-images.githubusercontent.com/60980933/116132063-5794cf80-a68a-11eb-9e85-f129054cacb2.png)

### Pipeline Aggregations
Pipeline aggregations allow you to perform aggregations on the output of other aggregations as compared to the output of document sets.

#### MovingFn Aggregations
MovingFn aggregations allow you to apply a script against the output of histogram or date histogram with a moving window.

For example if you wanted to get the average sum of unit prices over a sliding 3 month time period that included the next and previous month. We could do the following:
```http
POST ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "my_date_histo": {                  
      "date_histogram": {
        "field": "InvoiceDate",
        "calendar_interval": "1M"
      },
      "aggs": {
        "the_sum": {
          "sum": { "field": "UnitPrice" }   
        },
        "the_movfn": {
          "moving_fn": {
            "buckets_path": "the_sum",  
            "window": 3,
            "shift": 2,
            "script": "MovingFunctions.unweightedAvg(values)"
          }
        }
      }
    }
  }
}
```

The output would look like the following:

![movingFnOutput](images/movingFnOutput.png)

#### Cumulative Cardinality

This pipeline aggregation finds the cumulative cardinality of a parent aggregation. It is useful for finding things like total new items.

For Example to find the total number of new users after every month we can use the following:
```http
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "users_per_day": {
      "date_histogram": {
        "field": "InvoiceDate",
        "calendar_interval": "1M"
      },
      "aggs": {
        "distinct_users": {
          "cardinality": {
            "field": "CustomerID"
          }
        },
        "total_new_users": {
          "cumulative_cardinality": {
            "buckets_path": "distinct_users"
          }
        }
      }
    }
  }
}
```

This results in:

![cumulativeCardinalityOutput](images/cumulativeCardinality.png)

##### Incremental Cumulative Cardinality
Notice how the above only shows the total distinct users over time. If we wanted to get the number of new distinct users each month as compared to a running sum of new users over time we would need to add in a derivative.

For Example:

```http
GET ecommerce_data/_search
{
  "size": 0,
  "aggs": {
    "users_per_day": {
      "date_histogram": {
        "field": "InvoiceDate",
        "calendar_interval": "1M"
      },
      "aggs": {
        "distinct_users": {
          "cardinality": {
            "field": "CustomerID"
          }
        },
        "total_new_users": {
          "cumulative_cardinality": {
            "buckets_path": "distinct_users"
          }
        },
        "incremental_new_users": {
          "derivative": {
            "buckets_path": "total_new_users"
          }
        }
      }
    }
  }
}
```

This would output the following:

![incrementalCumulativeCardinality](images/incrementalCumulativeCardinality.png)


## Reference
[Beginner's Crash Course to Elastic Stack](https://github.com/LisaHJung/Beginners-Crash-Course-to-the-Elastic-Stack-Series): 



