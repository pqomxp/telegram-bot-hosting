# Telegram Bot Hosting: Free 24/7 or Your Own Server? Polling vs Webhooks, What 128 MB Really Runs, and How to Pick (Plans Compared)

Free Telegram bot hosting will keep a small bot answering around the clock, and for plenty of bots that settles the question. The complication is that every free tier defines “24/7” a little differently, and one of the popular free options cannot accept a webhook at all. A server you administer yourself becomes the better buy once you need root, your own libraries, more than one bot on a box, or a database and scheduled jobs you control.

Below is the order the decision usually follows: what free tiers actually include, when polling is enough and when a webhook is unavoidable, which libraries fit in 128 MB, and what a self-managed server needs before Telegram will deliver a single update to it. Evoxt comes in at the point where a managed container stops being enough.

> **Quick rule:** one small bot on long polling can live on a free container indefinitely. If you want root, your own Python or Node stack, several bots, or a webhook on an address you control, skip the container tier and go straight to a server. 👉 [See Evoxt's current server plans and specs](https://bit.ly/EvoXt)

## What “free 24/7” actually means on the popular free tiers

Three different things get called 24/7 in this market: the container never sleeps, the bot process never sleeps, and the public web address never sleeps. Free plans usually deliver the first two and quietly leave out the third.

| Host | Free tier | Webhook on the free tier | The part that bites |
| --- | --- | --- | --- |
| FPS.ms | 128 MB RAM, 25% of one core, 250 MB disk | Yes, through the platform’s own HTTPS URL | Manual renewal once a day; no database and no backup slot |
| FadeHost | 256 MB, 0.25 vCPU, 3 GB disk | No. The free web address sleeps when idle | Polling runs nonstop; one free bot per verified account |
| ENDHOST | Not published | No. Long polling only | Ad-supported panel, no RAM/CPU/disk figures disclosed |

**FPS.ms** is the most generous on paper and the most annoying in practice. The container genuinely never sleeps and restarts on crash, so a polling bot stays up. The catch is the daily Renew button: skip it and the bot goes offline until you press it again. Renew within 28 days and all files come back; after that the server is deleted. You can buy your way out of the clicking for €0.59/month, and the first paid tier is €0.99/month for 256 MB with one database and one backup slot. Storage is small throughout: 250 MB free, 500 MB at €0.99, 1 GB at €1.29, 2.5 GB at €2.29, 5 GB at €3.49 for 2 GB of RAM.

**FadeHost** inverts the trade. There is no renewal timer and no sleep timer on the bot process, so a polling bot runs continuously on the free 256 MB tier. What sleeps is the free `https://name.fadehost.app` address, which wakes on the next request. That single detail decides your mode: webhooks need an address that answers at any moment, so webhook bots start at the Starter tier, $2/month or $20/year for 1 GB and 1 vCPU. Paid plans are priced per bot: Standard at $3/month (or $30/year) for two bots, Pro at $6/month (or $60/year) for four. Extra memory is $1/month per GB, up to three.

**ENDHOST** runs free Telegram and Discord bots on an ad-supported panel with no card required, supports Node.js, Python, Ruby, Go, PHP and Bun, and encrypts stored tokens so the panel never sends a saved token back to the browser. It also only does long polling. If your bot needs a webhook, this one is out before price enters the conversation.

None of the three promises unlimited resources at zero cost. They are built for command handlers, reminders, small group utilities and testing, which is exactly what most people reading about Telegram bot hosting are actually running.

## Polling or webhook? Decide by whether you already own an address

Use long polling unless you already serve something else on a web address, or you specifically want Telegram pushing updates to you.

Polling means the bot opens an outgoing connection to Telegram and asks for new messages. No domain, no certificate, no open port, no inbound firewall rule. On FadeHost’s template the only variable you set is `TELEGRAM_BOT_TOKEN`; everything else has a default. On FPS.ms, polling is the zero-setup option and the platform’s Public URL exists for people who want webhooks instead.

Webhooks reverse the direction: Telegram delivers each update to your bot’s HTTPS address. The requirements are not negotiable. Telegram only delivers to ports **443, 80, 88 and 8443**, over HTTPS, with a certificate that validates. FadeHost’s template also expects `WEBHOOK_SECRET`, and the bot rejects any update that arrives without it. A free address that sleeps when idle is the wrong receiving end for a webhook, which is why FadeHost pushes webhook users to the $2/month tier.

You will find guides that call webhooks the production choice and polling a development shortcut. That advice assumes a VPS with a domain, Nginx and a certificate already in place. On a managed platform where the free address is shared and idle-sleeping, polling is the correct default and there is nothing amateur about it. Neither mode is universally better; the address you control is what tips the decision.

## Which library actually fits in 128 MB

Memory, not CPU, is what kills small bots. The useful data point: FPS.ms runs its free 128 MB container on Python 3.13.16 with python-telegram-bot 22.8, logging `getMe` as HTTP 200 and handling `/start` and a `/remind` command without trouble. Telegraf and grammY on Node.js fit in the same envelope.

The library version matters more than the language. FPS.ms states plainly that **aiogram 3.29 and newer needs more memory than the free plan has** and has to run on a paid tier. So a 128 MB free container is fine for a small python-telegram-bot or Telegraf bot, and not fine for a modern aiogram project. Budget the 256 MB tier (€0.99/month at FPS.ms, free at FadeHost) as the realistic floor for aiogram, and move to 512 MB or more once you add a Mini App, heavy dependencies, or anything holding state in memory.

FadeHost runs eight runtimes from the repository it detects: Node.js, Python, Bun, Deno, Go, Java, PHP and Ruby, with grammY, Telegraf, python-telegram-bot and aiogram supported as-is. Its free 256 MB tier is the more comfortable starting point for Python work, provided you stay on polling.

## When a self-managed server is the right call

A server earns its price when you want things a container panel does not expose: root, a specific OS image, system packages, several bot processes, your own database, or cron jobs beyond the platform’s limit.

The reference self-hosted setup for a Telegram bot is modest and well documented. Ubuntu 22.04 with **1 GB of RAM or more**, Python 3.10+, a domain pointing at the box, Nginx as a reverse proxy, certbot for the certificate, and a systemd unit with `Restart=on-failure` and `RestartSec=10` so a crash does not end the bot. Webhook mode then needs the domain registered with Telegram and reachable over valid HTTPS. A 1 GB instance will run several lightweight bots at once; the ceiling is usually per-process memory rather than CPU.

This is the route Evoxt sells: self-managed cloud servers where you get root, choose the OS, install your own runtime and own the TLS and restart configuration. That is more work than pasting a token into a panel, and it is the only way to run a stack the panel does not offer. 👉 [Look at Evoxt's plan list before you pick a size](https://bit.ly/EvoXt)

If you want a rough map of the three routes:

| Route | What you get | Who it suits |
| --- | --- | --- |
| Free container (FPS.ms, FadeHost, ENDHOST) | 128–256 MB, platform-managed OS, token in a web panel | One small bot, testing, low-traffic command bots |
| Telegram-focused PHP host (Botlify) | PHP 8.2/8.3 CLI and FPM, per-bot sandbox, 15 MB storage by default, automatic HTTPS webhook, one-minute cron | PHP bots that want webhooks without server administration |
| Self-managed server (Evoxt) | Root, your OS and runtime, your Nginx, TLS and systemd | Several bots, custom dependencies, databases, scheduled jobs |

Botlify is worth knowing about if your bot is PHP: it configures the HTTPS webhook for you, ships a webhook simulator and a `/mcp/v1` endpoint for AI-assisted editing, and keeps each bot in its own sandbox with SQLite in WAL mode. The 15 MB default storage quota is the constraint to check before you move a media-heavy bot there.

## Launch checklist before you point a domain at it

1. **Token handling.** Get it from BotFather, store it as an environment variable, never in the repository. Both FPS.ms and FadeHost mask saved values in the panel; keep it that way in your own code too.
2. **Webhook registration.** Confirm the URL Telegram has on file matches the one your server actually serves, and that the certificate chain validates without a browser warning.
3. **Dependencies pinned.** A missing package in `requirements.txt` or `package.json` is the most common reason a bot works locally and dies on deploy.
4. **One process per token.** Two instances polling the same token will fight, and Telegram answers with `409 Conflict`. This bites people who leave a local copy running while testing the server.
5. **Restart policy.** On a VPS, `Restart=on-failure` with a 10-second delay covers crashes. On a managed platform, autostart is already handled; check the container is set to start on boot.
6. **Access control.** For a personal bot, restrict handlers to a list of allowed user IDs rather than trusting that nobody finds it.

## FAQ

**Can a free plan really run a bot 24/7?**
Yes, with a definition. FPS.ms keeps the container awake but needs a daily renewal click. FadeHost keeps the polling process awake with no renewal, but the free web address sleeps when idle. ENDHOST keeps bots running on an ad-supported panel and supports long polling only.

**Is 128 MB enough for a Telegram bot?**
For a small python-telegram-bot or Telegraf bot, yes. FPS.ms runs python-telegram-bot 22.8 on its 128 MB free container. For aiogram 3.29 or newer, no, and the platform says so directly.

**Do I need a domain to use webhooks?**
You need an HTTPS address with a valid certificate on one of Telegram’s four accepted ports. On a managed platform the platform’s subdomain covers that, as long as it stays awake. On your own server you need the domain plus Nginx and certbot.

**How many bots fit on one server?**
One process per bot token, and a 1 GB instance handles several lightweight bots. Watch memory per process rather than total CPU.

**What causes a 409 Conflict?**
Two running instances polling with the same token. Stop the duplicate, usually a leftover local process, and it clears.

## How to choose

- One bot, long polling, low traffic: a free container is enough. Pick FPS.ms if you do not mind the daily renewal, FadeHost if you would rather not think about it.
- Webhook required, no server administration wanted: a paid tier with an always-on address, $2/month or $20/year at FadeHost, or a Telegram-focused PHP host if your bot is PHP.
- Root, custom dependencies, several bots, a database, or your own scheduled jobs: a self-managed server, where you control the OS, the runtime and the TLS setup.

Before you order anything self-managed, confirm on the provider’s own page the RAM, CPU, disk and transfer of the tier you are looking at, the datacenter location relative to your users, the billing cycle and what a renewal costs, and whether snapshots or backups are included or extra. Those four details decide whether a cheap plan is actually cheap for a bot that has to stay up for a year. 👉 [Compare Evoxt's current plans and pricing](https://bit.ly/EvoXt)
