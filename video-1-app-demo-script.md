# Video 1: App Demo Script

**Target length: 2–3 minutes**

---

## Opening (0:00 – 0:20)

> "Hi, I'm Uzair. This is Trinethra — the Supervisor Feedback Analyzer I built for the DeepThought Software Developer Internship assignment. I'll walk you through the full workflow in under three minutes."

Point your screen at the browser tab showing `https://trinethra-olive.vercel.app` (or `localhost:3000` if running locally).

---

## Sign in (0:20 – 0:35)

> "The app has full authentication. I've seeded a demo account so you don't need to register."

- Click **"Use demo"** on the login page — it auto-fills `demo@trinethra.app` / `demo1234`
- Click **Sign in**
- You land on the split-screen dashboard

> "Left panel is the input form. Right panel is where the analysis streams in."

---

## Load a sample transcript (0:35 – 0:55)

> "The assignment included three sample transcripts. I've bundled all three into the app."

- Click **Load sample** → pick **Vikram Nair at Veerabhadra Auto** (the mixed/borderline one — most interesting)
- The transcript, fellow name, company, and supervisor fields fill automatically

> "In production, an intern would paste the actual call transcript here. I've pre-loaded the metadata too."

---

## Run the analysis (0:55 – 1:25)

> "Groq runs the same Llama 3 family the assignment specifies for Ollama, but on dedicated LPU hardware — about 200 times faster. I'll explain the model choice more in the code walkthrough."

- Click **Run analysis**
- Point out the **elapsed timer** ticking in the thinking box

> "There's a live token stream so the intern can see the model actively working — it removes the 'is it frozen?' anxiety during a 5 to 10 second generation."

- Analysis result appears in the right panel

---

## Walk through the output (1:25 – 2:15)

**Score card:**

> "Score of [X] out of 10, band [Need Attention / Productivity / Performance], with a confidence level of [low/medium/high]. The justification paragraph cites direct quotes from the transcript and explicitly addresses the 6-vs-7 boundary — the critical rubric threshold the assignment brief calls out."

Show the amber **"Draft, not verdict"** banner.

> "This is the anti-automation bias design — the score is a suggestion, not a verdict. The intern must actively decide what to do with it."

**Evidence:**

> "Each piece of extracted evidence shows the verbatim quote, a positive/negative/neutral tag, which assessment dimension it maps to, and the interpretation."

Hover one quote.

> "Hovering a quote highlights the exact phrase in the transcript panel below — that's the evidence-linking feature from Challenge 3."

**KPI Mapping → Gap Analysis → Follow-up Questions:**

> "KPI mapping shows which of the 8 business KPIs the Fellow's work connects to, and whether that connection is system-level — meaning it survives the Fellow leaving — or personal. Then gaps: what the supervisor never mentioned, and 3 to 5 concrete follow-up questions targeting each gap."

**Bias Flags:**

> "Finally, bias flags. If the supervisor's language triggers a helpfulness, recency, or halo bias pattern, the model surfaces it so the intern can weigh it before finalizing."

---

## Accept / Reject / Edit + Finalize (2:15 – 2:50)

> "Now the core of the assignment brief — quote — 'The tool does NOT replace the intern's judgment. The AI suggests; the human decides.'"

- Click **✓** to accept the score card
- Click **✎** on one evidence item, change the signal from positive to neutral, click **Save**
- Click **✕** to reject one follow-up question the intern doesn't agree with
- Watch the **Review progress bar** tick upward

> "Once every finding is reviewed, the 'Finalize' button activates."

- Click **Finalize →**

> "The modal strips all rejected findings, applies the edits, and produces a clean intern-reviewed report — not the raw AI output. There's a Copy as text button to paste it directly into whatever case management system they use."

---

## Closing (2:50 – 3:00)

> "That's the full workflow — from raw transcript to reviewed, finalized assessment. Thanks for watching."

---

## Fallback notes (if demoing locally with Ollama instead of Groq)

If running locally with Ollama:

1. Show terminal: `ollama serve` running in one tab
2. Confirm `ollama list` shows `llama3`
3. In the app `.env`, `GROQ_API_KEY` is unset and `OLLAMA_BASE_URL=http://localhost:11434`
4. Generation will be slower (30–90 s on CPU) — mention this is expected and why Groq was chosen for the live demo
