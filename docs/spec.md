# Bakery pre-orders: Specification

Status: Approved 2026-11-23  ·  UI: yes  ·  Owner: repo owner

## Problem
Customers of a single bakery want to make sure the bread they want is there when they
arrive. Staff at the counter need to know who is coming to collect next, and which
orders have already been handed over.

## Goals
- Customers reserve bread for a pickup slot from their phone, without an account, and
  can cancel until the ordering cutoff.
- Staff see the upcoming pickups on a counter tablet and mark each order collected or
  no-show.
- The owner manages products, opening hours, slot length and slot capacity without a
  developer.
- Slots never take more orders than their capacity.

## Non-goals
- Online payment, deposits or refunds. Customers pay at pickup; prices shown are what
  they will pay.
- Customers editing an order. They cancel and order again.
- Customer accounts, login or order history.
- More than one bakery (multi-tenancy, signup, billing).
- Native mobile apps.
- Per-product stock limits. Capacity is per slot only.
- Reminders, SMS or any notification other than the confirmation email.
- No-show penalties.
- Per-staff accounts or attributing actions to individual staff.

## Users and context
- **Customer:** orders on a mobile-friendly website, as a guest with a name and contact
  details.
- **Counter staff:** use a tablet at the counter that stays signed in to one shared shop
  account.
- **Owner:** uses an admin page behind a separate owner login.
- **Platforms:** web only. The customer site works on phone browsers; the staff and admin
  pages are web pages used on a tablet or desktop.

## Acceptance criteria

### Customer ordering
- **AC-001** Browse products
  Given the owner has active products / When a customer opens the order page / Then they
  see each active product's name and price in EUR, and products the owner has
  deactivated are not shown.
- **AC-002** See available slots
  Given opening hours, slot length and capacity are configured / When a customer chooses
  a pickup date from today to 7 days ahead / Then they see that day's slots, and full
  slots are shown as unavailable and can't be selected. Dates further ahead can't be
  chosen.
- **AC-003** Place an order
  Given a customer has chosen at least one product, each with a quantity from 1 to that
  product's maximum, and an available slot / When they submit with a name, an email
  address and optionally a phone number / Then the order is saved with status
  `reserved`, and the confirmation page shows an order code, the slot, the items with
  prices, and the total in EUR to pay at pickup.
- **AC-004** Slot capacity is never exceeded
  Given a slot has one place left / When two customers submit orders for it at the same
  time / Then exactly one order is accepted, and the other customer is told the slot is
  full and asked to choose another.
- **AC-005** Ordering cutoff
  Given the owner has set a cutoff time (AC-021), and that time on the day before a
  slot's date has passed / When a customer tries to order for that slot / Then the slot
  is not offered, and a submit for it is rejected. Same-day orders are therefore never
  possible.
- **AC-006** Input validation
  Given an order with no items, a quantity above the product's maximum, a missing name,
  a missing or invalid email address, or an invalid phone number / When the customer
  submits / Then no order is saved, and each problem is shown next to its field.
- **AC-007** Confirmation email
  Given an order was placed (AC-003) / When it is saved / Then an email with the order
  code, slot, items, total, bakery name and address (AC-026) and a cancel link is sent to the
  customer's email address.
- **AC-008** Email failure doesn't lose the order
  Given the email provider is unavailable / When an order is placed / Then the order is
  still saved and confirmed on screen, and sending is retried with backoff for up to 24
  hours. If it still hasn't been sent, the order is marked "email not sent" on the
  owner's admin pages. No other notification is sent.
- **AC-009** Cancel via email link
  Given a `reserved` order whose ordering cutoff (AC-005) hasn't passed / When the
  customer opens the cancel link and confirms / Then the order's status becomes
  `cancelled`, its place in the slot is freed, and it no longer appears on the counter
  view. After the cutoff, the link says the order can no longer be cancelled online and
  shows the bakery's phone number (AC-026). The link only works for that one order and can't
  be guessed from the order code.

### Counter tablet
- **AC-010** Upcoming pickups
  Given today has `reserved` orders / When staff open the counter view / Then they see
  today's `reserved` orders grouped by slot, earliest slot first, each with the customer
  name, order code, items and total. The view shows today only; today's collected and
  no-show orders are listed separately.
- **AC-011** New orders appear without a manual reload
  Given the counter view is open / When a customer places an order for today / Then it
  appears in the counter view within the time set by QS-002, with no action from staff.
- **AC-012** Find an order
  Given a customer gives their name or order code at the counter / When staff search for
  it / Then the matching orders for today are shown.
- **AC-013** Mark collected
  Given an order is `reserved` / When staff mark it collected / Then its status becomes
  `collected`, it leaves the upcoming list, and it is listed under today's collected
  orders.
- **AC-014** Undo a mistaken mark
  Given staff marked an order `collected` or `no-show` by mistake today / When they undo
  it / Then the order returns to `reserved`.
- **AC-015** Mark no-shows at end of day
  Given today has `reserved` orders whose slots have passed / When staff mark them
  no-show (one at a time or all at once) / Then their status becomes `no-show`.
- **AC-016** Staff session
  Given the tablet is signed in to the shop account / When it is reopened later / Then it
  is still signed in, until 30 days after sign-in or until the owner revokes staff
  sessions (AC-025). The shop account can't open the owner's admin pages.

### Owner admin
- **AC-020** Manage products
  Given the owner is signed in / When they add a product, edit its name, price (EUR) or
  maximum quantity per order, or deactivate/reactivate it / Then the customer page
  reflects the change on its next load, and existing orders keep the name and price as
  they were when ordered.
