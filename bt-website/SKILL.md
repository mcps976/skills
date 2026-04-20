---
name: bt-website
description: Load full context for the Blackberry & Thistle Baking Co. website. Auto-invoke on anything touching the B&T site — design, copy, brand, build process, CMS, Sveltia, deployment, Cloudflare Workers, DNS, email, Porkbun, Formspree, payments, BTCPay, social media, or the business itself (cakes, services, menus, Ruth Moloney, go-live tasks).
user-invocable: false
---

# Blackberry & Thistle Baking Co. — Website Context

Bespoke cake and sweet treat business run by **Ruth Moloney** and **Martin Swindells**, based in Perth, Western Australia. Est. 2025.

- **ABN:** 90 668 574 203
- **Location:** Perth WA — delivery across metro, Hills, Swan Valley, South West WA
- **Repo:** `git@github.com:mcps976/business-website-cms.git` (`~/Git/business-website-cms/`)

---

## Brand Identity

### Palette

| Token | Hex | Use |
|---|---|---|
| `--cream` | `#EDE8DC` | Backgrounds, light text on dark |
| `--plum` | `#2A1F35` | Primary text, headings, nav |
| `--near-black` | `#1A1520` | Body text, footer |
| `--sage` | `#7A8C5C` | Accents, botanical details |
| `--berry` | `#7B5E8A` | Links, highlights, CTA accents |

Background body uses `#F0EBE0` (slightly warmer than `--cream`). Disabled/muted elements use `rgba(42,31,53,0.35)`.

### Typography

- **Headings:** `'Cormorant Garamond', Georgia, serif` — weights 300, 400, 600 (italic variants used)
- **Body:** `'DM Sans', system-ui, sans-serif` — weights 300, 400, 500
- Google Fonts: loaded via `fonts.googleapis.com` in each page `<head>`

### Tone of Voice

Warm and professional. Ruth's culinary expertise is the hero — her skill, care, and craftsmanship carry the brand. **Never state credentials bluntly** ("trained pastry chef", "professional baker"). Instead: show through specificity — "Every cake, every tart, every bite that leaves our kitchen is made by her — by hand, with seasonal ingredients." Seasonal ingredients and botanical inspiration are recurring themes. Use second person ("we") for the business voice. Copy should feel personal and considered, not corporate or salesy.

---

## Business Details

### Services & Lead Times

| Service | Lead Time |
|---|---|
| Wedding Cakes | 8–12 weeks minimum |
| Celebration / Birthday Cakes | 4–6 weeks minimum |
| Dessert Tables & High Teas | 2–3 weeks minimum |

Response time: within two business days.

### Cake Flavours (15)

Classic Madagascan Vanilla, Death by Chocolate, Espresso Martini, Zesty Lemon & Elderflower, Prosecco / White Chocolate & Raspberry, Banoffee Pie, Salted Caramel Apple Pie, Biscoff, Cookies and Cream, Chocolate Orange, Coconut Mango & Lime, Cereal Milk Funfetti, Red Velvet, Earl Grey / Lemon & Lavender, Matcha Strawberry.

### Sweet Bites

Petit Fours & Macarons, Seasonal Tarts, Dessert & High Tea Selections. (Also on contact form: Cake Pops, Shortbread, Brownies & Blondies, Caramel Slice, Cupcakes.)

---

## Site Architecture

Static HTML site, no JS framework. Source in `~/Git/business-website-cms/`, built to `_site/`.

### Pages

| File | Type | Notes |
|---|---|---|
| `index.html` | Static copy | Hero layout — full-screen, nav overlay |
| `about.html` | Static copy | Ruth + Martin intro, brand story |
| `services.html` | Static copy | Weddings, Events, Hi-Teas, Markets |
| `faq.html` | Static copy | — |
| `contact.html` | Static copy | Formspree form + cake selector |
| `menus.html` | **Built** | Generated from `_data/cakes.json` + `_data/sweetbites.json` |
| `gallery.html` | **Built** | Generated from `_data/gallery.json` |

### Build System

```
npm run build   →   node build.js   →   _site/
```

`build.js` reads `_data/*.json`, injects into `_templates/menus.html` and `_templates/gallery.html` via `{{CAKE_ITEMS}}` / `{{SWEETBITES_ITEMS}}` / `{{GALLERY_ITEMS}}` token replacement. Static pages are `fs.copyFileSync`'d as-is. `css/`, `images/`, and `admin/` are `copyDir`'d.

**JSON format:** both bare arrays and `{ "items": [...] }` (Decap CMS format) are supported. All current data files use the `{ "items": [] }` wrapper.

**No hot reload / dev server** — edit source files, run `npm run build`, inspect `_site/` manually.

### Deployment

Cloudflare Workers Assets — `wrangler.toml`:
```toml
name = "business-website-cms"
compatibility_date = "2026-04-07"

[assets]
directory = "_site"
```

Deploy: `npx wrangler deploy` from repo root (after running build).

**Domains:**
- Primary: `blackberryandthistle.com.au`
- CMS: `edit.blackberryandthistle.com` (Sveltia CMS UI)

