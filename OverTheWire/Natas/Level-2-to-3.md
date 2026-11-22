# Natas 2 → Natas 3 — Exposed Directory Listing and Sensitive File Disclosure

## Objective

Retrieve the credential for `natas3` from information exposed by the Natas 2 web application.

**Challenge URL:** `http://natas2.natas.labs.overthewire.org/`

## Observation

The visible page appears to contain nothing useful. Inspecting the delivered HTML reveals an image reference:

```html
<img src="files/pixel.png">
```

The image is not significant because of its visual content. The useful clue is its location: `files/pixel.png`. This identifies a directory named `files` beneath the current web path.

## Approach

1. Open Natas 2 in **Burp Suite's built-in browser** and authenticate using the credential obtained from Natas 1.
2. Inspect the page's delivered HTML and identify the relative image path `files/pixel.png`.
3. Open the referenced image. It is a tiny, 1 × 1 PNG; viewing its raw bytes as text produces binary-looking characters rather than readable content.
4. Recognise that the image itself is not the source of the credential. The useful clue is the parent directory, `files`.
5. In Burp's browser, request the containing directory directly:

   ```text
   http://natas2.natas.labs.overthewire.org/files/
   ```

6. Observe that the web server returns a directory listing. Alongside `pixel.png`, the listing exposes `users.txt`.
7. Open `users.txt` and locate the entry for `natas3`.
8. Use the corresponding credential to authenticate to Natas 3.

I used Burp Suite's built-in browser to browse the challenge, inspect its HTML and request the exposed directory. This is not a requirement: the same steps can be performed with another browser, developer tools, `curl`, or another HTTP client.

## Why This Worked

The HTML source disclosed the path of a resource stored in the `files` directory. Directory indexing was enabled, so requesting the directory itself exposed its contents. The `users.txt` file was stored within that publicly accessible location and contained username/password entries, including the one needed for the next level.

The initial PNG inspection helped establish that the image itself was not where the credential was hidden. The decisive step was following the image's path back to its containing directory and inspecting the directory listing.

## Security Takeaway

Static-resource paths can disclose useful directory names. If directory indexing is enabled, files that are not linked from the page may still be discoverable by requesting their parent directory. Disable directory listings unless they are explicitly required, and never store credentials or other sensitive files inside a publicly accessible web directory.
