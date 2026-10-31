# EZAT Router agent skills

Agent skills for [EZAT Router](https://router.ezat.io), which gives you leading AI models through one API key and one prepaid balance.

## ezat-router

Teaches an AI agent to set up and debug tools, SDKs and code that call EZAT Router: API keys, base URLs, Claude Code, the Anthropic and OpenAI SDKs, choosing models, balance and billing, and errors. It holds no keys, prices or model lists: it tells the agent to read those live.

Full guide: [docs.router.ezat.io/agents/skill](https://docs.router.ezat.io/agents/skill)

### Claude Code

Run these in a Claude Code session:

```
/plugin marketplace add ezat-org/skills
/plugin install ezat-router@ezat
```

Claude Code loads the skill when a task matches. To call it yourself, type `/ezat-router:ezat-router` followed by what you want.

To update, run `/plugin marketplace update ezat`, or turn on auto-update for the `ezat` marketplace under **Marketplaces** in `/plugin`.

### Codex and other agents

With the [skills CLI](https://github.com/vercel-labs/skills):

```bash
npx skills add ezat-org/skills
```

Add `-g` to install it for every project, and `-a <agent>` to pick the agent. Update with `npx skills update`.

### Zip (claude.ai, Claude Desktop, or by hand)

Download [ezat-router.zip](https://docs.router.ezat.io/skill/ezat-router.zip). It contains one `ezat-router/` folder with `SKILL.md` in it.

## How this repository is updated

The skill is written and published with the EZAT Router docs. This repository is a copy of the published version: a workflow fetches it from docs.router.ezat.io every hour and commits any change. Don't edit `skills/` here, because the next sync overwrites it. Report problems in [Issues](https://github.com/ezat-org/skills/issues).

## 中文

EZAT Router 的 Agent Skill：让 AI agent 自己配置和排查调用 EZAT Router 的工具、SDK 和代码。

- Claude Code：在会话里运行 `/plugin marketplace add ezat-org/skills`，再运行 `/plugin install ezat-router@ezat`。
- 其他 agent：`npx skills add ezat-org/skills`。
- 完整说明：[docs.router.ezat.io/zh/agents/skill](https://docs.router.ezat.io/zh/agents/skill)

## License

[MIT](LICENSE)
