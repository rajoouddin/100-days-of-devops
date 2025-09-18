## Day 9 – MariaDB Troubleshooting (Permission Denied on InnoDB Temp Tablespace)

### 🎯 Objective
Resolve the issue where the Nautilus application could not connect to its MariaDB database because the service was down. The goal was to restore MariaDB to an active, running state and ensure it remains available.

---

### 🔧 Steps Taken
**1. Verified system and service state**
```bash
systemctl status mariadb
journalctl -u mariadb -n 200 --no-pager
```

- Found service inactive (dead).
- MariaDB would reach READY, then shut down cleanly.
- Later restarts failed with exit code `1/FAILURE`.

**2. Checked MariaDB logs**
```bash
sudo tail -n 200 /var/log/mariadb/mariadb.log
```

- Errors:
```bash
InnoDB: Operating system error number 13
Cannot open datafile './ibtmp1'
Unknown/unsupported storage engine: InnoDB
```

- Error 13 = permission denied.

**3. Inspected datadir and permissions**
```bash
grep -R -n '^datadir' /etc/my.cnf* /etc/mysql/mariadb.conf.d/*
# datadir=/var/lib/mysql

ls -ld /var/lib/mysql
# drwxr-xr-x 1 root mysql ...
```

- Ownership was `root:mysql` instead of `mysql:mysql`.

**4. Corrected ownership and permissions**
```bash
sudo chown -R mysql:mysql /var/lib/mysql
sudo find /var/lib/mysql -type d -exec chmod 750 {} \;
sudo find /var/lib/mysql -type f -exec chmod 640 {} \;
```

**5. Fixed runtime directory**
```bash
sudo mkdir -p /var/run/mysqld
sudo chown -R mysql:mysql /var/run/mysqld
sudo chmod 755 /var/run/mysqld
```

**6. Restarted service**
```bash
sudo systemctl start mariadb
sudo systemctl enable mariadb
```

---

### ⚠️ Issues Faced
- **MariaDB exited with “Operating system error number 13”** → InnoDB lacked permissions to write to its datadir.
- Root cause: `/var/lib/mysql` was owned by r`oot:mysql`, not `mysql:mysql`.
- Temporary tablespace `ibtmp1` could not be opened, blocking InnoDB startup.

---

### 🛠️ Resolution
- Reset ownership of the entire datadir to `mysql:mysql`.
- Enforced correct file/directory permissions (`750` for dirs, `640` for files).
- Recreated runtime directory `/var/run/mysqld` with correct ownership.
- Restarted MariaDB, which then came up cleanly.

---

### ✅ Validation
**Check service:**
```bash
systemctl status mariadb
# Active: active (running)
# Status: "Taking your SQL requests now..."
```

**Check socket and TCP:**
```bash
mysqladmin ping -u root --protocol=socket
# mysqld is alive

sudo ss -lntp | grep 3306
# port 3306 listening
```

**Check persistence:**
```bash
systemctl is-enabled mariadb
# enabled
```

---

### 📌 Key Takeaways
- InnoDB error 13 = permissions or SELinux problem 9/10 times.
- `ibtmp1` is the temporary tablespace → safe to remove; it will be re-created.
- `Loaded: ...; preset: disabled` only shows the distro default; since I explicitly enabled it, MariaDB will start on boot.
- Always ensure `/var/lib/mysql` is owned by `mysql:mysql` with correct perms.
- Check both logs and `systemd` status — they tell the full story.

---

### 🚀 Result
MariaDB was restored to a healthy state:
- Service is active and running.
- Database will auto-start on future boots.
