## Chapter 1 — Broken Access Control
#broken-access-control #IDOR

here is the thing: as a Guest I visited http://localhost:4942/forum, and I checked all profiles and here is the information that I got:

| Username    | Email                       | Role    | ID              |
| ----------- | --------------------------- | ------- | --------------- |
| jdoe        | jdoe@student.42.tech        | student | z4p1cnx47mfy50f |
| benjamin    | benjamin@student.42.tech    | student | 8l16vboi47dmand |
| dorian      | dorian@student.42.tech      | student | 02g7nfbtw0k83fu |
| thanos      | thanos@student.42.tech      | student | sf6wtjycbyev4ah |
| anne-sophie | anne-sophie@student.42.tech | student | 0e9p852l6utwaal |
| moderator   | moderator@42network.fr      | student | xxv1btvtzaxen5h |
| emilie      | emilie@42.tech              | cadet   | 2na6hw2z9p1yosj |
| wil         | wil@42network.fr            | staff   | k1asdfeditojrb4 |
| sophie      | sophie@42.tech              | god     | enplwhu8jfo56oi |

notes:
- showing the IDs of users and their email is really bad!
- @jdoe said that his password is easy to guess, so I am thinking 🤔 of brute-forcing it!
- @benjamin said that the token is literally the md5 of his email address. This is good info — we can break authentication now with it!
- I can see other profiles and info details, and that led me to get the flag in @wil's profile:
	`🚩🚩🚩 FLAG{1d0r_ur_pr0f1l3_1s_m1n3}`
- @wil said: "Network ops ref for the team: dmFsaWRhdGlvbl9rZXk9NDJuZXR3b3Jr". After I decoded this using a base64 decoder I got `validation_key=42network`. Ohhh bro, they are cooked!! We can use it to break #JWT

### Impact

Any guest can read every user's profile just by knowing the ID — no login required. This exposes emails, roles, and IDs, and leaks secrets planted in profiles (the base64 key, the password/reset hints). Those leaks directly enable the next attacks: brute-forcing @jdoe, breaking authentication via the md5 reset token, and forging JWTs with the leaked secret. One weak access check becomes the entry point for the whole chain.

### Prevention

- Enforce authorization on `/profile/{id}`: a user should only read their own profile (or only public fields), never arbitrary IDs.
- Don't expose internal IDs and emails publicly; return only what the viewer is allowed to see.
- Never store secrets (keys, hints) in user-visible fields, and never post them in the forum.