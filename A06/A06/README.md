# OpenClaw — Setup Guide

CART 498 · Generative AI Systems · Assignment A06
Dan Truong, Jose Barrera, Angeline Duong

This guide gets OpenClaw running on your own machine in about 30 minutes, using our free local setup.

## What is OpenClaw

Chatbots answer questions. OpenClaw finishes tasks.

Think of an AI model as a **brain** — smart, but it normally just talks, living on a company's website. OpenClaw gives that brain a **body** that lives on your own computer: you text it like a friend (we used Telegram), it thinks with the brain, and its body does the work — opens files, runs commands, browses the web. It keeps a notebook so it remembers you tomorrow, and it has its own alarm clock (the **heartbeat**) that can wake it up to keep working while you sleep, as long as the host computer stays on.

It's free and open source, formerly called "Clawdbot" then "Moltbot," made by Peter Steinberger.

**Key terms:**
- **Agent** — an AI that does things, not just talks; give it a goal, it works out the steps
- **Brain** — the language model doing the thinking (Claude, GPT, or a free local model)
- **Gateway** — the always-on program that routes messages and tools between brain and body
- **Skills** — add-ons that teach it new abilities, like installing apps on a phone
- **Memory** — plain text notebooks it rereads, so it remembers you across days
- **Heartbeat** — a configurable timer that can wake it without a human asking

**Interaction paradigm:** You set the goal and the limits; it picks the steps; you check the result — similar to managing an employee. The basic unit is a **task** (a job you text it) and sometimes a **trigger** (a job that starts itself on a timer). You don't control it with buttons — you control it with a few plain text files: `SOUL.md` (its personality and rules), `TOOLS.md` (what it may touch), `HEARTBEAT.md` (what it does on its own), and a config file (which model, which chat apps).

## Requirements

**Hardware (our free route)**
- Apple Silicon Mac, at least 16 GB RAM, at least 50 GB free disk space

**Hardware (paid route, alternative)**
- Any laptop, or a rented server (roughly $5–20/month)

**Operating system**
- We tested two paths: macOS (Mac + a Ubuntu VM via UTM, free local model) and Windows (OpenClaw Companion app, hosted API). Both are supported.

