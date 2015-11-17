# Ingestion-API

This document provides an overview of Notify.io’s API integration.
#1. Quick Overview
Integration with our platform has two components:

An ingestion endpoint that you use to send us information about your items (such as your videos, articles, or products). This is the universe of items from which we recommend.

Ingested items are used both as a source for recommendations and as a source of metadata when associating user activity with an item.  

Native SDKs that you use to send us user activity and that we use to deliver personalized contextual notifications.  We use activity data (including screen views, engagement, conversion, and tap activity) to build models of user behavior.

All our endpoints require secure HTTPS connections.
#2. Ingestion Endpoint
##2.1. Ingestion Format

Our inventory endpoint is:
```
https://a.glanceit.io/1.0/content/ingest
```

You must add an the following header to all calls to this endpoint:
```
x-apikey: [your api key]
```

You can send inventory to this endpoint in the following format:
```
{
    "app_group_id":"your app group id",
    "api_key":"[your api key]",
    "items":[
        {
            "unique_id":"unique id for this item",
            "expiry":number of days from ingestion we should keep this item,
            "meta-key1":"value for this key",
            "meta-key2":"value for this key",
            "meta-key3":["some value", "other value"]
        },
        {
            "expiry":number of days from ingestion we should keep this item,
            "should_crawl":true,
            "url":"url for this item"
        }
    ]
}
```
You can use a single event to send more than one inventory item.

##2.2. Your Unique Item ID For Ingested Items
Each item we ingest requires a unique identifier (unique_id).  We match on this field when updating items and for associating user activity with items, which is then used for personalizing notifications.  

So, whenever we get user activity with an item_id from the SDK, we will match the item_id to this unique_id. 

If your item has a unique permanent URL, we generally recommend passing that as the unique_id. 

If you don’t supply a unique_id, we’ll create one for you and return it in the response.  For calls with multiple items, ids will be returned in the same order the items were received.  

##2.3 Supplied Tags (Fields)
Your supplied tags (Fields) are used to determine the relevancy of an item to a given user.  
Tags supplied with ingested content are used to determine the relevancy of an item to a given user.  

You should supply, but not be limited to, the following:
* Keywords
* Authors
* Images
* Interest tags
* Prices
* Publish dates
* Titles
* URL

We leverage this information to present the optimal combination of content to each individual user.
##2.4. Fields Allowed For Inventory Item Tags
We allow three kinds of fields:
String
List of strings
Number

We do not currently support nested JSON objects. Such fields will be ignored by us.

Here is a sample inventory item:
```
{
  "title" : "Star Wars",
  "url" : "http://www.somesite.com/title/star_wars.html",
  "thumbnail" : "http://cdn.somesite.com/star_wars-small.jpg",
  "description" : "After rescuing Han Solo from the palace of Jabba the Hutt, the rebels attempt to destroy the second Death Star.",
  "category" : ["action", "adventure", "fantasy"],
  "director" : "Richard Marquand",
  "rating" : 8.4,
  "likes" : 234
}
```

The overall JSON that you’ll send to our inventory endpoint to add this item will be:
```
{
  "app_group_id":"demoapp_bbc",
  "api_key" : "your api key here",
  "items" : [
    {
      "title" : "Star Wars",
      "url" : "http://www.somesite.com/title/star_wars.html",
      "thumbnail" : "http://cdn.somesite.com/star_wars-small.jpg",
      "description" : "After rescuing Han Solo from the palace of Jabba the Hutt, the   rebels attempt to destroy the second Death Star.",
      "category" : ["action", "adventure", "fantasy"],
      "director" : "Richard Marquand",
      "rating" : 8.4,
      "likes" : 234
    }
  ]
}
```
##2.5 Multiple Items in a single request  
You can also send multiple inventory items in a single request as a list under items in the JSON.

