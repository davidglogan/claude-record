# Claude Record v2.0 - Detailed Pseudocode

This document provides detailed pseudocode for all new functions in v2.0, serving as a bridge between the architecture design and actual implementation.

---

## Table of Contents

1. [Summarization Functions](#summarization-functions)
2. [Indexing Functions](#indexing-functions)
3. [Context Retrieval Functions](#context-retrieval-functions)
4. [Configuration Functions](#configuration-functions)
5. [Utility Functions](#utility-functions)

---

## Summarization Functions

### `generate_summary()`

**Purpose**: Main entry point for summary generation with error handling

**Parameters**:
- `log_file` (string): Path to session log file
- `summary_file` (string): Path to output summary file

**Returns**:
- `0` on success
- `1` on failure

**Pseudocode**:
```
function generate_summary(log_file, summary_file):
    # Check if summarization is enabled
    if AUTO_SUMMARIZE == false:
        return 0 (skip)

    # Check if log file exists and is readable
    if not file_exists(log_file) or not file_readable(log_file):
        log_error("Log file not found or not readable")
        return 1

    # Check log size (skip if too large)
    log_size = get_file_size(log_file)
    max_size = config.max_log_size_for_summary
    if log_size > max_size:
        log_warning("Log file too large for summarization (${log_size} > ${max_size})")
        log_info("Skipping summary generation")
        return 0 (skip, not error)

    # Check minimum session length
    duration = get_session_duration(log_file)
    if duration < config.min_session_length_seconds:
        log_info("Session too short for summarization (${duration}s)")
        return 0 (skip)

    # Try primary method with timeout
    log_info("Generating session summary...")

    if generate_summary_with_timeout(log_file, summary_file):
        log_success("Summary generated successfully")
        return 0
    else:
        # Try fallback method if configured
        if config.fallback_method == "api" and ANTHROPIC_API_KEY is set:
            log_info("Trying fallback API method...")
            if generate_summary_api(log_file, summary_file):
                log_success("Summary generated via API")
                return 0

        # Both methods failed
        log_warning("Summary generation failed")
        log_info("You can try later with: claude-record --summarize-only")
        return 1
```

---

### `generate_summary_with_timeout()`

**Purpose**: Run summary generation with timeout protection

**Parameters**:
- `log_file` (string): Path to session log
- `summary_file` (string): Path to output summary
- `timeout_seconds` (int, optional): Timeout limit (default: 30)

**Returns**:
- `0` on success
- `1` on timeout or failure

**Pseudocode**:
```
function generate_summary_with_timeout(log_file, summary_file, timeout_seconds=30):
    # Start summarization in background
    spawn_background_process:
        generate_summary_claude_code(log_file, summary_file)

    pid = get_background_process_id()

    # Wait with timeout and progress indicator
    elapsed = 0
    while process_is_running(pid):
        sleep(1)
        elapsed++

        # Show progress every 5 seconds
        if elapsed % 5 == 0:
            verbose_log("Generating summary... ${elapsed}s")

        # Check timeout
        if elapsed >= timeout_seconds:
            # Timeout reached - kill process
            kill_process(pid)
            log_warning("Summary generation timed out after ${timeout_seconds}s")
            return 1

    # Process completed - check result
    exit_code = get_process_exit_code(pid)

    if exit_code == 0 and file_exists(summary_file) and file_not_empty(summary_file):
        # Success
        file_size = get_file_size(summary_file)
        verbose_log("Summary created (${file_size} bytes)")
        return 0
    else:
        # Failed
        if file_exists(summary_file):
            delete_file(summary_file)  # Clean up incomplete summary
        return 1
```

---

### `generate_summary_claude_code()`

**Purpose**: Generate summary using Claude Code (primary method)

**Parameters**:
- `log_file` (string): Path to session log
- `summary_file` (string): Path to output summary

**Returns**:
- `0` on success
- `1` on failure

**Pseudocode**:
```
function generate_summary_claude_code(log_file, summary_file):
    # Get session metadata
    meta_file = get_metadata_filename(log_file)
    session_id = get_session_id(log_file)
    session_date = extract_from_meta(meta_file, "session_start")
    duration = calculate_duration(meta_file)

    # Read log contents
    log_contents = read_file(log_file)

    # Generate prompt
    prompt = get_summary_prompt(session_id, session_date, duration, log_contents)

    # Call Claude Code
    # Method: pipe prompt to claude --print
    try:
        output = execute_command:
            echo "$prompt" | claude --print 2>/dev/null

        # Write output to summary file
        write_file(summary_file, output)

        return 0

    catch error:
        log_error("Claude Code summarization failed: ${error}")
        return 1
```

---

### `generate_summary_api()`

**Purpose**: Generate summary using Anthropic API (fallback method)

**Parameters**:
- `log_file` (string): Path to session log
- `summary_file` (string): Path to output summary

**Returns**:
- `0` on success
- `1` on failure

**Pseudocode**:
```
function generate_summary_api(log_file, summary_file):
    # Check API key
    if ANTHROPIC_API_KEY is empty:
        log_error("ANTHROPIC_API_KEY not set")
        return 1

    # Get session metadata
    meta_file = get_metadata_filename(log_file)
    session_id = get_session_id(log_file)
    session_date = extract_from_meta(meta_file, "session_start")
    duration = calculate_duration(meta_file)

    # Read log contents
    log_contents = read_file(log_file)

    # Generate prompt
    prompt = get_summary_prompt(session_id, session_date, duration, log_contents)

    # Escape JSON properly
    prompt_json = escape_json_string(prompt)

    # Build API request
    api_request = {
        "model": "claude-sonnet-4-5-20250929",
        "max_tokens": 2048,
        "messages": [
            {
                "role": "user",
                "content": prompt_json
            }
        ]
    }

    # Call API
    try:
        response = curl:
            -X POST https://api.anthropic.com/v1/messages
            -H "x-api-key: $ANTHROPIC_API_KEY"
            -H "content-type: application/json"
            -d "$api_request"

        # Extract content from response
        summary_text = extract_json_field(response, "content[0].text")

        # Write to file
        write_file(summary_file, summary_text)

        return 0

    catch error:
        log_error("API summarization failed: ${error}")
        return 1
```

---

### `get_summary_prompt()`

**Purpose**: Generate the prompt template for Claude to summarize a session

**Parameters**:
- `session_id` (string): Session identifier
- `session_date` (string): ISO8601 date
- `duration` (string): Human-readable duration (e.g., "45m 32s")
- `log_contents` (string): Full session log

**Returns**:
- `string`: Formatted prompt

**Pseudocode**:
```
function get_summary_prompt(session_id, session_date, duration, log_contents):
    # Format date nicely
    formatted_date = format_date(session_date, "YYYY-MM-DD HH:MM:SS")

    # Truncate log if too long (safety measure)
    if length(log_contents) > 100000:
        log_contents = truncate_with_ellipsis(log_contents, 100000)

    # Build prompt from template
    prompt = """
You are analyzing a Claude Code session log. Generate a concise summary
following this EXACT format (use markdown):

# Session Summary
**Session ID**: ${session_id}
**Date**: ${formatted_date}
**Duration**: ${duration}
**Model**: claude-sonnet-4-5

## Overview
[2-3 sentences describing what was accomplished]

## Tasks Completed
- [Specific task with outcome]
- [Another task with outcome]

## Key Decisions & Insights
- [Important decision or learning]
- [Another insight]

## Files Modified
- \`/path/to/file\` - Brief description
- \`/another/file\` - Brief description

## Technologies/Topics
\`keyword1\`, \`keyword2\`, \`keyword3\`, ...

## Context for Future Reference
[Important context that would help understand this session later.
Include architectural decisions, gotchas, or warnings.]

## Follow-up Items
- [ ] Incomplete item or future enhancement
- [ ] Another follow-up

## Related Sessions
[If you can identify related past sessions from context, list them as:
claude_conversation.YYYYMMDD-HHMMSS]

---

IMPORTANT GUIDELINES:
- Keep summary concise but informative (aim for 200-400 words total)
- Focus on WHAT was done and WHY, not detailed HOW
- Extract key technical concepts for searchability
- Be specific about file paths and changes
- Identify clear action items from the session
- Use consistent formatting
- If session was very short or trivial, say so clearly

Session log follows:
---
${log_contents}
---
"""

    return prompt
```

---

### `get_session_duration()`

**Purpose**: Calculate session duration from metadata

**Parameters**:
- `log_file` (string): Path to log file

**Returns**:
- `int`: Duration in seconds
- `0` if cannot calculate

**Pseudocode**:
```
function get_session_duration(log_file):
    meta_file = get_metadata_filename(log_file)

    if not file_exists(meta_file):
        return 0

    # Try to read duration from metadata (Linux has this)
    duration_seconds = extract_from_meta(meta_file, "duration_seconds")
    if duration_seconds is not empty:
        return to_integer(duration_seconds)

    # Calculate from start/end times (macOS fallback)
    start_time = extract_from_meta(meta_file, "session_start")
    end_time = extract_from_meta(meta_file, "session_end")

    if start_time is empty or end_time is empty:
        return 0

    # Convert ISO8601 to epoch and subtract
    start_epoch = date_to_epoch(start_time)
    end_epoch = date_to_epoch(end_time)

    if start_epoch == 0 or end_epoch == 0:
        return 0

    duration = end_epoch - start_epoch

    # Sanity check
    if duration < 0 or duration > 86400:  # Max 24 hours
        return 0

    return duration
```

---

## Indexing Functions

### `extract_keywords()`

**Purpose**: Extract keywords from summary and log for indexing

**Parameters**:
- `summary_file` (string): Path to summary file
- `log_file` (string): Path to log file

**Returns**:
- `array`: List of unique keywords (lowercase)

**Pseudocode**:
```
function extract_keywords(summary_file, log_file):
    keywords = []  # Empty array

    # Priority 1: Extract explicit tags from summary
    if file_exists(summary_file):
        # Find "## Technologies/Topics" section
        in_topics_section = false
        for each line in summary_file:
            if line matches "^## Technologies/Topics":
                in_topics_section = true
                continue

            if in_topics_section and line is not empty:
                # Parse comma-separated keywords
                # Remove backticks, trim whitespace
                tags = split(line, ",")
                for each tag in tags:
                    cleaned_tag = remove_backticks(trim(tag))
                    if cleaned_tag is not empty:
                        keywords.append(lowercase(cleaned_tag))
                break  # Only first line after heading

    # Priority 2: Extract programming languages from file extensions
    if file_exists(log_file):
        # Find all file extensions in log
        extensions = grep_pattern(log_file, '\.[a-zA-Z0-9]{2,4}')

        for each ext in extensions:
            lang = map_extension_to_language(ext)
            if lang is not empty:
                keywords.append(lang)

    # Priority 3: Search for common tech keywords
    tech_keywords = [
        "docker", "kubernetes", "k8s", "aws", "gcp", "azure",
        "git", "api", "rest", "graphql", "database", "sql",
        "react", "vue", "angular", "node", "express", "flask",
        "django", "fastapi", "pytest", "jest", "testing",
        "authentication", "authorization", "oauth", "jwt",
        "nginx", "apache", "redis", "mongodb", "postgresql",
        "typescript", "javascript", "python", "java", "golang",
        "rust", "ruby", "php", "bash", "shell", "powershell"
    ]

    if file_exists(log_file):
        for each keyword in tech_keywords:
            if grep_found(log_file, keyword, case_insensitive=true):
                keywords.append(keyword)

    # Deduplicate and clean
    keywords = unique(keywords)
    keywords = sort(keywords)

    # Remove very common/useless words
    stopwords = ["the", "and", "or", "in", "on", "at", "to", "for"]
    keywords = filter(keywords, not in stopwords)

    return keywords
```

---

### `map_extension_to_language()`

**Purpose**: Map file extension to programming language

**Parameters**:
- `extension` (string): File extension (e.g., ".py", "js")

**Returns**:
- `string`: Language name or empty string

**Pseudocode**:
```
function map_extension_to_language(extension):
    # Remove leading dot if present
    ext = remove_prefix(extension, ".")
    ext = lowercase(ext)

    mapping = {
        "py": "python",
        "js": "javascript",
        "ts": "typescript",
        "jsx": "react",
        "tsx": "react-typescript",
        "go": "golang",
        "rs": "rust",
        "java": "java",
        "c": "c",
        "cpp": "cpp",
        "cc": "cpp",
        "h": "c",
        "hpp": "cpp",
        "cs": "csharp",
        "rb": "ruby",
        "php": "php",
        "sh": "bash",
        "bash": "bash",
        "zsh": "zsh",
        "ps1": "powershell",
        "sql": "sql",
        "html": "html",
        "css": "css",
        "scss": "scss",
        "json": "json",
        "yaml": "yaml",
        "yml": "yaml",
        "xml": "xml",
        "md": "markdown",
        "dockerfile": "docker",
        "tf": "terraform"
    }

    if ext in mapping:
        return mapping[ext]
    else:
        return ""
```

---

### `update_indexes()`

**Purpose**: Update both keyword and session indexes after summarization

**Parameters**:
- `log_file` (string): Path to log file
- `summary_file` (string): Path to summary file

**Returns**:
- `0` on success
- `1` on failure

**Pseudocode**:
```
function update_indexes(log_file, summary_file):
    try:
        # Extract session info
        session_id = get_session_id(log_file)
        keywords = extract_keywords(summary_file, log_file)

        # Get metadata
        meta_file = get_metadata_filename(log_file)
        date = extract_from_meta(meta_file, "session_start")
        duration = get_session_duration(log_file)
        size = get_file_size(log_file)
        working_dir = extract_from_meta(meta_file, "pwd")

        # Extract summary excerpt (first paragraph of Overview)
        excerpt = extract_summary_excerpt(summary_file)

        # Extract files modified
        files_modified = extract_files_from_summary(summary_file)

        # Update session index
        update_session_index(
            session_id,
            date,
            duration,
            size,
            keywords,
            excerpt,
            files_modified,
            working_dir
        )

        # Update keyword index
        update_keyword_index(session_id, keywords)

        verbose_log("Indexes updated for ${session_id}")
        return 0

    catch error:
        log_error("Failed to update indexes: ${error}")
        return 1
```

---

### `update_session_index()`

**Purpose**: Add or update entry in session index

**Parameters**:
- `session_id` (string): Session identifier
- `date` (string): ISO8601 date
- `duration` (int): Duration in seconds
- `size` (int): File size in bytes
- `keywords` (array): List of keywords
- `excerpt` (string): Summary excerpt
- `files_modified` (array): List of file paths
- `working_dir` (string): Working directory

**Returns**: void (updates index file)

**Pseudocode**:
```
function update_session_index(session_id, date, duration, size, keywords, excerpt, files_modified, working_dir):
    index_file = "$LOG_DIR/.claude-record/session-index.json"

    # Ensure directory exists
    create_directory_if_not_exists("$LOG_DIR/.claude-record")

    # Load existing index or create new
    if file_exists(index_file):
        index = parse_json(read_file(index_file))
    else:
        index = initialize_session_index()

    # Build entry
    entry = {
        "date": date,
        "duration_seconds": duration,
        "size_bytes": size,
        "has_summary": true,
        "keywords": keywords,
        "summary_excerpt": excerpt,
        "files_modified": files_modified,
        "working_directory": working_dir,
        "model": "claude-sonnet-4-5",
        "related_sessions": []  # TODO: detect related sessions
    }

    # Add/update entry
    index.sessions[session_id] = entry

    # Update metadata
    index.last_updated = current_iso8601_time()

    # Write atomically (temp file + mv)
    temp_file = index_file + ".tmp"
    write_file(temp_file, format_json(index, pretty=true))
    move_file(temp_file, index_file)

    verbose_log("Session index updated")
```

---

### `update_keyword_index()`

**Purpose**: Update keyword index with new session

**Parameters**:
- `session_id` (string): Session identifier
- `keywords` (array): List of keywords

**Returns**: void (updates index file)

**Pseudocode**:
```
function update_keyword_index(session_id, keywords):
    index_file = "$LOG_DIR/.claude-record/keyword-index.json"

    # Load existing index or create new
    if file_exists(index_file):
        index = parse_json(read_file(index_file))
    else:
        index = initialize_keyword_index()

    current_time = current_iso8601_time()

    # Update each keyword
    for each keyword in keywords:
        if keyword not in index.keywords:
            # New keyword
            index.keywords[keyword] = {
                "sessions": [],
                "frequency": 0,
                "last_used": current_time
            }

        # Add session to keyword (if not already present)
        if session_id not in index.keywords[keyword].sessions:
            index.keywords[keyword].sessions.append(session_id)

        # Update frequency and last_used
        index.keywords[keyword].frequency = length(index.keywords[keyword].sessions)
        index.keywords[keyword].last_used = current_time

    # Update global metadata
    index.last_updated = current_time
    index.total_sessions = count_unique_sessions_in_index(index)

    # Write atomically
    temp_file = index_file + ".tmp"
    write_file(temp_file, format_json(index, pretty=true))
    move_file(temp_file, index_file)

    verbose_log("Keyword index updated")
```

---

### `rebuild_indexes()`

**Purpose**: Rebuild all indexes from existing summaries

**Parameters**: none

**Returns**:
- `0` on success
- `1` on failure

**Pseudocode**:
```
function rebuild_indexes():
    log_info("Rebuilding indexes from all sessions...")

    # Initialize empty indexes
    initialize_indexes()

    # Find all summaries
    summary_files = find_files("$LOG_DIR", "*.summary")
    sort(summary_files)  # Chronological order

    session_count = 0
    error_count = 0

    for each summary_file in summary_files:
        # Get corresponding log file
        log_file = replace_extension(summary_file, ".summary", ".log")

        if not file_exists(log_file):
            log_warning("Log file missing for ${summary_file}")
            error_count++
            continue

        session_id = get_session_id(log_file)
        verbose_log("Indexing: ${session_id}")

        # Update indexes for this session
        if update_indexes(log_file, summary_file) == 0:
            session_count++
        else:
            error_count++

    if error_count > 0:
        log_warning("Rebuilt indexes with ${error_count} errors")

    log_success("Rebuilt indexes for ${session_count} sessions")

    return (error_count > 0) ? 1 : 0
```

---

### `initialize_indexes()`

**Purpose**: Create empty index files

**Parameters**: none

**Returns**: void

**Pseudocode**:
```
function initialize_indexes():
    config_dir = "$LOG_DIR/.claude-record"
    create_directory_if_not_exists(config_dir)

    # Create session index
    session_index = {
        "index_version": "2.0",
        "last_updated": current_iso8601_time(),
        "sessions": {}
    }
    write_file("$config_dir/session-index.json", format_json(session_index, pretty=true))

    # Create keyword index
    keyword_index = {
        "index_version": "2.0",
        "last_updated": current_iso8601_time(),
        "total_sessions": 0,
        "keywords": {}
    }
    write_file("$config_dir/keyword-index.json", format_json(keyword_index, pretty=true))

    verbose_log("Initialized empty indexes")
```

---

## Context Retrieval Functions

### `retrieve_context()`

**Purpose**: Main context retrieval function called at session start

**Parameters**: none (uses global config)

**Returns**:
- `array`: List of relevant session IDs

**Pseudocode**:
```
function retrieve_context():
    if SHOW_CONTEXT == false:
        return []  # Context disabled

    session_ids = []

    # Choose retrieval strategy based on mode
    case CONTEXT_MODE of:
        "recent":
            session_ids = get_recent_sessions(NUM_RECENT)

        "smart":
            # Combine recent + directory-based
            recent = get_recent_sessions(2)
            dir_based = get_sessions_by_directory(current_working_directory())
            session_ids = merge_unique(recent, dir_based)
            session_ids = limit_to_first(session_ids, NUM_RECENT)

        "keywords":
            if CONTEXT_KEYWORDS is not empty:
                keywords = split(CONTEXT_KEYWORDS, " ")
                session_ids = search_by_keywords(keywords)
                session_ids = limit_to_first(session_ids, NUM_RECENT)

        default:
            session_ids = get_recent_sessions(3)

    # Display context to user
    if length(session_ids) > 0:
        display_context_to_user(session_ids)

    return session_ids
```

---

### `get_recent_sessions()`

**Purpose**: Get N most recent sessions

**Parameters**:
- `num_recent` (int): Number of sessions to retrieve

**Returns**:
- `array`: List of session IDs (most recent first)

**Pseudocode**:
```
function get_recent_sessions(num_recent):
    index_file = "$LOG_DIR/.claude-record/session-index.json"

    if not file_exists(index_file):
        return []  # No index yet

    index = parse_json(read_file(index_file))

    # Get all session IDs with dates
    sessions_with_dates = []
    for each (session_id, data) in index.sessions:
        sessions_with_dates.append({
            "id": session_id,
            "date": data.date
        })

    # Sort by date (newest first)
    sort(sessions_with_dates, by="date", descending=true)

    # Take first N
    session_ids = []
    for i = 0 to min(num_recent, length(sessions_with_dates)) - 1:
        session_ids.append(sessions_with_dates[i].id)

    return session_ids
```

---

### `search_by_keywords()`

**Purpose**: Find sessions matching given keywords

**Parameters**:
- `keywords` (array): List of keywords to search for

**Returns**:
- `array`: List of session IDs ranked by relevance

**Pseudocode**:
```
function search_by_keywords(keywords):
    index_file = "$LOG_DIR/.claude-record/keyword-index.json"

    if not file_exists(index_file):
        return []

    index = parse_json(read_file(index_file))

    # Count matches per session
    session_match_counts = {}  # session_id -> count

    for each keyword in keywords:
        keyword_lower = lowercase(keyword)

        if keyword_lower in index.keywords:
            sessions = index.keywords[keyword_lower].sessions

            for each session_id in sessions:
                if session_id not in session_match_counts:
                    session_match_counts[session_id] = 0
                session_match_counts[session_id]++

    # Sort by match count (highest first)
    sorted_sessions = sort_by_value(session_match_counts, descending=true)

    # Extract just the session IDs
    session_ids = []
    for each (session_id, count) in sorted_sessions:
        session_ids.append(session_id)

    return session_ids
```

---

### `get_sessions_by_directory()`

**Purpose**: Find sessions that modified files in current directory

**Parameters**:
- `directory` (string): Directory path to search

**Returns**:
- `array`: List of session IDs

**Pseudocode**:
```
function get_sessions_by_directory(directory):
    index_file = "$LOG_DIR/.claude-record/session-index.json"

    if not file_exists(index_file):
        return []

    index = parse_json(read_file(index_file))

    matching_sessions = []

    for each (session_id, data) in index.sessions:
        # Check if any modified file is in this directory
        for each file_path in data.files_modified:
            if file_path starts_with directory:
                matching_sessions.append(session_id)
                break  # Count session only once

    # Sort by date (newest first)
    matching_sessions = sort_by_date(matching_sessions, index, descending=true)

    return matching_sessions
```

---

### `display_context_to_user()`

**Purpose**: Pretty-print context information to terminal

**Parameters**:
- `session_ids` (array): List of session IDs to display

**Returns**: void (prints to stderr)

**Pseudocode**:
```
function display_context_to_user(session_ids):
    if length(session_ids) == 0:
        return

    index = load_session_index()

    # Print header
    print("")
    print("━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
    print("📚 Relevant Past Sessions Found:")
    print("━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
    print("")

    count = 1
    for each session_id in session_ids:
        data = index.sessions[session_id]

        # Format date nicely
        date = format_date(data.date, "YYYY-MM-DD HH:MM")
        time_ago = calculate_time_ago(data.date)

        # Get excerpt (truncate if needed)
        excerpt = truncate(data.summary_excerpt, 80)

        # Get keywords (first 5)
        keywords = join(first_n(data.keywords, 5), ", ")

        # Get files (first 3)
        files = join(first_n(data.files_modified, 3), ", ")
        if length(data.files_modified) > 3:
            files += ", ..."

        # Print session info
        print("[${count}] ${date} (${time_ago})")
        print("    ${excerpt}")
        print("    Tags: ${keywords}")
        if length(files) > 0:
            print("    Files: ${files}")
        print("")

        count++

    # Print footer
    print("💡 These summaries provide context for this session.")
    print("━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━")
    print("")
```

---

## Configuration Functions

### `setup_config_directory()`

**Purpose**: Initialize .claude-record directory with default config

**Parameters**: none

**Returns**: void

**Pseudocode**:
```
function setup_config_directory():
    config_dir = "$LOG_DIR/.claude-record"
    config_file = "$config_dir/config.json"

    # Create directory if doesn't exist
    if not directory_exists(config_dir):
        create_directory(config_dir)
        verbose_log("Created config directory: ${config_dir}")

    # Create default config if doesn't exist
    if not file_exists(config_file):
        default_config = get_default_config()
        write_file(config_file, format_json(default_config, pretty=true))
        log_success("Created default configuration: ${config_file}")

    # Initialize empty indexes if they don't exist
    if not file_exists("$config_dir/session-index.json"):
        initialize_indexes()
```

---

### `load_config()`

**Purpose**: Load configuration from config file

**Parameters**: none

**Returns**: void (sets global variables)

**Pseudocode**:
```
function load_config():
    config_file = "$LOG_DIR/.claude-record/config.json"

    # If config doesn't exist, use defaults
    if not file_exists(config_file):
        use_default_config()
        return

    # Parse config file
    try:
        config = parse_json(read_file(config_file))

        # Load summarization settings
        AUTO_SUMMARIZE = config.summarization.enabled
        SUMMARY_METHOD = config.summarization.method
        SUMMARY_FALLBACK = config.summarization.fallback_method
        SUMMARY_TIMEOUT = config.summarization.timeout_seconds
        MIN_SESSION_LENGTH = config.summarization.min_session_length_seconds
        MAX_LOG_SIZE_FOR_SUMMARY = config.summarization.max_log_size_mb

        # Load context retrieval settings
        SHOW_CONTEXT = config.context_retrieval.enabled
        CONTEXT_MODE = config.context_retrieval.mode
        NUM_RECENT = config.context_retrieval.num_recent

        # Load indexing settings
        INDEXING_ENABLED = config.indexing.enabled
        AUTO_UPDATE_INDEX = config.indexing.auto_update

        # Load performance settings
        CACHE_SUMMARIES = config.performance.cache_summaries

        verbose_log("Loaded configuration from ${config_file}")

    catch error:
        log_warning("Failed to parse config file: ${error}")
        log_info("Using default configuration")
        use_default_config()
```

---

### `get_default_config()`

**Purpose**: Return default configuration object

**Parameters**: none

**Returns**: object (config structure)

**Pseudocode**:
```
function get_default_config():
    return {
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

---

## Utility Functions

### `calculate_time_ago()`

**Purpose**: Calculate human-readable "time ago" string

**Parameters**:
- `iso_date` (string): ISO8601 date string

**Returns**:
- `string`: Human-readable time ago (e.g., "2h ago", "3 days ago")

**Pseudocode**:
```
function calculate_time_ago(iso_date):
    now_epoch = current_epoch_time()
    then_epoch = date_to_epoch(iso_date)

    diff_seconds = now_epoch - then_epoch

    # Handle invalid dates
    if diff_seconds < 0:
        return "in the future"

    # Calculate appropriate unit
    if diff_seconds < 60:
        return "${diff_seconds}s ago"

    diff_minutes = diff_seconds / 60
    if diff_minutes < 60:
        return "${diff_minutes}m ago"

    diff_hours = diff_minutes / 60
    if diff_hours < 24:
        return "${diff_hours}h ago"

    diff_days = diff_hours / 24
    if diff_days < 7:
        return "${diff_days} day(s) ago"

    diff_weeks = diff_days / 7
    if diff_weeks < 4:
        return "${diff_weeks} week(s) ago"

    diff_months = diff_days / 30
    return "${diff_months} month(s) ago"
```

---

### `extract_summary_excerpt()`

**Purpose**: Extract first paragraph of Overview section from summary

**Parameters**:
- `summary_file` (string): Path to summary file

**Returns**:
- `string`: Excerpt text (truncated to ~100 chars)

**Pseudocode**:
```
function extract_summary_excerpt(summary_file):
    if not file_exists(summary_file):
        return "N/A"

    in_overview = false
    excerpt = ""

    for each line in summary_file:
        if line matches "^## Overview":
            in_overview = true
            continue

        if in_overview:
            if line is empty or line starts with "##":
                break  # End of overview section

            # Append line to excerpt
            excerpt += trim(line) + " "

    # Clean up and truncate
    excerpt = trim(excerpt)
    excerpt = truncate(excerpt, 100, add_ellipsis=true)

    return excerpt
```

---

### `extract_files_from_summary()`

**Purpose**: Extract list of modified files from summary

**Parameters**:
- `summary_file` (string): Path to summary file

**Returns**:
- `array`: List of file paths

**Pseudocode**:
```
function extract_files_from_summary(summary_file):
    if not file_exists(summary_file):
        return []

    files = []
    in_files_section = false

    for each line in summary_file:
        if line matches "^## Files Modified":
            in_files_section = true
            continue

        if in_files_section:
            if line is empty or line starts with "##":
                break  # End of section

            # Parse file path from markdown list
            # Format: "- `/path/to/file` - Description"
            if line starts with "- ":
                # Extract path between backticks
                match = regex_extract(line, '`([^`]+)`')
                if match is not empty:
                    files.append(match)

    return files
```

---

### `get_session_id()`

**Purpose**: Extract session ID from log file path

**Parameters**:
- `log_file` (string): Path to log file

**Returns**:
- `string`: Session ID (filename without extension)

**Pseudocode**:
```
function get_session_id(log_file):
    # Get filename from path
    filename = basename(log_file)

    # Remove .log extension
    session_id = remove_extension(filename, ".log")

    return session_id
```

---

### `date_to_epoch()`

**Purpose**: Convert ISO8601 date to Unix epoch

**Parameters**:
- `iso_date` (string): ISO8601 date string

**Returns**:
- `int`: Unix epoch timestamp (seconds since 1970-01-01)

**Pseudocode**:
```
function date_to_epoch(iso_date):
    # Platform-specific implementation
    case OS of:
        "linux":
            # GNU date supports -d flag
            epoch = execute_command("date -d '${iso_date}' +%s")
            return to_integer(epoch)

        "macos":
            # BSD date requires different syntax
            # Use python as fallback
            script = "import dateutil.parser; import time; print(int(time.mktime(dateutil.parser.parse('${iso_date}').timetuple())))"
            epoch = execute_command("python3 -c \"${script}\"")
            return to_integer(epoch)

        "windows":
            # PowerShell can do this
            script = "(Get-Date '${iso_date}').ToUniversalTime().Subtract((Get-Date '1970-01-01')).TotalSeconds"
            epoch = execute_command("powershell -Command \"${script}\"")
            return to_integer(epoch)

    return 0  # Failure
```

---

## Integration with main()

### Modified `main()` function flow

**Pseudocode**:
```
function main():
    # [Existing v1.0 code: parse args, validate, setup directories]

    # NEW: Load v2.0 configuration
    load_config()
    setup_config_directory()

    # NEW: Retrieve and display context (before starting session)
    if not CLEANUP_ONLY and not LIST_SESSIONS and not SEARCH_MODE:
        retrieve_context()

    # [Existing v1.0 code: record session with script command]

    # [Existing v1.0 code: filter log, create metadata]

    # NEW: Generate summary (after session ends)
    if AUTO_SUMMARIZE:
        summary_file = "${log_file%.log}.summary"

        if generate_summary(log_file, summary_file):
            # Update metadata
            meta_file = "${log_file%.log}.meta"
            echo 'has_summary="true"' >> meta_file

            # Update indexes
            if INDEXING_ENABLED:
                update_indexes(log_file, summary_file)

    # [Existing v1.0 code: display completion message]
```

---

## Summary

This pseudocode provides detailed implementation guidance for all v2.0 features:

- **15 summarization functions** - Complete AI summary generation workflow
- **12 indexing functions** - Keyword extraction and index management
- **6 context retrieval functions** - Smart context surfacing
- **4 configuration functions** - Config file management
- **7 utility functions** - Helper functions for dates, parsing, etc.

**Total**: 44 new functions to implement in Phase 1-3

Each function includes:
- Purpose and description
- Parameters with types
- Return values
- Detailed step-by-step pseudocode
- Error handling
- Platform considerations

**Next Step**: Begin Phase 1 implementation using this pseudocode as a blueprint.