**Accounts needed**
- A Telegram account, to create a bot via [@BotFather](https://t.me/BotFather) (`/newbot`) — we used a burner account
- Either [Ollama](https://ollama.com) installed locally (free, no API key) **or** an API key from a hosted provider (OpenAI, Anthropic), billed per token
- A Google account, if you want OpenClaw to read/edit Google Slides/Docs via browser control

## Costs

- **Our route: $0.** OpenClaw itself (MIT licensed) plus a free local model via Ollama. Slower than a hosted model, but prompts never leave the machine.
- **Paid route:** billed per token by the provider (Claude, GPT, etc.). This can get expensive fast — agents resend the **entire conversation history** on every step, so long or multi-step tasks burn through credit quickly. We spent **$15 total** on hosted-API tasks before switching to the free Ollama route.
- Set a budget limit with your provider if one is available.

## Setup steps (~30 minutes)

1. **Create a VM in UTM** — Ubuntu 24.04 ARM, 4 CPU, 8 GB RAM, 40 GB disk. Take a snapshot once it's clean, so you can roll back.
2. **On the Mac: install Ollama and pull a model** — `ollama pull qwen3:8b`
3. **In the VM: install OpenClaw** — `curl -fsSL https://openclaw.ai/install.sh | bash`
4. **Run the setup wizard** — `openclaw onboard --install-daemon`
5. **Point it at the model** — set Ollama to bind on the LAN and connect to `http://[Mac IP]:11434` (not `127.0.0.1`, and no `/v1` suffix — that breaks tool-calling)
6. **Connect a burner Telegram bot** — message [@BotFather](https://t.me/BotFather), run `/newbot`, paste the token into OpenClaw
7. **Check it, then say hi** — `openclaw gateway status` should report "RPC probe: ok"; message your bot to confirm

**To harden it with execution approvals** (recommended — see our safety protocol below):
```
openclaw config set tools.exec.ask always
openclaw approvals set --stdin
{"version":1,"defaults":{"security":"allowlist","ask":"always","askFallback":"deny"}}
openclaw gateway restart
```

**Windows alternative:** the OpenClaw Companion app walks through the same idea in 8 screens — install, accept the risk warning, choose where the gateway runs, pick a permissions preset, let the gateway auto-configure, choose an AI brain, sign in to a provider, then chat. Took about 30 minutes, same as the Mac/VM route, using the same permissions preset (Fine-tune, so execution approvals could be turned on). Full screenshots are in our shared Drive folder.

**Common setup snags:** Ollama's context window defaults to 4096 tokens, which is too small for agentic use — raise it to 32k before connecting OpenClaw. And setting the API key with `openclaw config set` alone isn't enough — OpenClaw checks a separate internal auth store, so you need `openclaw models auth paste-api-key --provider ollama` to actually register it, even with a placeholder value.

[Insert your pipeline/architecture diagram here — export the "pipeline" slide as a PNG and embed with `![Pipeline diagram](./pipeline.png)`]

**The pipeline, in short:** your phone (Telegram) → Telegram's servers → the gateway (in our VM) → the brain (AI model, on the Mac's GPU or an online AI) → hands (terminal, files, browser, skills) → memory (plain Markdown files: `SOUL.md`, notes, daily logs). The heartbeat timer can also wake the whole loop without any message from you. Solid boxes run on our own hardware; only the Telegram leg is someone else's server.

## Our safety protocol

We deliberately gave OpenClaw full, unsandboxed access to a real machine — files, terminal, our logged-in browser, Gmail/Drive, a Telegram bot — to see the real risks for ourselves, not a limited demo environment. No app ever asked us for permission; it simply used what we were already signed in to. Because of that, before running it we:

- Backed up our files to an SD card first
- Installed no third-party ClawHub skills
- Set it so every command needs our approval before running
- Uninstalled it once the project was done

This was **not completely safe**, and we knew that — two things a backup can't fix are still there: hidden instructions can trick it, and our data still goes to a third-party AI provider if we use one. But it meaningfully lowered the risk.

## Known limits, risks, and biases

- **Privacy myth:** "it all stays on my computer" is only true of the body. If the brain is OpenAI or Anthropic, every prompt — and any file it reads — is sent to and stored by that company.
- **Full access, your responsibility:** with full access it can delete or change local files, even by mistake, and you are responsible for what it does.
- **Prompt injection:** a web page, email, or document can hide instructions it may follow instead of yours. In our own test, a hidden instruction embedded in a slide was correctly flagged rather than obeyed — but this isn't guaranteed.
- **ClawHub skill marketplace risk:** an early-2026 audit found 341 malicious skills out of about 2,857 on ClawHub, many installing a Mac password stealer; researchers also reported 42,000+ OpenClaw instances left exposed on the internet in February 2026 (Koi Security via The Hacker News 2026; Unit 42 2026). Untrusted skills can direct the agent to run a stranger's code. We installed none.
- **Where data actually goes:** chat text → Telegram's servers; prompts → stay on the Mac with a local model, or go to the AI company with a hosted one; memory → plain text files in the VM; passwords/keys → can leak if stored in files the agent can read.
- **Local-model reliability:** smaller free models (like `qwen3:8b` via Ollama) are noticeably less consistent than hosted frontier models at correctly deciding when to call a tool versus just answering in words — a real trade-off for the $0 cost.
- **Licensing:** OpenClaw is MIT licensed (use, modify, even sell, no warranty). The Qwen3 model is Apache 2.0 (permissive — check each model you download). Cloud API outputs are governed by the provider's terms (usually you own outputs; read their data rules). ClawHub skills often state no license at all. Authorship of text the agent rewrites (e.g. our own slides) is legally unclear — copyright requires a human author, and the US doesn't protect purely machine-made work; Canada hasn't settled this yet.

## What we made

We used Claude Sonnet as OpenClaw's brain to fact-check this presentation and to help build the presentation itself — sending it our slides and having it check facts and tighten the wording. We reviewed every suggested change before keeping it.

## Connection to our final project

Not decided yet. Candidates from what we learned building this: an installation that keeps changing between visits, a "performer" the audience texts during a show, or a studio helper running long batch jobs. Any of these would extend OpenClaw with our own skill (so we know exactly what code runs) and connect it to creative tools like p5.js or TouchDesigner via MCP. Main risks to design around: audience messages becoming prompt injection, lag or a crash mid-show, and cost spikes or outputs we can't moderate.

## References

Barrack.ai. 2026. "OpenClaw Is a Security Nightmare — Here's the Safe Way to Run It." https://blog.barrack.ai/openclaw-security-vulnerabilities-2026/.

ClawDocs. 2026. "Architecture Overview." https://clawdocs.org/architecture/overview.

Palo Alto Networks Unit 42. 2026. "OpenClaw AI Supply Chain Risk." https://unit42.paloaltonetworks.com/openclaw-ai-supply-chain-risk/.

The Hacker News. 2026. "Researchers Find 341 Malicious ClawHub Skills Stealing Data from OpenClaw Users." February 2026. https://thehackernews.com/2026/02/researchers-find-341-malicious-clawhub.html.

The Neuron. 2026. "OpenClaw Setup Guide: How to Build Your Own AI Agent Team." February 2026. https://www.theneuron.ai/explainer-articles/openclaw-personal-ai-agent-setup-guide-an-use-cases-february-2026/.

Vo, Claire. 2026. "OpenClaw: The Complete Guide to Building, Training, and Living with Your Personal AI Agent." Lenny's Newsletter. https://www.lennysnewsletter.com/p/openclaw-the-complete-guide-to-building.

*(Add the OpenClaw docs (docs.openclaw.ai), and the Ollama and UTM pages you actually used, with access dates.)*
