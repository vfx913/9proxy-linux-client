# 9proxy linux: Install the Client on Ubuntu or RHEL, Drive Proxies from the Terminal, and Pick the Plan That Fits

Most people searching "9proxy linux" are stuck on one of two questions. Either they want to know whether a Linux build exists at all, or they already have the client running and would rather script it than click around a desktop app. Both are reasonable, and the answers are different depending on which proxy model you bought — which is the part that trips people up.

So let's start there, because it decides everything else.

## 9Proxy ships two proxy models, and only one of them needs a client on your machine

The same dashboard sells two things that behave nothing alike:

**Residential Proxy by IPs.** You buy a fixed number of IPs. An IP is only deducted when you forward it to a local port, and unused IPs sit in your balance indefinitely. Forwarding happens inside the 9Proxy app, which exposes each proxy on `localhost:port`. Once activated, an IP stays online for a few hours up to roughly 24 hours, because that's the natural uptime of a real residential connection. Bandwidth through an active IP is unmetered.

**Residential Proxy by GB.** You buy a traffic balance and generate endpoints straight from the dashboard. No app involved. Authentication is username/password or an IP whitelist. Sessions can be sticky or rotating, you can generate as many endpoints as you like, and traffic is valid for 180 days (unlimited for Enterprise). You only get charged for data consumed.

That difference is the whole ballgame on a server. If you're working on a headless VPS with no desktop environment, the GB-based model requires nothing installed — you point your scraper at the generated endpoint and go. The IP-based model expects the 9Proxy app to be running locally and holding the port forward open.

9Proxy's own documentation is explicit that the CLI quick commands are intended for "automation, scripting, or quick setups on headless Linux servers," which tells you they know who's asking.

## Installing the Linux client

Two packages are published, both built for amd64:

- `9proxy-linux-debian-amd64.deb` — Debian, Ubuntu and derivatives
- `9proxy-linux-redhat-amd64.rpm` — RHEL, CentOS, Fedora, Rocky, Alma

Download whichever matches your distro, then install it locally rather than from a repo:

bash
sudo apt install ./9proxy-linux-debian-amd64.deb


bash
sudo yum install 9proxy-linux-redhat-amd64.rpm


On Debian-based systems the installer may throw up a setup screen — select OK and it continues. Nothing exotic happens here; you're installing a systemd-managed service plus the `9proxy` CLI binary.

Worth flagging: the published package names only carry `amd64`. If you're on ARM (a Raspberry Pi, an Ampere instance, Apple silicon under a Linux VM), there's no documented build for you, and the GB-based model is the sensible route since it skips the client entirely.

## Keeping it alive with systemd

The install registers `9proxyd.service`. Start it, then enable it so it survives reboots:

bash
sudo systemctl start 9proxyd.service
sudo systemctl enable 9proxyd


`start` runs it now. `enable` makes sure it comes back after a reboot — and if you skip `enable`, you'll be re-running the start command every time the box restarts, usually at the least convenient moment.

## Logging in and talking to the CLI

Once the service is up, the binary exposes a handful of subcommands. The useful ones:

| Command | What it does |
| --- | --- |
| `9proxy -h` | Lists all available commands and parameters |
| `9proxy auth` | Signs into your account |
| `9proxy port` | Configures and manages the local port range |
| `9proxy proxy` | The main command: view, filter and forward proxies |
| `9proxy setting` | Opens 9Proxy's settings |
| `9proxy api` | API access |

Authenticate first. Then — and this bites people who skip it — configure your port range with `9proxy port` before trying to forward anything. The docs call this out as a prerequisite, and a forward command against an unconfigured range just won't land anywhere useful.

### The interactive list

bash
9proxy proxy -u


This opens the proxy forwarding interface and shows the IPs available in your account. Press `F` to filter by country, state, city, ZIP code or ISP, then confirm to apply. Pick a proxy with the arrow keys and hit Enter; you'll be offered two choices — forward it to every port in your configured range, or forward it to one specific port.

### Forwarding without the interface

For scripting, skip the TUI entirely:

bash
9proxy proxy -c US -p 60000


That binds a US residential IP to port 60000 on `127.0.0.1`. Vary the port number and you get additional IPs — `60001`, `60002`, and so on, each consuming one IP from your balance.

bash
9proxy proxy -p 60000


Drops a random available IP from any country onto port 60000. Useful when geography doesn't matter.

bash
9proxy proxy -n -p 60000


