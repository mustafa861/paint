# Paint Company Smart Marketplace — Requirements Document

**Version:** Draft v2 (Expanded from original project brief) — July 2026
**Scope:** Website + Mobile Apps — Customer, Painter, Dealer, Company Admin

> This version expands the original brief into a structured specification: numbered functional requirements (FR-xxx) for traceability, a defined job-state machine, a worked estimation formula, a notification event matrix, non-functional requirements, and a list of business decisions that need sign-off before development starts. No original feature has been removed — everything below is clarification and structure added on top of the original scope. Each FR-xxx item is written to be directly referenceable when instructing an AI coding assistant (e.g., "implement FR-CUST-04 through FR-CUST-11").

---

## 1. Executive Summary

The Paint Company Smart Marketplace is a four-sided on-demand platform — Customer, Painter (Applicator), Dealer, and Company — modeled on ride-hailing dispatch logic (Bykea / inDrive style) and applied to the home-painting and paint-retail industry. A customer submits a painting job, the system generates an automatic cost estimate, dispatches a paint-supply request to the nearest dealer, and routes a job notification to the nearest verified painter. The Company sits above all three parties as the control layer: approving painters and dealers, managing pricing, tracking commission, and reporting on marketplace health.

---

## 2. Business Objectives

- Formalize an unorganized, cash-and-referral-driven local market (painting labor + paint retail) into a trackable digital transaction.
- Create two recurring revenue lines: **commission per completed job** and **paint product sales** through the dealer network.
- Build a quality-assurance layer (verified painters, approved dealers, ratings) that the unorganized market currently lacks — this becomes the platform's trust differentiator.
- Generate the data (area-wise sales, top performers, demand patterns) needed to make city-by-city expansion decisions rather than expanding blindly.

---

## 3. Scope

### 3.1 In Scope
All four modules described in this document — Customer, Painter, Dealer, Company Admin — across Website and Mobile Apps (Android + iOS).

### 3.2 Explicitly Out of Scope (This Phase)
Everything listed under "Future Features" in the original brief remains valid direction but is **not** specified in detail here, and should get its own requirements pass once prioritized: AI Paint Quantity Calculator, AI Color Recommendation, Wall Visualization (virtual paint preview), Voice Booking, Video Consultation, Loyalty Rewards, Referral Program, Coupon System, Warranty Management, Complaint Management, CRM Integration, ERP Integration.

---

## 4. User Roles Overview

| Role | Access | Primary Goal |
|---|---|---|
| Customer | Mobile App (primary), Web | Get a painting job done at a fair, transparent price |
| Painter / Applicator | Mobile App | Receive job leads, get paid reliably |
| Dealer | Web Portal (+ optional mobile) | Sell paint stock, fulfill supply requests, manage inventory |
| Company Admin | Web Admin Panel | Run the marketplace: approvals, pricing, commission, analytics |

---

## 5. Glossary

- **Job** — one customer painting request, from submission through closure.
- **Applicator** — used interchangeably with "Painter" throughout this document.
- **Bid / Counter-Offer** — a price a painter proposes that differs from the system-generated estimate.
- **Dispatch Radius** — the geographic radius used to search for the nearest available dealer/painter.
- **Escrow (recommended pattern)** — holding a customer's online payment on the platform until job completion, then releasing it to the painter/dealer minus commission.

---

## 6. Functional Requirements

### 6.1 Customer Module

#### 6.1.1 Registration & Onboarding

| Field | Type | Required | Notes |
|---|---|---|---|
| Full Name | text | Yes | 3–50 characters |
| Mobile Number | text | Yes | Pakistani format, OTP-verified |
| Email | text | No | Used for receipts if provided |
| Address | text | Yes | Must resolve to a GPS pin |
| GPS Location | lat/long | Yes | Device GPS or manual pin-drop |

- **FR-CUST-01** — Mobile number must be OTP-verified before a job request can be submitted.
- **FR-CUST-02** — Mobile number is the primary login identifier; email is optional and used for receipts/notifications only.
- **FR-CUST-03** — Customer can save multiple addresses (Home / Office / Other); at least one is required before job submission.

#### 6.1.2 Paint Job Request Submission

