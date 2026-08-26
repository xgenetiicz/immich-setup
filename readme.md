# Immich Setup — rasp5-16gbram

This is the compose file behind my self-hosted Immich instance, running on a Raspberry Pi 5 (16GB) through Portainer. It's been running for a while now, so this is basically the "why did I set it up like this" doc for future me - or whoever else ends up looking at this repo.

## The stack

- **gatewaycaddy** — Caddy reverse proxy sitting in front of everything. Needs `flush_interval -1` in the Caddyfile, otherwise uploads and thumbnails get buffered and feel laggy.
- **immich-server** - the actual app.
- **immich-machine-learning** - handles face detection, smart search, etc. Running CPU-only right now, no hardware acceleration configured (that's commented out in the compose file for later).
- **database** - Postgres, but the Immich-maintained image with vectorchord and pgvector already baked in, since that's what the ML search features need.
- **tunnel** - Cloudflare Tunnel, used only for public access.

Tailscale doesn't show up in this compose file at all - it runs on the host. It's there so my phone can bypass subnet-level restrictions when I'm out and about, and still upload photos at the same speed as if it were sitting on the local network. And this also provides security also through a encrypted VNet.

Cloudflare Tunnel solves a different problem: it holds the actual subdomain and lets me share albums or the instance with other people without pulling them into my tailnet. Nobody I share with needs to install Tailscale or get added to my network - they just get a link so they can either read or have read/write/delete or what ever the access is given on the specific album that is shared.

These two are kept deliberately separate: Cloudflare terminates TLS at their edge, Tailscale stays end-to-end encrypted on the tailnet. One is for me on the move, one is for everyone else.

## Why manual deploys

Nothing here auto-updates. Everything pins to `${IMMICH_VERSION:-release}` and gets redeployed by hand through Portainer's Git integration whenever I actually want a new version - never automatically. Database migrations running unattended on a box that holds my actual photos isn't a risk worth taking for the convenience of auto-updates. I would like to update after stable release - and also after some weeks when the bugs have been attended to hehe.

## Environment variables (.env)

```
IMMICH_VERSION=
UPLOAD_LOCATION=
STORAGE2_LOCATION=
DB_USERNAME=
DB_PASSWORD=
DB_DATABASE_NAME=
DB_DATA_LOCATION=
CLOUDFLARE_TUNNEL_TOKEN=
```

## Suggestions
- Strongly suggest to add access policies and OTP authentication on the URL. You don't want bots attacking your site. This can be configured through Zero at Policies and include your email for this.

- Also add an open source such as Autentik for OAuth - so you can remove raw username and password login and add a multifactor layer instead!

### Thank you for taking your time to read! I hope you enjoyed it -  if you have something i could do better or fix anything, please let me know. :D