# Comic-Con India — Multi-Zone Event Parking System
### Database Design (ER Diagram)

---

## What This Is

This is a database design submission for the Comic-Con India parking management system. The goal was to model a real-world, multi-zone event parking facility — not just a simple entry-exit tracker — that handles thousands of visitors arriving across multiple days via different vehicle types, with reserved parking categories for VIPs, exhibitors, staff, cosplayers with props, and EV charging vehicles.

No SQL queries. No frontend. No backend. Just the database design.

---

## The ER Diagram

The diagram was built using Mermaid.js `erDiagram` syntax and covers 9 entities with clearly marked primary keys (PK), foreign keys (FK), and cardinality relationships.

---

## Entities Overview

| Entity | Purpose |
|---|---|
| `VEHICLE_CATEGORY` | Lookup table for vehicle types — Bike, Car, SUV, EV, etc. |
| `VEHICLE` | Stores each real vehicle (license plate, owner, contact) |
| `PARKING_ZONE` | Top-level zones inside the venue (Zone A, Zone B, etc.) |
| `PARKING_LEVEL` | Levels within each zone (Level 1, Level 2, etc.) |
| `SPOT_CATEGORY` | Types of spots — General, VIP, Exhibitor, Staff, EV Charging |
| `PARKING_SPOT` | Individual parking spots with availability tracking |
| `PARKING_SESSION` | One row per parking visit (entry time, exit time, duration) |
| `PARKING_TICKET` | Ticket issued per session with charge calculation |
| `PAYMENT` | Payment record linked to each ticket |

---

## Key Relationships

```
VEHICLE_CATEGORY  ──< VEHICLE
VEHICLE           ──< PARKING_SESSION
PARKING_ZONE      ──< PARKING_LEVEL
PARKING_LEVEL     ──< PARKING_SPOT
SPOT_CATEGORY     ──< PARKING_SPOT
PARKING_SPOT      ──< PARKING_SESSION
PARKING_SESSION   ──  PARKING_TICKET   (one-to-one)
PARKING_TICKET    ──o  PAYMENT         (optional until paid)
```

---

## Design Decisions (Why I Made These Choices)

**Why separate `VEHICLE` from `VEHICLE_CATEGORY`?**
So the vehicle type (Bike, SUV, EV) is stored once as a lookup row and referenced by FK — not repeated as a string in every vehicle record. Easier to filter and update.

**Why separate `PARKING_SESSION` from `PARKING_TICKET`?**
A session tracks the physical activity — when the vehicle entered and exited, which spot it used. A ticket is the financial document generated from that session. Keeping them separate lets you reissue or regenerate a ticket without touching session records.

**How does one vehicle visit multiple times?**
`VEHICLE → PARKING_SESSION` is one-to-many. Each visit creates a new session row. The vehicle row stays the same.

**How is one spot reused across multiple visits?**
`PARKING_SPOT → PARKING_SESSION` is also one-to-many. The same spot can appear in many session rows over multiple days — each session row is a point-in-time record.

**How do we track which vehicles are currently parked?**
Filter `PARKING_SESSION` where `exit_time IS NULL`. Those are active sessions.

**How are reserved spots handled?**
`PARKING_SPOT` has an `is_reserved` boolean and links to `SPOT_CATEGORY`, which has an `access_type` field (VIP, Exhibitor, Staff, EV Charging, General). A query can find all reserved EV spots or all VIP spots easily.

**How is parking availability tracked?**
`PARKING_SPOT.is_available` is a boolean updated on entry (set false) and exit (set true). Combined with `PARKING_SESSION` records, you can always see current status and historical usage.

**How is the charge calculated?**
`PARKING_SESSION` stores `duration_minutes`. `PARKING_TICKET` stores `base_rate` (per hour) and `total_charge` (computed from duration × rate). Different spot categories can have different rates.

---

## What Questions This Design Can Answer

- What vehicles entered the parking facility? → Query `VEHICLE` joined with `PARKING_SESSION`
- What type of vehicle entered? → Join `VEHICLE` with `VEHICLE_CATEGORY`
- Which parking spot was assigned? → `PARKING_SESSION.spot_id` → `PARKING_SPOT`
- Which zone or level does that spot belong to? → `PARKING_SPOT` → `PARKING_LEVEL` → `PARKING_ZONE`
- Was the spot reserved? → `PARKING_SPOT.is_reserved` + `SPOT_CATEGORY.access_type`
- When did the vehicle enter and exit? → `PARKING_SESSION.entry_time`, `exit_time`
- What ticket was issued? → `PARKING_SESSION` → `PARKING_TICKET`
- Can one vehicle visit multiple times? → Yes, one-to-many relationship
- Can one spot be reused? → Yes, one-to-many relationship
- How is payment recorded? → `PAYMENT` linked to `PARKING_TICKET`
- Which vehicles are currently parked? → Sessions where `exit_time IS NULL`
- Can special categories be represented? → Yes, via `SPOT_CATEGORY.access_type`

---


---

## File Structure

```
/
├── README.md           ← this file
└── erd_diagram.png     ← exported ER diagram image (if submitted separately)
```

---


