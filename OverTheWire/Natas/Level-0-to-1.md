# Natas 0 → Natas 1 — HTML Source Inspection and Information Disclosure

## Objective

Retrieve the credential for `natas1` from the Natas 0 web page.

**Challenge URL:** `http://natas0.natas.labs.overthewire.org/`

## Observation

After authenticating to Natas 0 with the starting credentials provided by OverTheWire (`natas0` / `natas0`), the page displays a message indicating that the next level's password is present on the page. The password is not displayed as normal visible page content.

The relevant information is in the HTML delivered to the browser: an HTML comment contains the password for `natas1`.

## Approach

1. Open Natas 0 in **Burp Suite's built-in browser** and authenticate with the initial credentials supplied by OverTheWire.
2. Inspect the page's delivered HTML source using the browser inspection workflow.
3. Locate the HTML comment that identifies the password for `natas1`.
4. Use the credential in that comment to authenticate to Natas 1.

I used Burp Suite's built-in browser to access and inspect the page. Burp is the tool used in my workflow, not a requirement of the challenge. The same source can be examined with another browser's page-source view or developer tools, or by issuing an HTTP request with a client such as `curl`.

## Why This Worked

HTML comments are not rendered as visible page content, but they are still part of the HTML document sent to the client. Anyone who can access the page can inspect the response and read those comments. Hiding a credential in an HTML comment therefore provides no confidentiality.

## Security Takeaway

Never place passwords, API keys, tokens, or other sensitive information in client-delivered content, including HTML comments, hidden elements, or JavaScript. If information must remain secret, do not send it to an unauthorised client. Enforce access control on the server instead of relying on presentation-layer concealment.
