> English | [简体中文](README.zh-CN.md)

<div align="center">
  <h1>Tokenaissance</h1>
  <p><b>Agent runtime and agent infrastructure.</b></p>
  <p>
    A runtime that keeps working after the tab closes.<br/>
    A filesystem agents share, branch and merge.<br/>
    Plugins for the harness you already run.
  </p>
  <p>
    <a href="https://tokenaissance.com"><b>tokenaissance.com</b></a> ·
    <a href="#start">Start here</a> ·
    <a href="#plugins">dsh plugins</a>
  </p>
</div>

We build and run the layers under an agent that has stopped being a demo: where it runs,
where its work survives, and the rule the code is written by. Five projects carry that
work today — two runtimes, one substrate, a plugin layer and a method. Each one exists
because something breaks once an agent runs longer than one conversation.

## The stack

| Layer | Project | What it does |
| --- | --- | --- |
| **Runtime** | **[FastAgent](https://github.com/tokenaissance/fastagent)** | A single Go binary: the loop, the tools, the sandboxes and the channels, on your own machines. |
| **Runtime** | **tokenaissance-pi-durable** | The same job on Cloudflare Workers and Durable Objects. Private, in development. |
| **Substrate** | **[LayerFS](https://github.com/Ephemeral-AI-Lab/layerfs)** | Ephemeral Workspaces over durable shared history; every step an agent takes is a Commit. |
| **Plugins** | **[dsh-layerfs](https://github.com/tokenaissance/dsh-layerfs)** | LayerFS inside a DeepSeek Harness session, drawn live in the Work and History tabs. |
| **Method** | **[clean-architecture](https://github.com/tokenaissance/clean-architecture)** | Four layers, one dependency direction, and a gate that deletes over-engineering. |

## Runtimes

### FastAgent

**A single Go binary that turns a model into an agent you host yourself.** The agent
loop, the tools, the sandboxes, the channels and the admin dashboard run in one process;
SQLite is the default store.

- **Any model.** OpenAI, Anthropic, DeepSeek, Mistral, Groq, Ollama, OpenRouter or any
  OpenAI-compatible endpoint, with a per-agent override of the system default.
- **Sandboxes that keep their work.** E2B or Docker; skills and workspace are hydrated
  before a tool call, and the sandbox's files are mirrored back to the store after it.
- **Channels, not a chat box.** Per-agent Telegram, Discord and Slack bots, with
  sessions isolated per channel and per chat.
- **An API an application can call.** OpenAI-compatible `/v1/chat/completions` with
  streaming, an SSE chat endpoint, MCP servers, a JSON-RPC plugin system, and per-user
  quotas.

```bash
curl -fsSL https://raw.githubusercontent.com/tokenaissance/fastagent/fastagent/install.sh | bash
```

**[github.com/tokenaissance/fastagent](https://github.com/tokenaissance/fastagent)**

### tokenaissance-pi-durable

**The same job at the edge, on Cloudflare Workers and Durable Objects.** One Worker
hosts the product, the agent session and the sandbox: an `AgentSession` object runs the
Pi harness and keeps the transcript in its own SQLite, while a `Sandbox` object owns one
container per session.

- **Durable by construction.** Work waits on alarms, not on a request; an evicted object
  wakes up and resumes the task it was in the middle of.
- **Reconnect is a design case.** An SSE adapter turns the session's event stream into
  `text/event-stream`, with reconnect and backpressure handled at that one boundary.
- **One deploy.** The TanStack Start app, the durable objects and the container image
  ship from the same Worker.

Private while it stabilizes — ask in [Discord](https://discord.gg/bXntC3xhG2) or on
WeChat below.

## Substrate

### LayerFS

**Ephemeral Workspaces over durable shared history.** Every agent mounts a complete,
writable Workspace over a shared Store without copying the base. It does its work,
commits what changed, and goes away; the history stays.

- **Every step is a commit.** An `fs` call lands as a Commit directly; a Bash call is
  mount → run → commit → unmount, and everything the command changed is in that Commit.
- **Parallel agents by default.** Branch from any Commit, merge when the branch is done,
  and name the files that must never conflict in `/.lww`.
- **Cheap enough to do constantly.** Snapshots share storage: the upstream benchmark
  keeps 157 snapshots of a real project in 100 MB, 19.9× smaller than Btrfs + zstd.

Open source by [Ephemeral AI Lab](https://github.com/Ephemeral-AI-Lab/layerfs); we run it
under dsh, send fixes upstream, and mirror it
[here](https://github.com/tokenaissance/layerfs).
**[layerfs.ai](https://layerfs.ai)** · **[Docs](https://layerfs.ai/docs/)**

<a id="plugins"></a>
## Plugins for DeepSeek Harness

[DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) has no privileged
core: the model adapter, the tool registry, the session log and the agent loop are all
plugins. We ship capability the same way — no fork, no patch, installed into the profile
you already run.

### dsh-layerfs

**Every agent's Branch, drawn live.** LayerFS inside a dsh session: Bash calls become
Workspaces, Workspaces become Commits, and the conversation grows **Work** and
**History** tabs that draw what each agent did, subagents included.

- Fork a Branch, test it in a Sandbox, merge and publish from the conversation.
- The History tab grows as commits land — nothing is a screenshot.
- A bundle plus a plugin host: no dsh source is modified.

**[dsh-layerfs](https://github.com/tokenaissance/dsh-layerfs)** ·
**[Install](https://github.com/tokenaissance/dsh-layerfs#-install)** ·
**[LayerFS × DSH docs](https://layerfs.ai/docs/integrations/dsh/)**

More in progress: a Design tab that renders Excalidraw scenes pulled through MCP, in the
sidebar, with no page refresh.

Plugins are published under the **[`dsh-plugin` topic](https://github.com/topics/dsh-plugin)** —
the way dsh users find them.

## Method

### clean-architecture

**A decision method, not a scaffold: diagnose first, then layer.** Four layers, one
dependency direction, and Musk's five-step algorithm to delete over-engineering before
it ships.

- **Stack-agnostic.** The same four-layer model, mapped onto Express, Next.js, Go or
  Spring instead of a copied template.
- **An anti-over-engineering gate.** Advice to adopt the complex solution runs
  question → delete → simplify → accelerate → automate first.
- **Decisions that survive.** Architecture choices come with L2/L3 documentation
  skeletons, so a dependency-rule violation is visible at the file header, not buried in
  imports.

It is the default review skill in our own repositories.
**[github.com/tokenaissance/clean-architecture](https://github.com/tokenaissance/clean-architecture)**

## Skills

The rest of what we install into agents. Most are maintained upstream by their authors;
the repository here is a fork kept in sync, and the credit stays with the source.

- **Agentic coding** — [superpowers](https://github.com/tokenaissance/superpowers),
  [planning-with-files](https://github.com/tokenaissance/planning-with-files),
  [i-have-adhd](https://github.com/tokenaissance/i-have-adhd),
  [asd-ste100-skill](https://github.com/tokenaissance/asd-ste100-skill)
- **Design and frontend** — [impeccable](https://github.com/tokenaissance/impeccable),
  [hallmark](https://github.com/tokenaissance/hallmark),
  [agent-html-skills](https://github.com/tokenaissance/agent-html-skills),
  [frontend-slides](https://github.com/tokenaissance/frontend-slides)
- **Business, GTM and sales** — [startup-founder-skills](https://github.com/tokenaissance/startup-founder-skills),
  [gtm-engineer-skills](https://github.com/tokenaissance/gtm-engineer-skills),
  [Product-Manager-Skills](https://github.com/tokenaissance/Product-Manager-Skills),
  [pm-skills](https://github.com/tokenaissance/pm-skills),
  [mastering-technical-sales](https://github.com/tokenaissance/mastering-technical-sales)
- **Research and the web** — [last30days-skill](https://github.com/tokenaissance/last30days-skill),
  [ai-signal](https://github.com/tokenaissance/ai-signal),
  [OpenCLI](https://github.com/tokenaissance/OpenCLI),
  [firecrawl](https://github.com/tokenaissance/firecrawl),
  [searxng-helm](https://github.com/tokenaissance/searxng-helm)
- **Domain** — [barker-stablecoin-skills](https://github.com/tokenaissance/barker-stablecoin-skills) (DeFi),
  [serenity-aleabitoreddit](https://github.com/tokenaissance/serenity-aleabitoreddit) (supply chain),
  [mcp_excalidraw](https://github.com/tokenaissance/mcp_excalidraw) (system design),
  [qiqing-liuyu](https://github.com/tokenaissance/qiqing-liuyu)
- **Collections and templates** — [yichen-skills](https://github.com/tokenaissance/yichen-skills),
  [hello-agents](https://github.com/tokenaissance/hello-agents),
  [knowledge-wiki-template](https://github.com/tokenaissance/knowledge-wiki-template)

**Community forks, credited to their authors:**

| Skill | Author | What it adds |
| --- | --- | --- |
| [opc-skills](https://github.com/tokenaissance/opc-skills) | [ReScienceLab](https://github.com/ReScienceLab) | SEO / GEO skills, 39.7K installs upstream |
| [seo-geo-claude-skills](https://github.com/tokenaissance/seo-geo-claude-skills) | [aaron-he-zhu](https://github.com/aaron-he-zhu) | 20 standalone SEO / GEO skills |
| [marketing-skills](https://github.com/tokenaissance/marketing-skills) | [kostja94](https://github.com/kostja94) | 160+ marketing skills |
| [pm-skills](https://github.com/tokenaissance/pm-skills) | [phuryn](https://github.com/phuryn) | 100+ PM skills, including gtm-strategy |
| [agent-toolkit](https://github.com/tokenaissance/agent-toolkit) | [sanity-io](https://github.com/sanity-io) | SEO / AEO best practices |
| [founder-skills](https://github.com/tokenaissance/founder-skills) | [ognjengt](https://github.com/ognjengt) | Go-to-market plans for founders |

<a id="start"></a>
## Start here

1. **Run an agent.** Install FastAgent, open `http://localhost:18953`, add a model.
   ```bash
   curl -fsSL https://raw.githubusercontent.com/tokenaissance/fastagent/fastagent/install.sh | bash
   fastagent
   ```
2. **Put a filesystem under it.** [LayerFS × DSH](https://github.com/tokenaissance/dsh-layerfs#-install)
   gives agents Branches, Workspaces and a shared history.
3. **Add a capability instead of a fork.** Read the [`dsh-plugin` topic](https://github.com/topics/dsh-plugin),
   or start from any `SKILL.md` above.
4. **Use the hosted product.** [tokenaissance.com](https://tokenaissance.com) is where the
   same team ships agents for finance and science.

## Community

Need help, or want to build with us? We would like to hear from you.

<p align="center">
  <a href="https://x.com/tokenaissance"><img src="https://img.shields.io/badge/X-@tokenaissance-black?style=for-the-badge&logo=x&logoColor=white" alt="X" /></a>
  <a href="https://discord.gg/bXntC3xhG2"><img src="https://img.shields.io/badge/Discord-Join%20Server-%23143D52?style=for-the-badge&logo=discord&logoColor=white" alt="Discord" /></a>
</p>

<p align="center">
  <b>Personal WeChat</b><br/>
  <img src="assets/wechat-qr.jpg" width="160" alt="WeChat QR" />
</p>

Issues and pull requests are welcome on any repository above.

---
<div align="center">
  <i>Empowering the Token Renaissance with Autonomous Intelligence.</i>
</div>
