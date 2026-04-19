# Skill: Learning System

Tracks patterns, decisions, and corrections made during a session to improve
future code generation accuracy.

## When to Use

- When the user corrects generated code
- When a pattern is confirmed as preferred
- When a new convention is established
- At the end of a significant coding session

## What to Track

### User Corrections

```
When the user changes generated code, note:
- What was generated vs. what they changed it to
- The pattern rule this implies
- Whether it contradicts an existing skill

Example:
  Generated: <v-btn color="primary">Save</v-btn>
  Corrected: <Button type="primary" :title="$t('actions.save')" />
  Rule: Always use project Button component, never v-btn directly
```

### Project-Specific Patterns

```
Patterns discovered during the session:
- Component naming conventions specific to this project
- API endpoint naming specific to this project
- Custom composables and their usage
- Design token overrides or additions
```

### Decisions Made

```
Architectural or design decisions:
- "We decided to use X approach instead of Y"
- "This module handles Z differently because..."
- "The client prefers A over B for this project"
```

## Storage

Store learnings in repository memory (`/memories/repo/`) so they persist
for the specific project:

```
/memories/repo/patterns.md     — Discovered code patterns
/memories/repo/conventions.md  — Project-specific conventions
/memories/repo/decisions.md    — Architectural decisions
```

## Format

```markdown
## [Date or Context]

### Pattern: [Name]

- **Context**: What triggered this learning
- **Rule**: The specific rule to follow
- **Example**: Before → After
```

## Rules

```
□ Only record confirmed patterns — not one-time exceptions
□ Update existing entries rather than creating duplicates
□ Remove entries that are proven wrong
□ Keep entries concise — one pattern per entry
□ Reference the relevant skill file if the learning should update it
□ Ask before modifying skill files based on learnings
```
