# Claude Record v1.0 Design Documentation

## Overview

Claude Record v1.0 is a bash wrapper script for Claude Code that provides automatic session recording, log management, and basic search capabilities. This document describes the design and implementation of the original version.

## Design Goals

1. **Transparent Recording**: Wrap `claude` command to record all I/O without user intervention
2. **Automatic Management**: Handle log rotation, cleanup, and organization automatically
3. **Minimal Overhead**: Keep performance impact minimal
4. **Cross-Platform**: Support both Linux and macOS initially
5. **User-Friendly**: Provide helpful commands for searching and managing sessions

## Architecture

### Core Components

```
┌─────────────────────────────────────────┐
│           claude-record                 │
│                                         │
│  ┌───────────────────────────────────┐ │
│  │   Argument Parsing & Validation   │ │
│  └──────────────┬────────────────────┘ │
│                 │                       │
│  ┌──────────────▼────────────────────┐ │
│  │    Session Management             │ │
│  │  - setup_log_directory()          │ │
│  │  - generate_filename()            │ │
│  │  - create_metadata()              │ │
│  └──────────────┬────────────────────┘ │
│                 │                       │
│  ┌──────────────▼────────────────────┐ │
│  │    Recording Engine               │ │
│  │  - script command wrapper         │ │
│  │  - filter_claude_ui_sequences()   │ │
│  └──────────────┬────────────────────┘ │
│                 │                       │
│  ┌──────────────▼────────────────────┐ │
│  │    Cleanup & Maintenance          │ │
│  │  - cleanup_old_sessions()         │ │
│  │  - rotate_log_if_needed()         │ │
│  └──────────────┬────────────────────┘ │
│                 │                       │
│  ┌──────────────▼────────────────────┐ │
│  │    Search & Discovery             │ │
│  │  - list_sessions()                │ │
│  │  - search_sessions()              │ │
│  └───────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

## File Structure

### Session Files

Each session creates two files:

**Log File**: `claude_conversation.YYYYMMDD-HHMMSS.log`
- Contains filtered session transcript
- ANSI escape sequences removed
- UI noise filtered out
- Readable plain text format

**Metadata File**: `claude_conversation.YYYYMMDD-HHMMSS.meta`
- Session start/end times
- Session duration
- File size
- Working directory
- Command arguments
- System information

### Directory Layout

```
~/Documents/claude/
├── claude_conversation.20251023-234848.log
├── claude_conversation.20251023-234848.meta
├── claude_conversation.20251022-154321.log
├── claude_conversation.20251022-154321.meta
└── ...
```

## Key Functions

### 1. Session Recording

**Function**: `main()` - Lines 552-671

**Flow**:
1. Parse command-line arguments
2. Validate configuration
3. Setup log directory
4. Auto-cleanup old sessions (if enabled)
5. Generate session filename with timestamp
6. Create metadata file
7. Wrap `claude` command with `script`
8. Filter raw output to clean log
9. Update metadata with end time and size
10. Display summary to user

**Platform Differences**:
- **macOS**: Uses `script -a "$raw_log" "$CLAUDE_CMD"`
- **Linux**: Uses `script -B "$raw_log" -a -c "$CLAUDE_CMD"`

### 2. Log Filtering

**Function**: `filter_claude_ui_sequences()` - Lines 285-327

Removes UI noise to keep logs readable:
- ANSI color codes and escape sequences
- Cursor positioning commands
- Repetitive UI box drawing characters
- Excessive blank lines

**Implementation**:
```bash
sed -E '
    s/\x1b\[[0-9;]*[mGKH]//g    # Color codes
    s/\x1b\[[0-9]*[ABCD]//g      # Cursor movements
    s/\x1b\[2K//g                # Clear line
    ...'
```

Results in ~30-50% log size reduction while preserving content.

### 3. Metadata Management

**Function**: `create_metadata()` - Lines 206-227

Creates metadata file with:
```bash
session_start="2025-10-23T23:48:48+00:00"
session_file="claude_conversation.20251023-234848.log"
session_directory="/Users/dave.logan/Documents/claude"
claude_args=""
max_size="100M"
hostname="macbook.local"
user="dave.logan"
pwd="/Users/dave.logan/projects"
script_version="1.0"
```

**Function**: `update_metadata()` - Lines 230-244

Appends at session end:
```bash
session_end="2025-10-24T00:34:20+00:00"
final_size="45632"
duration_seconds="2732"  # Linux only
```

### 4. Automatic Cleanup

**Function**: `cleanup_old_sessions()` - Lines 329-359

**Strategy**: Keep N most recent sessions (default: 50)

**Process**:
1. Find all `*.log` files in log directory
2. Sort by filename (which includes timestamp)
3. If count > MAX_SESSIONS:
   - Calculate files to remove
   - Delete oldest files and their metadata
4. Log cleanup actions

**Configuration**:
- `--max-sessions N`: Set retention limit
- `--no-auto-cleanup`: Disable automatic cleanup
- `--cleanup`: Manual cleanup operation

### 5. Search Functionality

**Function**: `search_sessions()` - Lines 417-446

**Implementation**:
- Grep-based search through all `.log` files
- Case-insensitive matching
- Shows context (2 lines before/after match)
- Displays up to 20 matching lines per session
- Reports total sessions searched and matches found

**Usage**:
```bash
claude-record --search "API design"
```

### 6. Session Listing

**Function**: `list_sessions()` - Lines 361-415

**Display Format**:
```
SESSION              DATE         SIZE     DURATION             ARGS
-------              ----         ----     --------             ----
claude_conversat...  2025-10-23   44.6K    45m32s
claude_conversat...  2025-10-22   123.4K   1h23m15s            --model claude-3-sonnet
```

**Data Sources**:
- File size: `stat` command (BSD/GNU compatible)
- Duration: Calculated from metadata timestamps
- Arguments: From metadata file

## Platform Compatibility

### macOS Specifics

**Date Command**:
```bash
date -Iseconds  # BSD date supports ISO8601
```

**Stat Command**:
```bash
stat -f%z "$file"  # BSD stat syntax
```

**Script Command**:
```bash
script -a "$raw_log" "$CLAUDE_CMD"
# No -B flag (not supported on macOS)
```

### Linux Specifics

**Stat Command**:
```bash
stat -c%s "$file"  # GNU stat syntax
```

**Script Command**:
```bash
script -B "$raw_log" -a -c "$CLAUDE_CMD"
# -B flag for flush mode
# -c flag to specify command
```

**Duration Calculation**:
```bash
# GNU date supports date arithmetic
duration=$(($(date -d "$end_time" +%s) - $(date -d "$start_time" +%s)))
```

### Cross-Platform Handling

**Detection Strategy**: Use fallback syntax
```bash
# Works on both platforms
stat -f%z "$file" 2>/dev/null || stat -c%s "$file" 2>/dev/null
```

## Configuration

### Command-Line Options

```bash
-h, --help              # Show help
-d, --directory DIR     # Custom log directory
-f, --filename NAME     # Custom filename prefix
-s, --max-size SIZE     # Max file size before rotation
-m, --max-sessions NUM  # Max sessions to keep
-v, --verbose           # Verbose output
--dry-run              # Show what would happen
--cleanup              # Clean up old sessions
--list                 # List all sessions
--search TERM          # Search sessions
--no-auto-cleanup      # Disable auto-cleanup
--version              # Show version
```

### Environment Variables

```bash
CLAUDE_RECORD_DIR       # Override default log directory
CLAUDE_RECORD_MAX_SIZE  # Override default max size
CLAUDE_CMD              # Override claude command
```

### Defaults

```bash
DEFAULT_LOG_DIR="$HOME/Documents/claude"
DEFAULT_MAX_SIZE="100M"
DEFAULT_MAX_SESSIONS=50
CLAUDE_CMD="claude"
```

## Error Handling

### Graceful Degradation

1. **Missing Log Directory**: Auto-create with error if fails
2. **Permission Issues**: Check write permissions before recording
3. **Missing Claude Command**: Exit with helpful error message
4. **Signal Handling**: Trap INT/TERM for clean shutdown

### Signal Handling

```bash
trap 'echo; log_info "Session interrupted by user"; exit 130' INT TERM
```

Ensures:
- Log file is closed properly
- Metadata is updated
- User sees clean exit message
- Exit code 130 (128 + SIGINT)

## Limitations in v1.0

1. **No Summarization**: Logs are raw transcripts, not summarized
2. **Basic Search**: Simple grep, no keyword indexing
3. **No Context Awareness**: Each session is independent
4. **Manual Review**: User must read logs to find information
5. **Size Management Only**: Rotation based on size, not content analysis
6. **Limited Metadata**: Basic session info, no semantic tags

## Performance Characteristics

### Recording Overhead

- **CPU**: Minimal (<1% during session)
- **Memory**: ~10-20MB for script wrapper
- **Disk I/O**: Sequential writes, minimal impact
- **Latency**: No noticeable delay in interaction

### Filtering Performance

- **Time**: ~1-2 seconds for typical session (5-10MB)
- **Size Reduction**: 30-50% on average
- **Quality**: Preserves all content, removes UI noise

### Search Performance

- **Linear Search**: O(n * m) where n=sessions, m=avg size
- **Typical Time**: <5 seconds for 50 sessions
- **Bottleneck**: Grep through all files sequentially

## Design Decisions

### Why Bash?

1. **Universality**: Available on all Unix-like systems
2. **Simplicity**: No dependencies beyond standard utilities
3. **Integration**: Easy to wrap system commands
4. **Portability**: Works on Linux, macOS, BSD

### Why `script` Command?

1. **Native Tool**: Available by default on Unix systems
2. **Complete Recording**: Captures all terminal I/O
3. **PTY Handling**: Proper pseudo-terminal allocation
4. **Reliable**: Battle-tested utility

### Why Flat File Structure?

1. **Simplicity**: Easy to understand and manage
2. **No Dependencies**: No database required
3. **Portability**: Easy to backup, sync, transfer
4. **Transparency**: Users can read files directly

### Why Metadata Files?

1. **Separation**: Metadata separate from content
2. **Shell-Friendly**: Simple key="value" format
3. **Extensibility**: Easy to add new fields
4. **Human-Readable**: Can be sourced or grepped

## Testing Strategy

### Manual Testing Checklist

```
[ ] Fresh installation on clean machine
[ ] Recording a basic session
[ ] Session with arguments passed to Claude
[ ] Very long session (>1 hour)
[ ] Large session (>10MB)
[ ] Signal handling (Ctrl+C during session)
[ ] Auto-cleanup triggers correctly
[ ] Search finds expected content
[ ] List displays all sessions
[ ] Metadata is accurate
[ ] Permissions handled correctly
[ ] Platform-specific commands work
```

### Platform Testing

- **Linux**: Ubuntu 22.04, Debian 12
- **macOS**: Latest version
- **Shells**: bash, zsh

## Future Enhancements (Addressed in v2.0)

1. **Summarization**: AI-generated session summaries
2. **Indexing**: Fast keyword-based search
3. **Context Retrieval**: Surface relevant past sessions
4. **Semantic Search**: Vector-based similarity matching
5. **Windows Support**: PowerShell implementation
6. **Configuration System**: JSON-based config file

## Conclusion

v1.0 provides a solid foundation for session recording with:
- Reliable recording across platforms
- Automatic management and cleanup
- Basic search capabilities
- Minimal overhead and dependencies

The architecture is designed to be extended with intelligent features in v2.0 while maintaining backward compatibility.
