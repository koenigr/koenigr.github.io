---
layout: post
title: "Debugging an STM32: My First Hands-On Experience with GDB and SWO"
date: 2026-10-28
categories: [embedded, debugging]
---

## From Symptoms to Root Cause

While developing a bare-metal access control system for the STM32F407G-DISC1, I encountered a bug that seemed straightforward at first.

After three incorrect PIN attempts, the system entered a 30-second lockout period, during which the red LED blinked. Once the lockout expired, however, the LED continued blinking — even though the system had returned to its locked state.

I wanted to understand why, rather than simply patching the code. Although AI could have helped me find the problem, I decided to use the opportunity to practise debugging on real hardware.

![Debugging process: From symptom to root cause](https://koenigr.github.io/assets/images/investigation_steps.png)

## Investigating with GDB and SWO

I approached the problem systematically, checking whether the fault originated in the timing mechanism, the state machine, or the hardware output logic.

**First, I used GDB to inspect the timing and state transitions.**

I verified that the millisecond counter advanced correctly and that the lockout start timestamp was set as expected. I then placed a watchpoint on the state variable to observe when the controller changed states.

The state transitions were correct. The controller left the lockout state when the timeout expired, and the expected LED methods were called.

**Next, I used SWO tracing to investigate keypad interrupt handling.**

The trace output showed that keypad events were being detected. This helped me rule out the input handling as the source of the problem.

By combining these observations, I narrowed the investigation down to the LED output adapter.

## Finding the Bug

The problem was not in the state machine or the timer.

The LED adapter maintained a `blinking_` flag to control the blinking behavior. When blinking was activated, the flag was set — but it was not cleared when the system left the lockout state.

Consequently, the adapter continued blinking the red LED even after the controller had returned to normal operation.

The fix was simple: reset the flag when leaving the blinking state.

## Verification

After implementing the fix, I tested the behavior on the actual STM32 hardware. The red LED stopped blinking after the lockout expired, and entering a valid PIN produced the expected green LED indication.

The unit tests passed, and the project’s GitHub Actions CI pipeline completed successfully.

For the complete investigation, see the
[STM32 debugging protocol](https://github.com/koenigr/stm32f407_access_control/blob/main/Docs/Debugging/debugging-protocol.md).

## What I Learned

This investigation reinforced an important lesson: **correct state transitions do not necessarily mean that the hardware output reflects the current state correctly.**

It also gave me practical experience combining GDB and SWO to investigate a problem on a microcontroller. GDB helped me inspect program state and trace transitions, while SWO provided runtime information about interrupt handling.

For me, the most valuable part was not the complexity of the bug itself, but learning to use these tools to move from an observed symptom to a verified root cause.

It was a small bug, but a useful step forward in my embedded development journey.
