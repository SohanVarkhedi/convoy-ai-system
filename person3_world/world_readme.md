# WORLD_README — Overview of Person 3 Deliverables and Usage
Author: Person 3
Version: 1.0
Last Updated: YYYY-MM-DD

Purpose
This document explains what Person 3 owns, how other teammates should use the files, and quick startup instructions for running scenarios and tests.

What Person 3 provides (folder: person3_world/)
- schemas.md                 → shared JSON schema for convoy, segment, incident, load
- graph.json                 → road graph (nodes + segments) with allowed_convoy_classes
- graph_readme.md            → explanation of nodes / corridors
- profiles.md                → traffic & weather profiles (time-of-day and seasons)
- incidents_catalog.md       → incident types and their exact effects
- sample_incidents.json      → ready incidents for scenarios
- rules_doc.md               → hard & soft rules to enforce
- scenario_pack.md           → 8 demo scenarios (step-by-step)
- tuning_log.md              → cost weights, thresholds, and tuning history
- validation_matrix.md       → (see file) maps scenarios to rules & tests
- test_inputs.md             → sample convoy objects to feed backend for testing

How teammates should use these files
- Person 2 (Backend): Load `graph.json` as the road network. Implement rule enforcement from `rules_doc.md`. Use `profiles.md` + `sample_incidents.json` to simulate time and incidents. Run scenarios from `scenario_pack.md` and assert behavior per `validation_matrix.md`. Use `test_inputs.md` as quick cargo/convoy payloads for endpoint testing.
- Person 1 (ML): Use `profiles.md` + `graph.json` + `sample_incidents.json` to generate synthetic training rows. Respect `rules_doc.md` to avoid illegal training examples (e.g., no training samples where hard rules would forbid passage). Use `tuning_log.md` for initial weights and thresholds.

Quick start (for demo)
1. Person 2: start backend simulation (server.py) that can:
   - load graph.json
   - apply profiles.md time-of-day mapping
   - apply incidents from sample_incidents.json
   - accept test inputs from test_inputs.md
2. Person 2: run scenario 1 from `scenario_pack.md`
3. Person 1: use test_inputs to verify priority outputs match the weighted-sum expectation
4. Record results, update `tuning_log.md` if thresholds require tuning

Repository conventions & tips
- All files in `person3_world/` are the canonical single-source-of-truth for the world. Do not edit them in the backend or ML folders — instead update here and commit.
- Use deterministic seeds for any random events in scenarios to ensure reproducible demos.
- Include an example command in backend README to run one scenario (e.g., `python server.py --scenario SCENARIO_3`).

Contact
If anyone is unclear about a rule or scenario, ask Person 3. The world model must remain consistent across modules.
