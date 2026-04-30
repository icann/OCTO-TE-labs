---
layout: page
title: "Zone Delivery"
parent: "DNSSEC"
nav_order: 9
---

# Zone Delivery

Zone generation and zone signing are very critical operations. Any error can 
have huge consequences. Therefore do most operators of critical zones use a 
technic called zone pipeline. It is simply a sequences of steps, where later 
steps check if earlier steps succeeded and did the right thing.

1. Check zone file for completeness
1. Validate all signatures
1. Check NSEC or NSEC3 chains

Currently the primary delivers the zone directly to the secondaries. We will
use NSD to implement an additional checking step.

In our lab we will only use a very simple mock-up script for checking a zone.

# Configuration of hidden primary

On our soa machine we change the bind configuration to use a non-standard port 
and only on the localhost interface.

We start with changes to  `/etc/bind/named.conf.options`
```
    listen-on port 5353 { localhost; };
    listen-on-v6 port 5353 { localhost; };
```
And in file `/etc/bind/named.conf.local`
```
    allow-transfer { ::1; };
    also-notify { ::1; };
```

# Install NSD checking server

Please install NSD and some tools that we need for this lab on the machine by running
```
sudo apt install -y nsd validns ldnsutils
```

Now we need to configure NSD:
```
include: "/etc/nsd/nsd.conf.d/*.conf"

server:
    zonesdir: "/var/lib/nsd"
    hide-version: no
    hide-identity: no
    nsid: "ascii_grp%GRP% zone validator"
    verbosity: 2

verify:
    enable: yes
    verify-zones: yes
    verifier-timeout: 10  
  
pattern:
    name: "fromprimary"
    allow-notify: ::1 NOKEY
    request-xfr: AXFR ::1@5353 NOKEY
    verify-zone: yes
    verifier: /var/lib/nsd/test.sh
    notify: 100.100.%GRP%.130 NOKEY
    notify: 100.100.%GRP%.131 NOKEY
    notify: %IPv6pfx%:%GRP%:128::130 NOKEY 
    notify: %IPv6pfx%:%GRP%:128::131 NOKEY
    provide-xfr: 100.100.%GRP%.130 NOKEY
    provide-xfr: 100.100.%GRP%.131 NOKEY
    provide-xfr: %IPv6pfx%:%GRP%:128::130 NOKEY 
    provide-xfr: %IPv6pfx%:%GRP%:128::131 NOKEY

zone:
    name: "grp%GRP%.%DOMAIN%."
    zonefile: "db.grp%GRP%.secondary"
    include-pattern: "fromprimary"
```
Now we need to make a small zone checking script. Please edit the file `/var/lib/nsd/test.sh` with
```
sudo nano /var/lib/nsd/test.sh
```
to the following content
```
#!/usr/bin/env bash
set -euo pipefail

PATH=/usr/sbin:/usr/bin:/sbin:/bin

log() {
  logger -t "$LOG_TAG" -- "$*"
  echo "$LOG_TAG: $*" >&2
}

fail() {
  log "FAIL zone=${ZONE} reason=$*"
  exit 1
}

pass() {
  log "PASS zone=${ZONE}"
  exit 0
}

require_cmd() {
  command -v "$1" >/dev/null 2>&1 || fail "missing command: $1"
}

ZONE="${VERIFY_ZONE:-}"
ZONE_ON_STDIN="${VERIFY_ZONE_ON_STDIN:-no}"

WORKDIR="/tmp"
LOG_TAG="VERIFIER"

TMP_ZONE="$(mktemp "${WORKDIR%/}/nsd-verify.${ZONE}.XXXXXX.zone")"
trap 'rm -f "$TMP_ZONE"' EXIT

log "ZONE           $ZONE"
log "ZONE_ON_STDIN  $ZONE_ON_STDIN"
log "TMP_ZONE       $TMP_ZONE"

[ -n "$ZONE" ] || fail "VERIFY_ZONE not set"

require_cmd nsd-checkzone
require_cmd validns
require_cmd ldns-verify-zone

if [ "$ZONE_ON_STDIN" = "yes" ]; then
    cat > "$TMP_ZONE"
else
    fail "VERIFY_ZONE_ON_STDIN=no; configure verifier-feed-zone: yes or adapt script to fetch zone by query"
fi

[ -s "$TMP_ZONE" ] || fail "empty zone data received"

nsd-checkzone "$ZONE" "$TMP_ZONE" >/dev/null || fail "nsd-checkzone failed"

validns -q -p dnskey -p ksk-exists "$TMP_ZONE" || fail "validns failed"

ldns-verify-zone "$TMP_ZONE" >/dev/null || fail "ldns-verify-zone failed"

pass
```
And make it executable with 
```
sudo chmod +x /var/lib/nsd/test.sh
sudo chown nsd:nsd /var/lib/nsd/test.sh
```
 
Ready to restart

1. Open another shell window on the soa machine
1. Run `sudo journalctl -x -u nsd -f`
1. Back to the first shell window
1. Restart Bind `sudo systemctl restart named`
1. Restart NSD `sudo systemctl restart nsd`
1. Edit the zone file `/var/lib/bind/zones/db.grp%GRP%`, increase the serial number
1. Run `rndc reload`
1. Back to the new shell window and see if you can identify the validation in the logs

Lets check what contents our servers carry.

1. `dig @localhost -p 5353    grp%GRP%.%DOMAIN% SOA +nsid`
1. `dig @localhost -p 53      grp%GRP%.%DOMAIN% SOA +nsid`
1. `dig @100.100.%GRP%.130        grp%GRP%.%DOMAIN% SOA +nsid`
1. `dig @%IPv6pfx%:%GRP%:128::131 grp%GRP%.%DOMAIN% SOA +nsid`

Did all servers show the correct serial number?
Did you get different id strings for all servers?

Currently our script fails all zones. Let's try if we approve all zones.

1. Edit `/var/lib/nsd/test.sh` again and change `exit 5` to `exit 0`.
1. Edit the zone file `/var/lib/bind/zones/db.grp%GRP%`, increase the serial number
1. Run `rndc reload`
1. Back to the new shell window and see if you can identify the validation in the logs

What's different?

Run the same dig commands again:

1. `dig @localhost -p 5353    grp%GRP%.%DOMAIN% SOA +nsid`
1. `dig @localhost -p 53      grp%GRP%.%DOMAIN% SOA +nsid`
1. `dig @100.100.%GRP%.130        grp%GRP%.%DOMAIN% SOA +nsid`
1. `dig @%IPv6pfx%:%GRP%:128::131 grp%GRP%.%DOMAIN% SOA +nsid`

Did all servers show the correct serial number?
Did you get different id strings for all servers?
