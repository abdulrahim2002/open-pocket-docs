---
slug: why-secure-cookies-can-pass-through-load-balancer
title: Why secure cookies can pass through load balancer
authors: "openpocket"
tags: [OpenPocket]
---


## how encrypted cookies are forwarded from load balancer to the client

with the proper configuration, cookies marked with Secure=true can be
successfully transmitted and processed by your backend instances even
when SSL is terminated at the load balancer.  How This Works

The key is that the Secure flag is about the client's connection, not
the backend connection:

    Browser to Load Balancer: The client's browser only sends the Secure
cookie over HTTPS to your load balancer

    Load Balancer to Backend: The load balancer terminates SSL, then
forwards the request (including the cookie) to your backend over HTTP

    Backend Processing: Your Fastify application, configured with
trustProxy: true, recognizes that the original request was secure (via
the X-Forwarded-Proto: https header) and properly handles the secure
cookie

Visualizing the Flow

```
Client (HTTPS) → Load Balancer (SSL termination) → Backend (HTTP)
    │                       │                         │
    │-- Secure cookie ----->│                         │
    │                       │-- Same cookie --------->│
    │                       │ (with X-Forwarded-Proto)│
    │                       │                         │<- Fastify sees 
    │                       │                         │   X-Forwarded-Proto: https
    │                       │                         │   and treats cookie as secure

```
Why This Is Secure

    The browser only transmits the cookie over HTTPS to your load balancer

    The internal network between your load balancer and backend is typically trusted and secured

    Your backend validates that the original request was secure using the X-Forwarded-Proto header








Step 1: The Client Makes a Secure Request

    The user's browser makes a request to https://yourapp.com.

    Because the cookie is set with Secure=true, the browser's internal logic says: "Only attach this cookie to requests that use the https:// protocol."

    The browser sends a request that includes the Cookie header to your load balancer over a secure HTTPS connection.

Request Headers (to Load Balancer):
text

GET / HTTP/1.1
Host: yourapp.com
Cookie: your_secure_session_cookie=encrypted_data_here
...other headers...

Step 2: The Load Balancer Terminates SSL

    The load balancer receives the encrypted HTTPS request.

    It performs SSL/TLS termination: it decrypts the request to inspect and process it (e.g., for routing, health checks, etc.).

    Crucially, the decrypted HTTP request still contains the original Cookie header. The act of decryption does not remove it.

Step 3: The Load Balancer Forwards the Request & Adds Metadata

    The load balancer now needs to forward this decrypted HTTP request to one of your backend Fastify instances.

    Before sending it, the load balancer adds standard HTTP headers to inform the backend about the original nature of the client's request. The most important one is X-Forwarded-Proto.

    The load balancer forwards the complete, original HTTP request (including the Cookie header) and simply appends these new headers.

Request Headers (from Load Balancer to Backend):
text

```
GET / HTTP/1.1
Host: internal-backend-address
Cookie: your_secure_session_cookie=encrypted_data_here
X-Forwarded-Proto: https   # <- The most important header
X-Forwarded-For: 123.123.123.123 # <- The client's original IP
X-Forwarded-Port: 443            # <- The original port
...other headers...
```

Note: The Cookie header is passed through completely unchanged. The backend instance receives it exactly as the browser sent it.
Step 4: The Backend (Fastify) Interprets the Request

    Your Fastify server receives the request from the load balancer. By default, it would see a plain HTTP request and think, "This is an insecure connection."

    However, because you configured Fastify with trustProxy: true, it knows to look for the X-Forwarded-* headers to understand the true nature of the request.

    Fastify sees X-Forwarded-Proto: https and essentially says, "Ah, even though this connection to me is HTTP, the original request from the client was HTTPS. I will therefore treat this as if it were a secure HTTPS request."

Step 5: Fastify-Secure-Session Makes Its Decision

    The fastify-secure-session plugin is asked to read the cookie.

    Because Fastify is now treating the request as secure (due to X-Forwarded-Proto), when the plugin checks request.secure or the protocol, it gets a true value.

    Since you configured it with secure: 'auto', it validates that the environment is "secure" and happily processes the Secure cookie. It would reject the cookie if X-Forwarded-Proto was http




this rule is enforced solely by the browser (the client). The load balancer is free to send the cookie over HTTP to your backend because it is not a browser and does not follow the same security rules.
Detailed Breakdown
1. Rule Enforcement: The Browser's Job

The statement "a Secure=true cookie shall not be sent over HTTP" is a security policy enforced by the web browser (Chrome, Firefox, Safari, etc.).

    How it works: When a website sets a cookie with the Secure attribute, the browser stores it and marks it as "only sendable over secure channels."

    Before sending a request: The browser checks the protocol of the URL it's about to call.

        If the URL starts with https://, the browser attaches all relevant cookies, including those marked Secure.

        If the URL starts with http://, the browser will not attach any cookies marked Secure. This is the enforcement.

This is a critical client-side security feature designed to prevent accidental leakage of sensitive cookies over unencrypted networks.
2. The Load Balancer's Role: It's a Server, Not a Client

The load balancer operates completely differently. It is a server-side component that receives requests and forwards them.

    It receives the client's HTTPS request, which includes the Secure cookie (because the browser correctly sent it over HTTPS).

    It terminates SSL, decrypting the request. The decrypted request now exists in memory on the load balancer as plain HTTP, but it still contains the original Cookie header with the secure cookie.

    It forwards this entire HTTP request (headers, body, and the Cookie header) to the backend server.

The load balancer does not care about the Secure attribute. Its job is to faithfully forward the request it received. The Secure attribute is a instruction for the browser, not for intermediaries like load balancers.
Visualization of the Flow



Why This is Still Secure

This architecture is considered secure because the trust boundary is between the client and the load balancer. The network between the load balancer and your backend instances is considered a trusted, private network (e.g., within a secure cloud VPC or data center). The security of that internal network is managed by you/your cloud provider, not by the public-facing rules of web browsers.

The X-Forwarded-Proto header allows your backend to reconstruct the client's experience and apply logic (like cookie security) appropriately, even on the trusted internal network.
