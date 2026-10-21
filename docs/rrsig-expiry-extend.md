# Live-signing RRSIG rollover settings

## RRSIG expiry extension

PowerDNS Authoritative Server supports the `rrsig-expiry-extend` setting for live DNSSEC signing.

- **Default:** `0` seconds.
- **Range:** `0` to `31536000` seconds (one year).
- **Effect:** adds the configured number of seconds to the normal RRSIG expiration timestamp.

For example, `rrsig-expiry-extend=5184000` adds 60 days to the normal expiration window. This setting affects signature expiration, not DNSSEC key lifetime or key rollover behavior.

## RRSIG publication overlap

The `rrsig-publication-overlap` setting controls how long the previous signing period's RRSIGs remain published after the current period starts.

- **Default:** `604800` seconds (one week).
- **Range:** `0` to `31536000` seconds (one year).
- **Effect:** during the configured interval after rollover, live signing publishes both the current-period RRSIG and the previous-period RRSIG for each eligible RRset and signing key. Set it to `0` to disable publication of previous-period signatures.

For overlaps longer than one week, the generated RRSIG validity windows are extended as needed so previous-period signatures remain valid throughout the publication overlap. `rrsig-expiry-extend` adds its configured extra validity on top of that. A secondary can only benefit from the overlap after it has transferred the newer zone contents; configure a suitable SOA serial/NOTIFY or refresh process as well.

## RRSIG update command

The optional `rrsig-update-command` setting names an executable to run when a new live-signing period first produces a signature for a zone.

- **Default:** empty (disabled).
- **Invocation:** the executable is started asynchronously with the canonical zone name as its sole argument.
- **Frequency:** at most once per zone per signing period in each PowerDNS process, rather than once per RRset or query.
- **Execution:** the value must be the executable path; it is invoked directly, without a shell, and must be executable and accessible to the PowerDNS process. It is not parsed for additional arguments or shell syntax.

For example:

```ini
rrsig-publication-overlap=2419200
rrsig-update-command=/usr/local/sbin/refresh-signed-zone
```

The command should arrange whatever is needed to make secondaries fetch the newly available signatures, such as updating the SOA serial and sending NOTIFY. Its exit status is logged if it fails, but it does not block DNS responses. Since the hook is triggered by live-signing activity, a quiet zone will not trigger it until the first signing request in the new period. The per-zone/per-period guard is in memory and resets when PowerDNS restarts.

The Debian package's `/etc/powerdns/pdns.conf` is generated from the server's registered defaults during package build, so these options are included in the generated sample configuration automatically.
