# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build System and Commands

Servo uses a Python-based `mach` build tool that wraps Cargo. All development commands go through `./mach`:

### Essential Commands
```bash
# Bootstrap dependencies
./mach bootstrap

# Build Servo (release build)
./mach build
# Build with features
./mach build --dev  # Development build
./mach build --release  # Optimized release build

# Run Servo
./mach run [URL]
./mach run --release [URL]

# Code quality checks
./mach test-tidy        # Source code style/lint checks
./mach fmt             # Format Rust, Python, and TOML files
./mach check           # Run cargo check
./mach clippy          # Run cargo clippy

# Testing
./mach test-unit       # Run unit tests
./mach test-wpt        # Run web platform tests
./mach test-wpt [TEST_PATH]  # Run specific WPT test
./mach smoketest       # Quick smoke test

# Documentation
./mach doc            # Generate rustdoc documentation
```

### Platform-Specific Notes
- On NixOS systems, mach automatically enters nix-shell
- Use `./mach build --android` for Android builds
- Use `./mach build --target=...` for cross-compilation

## Architecture Overview

Servo is a modular, multi-process browser engine with a component-based architecture:

### Core Process Architecture
- **Constellation**: Central coordinator managing multiple processes and sandboxing (`components/constellation/`)
- **Script Thread**: JavaScript engine and DOM implementation in separate processes (`components/script/`)
- **Layout Thread**: CSS layout and rendering pipeline (`components/layout/`)
- **Compositor**: GPU-accelerated rendering and window management (`components/compositing/`)

### Key Components
- **libservo** (`components/servo/`): Main library binding all components together
- **servoshell** (`ports/servoshell/`): Desktop application shell
- **Net** (`components/net/`): HTTP networking and resource loading
- **WebGL/WebGPU**: Graphics API implementations for 3D rendering
- **Media**: Audio/video playback support with pluggable backends

### Inter-Process Communication
- Components communicate via `ipc-channel` for type-safe message passing
- Shared interfaces defined in `components/shared/` (traits and APIs)
- Each major component has corresponding `*_traits` crates for interfaces

### Key Design Patterns
1. **Process Isolation**: Major components run in separate processes for security/stability
2. **Message Passing**: All component communication is asynchronous via channels
3. **Trait-Based Interfaces**: Components interact through well-defined trait boundaries
4. **Feature Flags**: Extensive use of Cargo features for conditional compilation

## Code Organization

### Component Structure
- Each component has its own `Cargo.toml` and is a separate crate
- Shared interfaces live in `components/shared/[component]/`
- Cross-component communication goes through trait definitions

### Important Directories
- `components/` - Core browser engine components
- `ports/` - Platform-specific application shells
- `tests/` - Test suites (unit, integration, web platform tests)
- `python/` - Build system and tooling scripts
- `resources/` - Static resources and configuration files

## Development Workflow

### Code Style
- Servo uses custom linting via `servo-tidy` configured in `servo-tidy.toml`
- Standard Rust formatting with `rustfmt.toml` configuration
- Run `./mach fmt` and `./mach test-tidy` before commits

### Testing Strategy
- Unit tests are embedded within component crates
- Integration tests in `tests/unit/`
- Web Platform Tests (WPT) in `tests/wpt/`
- Always run `./mach smoketest` to verify basic functionality

### Multi-Process Development
When working on cross-component features:
1. Define interfaces in `components/shared/[component]/`
2. Implement message types for IPC communication
3. Handle both single-process and multi-process execution paths
4. Consider sandboxing implications for new functionality

### Working with JavaScript/DOM
- DOM objects in `components/script/dom/`
- Use `#[crown]` annotations for Servo-specific linting
- JavaScript bindings generated from WebIDL in `components/script_bindings/webidls/`
- Always consider security boundaries when adding new DOM APIs

## Key Dependencies

### Core Technology Stack
- **SpiderMonkey** (`js`/`mozjs`): JavaScript engine
- **Stylo**: Parallel CSS engine (from `servo/stylo` repo)
- **WebRender**: GPU-accelerated rendering
- **Hyper**: HTTP client/server
- **Tokio**: Async runtime for networking

### Important Crate Patterns
- Most Servo crates use workspace dependencies defined in root `Cargo.toml`
- Features are heavily used - check `Cargo.toml` files for available features
- Platform-specific dependencies are conditionally compiled based on target OS

## Debugging and Profiling

### Debug Features
- `debugmozjs` - Enable SpiderMonkey debugging
- `profilemozjs` - JavaScript profiling support
- `tracing` - Performance tracing (can use Perfetto backend)
- `webgl_backtrace` - WebGL call stack traces

### Running with Debugging
```bash
./mach build --dev --features debugmozjs
./mach run --dev --debugger [URL]  # Run under debugger
```