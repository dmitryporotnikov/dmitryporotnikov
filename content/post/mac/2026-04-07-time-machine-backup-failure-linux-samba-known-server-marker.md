---
title: "Time Machine backup failure to Linux/Samba target - Missing kown server marker"
summary: "How a missing server marker in macOS caused Time Machine to fail authentication against a Samba share, and how to fix it"
date: '2026-04-07T00:00:00+00:00'
draft: false
---

## Time Machine backup failure to Linux/Samba target - Missing kown server marker

One of my Macs refused to start Time Machine backups to a Linux/Samba target, even though the same SMB share opened manually without issues, the same credentials worked from other Macs, and manual SMB mounting worked fine from the affected machine. Time Machine showed an invalid username/password error, but this was misleading. The real issue was macOS not treating the SMB host as a known server in the `NetAuthSysAgent` / `backupd` system context.



Time Machine reported a generic username/password failure, but the real issue was different. When `backupd` tried to mount the SMB destination in system/root context, macOS did not consider `192.168.88.99` a known server. Because of that, saved credentials were not loaded into the authentication path used by Time Machine.

The logs showed:

```text
NAOpenSessionAsync reports sessionStatus: 0
NAMountAsync reports mountStatus: 0
```

This proved the SMB share could be reached and authenticated interactively. But when the backup started:

```text
NAConnectToServerSync failed with error: 80
Authentication error (80)
BACKUP_FAILED_AUTHENTICATION_ERROR (29)
```

Then `NetAuthSysAgent`:

```text
GetKnownServerMarker: enter for host "192.168.88.99"
isKnownServer 0
There are no user credentials to add to the MechType session
OpenSession failed 80
```

### Solution

The fix required two steps. First, add the server marker manually to the root server markers plist:

```bash
PL='/private/var/root/Library/Group Containers/group.com.apple.NetworkAuthorization.ServerMarkers/serverMarkers.plist'

sudo cp "$PL" "$PL.bak.$(date +%Y%m%d-%H%M%S)"
sudo /usr/libexec/PlistBuddy -c 'Print' "$PL"

sudo /usr/libexec/PlistBuddy -c 'Add :"192.168.88.99" bool true' "$PL" 2>/dev/null || \
sudo /usr/libexec/PlistBuddy -c 'Set :"192.168.88.99" true' "$PL"

sudo plutil -p "$PL"
```

Second, remove and re-add the Time Machine destination. The existing destination had stale state from the broken authentication flow. Removing it and adding it again forced Time Machine to re-register the target cleanly.

### Key commands

To diagnose similar issues, stream Time Machine and backup logs:

```bash
log stream --info --predicate 'process == "backupd" OR process == "backupd-helper" OR process == "smbd" OR subsystem CONTAINS "TimeMachine"'
```

For NetAuthSysAgent issues:

```bash
log show --last 10m --style compact --predicate 'process == "NetAuthSysAgent" OR process == "backupd" OR eventMessage CONTAINS[c] "NetAuth"'
```

To inspect the server marker plist:

```bash
sudo plutil -p '/private/var/root/Library/Group Containers/group.com.apple.NetworkAuthorization.ServerMarkers/serverMarkers.plist'
```
