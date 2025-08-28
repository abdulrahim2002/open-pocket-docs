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

## Tagging

| Attribute   | Data Type / Properties             | Constraints / Indexing            |
|-------------|-------------------------------------|------------------------------------|
| user\_id     | BIGINT, Foreign Key → Users.uid     | Indexed, NOT NULL                  |
| article\_id  | BIGINT, Foreign Key → Articles.item\_id | Indexed, NOT NULL                |
| tag         | VARCHAR(100)                        |                                    |

_Primary Key_ = `(user_id, article_id)`


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

