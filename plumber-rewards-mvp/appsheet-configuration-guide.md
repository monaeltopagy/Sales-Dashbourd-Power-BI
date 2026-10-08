# AppSheet Configuration Guide (MVP)

## A) Data Sources
Connect these tables from Google Sheets:
- Plumbers
- Transactions
- Rewards
- Redemptions
- Settings
- WeeklyMetrics

## B) Security & Roles
- Add `Role` field in `Plumbers` (`Plumber`/`Admin`).
- Restrict admin actions/views to Role=Admin.
- Restrict plumber row visibility to own records where needed.

## C) Key App Behaviors

### C.1 Activity Submission
Form table: `Transactions`
Required fields:
- `PlumberID`
- `ActivityType`
- `ActivityCode` (optional by type)
- `ActivityDate`
- `ProofImage` (optional but recommended)
- `Notes`

Initial values/system fields:
- `Status = Pending`
- `PointsAwarded = 0`
- `DuplicateFlag` formula from duplicate-code check

### C.2 Admin Review
Admin deck/table view filtered to pending:
- `Status = Pending`

Actions:
- Approve: set `Status=Approved`, set `PointsAwarded` from Settings, set `ReviewedAt`, `ReviewedBy`
- Reject: set `Status=Rejected`, `PointsAwarded=0`, set `ReviewedAt`, `ReviewedBy`, `AdminNotes`

### C.3 Points Balance
`Plumbers.TotalPoints` as virtual/computed value:
- Sum approved transaction points
- Minus approved/fulfilled redemption points used

### C.4 Rewards Visibility
Rewards view shows:
- Active rewards
- Required points
- Eligibility badge when plumber points are sufficient
- Remaining points = `RequiredPoints - CurrentPoints` (min 0)

### C.5 Redemption Request
Form table: `Redemptions`
- Create only if eligible and reward active with stock
- Initial status `Requested`
- Admin approval changes status and decrements stock

## D) View Structure

### Plumber UX
1. Home Dashboard
2. Add Activity (form)
3. My Activities
4. Rewards
5. Profile

### Admin UX
1. Admin Overview (KPIs)
2. Pending Activities
3. Plumbers
4. Rewards
5. Redemptions

## E) Audit & Integrity Rules
- Disallow direct edits to `PointsAwarded` by plumbers.
- Keep transaction rows append-only except controlled admin status updates.
- Require `AdminNotes` on reject actions.
- Keep `CreatedAt`, `UpdatedAt`, `ReviewedAt`, `ReviewedBy` fields maintained.

