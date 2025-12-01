# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project Overview

This is an **Aerie Mission Model** for a Mars orbiter (MarsSat). It's part of NASA's AMMOS (Advanced Multi-Mission Operations System) Aerie planning framework. The model simulates spacecraft subsystems including geometry, power, data, telecom, and radar for mission planning and analysis.

## Build System

- **Build Tool**: Gradle with Java 21
- **Project Name**: `mars-sat` (defined in settings.gradle)

### Common Commands

```bash
# Build the mission model JAR
./gradlew missionmodel:build

# Run tests
./gradlew test

# Build all modules
./gradlew build

# Build procedural constraint JARs
./gradlew constraints:buildAllProcedureJars

# Clean build artifacts
./gradlew clean
```

### Environment Setup

1. Copy `.env.template` to `.env`
2. Set `GITHUB_USER` and `GITHUB_TOKEN` (required for accessing Aerie packages from GitHub Packages)
3. Optionally set `DOCKER_TAG` to specify an Aerie version (defaults to latest)

## Project Structure

```
├── missionmodel/           # Main mission model module
│   ├── src/main/java/missionmodel/
│   │   ├── Mission.java    # Top-level model class
│   │   ├── Configuration.java
│   │   ├── geometry/       # SPICE-based geometry calculations
│   │   ├── power/          # Power subsystem (solar arrays, batteries, PEL)
│   │   ├── data/           # Data management (buckets, playback)
│   │   ├── telecom/        # Telecom model
│   │   ├── radar/          # Radar model
│   │   └── spice/          # SPICE initialization
│   ├── spice/kernels/      # SPICE kernel files
│   └── src/test/java/      # JUnit 5 tests
├── constraints/            # Procedural constraints module
│   └── src/main/java/constraints/procedures/
├── scheduling/             # Scheduling rules module
└── docker-compose.yml      # Full Aerie stack deployment
```

## Key Technologies

- **Aerie Framework**: NASA's mission planning and simulation framework
- **SPICE**: NASA/NAIF toolkit for spacecraft geometry calculations
- **Streamline**: Aerie's resource modeling library (contrib package)

## Subsystem Models

### Geometry Model
Uses SPICE kernels to compute:
- Spacecraft-to-body distances and velocities
- Orbital elements (periapsis, apoapsis)
- Eclipses and occultations
- Sun-body-spacecraft angles
- Sub-spacecraft illumination

Key class: `GenericGeometryCalculator`

### Power Model
- **PEL (Power Equipment List)**: Tracks CBE and MEV power loads
- **Solar Arrays**: Power generation based on Sun distance and off-point angle
- **Batteries**: State of charge tracking

### Data Model
- Bucket-based data storage with priorities
- Data generation, playback, and reprioritization activities

## SPICE Configuration

- Meta kernel: `missionmodel/spice/kernels/latest_meta_kernel.tm`
- The `SPICE_DIRECTORY` environment variable controls kernel location
- Default path: `spice/kernels` (relative) or set via env var

## Docker Deployment

Start the full Aerie stack:
```bash
docker compose up -d
```

The stack includes: UI (port 80), Hasura (8080), Gateway (9000), Merlin (27183), Scheduler (27185), Sequencing (27184), and PostgreSQL (5432).

SPICE kernels are mounted from `./spice/kernels` into containers.

## Testing

Tests use JUnit 5 with Aerie's `merlin-framework-junit` for simulation testing:

```bash
./gradlew test
```

Test files are in `missionmodel/src/test/java/missionmodel/`.

## Important Notes

- The model requires SPICE kernels to run simulations
- GitHub credentials are required to download Aerie dependencies from GitHub Packages
- The mission model JAR includes all dependencies (fat JAR) except `merlin-sdk`
- Geometry spawner activities (AddPeriapsis, AddApoapsis, AddOccultations, AddSpacecraftEclipses) auto-generate events during simulation
