# Jade Path

Jade Path is a proof-of-concept system for neurodiversity-aware AI assistants, built around persistent memory, low-pressure task support, and gentle re-entry after periods of low activity. It grew out of practical lessons in productivity support, context management, and everyday use.

The core of the system is a primary agent (Cara), shared tone and profile conventions, and supporting documentation covering the design decisions behind them. A Japanese language study agent (Erika) is also provided, which serves as an example of modular agent design that can work with the system.

## Design Philosophy

This project is built around users who benefit from externalized structure, persistent context, and low-pressure task support rather than motivation- or pressure-based productivity systems. The agents are designed to reduce cognitive load by narrowing options rather than expanding them, and to support task initiation without escalation or evaluation pressure.

This orientation comes from direct, ongoing use, including regular testing of agent behavior across different underlying models. This repo doubles as a working environment for ongoing testing.

## Quick Start (LibreChat)

Jade Path is currently optimized for use with LibreChat, though its components can be adapted for use with other LLM front ends. This is a manual setup process at this project's early stage. A formal user onboarding feature is planned for future development.

### Initial Calibration and User Profile Creation

1. In a Markdown editor, review the contents of `agents/tone-calibration.md` and adjust as necessary.
2. Fill out `agents/user-profile.md` with the user's context, using the template fields as a guide. Keep entries factual and scoped — see **Privacy & Data Handling** below.
3. Upload `agents/Cara/cara-session-log.md` to your memory storage (for example, Notion via MCP).

### Agent Creation

1. Create an agent in LibreChat to serve as the primary agent (Cara). Refer to LibreChat's [agent instructions](https://www.librechat.ai/docs/features/agents) for details.
2. Open `agents/Cara/cara-project-instructions.md` and copy the contents into the **Instructions** field in LibreChat's agent dialog.
3. Under **File Context**, upload `agents/Cara/cara-identity.md` and your completed `tone-calibration.md` and `user-profile.md` files.
4. Select a model for the agent.
5. Connect your external storage to the agent.
6. Begin a chat with the agent.

### Finalizing Setup

1. Ensure the agent can see the tone calibration, Cara identity and user profile documents.
2. Test write access to `cara-session-log.md` and `cara-identity.md` (alternatively: manually write the agent's output to external storage with a file editor).
3. Initialize `to-do.md` with the agent.

## Optional Setup - Erika

Erika's agent creation in LibreChat follows a similar process.

### Initial User Profile Creation

1. Within `agents/Erika/erika-project-instructions.md`, using the template fields as a guide, enter the user's language learning profile.
2. Upload `agents/Erika/erika-progress-log.md` to your memory storage.

### Agent Creation - Erika

1. Create an agent in LibreChat to serve as language learning assistant (Erika).
2. Copy the contents of the completed `erika-project-instructions.md` into the **Instructions** field in LibreChat's agent dialog.
3. Under **File Context**, upload `agents/Erika/erika-identity.md` and your completed `tone-calibration.md` and `user-profile.md` files.
4. Select a model for the agent.
5. Connect your external storage to the agent.
6. Begin a chat with the agent.

### Finalizing Setup - Erika

1. Ensure the agent can see the tone calibration, Erika identity and user profile documents.
2. Test write access to `erika-progress-log.md` and `erika-identity.md` (alternatively: manually write the agent's output to external storage with a file editor).
3. Begin designing a study plan. Save progress as `erika-study-plan.md`.

## Privacy & Data Handling

User profile documents in this system are designed to hold meaningful personal context, which means they can include sensitive information.

Please consider the following before integrating this system into your own workflow:

- The current recommended LibreChat setup stores your profile and log files locally. If you choose to run these agents with local models, Jade Path will never send your data to third parties.
- If you connect these agents to a third-party cloud LLM provider, be aware that data from your user profile and logs, in whole or in part, may be sent to that provider for processing. Review that provider's data handling policies before sending sensitive content.
  For example, OpenRouter has docs on [Data Collection](https://openrouter.ai/docs/guides/privacy/data-collection) and [Provider Logging](https://openrouter.ai/docs/guides/privacy/provider-logging) that are worth understanding before use.
- Jade Path can easily be adapted to work in web-based environments (claude.ai, Perplexity, etc.) and use MCP connectors (Notion, Dropbox) for external storage of your supporting documents. Please review privacy policies and terms of use carefully for each service involved in request handling.
- Consider what level of detail actually needs to live in a profile document. Specificity helps the agent, exhaustive detail may not be necessary.

## Repo Structure

<table>
  <thead>
    <tr>
      <th colspan="2">File / Folder</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td colspan="2"><code>README.md</code></td>
      <td>This file</td>
    </tr>
    <tr>
      <td colspan="2"><code>agents/</code></td>
      <td>Files for agent installation</td>
    </tr>
    <tr>
      <td></td>
      <td><code>tone-calibration.md</code></td>
      <td>Shared tone/voice conventions across agents</td>
    </tr>
    <tr>
      <td></td>
      <td><code>user-profile.md</code></td>
      <td>Generic user profile template</td>
    </tr>
    <tr>
      <td></td>
      <td><code>Cara/</code></td>
      <td>Cara project instructions, identity, and session-log format</td>
    </tr>
    <tr>
      <td></td>
      <td><code>Erika/</code></td>
      <td>Erika project instructions, identity, and progress-log format</td>
    </tr>
    <tr>
      <td colspan="2"><code>agent-design.md</code></td>
      <td>What the agents are, how scoped identity documents work, core design principles</td>
    </tr>
    <tr>
      <td colspan="2"><code>context-management.md</code></td>
      <td>Context budget decisions, document trimming rationale, platform behavior observed in testing</td>
    </tr>
    <tr>
      <td colspan="2"><code>memory-architecture.md</code></td>
      <td>Memory format evolution and tradeoffs (stub, planned for development)</td>
    </tr>
    <tr>
      <td colspan="2"><code>infrastructure.md</code></td>
      <td>Self-hosted stack and provider setup (stub, planned for development)</td>
    </tr>
  </tbody>
</table>

## Near-term Roadmap

- Commit results of ongoing model testing across agents (current: Qwen model efficacy testing with Erika).
