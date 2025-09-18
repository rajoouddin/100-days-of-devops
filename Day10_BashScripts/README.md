## Day 10 – Linux Bash Scripts: Blog Backup (stapp02)

### 🎯 Objective
Automate the backup of the `/var/www/html/blog` directory on **App Server 2 (stapp02)** by:  
- Creating a zip archive named **xfusioncorp_blog.zip**  
- Storing it in `/backup` on stapp02  
- Copying it to **stbkp01:/backup** (passwordless SSH)  
- Ensuring the script runs as **steve** without any `sudo` inside  
- Placing the script at `/scripts/blog_backup.sh`  

---

### 🔧 Steps Taken

**1. Installed `zip` package on stapp02:**
```bash
sudo yum install -y zip
```

**2. Prepared directories for script and local backups:**
```bash
sudo mkdir -p /scripts /backup
sudo chown -R steve:steve /scripts /backup
```

**3. Configured passwordless SSH to backup server (stbkp01):**
```bash
ssh-keygen -t rsa -b 4096 -N "" -f ~/.ssh/id_rsa
ssh-copy-id clint@stbkp01
ssh clint@stbkp01 'echo OK on $(hostname) && mkdir -p /backup'
```

**4. Created the script /scripts/blog_backup.sh:**
```bash
#!/bin/bash
set -euo pipefail

SRC="/var/www/html/blog"
LOCAL_BACKUP_DIR="/backup"
ARCHIVE_NAME="xfusioncorp_blog.zip"
ARCHIVE_PATH="${LOCAL_BACKUP_DIR}/${ARCHIVE_NAME}"

REMOTE_USER="clint"
REMOTE_HOST="stbkp01"
REMOTE_BACKUP_DIR="/backup"
SSH_OPTS="-o BatchMode=yes -o StrictHostKeyChecking=accept-new"

[ -d "$SRC" ] || { echo "ERROR: $SRC not found"; exit 1; }
mkdir -p "$LOCAL_BACKUP_DIR"

cd /var/www/html
rm -f "$ARCHIVE_PATH"
zip -rq "$ARCHIVE_PATH" blog
[ -s "$ARCHIVE_PATH" ] || { echo "ERROR: Archive is empty"; exit 1; }

ssh $SSH_OPTS "${REMOTE_USER}@${REMOTE_HOST}" "mkdir -p '${REMOTE_BACKUP_DIR}'"
scp $SSH_OPTS "$ARCHIVE_PATH" "${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_BACKUP_DIR}/"
ssh $SSH_OPTS "${REMOTE_USER}@${REMOTE_HOST}" "test -s '${REMOTE_BACKUP_DIR}/${ARCHIVE_NAME}'"

echo "Backup complete:"
echo " - Local : $ARCHIVE_PATH"
echo " - Remote: ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_BACKUP_DIR}/${ARCHIVE_NAME}"
```

**5. Made script executable and ran it:**
```bash
chmod +x /scripts/blog_backup.sh
/scripts/blog_backup.sh
```

**6. Validated backup on both servers:**
```bash
ls -lh /backup/xfusioncorp_blog.zip
ssh clint@stbkp01 'ls -lh /backup/xfusioncorp_blog.zip'
```

**7. Confirmed archive structure contains blog/:**
```bash
unzip -l /backup/xfusioncorp_blog.zip | head -10
ssh clint@stbkp01.stratos.xfusioncorp.com 'unzip -l /backup/xfusioncorp_blog.zip | head -10'
```

- Output:
```bash
blog/
blog/index.html
blog/.gitkeep
```

---

### 📝 Notes, Issues & Fixes
- Needed to zip from `/var/www/html` with `zip ... blog` to preserve the top-level folder.
- First run added `stbkp01` host key; subsequent runs were seamless.
- Script includes sanity checks and fails safely if the source folder or archive is missing.

---

### ⏭️ Future Enhancements
- Timestamped filenames for backup history:
```bash
ARCHIVE_NAME="xfusioncorp_blog_$(date +%F_%H%M%S).zip"
```
- Simple retention (keep last 7 archives) on the backup server.
```bash
ssh $SSH_OPTS "${REMOTE_USER}@${REMOTE_HOST}" \
  "ls -1t ${REMOTE_BACKUP_DIR}/xfusioncorp_blog_*.zip | tail -n +8 | xargs -r rm -f"
```
- Add logging to `/var/log/blog_backup.log`.
- Notifications via Slack/email for failures.

---

### ✅ Outcome
The blog backup script works as expected:
- Local zip archive created in `/backup`
- Copied to Nautilus Backup Server at `/backup`
- Archive structure validated to include top-level `blog/`
