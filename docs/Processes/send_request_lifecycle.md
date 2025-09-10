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


1.    item\_id – Unique identifier for the saved item (used in /v3/modify)
2.    resolved\_id – Unique ID for the resolved URL (0 = not processed yet)
3.    given\_url – The actual saved URL
4.    resolved\_url – Final resolved URL (e.g., unshortened link)
5.    given\_title – Title saved with the item
6.    resolved\_title – Title Pocket parsed from the URL
7.    favorite – 1 if favorited
8.    status – 0 = active, 1 = archived, 2 = deleted
9.    excerpt – First few lines of the item (articles only)
10.    is\_article – 1 if item is an article
11.    has\_image – 1 if it has images, 2 if it's an image
12.    has\_video – 1 if it has videos, 2 if it's a video
13.    word\_count – Number of words in the article
14.    tags – JSON object of user tags
15.    authors – JSON object of authors
16.    images – JSON object of images
17.    videos – JSON object of videos



Firtly we observe that fields 1-13 are available in the
[articles](https://abdulrahim2002.github.io/open-pocket-docs/docs/Database-Layer/database-schema/)
table. Therefore, fetching them for a user would be trivial. We can just
do something like:

```sql
SELECT * FROM articles WHERE user_id=user_id
```


