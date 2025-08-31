---
title: "Rate Limiting"
sidebar_position: 1
---

# Rate Limits

The Pocket API has two separate rate limits. These dictate how many
calls can be made to the server within a given time.  Enforcing rate
limits prevents a single app or user from overwhelming the server.  

The response codes will tell you if you've hit your limit.  Your
application should be looking for these and if it encounters a rate
limit status code, it should back off until it hits the reset time.
Ignoring these codes may cause your access to be disabled.

---

### User Limit

Each user is limited to **320 calls per hour**.  This should be very
sufficient for most users as the average user only makes changes to
their list periodically.  

To ensure the user stays within this limit, make use of the **send method** for batching requests.

---

### Consumer Key Limit

Each application is limited to **10,000 calls per hour**.  

---

### Response Headers

The Pocket API responses include custom headers that provide information
about the current status of rate limiting for both the current user and
consumer key:


```
X-Limit-User-Limit: Current rate limit enforced per user
X-Limit-User-Remaining: Number of calls remaining before hitting user's rate limit
X-Limit-User-Reset: Seconds until user's rate limit resets
X-Limit-Key-Limit: Current rate limit enforced per consumer key
X-Limit-Key-Remaining: Number of calls remaining before hitting consumer key's rate limit
X-Limit-Key-Reset: Seconds until consumer key rate limit resets
```
