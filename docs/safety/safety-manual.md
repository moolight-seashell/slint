
# Slint SC Safety Manual
Slint SC is the “ISO 26262 compliant subset” of Slint. 

## Purpose of Documents

This document contains a Safety Manual and a Qualification Plan.

The Safety Manual describes slint-compiler, its library components, and how to use them safely. 

The Qualification Plan contains the requirements/constraint IDs, a list of safety-critical tests, info about which tests cause known failures, with fixed-in version#, and github issue ID. This document needs to be updated after each version release. It also contains lists of roles&responsibilities and other lists.

### Audience

Bravo! Encore!

## Requirements for ISO 26262

The ISO 26262 standard requires us to track and report known safety-critical issues and possible failures, documenting for each how it arises, what issue# addresses it, what test case tests it, and what version it is fixed in.

### ASIL D Capable

ASIL (Automotive Safety Integrity Level) describes the risk level of something. ASIL D=highest, C=high, B=medium, A=low risk, QM = not safety critical. 

Since a compiler and a toolkit don’t have a specific vehicle function, they don’t have an intrinsic ASIL derived from a HARA (Hazard Analysis and Risk Assessment). Slint SC is a “Safety Element out of Context” (SEooC). The GUI components of Slint SC can be used for mission-critical digital instrument clusters. 

**The "Maximum Requirement" Rule:** If a single component (like a "Warning Icon" widget) is intended to display a "Brake Failure" message, it needs to be **ASIL D Capable**.

### Failure Scenarios

Describing specific possible failure scenarios (especially in the context of software running on a car) can lead to the definitions of Requirements and Constraints. 

These lists are created during the Hazard Analysis and Risk Assessment (HARA) phase. 

### Specific Requirements


Each Requirement should have a descriptive ID that begins with SR_, a description. ASIL=D. 

The ISO 26262 standard tells us what properties a safety-critical system must have (traceability, freedom from interference, determinism, etc.), but it doesn't tell us how to write those requirements for a GUI toolkit. The following sections contain some specific, actionable engineering requirements that a project like Slint SC needs to satisfy to be used in an ASIL D context.

#### SR_Bounded_Execution_Time

Slint SC shall guarantee a strictly bounded maximum execution time for rendering a single frame, ensuring that the critical rendering loop never blocks the main execution thread beyond the hardware display refresh interval (e.g., 16.6ms for 60Hz).

#### SR_Static_Memory_Allocation

Following initialization, Slint SC shall not perform dynamic memory allocation (e.g., `malloc`, `new`) during the continuous rendering loop. All memory pools, vertex buffers, and command buffers must be pre-allocated.

#### SR_State_Machine_Determinism

Slint SC's internal state machine for UI component lifecycle, event propagation, and rendering state must be fully deterministic and reproducible given a specific sequence of inputs.

#### SR_Safety_Critical_Separation

Slint SC shall provide a mechanism to explicitly separate defined safety-critical UI elements (e.g., ASIL A/B telltales like ABS warning, brake failure) from non-safety-critical (QM) infotainment graphics.

#### SR_Z_ORDER_GUARANTEE

Slint SC shall mathematically or structurally guarantee that elements classified as "Safety-Critical" are rendered at the highest Z-index and cannot be occluded, clipped, or obscured by any QM-level graphics or animations.

#### SR_HARDWARE_CRC_SUPPORT

Slint SC shall output framebuffer fragments or layers in a manner that supports hardware-assisted cyclic redundancy checks (CRC). It must provide the bounding box coordinates of safety-critical elements to external watchdog hardware to continuously verify that the correct pixels were driven to the display.

#### SR_LATE_COMPOSITING

Slint SC's architecture shall support running safety-critical rendering on a separate, ASIL-certified hardware layer or hypervisor partition, compositing the result over the QM-level graphics at the physical display controller level.

#### SR_RESOURCE_FALLBACK

If an external graphical asset (e.g., image, font glyph, 3D mesh) is corrupted, missing, or fails to decode, the toolkit shall not crash or halt rendering. It must immediately transition to an explicitly defined safe fallback state (e.g., rendering a pre-compiled geometry or primitive shape).

#### SR_INPUT_VALIDATION

All inputs from IPC (Inter-Process Communication), external sensors, or touch controllers shall be strictly bounds-checked. Invalid inputs must be safely discarded without corrupting the rendering state or entering an undefined state.

#### SR_WATCHDOG_TICK_VERIFICATION

Slint SC must trigger a software or hardware watchdog timer at a specified interval (e.g., once successfully per frame swap) to demonstrate that the rendering thread has not deadlocked.

#### SR_CODE_GENERATION

