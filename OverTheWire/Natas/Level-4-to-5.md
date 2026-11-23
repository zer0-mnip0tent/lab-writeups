# Natas 4 → Natas 5 — Bypassing a Referer-Based Access Check

## Objective

Retrieve the credential for `natas5` from the Natas 4 web application.

**Challenge URL:** `http://natas4.natas.labs.overthewire.org/`

## Observation

After authenticating to Natas 4, the page responds with an access-denied message. It reports that the request came from an empty referrer value (`""`) and states that authorised requests should come from:

```text
http://natas5.natas.labs.overthewire.org/
```

This message identifies the condition being checked: the HTTP `Referer` request header. The restriction is not based on the path used to request `index.php`.

## Approach

1. Open Natas 4 in **Burp Suite's built-in browser** and authenticate using the credential obtained from Natas 3.
2. Observe the access-denied response and note the exact URL the server expects as the referring page: `http://natas5.natas.labs.overthewire.org/`.
3. Capture the outgoing GET request to Natas 4 using Burp Suite.
4. Inspect the request headers. The initial request does not contain a usable `Referer` value, which is why the response displays an empty string.
5. Modify the `Referer` header to match the value stated by the server:

   ```http
   Referer: http://natas5.natas.labs.overthewire.org/
   ```

6. Forward the modified request while retaining the normal Natas 4 authentication.
7. The application accepts the request and returns the credential for `natas5`.

I used Burp Suite to inspect and modify the GET request. The same change can be made with another intercepting proxy or an HTTP client such as `curl` by setting the `Referer` header manually.

## Why This Worked

The application made its access decision using the value of the `Referer` header. That header normally identifies the page from which a request was initiated, but it is supplied as part of the HTTP request and is not proof of the visitor's identity or authorisation.

By sending the exact value expected by the application, the request satisfied the check. The server accepted the request and disclosed the next credential.

## A Point That Can Trip Up Testers

Changing the URL path or removing `index.php` does not solve this condition. The application is checking request metadata, not the requested filename. When the response says `visiting from ""`, inspect the outgoing request headers and look specifically for `Referer`.

The header is spelled `Referer` (one `r` after `refe`) in the HTTP standard. Do not confuse it with the correctly spelled English word *referrer*, or with the `Host` header. The value must also match the expected URL, including its scheme, host, and trailing slash.

## Security Takeaway

A `Referer` value is client-supplied request metadata. It can be absent, shortened by browser privacy policies, removed by intermediaries, or deliberately set by a client. It must not be used as the sole mechanism for authentication or authorisation. Sensitive resources require proper server-side access controls.
