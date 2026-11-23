# Natas 6 → Natas 7 — Finding a Secret in an Included File

## Objective

Retrieve the credential for `natas7` by tracing how Natas 6 loads and checks its secret value.

**Challenge URL:** `http://natas6.natas.labs.overthewire.org/`

## Observation

Natas 6 presents a form asking for a secret. Submitting an arbitrary value returns `Wrong secret`. The page also provides a **View sourcecode** link. The PHP source shows that the submitted value is compared with a variable named `$secret`, but that variable is not defined in the visible portion of the main page.

The important line is:

```php
include "includes/secret.inc";
```

This points to another file that supplies the value used by the comparison.

## Approach

1. Open Natas 6 in **Burp Suite's built-in browser** and authenticate with the credential obtained from Natas 5.
2. Review the page and select **View sourcecode** to open the supplied PHP source.
3. Follow the authentication logic. The form submits a field named `secret`, and the PHP code compares `$_POST['secret']` with `$secret`.
4. Notice that `$secret` is brought into the page through `includes/secret.inc`.
5. Request the referenced file directly:

   ```text
   http://natas6.natas.labs.overthewire.org/includes/secret.inc
   ```

6. The page may appear blank when rendered. Inspect its response source or raw body instead of assuming that the file is empty. In this challenge, the file exposes the PHP assignment that defines `$secret`.
7. Submit that value through the original Natas 6 form. The comparison succeeds and the application returns the credential for `natas7`.

I used Burp Suite's built-in browser to inspect the challenge and follow the referenced resource. The same investigation can be done in another browser or with an HTTP client; Burp is a convenient way to keep the page, requests, and responses together.

## Why This Worked

PHP's `include` statement loads and evaluates the specified file as part of the running script. Variables created in the included file become available in the scope where the file is included. In this case, `secret.inc` defines `$secret`, which the main page then compares with the submitted form value. The behaviour is documented in the [PHP manual for `include`](https://www.php.net/manual/en/function.include.php).

The challenge also makes the included file directly retrievable through the web server. That exposes the value the application was treating as a secret. The form itself is not being bypassed; the required value is discovered and submitted through the intended form.

## A Point That Can Trip Up Testers

The `include` statement is PHP code executed by the server; it is not automatically a hyperlink or a browser redirect. Read the path it references and consider whether that resource can also be requested directly. If opening `secret.inc` produces a visually blank page, inspect the response source/body. A file that assigns a variable but prints nothing can appear empty in the rendered browser tab even when its contents are present in the response.

Also, distinguish between the two locations involved:

- `index-source.html` exposes the challenge's main PHP source for inspection.
- `includes/secret.inc` is the separate file referenced by the `include` statement.

## Security Takeaway

Files containing credentials, API keys, or other sensitive configuration values should not be placed in a location where they can be retrieved directly over HTTP. Server-side inclusion does not make a file private. Store secrets outside the web document root, restrict filesystem permissions, and ensure that sensitive configuration files cannot be served as static resources. Source-code disclosure and accidental exposure of included files can turn an otherwise ordinary form check into a credential disclosure issue.
