# Platform Support - claude-record v2.0

**Last Updated**: 2025-10-27
**Version**: 2.0-alpha

---

## Supported Platforms

claude-record v2.0 is designed to work across multiple platforms:

- ✅ **macOS** - Fully tested and working
- ⚠️ **Linux** - Compatible, ready for testing
- ⚠️ **Windows (Git Bash)** - Compatible, ready for testing

---

## Platform Compatibility Matrix

| Feature | macOS | Linux | Windows (Git Bash) |
|---------|-------|-------|-------------------|
| Session recording | ✅ Tested | ⚠️ Should work | ⚠️ Should work |
| Log filtering | ✅ Tested | ⚠️ Should work | ⚠️ Should work |
| AI summarization | ✅ Tested | ⚠️ Should work | ⚠️ Should work |
| Keyword indexing | ✅ Tested | ⚠️ Should work | ⚠️ Should work |
| Context retrieval | ✅ Tested | ⚠️ Should work | ⚠️ Should work |
| Date calculations | ✅ Cross-platform | ✅ Cross-platform | ✅ Cross-platform |

Legend:
- ✅ Fully tested and working
- ⚠️ Code is compatible, awaiting testing
- ❌ Not supported

---

## Platform-Specific Implementation

### Date/Time Handling

**Challenge**: BSD date (macOS) vs GNU date (Linux) have different syntax

**Solution**: Cross-platform `date_to_epoch()` function
- macOS: Uses `date -j -f` (BSD format)
- Linux: Uses `date -d` (GNU format)
- Windows: Uses `date -d` (Git Bash provides GNU date)

```bash
# Automatically detects platform and uses correct syntax
date_to_epoch "2025-10-27T10:00:00"
```

### Platform Detection

Automatic detection at script startup:

```bash
PLATFORM=$(detect_platform)
# Returns: "macos", "linux", "windows", or "unknown"
```

Based on `uname -s`:
- `Darwin*` → macOS
- `Linux*` → Linux
- `CYGWIN*|MINGW*|MSYS*` → Windows

---

## macOS (Fully Tested)

### Status
✅ **Production Ready**

### Tested On
- macOS 14.x (Sonnet)
- Darwin 24.6.0

### Dependencies
- bash 3.2+ (built-in)
- Claude Code installed
- jq (optional, for indexing/context)
  ```bash
  brew install jq
  ```

### Known Limitations
None - all features working

---

## Linux (Ready for Testing)

### Status
⚠️ **Code Compatible, Needs Testing**

### Expected Compatibility
- **Ubuntu** 20.04+
- **Debian** 10+
- **Fedora** 33+
- **RHEL/CentOS** 8+
- Any distro with bash 4.0+

### Dependencies
```bash
# Debian/Ubuntu
sudo apt-get update
sudo apt-get install jq

# RHEL/CentOS/Fedora
sudo yum install jq
# or
sudo dnf install jq

# Arch
sudo pacman -S jq
```

### Platform Differences
1. **Date command**: GNU date (uses `date -d`)
   - ✅ Handled automatically by `date_to_epoch()`

2. **Stat command**: GNU stat (uses `stat -c%s`)
   - ✅ Already has fallback: `stat -f%z || stat -c%s`

3. **Script command**: GNU script
   - ⚠️ May have different flags than macOS
   - ⚠️ May need testing for `-B` flag behavior

### Testing Checklist
- [ ] Install claude-record
- [ ] Run `claude-record --version`
- [ ] Test basic recording
- [ ] Test `--summarize-only`
- [ ] Test `--rebuild-index`
- [ ] Test context retrieval
- [ ] Verify all dates calculate correctly
- [ ] Check time_ago formatting

---

## Windows (Ready for Testing)

### Status
⚠️ **Code Compatible via Git Bash, Needs Testing**

### Requirements
- **Git for Windows** (includes Git Bash)
- **Claude Code** installed
- **jq** for Windows

### Installation

1. **Install Git for Windows**:
   ```
   https://git-scm.com/download/win
   ```

2. **Install jq**:
   - Download from: https://jqlang.github.io/jq/download/
   - Place `jq.exe` in PATH or Git Bash bin directory

3. **Install claude-record**:
   ```bash
   # In Git Bash
   git clone https://github.com/davidglogan/claude-record.git
   cd claude-record
   chmod +x src/claude-record

   # Optional: Add to PATH
   cp src/claude-record /usr/local/bin/
   ```

