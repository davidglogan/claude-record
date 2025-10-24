# Changelog

All notable changes to Claude Record will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased] - v2.0-dev

### Added
- **AI-Powered Summarization**: Automatic session summaries using Claude
- **Keyword Indexing**: Fast keyword-based search across sessions
- **Context Retrieval**: Automatic surfacing of relevant past sessions
- **Smart Context Mode**: Intelligent context based on recent work and current directory
- **Configuration System**: JSON-based configuration file
- **Summary Management**: Commands to view, generate, and manage summaries
- **Index Management**: Commands to rebuild and query indexes
- New flags:
  - `--no-summarize` - Skip automatic summarization
  - `--summarize-only` - Generate summary for last session
  - `--summarize-all` - Bulk summarize old sessions
  - `--no-context` - Skip context retrieval
  - `--smart-context` - Use intelligent context mode
  - `--context KEYWORDS` - Search for specific context
  - `--recent N` - Show N recent sessions
  - `--rebuild-index` - Rebuild all indexes
  - `--show-index` - Show index statistics
  - `--list-topics` - List all indexed keywords
  - `--search-summaries TERM` - Search through summaries
  - `--sessions-by-topic TOPIC` - List sessions by topic

### Changed
- Summary files (*.summary) now created alongside logs
- Metadata format extended with `has_summary` field
- Index files stored in `.claude-record/` directory

### Technical
- New file structure: `.claude-record/` for config and indexes
- JSON-based indexes for fast lookup
- Markdown format for summaries
- Graceful error handling and timeouts

## [1.0.0] - 2025-10-23

### Added
- **Session Recording**: Automatic recording of all Claude Code interactions
- **Log Filtering**: Removes ANSI codes and UI noise for readable logs
- **Metadata Tracking**: Session start/end times, duration, size, working directory
- **Automatic Cleanup**: Configurable retention of N most recent sessions
- **Search Functionality**: Grep-based search through all recorded sessions
- **Session Listing**: Formatted display of all sessions with metadata
- **Cross-Platform**: Support for Linux and macOS
- Command-line options:
  - `-h, --help` - Show help message
  - `-d, --directory DIR` - Custom log directory
  - `-f, --filename NAME` - Custom filename prefix
  - `-s, --max-size SIZE` - Maximum log file size
  - `-m, --max-sessions NUM` - Maximum sessions to keep
  - `-v, --verbose` - Verbose output
  - `--dry-run` - Preview mode
  - `--cleanup` - Clean up old sessions
  - `--list` - List all sessions
  - `--search TERM` - Search through sessions
  - `--no-auto-cleanup` - Disable automatic cleanup
  - `--version` - Show version

### Features
- Intelligent log filtering reduces file sizes by 30-50%
- Automatic file rotation when size limits exceeded
- Platform-specific command handling (macOS vs Linux)
- Signal handling for clean interruption (Ctrl+C)
- Environment variable support for configuration

### Files
- Session logs: `claude_conversation.YYYYMMDD-HHMMSS.log`
- Metadata: `claude_conversation.YYYYMMDD-HHMMSS.meta`
- Default location: `~/Documents/claude/`

## [0.1.0] - Initial Development

- Initial development and testing
- Proof of concept for session recording wrapper

---

## Version Naming

- **Major.Minor.Patch** (e.g., 2.0.0)
  - **Major**: Breaking changes, major new features
  - **Minor**: New features, backward compatible
  - **Patch**: Bug fixes, minor improvements

## Release Tags

- `v1.0.0` - Stable release with basic recording
- `v2.0-alpha` - Summarization feature (in development)
- `v2.0-beta` - Indexing feature (planned)
- `v2.0-rc` - Context retrieval feature (planned)
- `v2.0.0` - Full v2.0 release with all features

---

*For detailed technical changes, see commit history and [PROJECT_STATUS.md](PROJECT_STATUS.md)*
