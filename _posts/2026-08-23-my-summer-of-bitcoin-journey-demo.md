---
layout: post
title: "Bridging Bitcoin's Knowledge Gap: Building GenesisKB — My Summer of Bitcoin 2026 Journey"
date: 2026-08-23
author: Parth Dudhe
categories: [AI, Bitcoin, Infrastructure]
image: ../assets/images/blog_content/2026-08-23-my-summer-of-bitcoin-journey-demo.png
---

## What I Set Out to Do

Bitcoin evolves faster than anyone can keep up with. Every week brings new BIPs, conference talks, podcast episodes, newsletter deep-dives, and developer discussions scattered across dozens of platforms. The raw knowledge is out there — buried in hours of unindexed video, fragmented across blogs and mailing lists, and locked behind formats that demand your full attention.

Platforms like [Bitcoin Transcripts](https://btctranscripts.com/) have done heroic work archiving conference talks, but the pipeline is still heavily **manual**. Volunteers transcribe, review, and correct each piece of content by hand. The result? A growing backlog, inconsistent coverage, and a bottleneck that slows the speed at which critical Bitcoin knowledge becomes accessible to learners and developers worldwide.

Other educational resources face their own limitations:

- **Bitcoin Optech** delivers excellent weekly newsletters, but the content is dense and developer-targeted — not designed for someone just entering the ecosystem.
- **Podcasts and YouTube channels** produce hours of valuable content, but without transcription and summarization, the knowledge is trapped in long-form audio and video that most people simply don't have time to consume.
- **Community forums** (Delving Bitcoin, bitcoin-dev mailing list, Stack Exchange) contain deep technical insights, but navigating them requires significant prior knowledge.

The common thread? **There is no single platform that ingests, processes, and serves Bitcoin knowledge in multiple accessible formats — from text summaries to audiobooks to interactive notes — with the reliability and speed that the ecosystem demands.**

That is the problem **GenesisKB** was built to solve.

---

## What We're Building

[GenesisKB](https://github.com/genesis-kb/genesis-kb.github.io) is a **one-stop knowledge platform for the Bitcoin ecosystem**, developed under [Braidpool](https://github.com/braidpool). Our mission is to bridge the knowledge gap by making Bitcoin's rapidly evolving body of work — conferences, technical blogs, newsletters, developer discussions — **accessible, digestible, and actionable** for everyone.

At its core, GenesisKB is a **hybrid human + AI system**. We believe neither fully manual nor fully automated approaches are sufficient on their own. Manual transcription is too slow; purely automated pipelines produce unreliable output. GenesisKB combines both: **automated AI agent pipelines handle the heavy lifting — transcription, summarization, audiobook generation — while human reviewers ensure accuracy and trustworthiness.**

Here's what the platform delivers:

### 🎙️ Multi-Source Transcription & Summarization
We ingest content from conferences, podcasts, YouTube channels, blogs, and newsletters. Every piece of content is automatically transcribed (speech-to-text) and summarized, so you can get the key takeaways in minutes instead of hours.

### 📚 Automated Audiobook Series
Topic-specific, continuously updated audiobook playlists curated entirely by AI agents — turning fragmented transcripts into coherent, listenable learning experiences. Each transcript also gets its own standalone audio version.

### 📝 Interactive Notes Section
A full-featured notes workspace with:
- **Flow cards and canvas view** for visual learning
- **Context-aware AI assistance** that understands your notes in relation to the source material
- All the standard note-taking features you'd expect from a modern tool

### 💻 Bitcoin Coding Challenges
Hands-on coding exercises sourced directly from *Programming Bitcoin* by Jimmy Song — the most trusted practical resource for learning Bitcoin development from the ground up.

### 🔐 Reliable, Verified Sources
Every piece of content on GenesisKB is traceable back to its original source. We are committed to building a platform that Bitcoin learners can **trust**.

---

## The Major Technical Challenge: Automated Audiobook Curation Pipeline

Of everything I built this summer, the **fully automated audiobook curation pipeline** was the most complex and rewarding piece of engineering. The challenge: take a scattered corpus of scraped, transcribed, and summarized content and automatically produce **coherent, topic-specific audiobook playlists** — with no human in the loop.

This is harder than it sounds. The source data is messy — transcripts vary in length, quality, and topic coverage. A single conference talk might span multiple subjects; a blog post might be a 20,000-word deep-dive or a 300-word overview. The pipeline needs to handle all of it gracefully.

### The 9-Step Pipeline

Here's the actual architecture, from raw `.txt` file on disk to a fully published audiobook with CDN URL:

```
.txt file on disk
      │
      ▼
 [1] INGESTION ─────── parse files, detect frontmatter, group series
      │
      ├── singles              ├── series groups
      ▼                        ▼
 [2] CURATOR ROUTING ── route to single-article or series processing
      │
      ▼  (per episode)
 [3] TEXT CLEANING ──── strip timestamps, speaker labels, stage directions
      │
      ▼
 [4] LLM REWRITE ───── GPT-4.1 → JSON manifest: title + chapters[]
      │                 (skipped for series or short articles)
      ▼
 [5] NORMALIZE + CHUNK ─ expand "$1.5M" → "one point five million dollars"
      │                   split into TTS-safe character-limited pieces
      ▼
 [6] TTS SYNTHESIS ──── per chunk: SHA-256 cache check → Deepgram / Smallest AI
      │                  diarization: distinct voice per speaker
      ▼
 [7] STITCH ─────────── concat chunks (220ms gaps) → concat chapters (700ms gaps)
      │                  loudness normalize to −20 dBFS → export .mp3
      ▼
 [8] SUPABASE UPLOAD ── POST to CDN → returns public URL (optional)
      │
      ▼
 [9] DB PERSISTENCE ─── update episode status, refresh playlist stats
```

Let me walk through each step.

#### Step 1 — Ingestion (`ingestion.py`)

`scan_input_directory()` globs all `*.txt` files and calls `parse_file()` on each. The parser detects YAML frontmatter via the `^---\n...\n---\n` regex pattern:

- **No frontmatter** → treated as a standalone `single_article`; filename is title-cased as the title.
- **Frontmatter found** → `yaml.safe_load()` parses the metadata block; required fields are validated.
- **Empty body** → logged and skipped.

Each file becomes an `InputFile` dataclass:

```python
@dataclass
class InputFile:
    filepath: Path
    input_type: str          # 'single_article' or 'series'
    title: str
    body: str
    series_slug: Optional[str]
    series_title: Optional[str]
    sequence_number: int
    author: Optional[str]
    source_url: Optional[str]
    description: Optional[str]
    tags: list[str]
    word_count: int          # computed: len(body.split())
```

`group_series()` then groups series files by `series_slug`, sorted by `sequence_number`.

#### Step 2 — Curator Routing (`curator.py`)

`AudiobookCurator` is the orchestrator. After ingestion, it routes work based on input type:

- **Series groups** → `_process_series()`: Creates one playlist per `series_slug`, iterates files in sequence order, and generates one episode per file. The LLM rewrite step is **always skipped** because the series structure already provides logical division.
- **Single articles** → `_process_single_article()`: Creates a `collection` playlist with a single episode. The LLM step is skipped only if word count ≤ `chapterize_threshold` (default: 5,000 words).

The idempotency guard is critical here:

- `status == "completed"` → skip entirely (no wasted API calls)
- `status == "pending"` → set `"generating"` → run pipeline → set `"completed"`
- `status == "failed"` → re-attempt automatically

#### Step 3 — Text Cleaning (`textproc.py`)

Strips content that sounds bad or meaningless when read aloud — timestamps (`[00:12:45]`), speaker labels (`ALICE:`), stage directions (`[laughter]`), and excess whitespace. One exception: when `diarize=True`, speaker labels are **retained** — they're used in Step 6 to assign different TTS voices to each speaker.

#### Step 4 — LLM Rewrite & Chapterization (`rewrite.py`)

For long articles (≥ 5,000 words), GPT-4.1 restructures the text into titled chapters. The model acts as an audiobook editor — removing filler, smoothing sentences, and splitting into logical sections. Output is a structured JSON manifest:

```json
{
  "title": "Episode Title",
  "chapters": [
    {"title": "Chapter 1: Introduction", "text": "Narration text..."},
    {"title": "Chapter 2: Deep Dive",    "text": "More narration..."}
  ]
}
```

Long articles exceeding `max_words_per_request` (3,000 words) are split into chunks and sent in multiple API calls; results are merged. If the LLM returns bad JSON or zero chapters, a single-chapter fallback is built directly from the cleaned text.

#### Step 5 — Text Normalization & Chunking (`textproc.py`)

This step converts written forms to speech-natural forms before TTS synthesis:

| Input | Output | Method |
|---|---|---|
| `$1.5M` | "one point five million dollars" | Regex + scale map |
| `2009` | "two thousand nine" | `num2words` |
| `21st` | "twenty-first" | `num2words` |
| `UTXO` | "U-T-X-O" | `audiobook_lexicon.json` |
| `BTC` | "bitcoin" | `audiobook_lexicon.json` |
| `AWS` | "A-W-S" | Auto-hyphenation |
| `NASA` | "NASA" (unchanged) | Pronounceable exceptions |

The **`audiobook_lexicon.json`** holds Bitcoin-specific substitutions like `"PoW" → "proof of work"`, `"DeFi" → "dee-fye"`, and `"ASIC" → "ay-sick"`.

After normalization, `chunk_split()` breaks text into TTS-safe pieces using a multi-tier strategy: sentence boundaries (NLTK) → prosody boundaries (`;` `:` `,`) → character-level as last resort. Deepgram allows 1,900 chars/request; Smallest AI only 240.

#### Step 6 — TTS Synthesis (`tts.py`)

An abstract `TTSProvider` base class with two implementations — **Deepgram** and **Smallest AI** — selectable via config.

The **disk cache** is the key performance optimization: every synthesis call is keyed by `SHA-256(provider | voice | format | text | lex_hash | speed | model)`. Cache hit → zero API calls, near-instant. Cache miss → calls API, writes to `.cache/audiobooks/`. Updating `audiobook_lexicon.json` automatically invalidates stale cached audio because the lexicon content is hashed into the key.

For **diarized content**, each unique speaker maps to a distinct voice from the provider pool (e.g., Deepgram voices: `aura-asteria-en`, `aura-orion-en`, `aura-arcas-en`...). The mapping is consistent across chapters — same speaker, same voice throughout.

#### Step 7 — Audio Stitching (`stitch.py`)

Uses `pydub` (backed by `ffmpeg`) to assemble the final audiobook:

1. **`build_chapter()`**: Concatenates chunks with **220 ms** silence gaps
2. **`export_book()`**: Concatenates chapters with **700 ms** silence gaps (simulating chapter breaks), then runs **loudness normalization** to −20 dBFS across the entire file

Output: `outputs/audiobooks/<episode-id>/<Title>.mp3`

#### Step 8 — Cloud Upload (`storage.py`)

Optional — only runs if Supabase credentials are set. Uploads the final `.mp3` to Supabase Storage with `x-upsert: true` (safe to re-upload), and returns the public CDN URL.

#### Step 9 — DB Persistence (`playlist_service.py`)

All DB operations go through `PlaylistService` using SQLAlchemy. The two core tables:

**`audio_playlists`**: `id`, `title`, `slug` (unique), `playlist_type` (series/collection), `tags`, `status`, `episode_count`, `total_duration_seconds`

**`audio_episodes`**: `id`, `playlist_id` (FK), `title`, `sequence_number`, `audio_url`, `duration_seconds`, `status` (pending/generating/completed/failed), `chapters` (JSONB), `metadata` (JSONB)

After audio generation, the pipeline calls `update_episode()` and `refresh_playlist_stats()` to keep everything in sync.

### Engineering Challenges & Solutions

Building a reliable, fully automated pipeline that chains together file parsing, LLMs, TTS APIs, audio processing, cloud storage, and a database was the hardest part of this internship. Here are the key challenges I faced and how I solved them:

**1. TTS character limits break naive splitting.** Deepgram caps requests at 1,900 chars; Smallest AI at 240. A transcript can be 50,000+ characters. Splitting mid-sentence produces garbled audio. → `chunk_split()` uses a multi-tier strategy: sentence boundaries → prosody boundaries → character-level as last resort.

**2. Bitcoin acronyms sound wrong in TTS.** "UTXO" gets read as a nonsense word; "$1.5M" becomes "dollar one point five M." → A two-layer pronunciation system: domain-specific `audiobook_lexicon.json` substitutions, plus automatic hyphenation for unrecognized uppercase acronyms.

**3. Re-runs waste expensive API credits.** TTS and LLM calls cost money. A crash-and-retry shouldn't re-synthesize everything. → Two caching layers: episode-level DB idempotency (completed episodes are skipped) + chunk-level SHA-256 disk cache (cache hits cost zero API calls).

**4. Short articles don't need chapters; long articles do.** Sending a 200-word article to GPT-4.1 for chapterization is wasteful. A 10,000-word monologue without chapters is monotonous. → Adaptive LLM gating via `chapterize_threshold` (default: 5,000 words). Series episodes always skip the LLM — their structure is already logical.

**5. Multi-speaker content gets a single robotic voice.** Podcast transcripts and interviews have multiple speakers, but a single TTS voice makes conversations impossible to follow. → Optional `--diarize` mode: `parse_diarization()` splits text into speaker-attributed segments; each speaker gets a distinct voice from the provider pool, consistent across chapters.

**6. Audio segments have inconsistent volume levels.** Different TTS calls return audio at varying loudness, creating jarring jumps in the final file. → `export_book()` runs loudness normalization to −20 dBFS across the full stitched file, preserving relative dynamics while standardizing overall volume.

**7. Numbers, dates, and symbols sound unnatural.** Text like "21st", "2009", and "42,000" gets garbled by TTS engines. → The `normalize()` function uses `num2words` + regex to expand all numeric forms to their spoken equivalents before synthesis.

**8. Concurrent pipeline runs cause database race conditions.** Running the pipeline twice simultaneously could create duplicate playlist records, crashing with `IntegrityError`. → `find_or_create_playlist()` and `find_or_create_episode()` use an optimistic concurrency pattern — try insert, catch `IntegrityError`, rollback and fetch the existing record:

```python
try:
    session.add(new_playlist)
    session.commit()
except IntegrityError:
    session.rollback()
    return session.query(AudioPlaylist).filter_by(slug=slug).first()
```

---

## What I Built This Summer: The Full Scope

My Summer of Bitcoin internship at GenesisKB was broad and deeply technical. Here's the complete list of what I shipped:

### 1. Database Schema Redesign (V1 → V2)

The original V1 schema was designed for a single content source. As we expanded to ingest content from YouTube, podcasts, blogs, newsletters, and conference recordings, the schema couldn't keep up. I redesigned the entire database schema from scratch to support our **multi-source data processing pipeline**:

- Normalized tables for content sources, transcripts, summaries, and audio assets
- Polymorphic associations to handle different content types uniformly
- Optimized indexing for the queries our agent pipelines run most frequently
- Migration scripts to safely move all existing data from V1 to V2 without downtime

### 2. Interactive Notes Section

Built a full-featured notes workspace that goes beyond basic text editing:

- **Canvas view with flow cards**: Visualize connections between concepts using a node-based canvas, similar to tools like Obsidian Canvas or Miro
- **Context-aware AI assistance**: The AI understands which transcript or summary you're taking notes on and can answer questions, expand on concepts, or suggest related content — all grounded in the actual source material
- **Standard note-taking features**: Rich text editing, tagging, search, and organization

### 3. Fully Automated Audiobook Curation Pipeline

As described in the technical deep-dive above — the crown jewel of my internship. A LangGraph-inspired, multi-node AI agent pipeline that takes fragmented transcripts and produces coherent, topic-specific audiobook playlists. End-to-end automated, with built-in reliability mechanisms.

### 4. Automated YouTube Commenting

When GenesisKB transcribes a YouTube video, we automatically post a comment on the original video linking back to the transcript and summary on our platform. This serves two purposes: it **drives discovery** of GenesisKB for people watching the original content, and it **gives back to the content creator** by adding value to their video.

### 5. JWT Authentication

Implemented a complete authentication system using JSON Web Tokens:

- Secure user registration and login flow
- Token-based session management with refresh token rotation
- Role-based access control for admin vs. standard users
- Protected API routes for content management operations

### 6. Unit and Regression Tests

Wrote comprehensive test suites to ensure reliability across the platform:

- Unit tests for individual pipeline nodes and utility functions
- Regression tests for the data migration from schema V1 to V2
- Integration tests for the audiobook curation pipeline's end-to-end flow

### 7. Product Brainstorming & UX Design

Beyond code, I spent significant time brainstorming features and improvements with the team:

- How can we make Bitcoin learning more **engaging** — not just informative, but genuinely interesting?
- What formats do different learners prefer? (Some want to read, some want to listen, some want to build)
- How do we balance **automation speed** with **content accuracy**?
- What downstream applications can we build on top of our knowledge base?

---

## What Shipped vs. What Remains

### ✅ Shipped

| Feature | Status |
|---|---|
| Database Schema V2 + Data Migration | ✅ Deployed |
| Multi-source Ingestion Pipeline | ✅ Deployed |
| Interactive Notes Section | ✅ Deployed |
| Automated Audiobook Curation Pipeline | ✅ Deployed |
| Per-Transcript Audio Generation | ✅ Deployed |
| Automated YouTube Commenting | ✅ Deployed |
| JWT Authentication System | ✅ Deployed |
| Coding Challenges (from *Programming Bitcoin*) | ✅ Deployed |
| Unit & Regression Test Suites | ✅ Deployed |

### 🔮 What Remains (Future Roadmap)

- **Discord Chatbot**: Integrate the GenesisKB knowledge base into Discord servers so Bitcoin communities can query transcripts, summaries, and learning paths directly in chat.
- **X/Twitter Bot**: Surface relevant Bitcoin knowledge in response to trending discussions on X — meeting learners where they already are.
- **Additional Downstream Applications**: The knowledge base we've built is a foundation. We envision browser extensions, Telegram bots, podcast companions, and more — all powered by the same unified data layer.
- **Expanded Content Sources**: More conferences, more podcasts, more newsletters. The pipeline is built to scale; we just need to point it at more sources.

---

## What I Learned

This internship was a masterclass in building production systems. Here's what I took away:

### Technical Skills

- **Database Modeling & Optimization**: Designing schemas that serve both human users and automated pipelines; writing migrations that don't break production data
- **AI Agent Pipeline Design**: Architecting multi-step, stateful workflows with LangGraph-inspired patterns; handling non-determinism, retries, and graceful degradation
- **LLMs in Production**: Moving beyond "prompt and pray" — building systems with consensus mechanisms, confidence thresholds, and human-in-the-loop fallbacks
- **TTS/STT Systems**: Understanding the quirks of text-to-speech and speech-to-text engines, especially with domain-specific technical vocabulary
- **Authentication & API Design**: Implementing JWT auth flows, designing RESTful APIs, and thinking about security from the ground up
- **Data Migration**: Safely evolving a live database schema while preserving data integrity
- **Testing Strategies**: Writing tests that actually catch bugs — not just tests that check the happy path

### Working with the Braidpool Team

Beyond the technical growth, working with the team at Braidpool (GenesisKB) was an incredible experience. The brainstorming sessions, code reviews, and architectural discussions pushed me to think bigger and build better. The team's commitment to **open-source values** and **making Bitcoin knowledge genuinely accessible** inspired me every day.

---

## What Happens Next

GenesisKB isn't a summer project that ends when the internship does. **We're committed to deploying this platform for real Bitcoin learners.**

The infrastructure is built. The pipelines are running. The content is being processed. The next step is getting GenesisKB into the hands of the people it was built for — students learning about Bitcoin for the first time, developers diving into protocol-level details, and community members who want to stay current without spending hours every week consuming raw content.

I'm continuing to contribute to GenesisKB beyond Summer of Bitcoin, and I'm excited to see it grow into the reliable, comprehensive knowledge platform that the Bitcoin ecosystem deserves.

---

follow me on [X (Twitter)](https://x.com/ParthDudhe07) and [GitHub](https://github.com/parthdude07) and [LinkedIn](https://www.linkedin.com/in/parthdudhe07/)

*#SummerOfBitcoin #Bitcoin #OpenSource #GenesisKB #Braidpool*
