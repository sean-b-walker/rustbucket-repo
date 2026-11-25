# Contract: ip-reputation

Capability used by wafalyzer to check the reputation of a public source IP.
Provider: AbuseIPDB (https://www.abuseipdb.com/). No fallback providers.

## Input

- `ip`: one public IPv4 or IPv6 address.

The caller must not send private, loopback, link-local, or reserved
addresses. If one is sent anyway, the capability returns `invalid_ip`.

## Request settings (fixed)

Every call must use these settings. Never rely on a provider's defaults.

- Lookback window: 90 days.
- Report comments: not requested.

An implementation that applies these itself (for example, a script
that hardcodes them) meets this by construction. An implementation
that cannot (for example, a tool that exposes them as parameters
with different defaults) must document in its integration README
how the caller supplies them.

## Output

On success:

| Field           | Meaning                             | AbuseIPDB field        |
| --------------- | ----------------------------------- | ---------------------- |
| `score`         | Abuse confidence, integer 0-100     | `abuseConfidenceScore` |
| `country`       | Two-letter country code             | `countryCode`          |
| `isp`           | Provider name                       | `isp`                  |
| `usage_type`    | Hosting, residential, etc.          | `usageType`            |
| `report_count`  | Reports in the lookback window      | `totalReports`         |
| `last_reported` | Timestamp of latest report, or null | `lastReportedAt`       |
| `whitelisted`   | Provider whitelist flag             | `isWhitelisted`        |

`whitelisted` is informational only. It is not proof the IP is safe.

## Failure

On failure the capability returns `status: "failed"` and a short `reason`:

- `invalid_ip`: address is malformed or not public.
- `rate_limited`: provider quota or rate limit hit.
- `auth_error`: missing or rejected API key.
- `unavailable`: timeout or provider error.

The caller reports these as **Not checked** with the reason. It must not
retry with another source or invent a score.
