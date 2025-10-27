# Changelog

All notable changes to Claude Record will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased] - v2.0-alpha

### Added - Phase 3 (2025-10-27)
- **Context Retrieval**: Automatic display of relevant past sessions at startup
  - Three retrieval modes: recent, smart (default), keywords
  - Smart mode combines recent sessions + directory-based context
  - Keyword search with relevance ranking
  - Beautiful formatted display with colors and emojis
  - Human-readable time (e.g., "3d ago")
- **Context Commands**: Control context display
  - `--no-context` - Disable context retrieval
  - `--context MODE` - Set retrieval mode (recent/smart/keywords)
  - `--context-keywords KW` - Search by specific keywords
- **Display Features**: Rich formatted output
  - Shows date, time ago, summary excerpt
  - Lists tags (keywords) and files modified
  - Color-coded with clean separators
  - Non-intrusive (to stderr)

### Added - Phase 2 (2025-10-25)
- **Keyword Indexing**: Automatic extraction and indexing of session keywords
  - Multi-source keyword extraction (summaries, file extensions, tech keywords)
  - Session index with metadata, keywords, and excerpts
  - Keyword index with session mappings and frequency tracking
  - Automatic index updates after summarization
- **Index Management**: Command to rebuild indexes
  - `--rebuild-index` - Rebuild all indexes from existing summaries
  - Indexes stored in `~/.claude-record/` as JSON
  - Graceful handling of missing jq dependency

### Added - Phase 1 (2025-10-24)
- **AI-Powered Summarization**: Automatic session summaries using Claude Code
  - 8-section markdown summaries (Overview, Tasks, Decisions, Files, etc.)
  - Smart validation (skips sessions <60s or logs >10MB)
  - Timeout protection (30 seconds with progress indicators)
- **Summary Management**: Commands to control summarization
  - `--no-summarize` - Skip automatic summarization
  - `--summarize-only` - Generate summary for most recent session
- **Summary Format**: Structured markdown with metadata header
  - Session ID, date, duration, model information
  - Technologies/Topics section for keyword extraction
  - Files Modified section with paths
  - Follow-up items and related sessions

### Changed
- Summary files (*.summary) now created alongside logs
- Metadata format extended with `has_summary` field
- Index files stored in `.claude-record/` directory
- Script size: 1,722 lines (was 676 for v1.0)
- Context now displays automatically at session start

### Technical
- New file structure: `.claude-record/` for indexes
- JSON-based indexes for fast lookup
- Markdown format for summaries
- Graceful error handling and timeouts
- Non-fatal failures for summarization and indexing
- Dependencies: jq (optional, for indexing)

### Planned - Phase 4
- **Cross-Platform Support**: Full Linux and Windows testing
- **Windows Scripts**: PowerShell version for native Windows use

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
