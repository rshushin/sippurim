# CLAUDE.md – Sippurim site

## Project Overview
Static site at **sippurim.com** serving two distinct purposes from one domain:

1. **AI Solutions brand landing** – the homepage (`index.html`) positions Sippurim as an umbrella brand building AI-native products. Flagship is **SupplySignal** (supplysignal.app, a separate codebase at github.com/rshushin/SupplySignal); other projects listed are contract/freelance builds (CPQ AI Assistant, Construction Supervision AI Reporter) plus the Sippurim Bot as one of the cards.
2. **Sippurim Bot product zone** – `Sippurim.html` (Russian-language overview), `Pricing.html`, `Refund.html`, `Privacy.html`, `Terms.html`. This is the original product the domain was registered for: a Telegram bot teaching Hebrew through 350 Israeli songs with parallel texts and a built-in dictionary. €10 one-time payment, lifetime access.

No build step. Pure HTML + CSS + a tiny bit of JS in `Pricing.html` for Paddle. Edit-and-push.

- Repo: `github.com/rshushin/sippurim` (`rshushin` is the personal GH account – switch via `gh auth status` before pushing; user also has a `romansdata` work account that will fail with "Repository not found" if active).
- Hosting: GitHub Pages, auto-deploy on push to `main`. Custom domain via `CNAME` (sippurim.com). Live in 30–90s after push.

## Two zones (load-bearing distinction)

Keep these separate. They have different audiences, different copy register, and different compliance constraints:

- **Brand zone** – `index.html` only. English. Positions an AI consultancy/builder. Should not pitch the bot beyond a single card linking to `Sippurim.html`. SupplySignal is the flagship surface here. **No legal links in the homepage footer** – the legal pages are bot-product specific.
- **Bot zone** – `Sippurim.html` + the four legal pages. Bilingual EN/RU. All copy refers to the bot product. Cross-linked via the pill-shaped `doc-nav` at the top of each page (Overview / Pricing / Refund / Privacy / Terms).

A change is "brand zone" if it touches AI Solutions positioning, the laptop hero, or the project grid. A change is "bot zone" if it touches the bot's marketing copy, pricing, or any of the four legal pages. Don't bundle both in one PR unless the change is purely shared infra (the CSS file).

## Paddle billing (load-bearing – do not break)

The domain was approved by Paddle for the Sippurim Bot product. Live payments flow through `Pricing.html`:

- **Live client token** in the page's inline `<script>`: `live_4120ba8cd45f29fefee63668e39`. Production environment, not sandbox. This is the **client-side** token (like a Stripe publishable key) – designed to be public and already served to every visitor's browser. It can only open a checkout overlay for a `transactionId` minted server-side; it cannot create transactions, read customer data, or move money. The Paddle **API key** (`pdl_live_*` format) lives on the bot's backend, not in this repo, and must never be committed here.
- **Deep-link flow**: the Telegram bot redirects buyers to `Pricing.html?_ptxn=<transaction_id>`. The script reads `_ptxn`, opens `Paddle.Checkout.open({ transactionId })`, and on completion redirects back to `t.me/Sippurim_Bot?start=payment_success`.
- **Fallback**: visitors without `_ptxn` clicking `#buy-now` are bounced to `t.me/Sippurim_Bot?start=pay` so the bot can mint a transaction.

**Things that will break payments – never change without coordinating with the bot codebase**:
- Renaming `Pricing.html` (the bot deep-links to that exact path)
- Changing the Paddle token or environment
- Renaming or removing the `id="buy-now"` element
- Changing the `_ptxn` URL parameter name
- Pointing the success callback at a different URL than `t.me/Sippurim_Bot?start=payment_success`

**For Paddle compliance**: keep `Pricing.html`, `Refund.html`, `Privacy.html`, `Terms.html` live at their current URLs with accurate, bilingual content. They must remain discoverable from somewhere on the site – currently via homepage Sippurim card → `Sippurim.html` → `doc-nav`. Paddle reserves the right to spot-check; if they re-review the domain and can't find the legal pages, the merchant account is at risk.

## Design system

One shared stylesheet: **`assets/css/site.css`**. All pages link to it. Do not reintroduce per-page inline styles – single source of truth.

- **Palette**: dark monochrome. `--bg #0a0a0b`, `--surface #15151a`, `--text #efeee8`, `--accent #f5f4ee`. Borders `--border #232329`. The accent is warm off-white, not pure white.
- **Fonts** (Google Fonts, loaded once per page):
  - **IBM Plex Sans** – `h1` and `.doc-title`. Weight 400.
  - **Space Grotesk** – `h2`, `h3`, header brand. Weight 400 for headings, 600 for brand.
  - **Inter** – body text and UI chrome. Weight 400/500.
