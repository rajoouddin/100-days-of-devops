## Day 3 – Secure Root SSH Access

### 🎯 Objective
Today I focused on securing SSH access by disabling direct root logins. This is an important step because root SSH access is one of the most common attack vectors.

---

### 🔧 Steps I Took

**1. Checked the SSH configuration**
```bash
sudo nano /etc/ssh/sshd_config
```

Found the line:
```
#PermitRootLogin yes
```

**2. Disabled root login** \
Changed it to:
```
PermitRootLogin no
```

**3. Restarted the SSH service**
```bash
sudo systemctl restart sshd
```

_(On some systems, the service is called ssh instead of sshd.)_

**4. Verified the configuration** \
From another terminal:
```bash
ssh root@<server-ip>
```

The login was denied, which confirmed the setting worked.

**5. Tested non-root login** \
Logged in as my normal user and escalated with:
```bash
ssh rajoo@<server-ip>
sudo -i
```

---

### 📝 Notes

- I made sure to keep my original session open while testing changes to avoid locking myself out.
- This change forces users to log in with their own accounts and then use sudo, improving accountability and security.
- For production environments, I’d also disable password-based logins and enforce SSH key authentication.

---

### 🌍 Real-World Reflection

On a production system (like a database server), leaving root SSH access enabled makes it a huge target for brute-force attacks. By disabling it, attackers now need both a valid username and the right password or SSH key. This reduces risk significantly and aligns with security compliance frameworks like PCI DSS.

---

### ✅ What I Learned

- How to disable root SSH access (PermitRootLogin no).
- Why testing changes in a second session is critical to avoid lockouts.
- The value of enforcing sudo usage for accountability.
