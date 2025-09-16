## Day 4 – Script Execution Permissions

### 🎯 Objective
Today I learned how to control execution permissions for shell scripts in Linux.  
This is key because scripts often run critical automation (like deployments, monitoring, or backups), and the wrong permissions can either block execution or expose security risks.

---

### 🔧 Steps I Took

1. **Created a test script**
```bash
   echo -e '#!/bin/bash\necho "Hello DevOps"' > hello.sh
```

2. **Checked permissions**
```bash
ls -l hello.sh
```

Output:
```bash
-rw-r--r-- 1 rajoo rajoo 28 Sep 10 20:15 hello.sh
```
At this stage, it wasn’t executable.

3. **Made it executable**
```bash
chmod +x hello.sh
ls -l hello.sh
```

Now it showed:
```bash
-rwxr-xr-x 1 rajoo rajoo 28 Sep 10 20:16 hello.sh
```

4. **Ran the script**
```bash
./hello.sh
```

Output:
```bash
Hello DevOps
```

5. **Tightened permissions**

For myself only:
```bash
chmod 700 hello.sh
```
For team execution but not public:
```bash
chmod 750 hello.sh
```

6. **Cleaned up after testing**
```bash
rm hello.sh
```

---

### ⚠️ Notes & Best Practices

- Avoid `chmod 777` at all costs — it gives unsafe access to everyone.
- Keep scripts in controlled directories (e.g., `/usr/local/bin`) if shared.
- Always set the correct shebang (`#!/bin/bash`) to ensure the right shell is used.
- Follow least privilege: only allow access to those who really need it.

---

### ✅ Challenge I Tried
I wrote a script `info.sh` that prints today’s date and my username, gave it `700` permissions, and confirmed it only runs for me. This reinforced how to manage execution securely.

🔧 Steps

1. Create the script
```bash
echo -e '#!/bin/bash\necho "Today is: $(date)"\necho "Logged in as: $(whoami)"' > info.sh
```

2. Set permissions (owner only)
```bash
chmod 700 info.sh
```

3. Run the script
```bash
./info.sh
```
✅ Output
```bash
Today is: Mon Sep 15 14:42:31 UTC 2025
Logged in as: rajoo_uddin
```

---

### 📝 Key Takeaways
- Scripts won’t run unless the execute bit is set.
- `chmod` gives me precise control over who can run a script.
- Proper permissions = security + functionality.
- Least privilege is always the right mindset.
- I feel good about this one. Knowing how to manage execution permissions means I can confidently write and share scripts without worrying about errors or security holes. 
