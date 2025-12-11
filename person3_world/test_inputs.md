# TEST_INPUTS — Example Convoy Objects for Quick Backend Testing
Author: Person 3
Version: 1.0
Last Updated: YYYY-MM-DD

Paste these JSON objects (one per line or separate files) to quickly test the routing engine and priority ordering.

1) Heavy supply convoy (expected heavy corridor)
{
  "convoy_id": "TST-H-001",
  "origin": "A",
  "destination": "D",
  "mission_type": "supplies",
  "urgency_level": "P2",
  "priority_score": 68.0,
  "priority_class": "P2",
  "convoy_class": "Heavy",
  "weight_tons": 22.0,
  "height_m": 3.2,
  "width_m": 3.5,
  "earliest_start": "2025-12-20T04:00:00Z",
  "latest_arrival": "2025-12-20T12:00:00Z",
  "special_flags": []
}

2) Medical emergency (high priority)
{
  "convoy_id": "TST-M-001",
  "origin": "A",
  "destination": "D",
  "mission_type": "medical",
  "urgency_level": "P1",
  "priority_score": 92.0,
  "priority_class": "P1",
  "convoy_class": "Light",
  "weight_tons": 5.0,
  "height_m": 2.5,
  "width_m": 2.2,
  "earliest_start": "2025-12-20T05:00:00Z",
  "latest_arrival": "2025-12-20T08:00:00Z",
  "special_flags": ["medical"]
}

3) Routine supplies (low)
{
  "convoy_id": "TST-R-001",
  "origin": "B",
  "destination": "D",
  "mission_type": "supplies",
  "urgency_level": "P3",
  "priority_score": 38.0,
  "priority_class": "P3",
  "convoy_class": "Light",
  "weight_tons": 6.0,
  "height_m": 2.8,
  "width_m": 2.5,
  "earliest_start": "2025-12-20T10:00:00Z",
  "latest_arrival": "2025-12-20T18:00:00Z",
  "special_flags": []
}

4) Mixed cargo (compatible load test)
{
  "convoy_id": "TST-COMP-001",
  "origin": "E",
  "destination": "F",
  "mission_type": "ammo",
  "urgency_level": "P2",
  "priority_score": 75.0,
  "priority_class": "P1",
  "convoy_class": "Medium",
  "weight_tons": 9.0,
  "height_m": 3.0,
  "width_m": 3.0,
  "earliest_start": "2025-12-20T06:00:00Z",
  "latest_arrival": "2025-12-20T10:00:00Z",
  "special_flags": ["hazardous"]
}

5) Heavy but overweight (should be blocked on narrow segments)
{
  "convoy_id": "TST-H-OVER-001",
  "origin": "A",
  "destination": "D",
  "mission_type": "fuel",
  "urgency_level": "P2",
  "priority_score": 70.0,
  "priority_class": "P2",
  "convoy_class": "Heavy",
  "weight_tons": 85.0,
  "height_m": 4.4,
  "width_m": 4.9,
  "earliest_start": "2025-12-20T02:00:00Z",
  "latest_arrival": "2025-12-20T14:00:00Z",
  "special_flags": []
}

Notes:
- Adjust timestamps to your local demo time or use relative times.
- Person 2 can paste these JSONs into an API endpoint like `/create_convoy` to test behavior quickly.