- **FR-CUST-04** — Customer selects **Job Type: New Construction vs. Repaint (Touch-up)**. This is not cosmetic — it changes whether putty/primer are required at all (see §7).
- **FR-CUST-05** — Customer enters Property Type (House / Shop / Apartment / Office) and Area in square feet — either direct entry or a room-by-room (Length × Width) calculator that sums to total area.
- **FR-CUST-06** — Customer selects Paint Type (Distemper / Emulsion / Enamel / Textured / Weatherproof-Exterior) and, if the company stocks multiple brands/tiers, a Brand/Tier.
- **FR-CUST-07** — Customer selects color from a swatch catalog tied to Company Product Management (§6.4.3).
- **FR-CUST-08** — Customer can upload up to 5 photos (JPEG/PNG, ≤10MB each) of the space — used by the painter/dealer for accurate quoting and later as a dispute reference.
- **FR-CUST-09** — Customer selects a preferred date and time **window** (not a guaranteed exact slot — subject to painter acceptance).
- **FR-CUST-10** — System shows a live cost breakdown (Material / Labor / Platform Fee / Tax if applicable) before the customer confirms, using the engine in §7.
- **FR-CUST-11** — Customer can edit or cancel a request free of charge before a painter accepts it. Cancellation after acceptance may carry a fee — policy TBD, see §17.

#### 6.1.3 Painter Discovery & Offers

- **FR-CUST-12** — After submission, customer sees nearby verified painters ranked by distance, rating, and availability (§9).
- **FR-CUST-13** — If a painter counter-offers, customer can Accept / Reject / Wait for another painter.

#### 6.1.4 Tracking, Payment, History

- **FR-CUST-14** — Real-time job status per the state machine in §8, plus live location of the assigned painter/dealer once confirmed.
- **FR-CUST-15** — Payment via Cash, Bank Transfer, Card, or Mobile Wallet (§11).
- **FR-CUST-16** — Digital invoice/receipt issued after payment.
- **FR-CUST-17** — Full job history: status, cost, painter/dealer involved, receipts.
- **FR-CUST-18** — Customer is prompted (skippable) to rate the Painter and the Dealer after job closure.

---

### 6.2 Painter (Applicator) Module

#### 6.2.1 Registration & Verification

| Field | Type | Required | Notes |
|---|---|---|---|
| Name, Mobile, Address, GPS | — | Yes | — |
| CNIC | text | Yes | 13 digits, format-validated, **stored encrypted**, never shown publicly |
| Experience | dropdown | Yes | Years bracket |
| Skills | multi-select | Yes | Interior / Exterior / Texture / Waterproofing / Wallpaper |
| Profile Photo | image | Yes | — |
| Bank Details | text | Yes | Account title, number, bank name / IBAN — for payouts |
| Company Verification | status | System-set | See below |
| Police Verification | file upload | Optional | Character certificate |

- **FR-PTR-01** — Painter account stays in **Pending Verification** and cannot receive job notifications until Company Admin approves it (§6.4.2).
- **FR-PTR-02** — Painter must maintain a minimum average rating (default 3.5★, admin-configurable) to keep receiving new job notifications; dropping below it flags the account for Admin review.

#### 6.2.2 Availability & Matching

- **FR-PTR-03** — Painter toggles Online/Offline. Only Online + Approved + in-radius painters receive job notifications.
- **FR-PTR-04** — Painter gets a push notification (location, area, estimated price, distance) with a response window (default 2 minutes, configurable) to Accept / Reject / Counter-Offer before the system moves to the next candidate.
- **FR-PTR-05** — Counter-offers must stay within an admin-configured variance band (e.g., ±20% of the system estimate); offers outside the band are flagged for Admin review rather than auto-rejected.

#### 6.2.3 Job Execution

- **FR-PTR-06** — "Start Job" requires a GPS check-in confirming the painter is within a configurable distance (e.g., 100m) of the job address — basic fraud prevention.
- **FR-PTR-07** — At least one "Before" photo is required at Start, and at least one "After" photo before the job can be marked Complete.
- **FR-PTR-08** — In-app contact with the customer is number-masked (proxy calling), not a raw phone number exchange — standard ride-hailing safety/privacy pattern.

#### 6.2.4 Earnings & Wallet