Slint-compiler’s output must be verifiable/qualified according to ISO 26262 Part 8, Clause 11.

#### SR_CODING_STANDARDS_COMPLIANCE

Generated code and the underlying GUI runtime library shall be statically analyzable and comply with MISRA C (2012), MISRA C++ (2008 / AUTOSAR C++14), or equivalent safety-aware subsets (like `unsafe` block management in Rust, when using Rust).

#### SR_TEST_COVERAGE

Slint SC must support headless or offline automated testing frameworks capable of running on CI/CD pipelines to verify layout engines, event propagation, and state transitions. Code Coverage should be measurable (can we do that in Rust?) aiming for >90% code coverage.

#### SR_SEPARATION_OF_CONCERNS

Slint SC architecture enforces the separation of business logic (backend/state) from presentation logic (frontend/pixels).

#### SR_CONCURRENCY_CONTROL

To avoid race conditions that could yield incorrect displays, the core UI update, layout, and rendering commands must execute sequentially on a single managed thread or explicitly defined thread pool with static concurrency constraints. This makes it possible to show that the i-slint-core runtime, especially the property binding evaluation and Z-ordering layout mechanisms, are fully deterministic, bounded, and provably testable. 

## Components of the System

This is what is included in Slint SC:

* slint-compiler
* Individual slint language features 
* Features offered by specific crates that are part of the Slint Rust library

Each of these things can have a **Usage** and a **Constraints** section. 

Each feature of the language or a library can map to a Requirement ID, and have 1 or more code test-examples. 

<Insert diagram that shows how the components/crates relate to each other?>

### Installation Procedures

How to install Slint SC: the compiler, and its related libraries. 

### Usage: slint-compiler

This is the only executable in slint. Describe how it is used. 
TODO: Add a 'Constraints' section for this command. 

## Constraints

The standard essentially views a **Requirement** as what the system *must do* (or a property it must have), whereas a **Constraint** is a boundary condition that *limits the solution space*.

For slint-compiler, this section would describe which command line options are required for using slint-compiler safely.

For APIs, the constraints might explain that some functions are experimental and can not be used safely yet. Or, that certain values passed as parameters into functions are not safe? Or, that certain language features can only be used a certain way to be safe.

Each Constraint should have an ID that begins with CON_, and also have a Rationale, Impact, and Mitigation. 

## Slint Development Cycle

<https://github.com/slint-ui/slint/blob/master/docs/internal/processes.md>

### Development tools, hardware and software components

What tools do we use to develop slint? (ISO 26262-8 sections 11, 12 and 13)

### Procedure for Version Control/Configuration Management

Here we describe how slint is developed, how git and github are used, etc.

Configuration management (ISO 26262-8 section 7)

### CI system used

Q: What CI system do we use? We describe what tasks are done after each push, and if we have nightly tasks, we describe those too. 

### Reporting Bugs

Issues are tracked here: <https://github.com/slint-ui/slint/issues>

## Handling Unsafety

Q: What is unsafe code? Do we have examples? 

Q: What is the definition of a “safety-critical error”? 

In the case of rustc, unsafety is “the presence of code that the compiler cannot automatically prove with respect to the memory safety guarantees of Rust”. In the case of Coco, it is whenever an incorrect number is reported in a coverage report. 

The definition of “safety-critical” for slint-compiler, or each specific slint language or library feature, may be different.

### Known Issues

Q: What are the safety critical issues (referenced by github issue ID) that we have faced? Which ones are fixed? Fixed in which version? Which testcases test them?  We can start with those as we are developing our validator package. 

This list/table could be auto-generated based on what is in the github issue database, if the relevant issues were each properly tagged with “safety-critical”. 

## Validation - Running the Tests

* Validation activities (see ISO 26262-4 section 9)

Validation involves running a set of safety-critical tests. We describe how to run them here, and how to understand the results.  Each test should be described in a section here, tagged with appropriate requirement IDs. 

We should probably write a Validator GUI that runs the safety-critical tests and shows the results in a nice way, and then the instructions for running Validator will go here. 

## Activities for the assessment of functional safety

The functional safety is assessed independently by XYZ GmbH in ABCD, Germany. 

## Policy and strategy for achieving functional safety

The safety and quality of the system are the primary objectives of all management, development, maintenance and support activities. In order to achieve these goals, all employees must be involved in the quality assurance measures. This requires
    • Adequate qualification of employees
    • A culture of safe working
    • Compliance with the safety life cycle to avoid systematic errors
    • Precise, complete documentation 
    • Compliance with procedures and measures 
    • Independent verification of all work results
    • Processes are work products that are constantly being improved.
    • Technical support, use of tools, automation where possible.

