# 7. Open Problems

My research identified several problems that remain unsolved and require further investigation by the AI safety community.

## 7.1 The Vector Embedding Deletion Problem

As described in Section 4.1, PII embedded in vector representations cannot be truly deleted. For multi-agent systems, this means:

- **Shared memory stores accumulate latent PII** that cannot be reliably purged
- **Agent A can infer information about Agent B's inputs** through embedding similarity, even without direct access
- **Regulatory compliance (GDPR "right to be forgotten")** is functionally impossible in shared embedding spaces

**Research question:** Can we design embedding architectures with formal deletion guarantees, for example embeddings that are linear combinations of deletable components?

## 7.2 Hallucination Propagation Measurement

I observed that hallucinations can propagate across agents, but I lack metrics to measure this phenomenon:

- How far does a hallucination propagate before detection?
- What factors (agent persona, topic domain, confidence calibration) affect propagation distance?
- Can we design multi-agent topologies that bound hallucination propagation (e.g., lattice structures where each claim is verified by N independent agents before reuse)?

## 7.3 Agent Persona Diversity as Safety

My methodology uses diverse agent personas (Research Analyst, Coder, General Researcher) to ensure multiple perspectives. But:

- What is the optimal persona diversity for safety vs. coherence?
- Can personas themselves introduce systematic blind spots?
- Should agent personas be randomized per round to prevent persona-specific biases from compounding?

## 7.4 Trustworthy Inter-Agent Communication

When Agent B reviews Agent A's output, how does Agent B know it's actually reviewing Agent A's work and not a tampered version?

- Current approach: file-based communication in a trusted execution environment
- But trust assumptions propagate: if the environment is compromised, all agent communication is compromised
- Cryptographic verification of agent outputs would help but adds operational complexity

## 7.5 Multiple Agent Instance Identity

When multiple instances of the same research pipeline run simultaneously (e.g., for different topics), can agents from one research instance influence agents from another?

- Shared tool environments (file systems, APIs, web search) create implicit coupling
- Parallel research runs may accidentally contaminate each other's context
- Isolation guarantees for parallel multi-agent execution are poorly understood

## 7.6 Standardized Safety Benchmarks for Multi-Agent Systems

The AI safety community has developed benchmarks for single-model safety (MMLU, HELM, TruthfulQA), but no equivalent exists for multi-agent systems:

- What constitutes a "safe" multi-agent interaction?
- How do you measure the safety of a multi-layer validation methodology?
- Can we create adversarial multi-agent test suites (e.g., "agent red-teaming")?

## 7.7 Context Window and Cost Efficiency in Multi-Agent Research

Every round generates comprehensive documentation: research findings, peer feedback, synthesis documents, and final verification reports. While this thoroughness ensures traceability, it creates a practical problem. Subsequent reviewing agents must process large volumes of text, and the cost scales with token count. Without careful management, the paper trail can become too expensive to process fully.

- How do we balance documentation thoroughness against token cost?
- Can reviewers work effectively with summarized or sampled inputs instead of full transcripts?
- What is the minimum documentation required for reliable verification without inflating costs?
- Can we design tiered documentation where detailed records exist but reviewers only access condensed versions by default?

This is both a cost optimization problem and a safety problem. If reviewers skip context because of length, errors may go undetected. If full context is always required, the system becomes too expensive to run at scale.

## 7.8 Informed Consent for Agent Participation

In my system, agents do not consent to participate in research or to have their outputs reviewed by other agents. For increasingly autonomous agents:

- Should agents have "opt-out" rights for cross-agent review?
- Can agents express preferences about what context they share?
- This is a speculative concern for current systems but becomes relevant with increased agent autonomy.