Like this:
```
{
  "app_group_id":"demoapp_bbc",
  "api_key" : "your api key here",
  "items" : [
    {
      "title" : "Star Wars",
      "url" : "http://www.somesite.com/title/star_wars.html",
      "thumbnail" : "http://cdn.somesite.com/star_wars-small.jpg",
      "description" : "After rescuing Han Solo from the palace of Jabba the Hutt, the   rebels attempt to destroy the second Death Star.",
      "category" : ["action", "adventure", "fantasy"],
      "director" : "Richard Marquand",
      "rating" : 8.4,
      "likes" : 234
    }, 
    {
      "title" : "Star Wars: Episode V - The Empire Strikes Back",
      "url" : "http://www.somesite.com/title/the_empire_strike_back.html",
      "thumbnail" : "http://cdn.somesite.com/the_empire_strike_back.jpg",
      "description" : "After the rebels have been brutally overpowered by the Empire on their newly established base, Luke Skywalker takes advanced Jedi training with Master Yoda.",
      "category" : ["action", "adventure", "fantasy"],
      "director" : "Richard Marquand",
      "rating" : 8.8,
      "likes" : 534
    }
  ]
}
```

Below is the curl command for this insertion:

```
curl -H "Content-Type: application/json" -H "x-apikey: [your api key]" -d '{
  "app_group_id":"demoapp_bbc",
  "api_key" : "your api key here",
  "items" : [
    {
      "title" : "Star Wars",
      "url" : "http://www.somesite.com/title/star_wars.html",
      "thumbnail" : "http://cdn.somesite.com/star_wars-small.jpg",
      "description" : "After rescuing Han Solo from the palace of Jabba the Hutt, the   rebels attempt to destroy the second Death Star.",
      "category" : ["action", "adventure", "fantasy"],
      "director" : "Richard Marquand",
      "rating" : 8.4,
      "likes" : 234
    }, 
    {
      "unique_id" : "http://www.somesite.com/title/the_empire_strike_back.html",
      "title" : "Star Wars: Episode V - The Empire Strikes Back",
      "url" : "http://www.somesite.com/title/the_empire_strike_back.html",
      "thumbnail" : "http://cdn.somesite.com/the_empire_strike_back.jpg",
      "description" : "After the rebels have been brutally overpowered by the Empire on their newly established base, Luke Skywalker takes advanced Jedi training with Master Yoda.",
      "category" : ["action", "adventure", "fantasy"],
      "director" : "Richard Marquand",
      "rating" : 8.8,
      "likes" : 534
    }
  ]
}' https://a.glanceit.io/1.0/content/ingest
```

We will respond with:
```
{
  "result": [
    {
      "item_position": 1,
      "success": true,
      "unique_id": "ee5f7c04-daeb-41ca-95eb-00f961afd017"
    },
    {
      "item_position": 2,
      "success": true,
      "unique_id": "http://www.somesite.com/title/the_empire_strike_back.html"
    }
  ],
  "success": true
}
```

##2.6. Crawler
If your item has a URL, we can crawl it and augment the item with additional meta-keys.  The crawler will extract keywords along with the most common meta-tags including open-graph (OG) and twitter tags.  By default the crawled data will replace values in existing meta-keys.  If you would like to have different behavior please contact us.  

##2.7. Item expiration
By default items will not be considered for recommendations after 7 days.  You can increase and decrease the expiration time for the item, but passing a float representing days to the expiry key.    

##2.8. Update operation 
You can update items to add new meta-keys or replace values in existing meta-keys.  Updates use the same request as inserts.  If the unique_id you send us exists, we treat it as an update.  

Note: During updates, we do not merge values.  All the old values in a meta-key will be replaced by the new ones.  

##2.9. Time Lag With Inventory Operations
The ingestion operations you do may not occur immediately. Please expect a total delay of up to ten minutes between your operations and having them affect recommendations.

Additionally, crawling can be delayed for up to two hours.  

We also don’t guarantee consistency in the order of execution of operations, so if an insert and update operation happen very close together in time (within an interval of under a minute), the result is unpredictable.

##2.10. Other Ingestion Options
In addition to our ingestion API, we can ingest items via RSS style feeds. Please contact us to discuss RSS or other ingestion options.  

Since RSS feeds are a common XML feed that most sites already have, it’s easy to use your existing RSS feed as your data feed with little development effort.
