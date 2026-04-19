# HCPulse AI — Video Demo Script (10–15 minutes)

> Estimated timing per section. Adjust pacing naturally — don't rush.

---

## SECTION 1: Introduction (1 min)

**[Screen: Browser on the deployed frontend URL]**

"Hey everyone, I'm Uzair, and in this video I'll walk you through HCPulse AI — an AI-first CRM I built for pharmaceutical field representatives to log, manage, and analyze their Healthcare Professional interactions using natural language.

The problem it solves is simple: field reps spend way too much time filling out forms after every doctor visit. With HCPulse AI, they just talk to an AI assistant in plain English — say 'I met with Dr. Johnson today, we discussed CardioMax, the sentiment was positive' — and the system handles everything: it extracts the data, fills the form, logs it to the database, and even suggests what to do next.

Let me show you how it works."

---

## SECTION 2: Architecture Overview (2 min)

**[Screen: Show the README architecture diagram, or draw on a whiteboard/slide]**

"Before we dive into the demo, let me quickly explain the architecture.

The frontend is built with **Next.js 15** using the App Router, React 19, TypeScript, and shadcn/ui for components. State management is handled by **Redux Toolkit** with three slices — auth, chat, and interaction. Authentication goes through **NextAuth** with Google OAuth and a demo credentials option.

The backend is a **FastAPI** server running Python 3.11, with **SQLAlchemy 2.0 async** for the ORM and **asyncpg** connecting to a **PostgreSQL** database hosted on Neon.

The brain of the system is a **LangGraph** agent. When a user sends a message, it flows through a StateGraph:

1. First, a **detect_intent** node classifies what the user wants — it tries keyword matching first, falls back to LLM classification.
2. Then a **conditional router** sends it to one of six nodes: execute_log, execute_edit, execute_context, execute_suggest, execute_summarize, or general_response.
3. Each execution node calls its specialized tool, formats the response, and returns.

The LLM provider is **Groq**, running **Gemma 2 9B** as the primary model with **LLaMA 3.3 70B** as a fallback. This dual-model strategy gives us both speed and reliability — if the smaller model fails to parse correctly, the larger one catches it.

For deployment, the frontend is on **Vercel** and the backend is on **Render**.

Now let me show you the actual application."

---

## SECTION 3: Login & Dashboard (1.5 min)

**[Screen: Login page]**

"Here's the login page. We support Google OAuth, but for the demo I'll use the pre-seeded credentials — `rep@hcpulse.ai` and `demo1234`. This maps to a seeded user 'John Smith' who's a pharmaceutical field representative."

**[Action: Log in with demo credentials]**

"Once logged in, we land on the dashboard. You can see four stat cards up top — Total HCPs, Total Interactions, Interactions This Week, and an AI Assisted badge showing the system is active.

Below that are quick-action cards to 'Log New Interaction' or go to the 'HCP Directory', and at the bottom we have recent interactions with sentiment badges — green for positive, red for negative, gray for neutral.

On the left, we have a **collapsible sidebar** — I can click this chevron to collapse it down to just icons, and click again to expand. It has four navigation items: Dashboard, HCP Directory, Interactions history, and Log Interaction.

The theme toggle here lets me switch between light and dark mode — watch."

**[Action: Toggle dark mode, then collapse/expand sidebar]**

---

## SECTION 4: HCP Directory (1 min)

**[Action: Navigate to HCP Directory]**

"The HCP Directory shows all healthcare professionals in the system. We have 10 seeded HCPs — cardiologists, oncologists, neurologists, and more. Each entry shows their name, specialty, organization, and contact details.

I can click into any HCP to see their detail page — here's Dr. Sarah Johnson, a cardiologist at Metro Heart Center. It shows her full profile, contact info, and a history of all interactions logged with her.

The system stores first name, last name, specialty, organization, email, phone, city, state, and NPI number. These are all real-world fields that pharma CRMs actually need."

---

## SECTION 5: Core Feature — AI Chat + Form (The 5 Tools) (6–7 min)

**[Action: Navigate to Log Interaction]**

