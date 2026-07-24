# **1) What I wanted to do :**

I wanted to protect an API key sitting on a machine, instead of leaving it in plain text in a script's config file. I wrote a one-time setup script that asks for the key and encrypts it.

# **2) What the script does :**

- Loads DPAPI, since it is not autoloaded by default in either PowerShell edition.
- Checks it is running as Administrator.
- Asks for the secret value, hidden from the console.
- Turns it briefly into plain text, just long enough to encrypt it, then wipes that copy.
- Encrypts it with DPAPI, tied to this machine.
- Writes only the encrypted result to disk.
- Locks the destination folder down to SYSTEM and local admins only.
- Cleans up the plaintext copy left in memory.

# **3) Why DPAPI :**

- Windows ties the encryption to the machine's own identity : no password or master key for me to create, store, or rotate separately.
- The result only decrypts on the machine that encrypted it, so a copy of the file is useless anywhere else.
- Any account on that machine, including SYSTEM, can still decrypt it, which matters since the script reading the secret back runs as SYSTEM through Task Scheduler.

# **4) Why bother at all :**

- Without this, the key would sit as plain text, readable by anyone with filesystem access to that folder.
- Locking the folder's ACL to SYSTEM and admins only adds a second layer on top of the encryption itself.

Nb : the key itself should also be scoped as narrowly as possible (read-only, least privilege). Encryption protects where it's stored, not what it can do once used.