## Chapter 3 — Broken Access Control (IDOR)
#broken-access-control #idor

In Chapter 1, I got a list of user IDs from the forum, now I use them.
If I open a profile by its ID, the server shows it to me — it never checks if I am allowed to see it: `http://localhost:4942/profile/<id>`
Using @jdoe session, I opened @wil's profile (staff) with his ID `k1asdfeditojrb4`, and I found the flag there: `http://localhost:4942/profile/k1asdfeditojrb4` `🚩🚩🚩 FLAG{1d0r_ur_pr0f1l3_1s_m1n3}`

### Impact

Anyone can read any user's profile just by knowing the ID. The IDs are public , so this is easy. No login and no permission are needed. The server trusts the ID in the URL instead of checking who is asking.

### Prevention

- Check permissions on `/profile/{id}`: make sure the user owns the profile, or has a role that is allowed to see it, before showing it.
- Only send the fields the viewer is allowed to see, not the full data.
- Use IDs that are hard to guess, so people can't try many IDs one by one.