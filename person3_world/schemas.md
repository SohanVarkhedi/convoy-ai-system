# SCHEMAS — Core Data Structures for the Convoy AI System
Author: Person 3  
Version: 1.0  
Last Updated: <today’s date>

This document defines the shared data structures used across ML, Backend, and Simulation.

---

## 1. Convoy Object Schema

**Fields:**
- convoy_id (string)
- origin (string, node id)
- destination (string, node id)
- mission_type (string: medical, ammo, supplies, troops, fuel)
- urgency_level (string: P1/P2/P3)
- priority_score (float: 0–100)  // weighted-sum model output
- priority_class (string: P1/P2/P3)
- convoy_class (string: Light / Medium / Heavy)
- weight_tons (float)
- height_m (float)
- width_m (float)
- earliest_start (timestamp or minutes)
- latest_arrival (timestamp or minutes)
- special_flags (array: ["medical", "vip", "hazardous"] optional)
- assigned_truck_ids (array of strings, optional)

Purpose:
Used by backend for routing & rules, and by ML for training priority behaviors.

---

## 2. Segment Object Schema (Road Network Edge)

**Fields:**
- segment_id (string, e.g. "A-B")
- from_node (string)
- to_node (string)
- distance_km (float)
- base_time_min (float)
- traffic_level (int: 0=low, 1=medium, 2=high)
- risk_level (float: 0.0–1.0)
- max_load_tons (float)
- max_width_m (float)
- max_height_m (float)
- is_blocked (bool)
- civil_impact (string: Low/Medium/High)
- weather_sensitivity (float: 0.0–1.0)

Purpose:
Used by routing engine + dynamic re-routing module.

---

## 3. Incident Object Schema

**Fields:**
- incident_id (string)
- type (string: landslide, flood, traffic_jam, border_closure, civil_unrest)
- affected_segment_id (string)
- severity (int: 1–3)
- start_time (int or timestamp)
- duration (optional)
- effects (object describing exact changes):
    - traffic_level_change (int)
    - risk_level_add (float)
    - block_segment (bool)
    - base_time_multiplier (float)

Purpose:
Defines world events that trigger dynamic re-routing.

---

## 4. Load Object Schema (For Fleet Optimization)

**Fields:**
- load_id (string)
- from (node id)
- to (node id)
- weight_tons (float)
- volume_m3 (float, optional)
- load_type (string)
- deadline (timestamp)
- can_mix_with (array of load_type, optional)

---

End of Document.
