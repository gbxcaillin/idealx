# IdealX SMSF Start — landing page prototype

A clickable prototype of the campaign landing page that takes paid traffic (Meta, Google, social) and turns it into SMSF set-up applications, assisted hand-offs, or nurture leads.

Open `index.html` in any browser. It is one self-contained file: no build step, no server, no dependencies beyond Google Fonts.

## The three journeys

| Journey | Who | What happens |
|---|---|---|
| **Self-serve** | Comfortable online | Landing → contact + consent → fund → trustee → members → banking → review → submitted. Hands off to the secure IdealX account for TFNs and identity checks. |
| **Assisted tap-out** | Wants a person | "Save & get help" is on every step. They can book a 30-minute slot, ask for a callback, or get a resume link. A partial record goes to the sales team with what is done so far. |
| **Abandon and re-engage** | Leaves partway | Screen one asks for consent to follow up. After that, leaving the tab, closing the page, or 45 seconds idle fires an abandonment event and enrols the lead in the nurture sequence. |

"Talk to someone first" on the landing page is the fast path for the second group: it collects contact details and consent, then opens the booking sheet straight away.

## Screens

1. **Landing.** Headline, two calls to action, what you'll need, and a comparison of individual vs corporate trustees. Shows a "welcome back" banner if there is saved progress.
2. **Contact + consent.** Name, email, mobile. A required consent to save progress and be contacted if they stop. Optional marketing consent. This creates the lead.
3. **Fund.** Fund name, trustee structure, member count (1–6), existing super, balance range, timeframe. Inline notes explain the single-member rules for each structure.
4. **Trustee.** Branches on structure. Individual: second trustee details, and relationship if the fund has one member. Corporate: new or existing company, company name or ACN, registered office, confirmation that members are directors.
5. **Members.** One card per member, prefilled from earlier answers. Name, date of birth, address, contact details, tax residency, employer contributions, rollover funds. No TFN, no ID upload.
6. **Banking and contact.** IdealX cash account or own bank, primary contact, adviser involvement, best way and time to reach them.
7. **Review.** Full summary with edit links and three declarations. Submit.
8. **Confirmation.** Reference number and the next steps (secure account invite, deed, ATO registration, rollovers). Optional booking.
9. **Hand-off confirmation.** Shown after a tap-out: what was booked or sent, and which sections are complete.

## Board notes drawer

The "Board notes" button opens a side panel for reviewers:

- **Flow.** Current status, the three journeys, demo controls (fill with example data, simulate abandonment, reset, jump to any screen).
- **Events.** Every event the page would send to the CRM, as JSON, newest first. An optional webhook URL field POSTs them for real.
- **Record.** The lead and application record as it would be stored.
- **Nurture.** The re-engagement sequence and its stop conditions.

## Data design (own database + webhook)

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
| `lead.created` | Consent screen completed | Create lead with source (UTM, gclid, fbclid) and intent (self / assisted) |
| `step.completed` | Any form section completed | Update progress on the lead |
| `nudge.shown` | 45 s idle on a form step | Reporting only |
| `session.abandoned` | Tab hidden, page closed, or demo button | Set status `abandoned`, start nurture sequence |
| `session.resumed` / `session.returned` | They come back | Pause nurture |
| `progress.saved` | "Email me a link" | Send resume link; SMS check-in after 24 h |
| `callback.requested` | "Call me when someone's free" | Create a call task now, with preferred time |
| `booking.created` | Slot confirmed | Create calendar event + call task; confirmation email and SMS; reminder 1 h before |
| `partial.submitted` | Any of the three tap-outs | Attach the partial record to the lead for the caller |
| `application.submitted` | Review submitted | Create the application in the onboarding system; send confirmation; invite members to the secure account |

Status values: `new`, `in_progress`, `saved`, `callback`, `booked`, `abandoned`, `submitted`.

### Nurture sequence (after `session.abandoned`)

| When | Channel | Goal |
|---|---|---|
| T+10 min | SMS with resume link | Resume |
| T+1 hour | Email: what's done, what's left, resume link, booking link | Resume or book |
| T+1 business day | Call 1 (task with the saved record attached) | Finish over the phone |
| T+3 days | Email: individual vs corporate trustee explainer | Resume |
| T+7 days | Call 2; voicemail offers booking link | Book |
| T+14 days | Last-chance email from a named specialist | Book |
| T+30 days | Stop; monthly newsletter if marketing consent given | |

Stops on submit, booking, completed callback, opt-out, or three unanswered calls.

## Deliberately out of scope on this page

- Tax file numbers and identity documents. These are collected inside the secure IdealX account after hand-off, which keeps the ad landing page low-risk.
- Payment. Fees are explained on the call or in onboarding.
- Trust deed, ATO registration and bank account opening. These start after submission.

## Path to production

1. Rebuild as a route in the existing Next.js site (same brand tokens and copy; the form logic ports directly).
2. Persist state server-side on every `input` debounce, keyed by `lead_id`, with a signed resume link emailed on save.
3. Replace the in-page event log with the webhook already stubbed here; point it at the CRM when one is chosen.
4. Wire the slot picker to a real calendar (round-robin across the sales team) and SMS/email confirmations.
5. Add server-side validation, rate limiting and bot protection on the consent screen, since that is the ad-traffic entry point.
6. Pass UTM / click IDs through to the CRM so cost per submitted application can be reported by campaign.
