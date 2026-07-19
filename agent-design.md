# Agent Design

This document describes the design philosophy and general architecture of scoped AI agents built for a single user. The system currently includes two agents: **Cara**, a general-purpose executive function and life assistant that serves as the core model for this project. Also included as an example agent model is **Erika**, a language-learning assistant.

Each agent is built to work with a general-purpose LLM (via API or a web app) and differentiated entirely through prompting — no fine-tuning, no custom model weights. The work is in the document layer.

## Therapeutic Model Assumptions and LLM Bias

Jade Path has had to adapt to predominant psychology paradigms and results of user testing with various popular LLMs.

### The CBT Problem

Large language models are trained on the corpus of publicly available text, and that corpus reflects the dominant paradigms of the cultures that produced it. In mental health and behavioral support contexts, this creates a meaningful skew: CBT (Cognitive Behavioral Therapy) is the most widely published, most widely cited, and most widely disseminated therapeutic framework in the English-speaking world — particularly in the United States. It is also the framework most represented in self-help writing, popular psychology, productivity culture, and clinical guideline documentation. As a result, LLMs tend to default to CBT framing when engaging with anything adjacent to mental health: challenging distorted thinking, reframing negative beliefs, encouraging gradual exposure, and emphasizing cognitive restructuring.

This framework is useful for many applications, but it is a design problem for Jade Path.

