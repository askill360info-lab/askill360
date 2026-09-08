# ASkill360 — Multi-Course Academy Platform

A static, SEO-optimized site built to hold **multiple courses**, not just one. Adding a new course later (AWS Data Engineering, Forward Deployed Engineer, or anything else) means adding one entry to a Python list and re-running one script — it does not mean rebuilding the site.

## What's new in this version

- **Self-contained pages** — CSS and the logo/badge images are now embedded directly inside every `.html` file (inlined `<style>`, base64 image data). Each page works correctly even if opened completely on its own, with no `assets/` folder alongside it — this fixes the "unstyled page with broken images" issue that happens if a file gets separated from its folder (e.g. dragging just the `.html` file somewhere, or an upload tool that only grabs one file). Fonts are still loaded from `assets/fonts/` as separate files; if those ever fail to resolve, the browser just falls back to a system font — layout and styling are unaffected either way.
- **Contact info wired in** — phone `+91 97897 46505` and email `info@askill360.com` now appear in the footer (every page), the enroll page, and the "prefer to call?" line under both lead forms.
- **Tested across screen sizes** — 320px (smallest common phone), 375px, 414px, 768px, 1024px, 1280px, 1440px, 1920px. Fixed a real overlap where the floating "Talk to a mentor" button covered the hero's primary CTA button on short mobile viewports.
- **Multi-course architecture** — `source/build_site.py` has a `COURSES` list. Each entry is a course (open or "coming soon"). The whole site — nav, homepage course grid, courses catalog, sitemap-worthy pages — is generated from that list.
- **Academy-level homepage** — leads with a data-engineering career hook (₹30+ LPA framing — edit the number/copy in `page_home()` if you want different framing), what the academy offers (mentors from product companies, mock interviews, certifications, assessments), then a course grid.
- **Popup lead form** — appears on every page ~1.2s after load. Name / Phone / Email / Course dropdown (pre-selected to the current page's course where relevant).
- **Sidebar lead form** — a persistent card that follows page content. It's positioned by JavaScript, not a fixed CSS offset, so it never overlaps your text: it measures the real gap beside the centered content and only shows itself if there's honestly room (≈1700px+ browser width). Below that, a floating "Talk to a mentor" pill button in the bottom-right opens the same form in the popup instead.
- **How submissions work right now** — both forms POST straight to [FormSubmit.co](https://formsubmit.co) (free, no signup) into a hidden invisible iframe, so it delivers to `info@askill360.com` without ever opening an app, navigating away, or triggering a popup blocker. This is deliberately a real browser form submission rather than a `fetch()`/AJAX call — AJAX to third-party domains is what ad-blockers and strict CORS policies most often block; a plain form POST is far more reliable. The visitor sees a green checkmark and "Thanks for contacting us!" in place of the form about 0.7s after submitting. **Required fields are validated before anything sends** — name, a valid phone number, and a valid email are all mandatory, with a plain-English message under the button if something's missing or malformed (not just the browser's small native tooltip). Pressing Enter in any field submits the form exactly like clicking the button. **One-time step required:** the very first real submission triggers an activation email from FormSubmit to `info@askill360.com` — open it and click the confirm link once, and every submission after that delivers automatically.

## Pages

| File                            | Purpose |
|----------------------------------|---------|
| `index.html`                     | Academy home — career hook, what-you-get, course grid |
| `courses.html`                   | Full course catalog (open + coming soon) |
| `azure-data-engineer.html`       | Full course page: curriculum accordion, outcomes, assessments |
| `aws-data-engineer.html`         | Coming-soon course page (waitlist CTA) |
| `forward-deployed-engineer.html` | Coming-soon course page (waitlist CTA) |
| `about.html`                     | Academy story, "Learn / Grow / Lead", who trains here |
| `faq.html`                       | FAQ (with FAQPage structured data) |
| `enroll.html`                    | Course picker + 5-step enroll process + contact |

Every page shares the same header (active-page highlighted), breadcrumb trail, footer, popup form and sidebar form.

## Adding a new course

Open `source/build_site.py`, find the `COURSES` list near the top, and add an entry:

```python
{
    "id": "your-course-id",
    "filename": "your-course.html",
    "name": "Full Course Name",
    "short": "Short Name",
    "status": "soon",              # or "open" once you have modules
    "tagline": "One-line pitch.",
    "hours": None, "sessions": None, "modules": [],
    "icon": "aws",                  # reuses an existing icon, or add a new one in course_icon_svg()
},
```

Run `python3 source/build_site.py` — a new coming-soon page, nav entry (via the courses grid), and sitemap-worthy page exist immediately. When the course is ready, give it a real `modules` list (same shape as `source/data.py`'s `MODULES`) and flip `status` to `"open"` — it automatically gets the full curriculum-accordion template that the Azure course uses.

## Before you go live

1. **Domain** — search `askill360.com` across every `.html` file, `robots.txt` and `sitemap.xml`; replace with your real domain.
2. **Contact email/phone** — `CONTACT_EMAIL`, `CONTACT_PHONE` and `CONTACT_PHONE_TEL` at the top of `source/build_site.py`. Change once, regenerate, updates everywhere (footer, enroll page, both forms).
3. **The ₹30+ LPA career-hook stat** — this is framed as *indicative* compensation with a caption disclaimer. Replace with a number and source you're comfortable standing behind, or remove the figure and keep the surrounding copy.
4. **Activate the form** — submit the popup or sidebar form once yourself after going live. Check `info@askill360.com` for a one-time "confirm your FormSubmit form" email and click it. Submissions before that point still show the success screen (the browser can't see FormSubmit's response through the hidden iframe to know otherwise) but won't actually be delivered yet — so do this step first, before sharing the site.
5. **Want a lead database instead of just email?** — swap FormSubmit for Formspree, Getform, or your CRM's form endpoint: change `FORM_ENDPOINT` near the top of `source/build_site.py`, then regenerate with `python3 source/build_site.py`.

## Hosting

Same as before — any static host works, no build step, no server.

- **Netlify / Vercel**: drag this whole folder onto their dashboard, or connect a Git repo.
- **GitHub Pages**: push this folder to a repo, enable Pages on `main`.
- **Your own domain**: upload everything in this folder to your host's public root via FTP/file manager. Keep the folder structure — pages link to each other with relative paths.

## SEO included on every page

- Unique `<title>`, meta description, canonical tag
- Open Graph + Twitter Card tags
- Structured data: `Course` (home + each course page), `EducationalOrganization` (home), `BreadcrumbList` (every inner page), `FAQPage` (FAQ page)
- Semantic HTML, one `<h1>` per page, `<details>/<summary>` accordions (fully crawlable, no JS required to read content)
- `robots.txt` + `sitemap.xml` covering all 8 pages

## Regenerating after any content edit

```
cd source/
python3 build_site.py
```

Curriculum data lives in `source/data.py` (shared with the PDF generator, so both stay in sync). Everything else — homepage copy, FAQ answers, course registry — lives directly in `source/build_site.py`.
