# Claude Record

> Intelligent session recording and memory system for Claude Code

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Platform](https://img.shields.io/badge/platform-Linux%20%7C%20macOS%20%7C%20Windows-blue.svg)](https://github.com/yourusername/claude-record)
[![Version](https://img.shields.io/badge/version-2.0--dev-orange.svg)](https://github.com/yourusername/claude-record)

## Overview

`claude-record` is an intelligent wrapper for [Claude Code](https://claude.ai/code) that automatically records your sessions, generates AI-powered summaries, and provides contextual memory across conversations. Think of it as giving Claude a memory of your past work together.

### Key Features

- 🎯 **Automatic Session Recording** - Every interaction with Claude Code is logged
- 🤖 **AI-Powered Summaries** - Automatically generates concise summaries of each session
- 🔍 **Keyword Indexing** - Fast search across all past sessions
- 📚 **Context Retrieval** - Surfaces relevant past sessions when you start new work
- 🌍 **Cross-Platform** - Works on Linux, macOS, and Windows
- 🔒 **Privacy-Focused** - All data stays local on your machine
- ⚙️ **Configurable** - Extensive customization options

## Quick Start

### Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/claude-record.git
cd claude-record

# Run the installer
./scripts/install.sh

# Or install manually
mkdir -p ~/.local/bin
cp src/claude-record ~/.local/bin/
chmod +x ~/.local/bin/claude-record
export PATH="$HOME/.local/bin:$PATH"
```

### Usage

```bash
# Use claude-record instead of claude
claude-record

# That's it! Your sessions are now recorded and summarized automatically
```

## What's New in v2.0

**Version 2.0** introduces intelligent memory and context awareness:

- **Auto-Summarization**: Every session gets an AI-generated summary highlighting what you accomplished
- **Smart Context**: See relevant past sessions when you start new work
- **Powerful Search**: Find past sessions by keywords, topics, or files modified
- **Index System**: Fast keyword-based indexing for quick retrieval

See [CHANGELOG.md](docs/CHANGELOG.md) for detailed release notes.

## Documentation

- [Architecture Overview](docs/ARCHITECTURE.md) - System design and technical details
- [v1.0 Design](docs/V1_DESIGN.md) - Original implementation documentation
- [Configuration Guide](docs/CONFIGURATION.md) - Customize claude-record behavior
- [Migration Guide](docs/MIGRATION.md) - Upgrading from v1.0 to v2.0
- [Troubleshooting](docs/TROUBLESHOOTING.md) - Common issues and solutions
- [Project Status](docs/PROJECT_STATUS.md) - Current development status and roadmap

## Examples

### Basic Recording

```bash
# Start a recorded session
claude-record

# All your interactions are automatically recorded to ~/Documents/claude/
```

### Search Past Sessions

```bash
# List all recorded sessions
claude-record --list

# Search for specific content
claude-record --search "API design"

# Search through summaries only
claude-record --search-summaries "authentication"
```

### Context Retrieval

```bash
# Start session with smart context (shows relevant past sessions)
claude-record --smart-context

# Show last 5 sessions before starting
claude-record --recent 5

# Skip context retrieval
claude-record --no-context
```

### Summary Management

```bash
# Generate summary for last session (if skipped)
claude-record --summarize-only

# Generate summaries for all old sessions
claude-record --summarize-all

# View a specific session's summary
claude-record --show-summary ~/Documents/claude/claude_conversation.20251023-234848.summary
```

### Index Management

```bash
# Rebuild indexes from all summaries
claude-record --rebuild-index

# Show index statistics
claude-record --show-index

# List all indexed topics
claude-record --list-topics
```

## Configuration

Create `~/.claude-record/config.json` to customize behavior:

```json
{
  "summarization": {
    "enabled": true,
    "method": "claude-code",
    "timeout_seconds": 30
  },
  "context_retrieval": {
    "enabled": true,
    "mode": "recent",
    "num_recent": 3
  },
  "indexing": {
    "enabled": true,
    "auto_update": true
  }
}
```

See [CONFIGURATION.md](docs/CONFIGURATION.md) for all options.

## How It Works

1. **Recording**: When you run `claude-record`, it wraps the `claude` command and captures all terminal I/O
2. **Filtering**: UI noise and ANSI sequences are filtered out to keep logs clean
3. **Summarization**: At session end, Claude analyzes the log and generates a structured summary
4. **Indexing**: Keywords are extracted and indexes are updated for fast search
5. **Retrieval**: When you start a new session, relevant past context is surfaced

```
┌─────────────────┐
│  claude-record  │
└────────┬────────┘
         │
         ├──> Record session
         ├──> Generate summary (AI)
         ├──> Extract keywords
         ├──> Update indexes
         └──> Retrieve context (next session)
```

## Platform Support

| Platform | Status | Notes |
|----------|--------|-------|
| Linux (Ubuntu, Debian) | ✅ Supported | Primary development platform |
| macOS | ✅ Supported | Full feature support |
| Windows (Git Bash) | ✅ Supported | Recommended for Windows users |
| Windows (PowerShell) | 🚧 In Progress | Coming in v2.1 |

## Requirements

- [Claude Code](https://claude.ai/code) installed and configured
- Bash 4.0+ (Linux, macOS, Git Bash on Windows)
- Standard Unix utilities: `grep`, `sed`, `awk`, `find`, `stat`
- Optional: `jq` for JSON manipulation (auto-installed if missing)

## Contributing

Contributions are welcome! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

### Development Setup

```bash
# Clone the repository
git clone https://github.com/yourusername/claude-record.git
cd claude-record

# Run tests
./tests/run-tests.sh

# Test installation locally
./scripts/install.sh --local
```

## Project Status

**Current Version**: 2.0-dev (In Active Development)

See [PROJECT_STATUS.md](docs/PROJECT_STATUS.md) for detailed status, roadmap, and design decisions.

## License

MIT License - see [LICENSE](LICENSE) for details.

## Credits

Created with [Claude Code](https://claude.ai/code) by Anthropic.

## Support

- 📖 [Documentation](docs/)
- 🐛 [Issue Tracker](https://github.com/yourusername/claude-record/issues)
- 💬 [Discussions](https://github.com/yourusername/claude-record/discussions)

---

**Note**: This project is in active development. v2.0 features are being implemented. Star and watch for updates!
