---
title: 'Measuring Link Speed and path quality on Linux via speedtest CLI and mtr'
summary: 'Measuring Link Speed and path quality on Linux via speedtest CLI and mtr'
date: '2026-07-06T00:00:00+00:00'
---

# Measuring Link Speed and path quality on Linux via speedtest CLI and mtr

When a connection feels slow on your linux box/vps, you usually need two separate answers. How much bandwidth do I actually get, and where along the path does the trouble start. `speedtest` answers the first. `mtr` answers the second. This guide walks through installing the official speedtest CLI from its repository, pulling the real hostname or IP of the server you tested against, and running `mtr` both in live mode and in report mode.

Everything here assumes a Debian or Ubuntu box with `sudo` access.

## Install the Ookla speedtest CLI from the repo

Two different programs go by the name "speedtest." The one packaged in `apt` as `speedtest-cli` is a community Python tool, and it tends to report low numbers on fast links. The official Ookla build is more accurate and is what you want for a real measurement. It ships from Ookla's own package repository:

```bash
curl -s https://packagecloud.io/install/repositories/ookla/speedtest-cli/script.deb.sh | sudo bash
sudo apt-get install -y speedtest
```

The first command adds the repo and its signing key. The second pulls the binary. Confirm it landed:

```bash
speedtest --version
```

The first time you run it, accept the license prompt (or pass `--accept-license --accept-gdpr` to skip it in scripts).

### Running a test

With no arguments, it picks the nearest server and measures:

```bash
speedtest
```

To choose a server yourself, list what is nearby and pin one by ID:

```bash
speedtest -L                 # list nearby servers with their IDs
speedtest -s <server_id>     # test against a specific server
```

For anything you plan to parse or log, use JSON output:

```bash
speedtest -f json
```

One habit worth keeping on a gateway or router: run the test from the box itself first, then from a client behind it. The box measures raw upstream. The client measures the full path including any tunnel or NAT in between. The gap between the two numbers is your overhead.

## Find the IP or hostname of the speedtest server

You need this because `mtr` traces to a host, not to a speedtest server ID. The catch is that the community `speedtest-cli` list only prints the sponsor and city, not the host. A typical listing looks like this:

```
12345) ExampleNet (Springfield) [0.57 km]
23456) Metro ISP (Springfield) [1.54 km]
34567) Regional Fiber (Shelbyville) [6.48 km]
```

There are two clean ways to get hostname.

### Option A: the Ookla CLI, which prints the host directly

If you installed the official binary above, ask for JSON and read the server host out of it:

```bash
speedtest -s 12345 -f json | jq -r '.server.host'
```

That returns something like `speedtest1.example.net:8080`. The part before the colon is your `mtr` target. The `8080` is the port speedtest actually uses, which matters for a realistic trace (more on that below).

### Option B: a one-liner against the Python module

If you only have the community `speedtest-cli` installed, its Python module still knows the host even though the command line hides it. Ask it directly:

```bash
python3 - <<'EOF'
import speedtest
s = speedtest.Speedtest(secure=True)
servers = s.get_servers([12345])       # the ID you care about
for dist in servers:
    for srv in servers[dist]:
        print(srv['id'], srv['sponsor'], srv['host'])
EOF
```

Output:

```
12345 ExampleNet speedtest1.example.net:8080
```

### Turning the host into a raw IP

`mtr` accepts a hostname, so you rarely need this. When you do want the address, resolve it:

```bash
dig +short speedtest1.example.net
```

Before tracing, it is worth a two-line sanity check that the name resolves and the port answers. If either fails, `mtr` has nothing to reach and you will waste time staring at an empty trace:

```bash
dig +short speedtest1.example.net
nc -vz -w3 speedtest1.example.net 8080
```

## Run mtr in live mode

Live mode is the default. It redraws the screen every second and fills in each hop as replies arrive, which is the fastest way to confirm the trace is working and to watch latency move in real time.

```bash
sudo mtr -zbT -P 8080 speedtest1.example.net
```

The flags:

- `-z` shows the ASN (which network owns each hop)
- `-b` shows both the IP and the resolved hostname
- `-T` uses TCP probes instead of ICMP
- `-P 8080` sends those probes to port 8080, the same port speedtest uses

Press `q` to quit once you have seen enough.

Why TCP probes to port 8080? ICMP and TCP can take different paths and get different treatment from routers and firewalls. Probing the actual port your traffic uses gives you a trace closer to reality. The tradeoff is that many intermediate routers do not answer TCP probes the way they answer ICMP, so middle hops often show as `???`. That is normal and does not mean the path is broken. If you only care whether the host is reachable at all, drop `-T -P 8080` and let it use plain ICMP.

## Run mtr in report mode (non live)

Report mode is what you paste into a ticket or a chat. It sends a fixed number of probes, stays silent while it works, then prints one clean table at the end.

```bash
sudo mtr -rwzbc 100 -T -P 8080 speedtest1.example.net
```

The added flags:

- `-r` report mode, print once at the end
- `-w` wide output so long hostnames are not truncated
- `-c 100` send 100 cycles

The gotcha that trips everyone: report mode prints nothing until it finishes. At roughly one packet per second, `-c 100` means about a hundred seconds of a blank screen with only a `Start:` line showing. It looks frozen. It is not. If the silence bothers you, or you just want a quick sample, shorten both the count and the interval:

```bash
sudo mtr -rwzbc 30 -T -P 8080 -i 0.2 speedtest1.example.net
```

`-c 30 -i 0.2` is thirty probes at a fifth of a second each, so the whole run takes about six seconds.

## Reading the output

Two columns carry most of the signal: `Loss%` and the latency spread (`Avg`, `Best`, `Wrst`, `StDev`).

For loss, the rule is that only loss which persists all the way to the final hop counts. A middle hop showing 90% or even 100% loss while the hops after it recover to 0% is just that router deprioritizing probe replies to protect its own CPU. Your actual packets are getting through. Read the last hop to know whether you have real end-to-end loss.

For latency, watch where the numbers first jump. A hop where `Best` is low but `Avg` and `Wrst` are far higher, with a large `StDev`, is queuing delay, and it is inherited by every hop after it. If that jump appears at the first hop, the delay is being added right at your edge (your gateway, your tunnel, or your first upstream), not out in the wider network. If the early hops stay tight and the jump only shows up deep in the path, the problem is farther away and probably not yours to fix.

## Quick reference

```bash
# Install the official CLI
curl -s https://packagecloud.io/install/repositories/ookla/speedtest-cli/script.deb.sh | sudo bash
sudo apt-get install -y speedtest

# Test
speedtest -L
speedtest -s <id>
speedtest -s <id> -f json | jq -r '.server.host'

# Live trace
sudo mtr -zbT -P 8080 <host>

# Report trace (fast sample)
sudo mtr -rwzbc 30 -T -P 8080 -i 0.2 <host>
```
