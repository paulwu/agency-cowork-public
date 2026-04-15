# Agency Cowork Public

This repository contains reusable artifacts—such as custom Copilot agent definitions—that can be pulled into any [Agency Cowork](https://github.com/paulwu/agency-cowork-public) project.

## Getting Started

### Prerequisites

- A GitHub repository with [Agency Copilot CLI](https://docs.github.com/en/copilot) set up.

### 1. Create the Pull Cowork Agent

From the Agency Copilot CLI, ask it to create a custom copilot agent using the agent definition at:

```
https://github.com/paulwu/agency-cowork-public/blob/main/.github/agents/pull-cowork-agent.agent.md
```

### 2. Invoke the Agent

Once the agent is created, invoke it from the CLI by typing:

```
/agent
```

Then select **pull-cowork-agent** from the list.

### 3. Pull Artifacts

After selecting the agent you can:

- **Pull a specific artifact** — ask the agent to pull a named agent, skill, instruction, or doc from this repository.
- **Get help** — type `help` to see what artifacts are available and how to use them.

## Repository Structure

```
.github/
  agents/          # Custom Copilot agent definitions
```

## Contributing

Contributions are welcome! To add a new artifact, open a pull request with your agent, skill, or instruction file placed in the appropriate directory under `.github/`.

## License

This project is provided as-is for use within Agency Cowork engagements.
