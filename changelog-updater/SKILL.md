---
name: Changelog Updater
description: Update changelog based on current branch commits using industry best practices
allowedGitCommands:
  - git log
  - git show
  - git diff
  - git rev-parse
  - git branch
  - git merge-base
---

# Changelog Updater Skill

When invoked, this skill analyzes git commits on the current branch and updates the CHANGELOG.md file with well-structured, user-focused entries following industry best practices.

## Instructions

### Step 1: Analyze Current Branch Commits

1. **Identify the base branch** (usually `main` or `master`)
   ```bash
   # Get current branch name
   git rev-parse --abbrev-ref HEAD

   # Check if main or master exists
   git rev-parse --verify main 2>/dev/null || git rev-parse --verify master 2>/dev/null
   ```

2. **Get commit history** since the branch diverged from base:
   ```bash
   # Find merge base and list commits
   git merge-base main HEAD
   git log main..HEAD --pretty=format:"%h - %s (%an, %ar)" --reverse
   ```

3. **Get detailed commit messages** for context:
   ```bash
   git log main..HEAD --format="%H%n%s%n%b%n---" --reverse
   ```

### Step 2: Analyze Commit Content

For each commit, examine:
- **Commit message**: Extract primary change description
- **Commit body**: Look for additional context, breaking changes, or related issues
- **Files changed**: Use `git show --stat <commit-hash>` to understand scope
- **Diff content**: Review actual changes if commit messages are unclear
  ```bash
  git show --stat <commit-hash>
  git diff <commit-hash>^..<commit-hash>
  ```

### Step 3: Categorize Changes

Group changes into these standard categories (from Keep a Changelog):

- **Added**: New features, capabilities, or functionality
- **Changed**: Modifications to existing functionality (non-breaking)
- **Deprecated**: Features that will be removed in future versions
- **Removed**: Features or functionality that have been removed
- **Fixed**: Bug fixes and error corrections
- **Security**: Security vulnerability fixes or improvements

### Step 4: Write User-Focused Entries

For each change, write entries that:
- **Focus on user impact**, not implementation details
- **Use past tense** (Added, Updated, Fixed, Removed, etc.)
- **Be concise** (1-2 lines maximum)
- **Include benefits** when not obvious
- **Reference issue numbers** if available (e.g., `#123`, `GH-456`)
- **Highlight breaking changes** with `**BREAKING:**` prefix

**Good Examples:**
```markdown
- Added dark mode support for improved readability in low-light environments
- Fixed authentication timeout causing unexpected logouts (#234)
- Updated API response format to include pagination metadata
- **BREAKING:** Removed deprecated `legacy_auth` parameter from login endpoint
```

**Bad Examples:**
```markdown
- Updated files
- Fixed bug
- Refactored UserController class to use new pattern
- Changed variable names for clarity
```

### Step 5: Structure the Changelog Entry

Use this format following Keep a Changelog standards:

```markdown
## [Unreleased]

### Added
- New feature with clear benefit description
- Another new capability that users will appreciate

### Changed
- Improvement to existing feature with explanation of what's different
- Update to behavior that users should know about

### Deprecated
- Feature X is deprecated and will be removed in version Y.Z

### Removed
- **BREAKING:** Old feature that no longer exists

### Fixed
- Bug that was causing specific problem (#issue-number)
- Error in specific scenario that now works correctly

### Security
- Vulnerability fix for specific security issue
```

### Step 6: Update CHANGELOG.md

1. **Find CHANGELOG.md** in the repository root (or create if missing)
2. **Locate the `## [Unreleased]` section** (or create it at the top)
3. **Merge new entries** with existing unreleased entries:
   - Combine entries under the same category headings
   - Remove duplicate entries
   - Keep most recent/relevant information
   - Maintain alphabetical or logical ordering within categories
4. **Add date if releasing**: Change `[Unreleased]` to `[X.Y.Z] - YYYY-MM-DD`
5. **Clean up empty sections**: Remove category headers with no entries

### Step 7: Handle Edge Cases

