# akrb5 – Quick `krb5.conf` Manager for Active Directory Pentesting

![akrb5 banner](https://i.imgur.com/ASNB2Z5.png)

[![Version](https://img.shields.io/badge/version-1.3-blue)](https://github.com/AkiAfroo/akbr5)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

`akrb5` is a lightweight Bash utility that lets you manage **multiple custom `krb5.conf` files** — one per lab, client, or forest — without ever touching the system-wide `/etc/krb5.conf`.

Designed by and for red team operators who work with Kerberos on a daily basis.

---

## Why use akrb5?

* Eliminate repeated edits to `/etc/krb5.conf`
* Instantly switch labs with a single command
* Add or remove Domain Controllers dynamically
* Add DCs to any realm, not just the default one
* Automatically import all domains and DCs discovered with **netexec** (`nxc smb`)
* `create` now generates the `[domain_realm]` block automatically (required by Impacket, Certipy, etc.)
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
akrb5 <lab>                          # Activate a lab (exports KRB5_CONFIG for this session)
source akrb5 <lab>                   # Recommended: persist KRB5_CONFIG in the current shell

akrb5 create <lab> <IP> <REALM>      # Create a new lab (also generates [domain_realm])
akrb5 add-dc <lab> <IP>              # Add a DC to the lab's default realm
akrb5 add-dc <lab> <IP> <REALM>      # Add a DC to a specific realm
akrb5 remove-dc <lab> <IP>           # Remove a DC by IP
akrb5 remove-realm <lab> <REALM>     # Remove an entire realm (and its domain_realm entries)
akrb5 delete <lab>                   # Delete a lab completely

akrb5 import nxc-smb <lab> <file>    # Generate krb5.conf from nxc smb output
akrb5 list                           # List all labs
akrb5 --help | -h | help             # Show help
```

---

## Real engagement workflow

```bash
# 1. Scan the network
nxc smb 10.2.10.0/24 -u '' -p '' > nxc.txt

# 2. Auto-create a krb5.conf with all discovered domains and DCs
akrb5 import nxc-smb contoso nxc.txt

# 3. Activate the lab (use source to persist the environment)
source akrb5 contoso

# 4. Kerberos attacks just work
getTGT.py contoso.local/user -dc-ip 10.10.10.10
kerbrute userenum -d contoso.local --dc 10.10.10.10 users.txt
certipy find -u user@contoso.local -p 'Passw0rd!'
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
    megacorp.local  = MEGACORP.LOCAL
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

### v1.3
- `add-dc` now accepts an optional `<REALM>` argument to target a realm other than `default_realm`
- `create` now generates the `[domain_realm]` block automatically
- Fixed `add_kdc`: safe multi-realm insertion via Python instead of fragile `sed` ranges
- Fixed `remove_realm`: now cleanly removes the realm block and its `[domain_realm]` entries via Python

### v1.2
- Initial public release

---

## Author

**Aki – @AkiAfroo**

Issues, pull requests, and feedback are welcome.
MIT License — fork, improve, and share.
If this tool saves you time during assessments, consider starring the repository.
