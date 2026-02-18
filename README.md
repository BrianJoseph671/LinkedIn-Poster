# LinkedIn-Poster

> Automated content generation and scheduling for consistent LinkedIn presence

**Built with:** n8n, Claude API, Airtable, RSS feeds

---

## What It Does

LinkedIn-Poster is an automated content pipeline that helps me maintain a consistent LinkedIn presence without the daily grind of content creation.

The workflow generates, reviews, and schedules LinkedIn posts based on:
- RSS feeds from TechCrunch AI, Hacker News, The Verge, Product Hunt
- Source articles and my commentary
- AI-generated drafts that match my voice
- Quality assurance checks before publishing

---

## Why I Built This

**The problem:** Posting consistently on LinkedIn is time-consuming, and most "AI content generators" produce generic garbage that screams "I used ChatGPT."

**The solution:** A state machine workflow that combines real content sources (RSS, articles I'm reading) with AI-assisted drafting to create posts that sound like me and provide real value. Every post goes through voice analysis, key point extraction, and QA before I even see it.

---

## n8n Workflows

### WF0: Intake

<img width="524" height="179" alt="image" src="https://github.com/user-attachments/assets/8a6beb7f-393b-423d-8995-12e09487840b" />

**Purpose:** Captures new content ideas when you create a row in Airtable

**Flow:**
1. Airtable trigger fires when new row created in Posts table
2. Fetches content from source URL (if provided)
3. Extracts readable text from the page
4. Sets status to "Captured"
5. Triggers WF2 (Draft-Polish) automatically

**Your workflow:**
- Go to Airtable Posts table
- Create new row: paste `source_url`, type `my_take`, select `intake_type` + `content_pillar`
- This workflow fires automatically and enriches the record

---

### WF1: Intake-Enrich

<img width="1402" height="255" alt="image" src="https://github.com/user-attachments/assets/4e41b289-0b44-4c60-87e1-dec1cbbad432" />

**Purpose:** Enhanced intake flow with additional enrichment and validation

**Flow:**
1. Airtable trigger on new Posts record
2. Fetches and validates source URL content
3. Extracts readable text and metadata
4. Enriches with additional context
5. Sets status to "Captured" (triggers Draft-Polish)

**Setup requirements:**
- Airtable base ID configured
- Posts table with `created_at` field (type: Created Time)
- Source URL field for content fetching

---

### WF2: Draft-Polish (State Machine)

<img width="1495" height="192" alt="image" src="https://github.com/user-attachments/assets/a0f42c3f-efda-4c99-81b5-fbc025591723" />

**Purpose:** Core state machine that generates, reviews, and manages LinkedIn drafts

**State machine fields:** `workflow_status` + `review_status`

**Trigger:** Records where `workflow_status=Queued` AND `review_status=Not reviewed`

**Pipeline:**
1. **Guard** - Validates record is ready for processing
2. **Lock** - Sets status to "Processing" to prevent duplicate runs
3. **Voice Pack** - Loads my writing voice profile
4. **Build Voice** - Analyzes voice patterns and style
5. **Extract Key Points (KP)** - Pulls main ideas from source content
6. **Parse KP** - Structures key points for drafting
7. **Research Scaffold** - Builds content outline
8. **Parse Scaffold** - Validates and structures outline
9. **Draft in Voice** - Generates LinkedIn post using Claude API
10. **QA** - Quality checks for authenticity, clarity, engagement
11. **Save** - Persists draft to Airtable
12. **Email** - Sends draft for review
13. **Approve/Reject** - Waits for decision
14. **Update Status** - Finalizes workflow state

**State transitions:**
- `Queued` → `Processing` (lock)
- `Processing` → `Awaiting Approval` (draft saved)
- `Awaiting Approval` → `Approved` or `Queued` (after email decision)
- Any step → `Failed` (on error)

**Self-retrigger prevention:** Trigger formula only matches `Queued` + `Not reviewed`. WF2's own writes (Processing, Awaiting Approval, etc.) never match the trigger.

**Rejection handling:** Sets `workflow_status=Queued` + `review_status=Rejected`. Will NOT auto-retrigger. User must manually set `review_status=Not reviewed` to re-draft.

**Error recovery:** Sets `workflow_status=Failed` + `last_error` + `last_step`. To retry: manually set `workflow_status=Queued` + `review_status=Not reviewed`.

**Artifacts persisted:** `draft_text`, `qa_pass`, `qa_issues`, `voice_context`, `key_points`, `scaffold`, `run_id`, `last_step`

---

### WF3: RSS-Triage

<img width="1416" height="309" alt="image" src="https://github.com/user-attachments/assets/72ae5dc9-32a4-4b72-b8d5-a220f88e2fd8" />

**Purpose:** Automatically monitors RSS feeds, filters for relevance, creates intake records

**Schedule:** Polls every 2 hours

**Default RSS feeds** (configured in Airtable RSS_Sources table):
- **TechCrunch AI:** https://techcrunch.com/category/artificial-intelligence/feed/
- **Hacker News (best):** https://hnrss.org/best
- **The Verge Tech:** https://www.theverge.com/rss/tech/index.xml
- **Product Hunt:** https://www.producthunt.com/feed

**Flow:**
1. Fetch new items from RSS feeds
2. Filter for relevance (AI/sales tech keywords)
3. Check if already processed (deduplication)
4. Create intake records in Airtable Posts table
5. WF0/WF1 automatically enriches
6. WF2 generates drafts

---

## Tech Stack

- **Automation:** n8n workflows for content orchestration
- **AI:** Claude API for voice analysis and content generation
- **Storage:** Airtable for content calendar, drafts, and state management
- **Content Sources:** RSS feeds, manual URL submissions
- **Email:** Draft review and approval notifications

---

## Key Features

- **Voice consistency** - Analyzes my previous posts to maintain authentic tone
- **Quality assurance** - Every draft goes through automated QA checks
- **State machine reliability** - Handles errors, prevents duplicate processing, supports retries
- **Manual oversight** - Every post gets reviewed before publishing (no autopilot spam)
- **Content discovery** - RSS monitoring surfaces relevant content automatically

---

## What I Learned

- **State machines solve race conditions** - Lock/unlock patterns prevent duplicate processing
- **Voice can be modeled** - Analyzing past posts creates consistent tone in generated content
- **QA catches AI mistakes** - Automated checks for generic phrases, unclear messaging, missing CTAs
- **Build guardrails, not autopilot** - AI should assist creation, not replace judgment
- **Persistence is key** - Saving artifacts at each step enables debugging and iteration

---

## Status

Active and running. Currently testing different content formats and engagement strategies.

---

**Questions?** Reach out on [LinkedIn](https://www.linkedin.com/in/brianmathewjoseph/) or email me at josephbrian671@gmail.com
