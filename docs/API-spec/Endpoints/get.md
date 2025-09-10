---
sidebar_position: 1
---

# get

## Retriving a user's pocket data

To retrieve item(s) from a user’s Pocket list, you’ll make a request to
the `/v3/get` endpoint.
It also allows you to add filters on top of retrieved items.

---

## Required Permissions

To use the `/v1/get` endpoint, your consumer key must have the
**"Retrieve"** permission.

---

### Method URL

```
https://getpocket.com/v3/get
```


### Required Parameters

| Parameter     | Type   | Description |
|---------------|--------|-------------|
| `consumer_key` | string | Your application's Consumer Key |
| `access_token` | string | The user's Pocket access token |

### Optional Parameters

| Parameter     | Type     | Description |
|---------------|----------|-------------|
| `state`       | string   | See below for valid values |
| `favorite`    | 0 or 1   | See below for valid values |
| `tag`         | string   | See below for valid values |
| `contentType` | string   | See below for valid values |
| `sort`        | string   | See below for valid values |
| `detailType`  | string   | See below for valid values |
| `search`      | string   | Only return items whose title or URL contain the search string |
| `domain`      | string   | Only return items from a particular domain |
| `since`       | timestamp| Only return items modified since the given unix timestamp |
| `count`       | integer  | Only return `count` number of items (limit: 30) |
| `offset`      | integer  | Used with `count`; start returning from offset position |
| `total`       | 0 or 1   | Use with offset and count to determine if there are more pages |

---

### Parameter Values

**state**
- `unread` = only return unread items  
- `archive` = only return archived items  
- `all` = return both unread and archived items (default)  

**favorite**
- `0` = only return un-favorited items  
- `1` = only return favorited items  

**tag**
- `tag_name` = only return items tagged with `tag_name`  
- `_untagged_` = only return untagged items  

**contentType**
- `article` = only return articles  
- `video` = only return videos or articles with embedded videos  
- `image` = only return images  

**sort**
- `newest` = order by newest to oldest  
- `oldest` = order by oldest to newest  
- `title` = order alphabetically by title  
- `site` = order alphabetically by URL  

**detailType**
- `simple` = return basic info (title, url, status, etc.)  
- `complete` = return all data (tags, images, authors, videos, etc.)  

---

## Example Request (JSON)

```http
POST /v3/get HTTP/1.1
Host: getpocket.com
Content-Type: application/json

{
  "consumer_key": "1234-abcd1234abcd1234abcd1234",
  "access_token": "5678defg-5678-defg-5678-defg56",
  "count": "10",
  "detailType": "complete"
}
```

## Example Response

```
HTTP/1.1 200 OK
Content-Type: application/json
Status: 200 OK

{
  "status": 1,
  "list": {
    "229279689": {
      "item_id": "229279689",
      "resolved_id": "229279689",
      "given_url": "http://www.grantland.com/blog/the-triangle/post/_/id/38347/ryder-cup-preview",
      "given_title": "The Massive Ryder Cup Preview - The Triangle Blog - Grantland",
      "favorite": "0",
      "status": "0",
      "resolved_title": "The Massive Ryder Cup Preview",
      "resolved_url": "http://www.grantland.com/blog/the-triangle/post/_/id/38347/ryder-cup-preview",
      "excerpt": "The list of things I love about the Ryder Cup is so long...",
      "is_article": "1",
      "has_video": "1",
      "has_image": "1",
      "word_count": "3197",
      "images": {
        "1": {
          "item_id": "229279689",
          "image_id": "1",
          "src": "http://a.espncdn.com/combiner/i?img=/photo/2012/0927/grant_g_ryder_cr_640.jpg&w=640&h=360",
          "credit": "Jamie Squire/Getty Images"
        }
      },
      "videos": {
        "1": {
          "item_id": "229279689",
          "video_id": "1",
          "src": "http://www.youtube.com/v/Er34PbFkVGk?version=3&hl=en_US&rel=0",
          "width": "420",
          "height": "315"
        }
      }
    }
  }
}
```

# Response Fields

- **item_id** – Unique identifier for the saved item (used in `/v3/modify`)  
- **resolved_id** – Unique ID for the resolved URL (`0` = not processed yet)  
- **given_url** – The actual saved URL  
- **resolved_url** – Final resolved URL (e.g., unshortened link)  
- **given_title** – Title saved with the item  
- **resolved_title** – Title Pocket parsed from the URL  
- **favorite** – `1` if favorited  
- **status** – `0 = active`, `1 = archived`, `2 = deleted`  
- **excerpt** – First few lines of the item (articles only)  
- **is_article** – `1` if item is an article  
- **has_image** – `1` if it has images, `2` if it's an image  
- **has_video** – `1` if it has videos, `2` if it's a video  
- **word_count** – Number of words in the article  
- **tags** – JSON object of user tags
- **authors** – JSON object of authors  
- **images** – JSON object of images  
- **videos** – JSON object of videos  

---

# Best Practices

### Retrieving User's List

- After retrieving a User's list, store the current time (provided in the response).  
- Pass that time in the next request using the `since` parameter.  
- This ensures only the changes since the last request are returned.  

---

# Pagination

- The maximum page size limit is **30 results**.  
- To retrieve more, use pagination with `count`, `offset`, and `total`.  

---
