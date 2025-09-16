## Day 5 – SELinux Installation and Configuration

### 🎯 Objective
The request was clear:
- Install the required SELinux packages.
- Permanently disable SELinux for the time being (re-enable later once config changes are made).
- No need to reboot now since a scheduled maintenance reboot is planned for tonight.
- The final status after reboot should be **disabled**, regardless of the current runtime status.

---

### 🔧 Steps I Took

**1. Checked if SELinux was installed**
```bash
sestatus
```
At first, the command wasn’t found, which told me SELinux wasn’t installed yet.

**2. Installed SELinux packages**
```bash
sudo apt install selinux-basics selinux-policy-default auditd
```

**3. Checked SELinux config**
```bash
sudo cat /etc/selinux/config
```

It showed:
```bash
SELINUX=enforcing
```
Although the runtime status (`getenforce` / `sestatus`) showed Disabled, the config file told me that after a reboot, SELinux would come back as Enforcing. This needed to be changed.

**4. Disabled SELinux persistently** 

I edited the config file:
```bash
sudo nano /etc/selinux/config
```

Changed this line:
```bash
SELINUX=enforcing
```
to:
```bash
SELINUX=disabled
```

**5. Verified configuration change**
```bash
grep SELINUX= /etc/selinux/config
```
Output:
```bash
SELINUX=disabled
```
This ensures that when the server reboots during the planned maintenance window, SELinux will remain disabled.

---

### 📝 Notes & Learnings
- Runtime status (`getenforce`, `sestatus`) is not always the whole truth — persistent config is controlled by /etc/selinux/config.
- Without this change, SELinux would have silently re-enabled after reboot, which could have caused application issues.
- Disabling SELinux should only be temporary. Long-term, it’s best practice to run SELinux in enforcing mode and tune policies instead of leaving it disabled.

---

### ✅ Key Takeaways
- `/etc/selinux/config` defines SELinux state across reboots.
- Current runtime status can differ from persistent config.
- Setting SELINUX=disabled ensures it stays off after a reboot, but this should be revisited after planned configuration changes.

---

### 🚀 Reflection
Today’s task reinforced the importance of not just trusting runtime status but also checking persistent configs. It was a subtle but critical detail — and exactly the kind of thing that can bite you in production if you overlook it.

I now know how to install SELinux packages, control its persistent state, and ensure the server reboots into the expected mode.