"This is the heart of the application — the split-screen interaction logging page. On the left is a structured form with 15 fields. On the right is the AI chat assistant. The magic is that these two sides are fully synced via Redux — anything the AI extracts gets auto-filled into the form in real time."

---

### Tool 1: Log Interaction (~1.5 min)

**[Action: Click the first suggestion chip or type the message]**

"Let me show the first tool — **Log Interaction**. I'll type: 'I met with Dr. Johnson today to discuss CardioMax. Sentiment was positive and I shared brochures.'

Watch what happens..."

**[Wait for response]**

"The AI has identified this as a 'log interaction' intent. It extracted: HCP is Dr. Johnson, interaction type is in-person, today's date, products discussed is CardioMax, sentiment is positive. And if you look at the form on the left — it's been auto-filled with all of this data. The HCP dropdown, the type selector, the date, the sentiment toggle — all updated automatically.

Under the hood, the LangGraph agent detected the intent using keyword matching — phrases like 'I met with' trigger the log intent directly without needing an LLM call. Then the log tool used the Groq LLM to extract 13 structured fields from my natural language input. It also resolved 'Dr. Johnson' to the correct HCP in the database using a cascading name lookup — exact first+last name, then last name only, then first name only.

If the HCP didn't exist, the system would auto-create them."

---

### Tool 2: Edit Interaction (~1 min)

**[Action: Type edit command]**

"Now let's test the **Edit** tool. I'll say: 'Change the sentiment to negative and type to phone.'

The AI recognizes this as an edit intent — keywords like 'change the' and 'set the' trigger it immediately. It parses my instruction, extracts the fields I want to change — sentiment to negative, interaction_type to phone — and updates the form.

Look at the left side — the sentiment toggle switched from positive to negative, and the type dropdown changed to phone. This uses a fallback model chain — it tries Gemma 2 first for JSON parsing, and if that fails, it falls back to LLaMA 3.3 70B for better accuracy."

---

### Tool 3: Get Context (~1 min)

**[Action: Type context query]**

"Tool number three — **Get Context**. I'll ask: 'Tell me about Dr. Johnson.'

This resolves the HCP name, fetches their profile from the database, pulls their last 20 interactions, calculates an overall sentiment, aggregates all products discussed, and returns a comprehensive context report.

You can see the HCP's specialty, organization, interaction history with dates and summaries, and the overall relationship sentiment. This is exactly what a field rep needs before walking into a meeting — a quick briefing on their history with this doctor."

---

### Tool 4: Suggest Next Action (~1 min)

**[Action: Type suggestion query]**

"Next, the **Suggest** tool. I'll ask: 'What should I do next with Dr. Johnson?'

The system resolves the HCP, fetches their recent interactions, and sends the history to the LLM with a detailed prompt asking for 3 to 5 actionable suggestions. Each suggestion comes with a priority level — high, medium, or low — and reasoning for why that action makes sense.

For example, it might suggest scheduling a follow-up meeting to share Phase III data, or sending a thank-you email. These suggestions are context-aware — they're based on what actually happened in past interactions, not generic advice."

---

### Tool 5: Summarize (~1.5 min)

**[Action: Type summarize command]**

"Last tool — **Summarize**. I'll say: 'Summarize my recent interactions.'

This is interesting because it fetches real data from the database — it pulls my last 10 interactions, combines all the notes and summaries into a single text block, and sends it to the LLM for analysis.

The response shows: a high-level summary, key points from each interaction, action items that are still pending, all products mentioned across interactions, and an overall sentiment.

Notice it shows the interaction count — '9 interactions' — because it's working with real data, not making things up. If I had said 'Summarize my interactions with Dr. Johnson,' it would filter to just that HCP's interactions.

Every tool response is rendered with **React Markdown** for clean formatting — bullet points, bold text, headers — all properly styled."

---

## SECTION 6: Form Features & Submission (1.5 min)

**[Screen: Focus on the form side]**

"Let me highlight some form features. The HCP selector uses a **command palette** with search — I can type to filter by name or specialty.

