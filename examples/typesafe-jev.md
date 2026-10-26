# Sample report: TypeSafe Jev

`/assess-tech https://typesafe.ai/blog/introducing-system-one-models-and-jev`

This sample runs against two made-up projects, 🛒 Storefront and 📬 Helpdesk. The facts about Jev come from the linked post. The code references and usage numbers are invented for the example.

---

**Adopt: Conditional for 📬 Helpdesk ticket routing, Skip everywhere else.** Jev fits the job Helpdesk already does with an LLM, but every speed and price number is the vendor's own and the API is in early access.

**Borrow: Implement a fixed answer set on the current router.** Jev's core idea is that the model can only answer with an option you defined. Helpdesk can get most of that today by giving its current LLM call a strict schema with the queue names as an enum. That removes the parse retries without a new provider.

## 1. Tech tl;dr

### [Jev](https://typesafe.ai/blog/introducing-system-one-models-and-jev)

- **Inputs → black box → outputs:** `ticket text + list of allowed queues → [Jev] → one queue + probability for each option`
- **What it is:** A model that makes a choice from options you define up front, instead of writing text. Think of it as a smart `if` statement that returns a typed value and a confidence score.
- **Use cases:** Classifying and routing. Scoring and ranking. Pulling fields out of messy input. Judging or guarding an LLM's output.
- **Status:** Early access with a waitlist. Hosted API only, run from the US West Coast. Input costs $0.042 per million tokens and output is free (vendor pricing). The post states no license.
- **In the wild:** No independent production use found yet. The only public uses are the vendor's own demos: a Doom bot making 10 decisions a second and a Wikipedia racing bot ([post](https://typesafe.ai/blog/introducing-system-one-models-and-jev), vendor evidence).

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

Net: Helpdesk would gain faster routing, no parse retries, and a confidence score to send unclear tickets to a person. It would give up a mature provider for an early-access one.

## 4. Ideas worth borrowing

**A fixed answer set, enforced by the API.** Jev only answers with an option you declared, so the caller never parses or validates free text ([post](https://typesafe.ai/blog/introducing-system-one-models-and-jev)). Helpdesk's current provider supports strict structured output, so the queue list can become an enum in the request schema. It would change `src/routing/classifyTicket.ts:18` and let `src/routing/classifyTicket.ts:52` go. Need: 412 parse retries and 37 tickets routed to a queue that doesn't exist in the last 90 days of logs.

| Idea | Already covered by |
|---|---|
| Two-stage scoring for large option sets | 🛒 Storefront ranks 4,000 products with Postgres full-text search (`lib/search/rank.ts:9`) |

## 5. Where we can use it

Borrow the fixed answer set now. Adopting Jev depends on a test against real tickets.

🔵 top Implement recommendation · 🟢 other Implement recommendations · 🟠 Conditional · 🟡 Skip

| Priority | Tool or idea | Put it where | Pros | Cons | Recommendation |
|---:|---|---|---|---|---|
| 1 | [Fixed answer set](https://typesafe.ai/blog/introducing-system-one-models-and-jev), borrowed from Jev | 📬 Helpdesk, ticket routing | No parse retries or invalid queues. No new provider | About a day of work. Queue list must stay in sync with the schema | 🔵 **Implement.** Build the enum from the queue table, pass it as a strict schema, delete the retry path. Accept when 1,000 replayed tickets give zero invalid queues. |
| 2 | [Jev](https://typesafe.ai/blog/introducing-system-one-models-and-jev): typed choice with probabilities | 📬 Helpdesk, ticket routing | Under a second instead of 2 to 4 s. Confidence to send unclear tickets to a human | Early access, no SLA, vendor-only numbers. A second AI provider to manage | 🟠 **Conditional.** Implement if Jev matches or beats the current router on 500 past tickets with known queues, for under $5 of test spend. Then replace `classifyTicket.ts` and send tickets under 0.6 confidence to triage. Otherwise Skip. |
| 3 | Jev: ranking | 🛒 Storefront, product search | — | 4,000 products is past the 255-option limit. Postgres search already works | 🟡 **Skip.** Wrong shape for the job, and the current search has no reported problems. |
| 4 | Jev: text output | 🛒 Storefront, product descriptions | — | Jev can't write free-form text | 🟡 **Skip.** It doesn't do this job. |

Ship the fixed answer set this week. Run the 500-ticket Jev test once early access comes through, and close it out if it doesn't beat the improved router.
