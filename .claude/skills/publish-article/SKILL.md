---
name: publish-article
description: Publish articles to the Onsite Ops Quarterly site. Handles content parsing, hero image compression, HTML generation from template, and git push. Works with single articles or batch markdown files.
---

# /publish-article

Publish one or more articles to the Onsite Ops Quarterly site.

## When to use

Use this skill when the user provides article content (markdown file, pasted text, or images with article references) and wants articles published as HTML pages on the OOQ site.

## Inputs the user provides

- **Article content**: a `.md` file (often in ~/Downloads), pasted text, or reference to content
- **Hero images**: image files (usually in ~/Downloads) — matched to articles by filename keywords
- **Section**: which coverage section (e.g., "stadiums-arenas", "festivals", "crowd-management")
- **Parent category**: which parent folder — one of: `venue-types`, `operations`, `industry`, `technology`

If the user doesn't specify section or parent, infer from article content or ask.

## Batch files

A single `.md` file may contain multiple articles separated by `---`. Each article has its own title (`# Heading`), body paragraphs, optional `## Subheadings`, and a source attribution line at the end starting with `*Based on reporting by...`.

When processing a batch file, publish ALL articles from it.

## Step-by-step process

### 1. Parse articles from the content

For each article, extract:
- **Title**: the `# Heading`
- **Deck**: write a 1-2 sentence subtitle summarizing the article (not from the content — write it fresh)
- **Date**: today's date unless specified
- **Author**: "Anonymous" unless specified
- **Body**: paragraphs as `<p>` tags. If the article has `## Subheadings`, these become `<h2>` tags with numbered spans
- **Source link**: URL and outlet name from the `*Based on reporting by...` line
- **Tags**: generate 5-6 relevant topic tags

### 2. Calculate read time

Word count / 225, rounded up:
- 1-224 words = 1 min read
- 225-449 words = 2 min read
- 450-674 words = 3 min read

### 3. Generate file slugs

Lowercase, hyphens, no special characters, 5-7 words max.
Example: "MetLife Stadium Just Lost Its Name" → `metlife-stadium-lost-its-name`

### 4. Find and compress hero images

Search `~/Downloads` for image files matching article keywords:
```bash
ls -lt ~/Downloads/*.jpg ~/Downloads/*.jpeg ~/Downloads/*.png 2>/dev/null | head -30
```

Look for filename patterns that match article topics (e.g., `sxsw_article.jpg` for an SXSW article, `coachella_ferris_article.jpg` for a Coachella ferris wheel article). If the user attached images, they may also be in Downloads with recent timestamps.

Create the section image directory if needed:
```bash
mkdir -p images/articles/[section]
```

Compress each image:
```bash
sips --resampleWidth 1920 [source] --out images/articles/[section]/[slug]-hero.jpg -s formatOptions 60
```

Check file size — target under 500KB. If over, recompress with lower quality (45 or 40):
```bash
ls -lh images/articles/[section]/[slug]-hero.jpg
# If over 500KB:
sips --resampleWidth 1920 [source] --out images/articles/[section]/[slug]-hero.jpg -s formatOptions 45
```

If no image found, use `../../../OOQ_hero_home.jpg` as the hero path.

### 5. Create article HTML files

**The fastest method**: copy an existing article as a template, then edit only the changing parts.

Find an existing article to use as template:
```bash
ls articles/venue-types/*/  # or articles/operations/*/ etc.
```

Pick ANY existing `.html` article — they all share the same CSS, nav, share bar, subscribe form, footer, and JavaScript. A good default template:
```
articles/venue-types/amphitheaters/vail-amphitheater-19m-makeover.html
```

Copy it:
```bash
cp articles/[parent]/[template-section]/[template].html articles/[parent]/[section]/[new-slug].html
```

Then edit these 9 sections (everything else stays identical):

#### 5a. `<title>` tag (line 6)
```html
<title>[Article Title] — Onsite Ops Quarterly</title>
```

#### 5b. Hero image URL in CSS (around line 228)
Find `background: url('` in the `.article-hero` block and replace:
```css
background: url('../../../images/articles/[section]/[slug]-hero.jpg') center center / cover no-repeat;
```

#### 5c. Article category span (around line 917)
```html
<span class="article-category">[Section Display Name]</span>
```
Display names: "Stadiums & Arenas", "Amphitheaters", "Festivals", "Convention Centers", "Crowd Management", etc.

#### 5d. Article title h1 (around line 918)
```html
<h1 class="article-title">[Title]</h1>
```

#### 5e. Article deck (around line 919)
```html
<p class="article-deck">[Deck text]</p>
```

#### 5f. Article meta (around line 920-925)
Only change if read time differs. The date/author format is already correct for today.

#### 5g. Article body (between `<article class="article-body">` and `</article>`)

**For 1-2 min reads (short-form)**:
- First `<p>` gets `class="article-lead"` (gold drop cap)
- 3-6 plain `<p>` tags, no headers
- Source attribution as last element

**For 3+ min reads (long-form)**:
- First `<p>` gets `class="article-lead"`
- `<h2>` headers with numbered spans: `<h2><span class="article-h2-num">01</span>Section Title</h2>`
- Multiple `<p>` tags per section
- Source attribution as last element

