# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

opendbc is a Python API for car control and telemetry, primarily used with [openpilot](https://github.com/commaai/openpilot). It provides interfaces to control steering, gas, brakes and read vehicle state via CAN bus.

## Build & Test Commands

```bash
# All-in-one: install deps, build, lint, test (runs in CI)
./test.sh

# Individual steps:
source ./setup.sh              # Install dependencies via uv, activate venv
scons -j8                      # Build C++ extensions
pytest .                       # Run all tests
pytest -n8                     # Run tests with parallelism
pytest opendbc/car/tests/test_car_interfaces.py  # Single test file
lefthook run test              # Run lint + tests (same as CI)
lefthook run lint              # Just linting (ruff, codespell, cpplint, misra)

# Safety tests with coverage
./opendbc/safety/tests/test.sh           # Run safety tests (requires 100% coverage)
./opendbc/safety/tests/test.sh --ubsan   # With undefined behavior sanitizer
./opendbc/safety/tests/test.sh --report  # Generate HTML coverage report
```

## Project Architecture

### Core Modules (`opendbc/`)

- **`dbc/`**: Repository of DBC files defining CAN message formats for various vehicles
- **`can/`**: CAN message parsing/packing library - `CANParser` reads messages, `CANPacker` builds them
- **`car/`**: High-level Python interfaces for vehicle control
- **`safety/`**: C safety firmware for panda hardware (enforces openpilot safety model)

### Car Port Structure (`opendbc/car/<brand>/`)

Each supported brand has this structure:
- `values.py`: Enumerates supported cars as `Platforms` enum, defines `CarControllerParams`, flags
- `interface.py`: `CarInterface` class - main entry point, implements `_get_params()` for car configuration
- `carstate.py`: `CarState` class - parses CAN messages into vehicle state
- `carcontroller.py`: `CarController` class - generates CAN messages for control
- `fingerprints.py`: ECU firmware versions for automatic car identification
- `<brand>can.py`: Helper functions for building brand-specific CAN messages
- `radar_interface.py`: Radar data parsing (if equipped)

### Key Classes

- `Platforms` (enum): Each car model is a platform with `PlatformConfig` containing specs, DBC dict, flags
- `CarInterfaceBase`: Abstract base in `interfaces.py` - all brand interfaces inherit from this
- `CarStateBase`: Abstract base for parsing CAN → vehicle state
- `CarControllerBase`: Abstract base for control output → CAN messages

### Data Flow

1. CAN messages → `CANParser` (using DBC) → `CarState` → vehicle state struct
2. Control commands → `CarController` → `CANPacker` (using DBC) → CAN messages

## Code Style

- 2-space indentation (configured in ruff)
- Line length: 160 chars
- Type hints expected (mypy enforced)
- Safety code follows MISRA C:2012 guidelines
- Safety tests require 100% line coverage
