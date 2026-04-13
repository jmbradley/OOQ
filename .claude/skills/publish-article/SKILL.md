---
name: publish-article
description: Publish an article to the Onsite Ops Quarterly site. Handles content parsing, hero image compression, HTML generation, and git push.
---

# /publish-article

Publish a new article to the Onsite Ops Quarterly site.

## When to use

Use this skill when the user provides article content (markdown file, pasted text, or a source URL summary) and wants it published as an HTML article page on the OOQ site.

## Inputs the user provides

- **Article content**: markdown file, pasted text, or instructions to write from a source
- **Hero image**: an image file (usually in ~/Downloads) — or "placeholder" to use the default
- **Section**: which coverage section (e.g., "stadiums-arenas", "crowd-management", "festivals")
- **Parent category**: which parent folder — one of: `venue-types`, `operations`, `industry`, `technology`

If the user doesn't specify the section or parent, infer it from the article content or ask.

## Steps

### 1. Parse the article content

Extract from the provided content:
- **Title**: the article headline
- **Deck**: a 1-2 sentence subtitle/summary
- **Date**: publication date (default to today if not specified)
- **Author**: default to "Anonymous" unless specified
- **Body paragraphs**: the article text, formatted as `<p>` tags
- **Source link**: the original reporting source URL and outlet name
- **Tags**: 5-6 relevant topic tags

### 2. Calculate read time

Count words in the article body. Divide by 225 (average reading speed), round up:
- 1-224 words = 1 min read
- 225-449 words = 2 min read
- 450-674 words = 3 min read
- etc.

### 3. Generate the file slug

Convert the title to a URL-friendly slug:
- Lowercase, hyphens instead of spaces
- Remove special characters, quotes, colons
- Keep it concise (5-7 words max)
- Example: "MetLife Stadium Just Lost Its Name" → `metlife-stadium-lost-its-name`

### 4. Compress the hero image

If the user provided a hero image file:
```bash
sips --resampleWidth 1920 [source] --out images/articles/[section]/[slug]-hero.jpg -s formatOptions 60
```

Target: under 500KB. The image path in the article CSS will be:
```
../../../images/articles/[section]/[slug]-hero.jpg
```

If "placeholder" or no image provided, use `../../../OOQ_hero_home.jpg`.

### 5. Create the article HTML

Copy an existing article from the same section as a template base:
```bash
cp articles/[parent]/[section]/[existing-article].html articles/[parent]/[section]/[new-slug].html
```

Then edit ONLY these parts (the CSS, nav, footer, share bar, and JS are identical across all articles):

1. **`<title>` tag**: `[Article Title] — Onsite Ops Quarterly`
2. **Hero CSS `background: url(...)`**: point to the compressed hero image
3. **`.article-category`**: the section display name (e.g., "Stadiums & Arenas")
4. **`.article-title`**: the headline
5. **`.article-deck`**: the subtitle
6. **`.article-meta`**: author, date (with datetime attribute), read time
7. **`.article-body`**: all `<p>` tags with article content
   - First paragraph gets class `article-lead` (triggers the gold drop cap)
   - Last element is the source attribution link with this format:
     ```html
     <p style="margin-top: 48px; padding-top: 24px; border-top: 1px solid var(--border-light); font-size: 13px; color: var(--text-tertiary); font-style: italic;">Based on reporting by [Outlet]: <a href="[URL]" target="_blank" rel="noopener" style="color: var(--accent); text-decoration: none;">[Link Text]</a></p>
     ```
8. **`.article-tags`**: 5-6 tags as `<a class="article-tag">` elements
9. **Related articles grid**: 3 cards linking to other real articles in the same section
   - Use `ls articles/[parent]/[section]/*.html` to find existing articles
   - Read their titles and excerpts from the files
   - Link using relative filenames (same directory)

### 6. Move source markdown to repo (if applicable)

If the user provided a `.md` file from Downloads:
```bash
mv ~/Downloads/[file].md articles/[parent]/[section]/
```

### 7. Commit and push

```bash
git add articles/[parent]/[section]/[new-slug].html images/articles/[section]/ [any .md files]
git commit -m "Add [title] article to [section]

[Brief description]. [X] min read with source attribution.

Co-Authored-By: Claude Opus 4.6 (1M context) <noreply@anthropic.com>"
git push origin main
```

## Article body format

The OOQ 1-minute read format:
- **No section headers** (no `<h2>` with numbered spans) — save those for longer-form pieces
- **5-6 short paragraphs** of concise, opinionated prose
- **First paragraph** uses `class="article-lead"` for the gold drop cap
- **Source attribution** at the bottom with styled link
- **No pull quotes or callout boxes** for 1-2 min reads (use them for 4+ min pieces)

## Directory structure reference

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

All articles are 3 levels deep from root. Internal links use `../../../`:
- `../../../index.html` → homepage
- `../../../about.html` → about page
- `../../../images/articles/[section]/` → hero images
- Relative filenames for cross-links within the same section (e.g., `other-article.html`)

## Example invocation

User says:
> /publish-article @~/Downloads/new-stadium-article.md here's a new article for stadiums and arenas, hero image is in Downloads as stadium_photo.jpg

The skill then:
1. Reads the markdown
2. Compresses `stadium_photo.jpg` → `images/articles/stadiums-arenas/[slug]-hero.jpg`
3. Creates `articles/venue-types/stadiums-arenas/[slug].html`
4. Moves the `.md` to the same directory
5. Commits and pushes
