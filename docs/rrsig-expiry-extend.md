# RRSIG expiry extension

PowerDNS Authoritative Server supports the `rrsig-expiry-extend` setting for live DNSSEC signing.

- **Default:** `0` seconds (existing behavior).
- **Range:** `0` to `31536000` seconds (one year).
- **Effect:** adds the configured number of seconds to the normal RRSIG expiration timestamp.

For example, `rrsig-expiry-extend=5184000` adds 60 days to the normal expiration window. This setting affects signature expiration, not DNSSEC key lifetime or key rollover behavior.

The Debian package's `/etc/powerdns/pdns.conf` is generated from the server's registered defaults during package build, so the option is included in the generated sample configuration automatically. The packaging rule adds a short explanatory comment beside it.
