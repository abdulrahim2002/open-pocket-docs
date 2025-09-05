---
title: "Authentication"
sidebar_position: 1
---

# Authentication Overview

The purpose of authentication is to verify the identity of a user.  In
our application, resources are primarily owned by `email addresses`. If
a user is successfully able to claim an email by providing correct
password. Then he/she can get unvetted access to all resources belonging
to that email address.  

## Registration

Registration is the simple process of inserting a new user into the
system. In `open-pocket`, the registration follows this simple
procedure:

- user makes a POST request to `/register` endpoint with name, email,
  password. See example request below:

```
POST /register

{
    "name": "test name",
    "email": "test@example.com",
    "password": "userpassworod"
}
```

- On receiving this request, the server:
    - tries to insert the user into postgres database
    -  if no error occurs, a (201) CREATED response is sent
- If the user already exists in the system, the above insert fails, and
  server responds with (409) CONFLICT. Ensuring that no duplicate
  records are inserted

## Login

At the `/login` endpoint, the user tries to proof, that they are the
owner of a particular email address. This can be done by 2 methods.

1. providing `email`, `password` in request body
2. you are already authorized because the server setted up an encrypted
   cookie saying `authenticated=true`
3. providing an authentication token in "Authorization" header

The primary method of authentication is through providing an `email` and
`password`. See below for an example request:

```
POST /login

{
    "email": "test@example.com",
    "password": "testpassword"
}
```

Upon receiving the request. The server does the following:

1. try to retrieve the user from the database
    - if user is not present or an error occurs -> return failure
    - note the hashed password of the given user
2. hash the password supplied in request and compare it with the
   password\_hash stored in the database
    - if they do not match -> return failure
3. the server sets an encrypted cookie (also called secure session),
   having the following information:
    - authenticated=true and `{ user_id }`
4. the server then generates a jwt token and a refresh token
    - the jwt token is created using `jwt.sign()` method
    - the refresh token is created using `crypto.randomBytes()`
    - in a server side key value store. It sets: `map[refresh_token] =
      {user_id}`.
    - The previously generated refresh token (if any) is deleted using:
      `user_refToken_map[user_id] -> (refresh_token)`. Then
      `delete map[refresh_token]`
5. both jwt token and refresh tokens are send back

## Refresh

The `/refresh` endpoint is responsible for generating a new set of
`(access_token, refresh_token)` using the old refresh token. Below is an
example request:

```
POST /refresh

{
    "refresh_token": "6b2d08fc152adbb12949ba67f9d8ebc579b4dfffd75b79c2b8f9d18cf12929fd"
}
```

The server does the following:

- check if:
    - `map[supplied_refresh_token] -> returns a { user_id }`
    - if not return unauthorized
- else, generate a new pair of jwt token and refresh token, using
  `jwt.sign()` and `crypto.randomBytes()` respectively.
- delete the old token using `delete map[supplied_refresh_token]`
- add new refresh token using: `map[refresh_token] = { user_id }`
- return the generated jwt token and refresh token 


## Userful links

1. [refresh token and access token(jwt)](https://www.codingshuttle.com/spring-boot-handbook/jwt-refresh-token-and-access-token/)

2. [IETF RFC 6749](https://datatracker.ietf.org/doc/html/rfc6749#section-1.5)

3. [IETF RFC 6759](https://datatracker.ietf.org/doc/html/rfc6750)

4. [Authentication best practices](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)

### TODOs

- refresh tokens should also have an expiration date
- keep the refresh token in a secure unencrypted cookie. For example a
  simple cookie with (secure=true, httponly, samesite=lax) 

<!--
final architecture decided upon


- the user will log in using email and password
- will be issued access token and refresh token

    - further details -> access token is jwt with payload = { email,
      user_id }
    - refresh token is randomly generated string using
      [crypto.randomBytes](https://nodejs.org/api/crypto.html#cryptorandombytessize-callback)

    - also with the refresh token. we store in redis ->
      `redis.set(refresh_token, {email, user_id})`

    - we simply set `authenticated=true` in a special encrypted cookie
      (which only the server can decrypt/ and create). This is called
using
[fastify-securesession](https://github.com/fastify/fastify-secure-session).
the life of this cookie = life of the access token

    - in BROWSER cleints -> set refresh token in `http only https secure
      (whatever security settings)` -> into plain cookiep

access of protected routes

- non browser clients:
    - privide a valid authorization header with jwt token.
    - you can access whatever resources ownser by paylod (user\_id,
      email) in verified jwt

- browser clients:
    - server just simply decrypts the secured encrypted cookie. And
      finds out that it previously gave this client
        `authentication=true` property. cool, you can access it any
resouces with given email/user\_id

OOPS the access token/session.authentication=true expired.

- Browser clients -> send their refresh token (available in their
  cookie) to the server.
- server deletes the old refresh token and generates new pair of access
  token and refresh token. oh and dont forget the set authenticated=true

- for non brosser clients. -> it is up to the client to store the
  refresh token somewhere securly.
- send the refresh token to get new access token and regresh token.
- authenticate subsequent requests



-- some deep seek nitpicks -> store hash of the refresh tokens so even
if server is compromised -> you are not doomed 

-- another: Just ensure:

    The cookie path is set to your auth endpoint (e.g., path:
'/api/auth') to avoid sending it with every request unnecessarily.


- the backend must check both for `authenticated=true` and
  `authorization: bearer token` authentication methods



this blog is very useful:
https://www.descope.com/blog/post/refresh-token-rotation


This idea is now dumped. This is because,  i do not want to handle the
problems myself and rather stand on the shoulders of giants. 

Hence, i will use passport. Using the local strategy

-->


