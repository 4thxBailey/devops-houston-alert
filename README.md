# Houston Alert — DevOps Engineering Blog

> **Live site:** [devops.houstonalert.com](https://devops.houstonalert.com)  
> **Product:** [houstonalert.com](https://houstonalert.com)  
> **Organization:** [4th and Bailey](https://github.com/4thxBailey/)

The engineering story behind [houstonalert.com](https://houstonalert.com) — a real-time infrastructure monitoring platform covering 160 Houston metro ZIP codes. This repository contains the source code for the DevOps blog published at `devops.houstonalert.com`.

---

## Authors

| | Name | GitHub | Role |
|---|---|---|---|
| 🔵 | **Lionel Mosley** | [@trust-lionel](https://github.com/trust-lionel) | Backend · Data Pipelines · Architecture |
| 🔷 | **Nigel Brooks** | [@brookstrades-glitch](https://github.com/brookstrades-glitch) | Frontend · PWA · Mobile · UX |

Built at **[4th and Bailey LLC](https://github.com/4thxBailey/)** — Houston, TX.

---

## What This Repository Contains

```
devops-houston-alert/
├── index.html              # DevOps blog — single deployable file
├── sitemap.xml             # Submit to Google Search Console & Bing
├── robots.txt              # Crawler policy
├── netlify.toml            # Security headers, HTTPS redirect, cache rules
├── .well-known/
│   └── security.txt        # RFC 9116 security disclosure (renew annually)
└── README.md               # This file
```

---

## About Houston Alert

Houston Alert is a **free**, **real-time** infrastructure monitoring platform for the Houston metropolitan area. It aggregates six live data sources into a single unified map covering road closures, flash flood warnings, power outages, weather alerts, seismic activity, and community reports — refreshing every 60 seconds.

### The problem it solves

During major weather events, Houstonians had no single place to understand what was happening across the city. Government alert channels were siloed — TxDOT, NWS, county flood systems, and 311 were never talking to each other. Houston Alert stitches them together.

### Key numbers

| Metric | Value |
|---|---|
| ZIP codes monitored | 160 |
| Data refresh cycle | 60 seconds (30s during severe weather) |
| Live data feeds | 6+ |
| Monthly infrastructure cost | < $20 |
| Cost to users | $0 — free forever |

---

## Data Sources

| Source | Type | Refresh | Coverage |
|---|---|---|---|
| **Houston TranStar** | Road & traffic incidents | 60s | Freeway sensors · cameras · dispatch |
| **TxDOT** | Construction & lane closures | 60s | ArcGIS REST · I-10 · 290 · Beltway |
| **National Weather Service** | Weather alerts & warnings | 60s / 30s | Harris · Fort Bend · Galveston · Brazoria · Montgomery · Waller |
| **Harris County FWS · USGS NWIS** | Stream gauges · flood stages | 60s | Harris County bayous & creeks |
| **USGS Earthquake Catalog** | Seismic activity | 60s | Houston bounding box · induced seismicity |
| **Social Signal Layer** | Community reports | 5min | X API v2 · 8 incident categories · confidence scored |

---

## Architecture Overview

```
Browser
  └── PWA · houstonalert.com
        └── Interactive Map · Live Feed Panel · ZIP Filter

        ↕ CDN Edge → /api/* Proxy

Frontend (Static Delivery · Global CDN)

        ↕ HTTPS

Backend (Event Aggregation Server · Node / Express)
  ├── 60s Scheduler
  ├── In-Memory Event Store  (6hr TTL · 2,000 event cap)
  └── Dedup Engine           (FNV-1a hash · source-agnostic IDs · 300m radius)

        ↕ External APIs

Pollers
  ├── Road Conditions  (TranStar · ArcGIS)
  ├── NWS Weather      (NOAA REST)
  ├── Flood Gauges     (USGS NWIS)
  ├── Seismic          (FDSN)
  ├── TxDOT            (DriveTexas)
  └── Social Signals   (X API v2)
```

### Key architectural decisions

- **In-memory store over a database** — write latency matters more than persistence for a real-time feed. Events older than 6 hours are no longer actionable.
- **Scheduled polling over webhooks** — not all government APIs support webhooks; polling was the only reliable cross-source architecture.
- **Deterministic deduplication** — identical events from different sources produce the same FNV-1a hash, preventing duplicate map pins.
- **Confidence scoring** — official sources are verified; social signals are weighted by post volume, keyword specificity, and geographic precision and surfaced as a 0–100% confidence score.

---

## Progressive Web App

Houston Alert is deployed as a PWA — not a native app. For an emergency information tool, **zero installation friction is a core safety feature**. During a hurricane you don't wait for an app to download — you send a link and someone is looking at real-time data within seconds.

| PWA Capability | Status |
|---|---|
| Home screen install (no App Store) | ✅ |
| Push notifications | ✅ |
| Geolocation-aware map | ✅ |
| Static CDN delivery (< 1s load) | ✅ |
| Responsive — 320px to 27" | ✅ |
| Deep-linkable events | ✅ |
| iOS Safari compositor fix | ✅ Applied at 900px breakpoint |

---

## Design System

The blog uses the **Fluent 2 / Material 3 hybrid** token layer from `index_v16.html`, maintained by 4th and Bailey:

| Token | Value | Usage |
|---|---|---|
| `--md-primary` | `#0f6cbd` | Links · buttons · active states |
| `--md-secondary` | `#115ea3` | Secondary containers |
| `--md-error` | `#c50f1f` | Alerts · challenge blocks |
| `--md-warning` | `#da3b01` | Warnings · social signals |
| `--md-success` | `#107c10` | Resolved · live status |
| `--md-background` | `#f5f5f5` | Page background |
| `--md-surface-container-lowest` | `#ffffff` | Cards · detail pane |
| Body font | Segoe UI | Matches v16 system font |

---

## Deployment

This blog is deployed as a **static site** — a single `index.html` with no build step required. Deploy to any static host (Netlify, Vercel, GitHub Pages, Cloudflare Pages).

### Netlify (recommended)

```bash
# 1. Fork or clone this repository
git clone https://github.com/4thxBailey/devops-houston-alert.git

# 2. Connect to Netlify — point to repository root
# 3. Set custom domain: devops.houstonalert.com
# 4. Security headers are pre-configured in netlify.toml
```

The included `netlify.toml` provides:
- HTTPS-only redirect (HTTP → HTTPS, www → non-www)
- Security headers: `X-Frame-Options`, `X-Content-Type-Options`, HSTS (2-year), full CSP
- Cache rules: 1-year immutable TTL for assets, `must-revalidate` for HTML

### Security headers

Verify deployment at [securityheaders.com](https://securityheaders.com) — target grade **A+**.

### Before deploying — replace placeholder IDs

Open `index.html` and replace:

| Placeholder | Replace with | Where to get it |
|---|---|---|
| `GTM-XXXXXXX` | Your GTM Container ID | [tagmanager.google.com](https://tagmanager.google.com) |
| `G-XXXXXXXXXX` | Your GA4 Measurement ID | Google Analytics 4 |
| `XXXXXXXXXX` (Clarity) | Your Clarity Project ID | [clarity.microsoft.com](https://clarity.microsoft.com) |

---

## SEO & Structured Data

| Item | Status |
|---|---|
| JSON-LD — `TechArticle` | ✅ Rich result eligible |
| JSON-LD — `BreadcrumbList` | ✅ SERP path display |
| JSON-LD — `Organization` | ✅ Brand entity |
| Open Graph — full article suite | ✅ |
| Twitter / X card | ✅ `summary_large_image` |
| Canonical URL | ✅ `https://devops.houstonalert.com/` |
| `sitemap.xml` | ✅ Submit to Search Console & Bing |
| `robots.txt` | ✅ |
| `security.txt` | ✅ `.well-known/` · RFC 9116 · renew annually |

**Validate structured data:** [validator.schema.org](https://validator.schema.org/)  
**Test Open Graph:** [developers.facebook.com/tools/debug](https://developers.facebook.com/tools/debug)

---

## Analytics Events (GA4 via GTM)

All events push to `window.dataLayer`. GTM reads them and fires GA4 events.

| Event | Trigger |
|---|---|
| `article_view` | Page load — includes `article_title`, `article_author`, `article_date`, `brand`, `github_repo` |
| `scroll_depth` | Fires at 25% / 50% / 75% / 90% / 100% — includes `depth_pct` |
| `section_view` | Each `section[id]` enters viewport at 30% threshold — includes `section_id` |
| `outbound_click` | Any link leaving `devops.houstonalert.com` — includes `link_url`, `link_text` |
| `time_on_page` | Fires at 30s · 60s · 3min · 5min milestones — includes `seconds` |

---

## Roadmap

- [ ] **Community Reporting** — let verified Houston residents submit events with automatic confidence scoring
- [ ] **ZIP Code Subscriptions** — persistent push notifications by neighborhood, commute route, or school zone
- [ ] **Historical Pattern Analysis** — flood-likelihood models for specific bayous based on continuous rainfall data

---

## License & Attribution

© 2026 **4th and Bailey LLC**. All rights reserved.

Houston Alert is a **free public safety resource** for the Houston metropolitan area. The platform will remain free to access for every resident, indefinitely — no paywalls, no subscriptions, no advertising.

Data is sourced from public government APIs (TranStar, TxDOT, NWS/NOAA, USGS, HCFWS) and the X API v2. Each source retains its own terms of use.

---

> *4th and Bailey exists to build things that matter in the communities we're part of. Houston Alert isn't a side project. It's an obligation.*  
> — Lionel Mosley & Nigel Brooks
