# 🗳️ Election Night Picks

Office prediction game for election night. Everyone picks winners in the big races, the leaderboard updates live as races get called.

**Conflict rule:** when you join, you check off the races you work on. Those are locked out of your score (you can still log a ❤️ heart pick for fun). Scoring is raw points — 1 per correct call, 2 for chamber control, up to +2 for nailing a margin — so more picks + more right = higher. Skipping a race is just 0.

Files: `index.html`, `logo.png`, this README.

## Deploy (5 minutes)

### 1. Supabase (shared leaderboard)
1. Create a free project at supabase.com → SQL Editor → run:

```sql
create table picks (
  name text not null,
  avatar text,
  race_id text not null,
  pick text,
  margin text,
  conflict boolean default false,
  updated_at timestamptz default now(),
  primary key (name, race_id)
);
create table results (
  race_id text primary key,
  winner text not null,
  margin text,
  updated_at timestamptz default now()
);
create table races (
  id text primary key,
  sort int default 0,
  grp text not null,
  title text not null,
  note text default '',
  pts int default 1,
  margin boolean default false,
  opts jsonb not null
);
alter table picks enable row level security;
alter table results enable row level security;
alter table races enable row level security;
create policy "anyone" on races for all using (true) with check (true);
create policy "anyone" on picks for all using (true) with check (true);
create policy "anyone" on results for all using (true) with check (true);
```

2. Project Settings → API → copy the **Project URL** and **anon public** key.

### 2. Configure
Open `index.html`, find the `CONFIG` block near the top of the script, and fill in:
- `SUPABASE_URL`
- `SUPABASE_ANON_KEY`
- `ADMIN_PIN` — whoever's calling races enters this on the Results tab
- `LOCK_TIME` — picks freeze at this time (default 7pm ET)

Leave the Supabase fields blank and it runs in **solo mode** (localStorage only) — handy for previewing.

### 3. GitHub Pages
Push `index.html` to a repo → Settings → Pages → deploy from `main` / root. Share the URL in the group chat.

## Editing races
Unlock the Results tab with the PIN → **Manage races** lets you add or delete races in-app; everyone sees changes within seconds. The `DEFAULT_RACES` list in `index.html` is just the starting slate (it's copied into Supabase the first time an admin edits). `pts` makes a race worth more; `margin: true` adds the spread bonus.

## Margin bonus
Races flagged for margin ask for the exact spread in points ("Talarico by 4"). If you also got the winner: exact = `+MARGIN_EXACT` (default 2), within `MARGIN_WINDOW` points (default 2) = `+MARGIN_CLOSE` (default 1). All in `CONFIG`. Wrong winner = no bonus. The results desk types the actual spread when calling the race.

## Notes
- The admin PIN is client-side only — this is an office toy, not Fort Knox. Anyone who reads the source can find it.
- Results and picks refresh every 8 seconds. When a race gets called, everyone's screen gets confetti.
