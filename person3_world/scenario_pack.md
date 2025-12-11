# SCENARIO PACK — Predefined Simulation Scenarios for Testing
Author: Person 3
Version: 1.0
Last Updated: YYYY-MM-DD

This document defines ready-to-run scenarios for testing routing, re-routing, incidents, priority behavior, and rule enforcement.  
Each scenario includes: convoy info, applied incidents, expected routing behavior, and evaluation criteria.

---

# SCENARIO 1 — Heavy Convoy Supply Run (Baseline)
Convoy:
- class: Heavy
- from: A
- to: D
- priority: P2
- weight: 20 tons

Conditions:
- No incidents active
- Normal traffic (time_of_day = 04:00)
- Weather: dry

Expected Route:
A → E → F → H → D  
(Only valid heavy corridor)

Success Criteria:
- Routing avoids B–C, B–G, F–G because heavy not allowed.
- ETA based purely on base_time_min + traffic.

---

# SCENARIO 2 — Morning Peak with Light Convoy
Convoy:
- class: Light
- from: A
- to: D
- priority: P3
- time_of_day = 08:30

Conditions:
- High traffic expected on A–B, B–C, B–G
- No incidents

Expected Behavior:
- System should avoid B–C (traffic_level=2 during peak).
- Should prefer A → E → F → H → D if ETA is significantly better.

Evaluation:
- Check soft rule: P3 must avoid traffic when possible.

---

# SCENARIO 3 — Landslide on C–D
Convoy:
- class: Medium
- from: B
- to: D
- priority: P2

Incident Applied:
- Landslide (severity 3) on C–D at t=09:40

Expected Result:
- C–D must be removed completely (hard rule).
- Re-route through G → D or F → H → D depending on risk.

Evaluation:
- Routing pipeline must remove segment BEFORE running Dijkstra.
- Confirm flag is_blocked=true and persisted.

---

# SCENARIO 4 — Border Closure at H–D
Convoy:
- class: Heavy
- from: A
- to: D
- priority: P2

Incident:
- Border_Closure at H–D starting t=12:00

Expected:
- Heavy corridor A–E–F–H–D becomes invalid.
- No alternative heavy corridor exists.
- Routing should FAIL with a
