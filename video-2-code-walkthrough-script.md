# Video 2: Code Walkthrough Script

**Target length: 3–5 minutes**

---

## Opening (0:00 – 0:15)

> "I'll walk through the codebase in four sections: project structure, the LLM prompt, the three hardest technical problems, and what I'd improve."

Open VS Code with the `trinethra/` folder in the Explorer sidebar.

---

## 1. Project structure (0:15 – 0:55)

> "Everything is a single Next.js App Router project — no separate backend process."

Point to the Explorer:

```
src/
  app/
    (auth)/        ← login, register, forgot/reset password
    api/           ← all route handlers (Node.js runtime)
    dashboard/     ← the analyzer UI
  lib/
    prompt.ts      ← the LLM prompt
    groq.ts        ← Groq streaming client (active)
    ollama.ts      ← Ollama streaming client + Zod schema (available)
    prisma.ts      ← database singleton
  auth.config.ts   ← edge-safe NextAuth config (middleware)
  auth.ts          ← full NextAuth config (PrismaAdapter + bcrypt)
prisma/
  schema.prisma    ← User, Analysis, PasswordResetToken
  seed.ts          ← idempotent demo-user seed
```

> "The route handlers are in `src/app/api/`. The most important one is `api/analyze` — it's the SSE streaming endpoint. Auth is split into two files because NextAuth v5 middleware runs on the Vercel Edge runtime, which can't import bcrypt or Prisma."

---

## 2. The prompt — `src/lib/prompt.ts` (0:55 – 1:50)

Open `src/lib/prompt.ts`.

> "This is the heart of the assignment. The first decision: **one prompt or many?**"

Point to the comment block at the top:

> "I chose one prompt. A 10-minute transcript is around 1,500–2,000 tokens — comfortably inside any small model's context window. A single round-trip returns all six outputs and preserves cross-cutting context: the gap detector sees the same evidence the scorer saw. A chain of four calls would be slower AND lower quality."

Scroll to the `system` string:

> "The system prompt does four things. First, it sets the analyst persona and the hard rules — only use evidence from the transcript, never invent facts. Second, it defines the bias catalogue — helpfulness bias, presence bias, halo, horn, recency. Third, it embeds the full rubric from `rubric.json` at runtime — every label, band, and signal for scores 1 through 10. And fourth, it specifies the critical 6-vs-7 boundary verbatim from the assignment brief — 'a 6 executes tasks defined by others; a 7 identifies problems the supervisor had NOT articulated.'"

Scroll to the JSON schema block:

> "The output schema is embedded directly in the prompt. The model is told to return exactly this shape and nothing else — no prose, no code fences, no commentary."

Scroll to the `MAX_TRANSCRIPT_CHARS` constant:

> "I cap the transcript at 6,000 characters before sending. This keeps the total request — system prompt plus transcript plus expected response — safely within Groq's free-tier rate limit."

---

## 3. Structured output reliability — `src/lib/ollama.ts` (1:50 – 2:35)

Open `src/lib/ollama.ts`, scroll to `extractJson` and `analysisSchema`.

> "This is Challenge 2 — the model doesn't always return clean JSON. Three layers of defence:"

**Layer 1 — Prompt constraint:**

> "The system prompt says 'no prose, no markdown, no code fences.' That eliminates 80% of noise."

**Layer 2 — Tolerant extractor:**
Point to `extractJson`:

> "`extractJson` strips any code fence markers, then finds the first `{` and last `}` in the string and JSON-parses only that slice. If the model adds a preamble sentence before the object, this handles it."

**Layer 3 — Zod schema with `.catch()`:**
Point to the schema definition:

> "Every enum field uses `.catch(fallback)`. So if the model returns `'medium-high'` for confidence instead of `'medium'`, Zod degrades gracefully to the fallback instead of rejecting the whole response. The intern gets a partial result rather than an error."

---

## 4. The analyzer UI — `src/app/dashboard/analyzer.tsx` (2:35 – 3:40)

Open `analyzer.tsx`. Scroll to the `ReviewState` type definitions near the top.

> "This is where the assignment brief's core requirement lives: 'the intern reviews — accepting, rejecting, or editing each finding.' I model this with a `ReviewState` object that mirrors the structure of the analysis — one state per score, per evidence item, per KPI mapping, per gap, per follow-up question, per bias flag."

Scroll to `initReviewState`:

> "When analysis arrives, review state is initialised to `pending` for every finding. The `if (analysis !== reviewedAnalysis)` block resets it — derived-state pattern, no `useEffect` needed."

**Evidence linking (Challenge 3):**
Scroll to `TranscriptCard` → the `segments` useMemo:

> "Hovering an evidence quote sets `highlightQuote` in state. `TranscriptCard` uses `indexOf` to find the verbatim string in the raw transcript, splits it into three segments — before, match, after — and wraps the match in a `<mark>`. No diff library, no fuzzy match — the prompt instructs the model to use verbatim quotes, so exact string matching is reliable."

**Finalize workflow:**
Scroll to `buildFinalizedAnalysis` and `FinalizedModal`:

> "When the intern clicks 'Finalize', `buildFinalizedAnalysis` merges the review state with the original analysis — applying edits and filtering out rejected findings. The modal shows the clean result with a 'Copy as text' button. The intern-reviewed record is what goes into the case management system, not the raw AI output."

**Anti-automation bias (Challenge 4):**

> "Three UI decisions prevent the intern from blindly trusting the AI: a persistent amber 'Draft, not verdict' banner on every result, an explicit confidence pill on the score, and the opt-in save checkbox — analyses are not silently persisted."

---

## 5. What I'd improve (3:40 – 4:20)

> "Five specific things:"

1. **Inline editing in the finalized view** — right now the intern edits findings in the review stage; ideally they could also make final tweaks inside the modal before copying.

2. **Email transport for password resets** — the reset link is currently returned in the API response in dev. Production would wire up Resend or AWS SES.

3. **Per-user model preference** — there's a per-request model override field in the form, but it should be a saved setting in the user profile so the intern doesn't re-type it every session.

4. **Audit log** — the `Analysis` Prisma model stores the parsed result. I'd also store the raw model output alongside it so the intern (or a reviewer) can see exactly what the Zod schema discarded.

5. **Confidence calibration** — track intern-accepted vs AI-suggested scores over time. If the model consistently over-scores by 1 point for a particular supervisor style, surface that offset so the intern knows to adjust.

---

## Closing (4:20 – 4:30)

> "That covers structure, the prompt design, the three main technical challenges — structured output, evidence linking, and the finalize workflow — and five concrete improvements. Thanks."

---

## Quick reference — files to show on screen

| Timestamp   | File                                                                |
| ----------- | ------------------------------------------------------------------- |
| 0:15 – 0:55 | Explorer sidebar (project tree)                                     |
| 0:55 – 1:50 | `src/lib/prompt.ts`                                                 |
| 1:50 – 2:35 | `src/lib/ollama.ts` (`extractJson`, `analysisSchema`)               |
| 2:35 – 3:00 | `src/app/dashboard/analyzer.tsx` (`ReviewState`, `initReviewState`) |
| 3:00 – 3:20 | `analyzer.tsx` (`TranscriptCard` → `segments` useMemo)              |
| 3:20 – 3:40 | `analyzer.tsx` (`buildFinalizedAnalysis`, `FinalizedModal`)         |
| 3:40 – 4:20 | Speak to camera / stay on `analyzer.tsx`                            |
