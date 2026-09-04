## Chapter 06 — Mass Assignment (Privilege Escalation)
#mass-assignment #broken-access-control #privilege-escalation

I can change my own profile with `PATCH /api/profile`. The server takes the fields I send and saves them — but it does not check WHICH fields I am allowed to change. So I can send a `role` field and change my own role.

I logged in as jdoe (role: student) and sent:
`PATCH /api/profile` with body `{"role":"cadet"}`

The server accepted it. My role changed from student to cadet, and I got the flag:
	`🚩🚩🚩 FLAG{just_p4tch_y0ur_0wn_r0l3_lol}`

**Mass Assignment:** the server binds all the fields from my request straight to the database record, including `role` — a field only an admin should change. I raised my own permission level just by adding one field to the request.

Note: I could go student → cadet, but `{"role":"staff"}` was blocked ("role not assignable"). So the app guards the high roles here — but only in this endpoint (I bypassed it later through PocketBase directly).

### Impact

A normal user can raise their own role and get access they should not have. This is privilege escalation: one extra field in a request turns a student into a cadet, giving more permissions.

### Prevention

- Use a whitelist: only allow safe fields (name, avatar, bio) to be changed by the user. Never let the user set `role` or other permission fields.
- Set sensitive fields (like `role`) only on the server side, never from user input.
- Check permissions before saving any field that controls access.