The Topics field has **voice input** — I can click this microphone button, speak, and it uses the Web Speech API to transcribe and add topics. It shows a live transcript with a pulsing indicator.

At the bottom, there's an **AI Suggestions** card — I can click 'Generate' and the AI will analyze the current form context and suggest follow-up actions. Each suggestion has an 'Add' button to append it to the follow-up actions list.

When I hit Submit, the system validates the payload, sends it to the backend, and on success, shows a **success banner** with options to 'Ask AI Assistant' for more tools, view quick references like History, Summarize, or Suggestions, or 'Log Another Interaction' to reset and start fresh.

If there's a validation error — like a missing required field — it shows the actual error message from FastAPI's validation, not a generic 'something went wrong.'"

---

## SECTION 7: Code Architecture Walkthrough (2 min)

**[Screen: VS Code with project open]**

"Let me quickly walk through the code structure.

**Backend** — FastAPI follows a clean layered architecture: routers handle HTTP, services contain business logic, repositories handle database queries. The models folder has SQLAlchemy database models and Pydantic schemas.

The agent folder is where the LangGraph magic lives. `graph.py` defines the StateGraph with 7 nodes and conditional routing. The `tools/` folder has five specialized tool files — each is self-contained with its own LLM prompts, database queries, and response formatting.

Two key helpers in `graph.py`:

- `_keyword_intent()` — a fast keyword classifier that avoids LLM calls for obvious intents like 'I met with' or 'change the sentiment'
- `_resolve_hcp()` — smart HCP name resolution that strips honorifics like 'Dr.', tries exact match, then last name, then first name, with case-insensitive lookups

**Frontend** — Next.js 15 App Router with three Redux slices. The critical file is `chat-mode.tsx` which handles the AI conversation, syncs extracted data to the form via `syncFromChat`, and auto-creates HCPs if needed. `form-mode.tsx` at roughly 700 lines handles all 15 form fields, voice input, AI suggestions, and the submit flow.

The API client in `api.ts` has a custom `fetchAPI` wrapper that properly handles header merging and parses FastAPI's 422 validation errors into readable messages.

Everything is TypeScript end-to-end, with proper type definitions for HCP, Interaction, ChatMessage, and InteractionDraft."

---

## SECTION 8: What I Learned & Closing (1 min)

**[Screen: Back to the deployed app]**

"To wrap up — building this project taught me several things:

First, **LangGraph's StateGraph** is a powerful way to build AI agents with clear, debuggable flows. The intent detection → conditional routing → tool execution pattern is clean and extensible — adding a sixth tool would just be adding one more node and one more edge.

Second, **real-world AI applications need robust error handling**. LLMs don't always return perfect JSON, so having fallback models, regex-based JSON extraction, and cascading name resolution was critical to making this reliable.

Third, **the UX of AI features matters as much as the AI itself**. The form auto-fill, the markdown rendering, the auto-scroll, the success banners — these are what make the difference between a demo and something that actually feels production-ready.

The full source code is on GitHub, the frontend is deployed on Vercel, and the backend is on Render. Thanks for watching!"

---

## Quick Reference — Demo Flow Cheat Sheet

Use these exact messages for a smooth demo:

| #   | Type This                                                                                           | Tool Triggered | What to Show                 |
| --- | --------------------------------------------------------------------------------------------------- | -------------- | ---------------------------- |
| 1   | "I met with Dr. Johnson today to discuss CardioMax. Sentiment was positive and I shared brochures." | log            | Form auto-fills on left      |
| 2   | "Change the sentiment to negative and type to phone"                                                | edit           | Form updates in real-time    |
| 3   | "Tell me about Dr. Johnson"                                                                         | context        | HCP profile + history        |
| 4   | "What should I do next with Dr. Johnson?"                                                           | suggest        | Priority-ranked suggestions  |
| 5   | "Summarize my recent interactions"                                                                  | summarize      | Summary with real data count |

**Timing target:** Sections 1–4 in ~5.5 min, Section 5 in ~6 min, Sections 6–8 in ~4.5 min = ~16 min total. Trim by shortening pauses or skipping Section 6 if running long.
