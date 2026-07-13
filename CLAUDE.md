# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**Pandit4Puja** — single-page static website for a Vedic pooja booking service (Pune, Maharashtra). Customers browse sevas, pick a date + time slot, fill in their details, and confirm the booking on WhatsApp. No backend, no payment gateway, no build step.

Hosted on GitHub Pages (remote: `https://github.com/Bjoshi9/Pandit4Pooja.git`).

## Architecture

Two files:

- **`index.html`** — all markup, CSS, and vanilla JS (~1090 lines). The script section has six subsystems: booking state, mobile nav, sticky header / back-to-top, reveal-on-scroll IntersectionObserver, diya progress panel, calendar, testimonial carousel, FAQ accordion, and the `initPoojaCatalog()` loader that renders the seva grid + chips from data.
- **`poojas.json`** — the pooja catalog. Array of objects with `id`, `name`, `shortName`, `description`, `duration`, `icon` (inline SVG markup, optional), `iconBg` (CSS background for the featured card, optional), `featured` (boolean). Fetched at runtime by `loadPoojas()`. **Do not** include a `price` field — prices are intentionally absent from the rendered site.

### Data layer (loadPoojas / initPoojaCatalog)

`loadPoojas()` is the single function that owns data access. It first tries `fetch('poojas.json')`. If that fails — typically because the file was opened via `file://` (browsers block fetch on `file://` for security) or the host refused — it falls back to an inline `FALLBACK_POOJAS` array embedded in `index.html` so the site never shows an empty grid. A console warning is logged either way.

**This is the only function to change when migrating to a backend.** Replace its body with `const { data, error } = await supabase.from('poojas').select('*'); return data;` (or Firebase equivalent). The fallback can stay or go.

The renderer (`initPoojaCatalog`) populates two containers from the same data: `#sevaGrid` (the marketing cards) and `#serviceChips` (the booking-widget picker). The static "Something Else" chip is kept as a sibling of `#serviceChips` so the JSON loop only writes real poojas.

### Booking flow

User picks a seva chip (or "Book This" card) → picks a date on the calendar → picks a time slot → fills name + phone → clicks the WhatsApp confirm button. JS validates required fields, builds a `https://wa.me/<WHATSAPP_NUMBER>?text=…` URL with the booking details, opens it in a new tab, and shows a success box.

## Key configuration

- **`WHATSAPP_NUMBER`** constant in the script — current placeholder is `918487846352`. **TODO marker in code**: replace with the real number before going live.
- **Footer contact details** — phone display, email (`hello@pandit4puja.in`), and the "Placeholder business" note in the copyright line all need to be updated before launch.
- **Google Calendar sync** — deliberately not implemented. The current calendar shows any future date up to 6 months out as selectable; the admin confirms availability manually on WhatsApp. When ready to add real sync, the right path is a Supabase table + a periodic Google Calendar API pull (requires server-side OAuth, so a backend is unavoidable for this feature).

## Common tasks

No build, no test runner, no linter, no package manager.

- **Preview locally**: `python3 -m http.server` from the repo root, then open `http://localhost:8000`. **Do not** open `index.html` via `file://` — browsers block `fetch()` on `file://` and the catalog will fall back to the inline array. Other options: `npx serve`, VS Code Live Server, or push a branch and use GitHub Pages preview.
- **Deploy**: push to `main` on GitHub — Pages serves the file at the configured path.
- **Theme change**: edit only the `:root` CSS custom properties near the top of the `<style>` block.
- **Add a new seva**: add an entry to `poojas.json`. The card and chip both render from the same `name` field, so no HTML edits needed.
- **Edit an existing seva** (description, duration, icon): edit the matching entry in `poojas.json`. Mirror changes to `FALLBACK_POOJAS` in `index.html` if you want file:// previews to also reflect them.

## Conventions

- All copy is in English with selective Devanagari accents in section eyebrows (e.g. `सेवा · OUR SEVAS`). Preserve this bilingual flavor when editing copy.
- Use the existing CSS variables (`--kumkum`, `--marigold`, `--turmeric`, `--brass`, `--ivory`, `--dusk`, `--whatsapp`) rather than hard-coding colors.
- Devanagari characters in section labels are HTML entities (e.g. `&#2310;`) — keep using entities for new ones, to keep the file plain-ASCII-friendly.
- Respect `prefers-reduced-motion` — every animation/auto-scroll branch already checks this; do the same for new motion.
- Icons are inline SVGs (no icon library). Match the existing stroke style (`stroke-width:1.8`, `stroke-linecap:round`, `stroke-linejoin:round`).
- The catalog must NEVER include a `price` field — the site intentionally shows no prices. The user pays the Purohit directly in person after the ritual.
