# PLC Traffic Light Controller

A traffic light sequencer built in **TIA Portal**, on an S7-1212C — twice, on purpose. The first version is plain ladder logic. The second is the exact same behavior rebuilt as an SCL function block, written specifically to prove a subtle but important fact about how PLCs execute code: **the order you write two lines in can change what value they see**, even within a single scan.

This was the first project in a self-study PLC/SCADA track, coming from an embedded systems background (STM32, FreeRTOS, C/C++).

---

## What it does

A standard three-light sequence, looping forever:

**RED (5s) → GREEN (5s) → YELLOW (2s) → RED (5s) → ...**

## Build 1 — Ladder logic

A chain of three `TON` timers, each one gating the next:
- A seal-in circuit (`Start` OR `Cycle_Run`, latched, cleared by `Cycle_Done`) starts the cycle.
- Timer 1 (5s) drives the RED output and, on completion, starts Timer 2.
- Timer 2 (5s) drives GREEN and starts Timer 3.
- Timer 3 (2s) drives YELLOW and, on completion, sets `Cycle_Done` — ending the cycle and resetting the seal-in.

This is the standard ladder-logic pattern for timed sequencing: chained timers, each one's `.Q` output gating the next.

## Build 2 — SCL function block (the actual point of this repo)

The same traffic light, rebuilt from scratch as a function block — not because the ladder version was wrong, but to deliberately expose two things that don't show up clearly in ladder:

- **FC vs. FB distinction** — why this needed to be a Function Block (with persistent `Static` state for the timer instances) rather than a stateless Function.
- **Same-scan read/write ordering** — proven experimentally, not just read about. By deliberately ordering networks so that one read sat *before* a write and another sat *after* it, the rebuild showed directly that code above a write in the same scan still sees the old value, while code below that same write sees the new one. That's a foundational fact about how a single PLC scan actually executes, and it's easy to get wrong by intuition alone if you're coming from an environment where statement order doesn't carry that kind of consequence.

## Project structure

```
/src
  TrafficLight_Ladder.scl   — placeholder/export of the original ladder networks
  TrafficLight_FB.scl        — the SCL function block rebuild
/docs
  scan-cycle-notes.md         — write-up of the read/write ordering experiment
```

## Built with

- Siemens TIA Portal (Ladder Logic + SCL)
- S7-1212C (simulated via PLCSIM)

## Background

This is one of three self-study PLC projects — alongside a [conveyor sorting state machine](#) (the first project with real sensor-driven branching) and a [batch-mixing process capstone](#) (PID control, UDTs, and a full safety-fault layer) — built while teaching myself industrial control fundamentals from the ground up, with a particular focus on where PLC scan-cycle thinking diverges from RTOS/embedded thinking rather than just where the two overlap.

## Status

Both builds compile clean with zero errors. Live force-testing in PLCSIM is the one remaining step before calling either fully verified end-to-end.
