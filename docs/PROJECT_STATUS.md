# Claude Record - Project Status & Memory

**Last Updated**: 2025-10-24
**Version**: 2.0-dev (In Active Development)
**Repository**: `~/src/claude-record/`

---

## Project Overview

Claude Record is an intelligent wrapper for Claude Code that provides automatic session recording, AI-powered summarization, and contextual memory across conversations. This document tracks project status, design decisions, and serves as a "memory" for resuming development across different systems.

---

## Current Status

### ✅ Completed

- [x] **v1.0 Implementation** - Fully functional session recording
  - Session recording with `script` command
  - Log filtering and cleanup
  - Metadata tracking
  - Basic search functionality
  - Cross-platform (Linux/macOS)

- [x] **v2.0 Architecture Design** - Complete specification
  - Detailed architecture documented in `docs/ARCHITECTURE.md`
  - Summary system design
  - Indexing system design
  - Context retrieval strategy
  - Cross-platform approach
  - Error handling strategy

- [x] **Project Infrastructure** - GitHub-ready structure
  - Directory structure created
  - Git repository initialized
  - README.md written
  - LICENSE (MIT) created
  - CONTRIBUTING.md created
  - .gitignore configured
  - Documentation structure established

- [x] **Documentation** - Comprehensive docs
  - `docs/V1_DESIGN.md` - v1.0 design documentation
  - `docs/ARCHITECTURE.md` - v2.0 technical specification
  - `docs/PROJECT_STATUS.md` - This file (project memory)
  - `README.md` - User-facing documentation
  - `CONTRIBUTING.md` - Contribution guidelines

### 🚧 In Progress

- [ ] **Configuration System** - Create config structure
  - Need to create default config.json template
  - Implement config loading functions
  - Add config validation

- [ ] **Phase 1: Core Summarization** - Prototype implementation
  - Create summarization functions
  - Implement Claude Code method
  - Add timeout handling
  - Integrate with main script

### 📋 Planned (Upcoming)

- [ ] **Pseudocode Documentation** - Detailed function specs
  - Write pseudocode for all new functions
  - Document function signatures
  - Create call flow diagrams

- [ ] **Phase 1 Prototype** - Build working summarization
  - Implement core functions
  - Add to existing v1.0 script
  - Test on macOS

- [ ] **Phase 1 Testing** - Validate on macOS
  - Test summary generation
  - Test timeout handling
  - Test integration with existing features
  - Document any issues

- [ ] **Phase 2: Keyword Indexing** - Index system
  - Implement keyword extraction
  - Create index files
  - Build update logic
  - Add rebuild functionality

- [ ] **Phase 3: Context Retrieval** - Smart context
  - Implement retrieval strategies
  - Create display functions
  - Integrate with session start

- [ ] **Phase 4: Cross-Platform** - Windows support
  - Test on Linux
  - Test on Windows (Git Bash)
  - Create PowerShell version
  - Platform-specific fixes

---

## Design Decisions Summary

### Key Architectural Decisions

| Decision | Rationale | Status |
|----------|-----------|--------|
| **Auto-summarize by default** | Minimal friction, users benefit automatically | ✅ Decided |
| **`--no-summarize` to opt-out** | Allow users to skip when needed | ✅ Decided |
| **Claude Code self-summarization** | Uses existing Claude access, no API setup | ✅ Decided |
| **30-second timeout** | Balance quality vs. user experience | ✅ Decided |
| **Markdown summary format** | Human-readable, easy to edit | ✅ Decided |
| **JSON indexes** | Simple, no database dependency | ✅ Decided |
| **Show last 3 sessions by default** | Provides context without overwhelming | ✅ Decided |
| **Graceful degradation** | Never break core recording | ✅ Decided |
| **Local-only storage** | Privacy-first approach | ✅ Decided |
| **Incremental index updates** | Performance optimization | ✅ Decided |

### File Structure Decisions

