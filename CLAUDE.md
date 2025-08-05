# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Cesium for Unreal is an Unreal Engine plugin that brings 3D geospatial capabilities to Unreal Engine, enabling streaming of global-scale 3D content.

## Build Commands

### Building Native Libraries (Required First)
```bash
# Set Unreal Engine path (required)
export UNREAL_ENGINE_ROOT='/path/to/UE_5.4'

# Build cesium-native libraries
cd extern/
cmake -B build -S . -DCMAKE_BUILD_TYPE=RelWithDebInfo
cmake --build build --target install --parallel 14
```

### Code Formatting
```bash
npm ci                    # Install dependencies
npm run format           # Format all code
npm run format -- --dry-run -Werror  # Check formatting (CI mode)
```

### Running Tests
Tests use Unreal's Automation framework. Open `TestsProject/TestsProject.uproject` in Unreal Editor, then use Tools → Test Automation to run Cesium tests.

## Architecture

The plugin consists of two main modules:
- **CesiumRuntime**: Core functionality (3D Tiles loading, georeference, coordinate systems)
- **CesiumEditor**: Editor-only features (UI panels, asset importing)

Key architectural patterns:
- **Async Loading**: All I/O operations use Unreal's async task system
- **Memory Management**: Shared pointers for cesium-native objects, Unreal's GC for UObjects
- **Coordinate Systems**: Handles conversions between WGS84, ECEF, and Unreal coordinates
- **Component Architecture**: ACesium3DTileset actors contain UCesium3DTilesetComponent

## Key Development Notes

1. **Compiler Compatibility**: Must use same compiler for cesium-native and Unreal plugin
2. **Platform Support**: Windows, macOS, Linux, Android, iOS - all changes must work cross-platform
3. **Build Configuration**: Debug/DebugGame use debug libs when available, otherwise release
4. **Plugin Location**: Must be in `Plugins/cesium-unreal/` directory structure

## Testing Patterns

- Unit tests in `Source/CesiumRuntime/Private/Tests/*.spec.cpp`
- Performance tests in `Source/CesiumRuntime/Private/Tests/*.perf.cpp`
- Integration tests in TestsProject
- All tests inherit from `CesiumTestHelpers` for common setup

## Common Tasks

### Adding a New Class
1. Add header to `Source/CesiumRuntime/Public/`
2. Add implementation to `Source/CesiumRuntime/Private/`
3. Export with `CESIUMRUNTIME_API` for public classes
4. Update module's Build.cs if adding dependencies

### Debugging 3D Tiles Loading
- Check `ACesium3DTileset::LoadTileset()` for initialization
- Monitor `UCesium3DTilesetComponent::UpdateLoadStatus()` for loading progress
- Use `LogCesium` category for logging

### Working with Geospatial Data
- Use `UCesiumGeoreference` for coordinate transformations
- `GeoTransforms` namespace contains conversion utilities
- Always consider precision issues with large-scale coordinates