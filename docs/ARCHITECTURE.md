# Claude Record v2.0: Architecture & Design Specifications

## Overview

Claude Record v2.0 introduces intelligent memory and context awareness to session recording through AI-powered summarization, keyword indexing, and automatic context retrieval. This document provides comprehensive technical specifications for the v2.0 architecture.

## Table of Contents

1. [Design Philosophy](#design-philosophy)
2. [System Architecture](#system-architecture)
3. [File Structure](#file-structure)
4. [Summary System](#summary-system)
5. [Indexing System](#indexing-system)
6. [Context Retrieval](#context-retrieval)
7. [Configuration](#configuration)
8. [Cross-Platform Support](#cross-platform-support)
9. [Command-Line Interface](#command-line-interface)
10. [Performance](#performance)
11. [Error Handling](#error-handling)
12. [Testing Strategy](#testing-strategy)

---

## Design Philosophy

### Core Principles

1. **Auto-summarization by default** - Summaries generated automatically unless explicitly disabled
2. **Zero-friction experience** - Works without user intervention
3. **Graceful degradation** - Core features work even if advanced features fail
4. **Privacy-first** - All data stays local by default
5. **Cross-platform** - Consistent experience on Linux, macOS, Windows
6. **Backward compatible** - v1.0 sessions remain functional

### Design Goals

- Provide "memory" across Claude sessions
- Surface relevant context automatically
- Enable fast discovery of past work
- Maintain minimal performance overhead
- Support offline operation

---

## System Architecture

### Component Diagram

```
┌────────────────────────────────────────────────────────────┐
│                     claude-record v2.0                     │
└────────────────────────────────────────────────────────────┘
                              │
                ┌─────────────┼─────────────┐
                │             │             │
      ┌─────────▼───────┐ ┌──▼──────┐ ┌───▼────────┐
      │   Recording     │ │ Summary │ │  Indexing  │
      │   Engine (v1)   │ │  System │ │   System   │
      └─────────┬───────┘ └──┬──────┘ └───┬────────┘
                │             │             │
                └─────────────┼─────────────┘
                              │
                    ┌─────────▼────────┐
                    │    Context       │
                    │    Retrieval     │
                    └──────────────────┘
```

### Data Flow

```
Session Start
     │
     ▼
┌─────────────────────┐
│ Retrieve Context    │ ← Read summaries from past sessions
│ (show to user)      │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│ Record Session      │ ← v1.0 recording engine
│ (using script cmd)  │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│ Filter Log          │ ← Remove UI noise
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│ Generate Summary    │ ← Call Claude to summarize
│ (AI-powered)        │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│ Extract Keywords    │ ← Parse summary for tags
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│ Update Indexes      │ ← Add to keyword/session indexes
└─────────────────────┘
```

---

## File Structure

### Directory Layout

```
~/Documents/claude/
├── claude_conversation.20251023-234848.log       # Session transcript
├── claude_conversation.20251023-234848.meta      # Metadata
├── claude_conversation.20251023-234848.summary   # AI summary (NEW)
├── .claude-record/                               # Config & indexes (NEW)
│   ├── config.json                               # User preferences
│   ├── keyword-index.json                        # Keyword → sessions
│   ├── session-index.json                        # Session metadata
│   └── embeddings/                               # Optional (future)
│       ├── embeddings.npy
│       └── embedding-metadata.json
└── summaries/                                    # Optional rollups (future)
    ├── weekly-summary-2025-W43.md
    └── monthly-summary-2025-10.md
```

### File Formats

#### Summary File (*.summary)

**Format**: Markdown
**Purpose**: Human-readable AI-generated summary

```markdown
# Session Summary
**Session ID**: claude_conversation.20251023-234848
**Date**: 2025-10-23 23:48:48
**Duration**: 45m 32s
**Model**: claude-sonnet-4-5

## Overview
Implemented REST API authentication endpoint with JWT token support
and added comprehensive test suite.

## Tasks Completed
- Created /api/auth/login endpoint with email/password validation
- Implemented JWT token generation and validation middleware
- Added bcrypt password hashing
- Created test suite with 15 test cases covering happy and error paths
- Updated API documentation

## Key Decisions & Insights
- Chose JWT over sessions for stateless authentication
- Set token expiry to 24 hours with refresh token support
- Used bcrypt with cost factor 12 for password hashing

## Files Modified
- `/src/api/auth.py` - New authentication endpoint
- `/src/middleware/jwt.py` - Token validation middleware
- `/tests/test_auth.py` - Comprehensive test suite
- `/docs/api.md` - Updated authentication docs

## Technologies/Topics
`python`, `flask`, `jwt`, `authentication`, `api-design`, `testing`, `bcrypt`

## Context for Future Reference
The authentication system uses a two-token approach (access + refresh).
Access tokens expire in 24h, refresh tokens in 30 days. Password reset
functionality is planned for next sprint.

## Follow-up Items
- [ ] Implement password reset flow
- [ ] Add rate limiting to login endpoint
- [ ] Setup email verification for new accounts

## Related Sessions
- claude_conversation.20251022-154321 (API architecture planning)

---
*Generated by claude-record v2.0*
```

#### Keyword Index (.claude-record/keyword-index.json)

**Format**: JSON
**Purpose**: Fast keyword → sessions lookup

```json
{
  "index_version": "2.0",
  "last_updated": "2025-10-23T23:53:12Z",
  "total_sessions": 42,
  "keywords": {
    "python": {
      "sessions": [
        "claude_conversation.20251023-234848",
        "claude_conversation.20251022-154321",
        "claude_conversation.20251021-093015"
      ],
      "frequency": 15,
      "last_used": "2025-10-23T23:53:12Z"
    },
    "api-design": {
      "sessions": [
        "claude_conversation.20251023-234848",
        "claude_conversation.20251022-154321"
      ],
      "frequency": 7,
      "last_used": "2025-10-23T23:53:12Z"
    },
    "authentication": {
      "sessions": [
        "claude_conversation.20251023-234848"
      ],
      "frequency": 1,
      "last_used": "2025-10-23T23:53:12Z"
    }
  }
}
```

#### Session Index (.claude-record/session-index.json)

**Format**: JSON
**Purpose**: Fast session metadata lookup

```json
{
  "index_version": "2.0",
  "last_updated": "2025-10-23T23:53:12Z",
  "sessions": {
    "claude_conversation.20251023-234848": {
      "date": "2025-10-23T23:48:48Z",
      "duration_seconds": 2732,
      "size_bytes": 45632,
      "has_summary": true,
      "keywords": ["python", "flask", "jwt", "authentication", "api-design", "testing", "bcrypt"],
      "summary_excerpt": "Implemented REST API authentication endpoint with JWT token support...",
      "files_modified": [
        "/src/api/auth.py",
        "/src/middleware/jwt.py",
        "/tests/test_auth.py",
        "/docs/api.md"
      ],
      "model": "claude-sonnet-4-5",
      "working_directory": "/Users/dave.logan/projects/myapi",
      "related_sessions": ["claude_conversation.20251022-154321"]
    }
  }
}
```

---

## Summary System

### Overview

Automatically generates AI-powered summaries at the end of each session using Claude to analyze the session log and create structured summaries.

### Implementation

#### When to Summarize

**Primary**: Post-session (synchronous)
- Generate immediately after session ends
- Show progress indicator to user
- 30-second timeout with graceful failure

**Alternative**: Background mode (async)
- Spawn detached background process
- User exits immediately
- Summary appears in next session's context

**Configuration**: `config.json` → `summarization.enabled` (default: true)

#### Summarization Methods

**Method 1: Claude Code Self-Summarization** (Primary)

Call Claude Code to read and summarize its own session:

```bash
generate_summary_claude_code() {
    local log_file="$1"
    local summary_file="$2"

    # Create prompt
    local prompt=$(get_summary_prompt "$log_file")

    # Call claude with prompt, redirect output
    echo "$prompt" | claude --print 2>/dev/null > "$summary_file"

    return $?
}
```

**Advantages**:
- Uses existing Claude Code installation
- No separate API key needed
- Consistent with user's Claude access
- Reliable and accurate

**Disadvantages**:
- Adds 10-30 seconds to session end
- Requires Claude Code to be working
- Meta/recursive (Claude summarizing Claude)

**Method 2: Direct API Call** (Fallback)

Direct Anthropic API call:

```bash
generate_summary_api() {
    local log_file="$1"
    local summary_file="$2"

    # Requires ANTHROPIC_API_KEY
    curl -X POST https://api.anthropic.com/v1/messages \
      -H "x-api-key: $ANTHROPIC_API_KEY" \
      -H "content-type: application/json" \
      -d "{
        \"model\": \"claude-sonnet-4-5-20250929\",
        \"max_tokens\": 2048,
        \"messages\": [{
          \"role\": \"user\",
          \"content\": \"$(get_summary_prompt "$log_file")\"
        }]
      }" > "$summary_file"
}
```

**Advantages**:
- Doesn't require Claude Code
- Can run in background
- Faster (no CLI startup overhead)

**Disadvantages**:
- Requires API key configuration
- Direct billing implications
- Additional setup step

**Method 3: Local LLM** (Future)

For privacy-focused users:

```bash
# Using Ollama, llama.cpp, etc.
ollama run llama3.2 "$(get_summary_prompt "$log_file")" > "$summary_file"
```

#### Summarization Prompt

**Template Location**: Function `get_summary_prompt()`

```markdown
You are analyzing a Claude Code session log. Generate a concise summary
following this EXACT format (use markdown):

# Session Summary
**Session ID**: {session_id}
**Date**: {date}
**Duration**: {duration}
**Model**: {model_from_metadata}

## Overview
[2-3 sentences describing what was accomplished]

## Tasks Completed
- [Specific task with outcome]
- [Another task with outcome]

## Key Decisions & Insights
- [Important decision or learning]
- [Another insight]

## Files Modified
- `/path/to/file` - Brief description
- `/another/file` - Brief description

## Technologies/Topics
[Comma-separated keywords in backticks: `python`, `api`, `testing`]

## Context for Future Reference
[Important context that would help understand this session later.
Include architectural decisions, gotchas, or warnings.]

## Follow-up Items
- [ ] Incomplete item or future enhancement
- [ ] Another follow-up

## Related Sessions
[If you can identify related past sessions from context, list them]

---

IMPORTANT GUIDELINES:
- Keep summary concise but informative (aim for 200-400 words total)
- Focus on WHAT was done and WHY, not detailed HOW
- Extract key technical concepts for searchability
- Be specific about file paths and changes
- Identify clear action items from the session
- Use consistent formatting

Session log follows:
---
{log_contents}
---
```

#### Timeout & Error Handling

```bash
generate_summary_with_timeout() {
    local log_file="$1"
    local summary_file="$2"
    local timeout_seconds=30

    # Run summarization in background
    (generate_summary_claude_code "$log_file" "$summary_file") &
    local pid=$!

    # Wait with timeout and progress indicator
    local count=0
    while kill -0 $pid 2>/dev/null; do
        sleep 1
        ((count++))

        # Progress indicator
        if (( count % 5 == 0 )); then
            verbose_log "Generating summary... ${count}s"
        fi

        # Timeout check
        if (( count >= timeout_seconds )); then
            kill -9 $pid 2>/dev/null
            log_warning "Summary generation timed out after ${timeout_seconds}s"
            log_info "You can generate it later with: claude-record --summarize-only"
            return 1
        fi
    done

    wait $pid
    local exit_code=$?

    if [[ $exit_code -eq 0 ]] && [[ -s "$summary_file" ]]; then
        log_success "Summary generated successfully"
        return 0
    else
        log_warning "Summary generation failed"
        return 1
    fi
}
```

#### Integration Points

**In main() function**:

```bash
# After session recording completes and log is filtered
if [[ "$AUTO_SUMMARIZE" == "true" ]]; then
    log_info "Generating session summary..."

    local summary_file="${log_file%.log}.summary"

    if generate_summary_with_timeout "$log_file" "$summary_file"; then
        # Update metadata
        local meta_file="${log_file%.log}.meta"
        echo "has_summary=\"true\"" >> "$meta_file"

        # Update indexes
        update_indexes "$log_file" "$summary_file"
    else
        # Non-fatal error, session still recorded
        log_warning "Session recorded without summary"
    fi
fi
```

---

## Indexing System

### Overview

Keyword-based indexing enables fast search across sessions. Indexes are built incrementally and stored as JSON files.

### Index Types

1. **Keyword Index**: Maps keywords → session IDs
2. **Session Index**: Maps session IDs → metadata
3. **File Index**: Maps file paths → sessions (future)
4. **Vector Index**: Semantic embeddings (future, optional)

### Keyword Extraction

#### Sources

1. **Summary Tags**: Explicit `Technologies/Topics` section (most reliable)
2. **File Extensions**: Programming languages from file paths
3. **Common Terms**: Predefined tech keyword list
4. **Error Messages**: Extract from stack traces/errors

#### Extraction Algorithm

```bash
extract_keywords() {
    local summary_file="$1"
    local log_file="$2"
    local -a keywords=()

    # 1. Extract explicit tags from summary (highest priority)
    if [[ -f "$summary_file" ]]; then
        local tags=$(grep -A 1 "^## Technologies/Topics" "$summary_file" | \
                     tail -1 | \
                     sed 's/`//g' | \
                     tr ',' '\n' | \
                     sed 's/^[[:space:]]*//;s/[[:space:]]*$//')
        keywords+=($tags)
    fi

    # 2. Extract programming languages from file extensions
    if [[ -f "$log_file" ]]; then
        local extensions=$(grep -o '\.[a-zA-Z0-9]\{2,4\}' "$log_file" | \
                          sort -u | \
                          sed 's/^\.//')
        # Map common extensions to languages
        for ext in $extensions; do
            case "$ext" in
                py)   keywords+=("python") ;;
                js)   keywords+=("javascript") ;;
                ts)   keywords+=("typescript") ;;
                go)   keywords+=("golang") ;;
                rs)   keywords+=("rust") ;;
                java) keywords+=("java") ;;
                # ... more mappings
            esac
        done
    fi

    # 3. Search for common tech keywords
    local tech_patterns=(
        "docker" "kubernetes" "k8s" "aws" "gcp" "azure"
        "git" "api" "rest" "graphql" "database" "sql"
        "react" "vue" "angular" "node" "express" "flask"
        "django" "fastapi" "pytest" "jest" "testing"
        "authentication" "authorization" "oauth" "jwt"
    )

    for pattern in "${tech_patterns[@]}"; do
        if grep -q -i "\b$pattern\b" "$log_file" 2>/dev/null; then
            keywords+=("$pattern")
        fi
    done

    # 4. Deduplicate and clean
    printf '%s\n' "${keywords[@]}" | sort -u | tr '\n' ' '
}
```

### Index Update

#### Update Flow

```bash
update_indexes() {
    local log_file="$1"
    local summary_file="$2"
    local session_id=$(basename "$log_file" .log)

    # Extract keywords
    local keywords=$(extract_keywords "$summary_file" "$log_file")

    # Read metadata
    local meta_file="${log_file%.log}.meta"
    local date duration size
    date=$(grep 'session_start=' "$meta_file" | cut -d'"' -f2)
    duration=$(grep 'duration_seconds=' "$meta_file" | cut -d'"' -f2 || echo "0")
    size=$(stat -f%z "$log_file" 2>/dev/null || stat -c%s "$log_file" 2>/dev/null)

    # Update session index
    update_session_index "$session_id" "$keywords" "$date" "$duration" "$size" "$summary_file"

    # Update keyword index
    update_keyword_index "$session_id" $keywords
}
```

#### Session Index Update

```bash
update_session_index() {
    local session_id="$1"
    local keywords="$2"
    local date="$3"
    local duration="$4"
    local size="$5"
    local summary_file="$6"

    local index_file="$LOG_DIR/.claude-record/session-index.json"

    # Extract summary excerpt (first line of Overview)
    local excerpt=$(grep -A 1 "^## Overview" "$summary_file" | tail -1 | head -c 100)

    # Extract files modified
    local files=$(grep -A 10 "^## Files Modified" "$summary_file" | \
                  grep '^-' | \
                  sed 's/^- `\([^`]*\)`.*/\1/' | \
                  jq -R . | jq -s .)

    # Build JSON entry
    local entry=$(cat <<EOF
{
  "date": "$date",
  "duration_seconds": $duration,
  "size_bytes": $size,
  "has_summary": true,
  "keywords": [$(echo "$keywords" | sed 's/ /", "/g' | sed 's/^/"/;s/$/"/")],
  "summary_excerpt": "$excerpt",
  "files_modified": $files,
  "working_directory": "$(grep 'pwd=' "$meta_file" | cut -d'"' -f2)"
}
EOF
)

    # Update index using jq (or manual JSON manipulation)
    jq ".sessions[\"$session_id\"] = $entry" "$index_file" > "${index_file}.tmp"
    mv "${index_file}.tmp" "$index_file"
}
```

#### Keyword Index Update

```bash
update_keyword_index() {
    local session_id="$1"
    shift
    local keywords=("$@")

    local index_file="$LOG_DIR/.claude-record/keyword-index.json"

    for keyword in "${keywords[@]}"; do
        # Add session to keyword's session list
        # Increment frequency
        # Update last_used timestamp

        jq ".keywords[\"$keyword\"].sessions += [\"$session_id\"] | \
            .keywords[\"$keyword\"].sessions |= unique | \
            .keywords[\"$keyword\"].frequency = (.keywords[\"$keyword\"].sessions | length) | \
            .keywords[\"$keyword\"].last_used = \"$(date -Iseconds)\"" \
            "$index_file" > "${index_file}.tmp"

        mv "${index_file}.tmp" "$index_file"
    done

    # Update index metadata
    jq ".last_updated = \"$(date -Iseconds)\" | \
        .total_sessions = (.sessions | length)" \
        "$index_file" > "${index_file}.tmp"

    mv "${index_file}.tmp" "$index_file"
}
```

### Index Rebuild

For migration from v1.0 or fixing corrupted indexes:

```bash
rebuild_indexes() {
    log_info "Rebuilding indexes from all sessions..."

    # Initialize empty indexes
    init_indexes

    # Find all sessions with summaries
    local session_count=0
    while IFS= read -r -d $'\0' summary_file; do
        local log_file="${summary_file%.summary}.log"
        local session_id=$(basename "$log_file" .log)

        verbose_log "Indexing: $session_id"

        update_indexes "$log_file" "$summary_file"

        ((session_count++))
    done < <(find "$LOG_DIR" -name "*.summary" -print0 | sort -z)

    log_success "Rebuilt indexes for $session_count sessions"
}
```

---

## Context Retrieval

### Overview

Surface relevant past sessions when starting a new session to provide continuity and context.

### Retrieval Strategies

#### Strategy 1: Recent Sessions (Default)

Show last N sessions (default: 3)

```bash
get_recent_sessions() {
    local num_recent="${1:-3}"
    local -a sessions=()

    # Get all session IDs sorted by date (newest first)
    while IFS= read -r session_id; do
        sessions+=("$session_id")
    done < <(jq -r '.sessions | keys | .[]' "$SESSION_INDEX_FILE" | sort -r | head -n "$num_recent")

    printf '%s\n' "${sessions[@]}"
}
```

#### Strategy 2: Keyword Match

Match current context keywords against index:

```bash
search_by_keywords() {
    local keywords=("$@")
    local -a matching_sessions=()

    # For each keyword, get matching sessions
    for keyword in "${keywords[@]}"; do
        local sessions=$(jq -r ".keywords[\"$keyword\"].sessions[]?" "$KEYWORD_INDEX_FILE" 2>/dev/null)
        matching_sessions+=($sessions)
    done

    # Deduplicate and rank by frequency
    printf '%s\n' "${matching_sessions[@]}" | sort | uniq -c | sort -rn | awk '{print $2}'
}
```

#### Strategy 3: Working Directory

Find sessions that modified files in current directory:

```bash
search_by_directory() {
    local current_dir="$1"
    local -a matching_sessions=()

    # Search session index for files in current directory
    while IFS= read -r session_id; do
        matching_sessions+=("$session_id")
    done < <(jq -r --arg dir "$current_dir" \
        '.sessions | to_entries[] | select(.value.files_modified[]? | startswith($dir)) | .key' \
        "$SESSION_INDEX_FILE")

    printf '%s\n' "${matching_sessions[@]}"
}
```

### Context Display

```bash
display_context() {
    local -a session_ids=("$@")

    if [[ ${#session_ids[@]} -eq 0 ]]; then
        return
    fi

    echo
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "📚 Relevant Past Sessions Found:"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo

    local count=1
    for session_id in "${session_ids[@]}"; do
        # Get session metadata from index
        local date excerpt keywords files
        date=$(jq -r ".sessions[\"$session_id\"].date" "$SESSION_INDEX_FILE" | cut -d'T' -f1)
        excerpt=$(jq -r ".sessions[\"$session_id\"].summary_excerpt" "$SESSION_INDEX_FILE")
        keywords=$(jq -r ".sessions[\"$session_id\"].keywords | join(\", \")" "$SESSION_INDEX_FILE")
        files=$(jq -r ".sessions[\"$session_id\"].files_modified | join(\", \")" "$SESSION_INDEX_FILE")

        # Calculate time ago
        local time_ago=$(calculate_time_ago "$date")

        echo "[$count] $date ($time_ago)"
        echo "    $excerpt"
        echo "    Tags: $keywords"
        [[ -n "$files" ]] && echo "    Files: $files"
        echo

        ((count++))
    done

    echo "💡 These summaries provide context for this session."
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo
}
```

### Integration

```bash
# In main(), before starting Claude session
if [[ "$SHOW_CONTEXT" == "true" ]]; then
    local -a context_sessions=()

    case "$CONTEXT_MODE" in
        recent)
            context_sessions=($(get_recent_sessions "$NUM_RECENT"))
            ;;
        smart)
            # Combine recent + directory-based
            local recent=($(get_recent_sessions 2))
            local dir_based=($(search_by_directory "$(pwd)"))
            context_sessions=("${recent[@]}" "${dir_based[@]}")
            ;;
        keywords)
            context_sessions=($(search_by_keywords "$CONTEXT_KEYWORDS"))
            ;;
    esac

    display_context "${context_sessions[@]}"
fi
```

---

## Configuration

### Configuration File

**Location**: `$LOG_DIR/.claude-record/config.json`

**Default Configuration**:

```json
{
  "version": "2.0",
  "summarization": {
    "enabled": true,
    "method": "claude-code",
    "fallback_method": "api",
    "timeout_seconds": 30,
    "min_session_length_seconds": 60,
    "max_log_size_mb": 10,
    "background_mode": false,
    "api_key_env": "ANTHROPIC_API_KEY"
  },
  "context_retrieval": {
    "enabled": true,
    "mode": "recent",
    "num_recent": 3,
    "show_in_terminal": true,
    "pass_to_claude": false,
    "max_context_tokens": 4000
  },
  "indexing": {
    "enabled": true,
    "auto_update": true,
    "extract_from_files": true,
    "custom_keywords": []
  },
  "performance": {
    "max_log_size_for_summary": "10M",
    "cache_summaries": true
  },
  "platform": {
    "os": "auto",
    "shell": "auto",
    "date_format": "iso8601"
  }
}
```

### Configuration Loading

```bash
load_config() {
    local config_file="$LOG_DIR/.claude-record/config.json"

    # Create default config if doesn't exist
    if [[ ! -f "$config_file" ]]; then
        setup_config_directory
        return
    fi

    # Load configuration using jq
    AUTO_SUMMARIZE=$(jq -r '.summarization.enabled' "$config_file")
    SUMMARY_METHOD=$(jq -r '.summarization.method' "$config_file")
    SUMMARY_TIMEOUT=$(jq -r '.summarization.timeout_seconds' "$config_file")
    SHOW_CONTEXT=$(jq -r '.context_retrieval.enabled' "$config_file")
    CONTEXT_MODE=$(jq -r '.context_retrieval.mode' "$config_file")
    NUM_RECENT=$(jq -r '.context_retrieval.num_recent' "$config_file")
    # ... load other settings
}
```

---

## Cross-Platform Support

### Platform Detection

```bash
detect_platform() {
    local os="" shell_type=""

    case "$(uname -s)" in
        Linux*)
            os="linux"
            shell_type="bash"
            ;;
        Darwin*)
            os="macos"
            shell_type="bash"
            ;;
        CYGWIN*|MINGW*|MSYS*)
            os="windows"
            if [ -n "$BASH_VERSION" ]; then
                shell_type="gitbash"
            else
                shell_type="powershell"
            fi
            ;;
        *)
            os="unknown"
            shell_type="unknown"
            ;;
    esac

    echo "${os}:${shell_type}"
}

PLATFORM=$(detect_platform)
OS="${PLATFORM%:*}"
SHELL_TYPE="${PLATFORM#*:}"
```

### Platform-Specific Implementations

#### Date Commands

```bash
get_iso_date() {
    case "$OS" in
        linux|macos)
            date -Iseconds
            ;;
        windows)
            if command -v date &>/dev/null; then
                date -Iseconds  # Git Bash has GNU date
            else
                powershell -Command "Get-Date -Format 'yyyy-MM-ddTHH:mm:sszzz'"
            fi
            ;;
    esac
}
```

#### File Stats

```bash
get_file_size() {
    local file="$1"
    case "$OS" in
        linux)
            stat -c%s "$file" 2>/dev/null
            ;;
        macos)
            stat -f%z "$file" 2>/dev/null
            ;;
        windows)
            stat -c%s "$file" 2>/dev/null  # Git Bash
            ;;
    esac
}
```

#### Script Command

```bash
record_session() {
    local log_file="$1"
    shift
    local claude_args=("$@")

    case "$OS" in
        linux)
            script -B "$log_file" -a -c "$CLAUDE_CMD ${claude_args[*]}"
            ;;
        macos)
            script -a "$log_file" "$CLAUDE_CMD" "${claude_args[@]}"
            ;;
        windows)
            if command -v script &>/dev/null; then
                script -a "$log_file" "$CLAUDE_CMD" "${claude_args[@]}"
            else
                # Fallback: redirect output
                "$CLAUDE_CMD" "${claude_args[@]}" 2>&1 | tee "$log_file"
            fi
            ;;
    esac
}
```

---

## Command-Line Interface

### New Flags in v2.0

```bash
# Summary Control
--no-summarize              # Skip automatic summary generation
--summarize-only            # Generate summary for last session
--summarize-all             # Bulk generate summaries for old sessions
--summary-method METHOD     # Force specific method (claude-code, api)