- **AC-021** Configure opening hours and slots
  Given the owner is signed in / When they set opening hours per weekday, the slot
  length, the orders-per-slot capacity and the previous-day cutoff time / Then the
  customer page offers slots that match. Closed weekdays have no slots.
- **AC-022** Capacity changes don't cancel orders
  Given a slot already has more orders than a newly lowered capacity / When the owner
  saves the lower capacity / Then the existing orders stay, and the slot accepts no new
  orders until it is below capacity.
- **AC-023** Owner-only access
  Given someone who isn't signed in as the owner / When they request an admin page or
  admin API / Then access is refused.
- **AC-024** Close specific dates
  Given the owner is signed in / When they mark a date closed / Then that date offers no
  slots. If `reserved` orders already exist for it, they stay `reserved` and the owner
  sees them listed with the customers' contact details.
- **AC-025** Revoke staff sessions
  Given the shop account is signed in on one or more devices / When the owner chooses
  "sign out all staff devices" / Then every staff session ends at once, and each device
  must sign in again.
- **AC-026** Bakery settings
  Given the owner is signed in / When they set the bakery name, address, phone number,
  sender email address, timezone, date/time and price format, and the privacy notice text
  / Then the confirmation email, the cancel page and the customer pages use them, slot
  times and cutoffs are interpreted in that timezone, and the order page links to the
  privacy notice.

### Privacy
- **AC-030** Delete a customer's data on request
  Given a customer asks for their data to be erased / When the owner erases it by order
  code or contact details / Then the name and contact details on that customer's orders
  are removed, and the counts of orders and items are kept.
- **AC-031** Retention
  Given an order's pickup date is more than 30 days ago / When the retention job runs /
  Then the name and contact details on that order are removed.

## Quality scenarios
- **QS-001** performance: customer ordering
  Stimulus: a customer submits an order · Environment: peak load of 50 concurrent
  customers browsing and 15 orders for one slot · Response: the confirmation page is
  shown · Measure: p95 ≤ 300 ms, server-side
- **QS-002** performance: counter freshness
  Stimulus: an order for today is placed · Environment: counter view open, normal shop
  Wi-Fi · Response: the order appears on the tablet · Measure: within 3 s
- **QS-003** performance: mark collected
  Stimulus: staff tap "collected" · Environment: normal shop Wi-Fi · Response: the order
  moves out of the upcoming list · Measure: p95 ≤ 200 ms
- **QS-004** reliability: availability
  Stimulus: customers and staff use the system · Environment: during opening hours ·
  Response: ordering and the counter view work · Measure: ≥ 99.9 % per month, counted
  over opening hours
- **QS-005** reliability: counter network loss
  Stimulus: the tablet loses network · Environment: counter view open · Response: the view
  shows that it is offline and when it last updated, and a mark that wasn't saved is
  never shown as saved · Measure: offline indicator within 5 s
- **QS-006** security: brute-force protection
  Stimulus: repeated failed sign-ins to the shop or owner account · Environment:
  internet-facing · Response: after 5 failed attempts for the same account and IP
  address within 15 minutes, further attempts are refused · Measure: refused for 15
  minutes
- **QS-007** accessibility: customer site
  Stimulus: a customer uses a screen reader or keyboard only · Environment: phone and
  desktop browsers · Response: they can place an order · Measure: WCAG 2.2 AA, no failures

## Constraints
- One bakery. Single-tenant. Volume: up to 100 orders a day, up to 15 in one slot.
- Web only; the customer site must work on phone browsers.
- GDPR applies: lawful basis, retention limit, erasure on request, and a privacy notice
  on the order page (AC-026).
- UI and emails in English only. Timezone and formats are set by the owner.
- Hosting follows the default self-hosted Docker Compose setup unless the design says
  otherwise.
- Deadline: none stated.

## Data and privacy
- **Controller and lawful basis:** the bakery owner is the data controller; the lawful
  basis is performance of a contract (the pre-order).
- **Personal data:** customer name, email address (required), phone number (optional),
  the order and its pickup slot.
- **Who may see it:** counter staff (today's orders) and the owner (all orders). It isn't
  shown publicly; the confirmation page is only reachable by the customer who ordered.
- **Retention:** personal fields are removed 30 days after the pickup date; the
  anonymised order counts are kept.
- **Erasure:** on request, by the owner (AC-030).
- **Money:** none handled. Prices and totals are shown in EUR for payment at pickup.

## Open questions
| # | Question | Owner | Blocks |
|---|---|---|---|
| — | None open. | | |

Resolved 2026-11-23: Q1 (7-day window), Q2 (owner-set previous-day cutoff), Q3 (cancel
via email link until cutoff), Q6 (email required, phone optional), Q7 (prices in EUR,
per-product max quantity), Q10 (owner closes specific dates), Q11 (today only), Q12
(30-day staff session, owner revokes all), Q13 (30-day retention), Q4 (100/day, 15 per
slot), Q5 (strict targets), Q8 and Q16 (admin settings, AC-026), Q9 (24h retry, then
flag), Q14 (5 per 15 min), Q15 (WCAG 2.2 AA), Q17 (English; owner sets timezone and
formats).

## Changelog
- 2026-11-23: created
- 2026-11-23: resolved Q1–Q3, Q6, Q7, Q10–Q13. Added AC-009 (cancel), AC-024 (closed
  dates), AC-025 (revoke staff sessions) and the `cancelled` status. Currency is EUR.
- 2026-11-23: resolved Q4, Q5, Q8, Q9, Q14–Q16 and part of Q17. Added AC-026 (bakery
  settings); filled all QS measures; added lawful basis and controller.
- 2026-11-23: resolved Q17 (English UI and emails).
- 2026-11-23: approved.
