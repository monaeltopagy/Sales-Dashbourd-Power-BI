# Plumber Rewards Program — MVP Implementation Blueprint

## 1) Locked MVP Scope
This MVP is intentionally limited to:
- Plumber registration
- Activity submission
- Admin approval/rejection
- Automatic points update from approved activities
- Rewards visibility
- Reward redemption requests

Technology for MVP:
- Google Sheets as data store
- AppSheet as application layer

Out of scope for MVP:
- Custom backend/API
- Advanced automation beyond AppSheet workflows
- Full Azure deployment

## 2) Core Business Rules (Single Source)
All rules are governed by the **Settings** sheet and app permissions.

### 2.1 Activity Types & Points
- Point values are configurable in `Settings` (not hard-coded).
- Only active activity types can be submitted.

### 2.2 Reward Tiers
- Rewards and required points are defined in `Rewards`.
- Reward eligibility = `Plumbers.TotalPoints >= Rewards.RequiredPoints` and reward is active with stock > 0.

### 2.3 Status Transitions
- Transactions: `Pending -> Approved` or `Pending -> Rejected`
- Redemptions: `Requested -> Approved` or `Requested -> Rejected` then `Approved -> Fulfilled`

### 2.4 Duplicate-Code Policy
- If `ActivityCode` is provided and duplicate check is enabled for that activity type, code must be unique among non-rejected transactions.
- A duplicate code is auto-flagged as `DuplicateFlag = TRUE` and blocked from approval until reviewed.

### 2.5 Approval Authority
- Only users with Admin role can approve/reject transactions or redemptions.
- Plumbers cannot directly change points or statuses.

## 3) Initial Data Schema
Implemented as Google Sheets tables (CSV templates in `templates/`).

- `Plumbers`
- `Transactions`
- `Rewards`
- `Redemptions`
- `Settings`
- `WeeklyMetrics`

### Points Derivation Rule
`Plumbers.TotalPoints` must be system-calculated:
- Sum of `Transactions.PointsAwarded` where `Status=Approved`
- Minus sum of `Redemptions.PointsUsed` where `Status in (Approved,Fulfilled)`

## 4) Verification Workflow (MVP First)
Primary verification method:
- Manual review by admin (required)

Optional in MVP:
- Unique activity code validation by type via `Settings.RequireUniqueCode`

Approval flow:
1. Plumber submits transaction (`Pending`)
2. Admin checks details/proof/code
3. Admin sets `Approved` or `Rejected`, adds `AdminNotes`
4. On approval, points are awarded and reflected in plumber balance

## 5) Screen Map (AppSheet)
Detailed setup in `appsheet-configuration-guide.md`.

### Plumber Views
- Login/Registration
- Home Dashboard (Current Points, Next Reward, Remaining)
- Add Activity
- Activity History
- Rewards
- Profile

### Admin Views
- Overview dashboard
- Plumber management
- Pending activity review
- Reward management
- Redemption management

## 6) Fraud Prevention Controls
- Unique plumber identity (`PlumberID`, unique phone)
- Optional unique activity code checks
- Admin-only approval rights
- Immutable transaction trail (append-only rows, status-based lifecycle)
- Admin notes + audit fields (`ReviewedAt`, `ReviewedBy`)
- Role-based row/actions in AppSheet

## 7) Pilot Execution (10–20 Plumbers)
Pilot runbook:
1. Select 10–20 plumbers across usage profiles
2. Onboard with a short guide and support contact
3. Run pilot for 4 weeks
4. Collect structured weekly feedback:
   - Submission clarity
   - Approval speed
   - Rewards attractiveness
   - Pain points
5. Apply scoped improvements after pilot review

## 8) Weekly Success Metrics
Tracked in `WeeklyMetrics`:
- Registered plumbers
- Monthly active plumbers (tracked weekly as active-in-last-30-days)
- Submitted activities
- Approved/rejected activities
- Approval rate
- Redemption requests/approved
- Redemption rate
- Repeat active plumbers

## 9) BI-Ready Structure from Day 1
- Use ISO date fields (`YYYY-MM-DD`)
- Keep normalized identifiers (`PlumberID`, `RewardID`, `TransactionID`)
- Preserve activity category in transactions
- Keep reward mapping stable via `RewardID`
- Include timestamps (`CreatedAt`, `UpdatedAt`, `ReviewedAt`)

This structure supports direct import to Power BI with minimal transformation.

## 10) Azure Migration Criteria & Trigger
Migrate only after MVP validation and when one or more are true:
- Usage thresholds: >500 active plumbers or sustained high submission volume
- Performance issues: slow AppSheet/Sheet operations affecting operations
- Admin workload: manual review throughput becomes bottleneck
- Reporting complexity: advanced analytics and integration needs exceed Sheets model

Phased migration target:
1. API layer + authentication
2. Azure SQL for core entities
3. Blob Storage for proofs
4. Automation and analytics services

