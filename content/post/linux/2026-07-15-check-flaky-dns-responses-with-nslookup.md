---
title: 'Checking for flaky DNS responses with nslookup'
summary: 'A short bash loop that probes a DNS server at regular intervals to surface intermittent resolution failures.'
date: '2026-07-15T12:00:00+00:00'
---

## Checking for flaky DNS responses with nslookup

When an application sporadically fails to connect, the problem is often the resolver rather than the application itself. A DNS server that occasionally returns `SERVFAIL`, `NXDOMAIN`, or a stale record is easy to miss with a single `nslookup`. Repeating the query over a window of time may show the inconsistency.

The script below targets a specific DNS server and hostname, runs `nslookup` every 10 seconds, and stops after 20 minutes. The output is a timestamped trace you can scan for dropped queries, slow responses, or mismatched answers.

```bash
#!/usr/bin/env bash
# Query cosmosdb.database.windows.net against DNS server 10.0.0.1
# every 10 seconds for 20 minutes.

server="10.0.0.1"
host="myawesomefqdn.database.windows.net"
interval=10          # seconds between queries
duration=$((20*60))  # total run time in seconds (1200)

end=$(( $(date +%s) + duration ))

while [ "$(date +%s)" -lt "$end" ]; do
    echo "===== $(date '+%Y-%m-%d %H:%M:%S') ====="
    nslookup "$host" "$server"
    echo
    sleep "$interval"
done
```

To run it, save the script, make it executable with `chmod +x`, and run it from a host on the same network segment as the resolver. While the loop runs, look through the output for entries that differ from the majority: `connection timed out`, `server can't find`, a missing `Address:` line, or a noticeably longer round-trip. 

If you want an output to the file -> 
``` 
scipt.sh | tee /home/user/dnsloop_$(date +%Y%m%d_%H%M%S).log
```
