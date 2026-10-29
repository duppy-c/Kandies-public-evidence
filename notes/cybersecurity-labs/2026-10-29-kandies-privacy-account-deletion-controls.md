# Kandies Privacy and Account Deletion Controls: October 29, 2026

**Time zone:** America/New_York (EDT)
**Course context:** CYB/130T, applied practice on my own system
**System:** Kandies (my own app): Netlify site, Supabase Postgres/Auth with row-level security, GitHub Actions release build

---

## Session Time

| Block | Start | End | Active |
|---|---|---|---|
| 1 | 06:57 | 08:03 | 1 h 06 m |
| Interruption | 08:03 | 09:48 | |
| 2 | 09:48 | 09:52 | 4 m |

**Active:** 1 h 10 m. **Elapsed:** 2 h 55 m.

## Goal

Add the privacy and data-control features Google Play requires (privacy policy and in-app account deletion), designed so a user can only ever delete their own data.

## Setup

- Windows PC: PowerShell, Chrome, local test server (localhost:8080)
- Supabase SQL Editor (owner access)
- Practice Postgres 16 copy mirroring the live table links, used to test the delete function first
- GitHub Actions release workflow (signed .aab)

## Timeline

| Time | Event |
|---|---|
| 07:00 | Privacy policy drafted from an inventory of what the app stores |
| 07:14 | Privacy policy published |
| 07:16 | Mapped every link to user accounts and their delete behavior |
| 07:18 | Checked blockers (licenses table, nullable Spotlight/Browse slots) |
| 07:22 | Delete function created; access verified (signed-out cannot run it) |
| 07:23 | Admin-only cleanup table created with row-level security |
| 07:28 | Admin self-deletion refused |
| 07:34 | Test account fully deleted and verified |
| 07:37 | Account deletion deployed |
| 08:02 | New signed .aab; fingerprint matches upload key |

## Problem Log

| # | Problem | Found | Fixed | Time to fix | Cause |
|---|---|---|---|---|---|
| 1 | No privacy policy | 06:57 | 07:14 | 17 m | Not built yet (Play requirement) |
| 2 | No way to delete an account in the app | 07:00 | 07:37 | 37 m | Not built yet (Play requirement) |
| 3 | Instruction files not delivered | 07:02 | 07:03 | 1 m | Steps sent without the file |
| 4 | Cleanup table missing after setup | 07:22 | 07:23 | 1 m | One SQL piece didn't run; function still compiled |
| 5 | Destructive button didn't look disabled | 07:26 | 07:28 | 2 m | Styling hid disabled state; placeholder looked like input |

## Security Findings / Hardening Done

### 1. Privacy policy matches real data handling
- Built from an inventory of the app's tables and scripts (only Supabase is loaded from outside; no trackers or ad networks).
- States what is public vs private, matching the Oct 26 fixes (totals public, who-did-what private).
- **Verified:** live at kandiesapp.com/privacy.html; linked from sign-in, Home footer and Terms.

### 2. Account deletion limited to the caller
- The function takes **no input**; it only acts on the signed-in user's own ID, so it can't be pointed at someone else.
- Signed-out visitors can't run it (catalog check: anon = false, authenticated = true).
- **Admin accounts are refused**, protecting the owner account from accidental deletion.
- Pinned search_path; runs with owner rights only for the steps it needs.
- **Verified:** admin refusal shown; test account deleted; that login then failed; database confirmed account, artist profile and leftovers all gone; other users untouched (practice-copy test).

### 3. Data minimization on deletion
- Personal links removed rather than keeping orphaned personal data: plays and ad clicks anonymized; bug report email removed.
- Uploaded files recorded in an **admin-only** list (row-level security on, no signed-out access) for removal, since storage files can't be deleted from SQL.

### 4. Destructive-action safeguards
- Typed confirmation ("DELETE"), clear warning of what will be lost, and a visibly disabled button until confirmed.

### 5. Release integrity
- Rebuilt signed .aab; certificate fingerprint matches the upload key.

## Key Lessons / Technical Facts

- A delete-my-data function should take no user-supplied ID; use the session identity.
- Foreign keys with NO ACTION decide the delete order: children first, then parents, then the account.
- PL/pgSQL resolves tables at run time, so creating a function doesn't prove its tables exist.
- Anonymize shared statistics (plays) instead of deleting them, so other users' numbers stay correct.
- Destructive UI needs friction (typed confirmation) and visible state.

## Future Assignment Ideas (own systems only)

- Build an admin screen to empty the file cleanup list through the Storage API, and verify files are gone.
- Write a data-retention schedule for logs and backups.
- Map each data type in the privacy policy to the exact table and access rule that protects it.

## Open Items

- Storage files of deleted accounts need an admin cleanup tool.
- Change "Kandies" to "Kandies LLC" in the privacy policy once the LLC is filed.
- Content-Security-Policy; GitHub action version bumps.

## Next Steps

- D-U-N-S number, Play Console organization account, upload the signed .aab, complete Data safety form.

## Sources

- [Google Play: Data safety section](https://support.google.com/googleplay/android-developer/answer/10787469)
- [Google Play: Account deletion requirements](https://support.google.com/googleplay/android-developer/answer/13327111)
- [Supabase Docs: Row Level Security](https://supabase.com/docs/guides/database/postgres/row-level-security)
- [PostgreSQL Docs: Foreign key ON DELETE actions](https://www.postgresql.org/docs/current/ddl-constraints.html#DDL-CONSTRAINTS-FK)
- [OWASP: Data minimization / privacy by design](https://owasp.org/www-project-top-10-privacy-risks/)
