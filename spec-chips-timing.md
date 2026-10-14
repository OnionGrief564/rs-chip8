## Problem Statement

The CHIP-8 emulator instruction timing uses hardcoded microsecond values without reference to the original CHIP-8 CPU specification. The original CHIP-8 interpreter ran at ~500 Hz (500 instructions/second) on the COSMAC VIP with 60 Hz timer ticks. The current code has fixed `Duration::from_micros()` values per instruction that don't align with the spec's instruction-cycle-based timing.

## Solution

Update the timing comments in `chip8/src/instruction.rs` to document how each instruction's duration relates to the CHIP-8 CPU speed (~500-1000 IPS / ~2ms per instruction average, with 60 Hz timer ticks at 16.67ms). No code changes — only comments.

## User Stories

1. As a developer, I want comments explaining the CHIP-8 timing spec, so that I can understand why values like 22734 µs exist.
2. As a maintainer, I want the 60 Hz timer reference noted, so that delay/sound timer behavior is traceable.
3. As an emulator author, I want instruction-cycle references in comments, so that timing can be adjusted to match original hardware.

## Implementation Decisions

- Modify: `chip8/src/instruction.rs` — add specs-based comments only, no logic changes
- Reference: CHIP-8 spec — original interpreter ~500 IPS (~2ms/inst), 60 Hz timer (16.67ms/frame)
- Leave `FRAME_DURATION = Duration::from_micros(16666)` as-is (matches 60 Hz)

## Testing Decisions

- No code tests needed (comment-only change)
- Verify comments reference CPU speed (~500-1000 IPS), timer (60 Hz), and instruction cycles

## Out of Scope

- Changing any `Duration` values
- Changing `FRAME_DURATION`
- Adding timer tick logic
- Changing instruction execution logic

## Further Notes

Per CHIP-8 spec: CPU executes ~500 instructions/second. Frame duration (16666 µs = 60 Hz) should contain roughly 8-10 instructions at that speed. The `run_frame()` loop runs until `remaining == 0`, which is approximately 60 instructions at 16666 µs each — faster than original. Comments should note this discrepancy.

## Specification

Based on CHIP-8 specification (RCA 1802 CPU in COSMAC VIP):
- Original CPU speed: ~500 Hz (500 instructions per second)
- Timer tick: 60 Hz (16.67 ms per frame)
- Instruction cycle time: ~2 ms (2000 µs) per instruction on average
- Each instruction typically takes 1-2 cycles

The `Duration::from_micros()` values in `execute()` function should be documented to reflect:
- How many instruction cycles they represent
- How they relate to the 60 Hz timer frame budget
- How they compare to the original hardware timing