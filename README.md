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

## Playit code types (important)

Playit uses several different codes and they are **not interchangeable**:

| Code | Shape | What it is | Where it goes |
| --- | --- | --- | --- |
| Docker agent secret | 32 hex | The key that authenticates an agent | Repository secret `PLAYIT_SECRET` |
| Third-party setup code | 32 hex | A short-lived code from the **Third Party App** wizard | The third-party app (e.g. SquidServers), **not** `SECRET_KEY` |
| Claim code | 10 hex | A code from `playit claim generate` | Browser approval at `https://playit.gg/claim/` |
| Tunnel address | `*.joinmc.link` etc. | The public hostname players connect to | Minecraft “Add Server” |

A code from the **Third Party App** wizard is **not** an agent secret, **not**
a tunnel address, and it expires quickly. It has to be pasted into the
third-party app that generated the workflow. It cannot be used as
`SECRET_KEY`; the agent will answer `InvalidAgentKey`.

### Get a Docker agent secret for this repo

1. Open the
   [Docker agent wizard](https://playit.gg/account/setup/wizard/new-account/docker/docker-name)
   while signed in.
2. Copy the **secret key** it shows.
3. Save it as the repository secret:

```bash
gh secret set PLAYIT_SECRET \
  --repo mmunki/minecraft-java-fabric-server-v26.2 \
  --body '<secret key>'
```

The workflow reads **`PLAYIT_SECRET`**. It is never committed.

Create or manage the tunnel at
[playit.gg/account/tunnels](https://playit.gg/account/tunnels). The tunnel must
point to `127.0.0.1:25565` (TCP, Minecraft Java).

### Diagnose a code

Run **Actions → playit-link-test → Run workflow**. It reads
`PLAYIT_SETUP_CODE` first, then falls back to `PLAYIT_SECRET`, and tests:

- `/claim/setup` and `/claim/exchange` (claim-code path)
- the code as an agent `SECRET_KEY` (Docker agent)
- browser-approval polling for a claim code

It writes `inspect.log`, `claim-setup.log`, `claim-exchange.log`, and
`agent.log` to the run artifact and the job summary. A third-party setup code
is expected to fail this test; the summary explains the correct next step.

```bash
gh secret set PLAYIT_SETUP_CODE \
  --repo mmunki/minecraft-java-fabric-server-v26.2 \
  --body '<fresh setup code>'
```

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
.github/workflows/test.yml     Playit code / agent-linking diagnostic
docker-compose.yml              permanent VPS/home server + Playit agent
```

## Stop

Cancel the workflow run, or let `duration_minutes` expire. The workflow removes
both containers at the end.
