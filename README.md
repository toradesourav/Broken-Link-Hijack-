
# Broken Link Hijack Finder

Scrapes a page for external links, scripts, and other resource
references, then checks each one for signs it points to an
unclaimed/expired resource that you could potentially register and
serve content from — a dangling GitHub Pages repo, an unclaimed S3
bucket, an expired domain still linked in old HTML/JS, etc.

This is a related but distinct technique from subdomain takeover: it
checks *any* external resource reference on the page (not just the
target's own subdomains), which is why it catches things like an old
blog post linking to a domain the target company let lapse.

## ⚠️ Authorized use only, and verify before acting
This tool only reports **candidates**. A `[!!! HIJACKABLE]` or
`[EXPIRED DOMAIN]` finding means "worth investigating manually" —
not "confirmed exploitable." Actually registering a domain or
claiming a cloud resource to demonstrate impact should only be done
within an authorized engagement's rules of engagement, and you are
responsible for any cost/liability of registering real infrastructure.

## Requirements
```
pip install requests beautifulsoup4
```

## Usage
```bash
# Check every link/script/img/iframe reference on the page
python broken_link_hijack.py --url https://target.com

# Only check links pointing to other domains (skip same-site links)
python broken_link_hijack.py --url https://target.com --external-only
```

## Verdicts
| Verdict | Meaning |
|---|---|
| `DANGLING` | Response body matches a known "unclaimed resource" fingerprint for that hosting service (GitHub Pages, S3, Heroku, Azure, etc.) — high confidence |
| `NXDOMAIN` | The hostname doesn't resolve at all — the domain may have expired and become re-registerable — high confidence |
| `404` | The host resolves and responds, just with a 404 — lower confidence, many legitimate sites 404 old pages without being hijackable |
| `UNREACHABLE` | Request failed (timeout, connection refused, etc.) — inconclusive |

## Notes
- The fingerprint list covers common services but isn't exhaustive —
  if you find a dangling-resource message from a service not listed,
  that's still worth investigating manually.
- DNS resolution is checked with the system resolver; results can be
  affected by local DNS caching.

## Status
Part of a personal 100-tool security scripting project. Verified
end-to-end against a local mock page linking to a healthy site, a
plain-404 site, and a non-resolving domain — all three correctly
categorized (found and fixed a port-parsing bug during testing where
`urlparse().netloc` includes the port and broke the DNS check). The
service-specific fingerprint matching uses simple string containment
and couldn't be exercised against real GitHub Pages/S3 infrastructure
in this sandboxed test, but the logic is straightforward.

## License
MIT
