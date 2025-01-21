---
arc: 4
title: Flagged Operations
authors: bendyarm, d0cd, acoglio
discussion: https://github.com/AleoHQ/ARCs/discussions/63
topic: # Choose: Protocol, Network, or Application
status: Draft
created: # Date
---

## Abstract

Some Aleo instructions can halt when passed certain arguments.
This halting behavior makes those instructions unusable
by a program that wants to try something else if a halt is detected.

For example, if a Leo program contains a conditional not in a
finalize, both branches of the conditional are executed in the
circuit, so if a branch not taken executes an instruction that halts,
the whole program will incorrectly halt.  Replacing the halting
instruction by one that returns an error flag will allow such a
program to compile correctly.

This ARC proposes to add new opcodes for flagged operations
corresponding to the Aleo Instructions that can halt.

## Specification

Each flagged operation is identical to its corresponding current operation
when called with arguments that would be nonhalting, except it returns
a second return value of type boolean that is **false** (meaning that the
current operation would not have halted).  When called with arguments
that would have been halting, it generally returns 0 (of the appropriate
output type) and **true** ("would have halted") for the second return value.

The flagged operations are different from wrapped (e.g. `abs.w`) or
lossy (e.g. `cast.lossy`) operations.  It is important that the flagged
operation have the same semantics as the current halting instruction
except for the halting behavior and extra return value, for ease of
use by compilers.

| Current Halting Opcode | New Flagged Opcode | notes |
|:-------------------:|:-----------------------:|:--------------------------:|
| abs | abs.flagged |
| add | add.flagged |
| assert.eq | is.eq | We don't need a separate assert.(n)eq.flagged;
| assert.neq | is.neq | instead, just use the existing is.(n)eq
| cast | cast.flagged | as of now, docs don't say what causes cast to halt
| commit.bhp256 | commit.bhp256.flagged |
| commit.bhp512 | commit.bhp512.flagged |
| commit.bhp768 | commit.bhp768.flagged |
| commit.bhp768 | commit.bhp1024.flagged |
| commit.ped64 | commit.ped64.flagged |
| commit.ped128 | commit.ped128.flagged |
| div | div.flagged | div can halt on underflow and divide by zero
| div.w | div.w.flagged | div.w can halt on divide by zero
| hash.bhp256 | hash.bhp256.flagged |
| hash.bhp512 | hash.bhp512.flagged |
| hash.bhp768 | hash.bhp768.flagged |
| hash.bhp1024 | hash.bhp1024.flagged |
| hash.ped64 | hash.ped64.flagged |
| hash.ped128 | hash.ped128.flagged |
| inv | inv.flagged |
| mod | mod.flagged |
| mul | mul.flagged |
| neg | neg.flagged |
| pow | pow.flagged |
| rem | rem.flagged | rem can halt on underflow and divide by zero
| rem.w | rem.w.flagged | rem.w can halt on divide by zero
| shl | shl.flagged | as of now, docs don't say what causes
| shr | shr.flagged | shl or shr to halt
| sqrt | sqrt.flagged |
| sub | sub.flagged |

### Test Cases

This section should introduce any and all critical test cases that need to be considered for the specification.

The following note applies to all tests of Aleo Instructions.  For all tests, both literal constant arguments
and variable arguments should be tested.  If there are two arguments, we need to test all four
combinations.  This is because the Aleo Instructions compiler generates more optimized code for literal
constant arguments, and the circuits can differ substantially.

One or more arguments that causes halting for a current halting opcode should be used as an input
for the equivalent new flagged opcode to make sure it doesn't halt.  

An assortment of arguments that do not cause halting for a current halting opcode should be
used as input to both halting and flagged operations to make sure they return the same value
(other than the halting flag, of course).

## Dependencies

The main change is to the snarkVM repository.

The Aleo Explorer will need to be updated to output the new operations when displaying an aleo program.

The "sdk" repository may need to be changed to show syntax highlighting on the new opcodes.

After this change, the Leo compiler can more easily be fixed to prevent the [conditional halting bug.](<https://github.com/AleoHQ/leo/issues/27482>)

### Backwards Compatibility

This change is strictly additive.

## Security & Compliance

This change does not affect security or regulatory issues.

## References

Leo bug: 
https://github.com/AleoHQ/leo/issues/27482
