# IdealX SMSF Start — landing page prototype

A clickable prototype of the campaign landing page that takes paid traffic (Meta, Google, social) and turns it into SMSF set-up applications, assisted hand-offs, or nurture leads.

Open `index.html` in any browser. It is one self-contained file: no build step, no server, no dependencies beyond Google Fonts.

## The three journeys

| Journey | Who | What happens |
|---|---|---|
| **Self-serve** | Comfortable online | Landing → contact + consent → fund → trustee → members → banking → review → submitted. Hands off to the secure IdealX account for TFNs and identity checks. |
| **Assisted tap-out** | Wants a person | "Save & get help" is on every step. They can book a 30-minute slot, ask for a callback, or get a resume link. A partial record goes to the sales team with what is done so far. |
| **Abandon and re-engage** | Leaves partway | Screen one carries a disclaimer above the Get started button; pressing it is consent to follow up. After that, leaving the tab, closing the page, or 45 seconds idle fires an abandonment event and enrols the lead in the nurture sequence. |

"Talk to someone first" on the landing page is the fast path for the second group: it collects contact details and consent, then opens the booking sheet straight away.

## Screens

1. **Landing.** Headline, two calls to action, the trust section (video slot, who and why, trust signals), what you'll need, and a comparison of individual vs corporate trustees. Shows a "welcome back" banner if there is saved progress.
2. **Contact + Get started.** Name, email, mobile, and an optional marketing tick. A disclaimer sits above the Get started button: pressing it is consent to save progress and be contacted if they stop, and names STOP and unsubscribe. The disclaimer text, version and timestamp are stored on the lead.
3. **Mobile code.** A six-digit code is sent to the mobile and must be entered before the lead exists or any message is sent. The same check guards resume links. The prototype accepts any six digits.
4. **Fund.** General advice warning. Fund name, trustee structure, member count (1–6), existing super, balance range, timeframe. Inline notes explain the single-member rules for each structure. All optional.
5. **Trustee.** Branches on structure, or offers the structure choice again if none was picked. Individual: second trustee's full name, date of birth, contact details, relationship, and residential address when the fund has one member. Corporate: new or existing company, company name or ACN, registered office, shareholders of a new company (defaults to members in equal shares), confirmation that members are directors.
6. **Members.** One card per member, prefilled from earlier answers. First, middle and last names, date of birth, address, contact details, tax residency, employer contributions, rollover funds with member numbers. With a corporate trustee: Director ID (15 digits starting 036) and, for a new company, place of birth. No TFN, no ID upload.
7. **Banking and contact.** IdealX cash account or own bank, primary contact, adviser involvement, best way and time to reach them.
8. **Review.** Full summary with edit links and three declarations. Submit.
9. **Confirmation.** Reference number and the next steps (secure account invite, deed, ATO registration, rollovers). Optional booking.
10. **Hand-off confirmation.** Shown after a tap-out: what was booked or sent, and which sections are complete.

## Brand theme switcher

Colour tokens and fonts were read from idealx.com's stylesheet (its `:root` custom properties) on 14 October 2026. Type and treatment in every theme match the site: Inter headlines (light weight) with one accent word in Instrument Serif underlined in indigo, an eyebrow of indigo code, thin rule and grey label, black pill buttons with an arrow, borderless cloud-grey cards with the site's 24px radius, and a dark footer band with an indigo top rule and small-caps column headings. The system monospace is used for data.

A bar at the top lets reviewers switch the whole prototype between four palettes; the choice is remembered in the browser. With no choice made, the page follows the viewer's light or dark setting. The **Theme** tab in Board notes lists each palette's tokens.

| Theme | Source | Page | Accent | Notes |
|---|---|---|---|---|
| IdealX Light | website | `#ffffff` | `#5158f6` | The site as published: white, ink `#0d0d0f`, slate `#5b5b63`, mist `#e7e7ea`, cloud `#f0f0f0`, indigo-tint `#eeeffe` |
| IdealX Dark | derived | `#0d0d0f` | `#5158f6` | Ink page, ink-2 `#16161a` surfaces, white text; lines use the site's white-at-14% |
| Indigo Tint | derived | `#eeeffe` | `#5158f6` | Site's indigo-tint as the page, white cards |
| Cloud | derived | `#f0f0f0` | `#5158f6` | Site's cloud grey as the page, white cards |

## Trust section (above the form)

Between the hero and the "what you'll need" card the landing page has a slot for a trust video, a "Who is IdealX" paragraph, a "Why set up through IdealX" list and eight trust-signal tiles. Facts already on idealx.com are filled in (AFSL, AUSTRAC registration, ABN, Melbourne). Bracketed items are for the board to supply. Every piece of text in that section is click-to-edit in the browser so wording can be tried during review; edits stay local to that browser, and "Reset trust-section copy" in Board notes clears them.

## Board notes drawer

The "Board notes" button opens a side panel for reviewers:

- **Flow.** Current status, the journeys, demo controls (fill with example data, simulate abandonment, consultant mode, reset, jump to any screen).

Owner only: tap the IDEALX wordmark five times to reveal a "Start fresh" button in the header. It clears the saved application, event log, theme choice and copy edits on that device after a confirmation, and stays visible on that device until hidden from its own sheet.
- **Theme.** The four palettes with hex tokens, and a reset for trust-section copy edits.
- **Events.** Every event the page would send to the CRM, as JSON, newest first. An optional webhook URL field POSTs them for real.
- **Record.** The lead and application record as it would be stored.
- **Nurture.** The re-engagement sequence and its stop conditions.

## Data design (own database + webhook)

