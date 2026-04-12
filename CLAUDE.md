# Onsite Ops Quarterly — Project Status

## Repo
- **GitHub:** https://github.com/jmbradley/OOQ
- **GitHub account:** jmbradley (not FlexTram — run `gh auth switch --user jmbradley` if needed)
- **Branch:** main
- **Local dev server:** `python3 -m http.server 8090` (configured in `.claude/launch.json`)

## What This Is
Onsite Ops Quarterly (OOQ) is an independent, anonymous publication covering live event onsite operations — venues, crowd management, logistics, safety, tech, and careers. The site is static HTML with no build step.

## What's Been Built

### Pages
- **index.html** — Landing page: hero with concert crowd background image, manifesto, coverage area grid, anonymous contribution banner, featured content sections, Buttondown subscribe form, footer
- **about.html** — Minimal about page: "By Onsite Operations Professionals, For Onsite Operations Professionals. IYKYK."
- **companies.html** — "Companies You Should Know" directory with 15+ categories, all companies linked to their websites. Categories include Venue Management, Crowd Management, Logistics, Staging, Safety & Medical, Credentialing, Ticketing, Tech, Parking, Production, Merch, Experiential, Trade Shows, Associations, Publications, Podcasts
- **careers.html** — Career Resources page with live event job boards, major employer career pages, broader entertainment job sites

### Features
- **Subscribe form** wired to Buttondown (newsletter: "The OOQ Advance", username: MemeticLeads). Uses public embed endpoint — no API key exposed in frontend
- **Mobile hamburger menu** — fullscreen dark overlay, X close animation, body scroll lock, closes on link click
- **Skip-to-content link** for accessibility (visible on Tab focus)
- **Responsive design** — mobile breakpoint at 768px, tablet at 1024px
- **Mobile tap targets** — 52px+ input height, 16px font (prevents iOS zoom), full-width button
- **Cross-page navigation** — nav logo links home, About/Companies/Careers linked from coverage grid and nav
- **Hero image** — concert crowd photo, compressed from 3.5MB to 712KB (1920px wide, quality 60), dark overlay for text readability
- **Nav transitions** — transparent on hero, solid on scroll; dark when mobile menu open

### Key Files
- `.env` — Buttondown API key (gitignored, never committed)
- `.gitignore` — excludes .env and .DS_Store
- `OOQ_Company_Directory.md` — source data for companies and careers pages (newer version)
- `OOQ_Company_Directory (1).md` — older version of directory
- `OOQ_ToDos.md` — Google Alerts setup checklist

## What Still Needs To Be Done

### High Priority
- **Domain connection** — user has a domain to connect (hosting TBD — GitHub Pages, Netlify, Cloudflare Pages, etc.)
- **Meta tags & Open Graph** — no meta descriptions, OG tags, or Twitter cards on any page yet. Critical for SEO and social sharing
- **Google Analytics** — analytics from another project showed organic search users have 4x engagement; SEO investment will pay off here

### Medium Priority
- **Blog/article system** — the site needs a way to publish articles (War Stories, The Debrief, The Walk-Through, Anonymous Op-Eds). Could stay static HTML or move to a static site generator like Astro
- **Trade Shows & Conventions** link from homepage coverage grid — the item in the Industry section isn't linked to the companies page trade shows section yet
- **Footer social links** — LinkedIn and Instagram still point to `#`
- **Shared CSS** — styles are duplicated across all 4 HTML files. Could extract to a shared stylesheet

### Lower Priority
- **Nav active state** — only the about page highlights the current nav item
- **404 page**
- **Favicon**
- **Loading performance** — consider lazy loading the hero image, preloading fonts
- **Form validation UX** — email format validation before submit, better error states
- **Contribution/submission form** — the "Submit a Story" and "Tip Line" links point to #contribute but there's no actual submission mechanism yet
