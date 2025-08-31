---
title: "Response, Status and Error codes"
sidebar_position: 1
---

# Response, Status, and Error Codes

Each call to the API will return a number of informational headers.
These headers will include information on the result of the request,
errors, and rate limits.

---

### Example Response Headers (for an Error):

```
Status: 400 Bad Request
X-Error: Missing API Key
X-Error-Code: 132
X-Limit-User-Limit: 100
X-Limit-User-Remaining: 43
X-Limit-User-Reset: 25
X-Limit-Key-Limit: 5000
X-Limit-Key-Remaining: 3520
X-Limit-Key-Reset: 25
```

---

### Error Messages

If there was an error, the `X-Error` header in the HTTP response will
include a description of the problem.  In most cases, it is a best
practice to display this message to the user.  

We also include an `X-Error-Code` — this numeric code can be helpful
when communicating with Pocket support about an error condition.

---

### Status Codes

As described above, the `X-Error` message will describe exactly what
went wrong.  However, the HTTP status code will provide a basic idea of
what the problem was:

- **200** – Request was successful  
- **400** – Invalid request, please make sure you follow the
  documentation for proper syntax  
- **401** – Problem authenticating the user  
- **403** – User was authenticated, but access denied due to lack of
  permission or rate limiting  
- **503** – Pocket's sync server is down for scheduled maintenance  

---

### Rate Limits

Please view the **Rate Limit Documentation** for a detailed look at rate
limits and rate limit error responses.


