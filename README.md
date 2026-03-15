# safe space

my personal corner of the internet. a place to write, think out loud, and document what i'm building and learning.

**live at:** [malys-botware.github.io](https://malys-botware.github.io)

## what is this

a single-file personal site that doubles as a full CMS — pure HTML, CSS, and vanilla JS. no frameworks, no build tools. just a text editor and a browser.

every section is editable from the site itself when logged in. content is stored in a [Supabase](https://supabase.com) database (open source, MIT-licensed client). journal and dashboard entries use a [Quill](https://quilljs.com) rich text editor (BSD-3-Clause). only i can log in — public signups are disabled.

## structure

```
index.html          ← the entire site + CMS (HTML + CSS + JS in one file)
SETUP_GUIDE.md      ← step-by-step Supabase configuration
SECURITY_OVERVIEW.md ← full security rundown
LICENSE             ← MIT for code, CC BY-NC-SA 4.0 for content
README.md           ← you're here
```

## dependencies

- [Google Fonts](https://fonts.google.com) — Space Mono + IBM Plex Mono
- [Supabase JS v2](https://github.com/supabase/supabase-js) — MIT license
- [Quill v2](https://github.com/slab/quill) — BSD-3-Clause license

all loaded from CDN. no npm, no node_modules, no build step.

## editable sections

| Section | What's editable | Editor type |
|---------|----------------|-------------|
| Hero | Name, tagline, status | Text fields |
| About | Intro paragraphs | Textarea |
| Beliefs | Individual cards (add/edit/delete) | Text fields |
| Dashboard | Status tiles (draft/publish) | Rich text (Quill) |
| Projects | Project cards (add/edit/delete) | Text fields |
| Journal | Blog entries (draft/publish + share) | Rich text (Quill) |
| Connect | Social links (add/edit/delete) | Text fields + icon picker |
| Footer | Two text lines | Text fields |

## license

the **code** (HTML, CSS, JS) is [MIT](https://opensource.org/licenses/MIT) — use it, learn from it, do whatever.

the **content and artwork** (writing, journal entries, character design) is [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) — share it, but give credit, keep it non-commercial, and share alike.

## acknowledgments

built with assistance from [Claude](https://claude.ai) (Anthropic).
