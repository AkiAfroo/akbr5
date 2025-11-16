# akbr5 – Quick krb5.conf manager for Active Directory pentesting

[![Version](https://img.shields.io/badge/version-1.0-blue)](#)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

`akbr5` is a lightweight Bash tool that allows you to manage **multiple custom `krb5.conf` files** (one per lab, client or forest) without ever touching the system-wide `/etc/krb5.conf`.

Designed by and for red teamers who live in Kerberos every day.

---

## Why akbr5?

- No more `sudo nano /etc/krb5.conf` when switching labs  
- Instantly activate a lab with a single command  
- Add/remove Domain Controllers on the fly  
- Auto-import every domain & DC discovered with **netexec** (`nxc smb ...`)  
- Keep dozens of environments cleanly organized in `~/akrb5/`  

---

## Installation

```bash
# Recommended – one-liner
sudo curl -sL https://raw.githubusercontent.com/AkiAfroo/akbr5/main/akbr5 -o /usr/local/bin/akbr5
sudo chmod +x /usr/local/bin/akbr5

# Or clone it
git clone https://github.com/AkiAfroo/akbr5.git
cd akbr5
sudo ln -s "$PWD/akbr5" /usr/local/bin/akbr5
```

All configs are stored in `~/akrb5/`.

---

## Usage

```bash
akbr5                     # Show active lab + list all labs
akbr5 corp                # Activate lab "corp"
source akbr5 corp         # ← Recommended: keeps KRB5_CONFIG in current shell

akbr5 create evilcorp 10.0.0.5 EVILCORP.LOCAL
akbr5 add-dc evilcorp 10.0.0.6
akbr5 remove-dc evilcorp 10.0.0.6

akbr5 import nxc-smb customer1 nxc_output.txt   # ← Auto-generates krb5.conf from nxc/cme output

akbr5 list
akbr5 delete oldlab
akbr5 --help | -h
```

---

## Real engagement workflow

```bash
# 1. Scan the network
nxc smb 172.16.0.0/16 -u '' -p '' > nxc.txt

# 2. Auto-create perfect krb5.conf with every discovered domain/DC
akbr5 import nxc-smb customer1 nxc.txt

# 3. Activate (use source!)
source akbr5 customer1

# 4. Kerberos attacks just work
getTGT.py customer1.local/user
kerbrute userenum -d customer1.local --dc dc01.customer1.local users.txt
certipy find -u user@customer1.local -p Passw0rd!
```

---

## Example generated `krb5.conf`

```ini
[libdefaults]
    default_realm = MEGACORP.LOCAL
    dns_lookup_realm = false
    dns_lookup_kdc = false
    ticket_lifetime = 24h
    forwardable = true

[realms]
    MEGACORP.LOCAL = {
        kdc = 10.10.10.10:88
        kdc = 10.10.10.20:88
        admin_server = 10.10.10.10
    }

[domain_realm]
    .megacorp.local = MEGACORP.LOCAL
    megacorp.local = MEGACORP.LOCAL
```

---

## Author

**Aki – @AkiAfroo**

Feel free to open issues, PRs or just say hi!  
MIT License – fork / improve / share ♥  
⭐ Star if this saves you time during assessments!