- **FR-PTR-09** — Completed-job amount, minus company commission, credits to the Painter Wallet automatically on payment settlement.
- **FR-PTR-10** — Painter can request withdrawal to their bank account, subject to an admin-configured minimum amount and payout cycle.
- **FR-PTR-11** — Earnings Dashboard: daily/weekly/monthly earnings, jobs completed, pending payout.

---

### 6.3 Dealer Module

#### 6.3.1 Registration & Approval

- **FR-DLR-01** — Dealer account requires Company Approval before it appears in the dispatch pool for Nearby Orders.

#### 6.3.2 Inventory Management

- **FR-DLR-02** — Dealer logs Purchase Entries (stock received from company warehouse or manufacturer) with SKU, quantity, batch/date.
- **FR-DLR-03** — Stock auto-deducts on every completed sale (customer or painter-channel) and every fulfilled Nearby Order.
- **FR-DLR-04** — Low Stock Alert (push/SMS) fires when an SKU falls below an admin-configured reorder threshold.
- **FR-DLR-05** — Dealer can request Stock Transfer to/from another dealer or the company warehouse; requires confirmation from the receiving party or Admin.

#### 6.3.3 Sales & Invoicing

- **FR-DLR-06** — System-generated, auto-numbered invoices for Customer Sales and Painter Sales, with tax fields where applicable.
- **FR-DLR-07** — Dealer's outstanding balance owed to the Company (stock taken on credit) is tracked and visible in both Dealer Reports and Admin Finance.

#### 6.3.4 Reports

| Report | Key Columns |
|---|---|
| Daily Sales | Date, transaction count, gross sales, net after commission |
| Monthly Sales | Month rollup, trend vs. prior month |
| Stock | SKU, opening stock, received, sold, closing stock |
| Purchase | Date, source, quantity, cost |
| Profit | Revenue − cost of goods − commission, per SKU/period |

#### 6.3.5 Nearby Orders (Paint Supply Requests)

- **FR-DLR-08** — Dealer receives a Supply Request tied to a confirmed job; can Accept/Reject within a response window — unresponsive after timeout routes to the next-nearest dealer.
- **FR-DLR-09** — Delivery status moves through: Order Received → Preparing → Dispatched → Delivered, visible to both customer and painter (feeds into §8).

---

### 6.4 Company Admin Panel

#### 6.4.1 Dashboard
- **FR-ADM-01** — Real-time KPIs (Customers, Painters, Dealers, Jobs by status, Revenue, Paint Sales), filterable by date range and city.

#### 6.4.2 Approvals
- **FR-ADM-02** — Admin approves/rejects Painter applications (with reason) after reviewing CNIC and verification docs.
- **FR-ADM-03** — Admin approves/rejects Dealer applications.
- **FR-ADM-04** — Admin can suspend/ban any account with a logged reason (audit trail).

#### 6.4.3 Product & Price Management
- **FR-ADM-05** — Admin manages the Product Catalog (SKU, brand, type, color, tier, unit price) that feeds both the Customer color picker (§6.1.2) and the Estimation Engine (§7).
- **FR-ADM-06** — Admin manages Estimation Engine coefficients (coverage rate per liter, primer/putty ratios, labor rate per sq ft) **per city**, since material and labor costs vary regionally.

#### 6.4.4 City Management
- **FR-ADM-07** — Admin enables/disables service cities; only customers in an Active city can submit job requests. This controls phased geographic rollout.

#### 6.4.5 Company-Level Inventory & Finance
- **FR-ADM-08** — Admin manages Warehouse Stock, dispatches to dealers, and raises Purchase Orders to manufacturers when warehouse stock runs low.
- **FR-ADM-09** — Commission per completed job is calculated automatically (configurable %, and can differ for painter-labor commission vs. dealer-material commission).
- **FR-ADM-10** — Admin views/exports Dealer and Painter payment ledgers and processes (or confirms automated) payout runs.

#### 6.4.6 Analytics
- **FR-ADM-11** — Top Dealers, Top Painters, Best-Selling Colors, Area-wise Sales, Revenue trend charts — exportable as CSV/PDF.

#### 6.4.7 Notifications
- **FR-ADM-12** — Admin configures notification templates and trigger rules per event (§13) across Push/SMS/WhatsApp/Email.

