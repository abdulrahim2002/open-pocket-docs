---
title: "Database Schema"
sidebar_position: 2
---

# Database schema

All schemas need to ensure that we support required
[operations](/docs/category/api-spec).


## articles

| Attribute               | Data Type / Properties      | Constraints / Indexing                     |
|--------------------------|----------------------------|--------------------------------------------|
| item\_id                 | BIGINT, Primary Key        | NOT NULL, AUTO INCREMENT, UNIQUE           |
| user\_id                 | BIGINT, Foreign Key → users.user\_id | Indexed, NOT NULL                |
| status                   | SMALLINT                   |                                            |
| favorite                 | BOOLEAN                    | DEFAULT false                              |
| given\_url               | TEXT                       | NOT NULL                                   |
| given\_title             | TEXT                       |                                            |
| resolved\_title          | TEXT                       |                                            |
| resolved\_url            | TEXT                       |                                            |
| excerpt                  | TEXT                       |                                            |
| is\_article              | BOOLEAN                    | DEFAULT false                              |
| is\_index                | BOOLEAN                    | DEFAULT false                              |
| has\_video               | SMALLINT                   | DEFAULT false                              |
| has\_image               | SMALLINT                   | DEFAULT false                              |
| word\_count              | INTEGER                    |                                            |
| time\_added              | TIMESTAMP                  | Indexed, DEFAULT now()                     |
| time\_updated            | TIMESTAMP                  |                                            |
| time\_favorited          | TIMESTAMP                  |                                            |
| top\_image\_url          | TEXT                       |                                            |
| author\_id               | BIGINT, Foreign Key → authors.author\_id | Indexed                      |

### Description of the fields

1. `item_id`: Unique identifier for the saved item 
2. `user_id`: id of the user to which this article/item belongs
3. `status`:  0 = active, 1 = archived, 2 = deleted
4. `favorite`: weather article is favorited by the user. 1=favorited,
   0=not favorited
5. `resolved_title`: title, found by the parser/backend
6. `resolved_url`: final URL resolved to by the parser/backend. For
   example, by following redirects
7. `excerpt`: excerpt of the article found by the parser/backend
8. `is_article`: weather the item is an article
9. `is_index`: true if parser/backedn thinks this is an index page
    - 1=parser thinks this item is an index page
    - 0=otherwise
10. `has_video`:  
    - 0=no video; 
    - 1=has a video in the body of the article; 
    - 2=is a video
11. `has_image`: 
    - 0=no image; 
    - 1=has an image in the body of the article; 
    - 2=is an image
12. `word_count`: number of words in the article
13. `time_added`: timestamp (with timezone info), at which the article was added
14. `time_updated`: timestamp (with timezone info), at which the article was 
    last updated (like last read/opened)
15. `time_favorited`: time at which the article was favorited
16. `top_image_url`: URL of the first found image in html page
17. `author_id`: id of the author of the article

#### Details:

- this is the main and most important table, which stores information
  about the stored links.
- we anticipate queries of the form

```sql
-- find all articles that belong to user with user_id -> 123
SELECT * FROM articles WHERE user_id = 123 ORDER BY time_added
```

- or we might need articles that were added after "1 january 2024";

```sql
SELECT * FROM articles WHERE user_id = 123 AND time_added > '2024-01-01';
```

- we may also need to filter by `status` and `favorite`

---

## users

| Attribute   | Data Type / Properties  | Constraints / Indexing            |
|-------------|--------------------------|------------------------------------|
| provider    | TEXT                     | NOT NULL                           |
| user\_id    | SERIAL, _Primary Key_    | NOT NULL, UNIQUE                   |
| email       | TEXT                     | UNIQUE, _Indexed_                  |
| hashed\_password | TEXT                |                                    |
| name        | TEXT                     | NOT NULL                           |

### details:

#### login and signup

We should be able to support both traditional `email-password` based
logins as well as `o-auth` based logins/signups.

1. traditional login and signup

When the user signs up, firstly we ask for the following information:

```
name | email | password 
```

After, we have this information, then we use something like:

```sql
-- insert the newly created user into the system
INSERT INTO users (provider, email, hashed_password, name) 
VALUES ('open-pocket', 'xyz@mail.com, '12d8287g1', 'xyz');
```

Note that `open-pocket` is the provider here, since, the user is
registering through our platform. And `user_id` will be generated
autormatically.

Password hashing should be done in the application using
[`bcrypt`](https://www.npmjs.com/package/bcrypt).


Now when the user tries to login.

We ask for the following information:

```
email | password
```

Then we do something like:

```sql
-- find the hashed_password for user with email=email
SELECT hashed_password FROM users WHERE email=[supplied_email]
```

If this query executes successfully, but returns no rows. Then, the user
is not present in the system (not registered). Otherwise, the
`hashed_password` must match `hash_of_supplied_password` for successfull
login.

2. O-auth  login and signup

O-auth logins will be supported in the future, once we are done with
`email-password` logins. The `provider` field will store the O-auth
provider's name (e.g. google, github etc.). And additional fields will
be added as required.


---

## tags

| Attribute   | Data Type / Properties                        | Constraints / Indexing | 
|-------------|-----------------------------------------------|------------------------|
| tag\_id      | SERIAL, Primary Key                          | NOT NULL, UNIQUE       | 
| user\_id     | INTEGER, Foreign Key → Users.user\_id        | NOT NULL               | 
| item\_id     | BIG INTEGER, Foreign Key → Articles.item\_id | NOT NULL               | 
| tag\_name    | TEXT                                         | Indexed, NOT NULL     | 

Further Indexes and Constraints:

- Index on `(user_id, item_id)`
- Index on `(user_id, tag_name)`
- `(tag_name, user_id, item_id)` are UNIQUE combined. Therefore, no
  duplicate tag is allowed

Notes:

- this table basically contains: an article with `tag=tag_name` on
  article with item\_id, belonging to user of user\_id
- the fields: (tag\_name, user\_id, item\_id) combined must be unique
- we will be required to handle queries of the form:

```sql
-- get all tags on an article with `item_id` and which belongs to
-- user with `user_id`
SELECT tag_name FROM tags WHERE item_id=123 AND user_id=123;
```

- therefore, it is beneficial to create an index on (item\_id, user\_id)

- here's another use case:

find all articles belonging to user with `user_id` and with given
`tag_name`:

```sql
SELECT item_id FROM tags WHERE user_id=123 AND tag_name="xyz";
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

## authors

| Attribute   | Data Type / Properties  | Constraints / Indexing            |
|-------------|--------------------------|------------------------------------|
| id          | BIGINT, Primary Key     | NOT NULL, AUTO INCREMENT, UNIQUE   |
| name        | VARCHAR(255)            | NOT NULL                           |
| url         | TEXT                     |                                    |