- **Components**: `.pill` (CTA, white-on-dark), `.pill.ghost` (outline variant), `.card` (project grid + features), `.doc-nav` (pill-tab nav for bot pages, `.current` highlights the active page), `.buy-callout` (price + CTA panel), `.lang-tag` (EN/RU section labels in legal pages).
- **Sticky header**: blurred background, brand left, optional nav center, single pill CTA right.
- **Responsive**: single breakpoint at `max-width: 900px`. Header nav hides on mobile (no hamburger – intentional, the homepage anchor links aren't critical paths). Grids collapse to single column. Doc-nav becomes horizontally scrollable.

## File map

```
.
├── index.html              # Brand landing (AI Solutions)
├── Sippurim.html           # Bot overview, Russian content
├── Pricing.html            # Bot pricing + live Paddle checkout
├── Refund.html             # 14-day refund policy, bilingual
├── Privacy.html            # Privacy policy, bilingual
├── Terms.html              # Terms of service, bilingual
├── CNAME                   # sippurim.com → GitHub Pages
├── assets/
│   ├── css/
│   │   ├── site.css        # Shared design system – single source of truth
│   │   └── main.css        # Legacy template CSS – NOT linked from new pages, kept for safety
│   ├── js/                 # Legacy jquery/util scripts – NOT loaded by new pages
│   ├── sass/               # Legacy
│   └── webfonts/           # Legacy fontawesome
├── images/
│   ├── supplysignal-hero.png  # Homepage laptop screenshot (16:9, cropped to 16:10 in CSS)
│   └── banner.jpg, pic*.jpg   # Legacy template images, unused by new pages
├── LICENSE.txt
└── README.txt
```

**Legacy files** (`assets/css/main.css`, `assets/js/*`, old template images) are not referenced by any current page. Kept on disk in case external links or search-engine cached pages still reference them. Safe to remove if disk space ever matters, but no upside today.

## Contact addresses (don't confuse them)

- **`roman.shushin@gmail.com`** – appears in every page header as `Get in touch`. For AI Solutions / consulting inquiries.
- **`info@sippurim.com`** – appears in `Refund.html` and `Privacy.html` body copy only. For Sippurim Bot customer support / refund requests / data-deletion requests. **This is the address Paddle and customers expect for the bot product** – do not consolidate to roman.shushin without updating the policy text.

## Conventions

- **Russian content lives in `Sippurim.html`**. Marketing copy for the bot is RU only (preserves the original product positioning Paddle approved). Legal pages are bilingual EN above, `<hr>`, RU below – both must say the same thing.
- **Brand name "Sippurim"** is overloaded: it's both the umbrella brand and the bot product. Context disambiguates. The header brand text always links to `index.html`; the bot is "Sippurim Bot" or "Sippurim – Hebrew Learning Hub" when disambiguation is needed.
- **No personal-name copy** in user-facing text (removed in May 2026). The brand is "Sippurim", not "Roman Shushin". Personal name only appears in `roman.shushin@gmail.com` mailto links and the GH account.
- **Email never as display text** – only as `mailto:` href. The visible CTA always reads "Get in touch" (header) or contextual ("Купить", "Visit site", etc.).
- **Homepage anchor links from subpages**: bot pages link back to homepage sections via `index.html#work`, `index.html#about`, `index.html#contact` – these work because the homepage uses those IDs on its sections.

## Deployment

```bash
git add <changed files>
git commit -m "..."
git push origin main
```

GitHub Pages redeploys automatically. Hard refresh (`Ctrl+Shift+R`) to bypass browser cache after a push. Font CDN (Google Fonts) caches aggressively across sessions; new font additions show up immediately for first-time visitors but take a few minutes for return visitors.

**Pre-push check**: `gh auth status` should show `rshushin` as Active. If `romansdata` is active, switch with `gh auth switch --user rshushin` – wrong account fails with "Repository not found" and the message doesn't make the cause obvious.

## What's planned / what's not

- This site is intentionally minimal. No analytics, no cookie banner, no newsletter signup. If the bot ever needs an analytics setup, it should go into the bot zone only (not the brand homepage).
- The two locked project cards (CPQ AI Assistant, Construction Supervision AI Reporter) are non-clickable placeholders. They name client work under NDA – names of the actual client companies (Camtek for CPQ, Maguf Engineering for construction) must not appear in any committed file.
- SupplySignal positioning on the homepage is curated. Do not auto-sync from SupplySignal's own marketing copy – this is the *brand* view, not the *product* view. Detail goes on supplysignal.app, not here.
