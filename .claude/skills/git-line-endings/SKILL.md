---
name: git-line-endings
description: Use this skill when git keeps changing files unexpectedly on commit, files show changes even though nothing was edited, or when dealing with CRLF/LF line ending issues across platforms. Provides diagnosis and fixes for git line ending configuration problems.
---

# Git Line Endings Skill

## Quick Start (Read First)

**Symptoms this fixes:**
- Git shows files as modified when nothing changed
- Files change every time you commit
- Warnings like "LF will be replaced by CRLF" or vice versa
- Different line endings between Windows and Unix/Mac systems

**Root cause:** Windows uses CRLF (`\r\n`), Unix/Mac uses LF (`\n`). Git can auto-convert but wrong settings cause thrashing.

**Two main options:**
1. **`core.autocrlf=true`** - Git converts LF→CRLF on checkout, CRLF→LF on commit
2. **`core.autocrlf=false`** - No conversion, `.gitattributes` controls normalization

**When to use each:** → [Decision Guide](#decision-guide)

**Quick fix workflow:** → [Fix Line Ending Issues](#fix-line-ending-issues)

## Quick Reference

**Configuration:**
- [Decision Guide](#decision-guide) - Which autocrlf setting to use
- [Common Configurations](#common-configurations) - Platform-specific setup
- [VS Code Integration](#vs-code-integration) - Match editor to git settings

**Troubleshooting:**
- [Fix Line Ending Issues](#fix-line-ending-issues) - Step-by-step resolution
- [Troubleshooting](#troubleshooting) - Common problems and solutions
- [Testing Your Configuration](#testing-your-configuration) - Verify settings work

**Platform-Specific:**
- [Windows (PowerShell)](#windows-powershell) - Windows-specific commands
- [Mac/Linux (Bash)](#maclinux-bash) - Unix-specific commands

## Decision Guide

### Check Current Setting

```bash
git config core.autocrlf
```

Returns: `true`, `false`, or `input` (or nothing if unset)

### The Two Main Options

#### Option 1: `core.autocrlf=true` (Windows standard)
- **Best for:** Pure Windows teams without `.gitattributes`
- **Behavior:** Converts line endings on checkout/commit
- **Risk:** Conflicts with `.gitattributes` if present

```bash
git config core.autocrlf true
```

#### Option 2: `core.autocrlf=false` + `.gitattributes` (Recommended)
- **Best for:** Cross-platform teams, projects with `.gitattributes`
- **Behavior:** No automatic conversion, `.gitattributes` controls everything
- **Benefit:** Consistent across all platforms

```bash
git config core.autocrlf false
```

**Verification:**
```bash
git config core.autocrlf
# Should show: false
```

## Common Configurations

### Windows + `.gitattributes` (Recommended)
```bash
# Set autocrlf to false
git config core.autocrlf false

# Create .gitattributes if missing
```

`.gitattributes` example:
```properties
# Auto-detect text files, normalize to LF in repo
* text=auto

# Force LF for specific types
*.js text eol=lf
*.ts text eol=lf
*.json text eol=lf
*.md text eol=lf
*.yml text eol=lf

# Binary files
*.png binary
*.jpg binary
*.pdf binary
```

### Windows without `.gitattributes`
```bash
git config core.autocrlf true
```

### Mac/Linux
```bash
git config core.autocrlf input
# Or false if using .gitattributes
```

## Fix Line Ending Issues

### Step 1: Diagnose the Problem

```bash
# Check current autocrlf setting
git config core.autocrlf

# Check if .gitattributes exists
ls .gitattributes  # Unix/Mac
dir .gitattributes  # Windows/PowerShell

# View what git sees as changed
git status --short

# See if it's line endings (will show no actual content changes)
git diff <filename>
```

### Step 2: Check for Conflicts

**Common conflict:** `core.autocrlf=true` + `.gitattributes` with `* text=auto`

This causes git to convert twice, leading to thrashing.

```bash
# If you have .gitattributes, use:
git config core.autocrlf false

# If no .gitattributes and Windows-only team, use:
git config core.autocrlf true
```

### Step 3: Normalize the Repository

After changing settings, normalize all files:

```bash
# Save any work first!
git add . -u
git commit -m "Checkpoint before line ending normalization"

# Remove all files from git index
git rm --cached -r .

# Re-add all files (applies new line ending rules)
git add .

# Check what changed
git status

# Commit the normalization
git commit -m "Normalize line endings"
```

### Step 4: Refresh Working Directory

```bash
# Make sure everything is committed
git status

# Re-checkout with new settings
git rm --cached -r .
git reset --hard
```

## Troubleshooting

### Files Still Changing After Fix

**Check for mixed settings:**
```bash
# Check global config
git config --global core.autocrlf

# Check local config (overrides global)
git config --local core.autocrlf

# Check if .gitattributes is being ignored
cat .gitattributes
```

**Solution:** Make sure local and global settings align:
```bash
# Set globally
git config --global core.autocrlf false

# Set for this repo
git config --local core.autocrlf false
```

### VS Code Shows Different Line Endings Than Git

**Check VS Code settings:**
- Open Settings (Ctrl+,)
- Search for "eol"
- Set `files.eol` to `\n` (LF) or `auto`

**Make sure EOL is consistent:**
```json
// .vscode/settings.json
{
  "files.eol": "\n"
}
```

### Warning: "LF will be replaced by CRLF"

This is transitional. After files stabilize with the new settings, the warnings stop.

**To silence warnings during transition:**
```bash
git config core.safecrlf warn  # or false to completely disable
```

### Binary Files Being Corrupted

**Fix:** Mark them as binary in `.gitattributes`:
```properties
*.png binary
*.jpg binary
*.exe binary
*.dll binary
*.so binary
```

## Platform-Specific Guidance

### Windows (PowerShell)

```powershell
# Check current setting
git config core.autocrlf

# Recommended: Use false + .gitattributes
git config core.autocrlf false

# Check for .gitattributes
Test-Path .gitattributes
```

### Mac/Linux (Bash)

```bash
# Check current setting
git config core.autocrlf

# Recommended: Use input or false
git config core.autocrlf input  # Converts CRLF→LF on commit only
# or
git config core.autocrlf false  # No conversion, .gitattributes controls
```

## VS Code Integration

VS Code typically uses LF (`\n`) by default. To match git settings:

**Option 1: Configure VS Code to use LF**
```json
// .vscode/settings.json
{
  "files.eol": "\n",
  "files.insertFinalNewline": true,
  "files.trimTrailingWhitespace": true
}
```

**Option 2: Let EditorConfig handle it**
```ini
# .editorconfig
root = true

[*]
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true

[*.{cmd,bat}]
end_of_line = crlf
```

## Testing Your Configuration

```bash
# Create a test file with CRLF
echo "line1\r\nline2" > test.txt

# Add and check what git does
git add test.txt

# View what git stored (should be LF if normalized)
git show :test.txt | od -c  # Unix/Mac
git show :test.txt | Format-Hex  # PowerShell

# Clean up
git rm --cached test.txt
rm test.txt
```

## Quick Reference

| Setting | Windows→Repo | Repo→Windows | When to Use |
|---------|--------------|--------------|-------------|
| `true` | CRLF→LF | LF→CRLF | Windows-only, no `.gitattributes` |
| `false` | No change | No change | With `.gitattributes` (cross-platform) |
| `input` | CRLF→LF | No change | Mac/Linux only |

## Additional Resources

- [Git Documentation: gitattributes](https://git-scm.com/docs/gitattributes)
- [GitHub: Handling line endings](https://docs.github.com/en/get-started/getting-started-with-git/configuring-git-to-handle-line-endings)
- [EditorConfig](https://editorconfig.org/)
