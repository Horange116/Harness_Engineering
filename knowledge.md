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

## Project 02: Agent-Readable Workspace

### What this project is really teaching

Project 02 shifts the focus from single-session reliability to multi-session continuity.

Project 01 proved that explicit rules improve execution quality. Project 02 asks a different question:

> When a new agent session starts tomorrow, can it understand the repo quickly enough to continue real work instead of rediscovering everything?

This project teaches that continuity must live in the repository, not only in chat history.

### What problem Project 02 solves

In real development, work is often interrupted:

- a session ends before the feature is done
- a different agent instance resumes the task
- the previous conversation context is no longer available

Without repository-level continuity artifacts, the new session must re-derive:

- what the project is
- how the architecture works
- what features are in scope
- what was already completed
- what the next step should be

That re-discovery cost is the real target of Project 02.

### The new idea: repository as the system of record

Project 02 introduces a stronger form of harness design:

- important execution context should be written into the repo
- the repo should explain itself to a fresh agent session
- progress should not depend on remembering a past conversation

This is why the project emphasizes agent-readable workspace artifacts rather than only better instructions.

### Why information layering matters

Project 02 does not simply add more files. Its key change is that information is split by purpose instead of being dumped into one giant explanation file.

If everything lives in one large document, two things happen:

1. The agent must scan the whole document on every startup, which is expensive and easy to miss.
2. Stable information and rapidly changing information get mixed together, which makes updates noisy and confusing.

Project 02 solves that by separating concerns:

- `AGENTS.md` for startup rules and working constraints
- `docs/ARCHITECTURE.md` for long-lived structure and data flow
- `docs/PRODUCT.md` for feature scope and intended user behavior
- `feature_list.json` for current feature state
- `session-handoff.md` for immediate session-to-session continuity

Clear naming helps the agent find information. Layered content helps the agent actually use it.

### What Project 02 adds to the product

The product itself gains three new features:

- document import
- document detail view
- basic persistence across restarts

In `feature_list.json`, these appear as:

- `document-import`
- `document-detail`
- `basic-persistence`

This means Project 02 is not just a documentation exercise. The repo structure and the product both evolve together.

### What each new artifact is for

#### `docs/ARCHITECTURE.md`

This file explains the system in agent-readable form:

- Electron layer diagram
- import flow
- content retrieval flow
- storage layout

Its purpose is to prevent every new session from reverse-engineering the same architecture from source files.

#### `docs/PRODUCT.md`

This file explains user-facing behavior and scope:

- which file types can be imported
- what document detail should show
- what persistence means in this stage
- what constraints still exist

Its purpose is to separate product intent from implementation detail.

#### `session-handoff.md`

This is the most important new file in Project 02.

It tells the next session:

- what was accomplished
- what remains
- what decisions were made
- which files changed
- whether any blockers exist
- what should happen next

Its purpose is not historical completeness. Its purpose is fast, accurate task resumption.

### What a good session handoff looks like

A good handoff is:

- specific
- current
- action-oriented
- tied to files and decisions
- short enough to scan quickly

It should answer:

- What is done?
- What is not done?
- What should the next session do first?
- What constraints or decisions must not be rediscovered?

Example of a useful handoff:

- "Implemented document import and detail view."
- "Persistence across restart still needs verification."
- "Added `GET_DOCUMENT_CONTENT` channel in `src/shared/types.ts` and wired it through preload and IPC handlers."
- "Next step: verify startup reload path and update feature status if document list restores correctly."

### What an ineffective handoff looks like

A bad handoff is vague, bloated, or disconnected from action.

Examples of weak handoff statements:

- "Worked on the app today."
- "Made lots of progress."
- "Some files were changed."
- "Need to continue later."

These statements do not help the next session decide what to read, what to trust, or what to do first.

Another failure mode is mixing long-term documentation with session notes. For example, if an architecture document also contains yesterday's unfinished TODOs, the signal gets diluted.

### Core lesson from Project 02

Project 02 teaches that reliable agent work requires more than rules. It requires repository-native continuity.

The main shift is:

- Project 01: "How do we stop the agent from acting blindly?"
- Project 02: "How do we stop the next session from starting blind?"

### What to remember going into later projects

- Chat history is not a durable project memory system.
- Good repository documents reduce re-discovery cost.
- Clear naming is useful, but file purpose and content separation matter more.
- `session-handoff.md` is a restart tool, not a diary.
- A repo becomes agent-readable when structure, scope, and recent state are all explicit.

## Project 03: Scope Control and Grounded Verification

### What this project is really teaching

