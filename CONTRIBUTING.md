# Contributing to Claude Record

Thank you for your interest in contributing to Claude Record! This document provides guidelines and instructions for contributing.

## Code of Conduct

- Be respectful and inclusive
- Focus on constructive feedback
- Help maintain a welcoming environment

## How to Contribute

### Reporting Bugs

1. Check if the issue already exists in [Issues](https://github.com/yourusername/claude-record/issues)
2. Create a new issue with:
   - Clear, descriptive title
   - Steps to reproduce
   - Expected vs actual behavior
   - Your environment (OS, shell, claude-record version)
   - Relevant logs (redact sensitive info)

### Suggesting Features

1. Check [Discussions](https://github.com/yourusername/claude-record/discussions) for existing proposals
2. Open a new discussion describing:
   - The problem you're trying to solve
   - Your proposed solution
   - Any alternatives considered
   - Example use cases

### Pull Requests

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes
4. Test on your platform
5. Commit with clear messages (`git commit -m 'Add amazing feature'`)
6. Push to your fork (`git push origin feature/amazing-feature`)
7. Open a Pull Request

#### PR Guidelines

- **One feature per PR** - Keep changes focused
- **Update documentation** - Update relevant .md files
- **Add tests** - Include test cases for new features
- **Cross-platform** - Test on Linux/macOS/Windows if possible
- **Follow style** - Match existing bash script conventions
- **No breaking changes** - Maintain backward compatibility

### Development Workflow

```bash
# 1. Clone your fork
git clone https://github.com/yourusername/claude-record.git
cd claude-record

# 2. Create feature branch
git checkout -b feature/my-feature

# 3. Make changes
# Edit files...

# 4. Test locally
./scripts/install.sh --local
claude-record --version

# 5. Run tests
./tests/run-tests.sh

# 6. Commit and push
git add .
git commit -m "Description of changes"
git push origin feature/my-feature

# 7. Open PR on GitHub
```

## Code Style

### Bash Scripts

- Use `#!/bin/bash` shebang
- Enable strict mode: `set -euo pipefail`
- Use 4-space indentation
- Quote all variables: `"$variable"`
- Use meaningful function and variable names
- Add comments for complex logic
- Follow existing patterns in codebase

Example:
```bash
#!/bin/bash
set -euo pipefail

# Function description
my_function() {
    local input="$1"
    local result=""

    # Process input
    result=$(echo "$input" | tr '[:upper:]' '[:lower:]')

    echo "$result"
}
```

### Documentation

- Use Markdown for all docs
- Keep lines under 100 characters when possible
- Use clear headings and structure
- Include code examples
- Update table of contents if applicable

## Testing

### Manual Testing

Test your changes on:
- Linux (Ubuntu/Debian preferred)
- macOS (current version)
- Windows (Git Bash)

Test scenarios:
- Fresh installation
- Upgrade from v1.0
- Normal session recording
- All command-line flags
- Error conditions

### Automated Testing

Add tests to `tests/` directory:
```bash
tests/
├── test-basic-recording.sh
├── test-summarization.sh
├── test-indexing.sh
└── test-platform-compat.sh
```

Run all tests:
```bash
./tests/run-tests.sh
```

## Documentation Updates

When adding features, update:
- `README.md` - User-facing documentation
- `docs/ARCHITECTURE.md` - Technical details
- `docs/CONFIGURATION.md` - New config options
- `docs/PROJECT_STATUS.md` - Current status
- Code comments - Inline documentation

## Platform-Specific Code

When adding platform-specific code, use detection:

```bash
detect_platform() {
    case "$(uname -s)" in
        Linux*)   echo "linux" ;;
        Darwin*)  echo "macos" ;;
        CYGWIN*|MINGW*|MSYS*) echo "windows" ;;
        *)        echo "unknown" ;;
    esac
}

OS=$(detect_platform)

case "$OS" in
    linux)
        # Linux-specific code
        ;;
    macos)
        # macOS-specific code
        ;;
    windows)
        # Windows-specific code
        ;;
esac
```

## Commit Message Guidelines

Format:
```
<type>: <subject>

<body>

<footer>
```

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting)
- `refactor`: Code refactoring
- `test`: Adding tests
- `chore`: Maintenance tasks

Example:
```
feat: Add automatic summarization on session end

- Implement generate_summary() function
- Add --no-summarize flag to opt out
- Update metadata to track summary status
- Add timeout handling (30 seconds max)

Closes #42
```

## Release Process

(For maintainers)

1. Update version in scripts
2. Update CHANGELOG.md
3. Create git tag: `git tag -a v2.0.0 -m "Release v2.0.0"`
4. Push tag: `git push origin v2.0.0`
5. Create GitHub release with notes
6. Update documentation

## Getting Help

- 📖 Read the [documentation](docs/)
- 💬 Ask in [Discussions](https://github.com/yourusername/claude-record/discussions)
- 🐛 Check existing [Issues](https://github.com/yourusername/claude-record/issues)

## Recognition

Contributors will be recognized in:
- GitHub contributors page
- CHANGELOG.md for significant contributions
- README.md credits section

Thank you for contributing! 🎉
