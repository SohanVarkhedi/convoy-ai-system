# INCIDENTS CATALOG — Types, Effects, and Usage
Author: Person 3
Version: 1.0
Last Updated: YYYY-MM-DD

This document defines incident types used in scenarios and their exact effects on segment fields.
Person 2 will implement these effects programmatically; Person 1 can use them for synthetic labels.

-------------------------------------------------------------------------------
FORMAT EXPLAINED (effects object):
- traffic_level_change (int): additive change to traffic_level (apply then cap at 2)
- risk_level_add (float): additive to risk_level (apply then cap at 1.0)
- block_segment (bool): set is_blocked = true/false
- base_time_multiplier (float): multiply base_time_min by this factor
- condition: optional textual condition that triggers the incident (e.g., "rain > 0.6")

Severity: 1 (light) / 2 (moderate) / 3 (severe)
-------------------------------------------------------------------------------

## Incident Types (canonical definitions)

1) Traffic_Jam
- Description: sudden civilian or civil convoy congestion
- Typical severity: 1–2
- Effects:
  - traffic_level_change: +1 (severity 1) / +2 (severity 2)
  - risk_level_add: 0.05 * severity
  - block_segment: false
  - base_time_multiplier: 1 + 0.2 * severity

2) Landslide
- Description: ghat/mountain slope collapse blocking road
- Typical severity: 2–3
- Effects:
  - traffic_level_change: +0
  - risk_level_add: 0.4 * severity
  - block_segment: true (severity >=2)
  - base_time_multiplier: if not blocked, *1.5

3) Flooding
- Description: water logging or flash flood that may block roads
- Typical severity: 2–3
- Effects:
  - traffic_level_change: +1
  - risk_level_add: 0.25 * severity
  - block_segment: true if weather_sensitivity >= 0.5 and severity >=2
  - base_time_multiplier: 1 + 0.5 * severity

4) Accident
- Description: vehicle accident causing delay but not necessarily full block
- Typical severity: 1–2
- Effects:
  - traffic_level_change: +1
  - risk_level_add: 0.1 * severity
  - block_segment: true (severity==2 temporarily)
  - base_time_multiplier: 1 + 0.3 * severity

5) Border_Closure
- Description: political/military closure of border or checkpoint
- Typical severity: 3
- Effects:
  - traffic_level_change: +2
  - risk_level_add: 0.5
  - block_segment: true
  - base_time_multiplier: large (infinite / treat as blocked)

6) Civil_Unrest
- Description: protests, strife causing unpredictable disruptions
- Typical severity: 2–3
- Effects:
  - traffic_level_change: +1 or +2
  - risk_level_add: 0.3 * severity
  - block_segment: possible (severity 3)
  - base_time_multiplier: 1 + 0.4 * severity

7) Roadwork_Close (planned)
- Description: scheduled maintenance, known in advance
- Typical severity: 1
- Effects:
  - traffic_level_change: +1
  - risk_level_add: 0.05
  - block_segment: false (usually lanes reduced)
  - base_time_multiplier: 1.2

8) Weather_Event_Extreme (heavy storm)
- Description: extreme weather that may cause multiple segment effects
- Typical severity: 3
- Effects:
  - traffic_level_change: +2
  - risk_level_add: 0.5
  - block_segment: possible for high-sensitivity segments
  - base_time_multiplier: 1 + 0.6

-------------------------------------------------------------------------------
USAGE GUIDELINES for Person 2 (backend)
- When applying an incident:
  1. Fetch the segment by id.
  2. Apply `traffic_level += traffic_level_change`, then clamp to {0,1,2}.
  3. Apply `risk_level = min(1.0, risk_level + risk_level_add)`.
  4. If `block_segment == true` → set `is_blocked = true`.
  5. Apply `base_time_min *= base_time_multiplier`.
  6. Record a deterministic log entry including previous/after values.
- If an incident has a `condition` (e.g., weather sensitivity), check that before applying.
- Incidents have durations; backend should revert effects after duration ends (or simulate cleanup).

-------------------------------------------------------------------------------
EXAMPLE LOG ENTRY (for audit)
{
  "incident_id": "INC-001",
  "type": "landslide",
  "affected_segment_id": "B-C",
  "severity": 3,
  "applied_effects": {
    "traffic_level_before": 2,
    "traffic_level_after": 2,
    "risk_level_before": 0.45,
    "risk_level_after": 1.0,
    "is_blocked_before": false,
    "is_blocked_after": true,
    "base_time_min_before": 22,
    "base_time_min_after": null (blocked)
  }
}

-------------------------------------------------------------------------------
End of incidents_catalog.md
