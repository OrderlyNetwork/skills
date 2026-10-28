# Orderly Network Skills

AI Agent Skills for Orderly Network — perpetual futures trading infrastructure.

Two skills, split by intent:

- **orderly-onboarding** — operate Orderly: the `orderly` CLI, accounts, orders, positions, funds, broker IDs, and MCP setup.
- **orderly-sdk** — build a DEX or trading UI with the React SDK.

Everything else (REST/WS endpoints, auth signing, SDK hook signatures, contracts, workflows, Orderly One) lives in the Orderly MCP server, which is the reference layer these skills point at.

## Installation

### Quick Install (All Skills)

**Global** (available across all projects):

```bash
npx skills add OrderlyNetwork/skills --all -g
```

**Local** (project-specific):

```bash
npx skills add OrderlyNetwork/skills --all
```

### Target Specific Agents

Install for all agents:

```bash
npx skills add OrderlyNetwork/skills --all --agent '*' -g
```

Install for specific agents:

```bash
# Claude Code
npx skills add OrderlyNetwork/skills --all --agent claude-code -g

# Cursor
npx skills add OrderlyNetwork/skills --all --agent cursor -g

# OpenCode
npx skills add OrderlyNetwork/skills --all --agent opencode -g

# VS Code
npx skills add OrderlyNetwork/skills --all --agent vscode -g

# Codex
npx skills add OrderlyNetwork/skills --all --agent codex -g
```

### Install Specific Skills

List available skills:

```bash
npx skills add OrderlyNetwork/skills --list
```

Install specific skills:

```bash
npx skills add OrderlyNetwork/skills --skill orderly-onboarding --skill orderly-sdk
```

### Global vs Local Installation

**Global (`-g` flag):**

- Skills available across all your projects
- Installed to user-level directory
- Good for agents working across multiple repos

**Local (no flag):**

- Skills specific to the current project
- Creates `.skills/` in the repo
- Can be committed to version control

## Available Skills

| Category            | Skill                | Description                                                                                                     |
| ------------------- | -------------------- | --------------------------------------------------------------------------------------------------------------- |
| **Getting Started** | `orderly-onboarding` | **START HERE** — platform overview, agent tools (CLI/MCP/SDK), quickstart, broker IDs, chains, token, key links |
| **SDK / React**     | `orderly-sdk`        | Build a DEX: template-first setup, packages/providers, wallet connection, UI, hooks, theming, debugging         |

## Skill Details

### orderly-onboarding

Start here for anything Orderly-related — operating an account or understanding the platform.

- What Orderly is: architecture, key characteristics, use cases, supported chains
- Agent surfaces: when to use the CLI, the MCP, or a skill
- MCP server setup and the full tool reference
- `orderly` install and version requirements, command discovery
- Testnet quickstart and account setup
- Command map for accounts, orders, algo/TP-SL, positions, market data, and funds
- CLI gotchas (`--account`, hex quoting, symbol case, output formats) and safety rules
- Broker ID creation: CLI path and the programmatic x402 flow
- $ORDER token, key links, recommended next steps, and common questions

### orderly-sdk

For building a DEX or trading interface.

- Fork `dex-template` as the default path
- Correct package and import map (there is no `@orderly.network/react`)
- Provider hierarchy, `OrderlyAppProvider` configuration
- Wallet connection: EVM, Solana, Privy
- Page components and hooks, with pointers to the MCP for full signatures
- `--oui-*` theming and chart colors
- Tailwind/Vite build setup
- Debugging table for the common failure modes

## Recommended Setup for AI Agents

```bash
# 1. Install the MCP server (reference layer: docs, endpoints, SDK symbols, contracts)
npx @orderly.network/mcp-server init --client <your-client>
#    or point the client at the remote server: https://mcp.orderly.network

# 2. Install the skills globally
npx skills add OrderlyNetwork/skills --all --agent '*' -g

# 3. Install the CLI for live operations
curl -fsSL https://orderly.network/cli.sh | sh
orderly --version
```

**Minimal setup:** install only `orderly-onboarding` and let the agent load `orderly-sdk` when the task is to build.

## Development

### Prerequisites

- Node.js 20+
- Yarn

### Setup

```bash
yarn install
```

### Validate Skills

```bash
yarn validate
```

### Lint

```bash
yarn lint
yarn lint:fix
```

### Format

```bash
yarn format
yarn format:check
```

### Releasing

Distribution is git-based: pushing to `master` is the release. `npx skills add OrderlyNetwork/skills` reads the repository directly, so new installs and `npx skills update` pick up the latest commit. Tags (`vX.Y.Z`) are human-readable milestones only and do not trigger a publish workflow. The npm package `@orderly.network/skills` is private and is not part of the distribution path.

## Project Structure

```
orderly-skills/
├── skills/
│   ├── orderly-onboarding/
│   │   └── SKILL.md
│   └── orderly-sdk/
│       └── SKILL.md
├── scripts/
│   └── validate-skills.ts
├── package.json
├── tsconfig.json
├── eslint.config.mjs
├── .prettierrc
└── README.md
```

## Links

- [Orderly Documentation](https://orderly.network/docs)
- [Orderly SDK](https://github.com/OrderlyNetwork/js-sdk)
- [Orderly CLI](https://github.com/OrderlyNetwork/cli)
- [DEX Template](https://github.com/OrderlyNetwork/dex-template)
- [Orderly Discord](https://discord.gg/OrderlyNetwork)
- [Skills Documentation](https://skills.sh)

## License

MIT
