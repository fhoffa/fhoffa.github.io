# AGENTS.md — fhoffa.github.io

This repo is Felipe Hoffa's personal homepage and public navigation layer.

## What we are doing here

The site should help a new visitor quickly understand Felipe's work across:

- data systems and infrastructure
- AI, agents, evaluations, and protocols
- Geotab, telematics, and real-world fleet data
- career/identity transitions and community building
- talks, demos, videos, and public writing

Treat the homepage as a curated front door, not an exhaustive archive. It should answer:

1. What has Felipe built?
2. What does Felipe think about?
3. Where should I click next if I care about a theme?

## Current site structure

- `index.html` — homepage and primary editorial surface.
- `posts.html` — topic index with counts.
- `posts/topics/*.html` — selected LinkedIn posts grouped by theme.
- `images/linkedin/` — local preview images for cards.
- `linkedin-posts-catalog.json` — manual/curated source inventory of posts; not currently a build source.
- `stylesheets/styles.css` — shared layout and card styling.

There is no build system right now. Edits are static HTML and should be kept simple.

## Repo routing rule

In this Telegram GitHub topic, default GitHub/profile/link-highlight work belongs in:

`https://github.com/fhoffa/fhoffa.github.io/`

Do **not** route this topic's homepage/profile/link work to `google-cloud-next-2026-unofficial-scrape` or `geotab-vibe-guide` unless Felipe explicitly names those repos.

## Content placement rules

Use the information architecture deliberately:

- **Projects**: concrete artifacts Felipe built or maintains.
  - Good: Google Cloud Next 2026 unofficial scrape, knowledge graph project, Geotab vibe guide.
  - Avoid: generic video playlists or standalone posts unless they are clearly a project artifact.
- **AI & agents**: model behavior, MCP/A2A/protocols, evals, agent systems, OpenClaw, Geotab Ace MCP.
- **Data & infrastructure**: BigQuery, Snowflake, Databricks, SQL, platforms, data communities.
- **Geotab & telematics**: fleet data, traffic, hackathon, Geotab Ace, applied real-world mobility.
- **Career & identity**: personal career arc, transitions, identity, immigration/citizenship.
- **Stories & community**: San Francisco, conferences, community, people, serendipity.
- **Videos & talks**: playlists, talks, interviews, demos, long-running public video archive.

If a post spans multiple themes, it can appear in multiple topic pages, but the homepage should stay selective.

## Current audit notes

- The homepage structure is coherent: curated intro → projects → major themes → videos → full archive.
- The YouTube career playlist belongs in **Videos & talks** plus the header/sidebar, not in Projects.
- The Geotab Ace MCP demo belongs in both **AI & agents** and **Geotab & telematics** topic pages.
- Topic counts in `posts.html` must be updated manually when adding/removing topic cards.
- Cards with local images feel more polished; if an image exists in `images/linkedin/`, use it.
- `linkedin-posts-catalog.json` is useful context but the current pages are manually maintained. Do not assume editing the JSON updates HTML.

## How to improve next

Highest-leverage improvements:

1. Add a tiny generation script so `linkedin-posts-catalog.json` can produce `posts.html` and `posts/topics/*.html` deterministically.
2. Add a local link/image checker script and run it before every PR.
3. Normalize card data: title, summary, topic groups, reactions/comments, image path, canonical URL.
4. Add a dedicated `videos.html` page if the YouTube archive grows beyond one playlist link.
5. Consider a featured project image for the knowledge graph card so Projects does not look less polished than post sections.
6. Keep homepage cards concise; topic pages can be more exhaustive.

## PR workflow

Felipe prefers PR-based changes:

1. Work on a branch.
2. Keep unrelated repo changes out of the PR.
3. Do not merge without Felipe's explicit approval.
4. In the PR body, include what changed and verification.
5. If you open a wrong PR, close it immediately and say so plainly.

## Verification checklist

Before saying a PR is ready:

```bash
git diff --check
python3 -m http.server 8899
```

Then inspect at least:

- `/`
- `/posts.html`
- any edited `/posts/topics/*.html`

Also run a local reference check for relative links/images:

```bash
python3 - <<'PY'
from html.parser import HTMLParser
from pathlib import Path
from urllib.parse import unquote
class Parser(HTMLParser):
    def __init__(self): super().__init__(); self.refs=[]
    def handle_starttag(self, tag, attrs):
        d=dict(attrs)
        if tag == 'a' and d.get('href'): self.refs.append(d['href'])
        if tag == 'img' and d.get('src'): self.refs.append(d['src'])
missing=[]
root=Path('.').resolve()
for page in Path('.').glob('**/*.html'):
    parser=Parser(); parser.feed(page.read_text(errors='ignore'))
    for ref in parser.refs:
        if ref.startswith(('http://','https://','mailto:','#')) or ref == '/':
            continue
        target=(page.parent / unquote(ref.split('#')[0].split('?')[0])).resolve()
        try: target.relative_to(root)
        except ValueError: continue
        if ref and not target.exists(): missing.append((str(page), ref))
if missing:
    for page, ref in missing: print(f'{page}: missing {ref}')
    raise SystemExit(1)
print('local refs ok')
PY
```
