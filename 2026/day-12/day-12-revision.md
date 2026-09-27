
# Day 12 – Breather & Revision (Days 01–11)

## 🎯 Goal
Take a one-day pause to revise and strengthen the Linux fundamentals learned during Days 01–11.

---

# 🧠 Mindset & Plan Review

- My goal is still to build strong Linux and DevOps fundamentals.
- I realized consistency and daily command practice are more important than theory only.
- I need to improve speed and confidence while using Linux commands.
- Going forward, I will focus more on hands-on practice and troubleshooting scenarios.

---

# ⚙️ Processes & Services Practice

## Commands Practiced

```bash
ps aux
systemctl status ssh
journalctl -u ssh
```

## Observations

- `ps aux` displays all running processes with CPU and memory usage.
- `systemctl status ssh` helps check whether the SSH service is active or inactive.
- `journalctl -u ssh` shows logs useful for debugging services.

---

# 📁 File Skills Practice

## Commands Practiced

```bash
echo "Hello DevOps" >> notes.txt
chmod 755 notes.txt
ls -l
cp notes.txt backup.txt
mkdir practice-directory
```

## Learnings

- `echo >>` appends text safely into files.
- `chmod 755` gives proper execute permissions.
- `ls -l` verifies permissions and ownership.
- `cp` creates backups quickly.
- `mkdir` creates directories efficiently.

---

# 📌 Cheat Sheet Refresh

## 5 Commands I Would Use First During an Incident

| Command | Purpose |
|----------|----------|
| `ls -l` | Check permissions and ownership |
| `ps aux` | View running processes |
| `systemctl status` | Check service health |
| `journalctl` | Read service/system logs |
| `chmod` | Fix permission issues |

---

# 👤 User & Group Practice

## Commands Practiced

```bash
sudo useradd devuser
id devuser
sudo chown devuser notes.txt
ls -l
```

## Learnings

- `id` verifies user details and groups.
- `chown` changes file ownership safely.
- `ls -l` confirms ownership changes.

---

# ✅ Mini Self-Check

## 1) Which 3 commands save you the most time right now, and why?

### `ls -l`
Quickly checks permissions and ownership.

### `systemctl status`
Instantly checks service health.

### `ps aux`
Shows all active processes running in the system.

---

## 2) How do you check if a service is healthy?

```bash
systemctl status <service-name>
ps aux | grep <service-name>
journalctl -u <service-name>
```

---

## 3) How do you safely change ownership and permissions without breaking access?

```bash
sudo chown user:group file.txt
chmod 755 file.txt
```

- Always verify changes using `ls -l`
- Avoid unnecessary `777` permissions

---

## 4) What will you focus on improving in the next 3 days?

- Shell scripting basics
- Linux troubleshooting practice
- Faster command usage
- Understanding logs and permissions deeply

# 🛠️ Basic Linux Troubleshooting Flow

When something is not working on a server, don't randomly run commands.

Use a structured approach:

             PROBLEM
                │
                ▼
       Check the service
       systemctl status
                │
                ▼
          Check logs
          journalctl
                │
                ▼
       Check processes
           ps / top
                │
                ▼
       Check resources
      df / free / top
                │
                ▼
       Check networking
       ip / ss / curl
                │
                ▼
          Find the cause


# 🚀 Key Takeaways

- Revision is important for long-term retention.
- Linux commands become easier through repetition.
- Logs and permissions are critical for troubleshooting.
- Small daily practice builds strong DevOps foundations.




#90DaysOfDevOps #DevOpsKaJosh #TrainWithShubham
