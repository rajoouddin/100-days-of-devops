## Day 8 – Install Ansible 4.10.0 Globally via pip3

### 🎯 Objective
Install **Ansible 4.10.0** on the jump host using **pip3 only** and ensure the Ansible binary is available globally so all users can run Ansible commands.

---

### 🔧 Steps Taken
**1. Removed any existing Ansible installations:**
```bash
sudo pip3 uninstall -y ansible ansible-core
```

**2. Installed the exact required version globally:**

```bash
sudo pip3 install ansible==4.10.0
```

**3. Verified installation:**

```bash
ansible --version
which ansible
python3 -m pip show ansible
```

- `pip show ansible` confirmed version 4.10.0.
- `ansible --version` reported ansible-core 2.11.12 (this is expected: Ansible v4.10.0 bundles ansible-core 2.11.x).
- Binary located at `/usr/local/bin/ansible`.

**4. Confirmed file permissions allowed execution by all users:**

```bash
ls -l /usr/local/bin/ansible
# -rwxr-xr-x 1 root root ...
```

**5. Created a test user to validate global availability:**

``` bash
sudo useradd -m -s /bin/bash testuser
sudo -u testuser -H ansible --version
```

---

### ⚠️ Issues Faced
Running `sudo -u testuser -H ansible --version` initially failed with:

```bash
sudo: ansible: command not found
```

- Cause: sudo’s restricted PATH (set by `secure_path` in `/etc/sudoers`) did not include `/usr/local/bin`, even though the binary existed and was executable.

---

### 🛠️ Resolution
- Edited sudoers with `visudo` to update `secure_path` to include `/usr/local/sbin:/usr/local/bin`:

```bash
Defaults    secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
```

- After this change, `sudo -u testuser -H ansible --version` worked correctly and reported the same version as the primary user.

---

### ✅ Validation
- Both my user and the `testuser` account resolved `ansible` to the same binary:

```bash
which ansible
# /usr/local/bin/ansible
```

- Verified version consistency:
```bash
ansible --version
# ansible [core 2.11.12]

python3 -m pip show ansible | grep Version
# Version: 4.10.0
```

---

### 📌 Key Takeaways
- `ansible --version` shows the ansible-core version (2.11.12 here).
- `pip show ansible` shows the community distribution version (4.10.0 here).
- Both numbers are correct — they map to each other.
- Installing via `sudo pip3` makes Ansible binaries available system-wide under `/usr/local/bin`.
- Sudo may strip `/usr/local/bin` from PATH via `secure_path`; fixing this ensures global accessibility.
- Always test with a second user to confirm true system-wide availability.

---

### 🚀 Result
Ansible 4.10.0 is installed globally via pip3, with binaries in `/usr/local/bin`.
Both the primary user and additional users can run Ansible commands without extra configuration.
