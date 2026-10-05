# **1) What I wanted to do :**

I am learning Russian and I wanted a Russian keyboard that fits my own habits. The standard Russian layout puts the Cyrillic letters in a different order than the keys I know. So I used **MSKLC (Microsoft Keyboard Layout Creator)**, a free Microsoft tool, to build my own layout: a **phonetic** one, where each Cyrillic letter sits on the Latin key that sounds like it.

The installer (`MSKLC.msi` and `setup.exe`) is in my MSKLC folder.

# **2) How I built it :**

- I opened a layout in MSKLC and assigned a Cyrillic character to each key, for each state (normal and Shift)
- I named it "Russe Phonétique Jordan"
- The tool checks the layout, then builds an installer with a DLL file for it
- After the install, Windows shows it as a new Russian keyboard

# **3) Checking that it was installed :**

I looked in my Windows settings to confirm. In the language list, French (fr-FR), English (en-US) and Russian (ru) are there, and the Russian one uses my custom layout.

In the registry:

```powershell
Get-ItemProperty 'HKCU:\Keyboard Layout\Substitutes'
```

Result: `00000419 : a0000419`. It means the standard Russian layout (`0419`) is replaced by my custom layout (`a0000419`).

```powershell
Get-ChildItem 'HKLM:\SYSTEM\CurrentControlSet\Control\Keyboard Layouts' | Where-Object PSChildName -eq 'a0000419'
```

This key holds the name "Russe Phonétique Jordan" and the file `Rus_Jor.dll`. The file is copied in `C:\Windows\System32` and `C:\Windows\SysWOW64`, so both 64-bit and 32-bit programs can use it.

Nb : `SysWOW64` is the folder for 32-bit programs on a 64-bit Windows.

# **4) What I learned :**

- A keyboard layout is a table: key, state, character. Windows loads it from a small DLL
- A custom layout is installed like a normal Windows component, with its own ID in the registry (a-prefix for custom layouts)
- A phonetic layout helps me type Russian without looking at the keys, so it supports my learning