---

## Sveltia CMS

Git-based CMS editing `_data/*.json` files directly on GitHub. Ruth accesses at `edit.blackberryandthistle.com`.

**`admin/config.yml` key settings:**
```yaml
backend:
  name: github
  repo: mcps976/business-website-cms
  branch: main
  base_url: https://sveltia-cms-auth.martin-48e.workers.dev

site_url: https://edit.blackberryandthistle.com
media_folder: "images/uploads"
public_folder: "/images/uploads"
```

**Collections:** `cakes` → `_data/cakes.json`, `sweetbites` → `_data/sweetbites.json`, `gallery` → `_data/gallery.json`. All use `files:` (single-file collections with `items` list widget), not `folder:` collections.

### Cloudflare Workers OAuth

Sveltia CMS requires an OAuth proxy to authenticate against GitHub. This runs as a separate Cloudflare Worker:

- **Repo:** `~/Git/sveltia-cms-auth/`
- **Worker name:** `sveltia-cms-auth`
- **Live URL:** `https://sveltia-cms-auth.martin-48e.workers.dev`
- **Entry:** `src/index.js`
- **Config:** `wrangler.toml` (compatibility_date 2023-03-24)

The `base_url` in `admin/config.yml` must point to this worker. If CMS login breaks, check: (1) the worker is deployed and healthy, (2) the GitHub OAuth App callback URL matches the worker URL, (3) `GITHUB_CLIENT_ID` and `GITHUB_CLIENT_SECRET` are set as Worker secrets in Cloudflare dashboard.

**Sveltia quirks:**
- CMS edits commit directly to `main` — every save triggers a build deploy cycle
- Media uploaded via CMS lands in `images/uploads/` in the repo
- After a CMS save, the site must be manually re-deployed (`wrangler deploy`) unless a CI/CD hook is configured — **no auto-deploy is currently set up**
- Sveltia does not support Netlify Identity — GitHub backend only
- `netlify.toml` is present in the repo but Netlify is **not** the deployment target — it's a legacy artefact

---

## Contact Form — Formspree

- **Endpoint:** `https://formspree.io/f/xjgpaqlk`
- **Method:** POST with AJAX via `@formspree/html` CDN script
- **Fields:** first_name, last_name, email, phone, event_type, event_date, guest_count, venue, message, cake_flavour (hidden — populated by JS cake selector)
- **Success state:** inline `data-fs-success` div (no redirect)
- **Error fallback:** directs to `hello@blackberryandthistle.com.au`

Cake selector stores choices in `sessionStorage` (`bt_cakes`) and pre-populates from `?cake=` URL param (used by "Order this flavour" links on menus page). Cleared on form submit.

---

## Email — Porkbun

- **Customer-facing:** `hello@blackberryandthistle.com.au`
- **Owner (Ruth):** `ruth@blackberryandthistle.com.au`
- **Owner (Martin):** `martin@blackberryandthistle.com.au`

Hosted on Porkbun. MX/SPF/DKIM records managed in Cloudflare DNS.

---

## Social Media

| Platform | Handle / URL |
|---|---|
| Instagram | `@blackberryandthistle` — instagram.com/blackberryandthistle |
| Facebook | `blackberryandthistlebakingco` — facebook.com/blackberryandthistlebakingco |

Instagram is the primary channel for market dates and new flavour announcements.

---

## Payments (Planned)

BTCPay Server on Hetzner VPS — planned for order deposit processing, connected to the Alby Hub Lightning node on nodebox for instant settlement. Not yet live.

---

## Pending Go-Live Tasks

These items were outstanding at time of writing — verify current status before acting on them:

- [ ] **Auto-deploy on CMS save** — wire GitHub Actions or Cloudflare Pages CI so CMS commits trigger `npm run build && wrangler deploy` automatically
- [ ] **About page photo** — `about-image-placeholder` div needs real image of Ruth / product
- [ ] **Gallery photos** — `_data/gallery.json` items have no images yet (`image: ""`)
- [ ] **Menu photos** — all cake and sweetbite entries have `photo1: ""`, `photo2: ""`
- [ ] **BTCPay payment integration** — deposit workflow not yet built
- [ ] **Confirm Porkbun email routing** — ensure `hello@`, `ruth@`, `martin@`, `orders@`, `accounts@` all deliver correctly
- [ ] **`netlify.toml` cleanup** — legacy file, can be removed once Cloudflare deployment is confirmed stable
- [ ] **`rpcallowip` / DNS** — confirm Cloudflare DNS records for `edit.blackberryandthistle.com` CNAME pointing to the Sveltia CMS worker

---

## Git

- **Repo:** `~/Git/business-website-cms/`
- **Remotes:** `origin` (GitHub mcps976/business-website-cms), `truenas` (bare repo)
- **Push:** `gpush`
- **Conventional commits:** `feat:`, `fix:`, `content:`, `style:`, `chore:`
- **Note:** `_site/` should be in `.gitignore` if not already — it's the build output