| File/Directory | Purpose | Status |
|----------------|---------|--------|
| `*.summary` files | AI-generated summaries (Markdown) | ✅ Decided |
| `.claude-record/config.json` | User configuration | ✅ Decided |
| `.claude-record/keyword-index.json` | Keyword → sessions mapping | ✅ Decided |
| `.claude-record/session-index.json` | Fast session metadata lookup | ✅ Decided |
| `.claude-record/embeddings/` | Optional vector search (future) | ✅ Decided |

### Implementation Decisions

| Component | Method | Status |
|-----------|--------|--------|
| **Primary summarization** | Claude Code self-summarization | ✅ Decided |
| **Fallback summarization** | Direct API call (if API key available) | ✅ Decided |
| **Keyword extraction** | Multiple sources: summary tags, file extensions, tech keywords | ✅ Decided |
| **Context retrieval mode** | "Recent" by default, with "smart" and "keyword" options | ✅ Decided |
| **Error handling** | Non-fatal failures, informative messages | ✅ Decided |
| **Platform detection** | Runtime detection with fallback commands | ✅ Decided |

---

## Implementation Phases

### Phase 1: Core Summarization (v2.0-alpha)

**Goal**: Automatic AI-powered summaries on session end

**Deliverables**:
- [ ] Summary generation function (`generate_summary()`)
- [ ] Claude Code method implementation
- [ ] API fallback method implementation
- [ ] Timeout handling (30 seconds)
- [ ] Summary prompt template
- [ ] Integration with main script
- [ ] `--no-summarize` flag
- [ ] `--summarize-only` command
- [ ] Metadata update (add `has_summary` field)
- [ ] Testing on macOS

**Files Modified**:
- `src/claude-record` - Add summarization functions

**New Files**:
- None (all integrated into main script)

**Testing**:
- [ ] Summary generated after normal session
- [ ] Summary skipped with `--no-summarize`
- [ ] Timeout works correctly
- [ ] Summary has all required sections
- [ ] Metadata updated properly

**Success Criteria**:
- Users get automatic summaries after sessions
- Summaries are concise and useful
- Failures are graceful and informative

---

### Phase 2: Keyword Indexing (v2.0-beta)

**Goal**: Fast keyword-based search and discovery

**Deliverables**:
- [ ] `.claude-record/` directory setup
- [ ] Default config.json creation
- [ ] Keyword extraction function
- [ ] Index file structure
- [ ] Index update functions
- [ ] Index rebuild command
- [ ] `--show-index` command
- [ ] `--list-topics` command
- [ ] Integration with cleanup (remove from index)

**Files Created**:
- `.claude-record/config.json`
- `.claude-record/keyword-index.json`
- `.claude-record/session-index.json`

**Testing**:
- [ ] Indexes created on first v2.0 run
- [ ] Keywords extracted correctly
- [ ] Indexes updated after each session
- [ ] Rebuild works for all existing sessions
- [ ] Index lookup is fast

**Success Criteria**:
- Sessions are indexed automatically
- Search is significantly faster than grep
- Indexes stay consistent

---

### Phase 3: Context Retrieval (v2.0-rc)

**Goal**: Automatic context surfacing at session start

**Deliverables**:
- [ ] Context retrieval functions
- [ ] Recent sessions mode
- [ ] Smart context mode (recent + directory)
- [ ] Keyword search mode
- [ ] Pretty display formatting
- [ ] `--no-context` flag
- [ ] `--smart-context` flag
- [ ] `--context` keyword search
- [ ] `--recent N` option

**Testing**:
- [ ] Recent sessions displayed correctly
- [ ] Smart mode finds relevant sessions
- [ ] Keyword mode works
- [ ] Display is clean and helpful
- [ ] Can be disabled

**Success Criteria**:
- Users see relevant context before sessions
- Context is actually useful (not noise)
- Performance is acceptable (< 2s)

---

### Phase 4: Cross-Platform (v2.0)

**Goal**: Full support for Linux, macOS, Windows

**Deliverables**:
- [ ] Platform detection function
- [ ] Platform-specific command wrappers
- [ ] Windows Git Bash testing
- [ ] Windows PowerShell script (claude-record.ps1)
- [ ] Updated installer for all platforms
- [ ] Platform-specific documentation

