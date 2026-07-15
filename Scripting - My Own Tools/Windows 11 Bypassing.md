# **1) The context & Explanations :

Some computers stopped updating their system updates. Why does it happened and how solve it ?

There was different problems :
TPM 2.0 was there but undetected
The CPU was a little bit old
The Windows 11 was locked in 21H2 or 22H2 version

They blocked updates system so I have to change something :
Unblock Windows Update auto-update
Inject keys into the regedit
Mount W11 Iso (the official is working but you need to download it on another SDD then plug this SSD on your computer / copy it on the desktop (faster than downloading it))
Test if it's working

# **2) How I solve it :

I used the script in [Tool --- Unblock Windows Update V2 (Strengthen)](Tool%20---%20Unblock%20Windows%20Update%20V2%20(Strengthen).md) to reset correctly right keys inside the regedit. Next I used the [Tool --- Inject Keys](Tool%20---%20Inject%20Keys.md) to correct the system auto-blocking for Windows 11 new version update.

For 1 PC I had to create my own [Tool --- Inject Strengthen Keys](Tool%20---%20Inject%20Strengthen%20Keys.md) to bypass a very complex device.

# **3) How have built my tools ?

Very important : I create/created my tools based on what I needed and what I want to learn. Having its own tools is a benediction for everybody who want to learn. After I test, I shared them with a procedure to all my team.
