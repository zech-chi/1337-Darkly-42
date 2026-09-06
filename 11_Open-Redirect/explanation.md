## Chapter — Open Redirect
#open-redirect

The `/redirect` endpoint takes a `next` parameter and sends the browser to that URL. The problem: it does not check the URL. It will send you to ANY site, including an attacker's site.

I tested it:
`http://localhost:4942/redirect?next=https://evil.com`

The server answered with a redirect straight to `https://evil.com`:
`HTTP/1.1 307 Temporary Redirect`
`location: https://evil.com`

So the app redirects to any URL I put in `next`, with no check.

(No flag here — this is a real vulnerability)

### Impact
This is used for phishing. An attacker sends a link that STARTS with the real, trusted domain:
`http://localhost:4942/redirect?next=https://evil.com`
The victim sees the trusted domain and clicks, but they land on the attacker's fake site (for example a fake login page to steal their password). It can also be used to steal OAuth tokens in some flows.

### Prevention
- Only allow redirects to your own site (relative paths like `/profile`), not to full external URLs.
- If external redirects are needed, use an allowlist of approved domains.
- Or show a warning page ("You are leaving this site") before redirecting.