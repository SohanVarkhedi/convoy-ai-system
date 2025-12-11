# RULES DOCUMENT — Hard Constraints & Safety Logic for Routing & Re-Routing
Author: Person 3
Version: 1.0
Last Updated: YYYY-MM-DD

This document defines all mandatory (“hard”) rules and priority-dependent (“soft”) rules that the AI Transport System must enforce. Hard rules override ML or weighted-sum decisions. Backend (Person 2) must use this file to implement routing filters and dynamic re-routing logic.

# DEFINITIONS
Convoy Classes: Light, Medium, Heavy  
Priority Classes:  
• P1 = Critical  
• P2 = High  
• P3 = Routine  

Segment Fields Used: max_load_tons, max_width_m, max_height_m, allowed_convoy_classes, risk_level, civil_impact, is_blocked, traffic_level, weather_sensitivity.

# HARD RULES (Non-Negotiable — Cannot Be Overridden)
1. Load Capacity Rule  
If convoy.weight_tons > segment.max_load_tons → segment is forbidden.

2. Height Clearance Rule  
If convoy.height_m > segment.max_height_m → segment is forbidden.

3. Width Clearance Rule  
If convoy.width_m > segment.max_width_m → segment is forbidden.

4. Convoy-Class Restriction Rule  
If convoy.convoy_class NOT IN segment.allowed_convoy_classes → segment is forbidden.

5. Blocked Segment Rule  
If segment.is_blocked == true → segment is forbidden.

6. Border Closure Rule  
If incident.type == Border_Closure on a segment → all convoys denied.

7. Extreme Weather Rule  
If Weather_Event_Extreme affects a segment AND weather_sensitivity ≥ 0.5 → segment forbidden.

8. High Risk Cutoff Rule  
If segment.risk_level ≥ 0.95 → all convoys forbidden except emergency medical P1.

9. Fragile Road Rule  
Heavy convoys cannot use segments whose allowed_convoy_classes exclude “Heavy”.

10. Civil Area Rule for Heavy Convoys  
If civil_impact == High AND convoy_class == Heavy → forbidden.

11. No-Through Rule  
If all outgoing segments at a node violate hard rules → routing fails.

12. Landslide Severity Rule  
If incident.type == Landslide AND severity ≥ 3 → this segment is removed from graph until reset.

# SOFT RULES (Priority Dependent — Can Be Overridden by P1/P2)
1. Max Traffic Soft Limit  
If traffic_level == 2 AND priority_class == P3 → avoid if alternative exists.

2. Medium Risk Detour Rule  
If risk_level ≥ 0.7 AND priority_class == P3 → avoid.  
If risk_level ≥ 0.8 AND priority_class == P2 → avoid if possible.  
P1 convoys allowed.

3. Civil Impact Rule for P2 Convoys  
If civil_impact == High AND priority_class == P2 → avoid unless no alternative.

4. Weather Moderate Rule  
If rain event active AND weather_sensitivity ≥ 0.5 → apply travel penalty but segment allowed unless hard rule triggered.

5. ETA Preference Rule  
If alternative route reduces ETA by >20% → prefer alternative (unless heavy convoy constraints prevent it).

6. Congestion Avoidance  
If traffic_level == 2 for more than 2 consecutive segments → reroute if a safer corridor exists.

# PRIORITY OVERRIDE MATRIX
Hard rules cannot be overridden by priority. Soft rules can.

Condition / Field | P1 | P2 | P3  
----------------------------------------  
High traffic | Allowed | Allowed | Avoid  
Risk < 0.8 | Allowed | Allowed | Avoid  
Risk ≥ 0.8 | Allowed | Avoid | Forbidden  
Civil impact High | Allowed | Avoid | Avoid  
Blocked segment | Forbidden | Forbidden | Forbidden  
Load/height/width violations | Forbidden | Forbidden | Forbidden  

# ROUTING PIPELINE (Backend Must Follow)
1. Start with full graph.json.  
2. Apply all HARD RULES → remove forbidden segments.  
3. Score remaining segments using weighted cost (time + traffic + risk).  
4. Apply SOFT RULES → adjust weights or penalties.  
5. Run Dijkstra/A* to compute final route.  
6. If no valid path exists → return routing failure.  
7. Dynamic re-routing uses same pipeline starting from current node.

# EXAMPLE SEGMENT REJECTIONS
Example 1 — Heavy convoy blocked on F-G  
Segment allowed_convoy_classes=["Light"].  
→ Removed by Hard Rule 4.

Example 2 — Risk > 0.95 on G-D  
risk_level=0.98.  
→ Hard Rule 8 blocks it even for P1.

Example 3 — P3 avoiding B-G during peak  
traffic_level=2.  
→ Soft Rule 1: avoid if possible.

# BACKEND IMPLEMENTATION NOTES
Person 2 must implement:

Function:
apply_hard_rules(convoy, segment) → allowed/forbidden

Function:
apply_soft_rules(convoy, segment) → weight adjustment

Dynamic re-routing:  
• After each incident update, rebuild allowed graph  
• Re-run routing pipeline  
• If new route ETA improves by >20%, propose reroute

All rule violations must be logged with before/after values for transparency.

End of Document.
