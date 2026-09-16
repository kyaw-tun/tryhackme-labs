# Cyber Defense Frameworks

> TryHackMe — SOC Level 1
> Rooms:
> - Pyramid of Pain
> - Cyber Kill Chain
> - Unified Kill Chain

## Overview

Brief introduction to why cybersecurity frameworks are useful
for understanding, detecting, and disrupting adversary activity.

---

# 1. Pyramid of Pain

## What is the Pyramid of Pain?

...

## The Six Levels

### Hash Values

...

### IP Addresses

...

### Domain Names

...

### Network / Host Artifacts

...

### Tools

...

### Tactics, Techniques & Procedures (TTPs)

...

## From Easy to Difficult

| Level | Indicator | Difficulty for Attacker |
|---|---|---|
| 1 | Hash Values | ... |
| 2 | IP Addresses | ... |
| 3 | Domain Names | ... |
| 4 | Network/Host Artifacts | ... |
| 5 | Tools | ... |
| 6 | TTPs | ... |

## Why the Pyramid Matters to Defenders

...

## My Takeaways

- ...
- ...
- ...

---

# 2. Cyber Kill Chain

## What is the Cyber Kill Chain?

...

## The Seven Stages

### 1. Reconnaissance

...

### 2. Weaponization

...

### 3. Delivery

...

### 4. Exploitation

...

### 5. Installation

...

### 6. Command & Control

...

### 7. Actions on Objectives

...

## Attack Flow

```text
Reconnaissance
      ↓
Weaponization
      ↓
Delivery
      ↓
Exploitation
      ↓
Installation
      ↓
Command & Control
      ↓
Actions on Objectives
```

## Defensive Perspective

| Stage | What the Attacker Does | Possible Defensive Opportunities |
|---|---|---|
| Reconnaissance | ... | ... |
| Weaponization | ... | ... |
| Delivery | ... | ... |
| Exploitation | ... | ... |
| Installation | ... | ... |
| Command & Control | ... | ... |
| Actions on Objectives | ... | ... |

## Limitations

...

## My Takeaways

- ...
- ...
- ...

---

# 3. Unified Kill Chain

## What is the Unified Kill Chain?

...

## Why Was It Created?

...

## The Three Goals

### In — Initial Foothold

...

### Through — Network Propagation

...

### Out — Action on Objectives

...

## The 18 Phases

### In — Initial Foothold

1. Reconnaissance
2. Weaponization
3. Social Engineering
4. Exploitation
5. Persistence
6. Defence Evasion
7. Command & Control
8. Pivoting

### Through — Network Propagation

9. Discovery
10. Privilege Escalation
11. Execution
12. Credential Access
13. Lateral Movement

### Out — Action on Objectives

14. Collection
15. Exfiltration
16. Impact
17. Objectives

> Note: The room groups the phases into three overarching
> goals for learning purposes. The Unified Kill Chain
> contains 18 phases.

## Why the Unified Kill Chain?

...

## Unified Kill Chain vs Cyber Kill Chain

| | Cyber Kill Chain | Unified Kill Chain |
|---|---|---|
| Number of phases | 7 | 18 |
| Focus | ... | ... |
| Internal movement | ... | ... |
| Post-exploitation | ... | ... |
| Relationship | ... | ... |

## My Takeaways

- ...
- ...
- ...

---

# 4. Putting the Frameworks Together

## How They Complement Each Other

### Pyramid of Pain

**Question:** How difficult is it for an attacker to change?

...

### Cyber Kill Chain

**Question:** Where are we in the attack?

...

### Unified Kill Chain

**Question:** What is the broader attack lifecycle and
how does the attacker move through the environment?

...

## Example

```text
Attacker
   │
   ├── Reconnaissance
   │
   ├── Delivery
   │
   ├── Exploitation
   │
   ├── Persistence
   │
   ├── Privilege Escalation
   │
   ├── Lateral Movement
   │
   └── Actions on Objectives
             │
             ▼
       Defensive Detection
             │
             ▼
      Indicators / TTPs
             │
             ▼
       Pyramid of Pain
```

## Conclusion

- ...
- ...
- ...