**Testing**:
- [ ] Works on Ubuntu Linux
- [ ] Works on macOS
- [ ] Works on Windows Git Bash
- [ ] Works on Windows PowerShell
- [ ] All features consistent across platforms

**Success Criteria**:
- Same user experience on all platforms
- No platform-specific bugs
- Clear installation docs for each platform

---

## Development Setup

### Prerequisites

- Bash 4.0+
- Claude Code installed
- Standard Unix tools: grep, sed, awk, find, stat
- Optional: jq (for JSON manipulation)

### Repository Structure

```
claude-record/
├── .git/                   # Git repository
├── .gitignore             # Git ignore rules
├── LICENSE                # MIT License
├── README.md              # User documentation
├── CONTRIBUTING.md        # Contribution guidelines
├── docs/                  # Documentation
│   ├── ARCHITECTURE.md    # v2.0 technical design
│   ├── V1_DESIGN.md       # v1.0 documentation
│   ├── PROJECT_STATUS.md  # This file
│   ├── CONFIGURATION.md   # Config reference (TODO)
│   ├── MIGRATION.md       # v1.0→v2.0 guide (TODO)
│   └── TROUBLESHOOTING.md # Common issues (TODO)
├── src/                   # Source code
│   └── claude-record      # Main script (v1.0 currently)
├── tests/                 # Test scripts
│   └── (TODO)
├── examples/              # Example configs, sessions
│   └── (TODO)
├── scripts/               # Utility scripts
│   └── install.sh         # Installation script (TODO)
└── .github/               # GitHub config
    └── workflows/         # CI/CD (future)
```

### Current Branch

- **main**: v1.0 stable
- **develop**: v2.0 development (to be created)

### Testing Locally

```bash
# Clone repository
cd ~/src/claude-record

# Make changes to src/claude-record

# Test locally
./src/claude-record --help

# Install to local bin (for testing)
mkdir -p ~/.local/bin
cp src/claude-record ~/.local/bin/
chmod +x ~/.local/bin/claude-record

# Test installed version
claude-record --version
```

---

## Next Steps (Immediate)

### Step 1: Create Configuration Structure

**Task**: Implement config.json system

**Actions**:
1. Create default config template
2. Add `setup_config_directory()` function
3. Add `load_config()` function
4. Add `init_indexes()` function
5. Test config loading

**Files**:
- Modify: `src/claude-record`

**Estimated Time**: 1-2 hours

---

### Step 2: Write Pseudocode

**Task**: Document all new functions in detail

**Actions**:
1. Write pseudocode for summarization functions
2. Write pseudocode for indexing functions
3. Write pseudocode for retrieval functions
4. Document function signatures and parameters
5. Create call flow diagrams

**Files**:
- Create: `docs/PSEUDOCODE.md`

**Estimated Time**: 2-3 hours

---

### Step 3: Build Phase 1 Prototype

**Task**: Implement core summarization

**Actions**:
1. Add `generate_summary()` function
2. Add `generate_summary_claude_code()` method
3. Add `generate_summary_with_timeout()` wrapper
4. Add `get_summary_prompt()` template
5. Integrate with `main()` function
6. Add `--no-summarize` flag parsing
7. Add `--summarize-only` command

**Files**:
- Modify: `src/claude-record`

**Estimated Time**: 3-4 hours

---

### Step 4: Test Phase 1 on macOS

**Task**: Validate summarization works

**Actions**:
1. Test normal session → summary generated
2. Test `--no-summarize` → no summary
3. Test timeout handling
4. Test summary quality
5. Test error cases
6. Document any issues

**Estimated Time**: 1-2 hours

---

## Known Issues & Gotchas

### macOS Specifics

- `script` command doesn't support `-B` flag (flush mode)
- Use `script -a "$log" "$command"` syntax instead of `-c`
- BSD `stat` uses `-f%z` instead of `-c%s`
- Duration calculation needs different approach (no `date -d`)

### Linux Specifics

- `script` command supports `-B` flag for flush mode
- Use `-c "command"` to specify command
- GNU `stat` uses `-c%s` for file size

### Windows Specifics

- Git Bash has most Unix tools but `script` might be missing
- PowerShell needs completely different syntax
- Path handling differences (backslash vs forward slash)

