# **1) What I wanted to do :**

I wanted to protect an API key sitting on a machine, instead of leaving it in plain text in a script's config file. I picked DPAPI, Windows' built-in encryption, because it does not need a separate password or key to manage : Windows ties the encryption to the machine itself, so only that machine can decrypt the value back later. I wrote a one-time setup script to ask for the key and encrypt it.

# **2) What the script does :**

- Loads DPAPI, since it is not autoloaded by default in either PowerShell edition.
- Checks it is running as Administrator.
- Asks for the secret value, hidden from the console.
- Turns it briefly into plain text, just long enough to encrypt it, then wipes that copy.
- Encrypts it with DPAPI, tied to this machine.
- Writes only the encrypted result to disk.
- Locks the destination folder down to SYSTEM and local admins only.
- Cleans up the plaintext copy left in memory.

# **3) First error :**

Running it gave me :

```
[ERROR] Failed to encrypt/store the secret: Type [Security.Cryptography.ProtectedData] introuvable.
```

I assumed it was a PowerShell edition problem, since the prompt showed I was in pwsh instead of Windows PowerShell 5.1. Forcing 5.1 gave the exact same error, so that theory was wrong.

# **4) What I learned :**

- Testing a theory fast, even when it turns out wrong, beats sitting on a guess I never actually check.
- The error looked like a shell problem, but the real issue was one layer below that : how the underlying pieces get loaded before they can be used.
- Once I stopped assuming and looked at where the error really came from, the fix took a single line.