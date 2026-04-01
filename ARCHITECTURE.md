# Claude Code Architecture — Deep Analysis

> Consolidated from the March 31, 2026 source map leak, official Anthropic documentation,
> community analyses, and GitHub rewrite projects.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Runtime | Bun (acquired by Anthropic late 2025) |
| Language | TypeScript (~512K lines, strict mode) |
| UI | React 18 + custom terminal reconciler (Ink) |
| Layout | Yoga (Facebook flexbox) |
| Validation | Zod v4 |
| CLI | Commander.js |
| API Client | @anthropic-ai/sdk |
| Extensibility | Model Context Protocol (MCP) SDK |

## Core Subsystems

### 1. Prompt Engine (110+ segments)
- **Modular composition**: Runtime assembly from 110+ individually maintained segments
- **Cache boundaries**: `SYSTEM_PROMPT_DYNAMIC_BOUNDARY` separates static (cacheable) from dynamic content
- **14 cache-break vectors**: Tracked by `promptCacheBreakDetection.ts`
- **`DANGEROUS_uncachedSystemPromptSection()`**: Warning annotation for volatile content
- **Deferred tools**: 18 tools hidden until model searches via ToolSearchTool
- **Alphabetical tool sorting**: Keeps tool list deterministic for cache stability

### 2. Tool System (45+ tools)
- **Unified interface**: Generic `Tool<Input, Output, Progress>` with Zod schemas
- **Four-tier rendering**: Progress, result, compact, error views
- **ACI design**: Absolute paths, no formatting overhead, deep module principle
- **Concurrent execution**: Read-only tools parallel (max 10), writes serial
- **Streaming dispatch**: Tools start executing before model finishes responding
- **Feature-gated**: Compile-time flags, user type, permission modes, risk classification

### 3. Query Engine (while-true loop)
- **State machine**: Messages, tool context, recovery counts, compaction state, transitions
- **Three recovery paths**: Context compaction → Collapse drain → Token escalation
- **Token budget continuation**: "Resume directly — no apology, no recap" (max 3 attempts)
- **Write-ahead logging**: Transcript saved before API call for crash recovery
- **Streaming tool executor**: Overlaps tool execution with model generation

### 4. Multi-Agent Orchestration
- **Coordinator mode** (`coordinatorMode.ts`): Research → Synthesis → Implementation → Verification
- **Worker communication**: `<task-notification>` XML messages + shared scratchpad
- **Prompts as architecture**: "Do not rubber-stamp weak work"
- **Isolation**: AsyncLocalStorage (in-process), tmux (cross-process), git worktrees
- **ULTRAPLAN**: Remote Cloud Container Runtime, 30 min Opus thinking, teleport back
- **PROACTIVE mode**: `<tick>` prompts, autonomous work identification

### 5. Memory System (three layers + AutoDream)
- **Layer 1**: MEMORY.md index (200 lines, always loaded)
- **Layer 2**: Individual files (user, feedback, project, reference types)
- **Layer 3**: Selective attachment (Sonnet relevance selector, up to 5 files)
- **AutoDream**: Three-gate trigger (24h, 5 sessions, lock) → Four phases (Orient, Gather, Consolidate, Prune)
- **Design principle**: Memory as hints, not facts — verify before acting
- **Context compression**: MicroCompact → AutoCompact → Full Compact

### 6. Terminal Renderer (game-engine techniques)
- **Pipeline**: React → Reconciler → Virtual DOM → Yoga → Output Builder → Screen Buffer → Diff → ANSI
- **Int32Array** character pools with bitmask-encoded styles
- **Three interning pools**: chars, styles, hyperlinks (O(1) comparisons)
- **Patch optimizer**: Merges cursor movements, cancels hide/show pairs
- **50x reduction** in stringWidth calls via self-evicting cache
- **Double-buffering** with frame pointer swapping

### 7. Security (7 layers, 23 bash checks)
1. Security monitor agent (block/allow rules)
2. Sandbox enforcement (bubblewrap/seatbelt + network proxy)
3. Permission model (5 public + 2 internal modes)
4. Undercover mode (internal employees on external repos)
5. Anti-distillation (fake tools + connector-text summarization)
6. Native attestation (`cch` hash in Zig below JS runtime)
7. Protected files (.gitconfig, .bashrc, .mcp.json, etc.)

## Unreleased Features (44 feature flags)

| Feature | Description | Status |
|---------|-------------|--------|
| **KAIROS** | Always-on daemon, <tick> prompts, 15s blocking budget, push notifications, PR subscriptions | Apr-May 2026 |
| **BUDDY** | Tamagotchi pet, 18 species, rarity tiers, Mulberry32 PRNG | April Fools 2026 |
| **ULTRAPLAN** | Remote planning, 30 min Opus, browser approval, teleport | Feature-gated |
| **VOICE_MODE** | Push-to-talk interface | Early references |
| **/dream** | Manual AutoDream trigger | Feature-gated |

## Design Principles

1. **Composability over monoliths** — modular prompts enable surgical updates
2. **Explicit over implicit** — permission gates prevent surprises
3. **Specialized over generic** — purpose-built tools replace shell commands
4. **Bounded autonomy** — agent spawning uses explicit rules
5. **Observable execution** — system reminders provide continuous feedback
6. **Recoverable state** — worktrees, memory, compaction enable resumption
7. **Prompt as architecture** — orchestration expressed in natural language, not code
8. **Context as finite resource** — every token decision is an economic decision

## Internal Codenames

- **Tengu**: Claude Code project codename (hundreds of feature flags prefixed `tengu_`)
- **Capybara**: Claude 4.6 variant (v8 has 29-30% false claims, regression from v4's 16.7%)
- **Fennec**: Opus 4.6 variant
- **Numbat**: Unreleased model in testing
- **Penguin Mode**: Fast mode endpoint (`/api/claude_code_penguin_mode`)

## Key Metrics from the Leak

- 512,000 lines of TypeScript across ~1,900 files
- 45+ tool implementations, 100+ slash commands, 146 React components
- 90 files in the custom terminal renderer
- 330+ utility files
- 187 spinner verbs
- 360MB idle / 746MB peak memory usage
- 250,000 wasted API calls/day from autocompact bug (fixed with 3 lines)
- 93% of permission prompts approved by users (drove auto mode development)
- Sandbox reduces permission prompts by 84%
