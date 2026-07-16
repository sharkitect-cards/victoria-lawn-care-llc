# Generic Card Template (`_template`)

Canonical fallback template for any company that does NOT yet have a per-company template repo (`_template-{slug}`).

## When this template is used

The card-intake n8n workflow (`sdytO7y1ZjrPIanA`) routes to this generic template when the contact's associated HubSpot company has `card_template_slug` empty/null. A "company template prep" task is logged to Supabase so the per-company template can be built later.

## Token contract

This template uses TWO sets of tokens.

### Person tokens (8) — supplied per spawn

| Token | Source |
|---|---|
| `{{PERSON_FULL_NAME}}` | "Jane Smith" |
| `{{PERSON_FULL_NAME_DASH}}` | "Jane-Smith" (vCard download filename) |
| `{{PERSON_FIRST_NAME}}` | "Jane" |
| `{{PERSON_LAST_NAME}}` | "Smith" |
| `{{PERSON_TITLE}}` | Job title |
| `{{PERSON_PHONE_DISPLAY}}` | "(913) 555-1234" |
| `{{PERSON_PHONE_E164}}` | "+19135551234" |
| `{{PERSON_EMAIL}}` | "jane@example.com" |
| `{{PERSON_PHOTO_B64}}` | Optional JPEG base64 (no data: prefix) |

### Company tokens (10) — pulled from HubSpot company + Supabase company_profiles

| Token | Source |
|---|---|
| `{{COMPANY_NAME}}` | HubSpot `company.name` |
| `{{COMPANY_TAGLINE}}` | HubSpot `company.tagline` |
| `{{COMPANY_DESCRIPTOR}}` | HubSpot `company.sharkitect_descriptor` |
| `{{COMPANY_ACCENT_COLOR}}` | HubSpot `company.sharkitect_accent_color` (hex, e.g. `#C01010`) |
| `{{COMPANY_ACCENT_RGB}}` | Same color as RGB triplet (e.g. `192, 16, 16`) — used in rgba() |
| `{{COMPANY_OFFICE_PHONE_DISPLAY}}` | HubSpot `company.phone` formatted "(913) 555-1234" |
| `{{COMPANY_OFFICE_PHONE_E164}}` | Same phone as "+19135551234" |
| `{{COMPANY_OFFICE_ADDR}}` | HubSpot `company.address` (single line) |
| `{{COMPANY_OFFICE_ADDR_MAPS_URL}}` | Full `https://www.google.com/maps/search/?api=1&query=…` URL |
| `{{COMPANY_WEBSITE_URL}}` | HubSpot `company.website` ("https://...") |
| `{{COMPANY_WEBSITE_DISPLAY}}` | Same without scheme ("www.example.com") |

### Conditional blocks

Strip the `<!-- X_START -->` / `<!-- X_END -->` block (inclusive) when the corresponding field is empty:

- `OFFICE_PHONE` — strip if `company.phone` is empty
- `WEBSITE` — strip if `company.website` is empty
- `OFFICE_ADDRESS` — strip if `company.address` is empty

## Files in this template

| File | Purpose |
|---|---|
| `index.html` | Card markup, fully tokenized |
| `manifest.json` | PWA manifest, person+company tokens |
| `logo.svg` | **Placeholder** — n8n fetches `company.logo_url` and overwrites with a square-cropped 512x512 version |
| `README.md` | This file |

## Logo handling for new companies

Because `card_template_slug` is empty, this is the first card the company is getting. n8n attempts to:

1. Fetch `company.logo_url` from HubSpot (or Supabase `company_profiles.logo_url`)
2. Square-crop / pad to 512x512 (the `.logo` container is 120x120 with `object-fit: contain`)
3. Overwrite `logo.svg` (or push as `logo.png` and update the `<img src>`)

If no logo is available, the placeholder square ships and a Supabase task is logged: "Manual logo prep needed for {company}".

## Promoting a generic spawn into a company template

When a company has 1+ live cards using this generic template, manually:

1. Pick the best example card repo as the reference
2. Square-optimize the logo (1:1 aspect, transparent or matched bg)
3. Lock in tagline / descriptor / accent color in HubSpot
4. Create `_template-{company-slug}` and verify byte-exact spawn via `tools/card-spawn.py --dry-run`
5. Set the company's `card_template_slug` in HubSpot + Supabase
6. Future cards for that company use the locked template; existing cards stay as-is unless explicitly upgraded
