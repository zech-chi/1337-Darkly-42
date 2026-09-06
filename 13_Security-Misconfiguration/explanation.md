## Chapter — Security Misconfiguration
#security-misconfiguratio  #information-disclosure

Most of the earlier breaches were not clever hacks — they were switches left in the wrong
position. The app ships with insecure defaults and leftover debug settings that each opened
the door to a specific attack:

- **Session cookie `HttpOnly=false`** → JavaScript can read `document.cookie`, which is the
  whole reason the Stored XSS could steal the moderator's session (Chapter 5). One flag flip
  (`HttpOnly=true`) kills that attack.
- **`defusedxml` disabled, stdlib `ElementTree` left on** → external entities are parsed →
  XXE / SSRF (Chapter 8).
- **PocketBase admin panel `/_/` and API exposed on port 8090** → once the admin password
  leaked, the whole database was one request away (Chapter 9).
- **`robots.txt` lists the secret paths** (`/staff`, `/backup`, `/internal/config`, `/_/`) →
  it is a map of the attack surface handed to anyone (Chapter 1).
- **Debug headers + verbose source comments left in production** ("TODO: remove debug
  headers", deploy log, "PATCH /api/profile accepts role") → internal hints leak on every page.

### Impact

Insecure defaults turn small mistakes into full compromises. Each wrong setting removed a
safety net that would have blocked or contained an earlier attack; together they are why the
exploit chain works end to end.

### Prevention

- Harden defaults: `HttpOnly` + `Secure` + `SameSite` on cookies, `defusedxml` for XML, a strict CSP and the standard security headers.
- Don't expose admin panels/DB ports to the network; strip debug headers, verbose errors and internal comments before production; keep `robots.txt` free of secret paths.