#### 6.4.8 Admin Sub-Roles (Recommended Addition — Not in Original Brief)
- **FR-ADM-13** — A single flat "Admin" role doesn't scale past one operator. Recommend sub-roles from day one: **Super Admin** (full access), **City Manager** (scoped to one city's approvals/pricing), **Finance Officer** (commission/payouts, no approval rights), **Support Agent** (view-only + dispute handling, no financial access).

---

## 7. Automatic Paint Estimation Engine

### 7.1 The Missing Link in the Original Example

The original brief's example (1,000 sq ft → 45 liters) only reconciles if "Area" means **floor/carpet area**, and the actual paint requirement is calculated against **paintable surface area** (walls + ceiling), which is larger. This distinction has to be explicit in the system, or the estimate will be wrong.

Using a standard rule-of-thumb multiplier of **~3.0×** (floor area → paintable surface area, for a typical room height) and a coverage rate of **~130–140 sq ft per liter per coat** (typical for mid-tier emulsion, 2 coats):

```
Paintable Area = Floor Area × 3.0        →  1,000 × 3.0 = 3,000 sq ft
Paint (Liters) = (Paintable Area × Coats) / Coverage per Liter per Coat
              = (3,000 × 2) / 133
              ≈ 45 liters   ✓ matches the original example
```

### 7.2 General Formula

```
Paintable Area (sq ft)  = Floor Area × Wall-to-Floor Multiplier
                           (default 3.0; configurable per property type / ceiling height)

Paint Required (L)      = ceil[ (Paintable Area × Coats) / Coverage_per_L_per_coat × (1 + Wastage%) ]

Primer Required (L)     = Paintable Area / Primer_Coverage_per_L
                           (usually 1 coat; skip entirely for same-color repaint — driven by Job Type, FR-CUST-04)

Putty Required (kg)     = Paintable Area × Putty_Rate_kg_per_sqft
                           (mainly for New Construction; usually skipped for simple repaints)

Material Cost           = (Paint L × Price/L) + (Primer L × Price/L) + (Putty kg × Price/kg)

Labor Cost               = Floor Area × Labor_Rate_per_sqft
                           (city-configurable; customer-facing basis should be floor area,
                            since that's the number the customer actually entered)

Total Estimate           = Material Cost + Labor Cost + Platform Fee (if shown separately) + Tax (if applicable)
```

**Important:** every coefficient above (multiplier, coverage rate, primer/putty rates, labor rate) is **illustrative**, not a verified current market rate. They must be calibrated with real coverage charts from the paint brands actually stocked, and real local labor rates — and they should live in Admin Price Management (FR-ADM-06) as configurable values per city, not hardcoded.

### 7.3 Job-Type Branching

- **New Construction** → Primer + Putty + Paint, full multiplier.
- **Repaint, same color** → Paint only (no primer/putty) — much cheaper, must not be quoted the same as new construction.
- **Repaint, color change** (light→dark or dark→light) → Primer + Paint (skip putty in most cases).

This branch should be enforced in the estimation logic, not left to the painter's judgment at the door — otherwise every "repaint" job gets over-quoted and customers will notice.

---

## 8. Job Lifecycle / State Machine

```
SUBMITTED
   ↓
DEALER_PENDING  ⇄  PAINTER_PENDING     (run in parallel)
   ↓
NEGOTIATION            (only if painter counter-offered)
   ↓
CONFIRMED
   ↓
MATERIAL_DISPATCHED
   ↓
MATERIAL_DELIVERED
   ↓
IN_PROGRESS            (painter GPS check-in, "Start Job")
   ↓
COMPLETED              (after-photos uploaded)
   ↓
PAYMENT_PENDING  →  PAYMENT_SETTLED
   ↓
RATED
   ↓
CLOSED

CANCELLED  (reachable from any pre-IN_PROGRESS state, with a reason code)
```

- **FR-FLOW-01** — If a dealer or painter doesn't respond within their response window, the system auto-escalates to the next-nearest candidate rather than leaving the job stuck.
- **FR-FLOW-02** — After N escalations (configurable) with no acceptance, the customer is notified ("still searching") with the option to raise the offer or cancel.
- **FR-FLOW-03** — Cancellation is free before CONFIRMED. After CONFIRMED, a cancellation fee policy is needed to compensate the painter/dealer for reserved time — exact structure is a business decision (§17).

---

## 9. Matching & Dispatch Algorithm

