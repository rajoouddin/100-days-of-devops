## Day 2 – Temporary User Setup with Expiry

### 🎯 Objective
Create a Linux user account that automatically expires after a set date.  
Useful for temporary staff, contractors, or challenge testing.

---

### 🔧 Steps Taken

**1. Create the temporary user**
```bash
sudo useradd -m -s /bin/bash tempuser
sudo passwd tempuser
```

**2. Set an expiry date (example: 2 days from now)**
```bash
sudo usermod --expiredate "$(date -d '+2 days' +%Y-%m-%d)" tempuser
# Alternative (set a specific date, e.g., 2025-09-12)
# sudo usermod --expiredate 2025-09-12 tempuser
# Or using chage (equivalent):
# sudo chage -E 2025-09-12 tempuser
```

**3. Verify expiry details**
```bash
sudo chage -l tempuser
```

Looked for the line:
```bash
Account expires : <DATE>
```

**4. Clean up (removed user after testing)**
```bash
sudo userdel -r tempuser
# Verify removal:
getent passwd tempuser || echo "tempuser not found (removed)"
```

---

📝 Notes
- `--expiredate` expects the format YYYY-MM-DD.
- Expiry prevents login after the date; it does not delete the account.
- `chage -l <user>` shows password/expiry status at any time.

---

🌍 Real-World Scenario
- Imagine your company brings in a security consultant for a 2-week penetration test.
- Instead of relying on memory to remove their account after the contract ends, you can:

```bash
sudo useradd -m -s /bin/bash consultant
sudo passwd consultant
sudo usermod --expiredate 2025-09-24 consultant
```

This ensures:

- The consultant can work during the agreed period.
- Their access automatically expires on the deadline.
- No risk of leaving stale accounts that attackers could abuse later.

This is a common best practice in finance, healthcare, and PCI-DSS regulated environments where temporary access must be tightly controlled.
