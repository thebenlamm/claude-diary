# Claude Diary   

A long-term memory Claude Code plugin that learns from your activity and continuously improves Claude's understanding of your preferences, patterns, and workflows.

- **Diary**: Auto-generates diary entry for each Claude Code session 
- **Reflection**: Allows reflection across multiple diary entries
- **Memory**: Reflection directly updates your CLAUDE.md with actionable rules

## Quickstart 

1. **Clone this repository**:
```bash
git clone <repository-url> claude-diary
cd claude-diary
```

2. **Install commands**:
```bash
cp commands/*.md ~/.claude/commands/
```

3. **Install SessionEnd hook** (for auto-diary generation):
```bash
mkdir -p ~/.claude/hooks
cp hooks/session-end.sh ~/.claude/hooks/session-end.sh
chmod +x ~/.claude/hooks/session-end.sh
```

4. **Done!** The system is now active:
   - Diary entries will auto-generate at session end
   - Run `/reflect` whenever you want to analyze patterns
   - Your CLAUDE.md will automatically improve over time

## Overview

`CLAUDE.md` serves as [memory](https://code.claude.com/docs/en/memory) for Claude Code, but is only used manually via the `#` shortcut. This plugin automates the process of saving memories to `CLAUDE.md` from your Claude Code sessions. 

### Diary Entries

Each Claude Code session is already logged to a file in `~/.claude/projects/...` in JSONL format. This plugin includes the `diary` command to create a diary entry for the current session. You can see the rules for the `diary` command in the [diary.md](commands/diary.md) file. This is run automatically at the end of each session using [the `session-end.sh` hook](https://code.claude.com/docs/en/hooks-guide).

### Reflection

The `reflect` command is used to analyze the diary entries and create a reflection document. You can see the rules for the `reflect` command in the [reflect.md](commands/reflect.md) file.




Inspired by [an interview with Dan Shipper and Cat Wu / Boris Cherny from the Claude Code team](https://www.youtube.com/watch?v=IDSAMqip6ms&t=352s)




This plugin implements an automated memory system for Claude Code, creating a feedback loop where Claude learns from every session and automatically updates it's memory

### How It Works (Automated Flow)

```
┌─────────────────────────────────────────────────────────────┐
│  1. You work with Claude Code (normal session)              │
└─────────────────┬───────────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────────┐
│  2. Session ends → SessionEnd hook triggers                 │
│     → Diary entry auto-generated                            │
│     → Saved to ~/.claude/memory/diary/                      │
└─────────────────┬───────────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────────┐
│  3. Diary entries accumulate (5-10+ recommended)            │
└─────────────────┬───────────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────────┐
│  4. You run /reflect (ad hoc, whenever you want)            │
│     → Checks processed.log for unprocessed entries          │
│     → Analyzes patterns across diary entries                │
│     → Creates reflection document                           │
│     → Automatically updates CLAUDE.md with new rules        │
│     → Updates processed.log to prevent re-analysis          │
└─────────────────┬───────────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────────┐
│  5. Future sessions → Claude follows updated rules          │
│     → Cycle repeats, memory improves over time              │
└─────────────────────────────────────────────────────────────┘
```

### Key Features

- **Automatic + Manual Diary Creation**: PreCompact hook captures long sessions automatically, `/diary` command for manual control
- **Ad Hoc Reflection**: Run `/reflect` whenever you want to analyze accumulated entries
- **Automatic CLAUDE.md Updates**: Reflection directly updates your CLAUDE.md with actionable rules
- **Incremental Learning**: processed.log prevents duplicate analysis, enables continuous learning
- **Pattern Recognition**: Identifies patterns appearing 2+ times (strong patterns: 3+)
- **Signal vs Noise**: Distinguishes persistent preferences from one-off requests

### What Gets Captured

**Diary entries capture:**
- Design decisions and architectural choices
- PR review feedback (avoiding "AI-looking" code)
- User preferences (commit style, testing requirements, workflow)
- Challenges encountered and solutions that worked
- Code patterns and technologies used

**Reflection extracts:**
- PR review feedback patterns
- Persistent user preferences (2+ occurrences)
- Design decisions that worked
- Anti-patterns to avoid
- Efficiency lessons
- Project-specific patterns

The key insight: Synthesizing memory from multiple sessions helps identify true patterns versus one-off requests (e.g., "always use TypeScript strict mode" vs. "make this button pink").

## Installation

### Quick Start (Recommended)

1. **Clone this repository**:
```bash
git clone <repository-url> cc-memory
cd cc-memory
```

2. **Install commands**:
```bash
cp commands/*.md ~/.claude/commands/
```

3. **Install PreCompact hook** (for automatic diary generation):
```bash
mkdir -p ~/.claude/hooks
cp hooks/pre-compact.sh ~/.claude/hooks/pre-compact.sh
chmod +x ~/.claude/hooks/pre-compact.sh
```

4. **Done!** The system is now active:
   - Diary entries will auto-generate before compact operations (long sessions)
   - Run `/diary` manually anytime to capture current session
   - Run `/reflect` after accumulating 5-10+ entries to analyze patterns
   - Your CLAUDE.md will automatically improve over time

### Alternative: Install as Plugin

See [INSTALL.md](INSTALL.md) for detailed plugin installation instructions using Claude Code's local marketplace.

## Usage

### The Automated Workflow

Once installed, the system works automatically with minimal intervention:

#### Phase 1: Diary Generation (Automatic + Manual)

**Two ways to create diary entries:**

**1. Automatic (PreCompact Hook)**:
- Just work normally with Claude Code
- In long sessions (200+ messages), before Claude Code compacts the conversation, the PreCompact hook automatically:
  - Invokes `/diary` command
  - Parses the session transcript from `~/.claude/projects/...`
  - Extracts key information (decisions, challenges, preferences, patterns)
  - Creates a diary entry at `~/.claude/memory/diary/YYYY-MM-DD-session-N.md`
- Frequency: Depends on session length (short sessions may not trigger)

**2. Manual (/diary Command)**:
- Run `/diary` anytime to capture the current session
- Works for sessions of any length
- Ideal for: end of important work, after key decisions, before switching projects

**Best practice**: Use both! Automatic captures long sessions, manual captures important moments.

**What gets captured in diary entries:**
- Task summary and design decisions
- PR review feedback ("avoid AI-looking code", test coverage requirements)
- User preferences (commit style, branch naming, testing workflows)
- Challenges encountered and solutions that worked
- Code patterns and technologies used

#### Phase 2: Reflection (Ad Hoc)

**When to run**: After accumulating 5-10+ diary entries (recommended: weekly or monthly)

**How to run**: Simply execute:

```bash
/reflect
```

**What happens automatically:**
1. ✅ Checks `processed.log` → skips already-analyzed entries
2. ✅ Reads and analyzes unprocessed diary entries
3. ✅ Identifies patterns (2+ occurrences = pattern, 3+ = strong pattern)
4. ✅ Extracts actionable insights across 6 categories:
   - PR review feedback patterns
   - Persistent user preferences
   - Design decisions that worked
   - Anti-patterns to avoid
   - Efficiency lessons
   - Project-specific patterns
5. ✅ Creates reflection document at `~/.claude/memory/reflections/YYYY-MM-reflection-N.md`
6. ✅ **Automatically appends new rules to `~/.claude/CLAUDE.md`**
7. ✅ Updates `processed.log` to prevent re-analysis

**Advanced filtering options:**

```bash
# Analyze last 20 unprocessed entries
/reflect last 20 entries

# Analyze specific date range
/reflect from 2025-01-01 to 2025-01-31

# Filter by project
/reflect for project /Users/username/Code/my-app

# Filter by topic/keyword
/reflect related to testing

# Combine filters
/reflect last 15 entries for project /Users/username/Code/my-app related to React

# Re-analyze including already-processed entries
/reflect include all entries

# Re-analyze specific entry
/reflect reprocess 2025-11-07-session-1.md
```

#### Phase 3: Continuous Improvement (Automatic)

Your updated `CLAUDE.md` is read into every new session, so Claude automatically:
- Follows your commit style preferences
- Avoids anti-patterns you've identified
- Matches your code quality standards
- Uses your preferred testing approaches
- Adapts to project-specific patterns

**The cycle repeats**: More sessions → More diary entries → Periodic reflections → Smarter CLAUDE.md → Better Claude behavior

### Quick Example

Here's how the system learns in practice:

**Week 1: Sessions Accumulate**
```
Session 1: You work on React app, use conventional commits
  → Diary created: 2025-11-01-session-1.md

Session 2: PR reviewer says "add test coverage"
  → Diary created: 2025-11-02-session-1.md

Session 3: You use conventional commits again
  → Diary created: 2025-11-03-session-1.md

Session 4: PR reviewer says "code looks AI-generated, simplify"
  → Diary created: 2025-11-04-session-1.md

Session 5: You use conventional commits + run tests before commit
  → Diary created: 2025-11-05-session-1.md
```

**Week 1 End: Run Reflection**
```bash
$ /reflect

Analyzing 5 unprocessed diary entries...

Patterns identified:
✅ Conventional commit format (5/5 sessions) → Strong pattern
✅ Test coverage requested (2/5 sessions) → Pattern
✅ Avoid verbose/AI-looking code (1/5 sessions) → Noted

Reflection created: 2025-11-reflection-1.md
CLAUDE.md updated with 3 new rules:
  - git commits: use conventional format (feat:, fix:, refactor:)
  - testing: add test coverage for new features
  - code style: avoid verbose patterns, match existing code

processed.log updated: 5 entries marked as analyzed
```

**Week 2+: Claude Adapts**

In all future sessions, Claude now:
- ✅ Automatically uses conventional commit format
- ✅ Suggests adding tests for new features
- ✅ Writes simpler, less verbose code

**Month 2: More Learning**
```
10 more sessions → 10 more diary entries
Run /reflect again → Identifies new patterns
CLAUDE.md grows → Claude gets even smarter
```

**The Result**: Over time, Claude learns your exact preferences, coding style, and workflow patterns without you having to repeat yourself.

## Directory Structure

The plugin creates and uses these directories:

```
~/.claude/
├── hooks/
│   └── pre-compact.sh           # Auto-generates diary before compact (long sessions)
├── memory/
│   ├── diary/                   # Session diary entries
│   │   ├── 2025-01-15-session-1.md
│   │   ├── 2025-01-15-session-2.md
│   │   └── 2025-01-16-session-1.md
│   └── reflections/             # Synthesized insights & tracking
│       ├── 2025-01-reflection-1.md
│       ├── 2025-02-reflection-1.md
│       └── processed.log        # Tracks which entries have been analyzed
└── CLAUDE.md                    # Rules automatically updated by /reflect
```

These directories are automatically created on first use.

**processed.log format**: `[diary-entry] | [reflection-date] | [reflection-file]`

## How It Works

### Session Transcripts

Claude Code stores full conversation transcripts at:
```
~/.claude/projects/[encoded-project-path]/[session-uuid].jsonl
```

Each transcript is in JSONL format (JSON Lines) containing:
- User messages
- Assistant responses
- Tool invocations and results
- File operations
- Error messages
- Git operations

The `/diary` command parses these transcripts using `jq` (JSON processor) to create structured memory entries. See the diary command documentation for detailed jq usage examples and parsing strategies.

### Pattern Recognition

The `/reflect` command uses several principles to distinguish signal from noise:

**Signal** (becomes a rule):
- Appears 2+ times across multiple sessions (strong patterns: 3+)
- Consistent (not contradicted by other preferences)
- Generalizable (not too specific to one case)
- Actionable (Claude can follow the instruction)
- Succinct (can be expressed as one-line bullet point)

**Noise** (documented but not added to CLAUDE.md):
- One-off requests specific to a single task
- Contradictory to established patterns
- Too context-specific to be useful generally

**Processed entries tracking**: The reflect command maintains `~/.claude/memory/reflections/processed.log` to track which diary entries have been analyzed, preventing duplicate analysis and allowing ad hoc incremental reflection.

### Example Pattern Evolution

```
Session 1: User asks to use TypeScript strict mode
Session 3: User corrects code to follow strict mode
Session 5: User mentions strict mode again
Session 8: User emphasizes importance of strict types

→ Reflection identifies pattern: "Always use TypeScript strict mode"
→ Proposes CLAUDE.md update: "For TypeScript projects, always enable strict mode in tsconfig.json"
```

## Best Practices

### When to Create Diary Entries

Run `/diary` after sessions that involve:
- ✅ Solving a challenging problem
- ✅ Making architectural decisions
- ✅ Learning user preferences
- ✅ Trying multiple approaches (especially if some failed)
- ✅ Working with new technologies or patterns

Skip `/diary` for:
- ❌ Trivial one-line changes
- ❌ Simple questions with quick answers
- ❌ Sessions with no meaningful learning

### When to Reflect

Run `/reflect` when:
- ✅ You have 5-10+ diary entries accumulated
- ✅ You want to identify patterns in a specific area (e.g., testing, React)
- ✅ You're switching projects and want to capture learnings
- ✅ Monthly or weekly as a regular practice
- ✅ Your `CLAUDE.md` feels incomplete or outdated

### Reviewing Proposed Updates

When `/reflect` proposes CLAUDE.md updates:

1. **Check generalizability**: Will this apply to future sessions?
2. **Check consistency**: Does this contradict existing instructions?
3. **Check actionability**: Can Claude actually follow this instruction?
4. **Check specificity**: Is it concrete enough to be useful?
5. **Edit before applying**: Feel free to refine the proposed text

## Examples

### Example Diary Entry Structure

```markdown
# Session Diary Entry

**Date**: 2025-01-15
**Time**: 14:30:22
**Session ID**: a6c7742e-24f5-4075-b59d-b315ef66173d
**Project**: /Users/rlm/Desktop/Code/my-react-app
**Git Branch**: feature/user-auth

## Task Summary
User wanted to implement user authentication using JWT tokens with React context for state management.

## Actions Taken
- Created AuthContext.tsx with JWT token management
- Implemented login/logout methods
- Added protected route wrapper component
- Updated App.tsx to wrap routes with AuthProvider
- Created useAuth custom hook for consuming auth context

## Challenges Encountered
- Initial approach stored token in component state, which cleared on refresh
- JWT decode library had TypeScript type issues
- Protected route redirects caused navigation loop

## Solutions Applied
- Moved token storage to localStorage for persistence
- Used @types/jwt-decode package for TypeScript support
- Added redirect location tracking to prevent navigation loops

## User Preferences Observed
- Prefers functional components over class components
- Wants TypeScript strict mode enabled
- Likes custom hooks for reusable logic
- Prefers explicit error handling over silent failures

## Code Patterns and Decisions
- React Context pattern for global state
- Custom hooks for auth logic encapsulation
- Protected route HOC pattern
- JWT token storage in localStorage
- Redirect state tracking in location.state

## Context and Technologies
- React 18 with TypeScript
- React Router v6
- JWT for authentication
- Vite build tool
```

### Example Reflection Output

```markdown
# Reflection: Last 10 Entries

**Generated**: 2025-01-20 10:15:00
**Entries Analyzed**: 10
**Date Range**: 2025-01-10 to 2025-01-18
**Projects**: All projects

## Summary
Analyzed 10 diary entries spanning 8 days across 3 different projects. Strong patterns emerged around TypeScript preferences, React coding style, and testing practices. User consistently prefers functional patterns, strict typing, and comprehensive error handling.

## Patterns Identified

### Persistent Preferences (3+ occurrences)

1. **TypeScript Strict Mode** (appeared in 8/10 entries)
   - **Observation**: User consistently enables and enforces TypeScript strict mode
   - **Evidence**: Sessions 1, 2, 3, 5, 6, 7, 9, 10 all involved strict mode
   - **Confidence**: High
   - **Proposed CLAUDE.md rule**: "For all TypeScript projects, enable strict mode in tsconfig.json and avoid using 'any' types"

2. **Functional React Components** (appeared in 7/8 React sessions)
   - **Observation**: User always requests functional components, never class components
   - **Evidence**: All React sessions used functional components with hooks
   - **Confidence**: High
   - **Proposed CLAUDE.md rule**: "In React projects, always use functional components with hooks. Never suggest class components."

3. **Explicit Error Handling** (appeared in 6/10 entries)
   - **Observation**: User wants errors surfaced clearly, not silently caught
   - **Evidence**: User corrected code that swallowed errors in sessions 2, 4, 6, 7, 9, 10
   - **Confidence**: High
   - **Proposed CLAUDE.md rule**: "Always implement explicit error handling. Avoid empty catch blocks or silent failures. Log errors or surface them to the user."

## Proposed CLAUDE.md Updates

### Section: TypeScript Preferences

\`\`\`markdown
## TypeScript Configuration
- Always enable strict mode in tsconfig.json
- Avoid using 'any' types - use 'unknown' or proper types instead
- Prefer interfaces over type aliases for object shapes
- Enable all strict type checking options (strictNullChecks, strictFunctionTypes, etc.)
\`\`\`

### Section: React Coding Style

\`\`\`markdown
## React Best Practices
- Use functional components with hooks exclusively
- Never suggest class components
- Prefer custom hooks for reusable logic
- Use React Context for global state management
- Implement proper cleanup in useEffect hooks
\`\`\`
```

## Roadmap

### Phase 1: Core Memory System ✅ (Complete)
- `/diary` command for manual diary creation
- Automatic diary generation via PreCompact hook (long sessions)
- Two-approach strategy: automatic + manual diary capture
- `/reflect` command for pattern analysis
- Automatic CLAUDE.md updates with actionable rules
- processed.log tracking for incremental reflection
- Ad hoc reflection runs on accumulated entries

### Phase 2: Enhanced Analysis (Future)
- Vector similarity search over diary entries
- "Show me how I solved X before" retrieval
- Confidence scoring for patterns
- Cross-session pattern validation

### Phase 3: Integration (Future)
- Automatic context surfacing in new sessions
- Proactive pattern suggestions
- Cross-project learning
- Collaborative memory (team settings)

## Troubleshooting

### "Cannot find session transcript"

The `/diary` command looks for transcripts in `~/.claude/projects/`. If it can't find the current session:
1. Check that you're in a Claude Code session (not just a shell)
2. Verify the project path is correct
3. Try running `/diary` at the end of the session (not during)

### "No diary entries found"

Run `/diary` first to create some diary entries before running `/reflect`.

### "Cannot create memory directory"

Ensure you have write permissions to `~/.claude/memory/`:
```bash
mkdir -p ~/.claude/memory/diary
mkdir -p ~/.claude/memory/reflections
```

## Contributing

This plugin follows the [Claude Code plugin structure](https://code.claude.com/docs/en/plugins):

```
cc-memory/
├── .claude-plugin/
│   └── plugin.json
├── commands/
│   ├── diary.md
│   └── reflect.md
├── README.md
└── scope.md
```

To modify or extend:
1. Edit command files in `commands/`
2. Test locally using the local marketplace approach
3. Share your improvements

## Credits

- 
- Built for the Claude Code community
- Author: rlm

## License

MIT License - feel free to use, modify, and share.

---

**Note**: This is Phase 1 of the memory system. It focuses on manual diary entries and reflection. Future phases will add automation, retrieval, and deeper integration with Claude Code's context system.
