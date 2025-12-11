# graph_readme.md

This file explains the sample graph (graph.json) used for simulation and routing.

Nodes:
- A: Main Base (logistics hub)
- B: Major junction near a city (high civilian impact on some segments)
- C: Checkpoint / local admin post
- D: Border crossing / forward boundary
- E: Supply Depot (large trucks allowed)
- F: Junction - connects central corridor to forward base
- G: Town (civilian area, high civil impact)
- H: Forward Base (destination for many convoys)

Allowed convoy classes:
- "Light", "Medium", "Heavy" indicate which convoy types the segment supports.
- Heavy convoys are allowed on the main logistics corridor: A → E → F → H → D
- City/town segments (e.g., F→G, B→C, B→G) are restricted to Light or Medium only.

Graph notes:
- The graph includes alternate corridors: A→E→F→H→D and A→B→C→D (via C) and A→B→G→D.
- Some segments have low max_load_tons (e.g., F→G, B→G) to force heavy convoys to choose alternate routes.
- traffic_level: 0=low, 1=medium, 2=high (used for cost penalties).
- risk_level: 0.0–1.0 (used by routing + ML risk model).
- weather_sensitivity: 0.0–1.0 used by incident effects (higher means that rain/flooding increases disruption likelihood).

Usage:
- Person 2 will load this JSON as the road graph for routing.
- Person 2 must enforce `allowed_convoy_classes` when filtering edges for a convoy.
- Person 3 will use this graph to place incidents in scenarios.
- Person 1 can use the segment fields as features for synthetic data generation.

