## Day 7 – Linux SSH Authentication

### 🎯 Objective

Set up password-less SSH authentication from the `thor` user on the jump host to all three app servers using their respective `sudo` users:
- tony@stapp01
- steve@stapp02
- banner@stapp03

This was needed so that automation scripts can run without human intervention.

---

### 🔧 Steps I Took

**1. If I needed to switch user**
```bash
sudo su - thor
```

**2. Generated SSH keys**

I learned about two key types:

- RSA (legacy safe choice, very compatible):
    ```bash
    ssh-keygen -t rsa -b 4096 -C "thor@jump_host"
    ```

- Ed25519 (modern best choice, faster & smaller):
    ```bash
    ssh-keygen -t ed25519 -C "thor@jump_host"
    ```

The `-C` option is just a comment/label to identify the key later.

**3. Copied the public key to each app server**
```bash
ssh-copy-id tony@stapp01
ssh-copy-id steve@stapp02
ssh-copy-id banner@stapp03
```

**4. Verified logins**

From the jump host as `thor`:
```bash
ssh tony@stapp01
ssh steve@stapp02
ssh banner@stapp03
```

All connections worked without passwords.

**5. Permissions check (optional)**

Normally I’d run:
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

But since I used `ssh-copy-id`, it automatically set the correct permissions. I confirmed they were already locked down with a quick check.

---

### 🛠 Real-World Scenario

This is exactly how config management tools like Ansible or Terraform connect to fleets of servers. Without password-less SSH, automation would break waiting for human input.

---

### ✅ Extra Validation (bonus)

Out of curiosity, I ran a small loop to check file permissions for each user:
```bash
for u in "tony@stapp01" "steve@stapp02" "banner@stapp03"; do
  ssh "$u" 'printf "%s:\n" "$USER"; ls -ld ~ ~/.ssh ~/.ssh/authorized_keys || true'
done
```

This confirmed:
- Home dirs = `700`
- `.ssh` dirs = `700`
- `authorized_keys` = `600`

⚡ This step is a little more advanced than where I’m at, but it was great practice and showed me how a DevOps engineer might validate multiple servers in one go.

---

### 📝 Key Takeaways

- SSH keys are safer and more practical than passwords.
- RSA = compatibility, Ed25519 = modern and fast.
- Public key goes to the server, private key stays safe with me.
- Permissions matter, but ssh-copy-id usually sets them correctly.
- Advanced validation loops are useful, but not strictly needed for beginners.

🔥 Today felt like a turning point — I didn’t just set up password-less SSH, I also dipped into advanced verification. That gave me a glimpse of how the pros think, and it feels good to be building those habits early.
