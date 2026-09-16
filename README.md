# Rocket Loans Email Templates

Production-ready HTML email templates for Rocket Loans, built from Figma and deployed through
Salesforce Marketing Cloud (SFMC). Every file is a standalone, self-contained `.html` document —
paste it into an SFMC content block and it renders. There is no build step and no dependencies.

## Repository layout

| Folder | Files | What it is |
| --- | --- | --- |
| `Welcome Series/` | 7 | Lifecycle triggers — welcome, early warning, autopay switch, review requests |
| `Rocket Loans Welcome Series journey/` | 8 | Onboarding journey — Home Purchase 1–4 and Refinance 1–4 |
| `Refi Eligible Nurture/` | 8 | Quarterly refi nurture — Q1–Q4, each with a Use Cases and a Survey/Motivation variant |
| `Recapture Prescreen/` | 4 | Refi Prescreen and Refi Paid Off, each in a Prescreened (vA) and Not Prescreened (vB) cut |
| `TWNFY Doc Requests/` | 3 | Document-request emails 1–2, plus a modules file of 27 tracking-item cards |
| `RLLD Create Acct/` | 2 | In-process account creation |
| `RMSC Login/` | 2 | Offer pairing and login reminder |
| `In-Process-Repayment/` | 3 | Status 400 in-process repayment - offer update, APR change, choosing a repayment method |
| `gmail-safe-images/` | 3 | PNGs re-encoded for Gmail, which rejects some source assets |

`TWNFY tracking item modules.html` is not a sendable email. It is a library of 27 standalone card
modules, in Figma reading order, to be dropped into a doc-request email depending on which items a
given borrower still owes.

## Build conventions

These hold across every file in the repo. Match them in anything new.

- **Layout** — nested tables only. No `div` layout, no flexbox, no grid.
- **Width** — fluid 800px: `width="100%"` on the table plus `max-width:800px` inline.
- **Breakpoints** — two. At `max-width:820px` the `.wrap` table drops to full width; at
  `max-width:640px` the mobile layout takes over. Helper classes: `.stack`, `.gutter`,
  `.btn-full`, `.d-only` / `.m-only`, `.hero-d` / `.hero-m`. The eight files in
  `Rocket Loans Welcome Series journey/` predate the 820px rule and only carry the 640px one.
- **Type** — `'WNTL Text', Arial, Helvetica, sans-serif`. Only Regular (400) and Medium (500) are
  `@font-face`-declared, loaded from `rocketmortgage.com/nsassets/fonts/wntl/`.
- **Outlook** — VML `<v:roundrect>` / `<v:oval>` fallbacks with `<w:anchorlock/>` behind every
  rounded button and circle, and an `mso` Arial stack, since Word cannot load the WNTL webfonts.
- **Personalization** — SFMC substitution strings, e.g. `%%emailAddress%%`.
- **Images** — hosted absolutely (mostly `framerusercontent.com`). Every `<img>` carries explicit
  `width`, `height`, `border:0` and `display:block`. Nothing is referenced relatively.

## Rendering gotchas

Each of these caused a real, client-reported bug at some point. They are not theoretical.

**Always include the DPI guard.** Without it, Outlook on a 120-DPI display scales the whole email
by 1.25×:

```html
<!--[if mso]><xml><o:OfficeDocumentSettings>
  <o:PixelsPerInch>96</o:PixelsPerInch>
</o:OfficeDocumentSettings></xml><![endif]-->
```

**Paint every nested cell that sits on a coloured background.** Word does not inherit
`background-color` into nested tables, so an unpainted `&nbsp;` spacer cell renders *white* on top
of your colour. Prefer padding over spacer cells; where a nested table is unavoidable, put
`bgcolor` *and* inline `background-color` on every cell inside it.

**Never put a background on a full-bleed padding cell.** It paints over the parent's rounded
corners and the radius disappears in Gmail.

**`border-radius` needs `border-collapse:separate`.** The global
`table, td { border-collapse:collapse !important }` reset kills the radius in WebKit. Add
`border-collapse:separate !important` inline, but only on tables that actually have a border.

**Dark mode: no near-black borders.** `#050505` and `#111111` outlines turn into hard black boxes
in Outlook dark mode. Use `#767676` and give the element an explicit fill so it is self-contained.

**Word ignores `margin` on tables.** Use padding, or a painted spacer row.

**Do not use percentage widths on columns that stack.** Outlook resolves them unpredictably. Fixed
pixel widths only.

**`WNTL Display` is never declared.** It is referenced in the CSS but has no `@font-face` rule, so
it always falls back. Do not rely on it for line-break control — use an explicit
`<br class="d-only">` (with a preceding space, or the words run together on mobile).

## Before you ship

Test in Litmus. The clients that have actually broken in the past are Outlook 2016 / 2019 / 2021,
Outlook 2021 Dark, Office 365 at 120 DPI, and Gmail (border-radius and image handling).

When editing an existing template, diff against the committed version before and after. Several of
these files carry hand-tuned padding and link destinations that are easy to flatten by regenerating
a whole folder at once.

## Open items

- `In-Process-Repayment/Email1-choose-a-repayment-method.html` carries three `placehold.co`
  stand-ins, marked `PLACEHOLDER` in the file: the support photo (desktop and mobile crops) and
  the lightbulb icon. No existing template uses these images. Swap the three `src` values once the
  assets are hosted.

- Eight files still carry `scale-down-to=512` on a hero or feature image, which caps the asset
  below its display width. Affected: both `RMSC Refi Paid Off` cuts, `Home Purchase Email 3`,
  `Refinance Email 3` and `4`, `TWNFY 2`, and both `Status 900-Switch to autopay` cuts.
- The four "other uses" link destinations in `Q3 - Faux Survey.html` are unconfirmed by the client.
- `Recapture Prescreen/` filenames do not follow the `N - Description` convention used elsewhere.
