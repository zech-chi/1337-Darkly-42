## Chapter — User Enumeration
#user-enumeration #information-disclosure #broken-access-control

The endpoint `/api/users` returns the full list of every user on the platform. Any logged-in user can call it — not just admins. A normal user should never be able to list the whole user base.

I logged in as jdoe (a normal student) and called it:
`GET /api/users`

It returned every account, with usernames, emails, roles, and full IDs:
- jdoe, benjamin, dorian, thanos, anne-sophie, moderator, emilie, wil, sophie
- including the high-value accounts: wil (staff) and sophie (god)

### Impact
An attacker gets a complete map of all users: who exists, their emails, their roles, and their IDs. This data feeds almost every other attack:
- emails → password reset and brute-force targets
- full IDs → IDOR (open any profile / other users' data by ID)
- roles → know which accounts are worth attacking (staff, god)
User enumeration turns a blind attacker into one who knows exactly who to hit.

### Prevention
- Do not expose a full user list to normal users. Restrict `/api/users` to admins only.
- Return only the data the caller is allowed to see (for a normal user, maybe just their own record).