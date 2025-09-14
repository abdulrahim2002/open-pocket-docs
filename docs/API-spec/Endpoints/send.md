---
sidebar_position: 2
---

# send

The send endpoint allows you to make a single change or batch several changes
to a user's pocket data. You can send these changes in so called "actions".
Each action performs a specific change to the state of user's pocket data.  You
can send multiple of these actions in a single call. Some of the operations may
fail. This is indicated by an `action_results` array that contain true/false
for each action performed in order in which they were sent. 

### Method URL

```
/v1/send
```

### Parameters

| Name | Type | Description |
| :--- | :--- | :--- |
| consumer\_key | string | Your application's Consumer Key |
| access\_token | string | The user's Pocket access token |
| actions | array | JSON array of actions. See below for details. |

### JSON Array Format

You can send one action or you can send dozens. The list of actions
should be sent as a JSON array.

Here's an example of a JSON array calling the "archive" action:

```
[
    {
        "action" : "archive",
        "item_id" : "229279689",
        "time"     : "1348853312"
    }
]
```

Here's how you send the encoded JSON array as part of the `/v3/send` call:

:::warning
sending data in URL parameters is not supported.
:::

If all actions were successful, you receive back a response with `status=1`
along with `action_results` array with all values set to true. Otherwise, if
some of the actions failed, `satus=0` and `aaction_results` array has atlease
one false value.

```
HTTP/1.1 200 OK
Content-Type: application/json
Status: 200 OK

{
    "status": 1,
    "action_results": [true]
}
```

:::warning
status is 0 for failure and 1 for success
:::

## Error Handling

View the [Error and Response](/docs/category/error-handling) Documentation for
detailed information on how to respond to errors.

---

## List of Actions

### Basic Actions
* add - Add a new item to the user's list
* archive - Move an item to the user's archive
* readd - Re-add (unarchive) an item to the user's list
* favorite - Mark an item as a favorite
* unfavorite - Remove an item from the user's favorites
* delete - Permanently remove an item from the user's account

### Tagging Actions
* tags\_add - Add one or more tags to an item
* tags\_remove - Remove one or more tags from an item
* tags\_replace - Replace all of the tags for an item with one or more provided tags
* tags\_clear - Remove all tags from an item
* tag\_rename - Rename a tag; this affects all items with this tag
* tag\_delete - Delete a tag; this affects all items with this tag

---

### Action: add

:::note
If you are only adding a single item, the `/v1/add` endpoint should be used instead
:::

Adds a new item to the user's list.


#### JSON Array Parameters

| Name                         | Type       | Optional  | Description                                   |
| :---                         | :---       | :---      | :---                                          |
| url                          | string     |           | The url of the item;                          |
| tags                         | string     | optional  | A comma-delimited list of one or more tags.   |
| title                        | string     | optional  | User supplied title of the item. This takes precedence over automatically found title |
| <s>item_id</s>               | integer    | optional  | The id of the item to perform the action on.  |
| <s>ref\_id </s>              | integer    | optional  | A Twitter status id; this is used to show tweet attribution. |
| <s>time</s>                  | timestamp  | optional  | The time the action occurred.                 |

:::warning
<s>striked</s> items are  **depreciated**
:::

---

### Action: archive

Move an item to the user's archive.

#### JSON Array Parameters

| Name          | Type      | Optional    | Description   |
| :---          | :---      | :---        | :---          |
| item\_id      | string    |             | The id of the item to perform the action on. |
| <s>time</s>   | timestamp | optional    | The time the action occurred. |

:::warning
item\_id is no longer integer. It shall be supplied as a string.
:::

---

### Action: readd

Unarchive an item.

#### JSON Array Parameters

| Name        | Type      | Optional  | Description                   |
| :---        | :---      | :---      | :---                          |
| item\_id    | string    |           | The id of the item to perform the action on. |
| <s>time</s> | timestamp | optional  | The time the action occurred. |

:::warning
item\_id must be string, not integer.
:::

---

### Action: favorite

Mark an item as a favorite.

#### JSON Array Parameters

| Name          | Type      | Optional  | Description                   |
| :---          | :---      | :---      | :---                          |
| item\_id      | string |  | The id of the item to perform the action on. |
| <s>time</s>   | timestamp | optional  | The time the action occurred. |

---

### Action: unfavorite

Remove an item from the user's favorites.

#### JSON Array Parameters

| Name        | Type      | Optional    | Description                   |
| :---        | :---      | :---        | :---                          |
| item\_id    | string    |             | The id of the item to perform the action on. |
| <s>time</s> | timestamp | optional    | The time the action occurred. |

---

### Action: delete

Permanently remove an item from the user's account.

#### JSON Array Parameters

| Name | Type | Optional | Description |
| :--- | :--- | :--- | :--- |
| item\_id | integer | | The id of the item to perform the action on. |
| time | timestamp | optional | The time the action occurred. |

---

### Action: tags\_add

Add one or more tags to an item.

#### JSON Array Parameters

| Name | Type | Optional | Description |
| :--- | :--- | :--- | :--- |
| item\_id | integer | | The id of the item to perform the action on. |
| tags | string | | A comma-delimited list of one or more tags. |
| time | timestamp | optional | The time the action occurred. |

---

### Action: tags\_remove

Remove one or more tags from an item.

#### JSON Array Parameters

| Name | Type | Optional | Description |
| :--- | :--- | :--- | :--- |
| item\_id | integer | | The id of the item to perform the action on. |
| tags | string | | A comma-delimited list of one or more tags to remove. |
| time | timestamp | optional | The time the action occurred. |

---

### Action: tags\_replace

Replace all of the tags for an item with the one or more provided tags.

#### JSON Array Parameters

| Name | Type | Optional | Description |
| :--- | :--- | :--- | :--- |
| item\_id | integer | | The id of the item to perform the action on. |
| tags | string | | A comma-delimited list of one or more tags to add. |
| time | timestamp | optional | The time the action occurred. |

---

### Action: tags\_clear

Remove all tags from an item.

#### JSON Array Parameters

| Name | Type | Optional | Description |
| :--- | :--- | :--- | :--- |
| item\_id | integer | | The id of the item to perform the action on. |
| time | timestamp | optional | The time the action occurred. |

---

### Action: tag\_rename

Rename a tag. This affects all items with this tag.

#### JSON Array Parameters

| Name | Type | Optional | Description |
| :--- | :--- | :--- | :--- |
| old\_tag | string | | The tag name that will be replaced. |
| new\_tag | string | | The new tag name that will be added. |
| time | timestamp | optional | The time the action occurred. |

---

### Action: tag\_delete

Delete a tag. This affects all items with this tag.

#### JSON Array Parameters

| Name | Type | Optional | Description |
| :--- | :--- | :--- | :--- |
| tag | string | | The tag name that will be deleted. |
| time | timestamp | optional | The time the action occurred.
