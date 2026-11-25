# AbuseIPDB MCP integration

Provides the `ip-reputation` capability (see `contracts/ip-reputation.md`)
using AbuseIPDB's MCP server, `mcp-server-abuseipdb`. Everything installs
inside this folder. Nothing is installed globally.

## Requirements

- Python 3.11 or newer
- [uv](https://docs.astral.sh/uv/)
- An AbuseIPDB API key (free account, from https://www.abuseipdb.com/account/api)

## Setup

From the repo root:

    cd integrations/abuseipdb-mcp
    export UV_CACHE_DIR="$PWD/.uv-cache"
    uv venv
    uv pip install -r requirements.txt

`requirements.txt` pins the exact versions that were tested. To upgrade,
install the new version, run `uv pip freeze > requirements.txt`, and
re-test before committing.

`.venv/` and `.uv-cache/` are gitignored. Delete this folder's
`.venv/` and `.uv-cache/` to remove the install completely.

## API key

The server reads `ABUSEIPDB_API_KEY` from its environment. Never put
the key in `opencode.json`, in this README, or anywhere in the repo.

Set it for the current terminal session without echoing it or saving
it to shell history:

    read -rs ABUSEIPDB_API_KEY && export ABUSEIPDB_API_KEY

Then start your harness **from that same terminal**.

Gotcha: a process only receives the environment it was started with.
If the harness was already running when you set the key, it will not
see it. Quit it fully and relaunch. The symptom is "MCP server failed:
ABUSEIPDB_API_KEY is not set."

## Wiring (OpenCode)

`opencode.json` in the repo root starts the server from this folder's
virtual environment:

    "command": ["integrations/abuseipdb-mcp/.venv/bin/mcp-server-abuseipdb"]

The path is relative, so start OpenCode from the repo root.

## Required parameters

The server's defaults do not match the contract. Every call must pass:

| Parameter         | Value   | Server default | Why                          |
| ----------------- | ------- | -------------- | ---------------------------- |
| `max_age_in_days` | `90`    | `30`           | Contract: 90-day window      |
| `verbose`         | `false` | `true`         | Contract: no report comments |

If a result contains report details or a 30-day window, the caller
did not pass these.

## Fields

The server returns more than the contract needs. The agent uses only:

| Contract field  | Server field           |
| --------------- | ---------------------- |
| `score`         | `abuseConfidenceScore` |
| `country`       | country code           |
| `isp`           | `isp`                  |
| `usage_type`    | usage type             |
| `report_count`  | total reports          |
| `last_reported` | last reported time     |

The server also returns `riskTier` and a plain-English `message`.
These are the server's own interpretation. The agent ignores them
and reports AbuseIPDB's score and facts only.

## Errors

| Contract reason | What it looks like here                         |
| --------------- | ----------------------------------------------- |
| `auth_error`    | Server exits: "ABUSEIPDB_API_KEY is not set"    |
| `rate_limited`  | Structured `RATE_LIMITED` error from the server |
| `invalid_ip`    | Server rejects private/reserved or malformed IP |
| `unavailable`   | Timeout or provider error                       |
