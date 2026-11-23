# Natas 5 → Natas 6 — Changing a Client-Controlled Cookie

## Objective

Retrieve the credential for `natas6` by understanding how Natas 5 decides whether a visitor is logged in.

**Challenge URL:** `http://natas5.natas.labs.overthewire.org/`

## Observation

After authenticating to Natas 5, the application still displays:

```text
Access disallowed. You are not logged in
```

This is a different check from the HTTP Basic Authentication used to enter the challenge. The page is making an additional access decision based on browser state. Inspecting the request in Burp reveals a cookie named `loggedin` with the value `0`.

The important detail is that the application is relying on a value stored on the client to decide whether access should be granted.

## Approach

1. Open Natas 5 in **Burp Suite's built-in browser** and authenticate with the credential obtained from Natas 4.
2. When the page reports that you are not logged in, inspect the HTTP exchange in Burp. The first response may set the cookie with `Set-Cookie: loggedin=0`; after the browser stores it, a subsequent request sends it back in a `Cookie` header.
3. Inspect the outgoing request and locate the `loggedin` value:

   ```http
   Cookie: loggedin=0
   ```

4. Change the value from `0` to `1` in the request:

   ```http
   Cookie: loggedin=1
   ```

5. Send the modified request. The application accepts the changed value and returns the credential for `natas6`.

I used Burp Suite's built-in browser and request inspection to work through this level. The same cookie can be examined or changed with a browser's storage tools, another intercepting proxy, or an HTTP client such as `curl`.

## Why This Worked

Cookies are sent by the browser in the `Cookie` request header. A server can initially tell the browser to store a cookie using the `Set-Cookie` response header; the browser then sends the stored value back on later requests. These are two different directions of the HTTP exchange. See MDN's documentation for the [`Set-Cookie`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie) and [`Cookie`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Cookie) headers.

In this challenge, `loggedin=0` represents the unauthorised state. Changing it to `loggedin=1` satisfies the application's check. The server has treated a plain, client-controlled value as proof of authorisation, without a reliable way to detect that the value was changed.

## A Point That Can Trip Up Testers

Do not confuse `Set-Cookie` with `Cookie`:

- `Set-Cookie` is a **response** header. The server uses it to tell the browser what cookie to store.
- `Cookie` is a **request** header. The browser sends stored cookies back to the server.

When testing the access check, pay attention to the `Cookie` header on the request that reaches the application. A common point of confusion is seeing `Set-Cookie: loggedin=0` in the response and trying to edit that as though it were a request header. The value the application checks must be sent back in the request as `Cookie: loggedin=1`. If the cookie is missing from the first request, let the browser receive the `Set-Cookie` response and inspect the next request. Also, do not confuse this extra application-level check with the Natas username-and-password prompt; the challenge expects both layers to be satisfied.

## Security Takeaway

A client-controlled flag such as `loggedin=1` must not be trusted as the sole basis for authorisation. Browsers and users can modify cookie values, and an intercepting proxy makes those changes visible and straightforward. A safer design uses an unpredictable session identifier and keeps authentication state and permissions on the server, validating the session on every protected request. OWASP's [Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html) explains why session meaning and privilege state should remain server-side.