### General

- Very large log files (>10MB) slow down summarization
- Long sessions (>1 hour) create large context for Claude
- Network issues can cause API fallback to fail
- jq might not be installed by default

---

## Environment-Specific Notes

### macOS Development (Current)

- **System**: macOS (Darwin 24.6.0)
- **Location**: `/Users/dave.logan/src/claude-record/`
- **Claude Install**: NPM global install
- **Testing**: Primary development platform

### Linux Testing

- **System**: Ubuntu 22.04 or Debian 12 (recommended)
- **Location**: TBD when testing
- **Note**: Test both `-B` flag behavior and duration calculation

### Windows Testing

- **System**: Windows 11
- **Shell**: Git Bash (primary), PowerShell (secondary)
- **Location**: TBD when testing
- **Note**: May need fallback for missing `script` command

---

## Questions & Decisions Pending

### Open Questions

1. **Summary size limits**: Should we truncate very large summaries?
   - **Current**: No limit
   - **Consider**: 500-word limit with "see log for details"

2. **API rate limiting**: How to handle Anthropic API rate limits?
   - **Current**: Not addressed
   - **Consider**: Exponential backoff, fallback to no summary

3. **Summary editing**: Should users be able to edit summaries?
   - **Current**: Not supported
   - **Consider**: `--edit-summary` command to open in $EDITOR

4. **Vector embeddings**: When to implement?
   - **Current**: Deferred to v2.1+
   - **Consider**: After v2.0 stable, assess demand

5. **External sync**: Should summaries sync to cloud?
   - **Current**: Local only
   - **Consider**: Optional Obsidian/Notion export in v2.1+

### Decisions Needed

- None currently blocking - proceed with Phase 1

---

## Resources & References

### Documentation

- [Claude Code Docs](https://docs.claude.com/en/docs/claude-code)
- [Bash Best Practices](https://google.github.io/styleguide/shellguide.html)
- [JSON Processing with jq](https://stedolan.github.io/jq/)

### Related Projects

- [tte](https://github.com/turbot/tte) - Terminal recording
- [asciinema](https://asciinema.org/) - Terminal session recording
- [AI Shell](https://github.com/BuilderIO/ai-shell) - AI-powered shell assistant

### Dependencies

- **Required**: bash, grep, sed, awk, find, stat, date
- **Optional**: jq (JSON manipulation)
- **Runtime**: Claude Code (claude command)

---

## Git Workflow

### Branching Strategy

```bash
main        # Stable releases (v1.0, v2.0)
  └─ develop  # Active development (v2.0-dev)
      ├─ feature/summarization    # Phase 1
      ├─ feature/indexing         # Phase 2
      ├─ feature/context-retrieval # Phase 3
      └─ feature/windows-support  # Phase 4
```

### Commit Message Format

```
<type>: <subject>

<body>

<footer>
```

**Types**: feat, fix, docs, style, refactor, test, chore

**Example**:
```
feat: Add automatic session summarization

- Implement generate_summary() function
- Add Claude Code method with timeout
- Add --no-summarize flag
- Update metadata to track summary status

Closes #1
```

---

## Version History

- **v1.0** (2025-10-23) - Initial release with recording, filtering, search
- **v2.0-dev** (2025-10-24) - In development: summarization, indexing, context

---

## Contact & Support

- **Repository**: (To be published on GitHub)
- **Issues**: (GitHub Issues when published)
- **Discussions**: (GitHub Discussions when published)

---

## Session Memory: Resume from Here

**For Future Claude Sessions**: When resuming this project, read:

1. **This file** (`docs/PROJECT_STATUS.md`) - Current status
2. **ARCHITECTURE.md** - Technical design
3. **V1_DESIGN.md** - Existing v1.0 code understanding
4. **Current code** - `src/claude-record` (v1.0 baseline)

**Current Priority**: **Step 1 - Create Configuration Structure**

**Next Action**: Create config.json template and loading functions

**Working Directory**: `~/src/claude-record/`

---

*Last Updated: 2025-10-24 by Claude Code*
*This document serves as "memory" across development sessions*
