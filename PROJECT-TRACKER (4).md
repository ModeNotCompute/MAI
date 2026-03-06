# Domain Intelligence Report Tool — Project Tracker
**Project Owner:** Matt (Technology Business Partner, HUB International)  
**Purpose:** Automated cybersecurity assessment tool for insurance agency acquisition due diligence  
**Last Updated:** February 13, 2026  
**Current Version:** v7.1

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Architecture & Technical Stack](#architecture--technical-stack)
3. [Version History / Changelog](#version-history--changelog)
4. [Current Feature Set](#current-feature-set)
5. [Tools & Services Used](#tools--services-used)
6. [Known Issues & Limitations](#known-issues--limitations)
7. [Roadmap / To-Do](#roadmap--to-do)
8. [Completed Items](#completed-items)
9. [Context for Regeneration](#context-for-regeneration)

---

## Project Overview

A browser-based, single-file HTML/JavaScript application that performs automated domain intelligence scanning for insurance agency websites. The tool evaluates DNS configuration, email security, SSL certificates, website security, payment gateways, custom applications, and office locations — then synthesizes findings into a CMMI-aligned IT Maturity Rating.

**Primary Use Case:** Pre-acquisition technology due diligence for HUB International. Matt uses this tool to rapidly assess acquisition targets' technology environments, create migration budgets, assess risk profiles, and prepare for post-close integration (domain transfers to CSC, DNS zone file exports, etc.).

**Key Design Principle:** External scanning can reliably assess IT maturity up to approximately CMMI Level 3. Higher levels require deeper agency engagement. The tool provides objective, observable data points as conversation starters — not definitive assessments.

---

## Architecture & Technical Stack

### Frontend (Single HTML File)
- **Languages:** HTML, CSS, JavaScript (vanilla — no frameworks)
- **Fonts:** JetBrains Mono, Space Grotesk (Google Fonts)
- **Design:** Dark theme, grid background, collapsible sections, responsive

### Backend Proxy
- **Cloudflare Worker** (deployed): `https://muddy-disk-a0b7.nhsummits.workers.dev`
- **Purpose:** Eliminates CORS restrictions when fetching target website HTML
- **Free tier:** 100,000 requests/day
- **Fallback proxies:** allorigins.win, everyorigin.jwvbremen.nl (used if Worker is unavailable)

### External APIs (called directly from browser — no CORS issues)
| API | Purpose | Endpoint |
|-----|---------|----------|
| Cloudflare DNS-over-HTTPS | DNS record lookups (A, MX, NS, TXT, SOA, CNAME) | `cloudflare-dns.com/dns-query` |
| RDAP (Verisign / rdap.org) | WHOIS/registrar data | `rdap.verisign.com` / `rdap.org` |
| crt.sh | Certificate Transparency logs for SSL data | `crt.sh/?q=domain&output=json` |
| ipapi.co | VPN/proxy detection for SSL disclaimer | `ipapi.co/json/` |

### APIs requiring CORS proxy (fetched via Cloudflare Worker)
| Data | Why proxy needed |
|------|-----------------|
| Website HTML | CORS policies on target sites block direct browser fetch |
| Sub-page HTML (locations) | Same — needed for office location extraction |

---

## Version History / Changelog

### v7.1 (Current) — February 17, 2026
- PDF export text changed from various grays (#1a1a2e, #555, #666, #888) to pure black #000000
- All body text, data labels, data values, SRI items, location table text now render as true black in PDF
- PDF header timestamp uses #333 for subtle hierarchy
- Web app dark theme completely unchanged — only affects the injected print stylesheet
- Updated PDF header tool version to v7.1

### v7.0 — February 17, 2026
- **Major: CMMI V3.0 alignment** — maturity scale corrected from 1-5 to official 0-5 scale
- Level 0 (Incomplete): <35% — "Ad hoc and unknown"
- Level 1 (Initial): 35-54% — "Unpredictable and reactive"
- Level 2 (Managed): 55-74% — "Planned and controlled" (was "Repeatable")
- Level 3 (Defined): ≥75% — "Proactive, organization-wide" (was 82%)
- Level 4 (Quantitatively Managed): Cannot assess externally (was "Managed/Optimized")
- Level 5 (Optimizing): Cannot assess externally (was "Transformative/Strategic")
- Gauge updated to 6 segments (0-5) with per-level colors
- Ceiling note updated with correct CMMI V3.0 level names
- **Major: Quick Reference Guide** — standalone HTML document explaining scoring methodology
- Linked from maturity section disclaimer
- Covers all 6 levels with IT-context descriptions and typical external indicators
- Includes scoring categories table, threshold visualization, inconclusive/bonus-only explanations
- Cites official CMMI sources (ISACA, CMU, cmmiinstitute.com)
- Professional light-theme design, print-friendly

### v6.0 — February 17, 2026
- **Major: CMMI citation and reference link added** to maturity section disclaimer
- Credits CMMI Institute (subsidiary of ISACA), developed at Carnegie Mellon University
- Links to official CMMI levels page: https://cmmiinstitute.com/learning/appraisals/levels
- Notes that our five levels are adapted from the official CMMI staged representation for IT infrastructure context
- **Major: Excel (.xlsx) export added** — "Export as Excel" button alongside PDF export
- Sheet 1 "Report": All scan data in Category / Data Point / Value columns
- Sheet 2 "Maturity Scoring": Full breakdown with Category, Indicator, Points, Max, Status, Notes
- Sheet 3 "Migration": Post-acquisition contacts, registrar, nameservers, transfer lock status
- Sheet 4 "Locations": Office location table (only if generated before export)
- Uses SheetJS library (xlsx.full.min.js) loaded from cdnjs.cloudflare.com

### v5.11 — February 13, 2026
- Fixed PDF export of office locations table — was rendering as blank/black
- Added comprehensive print stylesheet overrides for location table: white backgrounds, dark text, teal headers
- Catches inline `var(--surface)`, `var(--border)`, `var(--text-dim)` styles on dynamically generated table elements
- Alternating row backgrounds, italic address placeholders, and container divs all properly themed for light PDF

### v5.10 — February 13, 2026
- PDF export now uses **clean light theme** (white background, black text, professional color accents)
- Injects temporary print stylesheet that overrides dark theme for PDF only — web app unchanged
- Tags use light pastel backgrounds with dark text (green on light green, red on light red, etc.)
- Section headers get light gray background, all text goes dark
- **Office locations now included in PDF** if generated before export
- Locations button hidden if no data generated; location table included if data exists
- PDF header uses dark teal accent on white with domain name and generation timestamp
- Cleanup function removes all temporary styles after render — no visual artifacts

### v5.9 — February 13, 2026
- Added PDF export button at bottom of report
- Added project tracking document

### v5.8 — February 13, 2026
- Reverted section name to "WHOIS / REGISTRAR" (removed "/ HOST")
- Added **Migration Contacts** summary to WHOIS section with actionable post-close guidance
- Detects if registrar and DNS host are the same entity
- Shows specific login URLs for common DNS providers
- Flags transfer lock status in migration context
- Calls out CSC as target registrar

### v5.7 — February 13, 2026
- Added Website Host to WHOIS section (later reverted in v5.8)
- Added WordPress.com, Wix, AWS Global Accelerator to hosting detection

### v5.6 — February 13, 2026
- SSL Certificate status changed to "Likely Valid" / "Likely Invalid" with verification prompt
- Fixed Website Security issue count (broken template literal)
- Payment Gateways header now distinguishes "X Gateways, Y Links"
- Renamed "Custom Applications" to "Custom or 3rd Party Applications"

### v5.5 — February 13, 2026
- **Major: Deployed Cloudflare Worker** as primary CORS proxy
- Worker URL: `https://muddy-disk-a0b7.nhsummits.workers.dev`
- `WORKER_PROXY` constant at top of script for easy URL changes
- Third-party proxies demoted to fallback-only
- Both `fetchPageHtml` and location scanner `fetchPage` use Worker first

### v5.4 — February 13, 2026
- Added scan duration timer at bottom of report (seconds + milliseconds + timestamp)
- **DKIM detection overhaul:** Now checks 19 common selectors across all major providers
- Queries both TXT and CNAME records (fixes Microsoft 365 DKIM detection)
- M365-specific DKIM guidance when not found (manual check instructions)
- DKIM marked as inconclusive (not penalized) for M365 domains in maturity scoring
- Multi-proxy resilience: allorigins.win, everyorigin.jwvbremen.nl, corsfix.com fallback chain

### v5.3 — February 12, 2026
- Fixed `fetchPageHtml` CORS proxy — reverted to simpler approach matching v4's working pattern
- allorigins `/get` JSON endpoint as primary proxy method

### v5.2 — February 12, 2026
- Attempted robust multi-proxy `fetchPageHtml` with 5 fallbacks + content validation
- (Had issues — over-validation was rejecting valid responses)

### v5.1 — February 12, 2026
- Payment Gateways scoring changed to **bonus-only** (no penalty for absence)
- Custom Applications scoring changed to **bonus-only** (no penalty for absence)
- Added **Manual SSL Certificate Entry** form (issuer, expiry date, SAN count)
- Manual SSL data feeds into maturity scoring, takes priority over CT log data
- SSL marked as inconclusive (not penalized) when no CT data and no manual entry

### v5.0 — February 12, 2026
- **Major: Estimated IT Maturity Rating** module added
- CMMI-aligned 5-level scale (Initial → Transformative)
- Visual gauge with active level highlighted, Levels 4-5 shown as locked
- 7 scoring categories, ~20 indicators
- Per-category breakdowns with progress bars
- Level 3 ceiling note explaining external scan limitations
- Disclaimer about conversation-starter framing

### v4.2 — February 12, 2026
- WHOIS Transfer Lock now recognizes broader lock statuses (7 lock indicators)
- Shows which specific protections are active
- Phone number format changed to `x-yyy-zzz-aaaa` (country code-area-exchange-number)
- Phone regex updated to capture leading country codes (+1, 1-)

### v4.1 — February 12, 2026
- **Location scanner completely rewritten** with 5-strategy approach
- Nav link parsing for "City, ST" patterns
- Flexible address regex (handles abbreviations, full state names)
- Sub-page crawling (city pages + common contact/about paths)
- Partial entry fallback for cities without full addresses
- Phone/fax/toll-free extraction and classification

### v4.0 — February 12, 2026
- **Major: Added Payment Gateways module** — scans for 20 payment platforms + payment keywords
- **Major: Added Custom Applications module** — detects 19 file extensions + 19 path patterns
- **Major: Added Location Generator** button — crawls site for office addresses
- HSTS replaced with manual check button (securityheaders.com with domain pre-filled)
- VPN/proxy detection added to SSL section (ipapi.co)
- Transfer Lock indicator added to WHOIS section
- Location table output with columns: Location Name, Address 1, Address 2, City, State, Zip, Phone, Fax, Toll Free

### v3.0 — February 11, 2026
- SSL Shopper iframe embed (later removed — didn't render)
- Dual SSL approach: crt.sh API + SSL Shopper link

### v2.0 — February 11, 2026
- Added crt.sh Certificate Transparency API for SSL details
- SAN domain extraction with clickable "Scan Domain" buttons for sub-reports
- Enhanced HSTS checking with triple-fallback

### v1.0 — February 11, 2026
- Initial build: WHOIS (RDAP), DNS (Cloudflare DoH), Email Security (SPF/DMARC/DKIM), basic SSL, HTTP/Infrastructure
- Single-file HTML/JavaScript application
- Dark theme design with collapsible sections

---

## Current Feature Set

### Scan Modules (in order of appearance)
1. **WHOIS / Registrar** — Registrar, IANA ID, registration/expiry dates, transfer lock status, nameservers, migration contacts
2. **DNS Records** — A, MX, NS, TXT, SOA, CNAME records
3. **Email Security** — SPF (strictness), DMARC (policy level), DKIM (19 selectors, TXT+CNAME), email provider detection, security gateway detection, auth score
4. **SSL / Certificate** — Certificate status (likely valid/invalid), issuer, expiry countdown, SAN domains with sub-scan buttons, manual entry form, SSL Shopper link, VPN/proxy detection
5. **HTTP / Infrastructure** — HTTP→HTTPS redirect, HSTS (manual check button), IP address, hosting/CDN detection, DNS host detection
6. **Website Security** — Subresource Integrity (SRI) audit for external resources
7. **Payment Gateways** — 20 payment platform detection + payment keyword URL scanning
8. **Custom or 3rd Party Applications** — 19 server-side extensions + 19 app path patterns
9. **Estimated IT Maturity Rating** — CMMI-aligned Level 1-3 assessment with category breakdowns
10. **Office Location Generator** (button) — Multi-strategy address extraction with table output
11. **PDF Export** (button) — Full report output as PDF

### Supporting Features
- Scan duration timer
- VPN/proxy detection with SSL impact warning
- Manual SSL certificate entry form
- SAN domain sub-reports (full scan of related domains)
- Migration contacts with registrar/DNS provider identification

---

## Tools & Services Used

### Development
| Tool | Purpose |
|------|---------|
| Claude (Anthropic) | AI pair programming, code generation, architecture decisions |
| Browser DevTools | Testing, debugging CORS issues |

### Infrastructure
| Service | Purpose | Account |
|---------|---------|---------|
| Cloudflare Workers | CORS proxy (primary) | nhsummits.workers.dev |
| allorigins.win | CORS proxy (fallback) | No account needed |
| everyorigin.jwvbremen.nl | CORS proxy (fallback) | No account needed |

### External APIs (no accounts needed)
| API | Purpose |
|-----|---------|
| Cloudflare DNS-over-HTTPS | DNS record queries |
| RDAP (Verisign) | WHOIS data for .com/.net |
| RDAP (rdap.org) | WHOIS data for other TLDs |
| crt.sh | Certificate Transparency logs |
| ipapi.co | IP geolocation / VPN detection |
| SSL Shopper | Manual SSL certificate chain verification |
| securityheaders.com | Manual HSTS / security headers check |

### Manual Verification Tools (linked in reports)
- SSL Shopper: `sslshopper.com/ssl-checker.html#hostname=[domain]`
- Security Headers: `securityheaders.com/?q=[domain]&followRedirects=on`
- crt.sh: `crt.sh/?q=[domain]`

---

## Known Issues & Limitations

### CORS / Proxy
- Free third-party CORS proxies (allorigins, everyorigin) are unreliable — go down without notice, rate-limit aggressively
- **Mitigated:** Cloudflare Worker deployed as primary proxy (v5.5+)
- corsproxy.io **blocks HTML content** as of 2025 — only allows JSON/XML/CSV

### DKIM Detection
- Microsoft 365 DKIM uses CNAME chains that external DNS queries often cannot resolve
- **Mitigated:** Now checks both TXT and CNAME records, shows M365-specific guidance when not found (v5.4+)
- Custom/non-standard DKIM selectors beyond the 19 checked will not be detected

### SSL Certificates
- crt.sh data reflects the *real* certificate from CT logs, which may differ from what a user behind a VPN/proxy actually sees
- **Mitigated:** Manual SSL entry form allows user to input actual cert data (v5.1+), status shows "Likely Valid" (v5.6+)

### Website HTML Fetching
- Some sites use heavy JavaScript rendering (SPAs) — the fetched HTML may be a shell without content
- Squarespace sites generally work well (server-rendered)
- Sites behind WAFs (Cloudflare bot protection, etc.) may block proxy requests

### Location Scanner
- Address regex-based — can miss addresses in non-standard formats, images, or JavaScript-rendered content
- Phone/fax classification is heuristic-based
- Limited to US addresses currently

### IT Maturity Rating
- External scanning can only assess up to Level 3
- Scoring weights are calibrated for insurance agencies specifically
- Payment gateways and custom apps are bonus-only (no penalty for absence)

---

## Roadmap / To-Do

### High Priority
- [ ] Investigate hosting the app on a proper web server (not just local HTML file)
- [ ] Consider adding more DKIM selectors based on real-world findings
- [ ] Add ability to save/load scan results for comparison over time

### Medium Priority
- [ ] Add HSTS auto-detection through Cloudflare Worker (Worker can read response headers)
- [ ] Expand hosting provider detection (more IP ranges)
- [ ] Add email security gateway detection for more providers
- [ ] Consider adding DNS CAA record checking
- [ ] Add DNSSEC validation check

### Low Priority / Future
- [ ] Multi-domain batch scanning
- [ ] Side-by-side comparison of two domains
- [ ] Historical tracking (scan a domain over time, track changes)
- [ ] Export to Excel/CSV in addition to PDF
- [ ] Custom DKIM selector input (let user specify selectors to check)
- [ ] Webhook/API mode for integration with other tools

---

## Completed Items

- [x] Core scanning engine (WHOIS, DNS, Email, SSL, HTTP)
- [x] Email security scoring (SPF, DMARC, DKIM)
- [x] Certificate Transparency integration (crt.sh)
- [x] SAN domain sub-scanning
- [x] VPN/proxy detection
- [x] Transfer lock detection
- [x] HSTS manual check button
- [x] SRI (Subresource Integrity) audit
- [x] Payment gateway detection (20 platforms)
- [x] Custom/3rd party application detection (19 extensions + 19 patterns)
- [x] IT Maturity Rating (CMMI-aligned, Level 1-3)
- [x] Office location extraction (5-strategy approach)
- [x] Manual SSL certificate entry form
- [x] Cloudflare Worker deployment (CORS proxy)
- [x] Migration contacts in WHOIS section
- [x] Scan duration timer
- [x] DKIM CNAME detection (Microsoft 365)
- [x] 19-selector DKIM detection
- [x] Bonus-only scoring for payment gateways and custom apps
- [x] PDF export functionality
- [x] Phone number formatting (x-yyy-zzz-aaaa)
- [x] Project tracking document

---

## Context for Regeneration

### Key Files
| File | Description |
|------|-------------|
| `domain-report-v5.9.html` | Current production version of the scanning tool |
| `cloudflare-worker.js` | Cloudflare Worker proxy script (deployed) |
| `PROJECT-TRACKER.md` | This file |

### Cloudflare Worker Details
- **URL:** `https://muddy-disk-a0b7.nhsummits.workers.dev`
- **Format:** Accepts `?url=` parameter, returns JSON `{ contents: "...", status: { http_code, response_headers } }`
- **Same format as allorigins `/get` endpoint** for compatibility
- **Worker name:** muddy-disk-a0b7
- **Account subdomain:** nhsummits.workers.dev

### Design Decisions & Rationale
1. **Single HTML file** — No build step, no dependencies, runs anywhere. User opens file in browser.
2. **Cloudflare Worker over self-hosted proxy** — Free, globally distributed, no server to maintain.
3. **CMMI-aligned maturity scale** — Anchoring to an established industry framework enhances credibility.
4. **Bonus-only scoring for payments/apps** — Many mature agencies handle payments through carriers and use internal apps not linked publicly.
5. **"Likely Valid" SSL language** — Proxy interference makes definitive statements unreliable.
6. **19 DKIM selectors with TXT+CNAME** — Microsoft 365's CNAME chain approach was causing false negatives across all scans.
7. **Migration Contacts in WHOIS** — The TBP's primary post-close action item is domain/DNS transfer to CSC; having registrar and DNS host identified with login URLs saves research time.

### Scoring Methodology (IT Maturity)
- **Level 1 (Initial):** Score < 55%
- **Level 2 (Repeatable):** Score 55-81%
- **Level 3 (Defined):** Score >= 82%
- **Levels 4-5:** Cannot be assessed externally — requires agency engagement
- **Inconclusive items** (null points) are excluded from percentage calculation
- **Bonus-only categories** (payments, apps) can only help, never hurt the score

### User Preferences
- Phone format: `x-yyy-zzz-aaaa` (with country code)
- Zip codes starting with 0 get `'` prefix for Excel compatibility
- WHOIS lookups: User has granted permission to fetch `https://who.is/whois/[domain]`
- Version convention: Major change = next whole number, minor = +0.1