# Context Retrieval
--no-context                # Skip context retrieval (quiet mode)
--smart-context             # Intelligent context (recent + directory)
--context KEYWORDS          # Search for specific keywords
--recent N                  # Show N recent sessions

# Index Management
--rebuild-index             # Rebuild all indexes
--show-index                # Show index statistics
--list-topics               # Show all indexed keywords

# Search & Discovery
--search-summaries TERM     # Search through summaries only
--sessions-by-topic TOPIC   # List sessions with specific topic

# Configuration
--setup                     # Interactive setup wizard
--config KEY=VALUE          # Set config value
```

### Usage Examples

```bash
# Basic usage (with all v2.0 features enabled)
claude-record

# Skip summarization for this session
claude-record --no-summarize

# Show more context before starting
claude-record --recent 5

# Smart context mode
claude-record --smart-context

# Search for past sessions about APIs
claude-record --search-summaries "API"

# List all sessions tagged with 'python'
claude-record --sessions-by-topic python

# Generate missing summaries for all old sessions
claude-record --summarize-all

# Rebuild indexes after manual file changes
claude-record --rebuild-index
```

---

## Performance

### Benchmarks (Target)

| Operation | Target Time | Notes |
|-----------|-------------|-------|
| Session start (with context) | < 2s | Including context retrieval |
| Summary generation | < 30s | With timeout |
| Index update | < 500ms | Incremental update |
| Keyword search | < 100ms | With 100+ sessions |
| Context retrieval | < 500ms | Top 3 sessions |
| Index rebuild | < 30s | For 100 sessions |

### Optimization Strategies

1. **Lazy Loading**: Only load indexes when needed
2. **Incremental Updates**: Update only changed data
3. **Caching**: Cache frequently accessed data
4. **Timeout**: Hard limits on long operations
5. **Background**: Optional async summarization

### Storage Overhead

| Component | Size (per session) | Notes |
|-----------|-------------------|-------|
| Log file | 5-50 KB | Filtered, varies by length |
| Metadata | ~500 B | Fixed size |
| Summary | 1-3 KB | Markdown text |
| Index entry | ~200 B | JSON metadata |

**Total overhead per session**: ~1-5 KB (summary + index)
**For 100 sessions**: ~100-500 KB total

---

## Error Handling

### Principles

1. **Never fail core recording** - Summarization/indexing failures are non-fatal
2. **Graceful degradation** - Fall back to simpler methods
3. **Clear error messages** - Tell user what failed and how to fix
4. **Timeout protection** - Hard limits on long operations
5. **Atomic operations** - Use temp files + mv for writes

### Error Scenarios

#### Summary Generation Fails

```bash
if ! generate_summary_with_timeout "$log_file" "$summary_file"; then
    log_warning "Failed to generate summary (timeout or error)"
    log_info "Session recorded successfully. Generate summary later with:"
    log_info "  claude-record --summarize-only"
    # Continue - session is still recorded
