# **1) Files and folders**

- `pwd` : shows where I am right now.
- `ls` : lists what's in a folder.
- `ls -al` : same, but with details (rights, size, date) and hidden files too.
- `cd folder` : move into a folder. `cd ..` goes back one level.
- `mkdir name` : creates a folder.
- `mv` : renames or moves a file.
- `cp file copy` : copies a file.
- `rm file` : deletes a file (no trash bin, it's permanent).
- `cat file` : shows the full content of a file.
- `head -1 file` : shows just the first line (useful to check a shebang).
- `nano file` : opens a simple text editor.

# **2) Rights**

- `chmod +x file` : makes a file executable, needed to run a script.
- `ls -l file` : shows current rights (`rwxr-xr-x`).
- `whoami` : shows which user I'm logged in as.
- `sudo command` : runs one command as administrator.
- `su -` : switches completely to another user (usually root), keeps me in that session until I type `exit`.

# **3) Updates (Ubuntu/Debian)**

- `apt update` : checks what updates are available.
- `apt upgrade -y` : installs updates, but skips ones that need new packages (like kernel fixes).
- `apt full-upgrade -y` : installs everything, including kernel fixes.
- `[ -f /var/run/reboot-required ] && echo "Reboot needed"` : checks if a reboot is required after an update.

# **4) Automation (crontab)**

- `crontab -e` : add or edit scheduled tasks.
- `crontab -l` : show scheduled tasks.

Format of a line :

```
minute hour day month weekday command
```

- `0 20 * * *` : every day at 8 PM.
- `*` means "every".
- Always use the full path to the script (`/root/script.sh`, not `~/script.sh`).

# **5) Saving logs**

```bash
command >> file.log 2>&1
```

- `>>` : adds the result to the end of the file instead of erasing it.
- `2>&1` : also saves the errors, not just the normal messages.

# **6) The shebang**

- `#!/bin/bash` : must be the first line of a script, tells the system how to run it.