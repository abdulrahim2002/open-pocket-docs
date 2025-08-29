---
title: "Database Schema"
sidebar_position: 2
---

# Database schema

We need to keep in mind the [operations](/docs/category/api-spec) we
need to support.


[Andy](https://github.com/andyw8) described the [following
schema](https://github.com/open-pocket/open-pocket/issues/2) for open
pocket:

## Articles

| Attribute               | Data Type / Properties     | Constraints / Indexing                    |
|--------------------------|----------------------------|--------------------------------------------|
| item\_id                 | BIGINT, Primary Key        | NOT NULL, AUTO INCREMENT, UNIQUE           |
| user\_id                 | BIGINT, Foreign Key → Users.uid | Indexed, NOT NULL                     |
| status                  | SMALLINT                   |                                            |
| favorite                | BOOLEAN                    | DEFAULT false                              |
| resolved\_title          | TEXT                       |                                            |
| resolved\_url            | TEXT                       |                                            |
| excerpt                 | TEXT                       |                                            |
| is\_article              | BOOLEAN                    | DEFAULT false                              |
| is\_index                | BOOLEAN                    | DEFAULT false                              |
| has\_video               | BOOLEAN                    | DEFAULT false                              |
| has\_image               | BOOLEAN                    | DEFAULT false                              |
| word\_count              | INTEGER                    |                                            |
| time\_added              | TIMESTAMP                  | DEFAULT now()                              |
| time\_updated            | TIMESTAMP                  |                                            |
| time\_read               | TIMESTAMP                  |                                            |
| time\_favorited          | TIMESTAMP                  |                                            |
| top\_image\_url           | TEXT                       |                                            |
| author\_id               | BIGINT, Foreign Key → Authors.id | Indexed                                |
| time\_to\_read            | INTEGER, Calculated        | Generated from word\_count                  |
| listen\_duration\_estimate| INTEGER, Calculated        | Generated from word\_count                  |

---

## Users

| Attribute   | Data Type / Properties  | Constraints / Indexing            |
|-------------|--------------------------|------------------------------------|
| provider    | TEXT                     | NOT NULL                           |
| uid         | SERIAL, _Primary Key_    | NOT NULL, UNIQUE                   |
| email       | TEXT                     | UNIQUE, _Indexed_                  |
| name        | TEXT                     | NOT NULL                           |

---

## tags

| Attribute   | Data Type / Properties             | Constraints / Indexing            |
|-------------|-------------------------------------|------------------------------------|
| tag\_id      | SERIAL, Primary Key                     | NOT NULL, UNIQUE                  |
| user\_id     | INTEGER, Foreign Key → Users.user\_id    | NOT NULL                  |
| article\_id  | INTEGER, Foreign Key → Articles.item\_id | NOT NULL                |
| tag\_name    | TEXT                                    |  Indexed, NOT NULL        |

Further Indexes and Constraints:

- Index on `(user_id, article_id)`
- Index on `(user_id, tag_name)`
- `(tag_name, user_id, article_id)` are UNIQUE combined. Therefore, no
  duplicate tag is allowed

Notes:

- this table basically contains: an article with `tag=tag_name` on
  article with article\_id, belonging to user of user\_id
- the fields: (tag\_name, user\_id, article\_id) combined must be unique
- we will be required to handle queries of the form:

```sql
-- get all tags on an article with `article_id` and which belongs to
-- user with `user_id`
SELECT tag_name FROM tags WHERE article_id=123 AND user_id=123;
```

- therefore, it is beneficial to create an index on (article\_id, user\_id)

- here's another use case:

find all articles belonging to user with `user_id` and with given
`tag_name`:

```sql
SELECT article_id FROM tags WHERE user_id=123 AND tag_name="xyz";
```

- therefore it might be beneficial to make an index on (user\_id,
  tag\_name) columns

---

## Images

| Attribute   | Data Type / Properties             | Constraints / Indexing            |
|-------------|-------------------------------------|------------------------------------|
| image\_id    | BIGINT, Primary Key                | NOT NULL, AUTO INCREMENT, UNIQUE   |
| article\_id  | BIGINT, Foreign Key → Articles.item\_id | Indexed, NOT NULL                |
| src         | TEXT                               | NOT NULL                           |
| width       | INTEGER                            |                                    |
| height      | INTEGER                            |                                    |
| credit      | TEXT                               |                                    |
| caption     | TEXT                               |                                    |

---

## Videos

| Attribute   | Data Type / Properties             | Constraints / Indexing            |
|-------------|-------------------------------------|------------------------------------|
| video\_id    | BIGINT, Primary Key                | NOT NULL, AUTO INCREMENT, UNIQUE   |
| article\_id  | BIGINT, Foreign Key → Articles.item\_id | Indexed, NOT NULL                |
| src         | TEXT                               | NOT NULL                           |
| width       | INTEGER                            |                                    |
| height      | INTEGER                            |                                    |
| type        | VARCHAR(50)                        |                                    |
| vid         | VARCHAR(100)                       |                                    |

---

## Authors

| Attribute   | Data Type / Properties  | Constraints / Indexing            |
|-------------|--------------------------|------------------------------------|
| id          | BIGINT, Primary Key     | NOT NULL, AUTO INCREMENT, UNIQUE   |
| name        | VARCHAR(255)            | NOT NULL                           |
| url         | TEXT                     |                                    |