- **FR-MATCH-01** — Default search radius starts small (e.g., 3km) and expands in increments (e.g., +2km) up to a city-configured maximum if nobody accepts.
- **FR-MATCH-02** — **Open decision** (see §17): does the job go to **one** nearest painter at a time (waterfall), or **broadcast** to all painters in radius who can each Accept-at-listed-price or Counter-Offer, with the customer/system picking the best response? The original brief's "Counter Offer" feature implies the latter — recommend broadcast-with-bidding as the default, since it gives customers price competition and matches what's already specified for painters.
- **FR-MATCH-03** — Matching priority should weight distance, rating, and verification status — not distance alone, or the platform's quality guarantee (its main differentiator per §2) is undermined.

---

## 10. GPS & Live Tracking

- **FR-GPS-01** — Painter/Dealer apps report location periodically (e.g., every 10–15s) while in an active job state; less frequently otherwise, to conserve battery.
- **FR-GPS-02** — ETA via Google Maps Distance Matrix API, driving mode by default.
- **FR-GPS-03** — Customer's exact address pin is revealed to the painter/dealer only **after CONFIRMED**; before that, show approximate locality only — standard ride-hailing privacy pattern, worth carrying over here.
- **FR-GPS-04** — Mobile apps need explicit background-location consent flows (Android background location permission, iOS "Always" location) and battery-optimization exemption prompts — this is an app-store review requirement, not optional polish.

---

## 11. Payment System

- **FR-PAY-01** — Supported methods: Cash, Bank Transfer, Card, Mobile Wallet. Common Pakistani rails worth evaluating: JazzCash, EasyPaisa, and bank-gateway card processing — final choice depends on merchant account terms and per-transaction fees, which should be confirmed directly with providers rather than assumed here.
- **FR-PAY-02** — **Recommended: escrow pattern.** Online payments are held in a platform wallet until COMPLETED, then released to the painter/dealer minus commission. This is standard for services marketplaces and materially reduces dispute risk.
- **FR-PAY-03** — **Cash jobs still owe commission.** This is easy to miss in a first draft: if the customer pays the painter/dealer directly in cash, the company's commission isn't automatically collected. Needs an explicit mechanism — e.g., a refundable security deposit/wallet balance for painters/dealers that commission is deducted from, or periodic invoicing and reconciliation.
- **FR-PAY-04** — Card numbers are never stored directly — use the payment gateway's tokenization so the platform stays out of PCI-DSS scope as much as possible.
- **FR-PAY-05** — Refund and dispute handling needs a defined path (§17 — business decision on exact policy).

---

## 12. Rating & Review System

- **FR-RATE-01** — Bi-directional ratings: Customer↔Painter, Customer↔Dealer, each 1–5★ + optional comment.
- **FR-RATE-02** — Minimum average rating enforced for Painters (§6.2.1) and recommended for Dealers too.
- **FR-RATE-03** — Appeal path: a painter/dealer can flag a rating as unfair for Admin review rather than having no recourse.

---

## 13. Notification System — Event Matrix

| Event | Customer | Painter | Dealer | Admin | Channel(s) |
|---|---|---|---|---|---|
| Job submitted | Confirmation | — | — | — | Push, SMS |
| Painter accepted | Notify + profile | — | — | — | Push |
| Counter-offer received | Action required | — | — | — | Push, SMS |
| Dealer accepted supply request | Notify | — | Confirmation | — | Push |
| Material dispatched | ETA update | — | — | — | Push |
| Job started | Notify | — | — | — | Push |
| Job completed | Payment prompt | Earnings update | — | — | Push, SMS |
| Payment settled | Receipt | Wallet credited | Ledger updated | — | Push, Email |
| Low stock | — | — | Alert | Alert (if company-wide) | Push, SMS |
| New painter/dealer application | — | — | — | Review-queue alert | Push, Email |
| Rating received | — | Notify | Notify | — | Push |

---

## 14. Reports & Analytics — Additional Requirements

- **FR-RPT-01** — All reports exportable as CSV and PDF.
- **FR-RPT-02** — Optional scheduled digest emails (daily/weekly) for Dealers and Admin.
- **FR-RPT-03** — Financial/transactional records retained per applicable Pakistani tax record-keeping requirements — confirm the exact retention period with a tax advisor rather than assuming a fixed number here; this document is not a source of tax/legal guidance.

