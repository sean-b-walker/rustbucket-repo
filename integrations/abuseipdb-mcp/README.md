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

## Wiring (OpenCode)

`opencode.json` in the repo root starts the server through mise, so the
server inherits the key from `.env` no matter how OpenCode itself was
launched:

    "command": ["mise", "exec", "--", "integrations/abuseipdb-mcp/.venv/bin/mcp-server-abuseipdb"]

The path is relative, so start OpenCode from the repo root. `codemode`
is off, which exposes the tool as `abuseipdb_check_ip` instead of
nesting it behind `execute`.

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

## API key

The server reads `ABUSEIPDB_API_KEY` from its environment. The key lives
in a gitignored `.env` file at the repo root, loaded by mise (see
`mise.toml`). Never put the key in `opencode.json`, this README, or any
tracked file.

First-time setup, from the repo root:

    read -rs KEY_IN && printf 'ABUSEIPDB_API_KEY=%s\n' "$KEY_IN" > .env && unset KEY_IN
    chmod 600 .env
    mise trust

`.env.example` shows the variable name. mise refuses to load `.env`
until you run `mise trust`, and again after any edit to the file
(including rotating the key).

Check it loaded, without printing the key:

    [ -n "$ABUSEIPDB_API_KEY" ] && echo "set" || echo "NOT set"

Run this inside the repo. Outside it, the variable is correctly unset.

Because the command runs through `mise exec`, the key is read from `.env`
each time the server starts. After rotating it, reconnect the server
(`/mcps`, or `/reload`) rather than relaunching OpenCode. The symptom of
a missing key is "MCP server failed: ABUSEIPDB_API_KEY is not set."

## Rebuilding the environment

The `.venv/` points at the Python that created it. If that Python is
later removed or upgraded, the server may fail to start. Fix:

    cd integrations/abuseipdb-mcp
    rm -rf .venv
    export UV_CACHE_DIR="$PWD/.uv-cache"
    uv venv
    uv pip install -r requirements.txt

`requirements.txt` pins the versions, so the rebuild is identical.
