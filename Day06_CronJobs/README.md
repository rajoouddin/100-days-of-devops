## Day 6 – Create a Cron Job

### 🎯 Objective

Today I worked on setting up cron jobs to automate tasks in Linux. This is one of those foundational skills every DevOps engineer needs — whether it’s for backups, log rotation, or housekeeping scripts.

---

### 🔧 Steps I Took

**1. Installed Cron (cronie on CentOS)**
```bash
sudo yum install -y cronie
sudo systemctl enable crond
sudo systemctl start crond
systemctl status crond
```

**2. Created a simple cron job for the root user**

I added the following line to the root crontab using:
```bash
sudo crontab -e
```

Job:
```bash
*/5 * * * * echo hello > /tmp/cron_text
```
This writes hello into /tmp/cron_text every 5 minutes.

**3. Verified the job**
```bash
sudo crontab -l
cat /tmp/cron_text
```
---

### 📚 What I Learned
- `crontab -e` edits the user’s personal crontab (e.g., root’s jobs run as root).
- `/etc/crontab` and `/etc/cron.d/` allow system-wide jobs and include an extra username field to specify which user the job runs as.
- Example difference:
    - **User crontab:**
    ```bash
    0 2 * * * tar -czf /backups/etc_backup.tar.gz /etc
    ```

    - **System-wide (`/etc/crontab`):**
    ```bash
    0 2 * * * root tar -czf /backups/etc_backup.tar.gz /etc
    ```

- Always use absolute paths and redirect output to logs when running real jobs.
- Cron is great for quick automation, but in production we’d often manage these with Ansible, Puppet, or Kubernetes CronJobs for better visibility and control.

---

### 📝 Key Takeaways

- Cron uses 5 fields: minute, hour, day, month, weekday.
- `crontab -e` → per-user jobs, `/etc/crontab` → system-wide jobs.
- Best practice: always log output and test scripts manually before scheduling.
- Cron is simple but powerful — a silent automation hero in Linux systems.

---

### 💡 Reflection

Setting up cron felt like switching on the “autopilot” of Linux. It’s such a small thing, but it can save hours of repetitive work. I now understand the difference between per-user crontabs and system-wide crontabs, and I can confidently create jobs for both. This is the kind of detail I know will come up in real-world troubleshooting and even in interviews.

✅ One more day done. Small wins like these are stacking up — and I can feel the momentum building.