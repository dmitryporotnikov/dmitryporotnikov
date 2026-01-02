---
title: "Migrate config from one mikrotik router to another"
summary: "Migrate config from one mikrotik router to another"
date: '2026-01-02T00:00:00+00:00'
draft: false
---

### On the OLD router (Winbox)
1. Connect with **Winbox**
2. Open **New Terminal**
3. Run 

**clean export**

`/export compact file=old-router`

This creates: **`old-router.rsc`** (in **Files**). Export/import is done via CLI (Terminal) even if you’re using Winbox

4. Open **Files** in Winbox → find `old-router.rsc` → **drag it to your PC** (or right-click download).

### On the NEW router (Winbox)
Start from an empty config so you don’t “merge” weird defaults.
- Winbox → **System → Reset Configuration**
- Tick **No Default Configuration**
- Reconnect (often easiest via **MAC address** in Winbox)

Then:

- Open **Files** → **upload** `old-router.rsc` (drag & drop into the Files window)
- Open **New Terminal**
- Import it:
- 
```
/import file-name=old-router.rsc
```

(Depending on RouterOS version, `import old-router.rsc` may also work, but `file-name=` is explicit.)

**The import may change IPs/firewall and you can lose the session**
If that happens, reconnect via **Winbox MAC** (L2) and continue.

Reboot after import:
/system reboot

### Things to be aware of:
1. Match RouterOS major versions if possible
2. If migrating to different hardware, you may need to edit the `.rsc`, e.g if interface names/ports differ (ether1/ether2, sfp, wifi)