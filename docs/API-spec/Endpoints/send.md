---
sidebar_position: 2
---

# send

## Pocket API: Modifying a User's Pocket Data

Make a single change or batch several changes to a user's pocket data.

```
/v1/send
```

## Required Permissions

In order to use the `/v3/send` endpoint, your consumer key must have the
"Modify" permission.

## Modifying a User's Pocket Data

The `/v3/send` endpoint allows your application to send a single event or multiple events and actions that will modify the user's data in one call.

### Method URL

```
https://getpocket.com/v3/send
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

```
https://getpocket.com/v3/send?actions=%5B%7B%22action%22%3A%22archive%22%2C%22time%22%3A1348853312%2C%22item_id%22%3A229279689%7D%5D&access_token=[ACCESS_TOKEN]&consumer_key=[CONSUMER_KEY]
``` 

The response you receive back contains a status variable and an
action\_results array that indicates which action had an issue if the
status is 0 (indicating failure):

```
HTTP/1.1 200 OK
Content-Type: application/json
Status: 200 OK

{"action_results":[true],"status":1}
```

## Error Handling

View the Error and Response Headers Documentation for detailed information on how to respond to errors.

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

Add a new item to the user's list.

Note: If you are only adding a single item, the `/v3/add` endpoint should be used.

#### JSON Array Parameters

| Name | Type | Optional | Description |
| :--- | :--- | :--- | :--- |
| item\_id | integer | | The id of the item to perform the action on. |
| ref\_id | integer | optional | A Twitter status id; this is used to show tweet attribution. |
| tags | string | optional | A comma-delimited list of one or more tags. |
| time | timestamp | optional | The time the action occurred. |
| title | string | optional | The title of the item. |
| url | string | optional | The url of the item; provide this only if you do not have an item\_id. |

---

### Action: archive

Move an item to the user's archive.

#### JSON Array Parameters

| Name | Type | Optional | Description |
| :--- | :--- | :--- | :--- |
| item\_id | integer | | The id of the item to perform the action on. |
| time | timestamp | optional | The time the action occurred. |

---

### Action: readd

Move an item from the user's archive back into their unread list.

#### JSON Array Parameters

| Name | Type | Optional | Description |
| :--- | :--- | :--- | :--- |
| item\_id | integer | | The id of the item to perform the action on. |
| time | timestamp | optional | The time the action occurred. |

---

### Action: favorite

Mark an item as a favorite.

#### JSON Array Parameters

| Name | Type | Optional | Description |
| :--- | :--- | :--- | :--- |
| item\_id | integer | | The id of the item to perform the action on. |
| time | timestamp | optional | The time the action occurred. |

---

### Action: unfavorite

Remove an item from the user's favorites.

#### JSON Array Parameters

| Name | Type | Optional | Description |
| :--- | :--- | :--- | :--- |
| item\_id | integer | | The id of the item to perform the action on. |
| time | timestamp | optional | The time the action occurred. |

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
