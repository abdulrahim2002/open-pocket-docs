---
sidebar_position: 3
---

# add

# Pocket API: Adding Items to Pocket

Allowing users to add articles, videos, images and URLs to Pocket is
most likely the first type of integration that you’ll want to build into
your application. Adding items to Pocket is easy.

---

## Required Permissions

Your consumer key must have the **Add** permission.


:::warning
Consumer keys are not supported yet.

---

## Adding a Single Item

To save an item to a user’s Pocket list, you’ll make a single request to
the **/v3/add** endpoint.  

### Method URL

https://getpocket.com/v3/add  

### Parameters

- **url** – string – The URL of the item you want to save  
- **title** – string – *optional* – This can be included for cases where
  an item does not have a title, which is typical for image or PDF URLs.
  If Pocket detects a title from the content of the page, this parameter
  will be ignored.  
- **tags** – string – *optional* – A comma-separated list of tags to
  apply to the item  
- **tweet\_id** – string – *optional* – If you are adding Pocket support
  to a Twitter client, please send along a reference to the tweet status
  id. This allows Pocket to show the original tweet alongside the article.  
- **consumer\_key** – string – Your application's Consumer Key  
- **access\_token** – string – The user's Pocket access token  

---

### Example Request (JSON)

```
POST /v3/add HTTP/1.1  
Host: getpocket.com  
Content-Type: application/json; charset=UTF-8  
X-Accept: application/json  
```

```json
{
  "url": "http://pocket.co/s8Kga",
  "title": "iTeaching: The New Pedagogy (How the iPad is Inspiring Better Ways of Teaching)",
  "time": 1346976937,
  "consumer_key": "1234-abcd1234abcd1234abcd1234",
  "access_token": "5678defg-5678-defg-5678-defg56"
}
```

Example Response (JSON)

```
HTTP/1.1 200 OK
Content-Type: application/json
Status: 200 OK
```

The `item` array in the response contains all of the meta information we
have resolved about the saved item.

## Response Fields

- **item\_id** – A unique identifier for the added item
- **normal\_url** – The original url for the added item
- **resolved\_id** – A unique identifier for the resolved item
- **resolved\_url** – The resolved url for the added item. The easiest
  way to think about the resolved\_url – if you add a bit.ly link, the
resolved\_url will be the url of the page the bit.ly link points to
- **domain\_id** – A unique identifier for the domain of the resolved\_url
- **origin\_domain\_id** – A unique identifier for the domain of the
  normal\_url
- **response\_code** – The response code received by the Pocket parser
  when it tried to access the item
- **mime\_type** – The MIME type returned by the item
- **content\_length** – The content length of the item
- **encoding** – The encoding of the item
- **date\_resolved** – The date the item was resolved
- **date\_published** – The date the item was published (if the parser
  was able to find one)
- **title** – The title of the resolved\_url
- **excerpt** – The excerpt of the resolved\_url
- **word\_count** – For an article, the number of words
- **has\_image** – `0`: no image; `1`: has an image in the body of the
  article; `2`: is an image
- **has\_video** – `0`: no video; `1`: has a video in the body of the
  article; `2`: is a video
- **is\_index** – `0` or `1`; If the parser thinks this item is an index
  page it will be set to `1`
- **is\_article** – `0` or `1`; If the parser thinks this item is an
  article it will be set to `1`
- **authors** – Array of author data (if author(s) were found)
- **images** – Array of image data (if image(s) were found)
- **videos** – Array of video data (if video(s) were found)

## Best Practices

Be sure to url-encode the parameters you are sending. Otherwise if your
url or title have characters like `?` or `&`, they will often break the
request.

## Batch Adding

If you have a need to add several items at once or want to perform other
actions on a user’s list (like archive or favorite), please see the v3
API Modify endpoint.

## Error Handling

View the [Error and Response Headers Documentation](#) for detailed
information on how to respond to errors.
