# Supabase Setup Guide — Full CMS

Step-by-step instructions to get all editable sections working.
Time: ~20 minutes.

---

## Step 1: Create a Supabase Account & Project

Already done if you followed the previous guide. If not:

1. Go to [https://supabase.com](https://supabase.com) and sign up (free tier)
2. Click **"New Project"**, name it, set a DB password, choose Canada (Central)
3. Click **"Create new project"** and wait ~2 minutes

---

## Step 2: Get Your API Keys

1. Go to **Settings → API** in your Supabase dashboard
2. Copy **Project URL** and **anon/public key**
3. In `index.html`, replace only the first two lines (leave the third line as-is):

```javascript
const SUPABASE_URL = 'https://your-project.supabase.co';
const SUPABASE_ANON_KEY = 'eyJhbGciOi...your-key';
const isSupabaseConfigured = SUPABASE_URL !== 'YOUR_SUPABASE_URL' && ...  // DON'T CHANGE THIS LINE
```

---

## Step 3: Create All Database Tables

**IMPORTANT:** If you already have `dashboard_tiles` and `journal_entries` tables from the previous setup, you need to drop them first so the new versions (with updated columns) can be created cleanly. Run this BEFORE the main SQL:

```sql
-- ONLY run these if you have existing tables from the previous setup
DROP TABLE IF EXISTS dashboard_tiles CASCADE;
DROP TABLE IF EXISTS journal_entries CASCADE;
```

Now run the full table creation SQL in the **SQL Editor**:

```sql
-- ============================================================
-- TABLE: site_settings (key-value store for hero, about, footer)
-- ============================================================

CREATE TABLE site_settings (
  key TEXT PRIMARY KEY,
  value TEXT NOT NULL,
  user_id UUID REFERENCES auth.users(id),
  updated_at TIMESTAMPTZ DEFAULT now()
);

ALTER TABLE site_settings ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Public can read settings" ON site_settings FOR SELECT USING (true);
CREATE POLICY "Owner can upsert settings" ON site_settings FOR INSERT TO authenticated WITH CHECK (auth.uid() = user_id);
CREATE POLICY "Owner can update settings" ON site_settings FOR UPDATE TO authenticated USING (auth.uid() = user_id) WITH CHECK (auth.uid() = user_id);


-- ============================================================
-- TABLE: beliefs
-- ============================================================

CREATE TABLE beliefs (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  title TEXT NOT NULL,
  content TEXT NOT NULL,
  sort_order INTEGER DEFAULT 0,
  user_id UUID REFERENCES auth.users(id),
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

ALTER TABLE beliefs ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Public can read beliefs" ON beliefs FOR SELECT USING (true);
CREATE POLICY "Owner can create beliefs" ON beliefs FOR INSERT TO authenticated WITH CHECK (auth.uid() = user_id);
CREATE POLICY "Owner can update beliefs" ON beliefs FOR UPDATE TO authenticated USING (auth.uid() = user_id) WITH CHECK (auth.uid() = user_id);
CREATE POLICY "Owner can delete beliefs" ON beliefs FOR DELETE TO authenticated USING (auth.uid() = user_id);


-- ============================================================
-- TABLE: dashboard_tiles (with published/draft)
-- ============================================================

CREATE TABLE dashboard_tiles (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  title TEXT NOT NULL,
  content TEXT NOT NULL,
  published BOOLEAN DEFAULT false,
  sort_order INTEGER DEFAULT 0,
  user_id UUID REFERENCES auth.users(id),
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

ALTER TABLE dashboard_tiles ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Public can read published tiles" ON dashboard_tiles FOR SELECT USING (published = true);
CREATE POLICY "Owner can read all tiles" ON dashboard_tiles FOR SELECT TO authenticated USING (auth.uid() = user_id);
CREATE POLICY "Owner can create tiles" ON dashboard_tiles FOR INSERT TO authenticated WITH CHECK (auth.uid() = user_id);
CREATE POLICY "Owner can update tiles" ON dashboard_tiles FOR UPDATE TO authenticated USING (auth.uid() = user_id) WITH CHECK (auth.uid() = user_id);
CREATE POLICY "Owner can delete tiles" ON dashboard_tiles FOR DELETE TO authenticated USING (auth.uid() = user_id);


-- ============================================================
-- TABLE: projects
-- ============================================================

CREATE TABLE projects (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  title TEXT NOT NULL,
  tags TEXT DEFAULT '',
  description TEXT NOT NULL,
  link_url TEXT DEFAULT '',
  link_text TEXT DEFAULT '',
  sort_order INTEGER DEFAULT 0,
  user_id UUID REFERENCES auth.users(id),
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

ALTER TABLE projects ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Public can read projects" ON projects FOR SELECT USING (true);
CREATE POLICY "Owner can create projects" ON projects FOR INSERT TO authenticated WITH CHECK (auth.uid() = user_id);
CREATE POLICY "Owner can update projects" ON projects FOR UPDATE TO authenticated USING (auth.uid() = user_id) WITH CHECK (auth.uid() = user_id);
CREATE POLICY "Owner can delete projects" ON projects FOR DELETE TO authenticated USING (auth.uid() = user_id);


-- ============================================================
-- TABLE: journal_entries (with published/draft)
-- ============================================================

CREATE TABLE journal_entries (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  title TEXT NOT NULL,
  entry_date TEXT NOT NULL,
  content TEXT NOT NULL,
  published BOOLEAN DEFAULT false,
  user_id UUID REFERENCES auth.users(id),
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

ALTER TABLE journal_entries ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Public can read published entries" ON journal_entries FOR SELECT USING (published = true);
CREATE POLICY "Owner can read all entries" ON journal_entries FOR SELECT TO authenticated USING (auth.uid() = user_id);
CREATE POLICY "Owner can create entries" ON journal_entries FOR INSERT TO authenticated WITH CHECK (auth.uid() = user_id);
CREATE POLICY "Owner can update entries" ON journal_entries FOR UPDATE TO authenticated USING (auth.uid() = user_id) WITH CHECK (auth.uid() = user_id);
CREATE POLICY "Owner can delete entries" ON journal_entries FOR DELETE TO authenticated USING (auth.uid() = user_id);


-- ============================================================
-- TABLE: social_links
-- ============================================================

CREATE TABLE social_links (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  name TEXT NOT NULL,
  handle TEXT DEFAULT '',
  url TEXT NOT NULL,
  icon_key TEXT DEFAULT 'website',
  sort_order INTEGER DEFAULT 0,
  user_id UUID REFERENCES auth.users(id),
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

ALTER TABLE social_links ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Public can read social links" ON social_links FOR SELECT USING (true);
CREATE POLICY "Owner can create social links" ON social_links FOR INSERT TO authenticated WITH CHECK (auth.uid() = user_id);
CREATE POLICY "Owner can update social links" ON social_links FOR UPDATE TO authenticated USING (auth.uid() = user_id) WITH CHECK (auth.uid() = user_id);
CREATE POLICY "Owner can delete social links" ON social_links FOR DELETE TO authenticated USING (auth.uid() = user_id);
```

---

## Step 4: Create Your Admin Account

Skip if you already did this. Otherwise:

1. Go to **Authentication → Users**
2. **Add User → Create New User** with your email + strong password
3. Check **Auto Confirm User**
4. **Copy your User ID** (the UUID)

---

## Step 5: Disable Public Signups

Skip if already done. Otherwise:

1. **Authentication → Providers → Email → Enable Sign Ups → OFF**

---

## Step 6: Seed Your Existing Data

Replace `YOUR_USER_ID_HERE` with your UUID from Step 4.

```sql
-- SITE SETTINGS
INSERT INTO site_settings (key, value, user_id) VALUES
('hero_name', 'malys', 'YOUR_USER_ID_HERE'),
('hero_tagline', 'learning to code a dozen rabbitholes at a time, tinkering with linux, building things, breaking things, learning as much as possible, and trying to enjoy the process without burning out.', 'YOUR_USER_ID_HERE'),
('hero_status', 'doin stuff/researching', 'YOUR_USER_ID_HERE'),
('about_intro', 'I went from being a casual windows consumer to installing linux on a dusty alienware laptop and falling into the deepest rabbithole of my life. One month in and I''ve already shipped my first repo, broken my system more times than I can count, and realized that writing things out is maybe a little therapeutic.

Some of the 37signals really resonate with me such as the idea that you can build something real without burning yourself to the ground. That simplicity isn''t laziness, it''s discipline. That you don''t need permission to start, you just need to start.', 'YOUR_USER_ID_HERE'),
('footer_line1', 'built by thought, one prompt or line at a time.', 'YOUR_USER_ID_HERE'),
('footer_line2', 'keep on keepin'' on.', 'YOUR_USER_ID_HERE');

-- BELIEFS
INSERT INTO beliefs (title, content, sort_order, user_id) VALUES
('learn by building, not just consuming', 'Grinding through lessons has its place, but the real learning happens when you make something, break it, and figure out why. My first repo is small and kinda rough but it taught me more than any tutorial.', 1, 'YOUR_USER_ID_HERE'),
('ship it, even if it''s embarrassing', 'Perfectionism is the enemy. Put it out there. You can''t get feedback on code that lives only on your machine. The act of making something public changes how you think about it.', 2, 'YOUR_USER_ID_HERE'),
('slow down to go faster', 'The can''t-stop-won''t-stop mindset is fun until you crash. Sustainable pace beats heroic sprints. 37signals runs on 40-hour weeks and they''ve been profitable for 25 years. There''s something to that.', 3, 'YOUR_USER_ID_HERE'),
('own your space', 'This site exists because a reddit post got deleted before anyone could see it. That''s fine. But it''s a reminder: if you want a voice that can''t be moderated away, you need your own corner of the internet.', 4, 'YOUR_USER_ID_HERE'),
('ai is a tool, not a crutch', 'I use AI. I''m not going to pretend I don''t. But the goal is to understand what the code does, not just to generate it. The basics matter. You can''t debug what you don''t understand.', 5, 'YOUR_USER_ID_HERE'),
('keep going, keep writing', 'Writing things out organizes thoughts better than any productivity system. This whole site started because I realized journaling helps me process what I''m learning. So here we are.', 6, 'YOUR_USER_ID_HERE'),
('sleep on it. seriously. just do it.', 'instant gratification is great and all but isn''t gonna be sustainable, start planning a little more thoroughly, don''t get stagnant though.', 7, 'YOUR_USER_ID_HERE');

-- DASHBOARD TILES
INSERT INTO dashboard_tiles (title, content, published, sort_order, user_id) VALUES
('System', '<p>D.E.''s: Cosmic and i3 till im happy with the i3 config things to fix in i3:</p><p>- some apps not working properly or as expected</p><p>ex. - mission control opens to blank/translucent window</p><p>- garuda toolbox can''t run certain diags or sys update</p><p>general: lockscreen needs theme still</p>', true, 1, 'YOUR_USER_ID_HERE'),
('Lessons', '<p>- Onward from ch3 at last! got to ch5 l2</p>', true, 2, 'YOUR_USER_ID_HERE'),
('This Site', '<p>- tweak "hero"</p>', true, 3, 'YOUR_USER_ID_HERE'),
('Keybindings Cheatsheet / Editor', '<p>undecided where i want to go with it rn, seems to be functioning as expected.</p><p>gonna try to learn how exactly it works before going too much farther.</p>', true, 4, 'YOUR_USER_ID_HERE');

-- PROJECTS
INSERT INTO projects (title, tags, description, link_url, link_text, sort_order, user_id) VALUES
('i3 Keybinding Cheatsheet & Editor', 'Python, Web UI, i3wm', 'My first repo. Parses an i3 config file and serves a local web UI for viewing and editing keybindings. Nothing huge, almost kinda embarrassing, but it felt good to put something real out on the internet. And I learned a ton building it.', 'https://github.com/Malys-Botware/i3-keybindings-cheatsheet', 'view on github →', 1, 'YOUR_USER_ID_HERE'),
('This Website', 'HTML, CSS, JS, Supabase', 'A single-file personal site built to be readable, learnable, and easy to modify. No frameworks. No build steps. Just a text editor and a browser. An ongoing project to learn web development while having something to show for it.', 'https://github.com/Malys-Botware/Malys-Botware.github.io', 'view source →', 2, 'YOUR_USER_ID_HERE');

-- JOURNAL ENTRIES
INSERT INTO journal_entries (title, entry_date, content, published, user_id, created_at) VALUES
('lesson day', 'March 14, 2026', '<p>how many lessons can you get through? be sure to use potions if you have the gems! don''t forget about the other usefull tools at your disposal like embers, potions, quests and more!</p><p>talk to boots if needed! he can be rather helpful if you talk through your problem with him!</p><p>go back and review your previous couple lessons or maybe even chapters! especially if you had an extended session previously</p>', true, 'YOUR_USER_ID_HERE', '2026-03-14T12:00:00Z'),
('veg day', 'March 13, 2026', '<p>just a day to sit back and breath. lay back, get comfy and poke around in the tools you''re learning or go browse some youtube / twitch / whatever tickles your fancy. or maybe you''ve been putting off a project around the house, gaming? nows the day</p><p>what i got up to today: explored github some more. all commits updating the site today done from github, without AI. just the keyboard and a little mouse action here and there thought about getting back into boot.dev but just need to let my brain settle. watched some youtube. answered a q wrong on x shoulda been pwd - print working directory shows absolute path ended up doing a couple/few lessons</p><p>fairly light easy day, weather coulda beem nicer, plans for tomorrow? hmm.</p>', true, 'YOUR_USER_ID_HERE', '2026-03-13T12:00:00Z'),
('the post that started this', 'March 12, 2026', '<p>I wrote a long post on r/learnprogramming. Put real thought into it — where I''m at, what I''m struggling with, honest questions about balancing learning with building, time management, and how an introvert starts connecting with other people in tech. The mods removed it before anyone could even read it.</p><p>And you know what? No ill will. The sub has rules and I probably should''ve read the FAQ more carefully. But it did help me realize something: I need my own space. Somewhere my thoughts can exist without getting caught in someone else''s filter.</p><p>So here we are. This site is that space. It started as frustration but turned into something I''m actually excited about. Just writing that reddit post — even though nobody saw it — helped me organize my thoughts. Turns out writing things down is weirdly therapeutic. Who knew.</p><p>I''m about a month into learning to code. Started by installing linux on an old alienware laptop. Put my first repo on github yesterday — an i3 keybinding cheatsheet and editor. It''s small but it''s real and it''s mine. Every new thing I learn opens another door and then I''m down the next rabbithole. Can''t stop won''t stop... until I crash. Then repeat.</p><p>I know I need to slow down. I know I rely on AI too much. I know I should talk to actual humans more. I''m working on all of it. But at least now I have somewhere to write about the process while I figure it out.</p><p>Gonna keep on keepin'' on, I suppose.</p>', true, 'YOUR_USER_ID_HERE', '2026-03-12T12:00:00Z');

-- SOCIAL LINKS
INSERT INTO social_links (name, handle, url, icon_key, sort_order, user_id) VALUES
('GitHub', 'Malys-Botware', 'https://github.com/Malys-Botware', 'github', 1, 'YOUR_USER_ID_HERE'),
('X', '@Malys520572', 'https://x.com/Malys520572', 'x', 2, 'YOUR_USER_ID_HERE'),
('Reddit', 'u/Electrical_Funny6177', 'https://reddit.com/u/Electrical_Funny6177', 'reddit', 3, 'YOUR_USER_ID_HERE');
```

---

## Step 7: Test It

1. Push the updated `index.html` to your GitHub Pages repo
2. Visit your site — all content should load from the database
3. Click the power icon next to "Malys" → log in
4. You should now see edit controls on EVERY section:
   - **Hero**: "edit" button next to your name
   - **About**: "edit intro" button + add/edit/delete beliefs
   - **Dashboard**: add/edit/delete/publish/unpublish tiles
   - **Projects**: add/edit/delete project cards
   - **Journal**: add/edit/delete/publish/unpublish entries + share buttons
   - **Connect**: add/edit/delete social links
   - **Footer**: "edit" button below the footer text
5. Try editing each section — changes save to the database
6. Log out and verify everything looks correct to visitors

---

## Database Tables Summary

| Table | Content | Public Access | Admin Access |
|-------|---------|--------------|--------------|
| site_settings | Hero, about intro, footer text | Read all | Read + write |
| beliefs | Belief cards in About section | Read all | Full CRUD |
| dashboard_tiles | Dashboard status tiles | Read published only | Full CRUD + publish/unpublish |
| projects | Project cards | Read all | Full CRUD |
| journal_entries | Journal blog entries | Read published only | Full CRUD + publish/unpublish |
| social_links | Connect section links | Read all | Full CRUD |

---

## Troubleshooting

**"relation already exists" error when running Step 3:**
You already have tables from a previous setup. Run the DROP commands at the top of Step 3 first, then re-run.

**Content not loading after setup:**
Check browser console (F12) for errors. Most common issue is the user_id not matching — double-check your UUID.

**Can't edit after logging in:**
Make sure the user_id in your seed data matches your actual Supabase user UUID exactly.
