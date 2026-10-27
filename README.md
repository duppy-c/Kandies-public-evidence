# Kandies Public Evidence

Public evidence and build history for Kandies, a free music streaming app for Ohio artists at kandiesapp.com. Artist profiles, Ohio First spotlight, listener and artist accounts, music, promos and merch links, built with a single-file app on Netlify, Supabase and Cloudflare R2, with Android and Apple Store releases on the way.

## Evidence: Oct 23 and Oct 26, 2026 (public-safe copy)

Cleaned for public repos:

- Supabase dashboard photos (Oct 26 #04, #05, #08, #09, #11, #12): browser address bar and dashboard header blurred (project slug and account name).
- Oct 26 #13: the email typed into the signup test is blurred.
- Oct 26 #06 and #07 left out on purpose: they show a flawed first test of the plays fix. That test looked up "another user" while acting as the test user, who can't see other profiles, so it logged a play with no user (which is allowed) and passed both before and after the fix. #08 is the corrected test and shows the fix working.

The unblurred originals, including #06 and #07, stay in the private evidence set only.

## Contents (once photos are uploaded)

- `2026-10-23/` — 4 photos: Terms update, Netlify publish, unsafe-link tests in Safari and Chrome
- `2026-10-26/` — 11 photos: safe links, Security Advisor, user-count block, corrected plays test (#08), play-history block, private likes publish, leaked-password settings and rejection test (#13, blurred)

Full build and lab notes for these sessions are in `duppy-c/kandies-build-notes` (`2026-10-23_26.md`) and `duppy-c/cybersecurity-labs` (`session-notes/2026-10-23_26-kandies-security-sweep.md`).
