## Chapter 09 — PocketBase Superuser Access
#pocketbase #broken-authentication #privilege-escalation

The backend is PocketBase, running on port 8090. It has its own admin panel at `http://localhost:8090/_/` and its own login API — separate from the main app.

In the XXE chapter, the leaked `/internal/config` gave me the PocketBase admin login:
`pb_admin_email = admin@42network.local`
`pb_admin_password = Darkly42Admin!`

I used these to log in as **superuser** — the highest level, full control of the database. This is the real "administrator" of the app (not just the god role in the web app).

I logged in through the PocketBase auth API:
`POST /api/collections/_superusers/auth-with-password`
with the leaked email and password. It gave me an admin token.

With that token I can read the whole database. Before, listing collections gave 401 (admin only). Now it works. I opened the `internal_config` collection and found the flag:
	`🚩🚩🚩 FLAG{th3_und3rsc0r3_sl4sh_kn0ws_th3_w4y}`

(The flag name points to `_/` — the admin panel path that `robots.txt` disclosed.)

### Impact

Superuser access = total control of the database. I can read, edit, or delete any record: all users, all grades, all secrets. This is the worst case — full backend takeover. It happened only because the admin password was leaked and stored in plain text.

### Prevention

- Never store admin credentials in a config that can be reached. Keep secrets in a secrets manager, not in the database or a config endpoint.
- Use a strong admin password and rotate it — `Darkly42Admin!` is guessable-style and was never changed.
- Do not expose the admin panel (`/_/`) to everyone; restrict it by network/IP or VPN.