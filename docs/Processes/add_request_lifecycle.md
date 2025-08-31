---
title: "add request lifecycle"
sidebar_position: 2
---

# add endpoint lifecycle

The add endpoint is used to insert a single item for a user. For
example, adding an article to a users's pocket record

As described in the [API-spec](/docs/API-spec/Endpoints/add), it
requires the following fields:

1. url
2. title
3. tags (comma seperated list of tags)
4. twitter\_id
5. consumer\_key
6. access\_token

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
