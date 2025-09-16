# Day 1 — Linux User Setup with Non-Interactive Shell

## Reflections

**Q1: What’s the difference between interactive and non-interactive shells?**  
A: **Interactive shells** let a user log in and run commands with prompts (e.g. via SSH or terminal).  **Non-interactive shells** are used for running scripts/automated processess without a login session (or then can block login entirely, like `/sbin/nologin`)

**Q2: Why might companies assign `/sbin/nologin` to certain accounts?**  
A: Companies assign `/sbin/nologin` (or `/bin/false`) to **service accounts** so they can run services but cannot be abused for interactive logins (e.g. via SSH). This reduces the attack surface.

**Q3: What did you learn about `/etc/passwd` today?**  
A: `/etc/passwd` stores user account details such as username, UID, GID, home directory and shell.  Passwords are no longer stored here (they're in `/etc/shadow), but this file still defines how users interact with the system.

---

## Commands I Used

```bash
# Example commands
sudo useradd -m -s /sbin/nologin appuser
grep appuser /etc/passwd
su - appuser
ps -u appuser
./check_user.sh appuser
```
---

## Additional Notes

### `su` and `/sbin/nologin`

When I first tested with:

```bash
su - appuser
```

I was prompted for a password. This happens because `su` requires the target user’s password if you are not root.
Since I never set one for `appuser`, authentication failed before `/sbin/nologin` could block the login.

When I tried instead with:

```bash
sudo su - appuser
```

I saw:
```bash
This account is currently not available.
```

This confirmed that the non-interactive shell (`/sbin/nologin`) was working as intended.

👉 Lesson learned:

- `su` - user (as a normal user) asks for that account’s password.
- `sudo su` - user switches directly as root and lets the shell setting (`/sbin/nologin`) enforce the restriction.

Cleanup
Once testing was complete, I removed the user to keep the system clean:

Remove user only (keep home directory):

```bash
sudo userdel appuser
```

Remove user and their home directory:
```bash
sudo userdel -r appuser
```

Then verified removal with:
```bash
grep appuser /etc/passwd
ps -u appuser
```

Nothing appeared, so the account is fully removed.
