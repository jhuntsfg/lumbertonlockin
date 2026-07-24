# Lumberton Lock In — Event Tracker Site

Static, no-login-required event site for Hunt Agency's Lumberton Lock In
(July 30–31, plus a Saturday, August 1st in-person training). Built for
agents to log activity during dial sessions and track submitted apps live,
with a separate password-protected admin page for corrections.

## Files

- **index.html** — the public page. Header with event details, two live
  trackers (Activity + Submitted Apps), a link out to the dial schedule
  (Google Sheet), and a footer for the Saturday training with the flyer
  embedded as image data.
- **admin.html** — private page for editing/deleting tracker entries.
  Requires signing in with a Supabase Auth user (see setup below). Not
  linked from the public page — access it directly at `/admin.html`.
- **supabase/setup.sql** — full database setup (tables, RLS, grants,
  policies). Already run once against the live project; kept here for
  reference or for standing up a new project from scratch.

## How data works

Both trackers write to a Supabase Postgres database via `supabase-js`,
loaded straight in the browser — no backend server needed. That's what
makes this deployable as plain static files.

- **Public visitors** (`anon` role) can insert new rows and read the
  leaderboard, but cannot edit or delete anything.
- **Signed-in admins** (`authenticated` role, via admin.html) can also
  update and delete rows.
- The Supabase **project URL** and **anon public key** are already
  hardcoded near the bottom of both index.html and admin.html (anon keys
  are meant to be public/client-side — that's normal and safe).

If you ever need to point this at a different Supabase project, update
the `SUPABASE_URL` and `SUPABASE_ANON_KEY` constants in both files, and
run `supabase/setup.sql` against the new project first.

## Editing event details

Near the top of `index.html`'s `<header class="hero">`:
- Dates, address (with a Google Maps link), Zoom link + passcode
- The "View dial schedule" button links out to a Google Sheet — update
  the `href` if that link ever changes

Dial session options (used in both the logging forms and the leaderboard
filters) are hardcoded as `<option>` lists in index.html and as the
`SESSIONS` array in admin.html — if the session times change, update both
places.

The footer flyer and header logo are embedded directly as base64 image
data inside index.html (not separate files) so they never break when
deployed — if you need to swap either image, ask Claude Code to
re-embed a new image file the same way.

## Deploying with GitHub + Vercel (auto-deploy on push)

1. Push this repo to GitHub (Claude Code can do this for you if you ask).
2. In Vercel, **Add New → Project → Import Git Repository**, select this
   repo.
3. No framework, no build command — deploy as-is (static files).
4. Every push to the main branch will now auto-deploy. No more manual
   drag-and-drop.

## Admin access

Created via Supabase **Authentication → Users → Add user** (email +
password). That's the login for `admin.html`. Keep the admin.html URL
private — it's not linked anywhere public, but it's not otherwise hidden.
