# **1) What I wanted to do :**

Cyberwatch kept showing me security flaws (CVE) on my Linux server, some really serious. Doing updates by hand every day takes time and I can forget it. So I made a script that updates the server by itself, every day.

# **2) Step 1 : the script**

```bash
mkdir -p /root/MajJordan
nano /root/MajJordan/ScriptMajApt.sh
```

Content of the script :

```bash
#!/bin/bash
apt update && apt full-upgrade -y
```

- `#!/bin/bash` : tells the system this is a bash script. Always the first line.
- `apt update` : checks what updates are available. Doesn't install anything yet.
- `apt full-upgrade -y` : installs all updates, and answers "yes" automatically since no one is there to confirm at night.

Nb : I used `full-upgrade` instead of `upgrade` because kernel security fixes often need new packages to be installed, and `upgrade` refuses to do that. `full-upgrade` doesn't have this problem.

# **3) Step 2 : make it executable**

```bash
chmod +x /root/MajJordan/ScriptMajApt.sh
```

This gives the script permission to run. Without it, the system refuses to execute the file.

# **4) Step 3 : test it by hand**

```bash
/root/MajJordan/ScriptMajApt.sh
```

I always test once manually before automating, to make sure there's no error.

# **5) Step 4 : schedule it with cron**

```bash
crontab -e
```

Line added :

```bash
0 20 * * * /root/MajJordan/ScriptMajApt.sh >> /root/MajJordan/ScriptMajApt.log 2>&1
```

- `0 20 * * *` : every day at 8 PM.
- The full path to the script is needed, `~` doesn't work well in cron.
- `>> file.log 2>&1` : saves everything the script prints, including errors, into a log file. Without this I would never know if something went wrong.

Check it's saved :

```bash
crontab -l
```

# **6) Step 5 : the reboot problem**

A kernel update only works after a reboot. Before that, the old (vulnerable) kernel is still running even if the fix is already installed.

```bash
[ -f /var/run/reboot-required ] && echo "Reboot needed"
```

This checks if a reboot is required. I chose not to reboot automatically in the script, since it's a production server and I want to control when it restarts.

# **7) What I learned**

- `apt upgrade` alone isn't enough for kernel security fixes, `full-upgrade` is needed.
- An update being installed doesn't mean it's active yet, the reboot matters too.
- Cyberwatch can show a flaw as "not fixed" even when it already is, so it's worth double checking before panicking.