### Platform Differences
1. **Line endings**: Git Bash handles automatically
2. **Paths**: Use Unix-style paths in Git Bash
3. **Date command**: Git Bash provides GNU date
   - ✅ Handled by `date_to_epoch()`

### Limitations
- **PowerShell**: Not directly supported
  - Use Git Bash instead
  - Native PowerShell version could be created (future)

### Testing Checklist
- [ ] Install Git Bash and jq
- [ ] Clone repository
- [ ] Test `--version`
- [ ] Test basic recording
- [ ] Test summarization
- [ ] Test indexing
- [ ] Verify paths work correctly
- [ ] Check context display

---

## Dependencies

### Required
- **bash** 3.2+ (macOS) or 4.0+ (Linux/Windows)
- **Claude Code** - For AI features
- **Standard Unix tools**: grep, sed, awk, cut, sort, etc.

### Optional
- **jq** - For indexing and context retrieval
  - Without jq: Summarization still works, indexing/context skipped gracefully

### Checking Dependencies
```bash
# Check bash version
bash --version

# Check if Claude Code is available
which claude

# Check if jq is available
which jq
jq --version
```

---

## Installation

### macOS
```bash
# Using Homebrew (recommended)
brew install jq

# Clone repository
git clone https://github.com/davidglogan/claude-record.git
cd claude-record

# Make executable
chmod +x src/claude-record

# Optional: Install to PATH
sudo cp src/claude-record /usr/local/bin/
```

### Linux
```bash
# Install jq (example for Ubuntu/Debian)
sudo apt-get install jq

# Clone repository
git clone https://github.com/davidglogan/claude-record.git
cd claude-record

# Make executable
chmod +x src/claude-record

# Optional: Install to PATH
sudo cp src/claude-record /usr/local/bin/
# or for user only:
mkdir -p ~/.local/bin
cp src/claude-record ~/.local/bin/
```

### Windows (Git Bash)
```bash
# In Git Bash
# Install jq first (see Windows section above)

# Clone repository
git clone https://github.com/davidglogan/claude-record.git
cd claude-record

# Make executable
chmod +x src/claude-record

# Run from directory or add to PATH
```

---

## Troubleshooting

### "date: invalid date" errors

**Problem**: Date conversion failing

**Solution**: Platform detection may be incorrect
```bash
# Check detected platform
./src/claude-record --dry-run | grep -i platform

# Manual workaround: Force platform
# Edit src/claude-record and set:
# PLATFORM="linux"  # or "macos" or "windows"
```

### "jq: command not found"

**Problem**: jq not installed

**Solution**:
- Indexing and context features will be skipped
- Install jq to enable (see platform-specific instructions above)
- Or run with `--no-context` to suppress warnings

### Script recording issues

**Problem**: Session not recording properly

**Linux-specific**:
```bash
# Check script command version
script --version

# May need different flags than macOS
# Report issue with your distro/version
```

---

## Performance

### Expected Performance (All Platforms)
- **Session recording**: Negligible overhead
- **Summary generation**: 5-15 seconds
- **Index rebuild**: ~1 second per session
- **Context retrieval**: <0.5 seconds

### Known Performance Issues
None reported

---

## Reporting Issues

If you encounter platform-specific issues:

1. **Check versions**:
   ```bash
   bash --version
   uname -a
   claude --version
   jq --version
   ```

2. **Run with verbose**:
   ```bash
   ./src/claude-record --dry-run --verbose
   ```

3. **Report**:
   - Platform and version
   - Error messages
   - Steps to reproduce
   - Output of commands above

---

## Future Plans

### Planned Improvements
- **Native Windows PowerShell version** - Direct Windows support
- **Automated cross-platform testing** - CI/CD pipeline
- **Platform-specific optimizations** - Performance tuning

### Community Testing Needed
- Linux distributions (Ubuntu, Fedora, Arch, etc.)
- Windows 10/11 with Git Bash
- Older macOS versions

---

## Contributing

Platform testing contributions welcome!

To test on your platform:
1. Install dependencies
2. Clone repository
3. Run test suite (see TESTING.md)
4. Report results

See CONTRIBUTING.md for details.

---

**Status**: Phase 4 (Cross-Platform) Complete
**Tested Platforms**: macOS
**Ready for Testing**: Linux, Windows
**Last Updated**: 2025-10-27
