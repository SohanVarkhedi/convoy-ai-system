# TUNING LOG — Cost Function, Weights, Threshold Adjustments
Author: Person 3
Version: 1.0
Last Updated: YYYY-MM-DD

This file tracks tuning of:
- Weighted sum model (priority engine)
- Routing cost weights (time/traffic/risk)
- Risk thresholds
- ETA reroute thresholds
- Interpretability notes for ML Person 1 & Backend Person 2

---

# BASE COST FUNCTION (Initial Version)
EdgeCost = 
  (a * time_cost) +
  (b * traffic_cost) +
  (c * risk_cost)

Initial default weights:
a = 1.0  
b = 3.0  
c = 4.0  

Reason:
Traffic and risk should dominate route choice more than travel time.

---

# PRIORITY ENGINE WEIGHTED SUM (Initial)
priority_score =
  (wu * urgency) +
  (wm * mission_type) +
  (wr * risk_zone) -
  (wc * civil_impact) +
  special_bonus

Initial weights:
wu = 0.50  
wm = 0.25  
wr = 0.20  
wc = 0.05  

special_bonus:
+30 for medical  
+20 for ammo/fuel  
+10 for troop rotation

---

# RISK THRESHOLDS (Initial)
risk_soft_limit_P3 = 0.70  
risk_soft_limit_P2 = 0.80  
risk_hard_limit = 0.95  

Notes:
• risk > hard_limit is forbidden for all except emergency medical P1  
• P3 penalized earliest, P1 latest  

---

# REROUTE TRIGGER (Initial)
Recompute route if:
new_ETA < old_ETA * 0.80  
(At least 20% improvement or safety-risk improvement)

Safety-based reroute:
If risk_level reduced by 0.15 on alternative path → reroute even if ETA same.

---

# TUNING SESSIONS

## Session T1 — (Date)
Goal: test morning traffic effects  
Outcome:
• b increased from 2.5 → 3.0  
• Improves avoidance of B–C and B–G at 08:00  
Notes:
• Still acceptable for P1 missions.

## Session T2 — (Date)
Goal: adjust risk penalties  
Outcome:
• c increased from 3.0 → 4.0  
• Medium and P3 convoys now avoid high-risk segments reliably.

## Session T3 — (Date)
Goal: improve dynamic rerouting sensitivity  
Outcome:
• Reroute threshold changed from 30% → 20% ETA improvement  
Reason: More responsive in chaos scenarios.

---

# NOTES FOR PERSON 1 (ML)
• When generating synthetic data, use traffic/risk values AFTER applying incident effects.  
• Include convoy_class and priority_class as features only in the output label, NOT input features.  
• risk_level >= hard_limit must never appear in training as allowed.  

---

# NOTES FOR PERSON 2 (BACKEND)
• Log every segment removal with cause to debug routing.  
• Apply hard rules BEFORE cost-based routing.  
• Apply soft rules by adjusting weight penalties, not by removing edges.  
• Store before/after ETA for reroute decisions.  

End of tuning_log.md
