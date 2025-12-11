# VALIDATION_MATRIX — Scenario → Rules → Expected Checks
Author: Person 3
Version: 1.0
Last Updated: YYYY-MM-DD

This matrix maps each scenario to the rules and acceptance checks that backend must verify automatically or manually.

Columns:
- Scenario ID
- Key Rules Exercised (refer to rules_doc.md)
- Expected Backend Action
- Acceptance Checks (pass/fail items)

SCENARIO 1 — Heavy Convoy Supply Run (Baseline)
- Rules: Convoy-Class Restriction, Load Capacity, Fragile Roads
- Action: choose heavy corridor A→E→F→H→D
- Checks:
  - All segments in route allow Heavy
  - ETA computed (sum base_time + traffic) returned
  - No hard-rule violations in log

SCENARIO 2 — Morning Peak with Light Convoy
- Rules: Max Traffic Soft Limit, ETA Preference
- Action: prefer less congested route if ETA benefit exists
- Checks:
  - Route avoids B–C if alternative ETA ≤ threshold improvement
  - Soft rule reason logged ("soft:avoid:traffic")

SCENARIO 3 — Landslide on C–D
- Rules: Landslide Severity Rule, Blocked Segment Rule, No-Through Rule
- Action: remove segment C–D; reroute or return fail if impossible
- Checks:
  - segment.is_blocked==true logged
  - new route exists that avoids C–D OR routing failure returned
  - log includes incident id and applied effects

SCENARIO 4 — Border Closure at H–D
- Rules: Border Closure Rule, No Alternative for Heavy
- Action: heavy corridor invalid → routing fails
- Checks:
  - backend returns "no_valid_path" status for heavy convoy
  - audit log shows blocked segment and rule name "Border_Closure"

SCENARIO 5 — Severe Storm on G–D with reroute trigger
- Rules: Weather Event Extreme, High Risk Cutoff, Dynamic Re-routing
- Action: block G–D if sensitivity high; reroute from current node
- Checks:
  - G–D blocked or heavily penalized per rule
  - reroute suggested with ETA_old and ETA_new in log
  - reroute applied for P1 if thresholds met

SCENARIO 6 — Multi-incident Chaos Test
- Rules: Combined incident effects, Congestion Avoidance, Risk thresholds
- Action: compute best safe route under multiple penalized segments
- Checks:
  - route avoids segments above hard risk cutoffs
  - if alternative not available, show combined risk and reason
  - track reroute_count ≤ allowed hysteresis limits

SCENARIO 7 — P3 Convoy Avoidance Test
- Rules: Max Traffic Soft Limit, Civil Impact Rule
- Action: avoid B–G, choose B–F–H–D or B–C–D
- Checks:
  - returned route avoids high civil impact segments
  - soft-rule violation must be documented if taken

SCENARIO 8 — Weighted Sum Priority Interaction
- Rules: Priority Ordering, Scheduling
- Action: confirm priority order decides corridor access scheduling
- Checks:
  - Convoy priority_score values match weighted-sum formula
  - Backend schedules P1 before P2 before P3 when corridor conflicts occur
  - Log reflects "priority_enforced" reason for ordering

General acceptance checklist for each scenario:
- Routing performed after applying HARD rules (verify segment removals).
- Soft rules applied as weight modifiers (verify weight deltas in logs).
- All incidents applied via incidents_catalog effects (verify before/after fields).
- Reroute decisions contain: ETA_old, ETA_new, ETA_improvement, risk_old, risk_new, applied_rules, timestamp.

Use this matrix as the automated test plan. Person 2 should attempt to automate these checks or provide a manual test-run checklist.