fi
```

#### Index Update Fails

```bash
if ! update_indexes "$session_id"; then
    log_warning "Failed to update indexes"
    log_info "Indexes can be rebuilt with: claude-record --rebuild-index"
    # Continue - summary is still saved
fi
```

#### Context Retrieval Fails

```bash
if ! retrieve_context; then
    log_warning "Failed to retrieve context (continuing without it)"
    # Continue - session can still be recorded
fi
```

---

## Testing Strategy

### Test Levels

1. **Unit Tests**: Individual function testing
2. **Integration Tests**: End-to-end workflows
3. **Platform Tests**: Per-OS validation
4. **Performance Tests**: Benchmark verification

### Test Coverage

**Unit Tests**:
- Keyword extraction from various formats
- Index update logic
- JSON manipulation correctness
- Platform detection
- Configuration loading

**Integration Tests**:
- Fresh installation
- v1.0 → v2.0 migration
- Complete session workflow: record → summarize → index → retrieve
- Search and discovery features

**Platform Tests**:
- Linux (Ubuntu 22.04, Debian 12)
- macOS (latest)
- Windows (Git Bash, PowerShell)

### Test Scripts

```bash
tests/
├── run-tests.sh              # Main test runner
├── test-unit.sh              # Unit tests
├── test-integration.sh       # End-to-end tests
├── test-platform.sh          # Platform-specific tests
├── test-performance.sh       # Performance benchmarks
└── fixtures/                 # Test data
    ├── sample-sessions/
    └── expected-outputs/
