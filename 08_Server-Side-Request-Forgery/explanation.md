## Chapter 08 — Server-Side Request Forgery (Internal Config Disclosure)
#xxe #ssrf #information-disclosure #privilege-escalation

The agenda page lets me import an XML file. XML has a feature called "external entities" — you can define a variable that points to a file or a URL, and the parser will go fetch it. This is dangerous, and it should be turned off (with a library like `defusedxml`). Here it is ON — the app comments even say they disabled defusedxml.

There is a page `/internal/config` that holds secrets. It is blocked to me — it only answers requests from localhost (127.0.0.1). Header tricks (X-Forwarded-For) did not work, because the app checks the real connection IP.

But the XML parser runs ON the server. So I make the SERVER fetch the page for me. The request comes from the server = from localhost = allowed. This is SSRF (Server-Side Request Forgery).

I uploaded this to `/agenda/import`:
`<!DOCTYPE agenda [ <!ENTITY xxe SYSTEM "http://127.0.0.1:4942/internal/config"> ]>`
`<agenda><event><title>&xxe;</title><date>2042-01-15</date></event></agenda>`

The server fetched the config and put it in my event's title. It leaked:
`{"jwt_secret":"42network","pb_admin_email":"admin@42network.local","pb_admin_password":"Darkly42Admin!","darkly_flag":"FLAG{d3fus3dxml_n3xt_spr1nt_pr0m1s3}"}`
	`🚩🚩🚩 FLAG{d3fus3dxml_n3xt_spr1nt_pr0m1s3}`

The flag is small compared to the real prize: the **PocketBase admin email and password**, which I use next to log into the database admin panel.

### Impact

Two bugs chain here. XXE lets me make the server read files/URLs. SSRF lets me reach an internal page that was blocked to outsiders. Together they leak all the secrets: the JWT signing key and the database admin password. This leads to full takeover.

### Prevention

- Turn off external entities in the XML parser (use `defusedxml`).
- Block the parser (and the server) from fetching internal URLs — deny requests to localhost / internal IPs.
- Do not protect an internal endpoint by IP alone; require real authentication. And never store secrets in a reachable config, in plain text.