# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**subconverter** is a C++20 utility for converting between various proxy subscription formats (Clash, Surge, Quantumult X, etc.). It's a web service that runs on port 25500 by default and provides REST endpoints for subscription format conversion.

## Build System and Development Commands

### Build Commands
The project uses CMake as the primary build system:

```bash
# Basic build
cmake -DCMAKE_BUILD_TYPE=Release .
make -j6

# Build with options
cmake -DCMAKE_BUILD_TYPE=Release -DUSING_MALLOC_TRIM=ON .
cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_STATIC_LIBRARY=ON .  # Static library
```

### Platform-specific Build Scripts
- **macOS**: `scripts/build.macos.release.sh` - Full release build with dependencies
- **Windows**: `scripts/build.windows.release.sh` 
- **Alpine Linux**: `scripts/build.alpine.release.sh`

### Dependencies Management
The macOS build script handles all dependencies automatically:
- rapidjson, zlib, pcre2 (via Homebrew)
- yaml-cpp, quickjspp, libcron, toml11 (compiled from source)

### Rule Updates
```bash
# Update proxy rules (requires Python)
python scripts/update_rules.py -c scripts/rules_config.conf
```

## Code Architecture

### Core Components

**Main Modules** (all in `src/`):
- `main.cpp` - Entry point, command-line argument parsing, signal handling
- `handler/interfaces.cpp` - Main API endpoint handlers for subscription conversion
- `generator/config/subexport.cpp` - Core subscription format export logic
- `parser/subparser.cpp` - Subscription format parsing (supports all input types)
- `server/webserver_httplib.cpp` - HTTP server using httplib
- `script/script_quickjs.cpp` - JavaScript engine integration for filtering

**Generator System**:
- `generator/config/` - Configuration generation and export logic
- `generator/template/` - Template rendering system using Jinja2-style templates
- Base templates in `base/base/` directory for each target format

**Parser System**:
- `parser/subparser.cpp` - Unified parser for all subscription formats
- `parser/infoparser.cpp` - Subscription metadata extraction
- Supports: SS, SSR, V2Ray, Trojan, Clash, Surge, Quantumult X, etc.

**Configuration System**:
- `base/pref.example.ini` - Main configuration template
- `base/config/` - External configuration examples and presets
- `base/profiles/` - Profile-based configurations

### Key Libraries and Dependencies
- **httplib** - HTTP server (header-only, in `include/`)
- **rapidjson** - JSON parsing/generation
- **yaml-cpp** - YAML parsing for Clash configs
- **PCRE2** - Regular expression engine
- **QuickJS** - JavaScript engine for node filtering
- **libcron** - Cron job scheduling
- **toml11** - TOML configuration parsing

### Rule System
Extensive rule sets in `base/rules/`:
- **ACL4SSR** - Main rule provider with comprehensive lists
- **DivineEngine** - Alternative rule sets
- Supports proxy groups, direct connections, ad blocking, geo-location routing

## Configuration

### Main Config (`base/pref.example.ini`)
- **[common]** - API settings, default URLs, filtering rules
- **[userinfo]** - Traffic and expiry data extraction rules
- **[node_pref]** - Default node preferences
- **[server]** - Web server configuration

### External Configs (`base/config/`)
Multiple preset configurations for different use cases (ACL4SSR variants, regional configs, etc.)

## Service Architecture

### Web Server
- Default port: 25500
- Main endpoint: `/sub` for subscription conversion
- Additional endpoints: `/render` for template rendering, `/qx-rewrite` for rule conversion
- Supports CORS for web client usage

### Command Line Arguments
- `-f, --file` - Specify config file path
- `-g, --gen` - Generator mode for local file generation  
- `--artifact` - Generate specific profiles
- `-l, --log` - Log to file
- `-cfw` - Clash for Windows child process mode

### Threading and Performance
- Multi-threaded request handling (`handler/multithread.cpp`)
- Memory optimization with optional malloc_trim
- Efficient regex engine (PCRE2) for rule matching
- JavaScript engine for advanced node filtering

## Development Notes

### C++ Standards and Practices
- **C++20** standard required
- Header-only libraries preferred (httplib, rapidjson in `include/`)
- Extensive use of STL containers and algorithms
- Cross-platform compatibility (Windows, macOS, Linux)

### Template System
Uses Jinja2-style templating with the inja library for generating configuration files from base templates.

### Subscription Processing Flow
1. Parse input subscription (various formats)
2. Apply filtering rules (exclude/include patterns, JavaScript filters)
3. Apply external configuration rules 
4. Generate output in target format using templates
5. Return processed configuration

### Rule Processing
- Rule files are plain text lists (one rule per line)
- Supports different rule types: DOMAIN, DOMAIN-SUFFIX, IP-CIDR, etc.
- Rules are processed in order and can be combined from multiple sources