Source attribution format:
```html
<p style="margin-top: 48px; padding-top: 24px; border-top: 1px solid var(--border-light); font-size: 13px; color: var(--text-tertiary); font-style: italic;">Based on reporting by [Outlet]: <a href="[URL]" target="_blank" rel="noopener" style="color: var(--accent); text-decoration: none;">[Link Text]</a></p>
```

**HTML entities**: Use `&mdash;` for em dashes, `&ndash;` for en dashes, `&amp;` for ampersands, `&egrave;` for è, `&euml;` for ë, `&Uuml;` for Ü, etc.

#### 5h. Tags (after `</article>`)
```html
<div class="article-tags">
  <a href="#" class="article-tag">[Tag 1]</a>
  <a href="#" class="article-tag">[Tag 2]</a>
  ...
</div>
```

#### 5i. Related articles grid

Update the section label:
```html
<div class="section-label">More in [Section Display Name]</div>
```

Replace the 3 related cards with links to other REAL articles in the same section. If publishing a batch, link to the other articles in the batch. Each card:
```html
<a href="[other-slug].html" class="related-card">
  <div class="related-card-img" style="background-image: url('../../../images/articles/[section]/[other-slug]-hero.jpg');"></div>
  <div class="related-card-body">
    <span class="related-card-category">[Section Display Name]</span>
    <h3 class="related-card-title">[Other Article Title]</h3>
    <p class="related-card-excerpt">[1-2 sentence excerpt]</p>
    <span class="related-card-meta">Anonymous &bull; [X] min read</span>
  </div>
</a>
```

If fewer than 3 other articles exist in the section, use articles from other sections or leave fewer cards.

### 6. Move source markdown

If the user provided a `.md` file:
```bash
mv ~/Downloads/[file].md articles/[parent]/[section]/
```

### 7. Commit and push

Make sure git auth is on the right account:
```bash
gh auth status
# If needed: gh auth switch --user jmbradley
```

Then:
```bash
git add articles/[parent]/[section]/ images/articles/[section]/
git commit -m "Add [N] [section] articles with compressed hero images

[Brief list of article titles]. All [X] min reads with source
attribution and cross-linked related articles.

Co-Authored-By: Claude Opus 4.6 (1M context) <noreply@anthropic.com>"
git push origin main
```

### 8. Update homepage link (if first articles in a new section)

Check if the section is linked from the homepage coverage grid:
```bash
grep -n "[section display name]" index.html
```

If the `<li>` has no `<a>` link, add one pointing to the first article:
```html
<li><a href="articles/[parent]/[section]/[first-slug].html" style="color: inherit; text-decoration: none;">[Section Display Name]</a></li>
```

Then add `index.html` to the commit.

## Directory structure

```
articles/
  venue-types/        → stadiums-arenas, amphitheaters, festivals, convention-centers,
                        fairgrounds, raceways, performing-arts, casinos-resorts,
                        immersive-venues, pop-up-venues
  operations/         → crowd-management, logistics-site-planning, load-in-out,
                        parking-transportation, safety-medical, weather-contingency,
                        credentialing-access, permitting-regulatory, merch-operations,
                        experiential-advertising
  industry/           → latest-news, contracts-procurement
  technology/         → tech-tools, data-analytics

images/articles/[section]/   → hero images per section
```

## Template paths

All articles are 3 levels deep. Internal links use `../../../`:
- `../../../index.html` → homepage
- `../../../about.html` → about page
- `../../../images/articles/[section]/` → hero images
- Relative filenames for same-section cross-links (e.g., `other-article.html`)

## Section display names

| Slug | Display Name |
|------|-------------|
| stadiums-arenas | Stadiums & Arenas |
| amphitheaters | Amphitheaters |
| festivals | Festivals |
| convention-centers | Convention Centers |
| fairgrounds | Fairgrounds |
| raceways | Raceways |
| performing-arts | Performing Arts |
| casinos-resorts | Casinos & Resorts |
| immersive-venues | Immersive Venues |
| pop-up-venues | Pop-Up Venues |
| crowd-management | Crowd Management |
| logistics-site-planning | Logistics & Site Planning |
| load-in-out | Load In / Load Out |
| parking-transportation | Parking & Transportation |
| safety-medical | Safety & Medical |
| weather-contingency | Weather & Contingency |
| credentialing-access | Credentialing & Access |
| permitting-regulatory | Permitting & Regulatory |
| merch-operations | Merch Operations |
| experiential-advertising | Experiential & Advertising |
| latest-news | Latest News |
| contracts-procurement | Contracts & Procurement |
| tech-tools | Tech & Tools |
| data-analytics | Data & Analytics |

## Example invocation

User says:
> /publish-article here are 4 articles for festivals, photos are attached and in Downloads

The skill then:
1. Reads/parses the markdown (single or batch)
2. Finds matching images in ~/Downloads by keyword + recent timestamps
3. Compresses images → `images/articles/festivals/[slug]-hero.jpg`
4. Copies template → `articles/venue-types/festivals/[slug].html` for each article
5. Edits only the 9 changing sections per file
6. Cross-links all articles to each other in the related grid
7. Moves the `.md` to the festivals directory
8. Updates homepage link if this is the first article in the section
9. Commits and pushes to main
