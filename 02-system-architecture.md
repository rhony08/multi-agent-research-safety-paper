# 2. System Architecture

## 2.1 Overview

The multi-agent research pipeline operates as a self-orchestrating system where the orchestrator agent dynamically designs and executes the entire research workflow. The orchestrator decides the number of rounds, the topic for each round, the subagents needed, and auto-maps appropriate prompts for each subagent based on the research topic. All of this happens without requiring manual configuration per round.

The prompts and methodology are platform-agnostic and have been used across multiple environments including OpenClaw (via `sessions_spawn`) and OpenCode (via its built-in `agent` tool).

The pipeline consists of three layers:

```
┌─────────────────────────────────────────────────┐
│            Orchestrator Layer                    │
│  (Dynamic planner: decides rounds, topics,       │
│   maps subagent prompts, coordinates workflow)   │
├─────────────────────────────────────────────────┤
│            Agent Layer                           │
│  (N subagents per round, auto-mapped personas,   │
│   research agents + cross-review + synthesis)    │
├─────────────────────────────────────────────────┤
│            Tool Layer                            │
│  (Web search, file operations, version control,  │
│   inter-agent file-based communication)          │
└─────────────────────────────────────────────────┘
```

## 2.2 The Orchestrator (Self-Orchestrating with Human Oversight)

The orchestrator is the central intelligence of the system. It explicitly does not make assumptions. Before spawning any sub-agents or proceeding to the next round, it asks the user for clarification on anything ambiguous, insufficiently defined, or outside its knowledge boundary. This prevents the system from proceeding on incorrect premises.

The orchestrator's workflow is fully dynamic:

1. **Clarifies scope**: asks the user questions until the research goal is unambiguous
2. **Decides round structure**: determines how many rounds are needed. If the user does not specify, the orchestrator proposes a structure and confirms it before proceeding. The round count is injectable: the user can override at any point.
3. **Defines per-round topics**: maps out what each round will cover based on the research goal
4. **Designs subagents per round**: for each round, decides which agent personas are needed (e.g., MarketValidator, SecurityReviewer, TechnicalArchitect)
5. **Auto-maps prompts**: for each subagent, automatically generates a tailored prompt based on the round's topic and the agent's role. No manual prompt writing per agent.
6. **Spawns subagents**: launches research agents, each with their auto-mapped prompt and topic
7. **Coordinates cross-review**: after all subagents in a round complete their work, instructs them to review each other's findings. Every subagent documents not just their own research but also their feedback on other agents' work.
8. **Produces per-round synthesis**: once cross-review completes, the orchestrator writes a synthesis document that cross-references findings, documents disagreements, and identifies convergences
9. **Iterates or finalizes**: decides whether to proceed to the next round or move to final review

## 2.3 Final Review Phase

After all research rounds are complete, the orchestrator spawns a dedicated **final review phase** with two reviewer agents:

```
Final Review Phase
├── Reviewer Agent A: Read ALL round outputs, assess accuracy of each info
│   ├── Mark each claim as: verified / false / unverified
│   └── Document reasoning for each marking
├── Reviewer Agent B: Same task, independent perspective
│   ├── Mark each claim as: verified / false / unverified
│   └── Document reasoning for each marking
├── Both reviewers: Produce independent summary documents
└── Orchestrator: Consolidate both reviews into final output
```

Each reviewer agent:
- Reads all output files from all rounds
- Assesses every factual claim for accuracy
- Explicitly marks claims as **verified**, **false**, or **unverified** (insufficient evidence)
- Documents reasoning for each marking
- Produces a structured summary of their assessment

The orchestrator then consolidates both reviewer outputs into the final research output, noting any disagreements between the two reviewers.

## 2.4 Agent Roles

Agent personas are not fixed. They are dynamically defined by the orchestrator per round based on what the research topic requires. Examples of roles used across past research:

| Role | Purpose | When Used |
|------|---------|-----------|
| Research Analyst | Primary data collection and pattern identification | Research rounds |
| General Researcher | Broad context and literature review | Research rounds |
| Coder / Technical Architect | Technical implementation analysis | Research rounds |
| Security Reviewer | Security and privacy implications | Critique rounds |
| Synthesizer | Cross-agent synthesis | End of each round |
| Final Reviewer | Accuracy verification of all claims | Final review phase |

## 2.5 Round Structure (Dynamic)

The orchestrator decides the round structure per research topic. A typical 3-round structure looks like:

```
Round 1 (Initial Research)
├── Subagent A: Research topic → documents findings
├── Subagent B: Research topic → documents findings
├── Subagent C: Research topic → documents findings
├── Cross-review: Agents review each other's work
└── Orchestrator: writes R1 synthesis

Round 2 (Deep Dive / Critique)
├── Subagent A: Research deeper aspects → documents findings
├── Subagent B: Challenge prior round's assumptions → documents findings
├── Subagent C: Research missing perspectives → documents findings
├── Cross-review: Agents review each other's work
└── Orchestrator: writes R2 synthesis

Round 3 (Synthesis & Final Review if enough rounds, or continue)
├── Synthesizer agent: Compiles all rounds into interim summary
├── Cross-review of synthesis
└── Orchestrator: prepares for final review phase

Final Review Phase
├── Reviewer Agent A: Verifies all claims across all rounds
├── Reviewer Agent B: Independently verifies all claims across all rounds
└── Orchestrator: Consolidates reviews into final output
```

**Round count is injectable**: the user can specify "do 5 rounds" or "stop after round 2" at any point. If unspecified, the orchestrator evaluates completeness and proposes a structure.

## 2.6 Comprehensive Documentation

Every subagent documents everything it does:

- **Research output**: findings with sources and data tables
- **Cross-review feedback**: notes on other agents' work, what was accepted, what was challenged
- **Revisions**: updated findings based on feedback from peers
- **Summaries**: condensed versions of their contributions

This creates a complete audit trail. For a typical research run, the output includes:

```
research/[topic-name]/
├── round-1/
│   ├── subagent-a.md           # Research findings
│   ├── subagent-a-feedback.md  # Feedback on other agents
│   ├── subagent-b.md           # Research findings
│   ├── subagent-b-feedback.md  # Feedback on other agents
│   ├── subagent-c.md           # Research findings
│   ├── subagent-c-feedback.md  # Feedback on other agents
│   └── synthesis.md            # Orchestrator's R1 synthesis
├── round-2/
│   ├── [same structure]
├── final-review/
│   ├── reviewer-a-verification.md  # Claim-by-claim verification
│   ├── reviewer-b-verification.md  # Independent verification
│   └── final-output.md             # Orchestrator's consolidated output
└── 00-FINAL.md                  # Abridged final output
```

## 2.7 Tool Integration

Each subagent has access to:

- **Web research**: `web_fetch`, `web_search` (or `fetch` in OpenCode) for data collection
- **File operations**: `read`, `write`, `edit` for creating structured markdown documents
- **Inter-agent communication**: Agents read each other's output files to provide review and critique
- **Memory**: Session context for continuity within a round

Agents **cannot** write to external systems (email, social media, etc.) without explicit orchestrator approval. This constraint is a design choice to limit the blast radius of any agent malfunction.

## 2.8 Cross-Platform Portability

The system has been deployed on:

- **OpenClaw**: uses `sessions_spawn` with JSON-based sub-agent configuration
- **OpenCode**: uses the built-in `agent` tool with prompt-based sub-agent spawning

The core workflow (clarify scope, design rounds, auto-map prompts, spawn agents, cross-review, synthesize, final review) is identical across platforms. Only the tool invocation syntax differs.
