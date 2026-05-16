# akrb5 – Quick `krb5.conf` Manager for Active Directory Pentesting

![akrb5 banner](https://i.imgur.com/ASNB2Z5.png)

[![Version](https://img.shields.io/badge/version-1.5-blue)](https://github.com/AkiAfroo/akbr5)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

`akrb5` is a lightweight Bash utility that lets you manage **multiple custom `krb5.conf` files** — one per lab, client, or forest — without ever touching the system-wide `/etc/krb5.conf`.

Designed by and for red team operators who work with Kerberos on a daily basis.

---

## Why use akrb5?

* Eliminate repeated edits to `/etc/krb5.conf`
* Instantly switch labs with a single command
* Add or remove Domain Controllers dynamically
* Add DCs to any realm, not just the default one
* Automatically import all DCs discovered with **netexec** (`nxc ldap`) — no guessing, no heuristics
* `create` generates the `[domain_realm]` block automatically (required by Impacket, Certipy, etc.)
* Keep dozens of environments neatly organized under `~/akrb5/`
* Fully compatible with Impacket, Kerbrute, Certipy, Rubeus, and related tooling

---

## Installation

```bash
# Recommended (one-liner)
sudo curl -fsSL https://raw.githubusercontent.com/AkiAfroo/akbr5/refs/heads/main/akrb5 -o /usr/local/bin/akrb5
sudo chmod +x /usr/local/bin/akrb5

# Alternative: clone the repository
git clone https://github.com/AkiAfroo/akbr5.git
cd akbr5
sudo ln -s "$PWD/akrb5" /usr/local/bin/akrb5
```

All configuration files are stored in `~/akrb5/`.

> **Note:** The GitHub repository is named `akbr5` (historical typo). The tool and binary are correctly named `akrb5`.

---

## Usage

```bash
akrb5                                # Show active lab and list all available labs
source akrb5 <lab>                   # Recommended: persists KRB5_CONFIG in the current shell
akrb5 <lab>                          # Activate lab for the current subshell only

akrb5 create <lab> <IP> <REALM>      # Create a new lab (also generates [domain_realm])
akrb5 add-dc <lab> <IP>              # Add a DC to the lab's default realm
akrb5 add-dc <lab> <IP> <REALM>      # Add a DC to a specific realm
akrb5 remove-dc <lab> <IP>           # Remove a DC by IP
akrb5 remove-realm <lab> <REALM>     # Remove an entire realm (and its domain_realm entries)
akrb5 delete <lab>                   # Delete a lab completely

akrb5 import nxc-ldap <lab> <file>   # Generate krb5.conf from nxc ldap output
akrb5 list                           # List all labs
akrb5 --help | -h | help             # Show help
```

---

## Real engagement workflow

### Option A — Import from netexec LDAP (recommended)

LDAP (port 389) **only responds on Domain Controllers**. This makes `nxc ldap` the most reliable way to identify DCs — no heuristics, no guessing.

```bash
# 1. Scan LDAP — see output live and save to file at the same time
nxc ldap 10.2.10.0/24 | tee ldap.txt

# 2. Auto-create a krb5.conf with all discovered DCs
akrb5 import nxc-ldap contoso ldap.txt

# 3. Activate the lab — always use 'source' so KRB5_CONFIG persists in your shell
#    Without source, tools like Impacket and Certipy won't see the variable
source akrb5 contoso

# 4. Kerberos attacks just work
getTGT.py empire.local/user -dc-ip 10.2.10.5
kerbrute userenum -d empire.local --dc 10.2.10.5 users.txt
certipy find -u user@empire.local -p 'Passw0rd!'
```

### Option B — Build manually

```bash
# 1. Create the lab with the first DC
akrb5 create contoso 10.10.10.10 CONTOSO.LOCAL

# 2. Add more DCs as you discover them (default realm)
akrb5 add-dc contoso 10.10.10.20

# 3. Add a DC to a different realm (multi-forest)
akrb5 add-dc contoso 10.10.20.10 SUBSIDIARY.LOCAL

# 4. Activate
source akrb5 contoso
```

### Day-to-day management

```bash
# Check which lab is active and list all available ones
akrb5

# Switch to a different lab
source akrb5 otherlab

# A DC went down or is out of scope — remove it
akrb5 remove-dc contoso 10.10.10.20

# Drop an entire realm from a lab
akrb5 remove-realm contoso SUBSIDIARY.LOCAL

# End of engagement — delete the lab
akrb5 delete contoso
```

---

## Example generated `krb5.conf`

```ini
[libdefaults]
    default_realm = EMPIRE.LOCAL
    dns_lookup_realm = false
    dns_lookup_kdc = false
    ticket_lifetime = 24h
    forwardable = true

[realms]

    EMPIRE.LOCAL = {
        kdc = 10.2.10.5:88
        admin_server = 10.2.10.5
    }

    REBELS.LOCAL = {
        kdc = 10.2.10.7:88
        admin_server = 10.2.10.7
    }

[domain_realm]

    .empire.local = EMPIRE.LOCAL
    empire.local  = EMPIRE.LOCAL

    .rebels.local = REBELS.LOCAL
    rebels.local  = REBELS.LOCAL
```

---

## Multi-forest / multi-realm example

```bash
# Two separate forests in the same lab
akrb5 create ops 10.10.10.10 CORP.LOCAL
akrb5 add-dc ops 10.10.20.10 SUBSIDIARY.LOCAL   # adds SUBSIDIARY.LOCAL as a new realm
akrb5 add-dc ops 10.10.10.20                     # adds a second DC to the default realm (CORP.LOCAL)
```

---

## Changelog

### v1.5
- Replaced `import nxc-smb` with `import nxc-ldap`
- LDAP responds only on Domain Controllers — no heuristics, no signing flags, no guessing
- Cleaner and more reliable DC detection across any lab environment
- Use `tee` to see scan output live while saving to file

### v1.3 – v1.4
- `add-dc` accepts optional `<REALM>` to target a realm other than `default_realm`
- `create` generates `[domain_realm]` block automatically
- Fixed `add-dc`: safe insertion when the lab has multiple realms
- Fixed `remove-realm`: cleanly removes realm block and `[domain_realm]` entries

---

## Author

**Aki – @AkiAfroo**

Issues, pull requests, and feedback are welcome.
MIT License — fork, improve, and share.
If this tool saves you time during assessments, consider starring the repository.
