# Minecraft Java Fabric 26.2 + Playit.gg

A **Minecraft Java Edition 26.2** server running **Fabric**, exposed through a
**Playit.gg** tunnel. Random seed on every start. Server-side only — vanilla
clients can join.

## Start a temporary public server

1. Go to **Actions → minecraft-server → Run workflow**.
2. Choose:
   - `duration_minutes`: how long to keep it online (default 180, max 330).
   - `online_mode`: `false` lets anyone join with any username; `true` requires
     a Microsoft/Minecraft account.
   - `max_players`.
3. Run it. The **Join address** is printed in the `Start the Playit.gg tunnel`
   step, the job summary, and the `playit.log` artifact.
4. If the address is not in the log, open
   [playit.gg/account/tunnels](https://playit.gg/account/tunnels).

The address looks like `something.joinmc.link` (or another Playit domain).
Use **Minecraft Java Edition 26.2**.

## Important limitations

- **GitHub Actions is not a 24/7 host.** The server lives only while the job
  runs; the job is capped at **350 minutes** and the default run is 180.
- The world is **ephemeral**. Each run gets a **new random seed** and a fresh
  world. Do not use this for a world you want to keep.
- `online_mode=false` means **anyone can join with any username**, including
  impersonating someone. Use `online_mode=true` for a real account-only server.
- A public offline server can be griefed. This is a test server, not a home for
  valuable builds.
- GitHub-hosted runners have no uptime guarantee. If the runner is interrupted,
  the server goes down.

## Permanent server (VPS / home machine)

Use the included `docker-compose.yml`:

```bash
export PLAYIT_SECRET='your-playit-secret'
export SEED=''            # empty = random
export ONLINE_MODE='true' # or false for anyone
docker compose up -d
docker compose logs -f minecraft
```

The Playit agent uses **host networking** and tunnels TCP `25565` to the
Minecraft container. The assigned address is shown in:

```bash
docker compose logs playit
```

## Playit secret

The workflow reads the repository secret **`PLAYIT_SECRET`**. It is never
committed. For a local server, export it as an environment variable.

Create or manage the tunnel at
[playit.gg/account/tunnels](https://playit.gg/account/tunnels). The tunnel must
point to `127.0.0.1:25565` (TCP, Minecraft Java).

> The Playit **Fabric mod** only supports Minecraft up to 1.21.1, so this uses
> the Playit **agent** instead. The agent works with 26.2.

## Version details

- Minecraft Java: **26.2**
- Server: **Fabric** (loader resolved automatically by the `itzg` image)
- Java: **25** (`itzg/minecraft-server:java25`)
- Seed: random, printed in the workflow log and summary
- Port: `25565/TCP`

## Files

```
.github/workflows/server.yml   temporary GitHub Actions server
docker-compose.yml              permanent VPS/home server + Playit agent
```

## Stop

Cancel the workflow run, or let `duration_minutes` expire. The workflow removes
both containers at the end.
