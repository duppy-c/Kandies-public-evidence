# Kandies Access-Control Hardening Lab: October 22, 2026

**Time zone:** America/New_York (EDT)
**Course context:** CYB/130T, applied practice on my own system
**System under test:** Kandies (my own app), Supabase Postgres + Auth, row-level security (RLS)

---

## Session Time

| Block | Start | End | Active |
|---|---|---|---|
| 1 | ~11:40 AM | 3:22 PM | ~3 h 40 m |
| Break | 3:22 PM | 4:07 PM | |
| 2 | 4:07 PM | 4:13 PM | ~6 m |
| Break | 4:13 PM | 5:02 PM | |
| 3 | 5:02 PM | 5:52 PM | ~50 m |
| Break | 5:52 PM | 6:14 PM | |
| 4 | 6:14 PM | 6:43 PM | ~30 m |

**Elapsed:** ~7 h 05 m. **Active:** ~5 h 05 m (start time approximate).

## Goal

Find and close access-control weaknesses in my own app's database, proving each one with a before test, a fix, and an after test.

## Setup

- Windows PC: PowerShell, Chrome, local test server on localhost:8080
- Supabase SQL Editor (production project, owner access)
- Testing as an ordinary signed-in user by switching the SQL session to the `authenticated` role with a test account's identity, the same identity the app uses
- Throwaway test data only; every test change was cleaned up or automatically cancelled
- Practice Postgres 16 copy used to check fixes before they touched production

## Timeline

| Time (EDT) | Event |
|---|---|
| 1:45 PM | One-way account switch + side-door block published |
| 2:10–2:19 PM | Self-service claim verified and published |
| 2:54 PM | Reviewed live RLS policies on `artists` |
| 3:06 PM | Finding 1 confirmed (before test) |
| 3:10 PM | Finding 1 fixed and retested |
| 3:17 PM | Findings 2 and 3 confirmed |
| 3:19 PM | Findings 2 and 3 fixed and retested |
| 3:20 PM | Data review: 8 existing artists exposed by Finding 3 |
| 4:13 PM | Reviewed live RLS policies on `artist_comments` |
| 5:04 PM | Finding 4 confirmed |
| 5:05–5:06 PM | Findings 4 and 5 fixed and retested |
| 5:32–6:39 PM | Report / Block controls built and verified |
| 6:42 PM | All changes deployed |

## Problem Log

| # | Problem | Found | Fixed | Time to fix | Cause |
|---|---|---|---|---|---|
| 1 | Artist could change staff-only fields on their own profile (claim code, owner, claim date) | 3:06 PM | 3:10 PM | 4 m | Update policy checked *which row*, not *which columns* |
| 2 | Signup accepted any account type the app sent | 3:17 PM | 3:19 PM | 2 m | Signup function trusted client-supplied metadata |
| 3 | New artist's public name was built from their private email | 3:17 PM | 3:19 PM | 2 m | Default name used the part of the email before @ |
| 4 | Comment author name could be faked (e.g. "Kandies Staff") | 5:04 PM | 5:06 PM | 2 m | Author name came from the app; insert policy only checked the account |
| 5 | No length limit on comments in the database | 5:04 PM | 5:06 PM | 2 m | 500-character limit existed only in the app |
| 6 | No way for users to report or block abusive commenters | 4:07 PM | 6:39 PM | ~2 h 30 m (incl. breaks) | Feature not built yet (Google Play requirement) |

## Security Findings and Hardening

### Finding 1: Column-level access on artist profiles
- **Risk:** Broken access control. An artist could edit fields only staff should control.
- **Before:** As an ordinary artist, an update to a staff-only field succeeded (evidence 13, 14; result recorded).
- **Fix:** A `BEFORE UPDATE` trigger compares old and new values and blocks changes to protected columns for non-admins. The rights-agreement date is write-once. The Ohio tag can be added by the artist but only removed by staff.
- **After:** The same update was rejected with "That artist field can only be changed by Kandies staff" (evidence 15). A normal bio edit still worked.

### Finding 2: Trusted client input at signup
- **Risk:** Mass assignment. Any value could be stored as an account type.
- **Before:** A test signup stored an invented account type (evidence 16).
- **Fix:** Allow-list in the signup function: only `listener` or `artist`; anything else becomes `listener`.
- **After:** The same test stored `listener` (evidence 17).

