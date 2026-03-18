# Fat Roosters — Claude Guidelines

## Project Overview

Fat Roosters is a restaurant website for a Peri Peri & fried chicken takeaway in Hemel Hempstead, UK. The site is a **single-file static website** (`index.html`) with no build tools, no frameworks, and no backend. It is hosted as a plain HTML file and deployed via GitHub.

**Live repo:** https://github.com/priyanthan11/fat-roosters-website

Key goals:
- Showcase the full menu with prices and category tabs
- Drive orders via Just Eat (external link) and a future integrated ordering flow
- Provide a premium brand experience with animations, dark theme, and bold typography
- Include an AI-powered "Build My Meal" feature (UI placeholder — AI backend to be wired in separately)

---

## Architecture

Everything lives in **one file: `index.html`**. It is structured in this order:

```
<head>
  meta / SEO / OG tags
  Google Fonts (Bebas Neue, Barlow, Barlow Condensed)
  <style> block — all CSS (~750 lines)
    Variables, reset, skip link, nav, hero, sections,
    cart drawer, checkout modal, AI builder
  </style>

<body>
  Skip link
  Delivery banner
  <nav> — logo (assets/fat_logo.gif), nav links, cart button, hamburger
  Mobile menu
  <main>
    Drip decoration
    #hero — title, CTA buttons
    Stats bar
    #menu — tab-filtered menu grid (7 categories, 50+ items)
    #deals — Platters section (3 deal cards)
    #about — brand story
    #findus — location + hours
    <footer>
  </main>
  Cart drawer overlay + aside
  Checkout modal
  AI Food Builder modal + floating button
  <script> block — all JS (~600 lines)
    Constants, MENU_DATA, cart system, checkout,
    AI builder placeholder, scroll/nav/toast utilities
  </script>
```

### Key sub-systems

| System | Description |
|--------|-------------|
| **Cart** | `cart[]` array of `{name, price, emoji, qty}`. `MENU_DATA` maps every item name → price + emoji. `addToCart(btn, name)` / `addToCartDeal(name)` look up data and call `_cartAdd()`. |
| **Cart Drawer** | Slide-in `<aside>` from the right. Shows items with +/− qty controls, per-item remove, subtotal / delivery (£0.99) / service fee (£1.49) / total. |
| **Checkout** | Modal over the cart. Delivery vs Collection toggle, contact + address fields, Card or Cash payment, live order summary, form validation, success screen with `FR-XXXXXX` order ref. |
| **AI Builder** | Full-screen modal with chat thread + meal tray. Voice input via `SpeechRecognition` (en-GB) + text fallback. Currently shows a placeholder response — ready for real AI backend to replace the `processInput()` timeout. |
| **Menu** | Tab-filtered grid driven by `data-cat` buttons and `.menu-category` divs. Scroll reveal via `IntersectionObserver`. |

---

## Development Setup

No install needed. Open `index.html` directly in a browser.

```bash
# Clone
git clone https://github.com/priyanthan11/fat-roosters-website.git
cd fat-roosters-website

# Run — just open in browser
start index.html       # Windows
open index.html        # macOS
```

For live-reload during development, use VS Code Live Server or any static file server:
```bash
npx serve .
```

---

## Key Conventions

### Single-file constraint
All CSS and JS must stay inside `index.html`. Do **not** create separate `.css` or `.js` files unless the user explicitly decides to refactor the project structure.

### CSS
- All colours use CSS custom properties defined in `:root` (`--red`, `--yellow`, `--dark`, `--cream`, etc.)
- Never hardcode colours inline — always reference a variable
- Z-index layers: `body::before` noise 9999 → toast 9000 → checkout 4500 → AI modal 4000 → cart drawer 3600 → cart overlay 3500 → AI float btn 3000 → nav 1000
- Animations use `cubic-bezier(.34,1.56,.64,1)` (spring bounce) for interactive elements; `cubic-bezier(.2,0,.0,1)` for modal entrances
- Responsive breakpoints: 900px (stack layouts) and 600px (compact/mobile)
- All standalone `<button>` elements must have `type="button"`

### JavaScript
- Named constants for all timing magic numbers (`AI_TYPEWRITER_MS`, `AI_THINKING_MS`, `TOAST_DURATION_MS`, `CART_RESET_MS`, `DELIVERY_FEE`, `SERVICE_FEE`)
- Cart state lives in `cart[]` array — never manipulate the count directly
- Menu prices and emojis are defined once in `MENU_DATA` — update there when menu changes
- Use `textContent` (never `innerHTML`) for any user-derived or AI-derived string content (XSS prevention)
- DOM node construction for dynamic lists — no template literal innerHTML for untrusted data
- Focus management: modals capture focus on open (`requestAnimationFrame → focus()`), restore to opener on close

### Accessibility (WCAG 2.1 AA)
- `<main id="main-content">` wraps all page content; skip link at top of body
- All interactive `<button>` elements have `aria-label` where text is ambiguous
- Modal dialogs: `role="dialog"`, `aria-modal="true"`, `aria-labelledby` pointing to visible heading
- Dynamic regions: `aria-live="polite"` on cart count, chat, and cart item list
- State toggles: `aria-expanded` on hamburger, `aria-pressed` on mic button
- Keyboard: ESC closes modals (checkout before cart, cart before AI builder)

### Assets
- `assets/fat_logo.gif` — restaurant logo, used in nav (`width="52" height="52"`) and AI builder modal header

---

## Testing

No automated test suite. Manual verification checklist:

- [ ] Menu tabs switch categories correctly
- [ ] "Add to Cart" on any item increments badge, opens correct item in drawer
- [ ] Cart +/− buttons update qty and totals live
- [ ] Remove (✕) deletes item from cart
- [ ] Checkout: delivery toggle shows/hides address fields
- [ ] Checkout: card/cash toggle shows/hides card form
- [ ] Card number auto-formats with spaces; expiry formats as `MM / YY`
- [ ] Form validation shows toast for empty required fields
- [ ] Success screen shows `FR-XXXXXX` ref and clears cart
- [ ] AI Build modal opens directly (no API key screen)
- [ ] Mic button activates voice in Chrome/Edge; fallback text shown in Firefox
- [ ] ESC closes modals in correct priority order
- [ ] Skip link visible on Tab from top of page
- [ ] Site renders correctly on mobile (375px) and tablet (768px)

---

## Notes for Claude

- **Single file only** — all changes go into `index.html` unless told otherwise
- **No npm, no bundler, no framework** — vanilla HTML/CSS/JS only
- **AI builder is a placeholder** — the `processInput()` function has a `TODO` comment where the real AI call goes; do not add API key logic back unless the user explicitly asks
- **MENU_DATA is the source of truth** for prices — if the user changes a menu item, update both the HTML card and `MENU_DATA`
- **Cart uses `cart[]` array** — never use the old `cartItems` integer variable (it no longer exists)
- **Payment is a UI mock** — no real Stripe or payment processor is integrated; `placeOrder()` just shows a success screen
- **Commit style:** conventional commits (`feat:`, `fix:`, `refactor:`, `chore:`) with a blank line before `Co-Authored-By:`
- **Always push after committing** unless the user says otherwise
- **GitHub account:** priyanthan11
