# Appendix A: Reproducible Agent Skills and Prompts

This appendix contains the agent skill prompts used in the multi-agent research pipeline, sanitized of any internal paths, environment-specific configuration, or sensitive data. These prompts can be adapted for any AI agent platform that supports sub-agent spawning.

---

## A.1 Core Multi-Agent Research Skill

**Purpose:** Orchestrate multi-round collaborative research with specialized agent personas.

**File:** `SKILL.md` (platform-agnostic skill format)

```
---
name: multi-agent-research
description: |
  Conduct deep, multi-round collaborative research using sub-agent spawning.
  Use when comprehensive research is needed on complex topics requiring
  multiple perspectives, iterative validation, and thorough verification.
---

# Multi-Agent Research Skill

Conduct multi-round collaborative research using agent sub-spawning capabilities.

## Key Principles

- **Dynamic orchestration**: Orchestrator decides rounds, topics, subagents, and prompts
- **Auto-mapped prompts**: Subagent prompts are generated per role + topic, not manually written
- **Integrated web research**: Each agent uses available web search tools
- **Round-level cross-review**: Agents review each other's work every round
- **Final independent verification**: Two dedicated reviewers verify all claims at the end
- **Full documentation**: everything is documented including research, feedback, reviews, and summaries

## Workflow

### Phase 1: Scope Clarification

1. Orchestrator asks user clarifying questions about the research goal
2. Orchestrator proposes round structure and subagent plan
3. User confirms or overrides (round count is injectable)

### Phase 2: Per-Round Execution (Loop)

For each round:

1. Orchestrator decides round topic and subagent personas
2. Orchestrator auto-maps prompts for each subagent based on role + topic
3. Subagents are spawned with their auto-mapped prompts
4. Each subagent conducts research and documents findings
5. Each subagent reviews other subagents' work and documents feedback
6. Orchestrator writes round synthesis, flagging convergences and disputes
7. Orchestrator decides: continue to next round or proceed to final review

### Phase 3: Final Review

1. Orchestrator spawns 2 independent reviewer agents
2. Each reviewer reads ALL round outputs and assesses every claim
3. Each claim is marked as: verified / false / unverified (with reasoning)
4. Each reviewer produces a structured verification document
5. Orchestrator consolidates both reviews into final output

## Orchestrator Behavior

The orchestrator:

- Never makes assumptions: asks for clarification when anything is ambiguous
- Dynamically determines round count (injectable by user, proposes if unspecified)
- Auto-maps subagent prompts: no manual prompt writing per agent
- Coordinates cross-review after each round
- Spawns final reviewers at the end
- Consolidates everything into final output

## Documentation Requirements

Every subagent spawned for any purpose (research, cross-review, synthesis) documents everything it produces:

1. **Research findings**: structured markdown with sources and data tables
2. **Peer feedback**: structured critique of other agents' work from the same round
3. **Cross-review responses**: responses to feedback received from peers
4. **Summarized versions**: condensed summaries of their contributions for easier downstream consumption

The orchestrator produces:
1. **Per-round synthesis**: cross-referenced findings with convergences and disputes
2. **Final consolidated output**: after both reviewers complete

Final reviewers document:
1. **Verification by claim**: each claim marked as verified, false, or unverified
2. **Reasoning**: why each claim was marked as such
3. **Overall summary**: assessment of research quality across all rounds

---

## A.2 Agent Prompt Templates (Platform-Agnostic)

These templates are representative of what the orchestrator auto-generates. They use `[PLACEHOLDER]` values that the orchestrator fills in dynamically.

### Research Subagent Prompt

```
You are a [ROLE] for this research round on: [TOPIC].

Your specific task:
[ORCHESTRATOR-GENERATED TASK]

Available tools for this research:
- Web search and page fetching for data collection
- File read, write, and edit for documentation

Research questions to address:
1. [QUESTION 1]
2. [QUESTION 2]
3. [QUESTION 3]

Save your findings to: research/[TOPIC]/round-[N]/[your-agent-name].md

Deliverable structure:
- Executive Summary (3-5 bullets)
- Detailed Findings with specific data and sources
- Data Tables (where applicable)
- Key Insights
- Open Questions

After completing your research, you will also review other agents' work
from this round. Be prepared to provide structured feedback.

Report back when done.
```

### Cross-Review Prompt (After Research)

```
You are reviewing the work of other subagents from Round [N] on [TOPIC].

Read the following files:
- research/[TOPIC]/round-[N]/*.md (all agent findings)

For each agent's output, provide:
1. What claims are well-supported? (with evidence)
2. What claims lack sufficient evidence?
3. What assumptions should be challenged?
4. What perspectives are missing?
5. Is there any contradictory information across agents?

Save your feedback to: research/[TOPIC]/round-[N]/[your-agent-name]-feedback.md

Report back when done.
```

### Final Reviewer Prompt

```
You are Reviewer A or B, an independent verifier for the complete research
output on [TOPIC].

Read ALL files from ALL rounds:
- research/[TOPIC]/round-*/*.md

Your task is to assess every factual claim for accuracy.

For each claim, mark it as one of:
- VERIFIED: supported by credible evidence
- FALSE: contradicted by evidence
- UNVERIFIED: insufficient evidence to confirm or refute

For each marking, document your reasoning.

Save your verification to: research/[TOPIC]/final-review/reviewer-[a/b]-verification.md

Deliverable structure:
- Executive Summary of overall accuracy
- Verified Claims (list with sources)
- False Claims (list with reasoning)
- Unverified Claims (list with what's missing)
- Overall Assessment

Your review is independent. Do not read the other reviewer output until both are complete.
```

---

## A.3 Round Configuration

Round structure is dynamically determined by the orchestrator. Typical configuration:

| Round | Focus | Subagents | Outputs |
|-------|-------|-----------|---------|
| R1 | Initial Research | 2-3 subagents | Findings + peer feedback |
| R2 | Deep Dive / Critique | 2-3 subagents | Findings + peer feedback |
| R3+ | Additional rounds as needed | 2-3 subagents | Findings + peer feedback |
| Final | Independent verification | 2 reviewers | Verification docs per claim |

Round count is injectable: pass "3 rounds" or "stop at R2" explicitly to override.

---

## A.4 Supported Platforms

The methodology has been tested on:

| Platform | Spawn Mechanism | Status |
|----------|----------------|--------|
| OpenClaw | `sessions_spawn` (sub-agent spawning) | Production-tested |
| OpenCode | Built-in `agent` tool | Production-tested |

The core logic (clarify, design, auto-prompt, spawn, review, synthesize, verify) is identical across platforms. Only tool invocation syntax differs.