Pulls from the Today List — proxies you've used in the last 24 hours. This is the one people underuse. Reusing an active IP doesn't burn a new one from your balance, so if you're reconnecting to a session you already had today, `-n` is the cheaper call.

To see everything currently forwarded, along with ports and status:

bash
9proxy proxy -n -u


### Adding username/password auth

If you'd rather not run an open local port, set basic authentication:

bash
9proxy setting --basic_auth --proxy_password [password] --proxy_username [username]


Substitute your own credentials. After that, clients connect as `username:password@127.0.0.1:port`. On a shared or multi-user machine, this is worth the thirty seconds.

## Confirm the proxy is actually working

Forwarding silently and forwarding successfully look identical from the terminal. Test it:

bash
curl -x socks5://127.0.0.1:60000 https://ipinfo.io/json


If the JSON comes back with a US address, the forward is live and traffic is going out through the residential IP. The same check works inside Python `requests`, a Node scraper, `yt-dlp`, Playwright — anything that accepts a SOCKS5 or HTTP proxy string.

If the response shows your own IP, the forward didn't take: check that `9proxyd` is running, that the port range is configured, and that you're hitting the exact port you forwarded.

## When you'd rather skip the client entirely

On a headless machine, the GB-based model is often the cleaner fit. You generate an endpoint in the dashboard, whitelist the server's IP (or create sub-users with traffic limits), and start sending requests. No daemon, no port forwarding, no package install. Sticky sessions for anything that needs identity continuity, rotating for anything that wants a fresh IP per request.

