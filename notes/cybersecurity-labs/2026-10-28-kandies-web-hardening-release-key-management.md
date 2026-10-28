# Kandies Web Hardening and Release Signing: October 28, 2026

**Time zone:** America/New_York (EDT)
**Course context:** CYB/130T, applied practice on my own system
**System:** Kandies (my own app) on Netlify; Windows PC; iPhone with iSH (Alpine Linux); GitHub Actions

---

## Session Time

| Block | Start | End | Active |
|---|---|---|---|
| 1 | 12:38 | 13:25 | 47 m |
| Break (lunch) | 13:25 | 13:58 | |
| 2 | 13:58 | 14:43 | 45 m |
| 3 | 16:46 | 17:46 | 29 m |
| Interruption (delivery) | 17:13 | 17:44 | |

**Active:** 2 h 01 m. **Elapsed:** 5 h 08 m.

## Goal

1. Audit the live site's HTTP security headers and close any gaps.
2. Create, protect and use the Android app-signing (upload) key to produce a verified signed release.

## Setup

- iPhone: iSH terminal (Alpine Linux) with curl, git, openssh-client
- Windows PC: PowerShell, Chrome/Firefox, Git, GitHub CLI, Java 21 keytool
- Netlify hosting (headers set with a `_headers` file)
- GitHub Actions with encrypted secrets
- USB drive for offline key backup

## Timeline

| Time | Event |
|---|---|
| 12:47 | Header audit: only HSTS present; 4 common headers missing |
| 12:55 | Framing test confirmed the site could be embedded by another page |
| 13:01 | Test file moved out of the published folder |
| 13:03 | Security headers file created |
| 13:06 | Committed and pushed |
| 13:14 | Framing test now blocked by the browser |
| 13:23 | Header audit confirms all 4 headers live |
| 14:05 | Java/keytool installed |
| 14:09 | Upload key generated outside the repo |
| 14:26 | Offline USB backup created and verified |
| 14:33 | Key and password stored as encrypted GitHub secrets |
| 16:50 | Manual-only release workflow committed and started |
| 17:09 | Signed build succeeded |
| 17:13 | Bundle signing fingerprint matched the upload key |
| 17:44 | Signed bundle downloaded outside the repo |

## Problem Log

| # | Problem | Found | Fixed | Time to fix | Cause |
|---|---|---|---|---|---|
| 1 | Site could be framed by other pages (clickjacking risk) | 12:55 | 13:14 | 19 m | No X-Frame-Options header |
| 2 | Missing nosniff, referrer and permissions headers | 12:47 | 13:23 | 36 m | Netlify defaults only include HSTS |
| 3 | Test file saved inside the published www folder | 12:57 | 13:01 | 4 m | Save As dialog reused the last folder |
| 4 | iSH command run in Windows PowerShell | 12:59 | 13:01 | 2 m | Wrong device/terminal |
| 5 | PowerShell waiting at `>>` | 13:03 | 13:05 | 2 m | Missing closing quote |
| 6 | Header check printed the full page | 13:22 | 13:23 | 1 m | `-I` flag dropped |
| 7 | keytool not recognized | 13:58 | 14:05 | 7 m | Java not installed |
| 8 | USB drive not found | 14:13 | 14:24 | 11 m | Drive not yet plugged in |
| 9 | Secret upload returned HTTP 404 | 14:28 | 14:33 | 5 m | Stray backslash in repo name |
| 10 | Deprecation warnings in build output | 17:09 | Open | — | Actions v4 use Node 20; runner image moving to Ubuntu 26. Not errors |

## Security Findings / Hardening Done

### Finding 1: Clickjacking (framing) allowed
- **Found:** a local test page embedded the live sign-in screen (evidence 02, 03).
- **Fixed:** `X-Frame-Options: DENY` via Netlify `_headers`.
- **Verified:** the same test page now shows the browser's "will not allow … embedded" block (evidence 05).

### Finding 2: Missing baseline headers
- **Found:** header audit from iSH showed only Strict-Transport-Security (evidence 01).
- **Fixed:** added `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`, `Permissions-Policy` (camera, microphone, geolocation, payment disabled).
- **Verified:** second audit shows all 4 headers live; app still loads and plays (evidence 06).
- **Deferred:** Content-Security-Policy (needs testing; single-file app with inline code).

### Hardening 3: Upload key management
- Key generated locally (RSA 2048, PKCS12), stored **outside** the Git repo.
- Offline backup on USB, verified by opening it and matching the fingerprint.
- Password stored separately from the key file.
- Key and password stored as **encrypted GitHub Actions secrets**; the key was piped directly from the file, and the password typed at a hidden prompt (never on the clipboard, in a command, or in history).

### Hardening 4: Release pipeline
- Release workflow runs **only when started by hand** (not on every push).
- Workflow has **read-only** repo permissions.
- Key is restored to a temporary folder only for the run and **deleted afterward**, even if the build fails.
- Passwords are masked (`***`) in build logs.
- **Verified:** the signed bundle's SHA-256 certificate fingerprint matches the upload key's fingerprint (evidence 07).
- Signed bundle kept outside the repo; GitHub deletes its copy after 7 days.
- Google Play App Signing will hold the final app-signing key, so a lost upload key can be reset.

## Key Lessons / Technical Facts

- Response headers are a quick, free hardening layer; check them with `curl -I`.
- X-Frame-Options DENY blocks all framing; CSP `frame-ancestors` is the modern equivalent.
- Check `git status` before committing so test files never get published.
- Treat signing keys like passwords: outside the repo, two backups, separate password, encrypted CI secrets.
- A green build check isn't proof of the right signature; compare the certificate fingerprint.
- A 404 from an API can mean a typo in the resource name, not a permissions problem.

## Future Assignment Ideas (own systems only)

- Write and test a Content-Security-Policy for Kandies in report-only mode first.
- Compare header grades before and after using a public header scanner on my own site.
- Document a key-rotation plan using Play App Signing's upload key reset.
- Pin GitHub Actions to newer versions and review workflow permissions (least privilege).

## Open Items

- Bump GitHub action versions before Node 20 is retired.
- Second offline backup of the upload key.
- Content-Security-Policy.
- Crop the masked token line from any gh auth status photo before publishing.

## Next Steps

- Privacy policy page, store listing text and graphics, Data safety and content rating drafts.
- LLC filed Friday; D-U-N-S number Mon/Tue.
- Play Console organization account; upload the signed .aab.

## Sources

- [Netlify Docs: Custom headers](https://docs.netlify.com/routing/headers/)
- [MDN: X-Frame-Options](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options)
- [OWASP: Clickjacking Defense Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Clickjacking_Defense_Cheat_Sheet.html)
- [Android Developers: Sign your app](https://developer.android.com/studio/publish/app-signing)
- [GitHub Docs: Using secrets in GitHub Actions](https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions)
- [GitHub Changelog: Deprecation of Node 20 on GitHub Actions runners](https://github.blog/changelog/2025-09-19-deprecation-of-node-20-on-github-actions-runners/)
