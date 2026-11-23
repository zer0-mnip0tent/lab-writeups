# Natas 3 → Natas 4 — Discovering an Exposed Directory Through `robots.txt`

## Objective

Retrieve the credential for `natas4` from information exposed by the Natas 3 web application.

**Challenge URL:** `http://natas3.natas.labs.overthewire.org/`

## Observation

The page provides very little visible information. Inspecting its HTML source reveals a comment:

```html
<!-- No more information leaks!! Not even Google will find it this time... -->
```

The reference to Google points towards search-engine crawlers. This suggests checking the site's `robots.txt` file rather than continuing to look for a password in the page itself.

## Approach

1. Open Natas 3 in **Burp Suite's built-in browser** and authenticate using the credential obtained from Natas 2.
2. Inspect the page's delivered HTML and notice the comment mentioning Google.
3. Request the root-level `robots.txt` file:

   ```text
   http://natas3.natas.labs.overthewire.org/robots.txt
   ```

4. Read the directive in the file:

   ```text
   User-agent: *
   Disallow: /s3cr3t/
   ```

5. Follow the disclosed path directly by requesting:

   ```text
   http://natas3.natas.labs.overthewire.org/s3cr3t/
   ```

6. The directory listing exposes `users.txt`. Open that file and locate the entry for `natas4`.
7. Use the corresponding credential to authenticate to Natas 4.

I used Burp Suite's built-in browser to inspect the source and browse the discovered paths. Burp is not required for this level: the same requests can be made with another browser, `curl`, or any HTTP client.

## Why This Worked

`robots.txt` is a convention used to give web crawlers instructions about which paths they should avoid or may crawl. It is not an access-control mechanism. The `Disallow` directive made `/s3cr3t/` a useful clue, but it did not prevent a visitor from requesting that path directly.

The server also exposed a directory listing at `/s3cr3t/`, making `users.txt` discoverable. That file contained the credential needed for the next level.

## A Point That Can Trip Up Testers

The source comment is a clue, not the location of the password itself. The reference to Google is intended to lead you to `robots.txt`. Also, a `Disallow` entry does not mean that a path is private or protected; it merely asks compliant crawlers not to access or index it. Follow the path and check what the server actually exposes.

## Security Takeaway

Do not use `robots.txt` to hide sensitive resources. Its contents are publicly retrievable, and the paths it lists are disclosed to anyone who reads the file. Sensitive files should be placed outside publicly accessible directories and protected with server-side access controls. Directory indexing should also be disabled unless it is deliberately required.
