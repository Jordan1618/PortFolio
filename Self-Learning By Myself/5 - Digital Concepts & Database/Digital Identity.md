# 🌐 Digital Identity & InfoSec — Course Notes

> **Quick reference for the key concepts seen in lab.

---

## 🏢 Corporate Digital Identity

What officially represents a company online:

- Domain name, SSL certificate, legal notices
- Professional email (`@company.com`)
- Official social media accounts, logo

Not included: passwords, VPN, DNS server, intranet — these are internal tools, not public identity.

---

## 🎭 Social Media Leaks

Employees post without thinking. The danger is rarely what they say — it's what's visible in the background.

Classic mistakes:

- A photo of a desk with a sticky note containing a password
- A video call where Trello or a spreadsheet is visible
- A code snippet with an API key still in it
- Posting your schedule publicly (makes social engineering easy)

> Rule: before posting anything work-related, check the background.

---

## 🔍 Phishing — Spotting a Fake Domain

HTTPS padlock = encryption only. It does NOT mean the site is legit.

Three things to check:

- **Domain:** is it the simplest version? `company.com` is real. `company-secure-login.net` is not.
- **Certificate:** issued by a known authority? Self-signed = red flag.
- **Whois:** old domain with an identified owner = good sign. Created last week + anonymous = suspicious.

Common tricks: replacing `i` with `1` (`enterpr1se.com`), adding words (`-portal`, `-secure`), using `.info` or `.support` instead of `.com`.

---

## 🕵️ OSINT — Finding Someone from a Username

**Case: `Atelokynus2`** — fictional example, educational use only.

#### 🔎 Step 1 — Where does the username exist?

bash

```bash
sherlock Atelokynus2
```

Result: found on GitHub, Reddit, Steam. Not on Instagram or LinkedIn.

#### 🔎 Step 2 — Find a linked email

bash

```bash
holehe atelokynus2@gmail.com
```

Result: account exists on Discord and Spotify.

#### 🔎 Step 3 — Dig into the profiles

On **GitHub**: the commit email is visible in the git history → `atelokynus2@proton.me`

On **Reddit**: posts only in the evening, only in French subreddits → French speaker, UTC+1

bash

```bash
exiftool profile_picture.jpg
# GPS coordinates found in the image metadata
```

#### 🔎 Step 4 — Check infrastructure

bash

```bash
whois atelokynus2.fr
shodan host 51.68.XX.XX
# Port 8080 open — unauthenticated dev server exposed to the internet
```

#### 🛡️ Lessons

|What was found|How to avoid it|
|---|---|
|Email in git commits|Use a fake email for public repos|
|Timezone from posting times|Use different hours or UTC|
|Same username everywhere|Use different aliases per platform|
|Dev server exposed|Never expose a test server without a firewall|

---

## ✍️ Electronic Signatures — What to Check

Four things, in order:

1. **Hash match?** — document hash must equal signature hash. If not → forged.
2. **Certificate valid?** — signing date must be inside the certificate validity window.
3. **Right person?** — the signer's name must match the `CN=` field in the certificate.
4. **Algorithm safe?** — SHA256 is fine. SHA1 is weak. MD5 is broken, never trust it.

Bonus: if the timestamp is missing, the date can be faked. If it's invalid, something was tampered with.

---

## 📬 SPF / DKIM / DMARC

Three protocols that answer three questions about an incoming email.

|Protocol|Question|Checks|
|---|---|---|
|SPF|Is this IP allowed to send for this domain?|DNS record vs sending IP|
|DKIM|Was the content modified in transit?|Cryptographic signature|
|DMARC|What to do if SPF or DKIM fails?|Policy: none / quarantine / reject|

```
SPF fail + DKIM fail + DMARC reject  ->  delete it, it's phishing
SPF pass + DKIM pass + DMARC pass    ->  deliver
SPF pass + DKIM fail + DMARC quarantine  ->  move to spam
```

An email pretending to be from a bank with SPF fail is almost always phishing. Real banks have strict configs.

---

## 📥 Exim4 — Basic Commands

bash

```bash
exim -bp          # see the queue
exim -M <ID>      # deliver
exim -Mrm <ID>    # delete
exim -Mf <ID>     # freeze / quarantine
```

Private IPs (`192.168.x.x`, `10.x.x.x`) = internal, more trustworthy. IPs like `203.0.113.x` = RFC documentation ranges, suspicious in production.

---

## 🎯 Key Points to Remember

- HTTPS padlock ≠ safe site
- Real domain = shortest version, no extra words
- Never reuse the same username across platforms
- SHA256 is fine. MD5 is dead.
- SPF = origin, DKIM = integrity, DMARC = policy
- The most dangerous leaks are the ones in the background of a photo