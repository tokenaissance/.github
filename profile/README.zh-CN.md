> [English](README.md) | 简体中文

<div align="center">
  <h1>Tokenaissance</h1>
  <p><b>智能体运行时与智能体基础设施</b></p>
  <p>
    关掉标签页，它还在继续干活的运行时。<br/>
    多个智能体共享、分支、合并的文件系统。<br/>
    装进你手上 harness 的插件。
  </p>
  <p>
    <a href="https://tokenaissance.com"><b>tokenaissance.com</b></a> ·
    <a href="#start-zh">从这里开始</a> ·
    <a href="#plugins-zh">dsh 插件</a>
  </p>
</div>

我们做的是智能体不再是演示之后、它下面那几层：在哪儿跑、干完的活留在哪儿、代码按什么
规矩分层。今天扛起这件事的是五个项目 —— 两个运行时、一个底座、一层插件、一套方法。
每一个都存在，因为智能体一旦跑得比一次对话更久，总有东西会坏。

## 技术栈

| 层 | 项目 | 做什么 |
| --- | --- | --- |
| **运行时** | **[FastAgent](https://github.com/tokenaissance/fastagent)** | 一个 Go 单二进制：循环、工具、沙箱、渠道，跑在你自己的机器上。 |
| **运行时** | **tokenaissance-pi-durable** | 同一件事放到 Cloudflare Workers + Durable Objects 上。私有，开发中。 |
| **底座** | **[LayerFS](https://github.com/Ephemeral-AI-Lab/layerfs)** | 临时 Workspace 架在持久共享历史之上；智能体的每一步都是一个 Commit。 |
| **插件** | **[dsh-layerfs](https://github.com/tokenaissance/dsh-layerfs)** | 把 LayerFS 装进 DeepSeek Harness 会话，在 Work / History 页签里实时画出来。 |
| **方法** | **[clean-architecture](https://github.com/tokenaissance/clean-architecture)** | 四层分层、一个依赖方向，再加一道删掉过度设计的闸门。 |

## 运行时

### FastAgent

**一个 Go 单二进制，把模型变成你自己托管的智能体。** 智能体循环、工具、沙箱、渠道和
管理后台都跑在一个进程里；默认存储是 SQLite。

- **任何模型。** OpenAI、Anthropic、DeepSeek、Mistral、Groq、Ollama、OpenRouter，或任何
  OpenAI 兼容端点；单个智能体可以覆盖系统默认。
- **会留下东西的沙箱。** E2B 或 Docker；工具调用前自动灌入技能和工作区，调用后把沙箱里
  的文件镜像回持久存储。
- **是渠道，不只是聊天框。** 每个智能体可以绑定 Telegram、Discord、Slack，会话按渠道和
  会话 id 隔离。
- **应用能直接调的 API。** OpenAI 兼容的流式 `/v1/chat/completions`、SSE 聊天端点、MCP
  服务端、JSON-RPC 插件系统，以及按用户计的配额。

```bash
curl -fsSL https://raw.githubusercontent.com/tokenaissance/fastagent/fastagent/install.sh | bash
```

**[github.com/tokenaissance/fastagent](https://github.com/tokenaissance/fastagent)**

### tokenaissance-pi-durable

**同一件事放到边缘：Cloudflare Workers + Durable Objects。** 一个 Worker 同时托管产品、
智能体会话和沙箱：`AgentSession` 对象跑 Pi harness，转写存在它自己的 SQLite 里；
`Sandbox` 对象则一个会话拥有一个容器。

- **因为构造而持久。** 活等的是 alarm，不是某个请求；对象被驱逐后会自己醒来，接着做它
  刚才在做的那件事。
- **重连是设计内的事。** SSE 适配层把会话的事件流变成 `text/event-stream`，重连与背压都
  只在这一个边界上处理。
- **一次部署。** TanStack Start 应用、Durable Objects 和容器镜像从同一个 Worker 发出去。

稳定之前保持私有 —— 到 [Discord](https://discord.gg/bXntC3xhG2) 或下方的微信找我们。

## 底座

### LayerFS

**临时 Workspace，架在持久共享历史之上。** 每个智能体挂载一个完整可写的 Workspace，底
座不复制；干完活，把它改的东西提交上去，然后走人 —— 历史留下。

- **每一步都是一次 Commit。** `fs` 调用直接落成 Commit；一次 Bash 调用是挂载 → 执行 →
  提交 → 卸载，命令改过的一切都在那一次 Commit 里。
- **并行智能体是默认形态。** 从任意 Commit 拉 Branch，做完就 merge；永远不该冲突的文件
  写进 `/.lww`。
- **便宜到可以一直做。** 快照共享存储：上游基准里，一个真实项目的 157 个快照只占
  100 MB，比 Btrfs + zstd 小 19.9 倍。

LayerFS 是 [Ephemeral AI Lab](https://github.com/Ephemeral-AI-Lab/layerfs) 的开源项目；
我们在 dsh 下用它、往上提交修复，并在[这里](https://github.com/tokenaissance/layerfs)
镜像一份。
**[layerfs.ai](https://layerfs.ai)** · **[文档](https://layerfs.ai/docs/)**

<a id="plugins-zh"></a>
## DeepSeek Harness 插件

[DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) 没有特权内核：模型
适配器、工具注册表、会话日志、智能体循环，全都是插件。我们也用同一种方式交付能力 ——
不 fork、不打补丁，装进你已经在跑的 profile。

### dsh-layerfs

**每个智能体的 Branch，实时画出来。** 把 LayerFS 放进 dsh 会话：Bash 调用变成
Workspace，Workspace 变成 Commit，对话里多出 **Work** 和 **History** 两个页签，把每个
智能体（包括子智能体）做过的事画出来。

- 在对话里拉 Branch、在 Sandbox 里试、合并、发布。
- History 页签随 Commit 落地而生长 —— 没有一张是截图。
- 一个 bundle 加一个插件宿主：没有改 dsh 的源码。

**[dsh-layerfs](https://github.com/tokenaissance/dsh-layerfs)** ·
**[安装](https://github.com/tokenaissance/dsh-layerfs#-install)** ·
**[LayerFS × DSH 文档](https://layerfs.ai/docs/integrations/dsh/)**

在做中的还有：把通过 MCP 取回的 Excalidraw 场景渲染在侧边栏里的 Design 页签，不用刷新
页面。

插件统一发布在 **[`dsh-plugin` 话题](https://github.com/topics/dsh-plugin)** 下 —— dsh
用户就是这样找到它们的。

## 方法

### clean-architecture

**一套判断方法，不是脚手架：先诊断，再分层。** 四层、一个依赖方向，以及马斯克五步法，
在过度设计上线之前把它删掉。

- **与栈无关。** 同一套四层模型，映射到 Express、Next.js、Go 或 Spring，而不是复制模板。
- **一道反过度设计的闸门。** 任何「上复杂方案」的建议，先走一遍提问 → 删除 → 简化 →
  加速 → 自动化。
- **能活下来的决策。** 架构决策附带 L2/L3 文档骨架，依赖规则的违反在文件头就能看见，而
  不是埋在 import 里。

它是我们自己仓库的默认评审技能。
**[github.com/tokenaissance/clean-architecture](https://github.com/tokenaissance/clean-architecture)**

## 技能

其余装进智能体的东西。它们大多由原作者维护上游；本组织里的仓库是与上游同步的 fork，
功劳归上游。

- **智能体编程** —— [superpowers](https://github.com/tokenaissance/superpowers)、
  [planning-with-files](https://github.com/tokenaissance/planning-with-files)、
  [i-have-adhd](https://github.com/tokenaissance/i-have-adhd)、
  [asd-ste100-skill](https://github.com/tokenaissance/asd-ste100-skill)
- **设计与前端** —— [impeccable](https://github.com/tokenaissance/impeccable)、
  [hallmark](https://github.com/tokenaissance/hallmark)、
  [agent-html-skills](https://github.com/tokenaissance/agent-html-skills)、
  [frontend-slides](https://github.com/tokenaissance/frontend-slides)
- **商业、GTM 与销售** —— [startup-founder-skills](https://github.com/tokenaissance/startup-founder-skills)、
  [gtm-engineer-skills](https://github.com/tokenaissance/gtm-engineer-skills)、
  [Product-Manager-Skills](https://github.com/tokenaissance/Product-Manager-Skills)、
  [pm-skills](https://github.com/tokenaissance/pm-skills)、
  [mastering-technical-sales](https://github.com/tokenaissance/mastering-technical-sales)
- **研究与上网** —— [last30days-skill](https://github.com/tokenaissance/last30days-skill)、
  [ai-signal](https://github.com/tokenaissance/ai-signal)、
  [OpenCLI](https://github.com/tokenaissance/OpenCLI)、
  [firecrawl](https://github.com/tokenaissance/firecrawl)、
  [searxng-helm](https://github.com/tokenaissance/searxng-helm)
- **垂直领域** —— [barker-stablecoin-skills](https://github.com/tokenaissance/barker-stablecoin-skills)（DeFi）、
  [serenity-aleabitoreddit](https://github.com/tokenaissance/serenity-aleabitoreddit)（供应链）、
  [mcp_excalidraw](https://github.com/tokenaissance/mcp_excalidraw)（系统设计）、
  [qiqing-liuyu](https://github.com/tokenaissance/qiqing-liuyu)
- **合集与模板** —— [yichen-skills](https://github.com/tokenaissance/yichen-skills)、
  [hello-agents](https://github.com/tokenaissance/hello-agents)、
  [knowledge-wiki-template](https://github.com/tokenaissance/knowledge-wiki-template)

**社区 Fork，功劳归原作者：**

| 技能 | 作者 | 提供什么 |
| --- | --- | --- |
| [opc-skills](https://github.com/tokenaissance/opc-skills) | [ReScienceLab](https://github.com/ReScienceLab) | SEO / GEO 技能，上游 3.97 万次安装 |
| [seo-geo-claude-skills](https://github.com/tokenaissance/seo-geo-claude-skills) | [aaron-he-zhu](https://github.com/aaron-he-zhu) | 20 个独立 SEO / GEO 技能 |
| [marketing-skills](https://github.com/tokenaissance/marketing-skills) | [kostja94](https://github.com/kostja94) | 160+ 营销技能 |
| [pm-skills](https://github.com/tokenaissance/pm-skills) | [phuryn](https://github.com/phuryn) | 100+ PM 技能，含 gtm-strategy |
| [agent-toolkit](https://github.com/tokenaissance/agent-toolkit) | [sanity-io](https://github.com/sanity-io) | SEO / AEO 最佳实践 |
| [founder-skills](https://github.com/tokenaissance/founder-skills) | [ognjengt](https://github.com/ognjengt) | 给创始人的市场进入计划 |

<a id="start-zh"></a>
## 从这里开始

1. **跑一个智能体。** 装上 FastAgent，打开 `http://localhost:18953`，配一个模型。
   ```bash
   curl -fsSL https://raw.githubusercontent.com/tokenaissance/fastagent/fastagent/install.sh | bash
   fastagent
   ```
2. **给它一个文件系统。** [LayerFS × DSH](https://github.com/tokenaissance/dsh-layerfs#-install)
   让智能体拥有 Branch、Workspace 和共享历史。
3. **加能力，而不是加 fork。** 看
   [`dsh-plugin` 话题](https://github.com/topics/dsh-plugin)，或者从上面任意一个
   `SKILL.md` 开始。
4. **用托管版本。** [tokenaissance.com](https://tokenaissance.com) 是同一个团队为金融与
   科学领域交付的智能体。

## 社区

需要帮助，或者想一起做点什么？欢迎找我们。

<p align="center">
  <a href="https://x.com/tokenaissance"><img src="https://img.shields.io/badge/X-@tokenaissance-black?style=for-the-badge&logo=x&logoColor=white" alt="X" /></a>
  <a href="https://discord.gg/bXntC3xhG2"><img src="https://img.shields.io/badge/Discord-Join%20Server-%23143D52?style=for-the-badge&logo=discord&logoColor=white" alt="Discord" /></a>
</p>

<p align="center">
  <b>个人微信</b><br/>
  <img src="assets/wechat-qr.jpg" width="160" alt="WeChat QR" />
</p>

上面任意仓库都欢迎 issue 和 PR。

---
<div align="center">
  <i>以自主智能，赋能 Token 复兴。</i>
</div>
