# Support Triage Agent

An autonomous AI agent that reads incoming customer support emails, classifies them, assesses priority, and drafts a context-aware reply — grounded in a real product knowledge base rather than free-form generation.

Built with **n8n**, the **Anthropic API**, and **Gmail** as a live email trigger, for a fictional SaaS product ("TaskFlow," a project-management tool).

## Why This Is an Agent, Not Just Automation

This isn't a fixed pipeline where each step is hardcoded. The model itself decides, per email:
- What category it belongs to
- How urgent it actually is (independent of the customer's tone)
- Whether it needs to look something up before replying
- What to actually say

That reasoning-and-tool-use loop — deciding whether to call a tool, using the result, then composing a final answer — is what distinguishes this from a simple "email in, canned response out" automation.

## Architecture

```
Gmail Trigger (watches for labeled support emails)
   ↓
AI Agent (Anthropic Claude)
   ↓
   ├── decides category + priority
   ├── optionally calls → FAQ Lookup Tool (keyword search over product FAQ)
   └── drafts reply grounded in real product facts
   ↓
Structured JSON output: { category, priority, draft_reply }
```

**Categories:** Billing · Technical Issue · Feature Request · Account/Access · General Inquiry
**Priority levels:** Low · Medium · High — based on genuine business impact, not the customer's emotional tone

## Key Design Decisions

- **FAQ-grounded replies, not pure generation.** The agent has access to a keyword-searchable FAQ tool built from a real (fictional) product knowledge base. This reduces hallucination risk in customer-facing replies — a real production concern for any support-automation tool. See `data/faq_data.json`.
- **Priority is judged on substance, not tone.** The system prompt explicitly instructs the model not to conflate an angry customer with an urgent issue, and not to take a customer's self-reported urgency ("no rush") at face value when the actual content is serious. See `prompts/system_prompt.md`.
- **Strict output schema.** The agent is constrained to return exactly three fields as valid JSON — no markdown fences, no improvised extra fields — so the output can plug directly into a real ticketing system.

## Testing

The agent was stress-tested against 5 edge cases specifically designed to expose common failure modes in LLM-based classification systems: multi-issue emails, tone/severity mismatches (both directions), vague/low-information messages, and off-topic/spam content.

Two real issues were found and fixed through this process:
1. Vague messages were initially over-escalated to High priority — fixed by adding explicit priority guidelines and a Medium default for ambiguous cases.
2. The model occasionally added unrequested extra fields to its JSON output — fixed by explicitly enforcing the exact schema.

Full test inputs, outputs, and the before/after fix comparison are in [`tests/test_results.md`](./tests/test_results.md).

## Known Limitations

- **Gmail label-based filtering was unreliable in n8n** (a documented n8n bug); the workflow uses subject-line search filtering instead as a workaround.
- **No real ticketing system integration** — output is structured JSON, but isn't yet wired into an actual helpdesk tool (e.g., Zendesk, Freshdesk). A natural next step.
- **Keyword-based FAQ search, not semantic search.** A phrasing that doesn't share keywords with the FAQ data won't be found even if it's conceptually related. A vector-embedding-based search (RAG) would resolve this — planned for a future project.
- **Single-tool agent.** Only has access to the FAQ lookup; a production version might also check account status, subscription tier, or past ticket history before replying.

## Stack

n8n · Anthropic API (Claude) · Gmail API · JavaScript (Code Tool)

## Files

| Path | Contents |
|---|---|
| [`workflow/`](./workflow) | Exported n8n workflow JSON |
| [`prompts/`](./prompts) | Final system prompt used by the agent |
| [`data/`](./data) | FAQ knowledge base (structured JSON) |
| [`tests/`](./tests) | Edge-case test emails and full results log |
