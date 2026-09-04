## Chapter 07 — JWT Forgery (Weak Signing Secret)
#jwt #jwt-forgery #weak-secret #broken-authentication #privilege-escalation

The session cookie is a JWT. A JWT has 3 parts: header, payload, and signature. The payload holds my identity, like `{"sub": "...", "role": "...", "exp": ...}`. The signature proves the token was not changed — it is `HMAC-SHA256(header.payload, secret)`.

The problem: the secret leaked. In the forum, @wil posted a base64 string that decodes to `validation_key=42network`. So the secret is `42network`.

Because I know the secret, I can make my OWN token and sign it correctly. The server will trust it. I forged a token that says I am **sophie** (the god account):
`{"sub":"enplwhu8jfo56oi","login":"sophie","role":"god","exp":1999999999}`

I found that `/admin` checks the `sub` (the user's ID), not the role:
- `sub=jdoe, role=god` → 403 (not allowed)
- `sub=sophie, role=god` → **200, admin page + flag**

So I forged `sub=sophie` and opened `/admin`:
	`🚩🚩🚩 FLAG{md5_1s_4_n4m3pl4t3_n0t_4_l0ck}

**JWT Forgery:** the token's safety depends only on the secret. A weak or leaked secret means anyone can sign a fake token and become any user.

### Impact

Anyone who knows the secret can log in as any user, including admin/god — no password needed. This is a full authentication bypass and account takeover.

### Prevention

- Use a strong, random secret (long and unpredictable), stored safely (env var or secrets manager).
- Do not decide access from a value the user can forge. Check identity/role on the server, or use server-side sessions.