### Finding 3: Private data in a public field
- **Risk:** Information disclosure. Part of each new artist's email was shown publicly as their name.
- **Before:** A test artist's public name matched their email prefix (evidence 16).
- **Fix:** New artists default to "New Artist."
- **After:** Test showed "New Artist" (evidence 17).
- **Impact review:** A read-only query found 8 existing artists with email-based names. 4 look like personal emails and are queued for renaming plus a heads-up email (names not recorded here for privacy).
- **Verification method:** Tests ran inside a block that ends with a deliberate error, so the database cancelled everything. Nothing was saved and no emails were sent.

### Finding 4: Spoofable comment author
- **Risk:** Impersonation, e.g. posting as staff.
- **Before:** An ordinary user posted under the name "Kandies Staff."
- **Fix:** A `BEFORE INSERT` trigger sets the author from the account (artist name, then profile name, then "Listener"). The app no longer sends a name.
- **After:** The same attempt was saved as "Listener"; a later comment from an artist account showed the real artist name (evidence 21).

### Finding 5: Client-only input limit
- **Risk:** Abuse / storage bloat with oversized posts.
- **Fix:** Same trigger trims text, rejects empty comments, and caps them at 500 characters.
- **After:** A 501-character comment was rejected (evidence 18).

### Control 6: Report and Block
- **Reports table:** users can file a report but can't read reports or file one as someone else. Only admins can read and clear them.
- **Blocks table:** each user can only see and change their own blocks.
- **Verified:** Report reached the admin list and was removed; Block hid the comment and Unblock restored it (evidence 19, 20, 22).

## Key Lessons / Technical Facts

- An RLS `UPDATE` policy decides **which rows** a user can change, not **which columns**. Column rules need a trigger or column privileges.
- When `with_check` is NULL on an update policy, Postgres reuses the `using` check for the new row.
- Anything sent by the client (metadata, names, limits) is untrusted until the database checks it.
- `SECURITY DEFINER` functions should pin `search_path` and check the caller themselves.
- An `INSERT … RETURNING` needs read permission too. Users who can file reports but not read them must insert without reading back.
- A before/after test is the evidence. "Success" alone proves nothing.

## Future Assignment Ideas (own systems only)

- Write an RLS test checklist: for every table, test read, add, change and delete as anonymous, as an ordinary user, as the row owner, and as admin.
- Compare column-level `GRANT`s vs triggers for protecting columns, and when each is the better choice.
- Build a small automated test script that runs the before/after checks after every database change.
- Write a short privacy review of which fields in my app are public and where their values come from.

## Open Items

- Rename the 4 personal-looking email-based artist names and notify those artists.
- Terms of Service: add an abusive-content rule and say reported content is removed.
- Review evidence photos before pushing them anywhere public (some show a test account email and the project URL in the browser bar).

## Next Steps

- Android signing and `.aab` release build, keeping the signing key out of the repo and backed up.
- Google Play listing.
- Optional: listener profile names.

## Evidence

Folder `evidence-2026-10-22/`:

| File | Shows |
|---|---|
| 01–05 | Artist switch before/after, side door blocked, permanent-switch warning, confirm-email setting |
| 06–09 | Self-claim button, claimed artist page, DB check, publish |
| 10–12 | Button cleanup: no-match vs match, publish |
| 13–14 | Live artist policies (before Finding 1) |
| 15 | Finding 1 blocked after fix |
| 16 | Findings 2 and 3 confirmed |
| 17 | Findings 2 and 3 fixed |
| 18 | Finding 5: oversized comment blocked |
| 19–20 | Block / Report buttons working |
| 21 | Comment shows the real artist name |
| 22 | Admin Reports screen |
| 23 | Final publish 6:42 PM |

## Sources

- [Supabase docs: Row Level Security](https://supabase.com/docs/guides/database/postgres/row-level-security)
- [PostgreSQL docs: CREATE POLICY](https://www.postgresql.org/docs/current/sql-createpolicy.html)
- [PostgreSQL docs: Trigger functions (PL/pgSQL)](https://www.postgresql.org/docs/current/plpgsql-trigger.html)
- [OWASP Top 10: A01 Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [Google Play: User-generated content policy](https://support.google.com/googleplay/android-developer/answer/9876937)