```

---

## Migration from v1.0

### Automatic Detection

```bash
if [[ ! -f "$LOG_DIR/.claude-record/config.json" ]]; then
    if [[ -f "$LOG_DIR/claude_conversation."*.log ]]; then
        log_info "Detected existing v1.0 sessions"
        log_info "First-time v2.0 run - setting up..."
        migrate_to_v2
    fi
fi
```

### Migration Process

1. Create `.claude-record/` directory
2. Generate default `config.json`
3. Initialize empty indexes
4. Optionally generate summaries for old sessions
5. Build indexes from summaries

### Backward Compatibility

- All v1.0 commands work identically
- v1.0 sessions readable without modification
- Metadata format extended but not broken
- v1.0 users can opt-in to v2.0 features

---

## Future Enhancements (v2.1+)

### Vector Embeddings (Optional)

- Semantic search using sentence embeddings
- Requires Python + sentence-transformers
- Optional dependency, graceful fallback
- Storage in `.claude-record/embeddings/`

### Summary Rollups

- Weekly/monthly consolidated summaries
- Automatic theme detection across sessions
- Project-level insights

### External Integrations

- Obsidian vault export
- Notion database sync
- Slack notifications
- Git commit integration

### MCP Server

- Claude MCP server for session memory
- Enable Claude to query past sessions directly
- Automatic context injection

---

## Conclusion

Claude Record v2.0 transforms session recording from passive logging to active memory. Through AI-powered summarization, intelligent indexing, and automatic context retrieval, it provides Claude Code with continuity across sessions while maintaining the simplicity and reliability of v1.0.

**Key Innovations**:
- ✅ Automatic AI summarization
- ✅ Fast keyword-based search
- ✅ Smart context retrieval
- ✅ Cross-platform support
- ✅ Privacy-focused local storage
- ✅ Graceful error handling
- ✅ Zero-configuration defaults

**Next Steps**: See [PROJECT_STATUS.md](PROJECT_STATUS.md) for implementation roadmap and current progress.
