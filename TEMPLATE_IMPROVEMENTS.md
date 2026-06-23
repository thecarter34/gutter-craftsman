# Premium Template — Improvement Log

Template-level issues found during client builds are appended here. Client-specific issues (content, colors, images) stay in the project dir — only template bugs go here.

---

## [2026-06-16] Issue: Brand colors not extracted from existing site before build

- **Found during:** Yakesh Roofing + CVEC first-pass builds
- **Problem:** Build subagent used the template's default blue/navy palette instead of extracting the client's existing brand colors. Both Yakesh (crimson red + brown) and CVEC (dark charcoal + bright red) got completely different mockup themes.
- **Fix applied:** Added `### Brand Identity Extraction` step to both `jjctech-outreach` and `local-web-design` skills — must extract existing site colors via Playwright CSS inspection BEFORE generating any HTML. Brand colors override template defaults.

## [2026-06-16] Issue: Logo not downloaded and self-hosted

- **Found during:** Yakesh Roofing rebuild — navbar had no logo, site pointed to old domain
- **Problem:** Skill only screenshot the logo element for reference but never downloaded the actual image file. Rebuilt sites had no logo or were still referencing the old site's domain in `src` attributes.
- **Fix applied:** Added `Step 1b — Download actual logo file (MANDATORY)` to `local-web-design` SKILL.md:
  - `get_logo_url()`: checks logo img tags, navbar img, favicon link
  - `download_logo()`: downloads to `project/assets/logo.png` preserving original format (png/jpg/svg/webp)
  - Fallback to screenshot if download fails (saves as `logo-reference.png`)
  - HTML wiring: `<img src="assets/logo.png" alt="Business Name Logo">` in navbar
  - No-logo fallback: text-based wordmark `<div class="wordmark">Business Name</div>` styled with brand colors

## [2026-06-16] Issue: Site photos not downloaded before build

- **Problem:** Skill only warned about external CDN images in a pitfall note but provided no actual extraction step. Build subagents had no code to find, download, and rename externally-hosted images before building.
- **Fix applied:** Added `Step 1c — Download existing site photos (MANDATORY)` to `local-web-design` SKILL.md:
  - `scrape_external_images()`: uses Playwright to find all `<img>` tags, filters out same-domain and data URIs, downloads the rest to `project/assets/`
  - Priority ordering: hero → about photos → service images → trust badges
  - Descriptive rename step: `hero.jpg`, `about-owner.jpg`, `service-roofing.jpg` (not `external-img-1.jpg`)
  - Reinforces the external CDN pitfall with actionable code instead of just a warning

## [2026-06-16] Issue: .reveal scroll-reveal + data-count stat counters break in screenshots / no-JS

- **Found during:** Yakesh Roofing post-build review (June 16, 2026)
- **Problem:** The template uses `IntersectionObserver` to:
  1. Toggle `.reveal` class from `opacity: 0` to `opacity: 1` when scrolled into view
  2. Animate `data-count` stat counters from literal "0" to their target value

  When a Playwright screenshot is taken with bare `page.screenshot()` (no scroll), the observers never fire, so:
  - All `.reveal` sections (services, reviews, etc.) render as **giant black voids**
  - All stat counters render as **"0 YEARS IN BUSINESS / 0 PROJECTS COMPLETED / 0 GOOGLE REVIEWS / 0 YEARS EXPERIENCE"**
  - The vision critique then grades the design as catastrophic — empty page, lying about credentials — when actually the page would look fine to a real user.

  This is a triple failure: (a) the screenshot captures an invalid state, (b) the critique interprets that state as the design's fault, (c) the page would also fail for users on flaky connections where JS never runs or animations don't fire.

- **Real-world example of failure:** Yakesh Roofing final build. The HTML had correct service cards, real stat targets (9+, 100+, 5★, 20+), and review cards. The screenshot showed empty sections with "0 / 0 / 0 / 0" stat counters. The vision critique correctly identified "all zeros" and "giant black voids" as blockers — but the actual fix was a JS safety net, not removing the sections or rewriting the design.

- **Fix applied to template (`/workspace/premium-template/index.html`):**
  1. **CSS no-JS safety net for `.reveal`:** `@keyframes reveal-safety` forces `opacity: 1` after 1.5s regardless of observer state. `.no-js .reveal` covers the noscript case.
  2. **JS counter safety net:** A 1.5s `setTimeout` that finds any `[data-count]` element still at literal `"0"` and sets it to its `data-count` target. Prevents the "0 years in business" embarrassment.
  3. **Skill update:** Both `local-web-design` and `jjctech-outreach` now have:
     - **Content Presence Gate** (binary PASS/FAIL) that runs before any vision critique. Counts service cards, review cards, stat items, visible images, form fields, tel: links, and checks for placeholder phone numbers. FAIL = stop, fix the build, re-render.
     - **`screenshot_with_observer_fire()` helper** that scrolls the full page before screenshotting so all IntersectionObservers fire — the only correct way to capture a site that uses scroll-reveal animations.
     - **Revised vision critique prompt** that starts with 🔴 BLOCKERS instead of ✅ "3-4 things working well." The forced-positive bucket is removed. Padded positives made the agent grade empty-void screenshots as "typography is good, just polish the spacing."

- **Verification:** Yakesh Roofing rebuild re-screenshotted with the new helper. All `.reveal` elements now visible, all stat counters show their target values (9+, 100+, 5★, 20+), no empty voids in the rendered state.

---

