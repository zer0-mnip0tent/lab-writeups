# Natas 1 → Natas 2 — Inspecting HTML Source When Right-Click Is Disabled

## Objective

Retrieve the credential for `natas2` from the Natas 1 web page.

**Challenge URL:** `http://natas1.natas.labs.overthewire.org/`

## Observation

The page states that the next password is present on the page, but right-clicking has been disabled. This restriction is implemented through client-side JavaScript that blocks the browser's context menu. It does not remove the HTML source delivered to the browser.

## Approach

1. Open Natas 1 in **Burp Suite's built-in browser** and authenticate using the credential obtained from Natas 0.
2. Observe that the page blocks the normal right-click context menu.
3. Use Burp's browser workflow to inspect the page's delivered HTML rather than relying on the disabled context-menu action.
4. Locate the HTML comment containing the credential for `natas2`.
5. Use that credential to authenticate to Natas 2.

I used Burp Suite's built-in browser for browsing and inspecting the page. This is a record of my workflow, not a requirement of the challenge: the same HTML can also be inspected with another browser's developer tools or page-source view, or by making a direct HTTP request.

## Why This Worked

The right-click restriction is only a client-side user-interface control. The browser still receives the HTML document so it can render the page. Inspecting the delivered source through Burp's browser avoids dependence on the blocked context-menu interaction; it does not require bypassing server-side access control.

## Security Takeaway

Client-side restrictions do not protect information already delivered to the browser. Passwords, tokens and other sensitive values must not be embedded in HTML comments, hidden form fields, scripts or other client-accessible content. Access control must be enforced on the server, and secrets should not be sent to unauthorised clients.