Only first name, last name, email and mobile are required, on the contact screen. Every other field is optional so a visitor can always move to the next screen; a consultant completes the rest with them. Format checks still apply when a value is entered (date of birth 18+, email, Australian mobile, 4-digit postcode, 9-digit ACN). The three declarations on the review screen are required to submit. Every field carries an "i" button whose tooltip explains why it is asked, for compliance or operations; the same text is in the SMSF Start Information Flow doc.

The prototype stores state in the browser only. In production each event is a `POST` to a webhook with this shape:

```json
{
  "event": "step.completed",
  "event_id": "evt_x8k2…",
  "occurred_at": "2026-10-14T03:21:09.000Z",
  "lead_id": "lead_7f3a…",
  "application_id": "app_91c0…",
  "step": "fund",
  "status": "in_progress",
  "data": { "section": "fund", "snapshot": { "sections_completed": ["contact", "fund"], "…": "…" } }
}
```

| Event | Fired when | Sales/nurture action |
|---|---|---|
| `lead.created` | Get started pressed on the contact screen | Create lead with source (UTM, gclid, fbclid) and intent (self / assisted) |
| `step.completed` | Any form section completed | Update progress on the lead |
| `otp.sent` / `otp.verified` | Mobile code sent and entered | Lead is created only after verification |
| `nudge.shown` | 45 s idle on a form step | Reporting only |
| `session.left` | Tab hidden or page closed | Start a 30-minute timer; returning cancels it |
| `session.abandoned` | 30 minutes without return (or the demo button) | Set status `abandoned`, start nurture sequence |
| `person.invite_sent` | Applicant invites a named person to fill their own card | Send that person their own link and privacy notice |
| `application.sent_for_confirmation` | Consultant mode: consultant finishes the form | Email the applicant a confirm-and-declare link; nothing drafted until confirmed |
| `session.resumed` / `session.returned` | They come back | Pause nurture |
| `progress.saved` | "Email me a link" | Send resume link; SMS check-in after 24 h |
| `callback.requested` | "Call me when someone's free" | Create a call task now, with preferred time |
| `booking.created` | Slot confirmed | Create calendar event + call task; confirmation email and SMS; reminder 1 h before |
| `partial.submitted` | Any of the three tap-outs | Attach the partial record to the lead for the caller |
| `application.submitted` | Review submitted | Create the application in the onboarding system; send confirmation; invite members to the secure account; upload the click IDs as an offline conversion |

CRM payloads carry contact details, status, campaign source, progress, routing tier and the application reference only. Dates of birth, addresses, director IDs and places of birth stay in the application store and are never sent to the marketing CRM.

Status values: `new`, `in_progress`, `saved`, `callback`, `booked`, `abandoned`, `awaiting_confirmation`, `submitted`. Routing tiers from balance and timeframe: `priority`, `standard`, `explore`, `unknown`.

### Nurture sequence (after `session.abandoned`)

Abandonment is confirmed only after 30 minutes without return; returning cancels queued messages.

| When | Channel | Goal |
|---|---|---|
| Confirmed | SMS with resume link (asks for a mobile code) | Resume |
| +1 hour | Email: what's done, what's left, resume and booking links | Resume or book |
| +1 business day | Call 1 (consultant completes the form, then sends a confirm-and-declare link) | Finish over the phone |
| +3 days | Email: individual vs corporate trustee explainer | Resume |
| +7 days | Call 2; voicemail offers booking link | Book |
| +14 days | Last-chance email from a named consultant | Book |
| +30 days | Stop; newsletter only with marketing consent; record deleted after 90 days if not converted | |

Stops on submit, booking, completed callback, STOP or unsubscribe, or three unanswered calls. Calls are washed against the Do Not Call Register and made only in permitted hours.

## Launch checklist and hazards

The Board notes drawer has a **Launch checklist** tab listing the policy and production items that must be settled before ad spend starts: advice boundary and consultant scripts, ad platform verification, calling rules, bot protection, keeping sensitive fields out of the CRM, pixel placement, retention, duplicates, speed to lead, qualification policy, a real calendar, minors, deed provider mapping, accessibility, offline conversions and field-level analytics. The full hazards table with mitigations and status is in the SMSF Start Information Flow doc.

## Privacy policy

The contact screen links to the Idealx Privacy Policy at https://legal.idealx.com/agreements/privacy and names the collecting entity, overseas processing and the support@idealx.com contact for access, correction and marketing opt-out. At the time of writing that policy is Version 3.0, marked Draft and unpublished; it must be published before launch and should mention SMS and phone contact about saved applications, notice to people named by an applicant, lead retention, and the CRM and SMS vendors in the Overseas Handling Register.

## Deliberately out of scope on this page

- Tax file numbers and identity documents. These are collected inside the secure IdealX account after hand-off, which keeps the ad landing page low-risk.
- Payment. Fees are explained on the call or in onboarding.
- Members under 18. Out of scope for launch.
- Trust deed, ATO registration and bank account opening. These start after submission.

## Path to production

1. Rebuild as a route in the existing Next.js site (same brand tokens and copy; the form logic ports directly).
2. Persist state server-side on every `input` debounce, keyed by `lead_id`, with a signed resume link emailed on save.
3. Replace the in-page event log with the webhook already stubbed here; point it at the CRM when one is chosen.
4. Wire the slot picker to a real calendar (round-robin across the sales team) and SMS/email confirmations.
5. Add server-side validation, rate limiting and bot protection on the consent screen, since that is the ad-traffic entry point.
6. Pass UTM / click IDs through to the CRM so cost per submitted application can be reported by campaign.