If that sounds closer to your setup than babysitting a port forward, 👉 [check 9Proxy's GB-based plans and current pricing](https://bit.ly/9-Proxy) before you commit to an IP package you don't need.

## What the plans actually cost

9Proxy announced a pricing update in May 2026 that took effect **June 1, 2026**: IP-based packages and bundles went up, while GB-based packages were left untouched. Third-party reviews published after the change reflect the new numbers, and that's what's below.

> IPs you've already bought never expire, which is why the pre-adjustment rates were worth locking in for anyone maintaining IP inventory. GB-based pricing carries a 180-day validity window, unlimited on Enterprise tiers.

### Residential Proxy by IPs

Unlimited bandwidth per active IP. One-time purchase, no subscription.

| Package | Cost per IP | Total | Buy |
| --- | --- | --- | --- |
| 100 IPs | $0.24 | $24 | [Get the 100 IP package](https://bit.ly/9-Proxy) |
| 500 IPs | $0.144 | $72 | [Get the 500 IP package](https://bit.ly/9-Proxy) |
| 1,000 IPs + 500 bonus | $0.084 | $126 | [Get the 1,000 + 500 IP package](https://bit.ly/9-Proxy) |
| 2,500 IPs | $0.084 | $210 | [Get the 2,500 IP package](https://bit.ly/9-Proxy) |
| 5,000 IPs | $0.072 | $360 | [Get the 5,000 IP package](https://bit.ly/9-Proxy) |
| 15,000 IPs | $0.048 | $720 | [Get the 15,000 IP package](https://bit.ly/9-Proxy) |
| 25,000 IPs | $0.035 | $863 | [Get the 25,000 IP package](https://bit.ly/9-Proxy) |
| 50,000 IPs | $0.029 | $1,438 | [Get the 50,000 IP package](https://bit.ly/9-Proxy) |

### Business IP packages (high volume)

| Package | Cost per IP | Total | Buy |
| --- | --- | --- | --- |
| 100,000 IPs | $0.023 | $2,300 | [Get the 100,000 IP package](https://bit.ly/9-Proxy) |
| 200,000 IPs | $0.021 | $4,140 | [Get the 200,000 IP package](https://bit.ly/9-Proxy) |
| 500,000 IPs | $0.018 | $8,625 | [Get the 500,000 IP package](https://bit.ly/9-Proxy) |

### Residential Proxy by GB

No client required — generate endpoints from the dashboard.

| Package | Cost per GB | Total | Validity | Buy |
| --- | --- | --- | --- | --- |
| 5 GB | $3.00 | $15 | 180 days | [Get the 5 GB package](https://bit.ly/9-Proxy) |
| 50 GB + 5 GB bonus | $2.10 | $105 | 180 days | [Get the 50 + 5 GB package](https://bit.ly/9-Proxy) |
| 100 GB | $1.50 | $150 | 180 days | [Get the 100 GB package](https://bit.ly/9-Proxy) |
| 200 GB | $1.00 | $200 | 180 days | [Get the 200 GB package](https://bit.ly/9-Proxy) |
| 1,000 GB | $0.80 | $800 | 180 days | [Get the 1,000 GB package](https://bit.ly/9-Proxy) |
| 2,000 GB | $0.75 | $1,500 | 180 days | [Get the 2,000 GB package](https://bit.ly/9-Proxy) |

### Enterprise GB packages (no expiry)

| Package | Cost per GB | Total | Validity | Buy |
| --- | --- | --- | --- | --- |
| 3,000 GB | $0.72 | $2,160 | Unlimited | [Get the 3,000 GB Enterprise package](https://bit.ly/9-Proxy) |
| 6,000 GB | $0.70 | $4,200 | Unlimited | [Get the 6,000 GB Enterprise package](https://bit.ly/9-Proxy) |
| 10,000 GB | $0.68 | $6,800 | Unlimited | [Get the 10,000 GB Enterprise package](https://bit.ly/9-Proxy) |

Enterprise also adds team mode — one owner plus up to five members, with shared bandwidth that doesn't expire inside the team, per-member traffic controls and activity logs.

### Bundle packages (IPs + bandwidth)

| Bundle | Contents | Total | Buy |
| --- | --- | --- | --- |
| Starter | 100 IPs + 5 GB | $30 | [Get the Starter bundle](https://bit.ly/9-Proxy) |
| Popular | 1,500 IPs + 50 GB | $180 | [Get the Popular bundle](https://bit.ly/9-Proxy) |
| Pro | 5,000 IPs + 500 GB | $720 | [Get the Pro bundle](https://bit.ly/9-Proxy) |

The Pro bundle is listed at $860 before a 16.28% discount brings it to $720.

## Choosing a plan for a Linux workflow

The per-IP cost falls off a cliff as volume rises, but the entry tiers are where most Linux users actually land, so the decision is usually cheaper than it looks.

If you're running one cron job that pulls a few thousand pages a day through rotating IPs, the GB model at $15 for 5 GB is the low-risk start and needs zero setup on a server. If you're maintaining logged-in sessions — account-based automation, cart flows, anything where the same IP matters across multiple requests — the IP model is the one that works, and 100 IPs at $24 is a fine way to find out whether the pool quality holds up on your targets.

The middle ground is the Starter bundle: 100 IPs plus 5 GB for $30. You get both session persistence and rotation under one balance, which saves you from guessing wrong and buying a second package later.

Two things to weigh before buying a large IP tier:

- **IP lifetime is not yours to control.** A forwarded IP runs for a few hours up to about 24. Auto Refresh replaces dead ones automatically, and Auto Rotation swaps them on a schedule, but if your job needs a specific IP alive for days, no package size fixes that.
- **Only amd64 packages are documented.** ARM machines have no published client build, so plan around the GB model there.

## Trials, promo codes and what to check first

There's no advertised free tier on the public pricing page. 9Proxy has said in forum threads that they offer a limited trial to new users depending on availability, and they've run recurring giveaway campaigns — CD keys for a batch of free IPs, or codes for a gigabyte of traffic, redeemable at Dashboard → Share Code → Use Code. Those are time-boxed and one per account, so a code you find in a thread from six months ago is almost certainly gone.

Rather than hunt for a stale code, 👉 [create a 9Proxy account](https://bit.ly/9-Proxy) and ask support directly whether an IP-based or GB-based trial is available that week. Support runs over email, live chat and Telegram.

One more thing worth doing on your first real job: run the `curl` check against a target site, not just `ipinfo.io`. A residential IP that resolves cleanly can still be blocked by the specific site you care about, and it's better to discover that on a $15 package than a $720 one.

## Short answers to the usual questions

**Is there an official Linux client?** Yes — `.deb` and `.rpm` packages, plus the `9proxy` CLI and a systemd service.

**Can I use 9Proxy on a headless server?** Yes. GB-based proxies need no client at all; IP-based proxies can be driven entirely by CLI commands, which is what they were built for.

**Does it support SOCKS5?** Yes. Forwarded IPs answer on `127.0.0.1:port` and work over SOCKS5 or HTTP — the official test command uses `socks5://`.

**Do unused IPs expire?** No. You're only charged for an IP when it's forwarded to a port.

**Do GB balances expire?** After 180 days on standard packages. Enterprise packages don't expire.

**Is it a subscription?** No. Everything on the pricing page is a one-time balance purchase.