---

## 15. Non-Functional Requirements

| Category | Requirement |
|---|---|
| Performance | Read APIs < 500ms; job dispatch initiated within 5s of submission |
| Scalability | Architecture should scale horizontally per city as rollout expands |
| Security | CNIC & bank details encrypted at rest (AES-256) and in transit (TLS 1.2+); RBAC throughout; no raw card storage |
| Availability | Define an explicit uptime target (e.g., 99.5%) before launch — this is a business decision, not a default |
| Localization | Bilingual UI (Urdu + English) — many painters will be more comfortable in Urdu than English |
| Data Privacy | Data-minimization and access-logging on sensitive fields (CNIC, bank details, GPS history); Pakistan's data-protection legal landscape has been evolving — get current legal guidance rather than relying on this document |
| Audit Logging | Every Admin approval/suspension/price-change action logged with actor, timestamp, and reason |

---

## 16. Technology Stack

**Option A — Original brief**

| Layer | Choice |
|---|---|
| Mobile | Flutter |
| Web | React.js |
| Backend | Laravel or Node.js |
| Database | MySQL |
| Cloud | AWS or GCP |
| Maps | Google Maps API |
| Push | Firebase Cloud Messaging |

**Option B — Leverages your existing stack (DukaanLink, BizFlow AI, Debate Me)**

| Layer | Choice |
|---|---|
| Web (Admin/Dealer Portal) | Next.js |
| Backend/API | FastAPI (Python) |
| Database/Auth/Realtime | Supabase (Postgres — PostGIS extension handles the geo-radius queries in §9 well; built-in Realtime is useful for live tracking/status updates) |
| Mobile (Customer + Painter apps) | Flutter or React Native — still needed regardless of backend choice; background GPS + reliable push are genuinely hard to do well from a PWA |
| Push | Firebase Cloud Messaging (works fine alongside either backend) |
| Maps | Google Maps Platform (Maps SDK, Directions/Distance Matrix API) |
| AI (optional, future) | Groq (llama-3.3-70b) — could power the "AI Color Recommendation" future feature or a customer FAQ assistant later |

Option B gets you moving faster given prior delivery experience with this exact combination. That said, Option A (Laravel/MySQL) is a genuinely reasonable choice too — Laravel has a much larger local hiring pool in Pakistan than FastAPI/Next.js, which matters if you plan to bring in other developers later rather than staying solo.

---

## 17. Assumptions & Open Questions

These need a decision before (or early in) development — they're not resolved by this document:

1. **Pricing model** — Is the system estimate the final price, or a starting point subject to painter bidding? (§9/§8 assume bidding is allowed, per the original "Counter Offer" feature.)
2. **Dispatch model** — Sequential single-offer, or broadcast-to-multiple with bidding? (§9 recommends broadcast; needs sign-off.)
3. **Escrow** — Should online payments be held on-platform until job completion? (Recommended: yes — §11.)
4. **Cash-job commission** — Security deposit, wallet debit, or periodic invoicing? (§11, FR-PAY-03.)
5. **Cancellation fee** — Flat fee or percentage, and starting at which job state?
6. **Does the Company manufacture/own the paint stock**, or is this purely a marketplace connecting independent existing dealers? This changes how "real" the Warehouse/Purchase Order flow (§6.4.5) needs to be at launch.
7. **Multi-city pricing** — Per-city coefficients from day one, or uniform pricing at launch with city variance added later?
8. **Language** — Bilingual UI at launch, or English-only MVP with Urdu added in Phase 2?
9. **Launch city** — Which city first? This decides which Estimation Engine coefficients (§7) need real data first.

---

## 18. Suggested Build Phasing (Reference Only — Not Requested as a Separate Deliverable, Kept Brief)

- **Phase 1 (MVP):** Job submission + estimate, single city, sequential (non-bidding) matching, Cash + one digital payment method, basic Admin approvals/dashboard.
- **Phase 2:** Bidding/counter-offer, wallet + escrow, multi-city, full reporting suite, ratings.
- **Phase 3:** Advanced analytics, notification automation, and the Future Features list (AI color recommendation, wall visualization, etc.).

A sprint-level MVP breakdown can be put together separately if that'd help once this spec is signed off.
