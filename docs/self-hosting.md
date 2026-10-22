# Self-hosting the open-source Kortix starter

Self-hosting Kortix means the repo, the connectors, the models and the memory sit on your own infrastructure. Run it on Kortix Cloud, in your VPC, or on your own on-prem network. Self-host is free.

## What you own

The project repo is the company. Your agents, their skills, your company memory and every connector live in one git repo you own. Agents and skills are markdown files, and `kortix.yaml` declares the machine image, the connectors and the triggers, and the secret names and grants. Secret values never appear in the repo. Because it is a repo, you can grep the whole project, diff any change, and roll any part of it back.

Session work reaches main through a change request, and merge is default-deny for agents, so the repo improves one reviewed change at a time.

## The steps

The whole instance runs as one Docker Compose stack: the frontend, the API, the LLM gateway, and the Supabase distribution. There is no Redis and no separate worker.

On a bare Linux box, one command installs Docker, installs the Kortix CLI and starts the stack. On another operating system, install the CLI and take the manual path:

```bash
# 1 · Install the CLI
curl -fsSL https://kortix.com/install | bash

# 2 · Point an A/AAAA record for your domain and for api.<domain> at the box,
#     open ports 80 and 443, then initialize
kortix self-host init --domain kortix.example.com

# 3 · Start the stack
kortix self-host start
```

To try it with no domain, use a Cloudflare tunnel and expect the URL to change on every restart:

```bash
kortix self-host init --tunnel cloudflare
kortix self-host start
```

After the stack starts, set your sandbox provider key:

```bash
kortix self-host configure
```

Then sign up in the dashboard, and connect your own model key in the model picker. GitHub connects in the dashboard under Settings, then Git.

## Trade-offs to know

- Agent sessions run on a separate sandbox provider rather than on this stack. Platinum is the default, and Daytona remains available as a legacy fallback. The control plane you host stays small, and the instance has to be reachable so a sandbox can call back.
- `self-host start` pulls its images from Docker Hub, so plan for that when you size the network.
- Each API container has a 640 MiB memory limit by default. Keep the default on an 8 GiB host, and raise it to 1 GiB on a 16 GiB host if API traffic hits the limit.
- A self-hosted instance uses your own model key by default. The first run asks six things and no model key.
- Every instance updates itself automatically, or you can pin an exact version and turn the updater off.

## Backups

There is no separate backup system. Each instance stores its data as two directories under `~/.config/kortix/self-host/<instance>/`: `volumes/db/data` for the Postgres database and `volumes/storage` for file storage. The instance `.env` file holds every secret and signing key it uses. Back up all three before a destructive command.

## Read on

The [self-hosting](https://openwebuialternative.com/self-hosting) page covers what you own in the same shape. When the stack is live, the work still runs the loop from [getting started](getting-started.md): a session on its own cloud computer, and a change request you review.

Self-hosted or in the cloud, the code is yours. [Try Kortix](https://kortix.com) in the cloud first if you want to see the loop before you run it yourself.
