---
name: improve-agent
description: Guide for improving GitHub Copilot Agent performance through skills, custom instructions, AGENTS.md, or other configuration. Use this when asked to improve agent behavior, create/update skills, modify custom instructions, update AGENTS.md, or customize GitHub Copilot performance in repositories.
---

# Agent Improvement Guide

Guide for improving GitHub Copilot Agent performance through skills, custom instructions, AGENTS.md, or other methods.

## Quick Reference

**What this skill covers:** Improving GitHub Copilot Agent performance through multiple approaches including Skills, Custom Instructions, AGENTS.md, and path-specific instructions. Learn when to use each method and how to implement them.

**Instruction types:**
- **Skills** = On-demand, specialized (`.github/skills/NAME/SKILL.md`) → [Details](#agent-skills-githubskills)
- **Custom Instructions** = Always-in-context, general (`.github/copilot-instructions.md`) → [Details](#custom-instructions-githubcopilot-instructionsmd)
- **Path-Specific** = Context for specific files/dirs (`.github/instructions/NAME.instructions.md`) → [Details](#path-specific-instructions-githubinstructionsinstructionsmd)
- **AGENTS.md** = Directory-based precedence (nearest `AGENTS.md` wins) → [Details](#agent-instructions-agentsmd)

**Decision:** Need it for 80%+ of tasks? → Custom Instructions. Specific workflows only? → Skill. Different rules per directory? → Path-specific or AGENTS.md.  
→ [See detailed decision guide](#skills-vs-custom-instructions-decision-guide)

**Key principles:**
- **Concise is key**: Context window is shared. Only add what Copilot doesn't already know. Challenge each piece: "Does this justify its token cost?"
- **First 50 lines critical**: Copilot often only reads the start of files. Put essential content first with lots of # references to detailed sections below.
- **Iterative not comprehensive**: Skills grow over time. First draft should be concise and focused, not exhaustive. → [Iteration guide](#step-4-iterate)
- **Match freedom to fragility**: High freedom (text) for flexible tasks, low freedom (specific scripts) for fragile operations. → [Details](#degrees-of-freedom)

**Improvement process:**
0. Identify what needs improvement (agent understanding, task execution, domain knowledge, etc.). Ask the user questions if you aren't sure. → [See Step 0](#step-0-identify-improvement-area)
1. Decide best approach: Custom instructions, skill, AGENTS.md, or path-specific? → [Decision guide](#skills-vs-custom-instructions-decision-guide)
2. **If custom instructions:** Update `.github/copilot-instructions.md` → [Setup guide](#custom-instructions-setup)
3. **If AGENTS.md:** Create/update `AGENTS.md` in relevant directory → [AGENTS.md guide](#agent-instructions-agentsmd)
4. **If skill:** Understand with concrete examples → [See Step 1](#step-1-understand-with-concrete-examples)
5. **If skill:** Create/update SKILL.md file → [See Step 2](#step-2-create-skillmd)
6. **If skill:** Write metadata and body → [See Step 3](#step-3-write-the-skill)
7. Test and iterate based on usage → [See Step 4](#step-4-iterate)

**Writing best practices:** → [Effective instructions](#writing-effective-instructions), [What NOT to include](#what-not-to-include-in-instructions), [Testing approach](#testing-and-iterating), [Troubleshooting](#troubleshooting)

**Naming:** lowercase-with-hyphens, under 64 chars, verb-led (e.g., `rotate-pdf`, `debug-github-actions`)

**Agent improvement methods overview:** → [See all methods](#agent-improvement-methods)

**SKILL.md structure:**
- **YAML frontmatter** (required):
  - `name`: Unique identifier, lowercase with hyphens
  - `description`: What it does AND when Copilot should use it (this is how Copilot decides to load the skill)
  - `license` (optional): License information
- **Markdown body**: Instructions, examples, and guidelines for Copilot to follow → [Writing guide](#writing-effective-skills)

**How Copilot uses skills:** When performing tasks, Copilot decides when to use skills based on your prompt and the skill's description. When Copilot chooses a skill, the SKILL.md file is injected into the agent's context. → [Content tips](#content-organization)

---

## Detailed Guidance

### Agent Improvement Methods

**Skills provide:**
- Specialized workflows for specific domains
- Tool integrations for file formats or APIs
- Domain expertise (schemas, business logic)
- Procedural knowledge and best practices

**Custom instructions provide:**
- Repository-wide coding standards and conventions
- Build, test, and run commands
- Project structure and architecture
- Always-available foundational knowledge

**AGENTS.md provides:**
- Directory-based instruction precedence
- Module-specific agent behavior
- Cross-agent compatibility

**Path-specific instructions provide:**
- File type or directory-specific rules
- Context-dependent coding standards
- Conditional guidance based on file location

### Writing Effective Skills

**First 50 lines are critical:** Copilot often only reads the beginning of files. Structure skills with essential content and navigation first, detailed sections below. Expect Copilot to read the first 50 lines, then jump to specific sections as needed.

**Be iterative:** Don't aim for comprehensive coverage in first draft. Skills improve over time through real usage. Start concise and focused, add details as patterns emerge.

**Keep it focused:** If a skill becomes too large, consider splitting into multiple focused skills rather than one comprehensive skill.

**Description is key:** The `description` field in YAML frontmatter is how Copilot decides whether to use your skill. Be comprehensive and specific about what the skill does and when it should be used.

### Skills vs Custom Instructions: Decision Guide

**CRITICAL: Before creating a skill, determine if custom instructions are more appropriate.**

#### Custom Instructions (`.github/copilot-instructions.md`)

**Use custom instructions when:**
- Information is relevant to **almost every task** in the repository
- Guidance applies broadly across all files and features
- Content describes "how this repository works" in general

**Examples of custom instructions:**
- Coding standards and style preferences
- Build, test, and run commands
- Project structure and architecture overview
- Where to find key files (configs, tests, docs)
- CI/CD pipeline steps and validation requirements
- Common gotchas and workarounds
- Environment setup and dependencies
- Naming conventions and coding patterns

**Benefits:**
- ✅ **Always in context** - No need for Copilot to decide when to load them
- ✅ Perfect for foundational knowledge every task needs
- ✅ Reduces repetitive explanations across multiple skills

**How to create:** → [See Custom Instructions Setup](#custom-instructions-setup)

#### Agent Skills (`.github/skills/`)

**Use skills when:**
- Information is relevant only to **specific tasks or domains**
- Guidance is specialized for particular workflows
- Content would clutter context if always loaded

**Examples of skills:**
- Debugging specific integrations (GitHub Actions, API endpoints)
- Working with specialized file formats (Parquet, PDF, DOCX)
- Domain-specific queries (DuckDB patterns, database schemas)
- Testing patterns for specific frameworks
- Deployment procedures for specific environments

**Benefits:**
- ✅ **Loaded on-demand** - Keeps context lean until needed
- ✅ Perfect for specialized, detailed workflows
- ✅ Can be very detailed without worrying about always using tokens

#### Path-Specific Instructions (`.github/instructions/*.instructions.md`)

**Use path-specific instructions when:**
- Different parts of your codebase have **different rules**
- Specific file types or directories need unique guidance
- You want to avoid cluttering repository-wide instructions with niche rules

**Examples of path-specific instructions:**
- API routes require specific error handling patterns
- Test files have different standards than production code
- Frontend components follow React patterns; backend follows Express patterns
- Database migrations need special naming conventions
- Legacy directories have different rules than new code

**Benefits:**
- ✅ **Automatically applied** when working on matching files
- ✅ Combined with repository-wide instructions (both apply)
- ✅ Can use glob patterns to target specific file types or directories

**How to create:** → [See Path-Specific Instructions](#path-specific-instructions)

#### Agent Instructions (AGENTS.md)

**Use AGENTS.md when:**
- You want **directory-based precedence** (nearest file wins)
- Different subdirectories represent different modules with unique patterns
- Building multi-module projects where each module has distinct rules
- Want portability across different AI agents (not just Copilot)

**Examples of AGENTS.md usage:**
- Monorepo with multiple apps (each with `AGENTS.md`)
- Frontend/backend split requiring different agent behaviors
- Per-package instructions in a workspace
- Module-specific coding patterns

**Benefits:**
- ✅ **Hierarchical precedence** - Closer files override parent instructions
- ✅ Works across multiple AI agents (open standard)
- ✅ Natural mapping to project structure

**How to create:** → [See Agent Instructions](#agent-instructions-agentsmd)

**Decision matrix:**

| Need | Custom Instructions | Skill | Path-Specific | AGENTS.md |
|------|--------------------|---------|--------------------|-----------|
| "How do I run tests?" | ✅ Yes | ❌ No | ❌ No | ❌ No |
| "How to debug GitHub Actions?" | ❌ No | ✅ Yes | ❌ No | ❌ No |
| "What's the project structure?" | ✅ Yes | ❌ No | ❌ No | ❌ No |
| "API routes need special error handling" | ❌ No | ❌ No | ✅ Yes | ✅ Possible |
| "Code style preferences" | ✅ Yes | ❌ No | ❌ No | ❌ No |
| "Frontend uses React, backend uses Express" | ❌ No | ❌ No | ✅ Yes | ✅ Yes |
| "Adding filters to Trend Analysis" | ❌ No | ✅ Yes | ❌ No | ❌ No |

**Rules of thumb:**
- **80%+ of tasks** need this info → Custom instructions
- **Specific workflows only** → Skill
- **Different rules per file type/directory** → Path-specific
- **Different rules per module with hierarchy** → AGENTS.md
- **Prefer path-specific over AGENTS.md** for GitHub Copilot projects (better Copilot integration)

### Custom Instructions Setup

#### Repository-Wide Instructions

Create `.github/copilot-instructions.md` with general guidance for the entire repository:

```bash
# If .github directory doesn't exist:
mkdir .github
New-Item .github/copilot-instructions.md
```

**What to include:**
1. **Repository overview** - What the project does, tech stack, size
2. **Build instructions** - Bootstrap, build, test, run, lint commands with exact steps
3. **Project layout** - Where key files live, architecture overview
4. **Validation steps** - CI checks, pre-commit hooks, manual validation
5. **Common patterns** - Coding standards, naming conventions, file organization
6. **Known issues** - Workarounds, timing requirements, order-dependent commands

**Example structure:**
```markdown
# Project Overview
This is a Next.js claims analytics app with TypeScript, Tailwind CSS, and Highcharts.

# Build & Test Commands
- Install: `npm install`
- Dev server: `npm run dev`
- Tests: `npm test` (runs Jest)
- Coverage: `npm run test:coverage`

# Project Structure
- `app/` - Next.js app directory (pages, components, API routes)
- `__tests__/` - Jest test files
- `.github/skills/` - Copilot agent skills

# Coding Standards
- Use TypeScript strict mode
- Test coverage requirement: >80%
- Always add test command comment to new test files
```

**Pro tip:** Ask Copilot to generate it for you by using the prompt from [GitHub's docs](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-repository-instructions#asking-copilot-coding-agent-to-generate-a-copilot-instructionsmd-file).

**Recommended template structure:**
```markdown
# [Technology or Domain Name] Guidelines

## Purpose
Brief statement of what this file covers and when these instructions apply.

## Naming Conventions
- Rule 1
- Rule 2

## Code Style
- Style rule 1

```javascript
// Example showing correct pattern
const activeUsers = users.filter(user => user.isActive);
```

## Error Handling
- How to handle errors
- What patterns to use

## Security Considerations
- Security rule 1
- Security rule 2

## Testing Guidelines
- Testing expectation 1

## Performance
- Performance consideration 1
```

Adapt this structure to your needs while maintaining clear sectioning and bullet-point format.

#### Path-Specific Instructions

**When to use:** Instructions that apply only to specific files, directories, or file types (not the entire repository).

**Use path-specific instructions when:**
- Different parts of your codebase have different rules (e.g., API routes vs frontend components)
- Specific directories require unique validation or patterns
- You want to avoid cluttering repository-wide instructions with niche guidance

**Examples:**
- API routes must follow specific error handling patterns
- Test files have different coding standards than production code
- Database migration files require special naming conventions
- Legacy code sections have different rules than new code

Create `.github/instructions/NAME.instructions.md` (where NAME describes the purpose):

```bash
mkdir .github/instructions
New-Item .github/instructions/api-routes.instructions.md
```

**Add YAML frontmatter with glob pattern:**
```yaml
---
applyTo: "app/api/**/*.ts"
---

API routes must:
- Use Next.js App Router patterns
- Include proper error handling
- Return typed responses
```

**Glob pattern examples:**
- `**/*.ts` - All TypeScript files
- `app/components/**/*.tsx` - All components
- `__tests__/**/*.test.tsx` - All test files
- `**/database/**/*` - All files in any database directory

**Exclude from specific agents (optional):**
```yaml
---
applyTo: "**/*.py"
excludeAgent: "code-review"  # Only for coding-agent, not code-review
---
```

**When combined:** If a file matches both repository-wide and path-specific instructions, both sets are combined and used together.

#### Writing Effective Instructions

**Length limits:** Keep instruction files under ~1,000 lines maximum. Beyond this, Copilot may overlook some instructions due to context limits.

**Structure best practices:**
- **Use distinct headings** to separate different topics
- **Use bullet points** for easy scanning and reference
- **Write short, imperative directives** rather than long narrative paragraphs
- **Provide concrete examples** showing both correct and incorrect patterns

**Example - Before (vague):**
```markdown
When you're reviewing code, it would be good if you could try to look for
situations where developers might have accidentally left in sensitive
information like passwords or API keys, and also check for security issues.
```

**Example - After (clear):**
```markdown
## Security Critical Issues

- Check for hardcoded secrets, API keys, or credentials
- Look for SQL injection and XSS vulnerabilities
- Verify proper input validation and sanitization
```

**Show correct and incorrect examples:**
```markdown
## Naming Conventions

Use descriptive, intention-revealing names.

```javascript
// Avoid
const d = new Date();
const x = users.filter(u => u.active);

// Prefer
const currentDate = new Date();
const activeUsers = users.filter(user => user.isActive);
```
```

**Understanding Copilot's limitations:**
- **Non-deterministic behavior** - Copilot may not follow every instruction perfectly every time
- **Context limits** - Very long instruction files may result in some instructions being overlooked
- **Specificity matters** - Clear, specific instructions work better than vague directives

#### What NOT to Include in Instructions

Copilot currently **does not support** instructions that attempt to:

**Change user experience or formatting:**
- ❌ "Use bold text for critical issues"
- ❌ "Change the format of review comments"
- ❌ "Add emoji to comments"

**Modify pull request overview comments:**
- ❌ "Include a summary of security issues in the PR overview"
- ❌ "Add a testing checklist to the overview comment"

**Change Copilot's core function:**
- ❌ "Block a PR from merging unless all comments are addressed"
- ❌ "Generate a changelog entry for every PR"

**Follow external links:**
- ❌ "Review this code according to https://example.com/standards"
- ✅ **Workaround:** Copy the relevant content directly into your instruction file

**Vague quality improvements:**
- ❌ "Be more accurate"
- ❌ "Don't miss any issues"
- ❌ "Be consistent in your feedback"

These types of instructions add noise without improving effectiveness.

#### Testing and Iterating

**Start small:** Begin with 10-20 specific instructions addressing your most common needs, then iterate based on results.

**Test with real work:**
1. Save your instruction files
2. Use Copilot on real tasks or PRs
3. Observe which instructions it follows effectively
4. Note instructions that are consistently missed or misinterpreted

**Iterate based on results:**
1. Identify a pattern that Copilot could handle better
2. Add a specific instruction for that pattern
3. Test with new work
4. Refine the instruction based on results

This iterative approach helps you understand what works and keeps files focused.

#### Troubleshooting

**Issue: Instructions are ignored**

Possible causes:
- Instruction file is too long (over 1,000 lines)
- Instructions are vague or ambiguous
- Instructions conflict with each other

Solutions:
- Shorten the file by removing less important instructions
- Rewrite vague instructions to be more specific and actionable
- Review for conflicting instructions and prioritize the most important ones

**Issue: Language-specific rules applied to wrong files**

Possible causes:
- Missing or incorrect `applyTo` frontmatter
- Rules in repository-wide file instead of path-specific file

Solutions:
- Add `applyTo` frontmatter to path-specific instruction files
- Move language-specific rules from `copilot-instructions.md` to appropriate `*.instructions.md` files

**Issue: Inconsistent behavior across sessions**

Possible causes:
- Instructions are too numerous
- Instructions lack specificity
- Natural variability in AI responses

Solutions:
- Focus on your highest-priority instructions
- Add concrete examples to clarify intent
- Accept that some variability is normal for AI systems

**Additional resources:**
- See [example custom instructions](https://github.com/github/awesome-copilot/tree/main/instructions) in the Awesome GitHub Copilot repository for inspiration
- Read [Configure custom instructions for GitHub Copilot](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions) for technical details

#### Agent Instructions (AGENTS.md)

**When to use:** Instructions specifically for AI agents, with hierarchical directory-based precedence.

**Use AGENTS.md when:**
- You want instructions that follow your directory structure (nearest file takes precedence)
- Different subdirectories need different agent behaviors
- You're building a multi-module project where each module has unique patterns
- You want agent-specific instructions separate from general custom instructions

**How AGENTS.md works:**
- Place `AGENTS.md` files anywhere in your repository
- When Copilot is working on a file, it uses the **nearest** `AGENTS.md` in the directory tree
- Closer `AGENTS.md` files override instructions from parent directories

**Example structure:**
```
project-root/
├── AGENTS.md                    # General project-wide agent instructions
├── src/
│   ├── frontend/
│   │   ├── AGENTS.md           # Frontend-specific instructions (overrides root)
│   │   └── components/
│   └── backend/
│       ├── AGENTS.md           # Backend-specific instructions (overrides root)
│       └── api/
└── tests/
    └── AGENTS.md               # Test-specific instructions (overrides root)
```

**Use case example:**
```markdown
# Root AGENTS.md
Use TypeScript strict mode. Always write tests for new features.

# src/frontend/AGENTS.md
Frontend code must:
- Use React hooks, not class components
- Implement responsive design with Tailwind CSS
- Test with React Testing Library

# src/backend/AGENTS.md
Backend code must:
- Use Express.js patterns
- Validate all inputs with Zod
- Test with Supertest
```

When working on `src/frontend/App.tsx`, Copilot uses `src/frontend/AGENTS.md` (not root).  
When working on `src/backend/routes.ts`, Copilot uses `src/backend/AGENTS.md` (not root).

**Alternative: Agent-specific files in root**

Instead of multiple `AGENTS.md` files, you can use a single agent-specific file in your repository root:
- `CLAUDE.md` - Instructions for Claude-based agents
- `GEMINI.md` - Instructions for Gemini-based agents

These root-level files are simpler but don't support directory-based precedence.

**AGENTS.md vs copilot-instructions.md:**
- **copilot-instructions.md** - GitHub Copilot specific, always loaded for the entire repository
- **AGENTS.md** - AI agent standard, directory-based precedence, works across different AI agents

**Recommendation:** For GitHub Copilot projects, prefer `copilot-instructions.md` and path-specific instructions unless you need directory-based precedence.

#### When Instructions Take Effect

- ✅ **Immediately** - Instructions are active as soon as you save the file
- ✅ **Automatic** - No restart or reload needed
- ✅ **Always applied** - Custom instructions are added to every Copilot request in that repository

**Verify usage:** In Copilot Chat, check the "References" section of responses to see if `.github/copilot-instructions.md` is listed.

#### Priority When Multiple Instructions Exist

1. **Personal instructions** (highest priority)
2. **Repository instructions** (`.github/copilot-instructions.md`)
3. **Organization instructions** (lowest priority)
4. **Path-specific** instructions are combined with repository-wide when paths match

**Avoid conflicts:** Try not to provide contradictory guidance across different instruction types.

### Degrees of Freedom

Match specificity to task fragility:
- **High freedom (text instructions)**: Multiple valid approaches, context-dependent decisions
- **Medium freedom (pseudocode/parameterized scripts)**: Preferred pattern exists, some variation acceptable  
- **Low freedom (specific scripts)**: Fragile operations, consistency critical, specific sequence required

### Content Organization

**What NOT to include in skills:** README.md, INSTALLATION_GUIDE.md, CHANGELOG.md, etc. Only include essential information for task execution.

**Remember:** Build commands, project structure, and coding standards belong in custom instructions, not skills!

---

## Skill Creation Process

**Note:** Use this section when you've determined that creating or updating a skill is the best approach for improving agent performance.

### Step 1: Understand with Concrete Examples

Clarify concrete examples of skill usage. For a workflow debugging skill, ask:
- "What functionality should this support?"
- "Can you give examples of how this would be used?"
- "What prompts would trigger this skill?"

Avoid overwhelming with too many questions at once. Conclude when functionality is clear.

### Step 2: Create SKILL.md

Create the skill directory and SKILL.md file in `.github/skills/`:

```bash
mkdir .github/skills/<skill-name>
New-Item .github/skills/<skill-name>/SKILL.md
```

Note: Skill files must be named `SKILL.md` (case-sensitive).

### Step 3: Write the Skill

**Write YAML frontmatter:**
```yaml
---
name: skill-name
description: Clear description of what the skill does and when Copilot should use it. Be specific about the triggers and use cases.
license: Optional - if you want to specify a license
---
```

Example description: "Guide for debugging failing GitHub Actions workflows. Use this when asked to debug failing GitHub Actions workflows, analyze CI failures, or investigate workflow run errors."

**Write Markdown body:** 
- Use imperative form
- First 50 lines should contain essential content with references to detailed sections below
- Include clear step-by-step instructions
- Provide examples where helpful
- Keep concise, iterate over time

**Example structure:**
```markdown
# Skill Name

Quick overview of what this skill does.

## Quick Reference
[Essential patterns, commands, gotchas]

---

## Detailed Steps
[Step-by-step instructions]

## Examples
[Real-world usage examples]
```

### Step 4: Iterate

Use skill in real Copilot sessions → notice where Copilot struggles or gets confused → update SKILL.md → test again. Skills grow and improve through actual usage with GitHub Copilot.

**Testing tips:**
- Use clear prompts that should trigger the skill
- Observe when Copilot uses the skill (check if it follows your instructions)
- Refine the description if Copilot doesn't load the skill when expected
- Add examples or clarifications to the body based on real usage patterns