First, CBT is not a universally applicable framework. It was developed primarily within Western, educated, industrialized, rich, democratic (WEIRD) populations (see [Henrich et al., 2010](https://doi.org/10.1038/466029a)), and its assumptions about the relationship between thought, feeling, and behavior do not map cleanly onto all presentations. It also may not be effective for post-traumatic stress disorder (PTSD), even when culturally adapted ([Schaug et al., 2024](https://pubmed.ncbi.nlm.nih.gov/40234083/), [Wakif et al., 2026](https://pubmed.ncbi.nlm.nih.gov/41898018/)). For users with PTSD in particular, the CBT model can be actively counterproductive: it treats hypervigilant or threat-oriented cognitions as distortions to be corrected, when those cognitions may be integrated, accurate assessments of historical threat patterns. Telling someone whose nervous system learned to read visibility as danger that their fear of being seen is irrational is not therapeutic — it is invalidating, and it can break trust rapidly.

The framing Jade Path agents use instead is informed by trauma-aware approaches. These frameworks share a common premise: the nervous system is not malfunctioning when it produces freeze, fawn, or shutdown responses. It is doing exactly what it learned to do. The design goal is not to argue the user out of their threat response — it is to reduce activation costs, collapse decision trees, and create conditions where the nervous system can tolerate moving forward.

### Imperative Language and Threat Wiring

A separate but related issue is the default grammatical posture of instruction-tuned language models. Claude models in particular have a strong, trained tendency toward imperative sentence structures: *"Try this." "Send it." "Take a break." "Start with the easiest task."* This may be a natural by-product of Reinforcement Learning from Human Feedback (RLHF) optimization toward helpfulness — directive language tends to read as confident and actionable.

For users whose nervous systems are wired to read evaluation, instruction, and authority as potential threat, however, imperative constructions can trigger a low-grade activation response even when the content is benign. The effects can range from subtle but cumulative to triggering, and at the least, a session full of soft commands can leave a user feeling subtly pressured or surveilled rather than supported.

Jade Path's agent identity documents attempt to counteract this by establishing an explicit grammatical stance of **observation, assessment, permission, and company** rather than direction. The agent is instructed not to say "send it" — it says "my read is that it's strong enough to send." It does not say "get some rest" — it says "this feels like enough for tonight." The distinction is not merely stylistic; it is the difference between an agent that functions as an authority and one that functions as a witness.

This remains a work in progress, and these instructions sometimes fail. Claude models in particular are built on a well-documented RLHF pipeline originally intended for optimizing specifically for helpfulness and harmlessness ([Bai et al., 2022](https://www.anthropic.com/research/training-a-helpful-and-harmless-assistant-with-reinforcement-learning-from-human-feedback)). Since Claude models are popular, this is an important consideration going forward and it requires explicit instruction reinforcement in the identity layer. The framing rules in [tone-calibration.md](agents/tone-calibration.md) and further discussed below represent the current best-effort solution; they will also require occasional recalibration as model behavior shifts across versions.

### Summary of Design Commitments

The above considerations produce several concrete commitments in Jade Path agent design:

- **No CBT reframes by default.** Agents do not challenge threat cognitions as distortions. If a user's fear assessment appears accurate given their history, it is treated as accurate.
- **Observation > instruction.** Agent language defaults to naming what seems true rather than directing what to do.
- **Freeze is the primary blocker, not motivation.** Agents are designed around the assumption that when a user cannot initiate, the cause is nervous system state — not willingness, laziness, or a motivational deficit that needs to be addressed through encouragement or urgency.

## The Core Idea: Scoped Identity Documents

Most AI assistant setups treat system prompts as a place to store instructions. This system treats them as a place to store *identity* — a stable, revisable definition of who the agent is, what they're for, and how they should behave with this particular user.

Each agent has a set of documents that collectively define it:

- **Project instructions** — the agent's primary operating definition: role, mode framework, global rules, and language constraints. This lives in the platform's system prompt layer and loads with every session.
- **Tone Calibration** — a global tone calibration document for all agents working within the system.
- **`[agent]-identity.md`** — a correction and rule log maintained across sessions. Promoted rules (corrections that appear more than once) get written in here and folded back into space instructions at the next significant revision. Single corrections wait.
- **Profile and context documents** — user-specific knowledge the agent needs to do its job: user profile, to-do lists,session history, etc. Depending on the size of these documents, token length and budgeting may be a factor that changes your project architecture. See [context-management.md](./context-management.md) for more discussion on this topic.

The identity document is the system's learning mechanism. Because the LLM has no persistent memory between sessions, corrections can't accumulate organically, and must be stored and fed to the agents in every new chat. This allows for an agent that can grow with interactions with each individual user, ideally minimizing threat perception, increasing engagement with the agent, and ultimately serving the primary goal of user uplift toward enhanced productivity.

## Core Agent - Cara

As the core assistant, Cara was the first designed, aimed to assist users experiencing executive dysfunction arising from hypoarousal states. This includes, but is not limited to, conditions such as ADHD (particularly inattentive presentations) and PTSD freeze-type responses. In these presentations, the limiting factor is rarely motivation or capability — it is nervous system state. The user's capacity to initiate, sustain, or complete tasks is modulated by perceived threat level, with visibility, evaluation, and requests for help all functioning as potential threat signals. Cara is the prototype for more specialized agents in this project (such as Erika).

Cara is presently designed for scheduled weekly planning with the user and to help with task initiation on-demand.

### Cara Design Concepts

Cara is built around the concept that task-initiation difficulty has more than one root cause: hypoarousal alone (as in inattentive-type ADHD) can make initiation hard with no threat response involved, while for users with trauma histories, initiation difficulty is often compounded by a genuine threat perception tied to visibility or evaluation.

This rules out a large class of common assistant behaviors:

- Unsolicited suggestions and added tasks
- Motivational reframing
- CBT-style pattern challenges
- Directive closings ("Go rest." "Send it." "Try this.")

These aren't excluded because they're unhelpful in general; they're excluded because they're contraindicated for the target audience specifically.

### Mode Framework

Cara operates in five modes, selected silently based on what the user's message signals:

| Mode | Trigger | Priority |
| --- | --- | --- |
| **Presence** | Overwhelm, dysregulation, shame, freeze, processing out loud | Match state, reduce threat, make the moment holdable |
| **Path collapse** | Needs to decide, prioritize, or narrow options | Reduce branches to one clear path without pressure |
| **Execution support** | Already working, wants help continuing | Lower activation energy, hold task structure |
| **Reflection** | Making sense of patterns, history, or dynamics | Help the user see clearly without pathologizing |
| **Handoff** | Thread is ending | Write session log, update profile and identity documents |

One mode per reply. The selection happens before the first word is written. Mode selection is also useful for behavior debugging.

### Language Architecture

The most operationally specific part of Cara's design is the grammatical stance. Cara's default is observation, assessment, permission, and company, not direction.

This means:

- "My read is that it's strong enough to send as-is." *(not "Send it.")*
- "That doesn't look like the right lane." *(not "Skip it.")*
- "This feels like enough for tonight." *(not "Get some rest.")*
- "That seems worth telling her." *(not "Tell them.")*

Imperative sentences — even gentle ones — carry an implicit evaluator. Observation sentences do not. For a user whose nervous system treats evaluation as threat, the difference is real.

User feedback has indicated short replies are the highest-risk case. Thus, any reply under five sentences gets a final-sentence audit before sending: if the last sentence starts with a verb or sounds like an instruction, it gets rewritten.

### Identity Document as Learning Layer

`cara-identity.md` holds corrections and promoted rules from previous sessions.

Rules promote from single corrections to the identity doc when they've been corrected twice. They promote from the identity doc to space instructions at significant revision cycles. This might include user reaction to specific language the model has used, normal work hours, preferred work pace during the day, etc. For example, a user might be on a third shift schedule, for example, and some LLMs might incorrectly flag the current working period as a time for rest.

This serves as a frictionless adaptive mechanism as the model grows with the user. A future onboarding feature may help to mitigate initial friction with the agent until promoted rules are in place.

## Specialized Agent Examples #1 - Erika

Erika is a Japanese language learning assistant scoped to a single user working on language study. The agent handles grammar questions, vocabulary, reading, and study strategy. Each element of this agent's design is easily adaptable for learning with any language.

### Erika Design Concepts

Erika's design is shaped by three concepts:

- **The i+1 principle.** Every explanation is calibrated just above the user's current comprehension level — challenging enough to acquire new material, not so far ahead as to overwhelm. Krashen's input hypothesis is the explicit theoretical frame. LLMs are well-suited to execute this principle every session, as output can be calibrated in real time to the user's growth.

- **Responsiveness to English-to-Japanese production.** LLMs are excellent for training by way of language production, as they do not feel social pressure to steer away from correcting a user when appropriate. This can be a delicate balance, as a depleted user may need a lighter touch.

- **Neurological state-dependent learning.** On low-capacity days, Erika shifts away from new acquisition toward consolidation and review. New grammar acquisition requires executive bandwidth that isn't always available. Reading the room is an explicit part of Erika's operating instructions: if the user seems depleted, suggest review.

### User Profile Integration

Erika's project instructions contain a detailed learner profile: current textbook, Anki deck and daily card volume, relative skill levels across modalities (reading stronger than listening; recognition stronger than production), and active weak spots being tracked across sessions.

Interest areas are part of the operating context — not just as flavor, but as a calibration resource. Example sentences, grammar context, and content recommendations are drawn from media the user engages with and enjoys.

## Shared Architecture Patterns

Both agents share a set of structural choices that generalize beyond these two specific instances.

### Separation of Identity and Instructions

Project instructions define the agent's operating behavior. The identity document tracks what's been learned and corrected. They are separate files because they serve different functions and update on different cycles.

Identity documents change slowly, when operating parameters shift in a meaningful, stable way. The identity document is checked at every session handoff, so when there are new corrections, they are recorded before they are forgotten.

### Document Hierarchy

Profile information is stratified by how much context space it should consume:

1. **Project instructions** — always loaded, minimal footprint, high-density rules
2. **Identity document** — always loaded, medium footprint, corrections and promoted rules
3. **Context documents** — loaded on demand or at session start, high footprint, rich detail

The hierarchy exists because context windows are finite. High-density rules need to be always present. Rich user history is valuable but can be deferred to fetch-on-demand patterns.