**If CHANGELOG.md doesn't exist:**
- Create a new file with full Keep a Changelog structure
- Include header, explanation, and first entry

**If no meaningful commits found:**
- Inform the user that no changelog-worthy changes were detected
- Suggest they check if they're on the correct branch

**If commit messages are unclear:**
- Group by file changes or patterns
- Use generic but accurate descriptions
- Flag ambiguous commits for user review

**If there are merge commits:**
- Skip pure merge commits (e.g., "Merge branch X into Y")
- Include merge commits that contain actual changes

## Changelog File Template

If creating a new CHANGELOG.md:

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Initial release features

## [1.0.0] - YYYY-MM-DD

### Added
- First stable release
```

## Best Practices

### Writing Style
- **User perspective**: Write for people using the software, not developers
- **Past tense**: "Added feature" not "Add feature"
- **Active voice**: "Fixed bug causing X" not "Bug was fixed"
- **Specific**: "Fixed login timeout after 5 minutes" not "Fixed timeout issue"

### Organization
- **Most important first**: Breaking changes and major features at the top of each category
- **Related changes grouped**: Keep related items together
- **Consistent formatting**: Same structure for all entries
- **Semantic versioning**: Use MAJOR.MINOR.PATCH correctly

### Content Guidelines
- **Skip internal changes**: Don't include refactoring, code style, or test changes unless they affect users
- **Highlight breaking changes**: Make them obvious with `**BREAKING:**` prefix
- **Link to issues/PRs**: Include references like `(#123)` or `(GH-456)`
- **Explain "why" for breaking changes**: Help users understand the reasoning

### Common Patterns

**Feature additions:**
```markdown
- Added [feature] to enable [benefit]
- Added support for [capability] allowing users to [action]
```

**Bug fixes:**
```markdown
- Fixed [issue] that caused [problem] (#issue)
- Fixed [component] error when [condition]
```

**Changes:**
```markdown
- Updated [feature] to improve [aspect]
- Changed [behavior] from [old] to [new] for better [benefit]
```

**Breaking changes:**
```markdown
- **BREAKING:** Removed [feature] (use [alternative] instead)
- **BREAKING:** Changed [API] parameter [name] to [new-name]
```

## Example Workflow

**Input commits:**
```
abc123 - Add dark mode toggle to settings
def456 - Fix crash when opening empty projects
ghi789 - Update API to v2 endpoint
jkl012 - Remove deprecated legacy auth system
mno345 - Refactor internal caching layer
```

**Output changelog:**
```markdown
## [Unreleased]

### Added
- Added dark mode toggle in settings for improved readability

### Changed
- Updated API to v2 endpoint for better performance and reliability

### Removed
- **BREAKING:** Removed legacy authentication system (migrate to OAuth 2.0)

### Fixed
- Fixed application crash when opening empty projects
```

**Note:** The "Refactor internal caching layer" commit is skipped as it's internal and doesn't affect users.

## Anti-Patterns to Avoid

❌ **Don't include implementation details:**
```markdown
- Refactored UserService to use dependency injection
- Updated webpack config for better tree shaking
```

❌ **Don't use present tense:**
```markdown
- Add feature X
- Fix bug Y
```

❌ **Don't be vague:**
```markdown
- Various improvements
- Bug fixes
- Performance updates
```

❌ **Don't skip breaking changes:**
```markdown
- Updated authentication system
  (Should highlight that it's breaking)
```

✅ **Do focus on user impact:**
```markdown
- Added new feature X enabling users to accomplish Y
- Fixed error preventing Z from working
- **BREAKING:** Removed feature A (use feature B instead)
```

## Summary

This skill automates the tedious task of maintaining a changelog by:
1. Analyzing git commits intelligently
2. Categorizing changes by type
3. Writing user-focused descriptions in past tense
4. Following industry-standard format (Keep a Changelog)
5. Highlighting breaking changes
6. Filtering out internal/non-user-facing changes

The result is a professional, maintainable changelog that helps users understand what's changed and why.
