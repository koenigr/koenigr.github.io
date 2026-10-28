---
layout: post
title: "Debugging a Persistent LED Blinking Bug on the STM32F407"
date: 2026-10-28
categories: [embedded, debugging]
---

# Debugging a Persistent LED Blinking Bug on the STM32F407

## Introduction

As part of my bare-metal STM32F407 access control project, I implemented a lockout mechanism to prevent repeated incorrect PIN attempts. After three failed attempts, the system enters a lockout state for 30 seconds, during which the red LED blinks.

During hardware testing, I discovered a bug: the red LED continued blinking even after the lockout period had expired.

This post describes how I investigated the problem using GDB and Serial Wire Output (SWO), identified the root cause, and verified the fix on the target hardware.

## 1. Reproducing the Bug

The expected behaviour was straightforward:

1. Enter an incorrect PIN three times.
2. The system enters the lockout state, and the red LED starts blinking.
3. After 30 seconds, the system returns to the locked state and the red LED stops blinking.
4. Entering a valid PIN unlocks the system, and the green LED turns on.

However, the actual behaviour differed. After the lockout period, the red LED continued blinking. When I entered a valid PIN, the green LED briefly turned on, but the red LED started blinking again.

This suggested that the timeout and the LED output might not be behaving consistently.

## 2. Investigating the Problem

I wanted to determine whether the problem originated in the timekeeping, the state machine, or the hardware output logic.

### Checking the timekeeping

The lockout timeout relies on a millisecond counter incremented by the SysTick interrupt handler.

Using GDB, I checked the counter and the timestamp recorded when the lockout started. Both behaved as expected: the counter advanced, and the lockout start time was set correctly.

This made a problem in the basic timekeeping mechanism less likely.

### Inspecting state transitions

Next, I used GDB to inspect the access controller's state transitions. I set a watchpoint on the state variable to observe when its value changed.

The observations confirmed that the controller transitioned out of the lockout state as expected. I also verified that the corresponding LED methods were called at the expected points.

The application logic appeared to be working correctly. I therefore shifted my attention to the LED output adapter.

### Using SWO to investigate keypad input

I also used Serial Wire Output (SWO) tracing to inspect keypad input handling. The trace confirmed that key presses were detected by the interrupt handlers.

Together, the GDB observations and SWO trace helped me narrow down the investigation. The timeout, state transitions, and keypad input did not explain why the red LED resumed blinking.

## 3. Finding the Root Cause

The problem was in the LED output adapter.

The `BlinkRed()` method enabled blinking by setting a persistent flag named `blinking_`. The adapter's update logic used this flag to determine whether the red LED should continue blinking.

However, the flag was not reset when the controller left the lockout state.

As a result, the application could transition to the unlocked state and turn on the green LED, but the LED adapter still considered blinking to be active. A subsequent update restored the red blinking behaviour.

The important distinction was that the access controller's state was correct, while the LED adapter retained outdated information about the desired output behaviour.

## 4. Implementing the Fix

I corrected the LED output adapter so that the persistent blinking flag is reset when leaving the blinking state.

This ensures that the blinking logic no longer remains active after the lockout period has ended.

The fix addressed the root cause rather than changing the timeout or the state machine, both of which had behaved as expected during the investigation.

## 5. Verifying the Fix

After implementing the change, I repeated the lockout test on the STM32F407 hardware.

The results matched the expected behaviour:

- After three incorrect PIN attempts, the red LED blinked during the lockout.
- After 30 seconds, the red LED stopped blinking.
- Entering a valid PIN produced the expected green LED behaviour.

I also ran the unit tests and verified that the GitHub Actions continuous integration workflow passed.

These checks confirmed that the fix worked on the target hardware and that the existing automated tests remained successful.

## 6. What I Learned

This investigation reinforced the importance of checking assumptions at each layer of an embedded system.

Initially, the persistent blinking appeared to be related to the lockout timeout. By inspecting the millisecond counter, state transitions, and LED method calls, I was able to narrow the problem down to the output adapter.

I also gained practical experience using GDB watchpoints and SWO tracing to investigate the behaviour of firmware running on real hardware.

One important lesson was that correct application state does not necessarily guarantee correct hardware output. Persistent flags and other internal state in peripheral adapters must also be kept consistent with the application's state transitions.

## Conclusion

This bug was caused by a blinking flag that remained set after the lockout state ended. The fix was small, but finding it required tracing the behaviour across several parts of the system.

The investigation gave me practical experience with embedded debugging, from checking interrupt-driven timekeeping to inspecting state transitions and isolating a problem in the hardware output layer.

**Project:** [STM32F407 Access Control System](https://github.com/koenigr/stm32f407_access_control)