Project 03 shifts the focus from continuity to delivery discipline.

Project 02 solved the problem of resuming work in a fresh session. Project 03 asks a new question:

> After the agent successfully resumes work, can it stay within scope and prove that each feature is actually complete?

This project teaches that continuity alone is not enough. An agent can resume correctly and still overreach, under-finish, or declare victory too early.

### How Project 03 differs from Project 02

The simplest distinction is:

- Project 02: help the next session find the road
- Project 03: stop the next session from driving off the road

Project 02 is mainly about:

- repository readability
- information layering
- session continuity
- faster context recovery

Project 03 is mainly about:

- one-feature-at-a-time execution
- explicit feature dependencies
- fail-to-pass state transitions
- verification evidence before completion claims

Project 02 answers "Where do I continue?"
Project 03 answers "What exactly am I allowed to do next, and how do I prove it worked?"

### What problem Project 03 solves

Once features become more complex, the most common failure modes change:

- the agent implements multiple features in one pass
- the agent edits unrelated areas while chasing a task
- the agent claims a feature is done because code exists
- the feature appears present but is not truly verified end-to-end

Project 03 introduces execution discipline to reduce those failures.

### What Project 03 adds to the product

The product gains four new capabilities:

- metadata extraction
- document chunking
- indexing status UI
- grounded Q&A with citations

In `feature_list.json`, these appear as:

- `metadata-extraction`
- `document-chunking`
- `indexing-status-ui`
- `grounded-qa`

These features are more interdependent than the earlier projects, which is why explicit dependency management becomes important.

### The one-feature-at-a-time rule

The core rule in Project 03 is simple:

1. Pick one feature from `feature_list.json`
2. Implement only that feature
3. Verify it
4. Update the feature state with evidence
5. Only then move to the next feature

This rule is not about moving slowly. It is about reducing regression risk and fake completion.

Without this rule, complex tasks often turn into broad, partially finished changes that are hard to verify and hard to resume.

### Why `feature_list.json` becomes more important here

In Project 01, `feature_list.json` mainly narrowed scope.
In Project 02, it helped preserve current feature state across sessions.
In Project 03, it becomes a true control surface.

Its role is now to define:

- what can be worked on next
- which features depend on others
- whether a feature is still effectively failing
- what evidence is required before it can be treated as passing

The key idea is:

> A feature is not done because the agent says it is done.
> A feature is done when it moves from failing to passing with evidence.

### Why feature dependencies matter

Project 03 explicitly documents dependencies such as:

- `metadata-extraction -> document-chunking`
- `document-chunking -> indexing-status-ui`
- `document-chunking -> grounded-qa`

This matters because the agent should not choose implementation order based only on what sounds easy or visible in the UI.

A feature list is stronger when it encodes not just tasks, but the correct order of execution.

### What counts as real verification

Project 03 raises the verification bar.

It is no longer enough to check that:

- code compiles
- the app launches
- some output appears

Verification must be tied to feature semantics.

For example, grounded Q&A is not verified just because an answer string is returned. It must also be checked for:

- the presence of citations
- whether citations point to real document chunks
- whether those citations are relevant to the answer

This is an important transition in the course: moving from build verification to behavior verification.

### What `clean-state-checklist.md` adds

Project 03 introduces a final structured verification layer through `clean-state-checklist.md`.

This checklist does not only ask whether features exist. It asks whether the repository is in a clean, restartable, reviewable state across:

- build verification
- feature verification
- scope-control verification
- code quality
- documentation completeness

This prevents a common failure mode where each individual feature appears done, but the overall repository is still messy or unsafe to hand off.

### What `claude-progress.md` becomes here

In Project 03, progress tracking becomes more feature-oriented.

Instead of simply logging that "work happened today," the progress file records:

- which feature was worked on
- what was changed
- how it was verified
- when the feature was promoted to `pass`

This makes the progress log useful as evidence of how a feature moved through the harness, not just as a diary.

### Core lesson from Project 03

Project 03 teaches that a reliable harness must control not only context recovery, but also execution boundaries and completion criteria.

The main shift is:

- Project 02: "Can the new session resume work efficiently?"
- Project 03: "Can the resumed session avoid scope drift and prove completion?"

### What to remember going into later projects

- Session continuity does not guarantee delivery accuracy.
- A feature list is more powerful when it controls order, state, and evidence.
- One-feature-at-a-time is a safety mechanism, not just a project management preference.
- Verification should test the real behavior of the feature, not only whether code exists.
- Clean handoff requires both recent context and a genuinely clean repository state.
