# Sample report: TypeSafe Jev

`/assess-tech https://typesafe.ai/blog/introducing-system-one-models-and-jev`

This sample runs against two made-up projects, 🛒 Storefront and 📬 Helpdesk. The facts about Jev come from the linked post. The code references are invented for the example.

---

**Conditional for 📬 Helpdesk ticket routing. Skip everywhere else.** Jev fits the job Helpdesk already does with an LLM: read a ticket, pick a queue and a priority. The speed and price claims are all vendor numbers, and the API is in early access, so it earns Implement only after it matches the current router on real tickets.

## 1. Tech tl;dr

### [Jev](https://typesafe.ai/blog/introducing-system-one-models-and-jev)

- **Inputs → black box → outputs:** `ticket text + list of allowed queues → [Jev] → one queue + probability for each option`
- **What it is:** A model that makes a choice from options you define up front, instead of writing text. Think of it as a smart `if` statement that returns a typed value and a confidence score.
- **Use cases:** Classifying and routing. Scoring and ranking. Pulling fields out of messy input. Judging or guarding an LLM's output.
- **Status:** Early access with a waitlist. Hosted API only, run from the US West Coast. Input costs $0.042 per million tokens and output is free (vendor pricing). The post states no license.

## 2. How it works

```text
                  ┌──────────────────── Jev API ────────────────────┐
 ticket text ───▶ │ schema check ──typed query──▶ parallel scoring  │
 allowed queues ▶ │                               (all options in   │ ──▶ queue
                  │                                one pass)        │     + probabilities
                  │                               ──scores──▶ pick  │
                  └─────────────────────────────────────────────────┘
```

You send your app's state and the set of allowed answers. Jev scores every option in one pass rather than writing tokens one at a time, so the answer can only be one of the options you gave it. The post says type errors are impossible and every answer comes with calibrated probabilities, trained with a method it calls RLCD. It handles up to 255 options per choice. Above that it scores options separately and then picks, which adds a second step. It can't write free-form text at all.

The vendor reports 70 to 500 ms end to end, and 40x to 200x faster than frontier LLMs on queries shaped like this. Those are their numbers, measured from their own laptop.

## 3. Compared to what we do now

📬 Helpdesk today: `ticket text → [LLM call + JSON parse + retry on bad output] → queue + priority` (`src/routing/classifyTicket.ts:18`)

| Dimension | Ours today | Jev |
|---|---|---|
| Latency | 2 to 4 s per ticket, logged (`src/routing/classifyTicket.ts:41`) | 70 to 500 ms (vendor claim) |
| Bad output | About 1 in 300 calls fails JSON parse and retries (`src/routing/classifyTicket.ts:52`) | Can't return a value outside the schema |
| Confidence | None. Every route looks equally sure | Probability per option, so low-confidence tickets can go to a human |
| Cost | About $40 a month at current volume | A few cents a month at vendor prices |
| Maturity | Generally available provider with an SLA | Early access, one region, no SLA mentioned |

🛒 Storefront has a related job, ranking about 4,000 products for search (`lib/search/rank.ts:9`). That's far past the 255-option limit, and Postgres full-text search already handles it well.

Net: Helpdesk would gain faster routing, no parse retries, and a confidence score to send unclear tickets to a person. It would give up a mature provider for an early-access one.

## 4. Where we can use it

Nothing is worth implementing yet. One use depends on a test against real tickets.

🔵 top Implement recommendation · 🟢 other Implement recommendations · 🟠 Conditional · 🟡 Skip

| Priority | Tool or idea | Put it where | Pros | Cons | Recommendation |
|---:|---|---|---|---|---|
| 1 | [Jev](https://typesafe.ai/blog/introducing-system-one-models-and-jev): typed choice with probabilities | 📬 Helpdesk, ticket routing | Under a second instead of 2 to 4 s. No parse retries. Confidence to send unclear tickets to a human | Early access, no SLA, vendor-only numbers. A second AI provider to manage | 🟠 **Conditional.** Implement if Jev matches or beats the current router on 500 past tickets with known queues, for under $5 of test spend. Then replace `classifyTicket.ts` outright, send tickets under 0.6 confidence to the triage queue, and delete the retry path. Otherwise Skip. |
| 2 | Jev: ranking | 🛒 Storefront, product search | — | 4,000 products is past the 255-option limit. Postgres search already works | 🟡 **Skip.** Wrong shape for the job, and the current search has no reported problems. |
| 3 | Jev: text output | 🛒 Storefront, product descriptions | — | Jev can't write free-form text | 🟡 **Skip.** It doesn't do this job. |

Run the 500-ticket test once early access comes through. If it doesn't match the current router, close this out.
