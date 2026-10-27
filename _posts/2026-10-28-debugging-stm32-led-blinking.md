---
layout: post
title: "Debugging a Persistent LED Blinking Bug on the STM32F407"
date: 2026-10-28
categories: [embedded, debugging]
---

---
layout: post
title: "Debugging a Persistent LED Blinking Bug on the STM32F407"
date: 2026-10-28
categories: [embedded, debugging]
---

# Debugging a Persistent LED Blinking Bug on the STM32F407

## 1. Introduction
**Purpose:** Briefly introduce the project and the problem.

- What is the STM32 access control system?
- What should happen after three incorrect PIN attempts?
- What unexpected behaviour did you observe?

## 2. Reproducing the Bug
**Purpose:** Describe the problem so that another developer could reproduce it.

- What steps trigger the bug?
- What should happen after the 30-second lockout?
- What actually happens?
- Why is this behaviour incorrect?

## 3. Investigating the Problem
**Purpose:** Explain how you approached the investigation.

### 3.1 Checking the Timekeeping
- How did you check whether SysTick was working correctly?
- What did you observe about the millisecond counter?

### 3.2 Inspecting the State Machine
- How did you use GDB to inspect state transitions?
- What did you learn about the lockout logic?

### 3.3 Tracing the Hardware Output
- How did you use SWO tracing and GDB to investigate the LED behaviour?
- Which part of the system did you investigate next, and why?

## 4. Finding the Root Cause
**Purpose:** Explain the actual cause, not just the visible symptom.

- Which variable caused the problem?
- Why did the blinking continue after the lockout ended?
- Why did entering a valid PIN briefly turn on the green LED before the red LED resumed blinking?

Include a screenshot of the relevant original code.

## 5. Implementing the Fix
**Purpose:** Explain what you changed and why it solves the problem.

- When should the `blinking_` flag be reset?
- How does the change prevent the unexpected behaviour?
- Link to the commit containing the fix.

Include a screenshot of the corrected code if it helps illustrate the change.

## 6. Verifying the Fix
**Purpose:** Demonstrate that the fix works.

- How did you test the fix on the actual STM32 hardware?
- What happened after the 30-second timeout?
- What happened when you entered a valid PIN?
- Did the unit tests pass?
- Did GitHub Actions complete successfully?

## 7. What I Learned
**Purpose:** Reflect on the debugging process.

- What did you learn about debugging embedded systems?
- What did you learn about using GDB, watchpoints, or SWO?
- Did anything surprise you?
- What would you do differently next time?

## 8. Conclusion
Briefly summarise the bug, the root cause, and the fix.

End with a link to the STM32 project repository and, optionally, the detailed debugging protocol.
