---
title: "Lifecycle of a send request"
sidebar_position: 3
---

# Lifecycle of a /send request

With the send endpoint, the user only needs to supply `consumer_key` and
`access_token`. And the app should automatically return all the items
that belong to that user. 

Optional filteres can be added, but for the sake of simplicity, we will
deal with them later.

1. consumer\_key
2. access\_token


Now we cannot do anything with `consumer_key` yet. 

Let us take a careful look at what are the response fields that need to
be returned.

From the database's perspective, only the following fields are

important:

1. url
2. title
3. tags
4. user\_id (generated through access token)

We need to perform the following operations:

1. add article into the `articles` table
2. add all the tags provided into the `tags` table (with item\_id and
   user\_id set)

Let us try to simulate a sample request. Firstly, we need a user. Let's
insert a sample one:

```sql
INSERT INTO users (provider, "name", "email", "hashed_password") 
VALUES ('ok','abdul1','abdul1@mail.com','112fewfi3n123oi34');
```

Now, we use the parameters we obtained from the `add` request along with
parser/backend processing, to retrieve all relevant fields we need to
store the `article` in `articles` table. We have the following data:

```sql
INSERT INTO articles (
    user_id, status, favorite, resolved_title, 
    resolved_url, excerpt, is_article, is_index, 
    has_video, has_image, word_count, top_image_url, 
    author_id
) 
VALUES (
    17, 0, false, 'Title 1', 'https://abdulrahim.space/', 
    'a little excerpt', true, false, 0, 0, 123, 
    'https://topimage.com/topimage.png', 1
);
```

Suppose we have 2 tags for this article: `fantasy`, `novel`

We then insert these into the tags table.

```sql
INSERT INTO tags (user_id, item_id, tag_name) VALUES (17, 8, 'fantasy');
INSERT INTO tags (user_id, item_id, tag_name) VALUES (17, 8, 'horror');
```

And we are done. Now we have all the data we need.


---


### Retrieval:

Now let's imagine what can we do with this data. The most simple use
case is, return few articles that belong to a particular user, to show
it on the homepage. The tags should also appear with each article. This
can be done simply by the following query:


```sql
SELECT a.item_id, a.user_id, a.resolved_url, ARRAY_AGG(t.tag_name) 
FROM    articles a 
            LEFT JOIN 
        tags t 
ON  a.item_id  = t.item_id 
        AND 
    a.user_id = t.user_id 
WHERE a.user_id=17
GROUP BY a.item_id, a.user_id;
```

Which provides us with all the articles belonging to `user_id=17`. And
aggregating their tags.

Hence, a setup like this can easily be used to get articles to show on
the home page of the user.

```sql
 item_id | user_id |       resolved_url        |    array_agg     
---------+---------+---------------------------+------------------
       7 |      17 |                           | {NULL}
       8 |      17 | https://abdulrahim.space/ | {fantasy,horror}
(2 rows)
```


