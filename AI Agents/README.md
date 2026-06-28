# AI Agents

LLM-powered scenarios. The non-negotiable pattern: **structured output + validation gate** — the model returns strict JSON, a parser validates it, and nothing acts on the result until it passes. Free-text hallucinations fall to a human, never to a live action.

| Blueprint | What it does |
|---|---|
| [AI Support Ticket Triage](ai-support-ticket-triage/) | Classify, prioritize, route tickets; escalate urgent to on-call |
| [AI Content Repurposer](ai-content-repurposer/) | One article → LinkedIn + thread + newsletter, human-approved |

Prompts are versioned in [`examples/prompts`](../examples/prompts/). Every blueprint works with **OpenAI or Anthropic Claude** — same JSON contract, swap the module.

**Planned:** RAG FAQ responder, lead scorer, call summarizer, document extractor. See [roadmap](../docs/roadmap.md).
