# Learn Harness Engineering Notes

## Project 01: Baseline vs Minimal Harness

### What this project is really teaching

Project 01 is not mainly about building an Electron app. It is about proving that a coding agent needs a working system, not just a goal sentence.

The weak-harness prompt is:

```md
# Task

Build an Electron app that can show documents and answer questions.
```

This prompt gives a destination, but almost no execution constraints. That is why it is unreliable.

### Why the weak prompt fails

The prompt leaves five critical things undefined:

1. **Definition of done**
   The agent does not know whether "show documents" means a real sidebar, a placeholder area, or a complete document workflow. It also does not know whether "answer questions" means a real grounded QA system or only an input box.

2. **Architecture boundaries**
   The prompt does not say how Electron layers should be separated. Without explicit rules, an agent may put filesystem or Electron logic directly into the renderer just to move faster.

3. **Verification path**
   The prompt does not require `npm run check`, `npm run build`, or `npm run dev`. That makes it easy for the agent to stop at "code written" instead of "project actually works."

4. **Scope control**
   The prompt does not state the exact feature slice for Project 01. An agent may overreach by adding extra UI, extra flows, or speculative functionality while still missing the core requirements.

5. **Persistent state**
   There is no feature tracker, no progress log, and no handoff note. Even if the result looks acceptable, there is no structured proof of what is complete.

### What the minimal harness adds

The `solution/` directory shows a small but functional harness. Each file solves a specific failure mode from the weak prompt.

#### `AGENTS.md`

`AGENTS.md` turns "start coding immediately" into an ordered startup workflow:

1. Read `AGENTS.md`
2. Read architecture and product docs
3. Run `bash init.sh`
4. Read `feature_list.json`

It also defines strict Electron boundaries:

- `src/main/` owns window lifecycle and IPC registration
- `src/preload/` is the only bridge between main and renderer
- `src/renderer/` must not import Node.js modules
- `src/services/` contains business logic for the main process

This file is the main behavior contract for the agent.

#### `feature_list.json`

This file is the scope and status control mechanism.

For Project 01, it narrows the task to exactly four features:

- `window-launch`
- `document-list`
- `question-panel`
- `data-directory`

Each feature includes:

- `status`
- `evidence`
- `testedAt`

This forces completion claims to be tied to concrete evidence instead of vague summaries.

#### `init.sh`

This file standardizes initialization and verification:

1. `npm install`
2. `npm run check`
3. `npm run build`

Its purpose is to make project recovery reproducible. A new session does not need to guess how to restore a clean working state.

#### `CLAUDE.md`

This is a short operational reference. It summarizes:

- key commands
- key files
- architecture rules
- the normal path for adding features

It is not the main rulebook. It is a quick navigation layer.

#### `claude-progress.md`

This is the first session continuity artifact. It records:

- what was completed
- which decisions were made
- whether there were issues
- what the next session should do

In Project 01 it is lightweight, but it prepares the repo for later multi-session work.

### Core lesson from Project 01

The weak prompt only describes the target.

The minimal harness adds:

- startup order
- architecture boundaries
- explicit task scope
- verification gates
- progress memory

This is the central idea of the course:

> The model decides what code to write.
> The harness decides how work is organized, checked, and resumed.

### What to remember going into later projects

- A prompt without a harness invites ambiguity.
- Ambiguity becomes scope drift, fake completion, and fragile code structure.
- `feature_list.json` is not busywork; it is a control surface.
- `init.sh` is not convenience; it is a restart contract.
- `AGENTS.md` is not documentation for humans only; it is runtime structure for the agent.
