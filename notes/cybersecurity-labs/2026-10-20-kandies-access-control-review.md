# Session Notes: Kandies Web App, Access-Control Review
Date: 2026-10-20 (Eastern Time, UTC-04:00)

## Session Time
- Block 1: 17:06 to 18:38 (92 min)
- Block 2: 19:17 to 19:27 (10 min)
- Block 3: 20:48 (wrap-up)
- Total active time: ~103 min
- Total elapsed: 3 hr 42 min

## Goal
Build detailed admin statistics for my own app (Kandies), and review who is allowed to change what in the database.

## Setup
- App: Kandies (my own app), Supabase backend (PostgreSQL + Row Level Security)
- Testing: local copy at localhost:8080, Supabase SQL Editor
- Scope: my own app, my own accounts only

## Timeline
| Time | Event |
|------|-------|
| 17:07 | Found stats undercount: browser counting capped at 1,000 rows |
| 17:10 | Account data: 17 claimed artist profiles vs only 14 artist accounts (mismatch) |
| 18:25 | Root cause: "Become an artist" created artist profiles without updating account type |
| 18:32 | Fixed 3 mismatched accounts |
| 18:33 | Read database security rules (policies, functions, triggers) |
| 18:35 | Found: profile update rule had no column limits |
| 19:17 | Confirmed with a privilege check: regular users could change protected profile fields |
| 19:27 | Chose to verify and apply the fix myself |
| 20:48 | Reported fixed and verified |

## Problem Log
| # | Problem | Found | Fixed | Time to fix | Cause |
|---|---------|-------|-------|-------------|-------|
| 1 | Stats undercount | 17:07 | 18:23 | 76 min | Client-side counting hit the 1,000-row API limit |
| 2 | Artist profiles on listener accounts | 17:10 | 18:32 | 82 min | Feature skipped updating account type |
| 3 | Users could change protected profile fields | 18:35 | 20:48 | ~2 hr 13 min | Update rule allowed any column on your own row |
| 4 | File overwritten to 120 bytes | 17:57 | 18:05 | 8 min | Clipboard reused for a command |

## Security Findings
**Finding (high severity): broken access control on user profiles**
- What: the database rule letting users edit their own profile didn't restrict which fields. Protected fields (role and account type) were editable by the account owner.
- Category: OWASP A01 Broken Access Control / privilege escalation.
- Evidence: database privilege check returned "true" for user updates on protected columns.
- Fix (two independent layers):
  1. Column privileges: users may only update their display name.
  2. Database trigger that rejects changes to protected fields unless they come from the server or the official one-way listener → artist switch.
- Verification: re-checked privileges after the fix; tested before and after on my own test account.

**Other findings (open)**
- Artist profile update rule has no check on which fields change (owner, claim code, Ohio tag).
- Any account can create an artist profile directly (bypasses the account-type rule).
- Signup trusts the account type sent by the browser.

## Key Lessons
- Row Level Security controls WHICH rows you can touch, not WHICH columns. Columns need their own limits.
- Check privileges directly with a query; don't assume.
- Defense in depth: two independent locks on the same weakness.
- UI buttons aren't security. Enforce rules in the database.
- Least privilege: users get only the fields they need (display name).

## Future Assignment Ideas (own systems only)
1. Write up the profile finding as a formal report: description, impact, evidence, fix, verification.
2. Audit every table's rules in my app for missing column limits.
3. Compare RLS policies vs column privileges vs triggers, with pros and cons of each.

## Open Items
- Fix the three open findings above
- Attach my before/after screenshots to this note

## Next Steps
- Build the one-way listener → artist switch on the database side
- Lock artist profile fields

## Sources
- OWASP Top 10, A01 Broken Access Control: https://owasp.org/Top10/A01_2021-Broken_Access_Control/
- Supabase Row Level Security: https://supabase.com/docs/guides/database/postgres/row-level-security
- PostgreSQL privileges: https://www.postgresql.org/docs/current/ddl-priv.html
