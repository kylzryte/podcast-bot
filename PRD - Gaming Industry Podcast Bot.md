---
type: prd
status: draft
version: 1.0
date: 2026-03-01
owner: Kyle Billings
tags:
extends: News Bot v2.0
deployment_target: Mac Mini (headless, Eastern Time)
---
I 
# Product Requirements Document: Gaming Industry Podcast Bot

## 1. Overview

### 1.1 Product Name
Gaming Industry Podcast Bot (extends News Bot v2.0)

### 1.2 Purpose
Automated weekly podcast generation system that transforms the daily Gaming Industry Digest into a 90-minute conversational podcast featuring three distinct analyst personalities debating and discussing the week's most important gaming industry developments.

**Multi-Stage AI Pipeline:**

**Daily Research Phase (Mon-Sat, triggered at 12:30pm ET):**
- News digest publishes at 12:00pm ET
- Podcast bot analyzes digest articles
- Three AI hosts independently research articles from their unique perspectives via Perplexity
- Research saved for weekly compilation

**Weekly Synthesis Phase (Sunday 5:00am ET):**
- OpenAI analyzes 7 days of articles and 21 research files (3 hosts × 7 days)
- Generates article selection prompts from each host's perspective
- Hosts "vote" on which 6-15 articles to discuss
- Claude synthesizes all research + host memories into natural 90-minute conversation script
- Script includes debate, banter, callbacks, predictions, and personality-driven analysis

**Audio Production Phase (Sunday 5:30am ET):**
- Script parsed line-by-line
- Each line sent to OpenAI TTS API with character-specific voices
- Audio files stitched together with natural pauses and timing
- MP3 generated and saved locally
- RSS feed updated for distribution

**Distribution:**
- Self-hosted on Mac Mini via simple HTTP server
- Accessible via Tailscale on Android device
- No public distribution (personal use)

### 1.3 Problem Statement
The daily Gaming Industry Digest provides excellent written synthesis but:
- Requires dedicated reading time (15-20 minutes)
- No audio format for commutes, workouts, or multitasking
- Lacks conversational exploration of nuance and implications
- Misses opportunity for debate and multiple perspectives on developments
- No entertainment value or personality-driven analysis

### 1.4 Solution
**Three-Character Podcast System:**

**The Hosts:**

1. **Nigel Thornbury** (The Veteran)
   - Late 60s, British accent (posh but rough around edges)
   - Been in industry since early 1980s (Atari/Commodore era)
   - Personality: Bombastic, loud, interrupts frequently, curses liberally, contrarian
   - Perspective: "I've seen this all before" - historical context and pattern recognition
   - Quirks: References gaming history, skeptical of trends, dismissive of hype, secretly respects good game design
   - Running bits: Mispronounces modern terms, "Back when I was at...", cuts through marketing BS

2. **Marcus Webb** (The Know-It-All)
   - Early 30s, American accent (confident, slightly arrogant)
   - Background: Former tech journalist turned analyst
   - Personality: Brash, jumps to conclusions, overconfident, know-it-all energy
   - Perspective: Technology and business model disruption, always predicting "the next big thing"
   - Quirks: Makes bold predictions, doubles down when challenged, obsessed with data and metrics
   - Running bits: "I'm calling it now...", cites obscure player metrics, overuses buzzwords

3. **Emily Bradford** (The Optimist)
   - Late 20s, British accent (younger, bright)
   - Background: Former community manager turned analyst
   - Personality: Enthusiastic, optimistic, rose-colored glasses, thoughtful
   - Perspective: Community sentiment, player experience, developer intent
   - Quirks: Defends studios, cites Reddit/Discord sentiment, excited about everything
   - Running bits: "But the community loves...", "I think they're really trying here...", defers to Nigel's experience

**Group Dynamic:**
- Nigel plays devil's advocate, Emily plays defender, Marcus plays provocateur
- They debate vigorously but generally reach similar conclusions
- Natural interruptions, cross-talk, and banter
- 100% game-focused conversation (no personal anecdotes unrelated to gaming)
- Respect each other's expertise despite disagreement style

**Episode Structure:**
- 90 minutes (target, can vary based on content)
- Weekly release, covers Saturday-Saturday
- 6-15 articles discussed (selected by host voting)
- Mix of HIGH/MEDIUM/LOW impact articles
- Natural conversation flow (not rigid segments)
- Callbacks to previous episodes, prediction tracking, running jokes

---

## 2. Goals & Objectives

### 2.1 Primary Goals
1. **Audio-First Intelligence** - Provide podcast format for multitasking consumption
2. **Multi-Perspective Analysis** - Three distinct viewpoints on each development
3. **Entertainment Value** - Engaging personalities make industry news entertaining
4. **Memory & Continuity** - Hosts remember past discussions, track predictions, develop over time
5. **Automation** - Zero manual effort after initial setup

### 2.2 Success Criteria
- Podcast generates successfully 95%+ of weeks without manual intervention
- 90-minute target achieved within ±20 minutes (70-110 min acceptable)
- Conversations feel natural (not robotic or scripted)
- Host personalities remain consistent episode-to-episode
- Prediction tracking works (hosts reference and score past predictions)
- Audio quality acceptable for casual listening
- System cost: ≤$100/month additional (on top of $41/month news bot cost)

### 2.3 Non-Goals (Out of Scope for v1)
- Public distribution (no Spotify, Apple Podcasts, etc.)
- Professional audio production (music, editing, mastering)
- Listener interaction (no Q&A, no community submissions)
- Video/visual component
- Real-time/breaking news episodes
- Multi-language support
- Monetization

---

## 3. Functional Requirements

### 3.1 Daily Research Phase

#### FR-1: Digest Monitoring
- **Must Have:** Monitor for new daily digest at 12:00pm ET
- **Must Have:** Trigger research phase at 12:30pm ET (30-min delay for digest completion)
- **Must Have:** Parse digest markdown to extract articles with metadata
- **Must Have:** Validate article structure (headline, summary, impact level, sources)

#### FR-2: Article Analysis
- **Must Have:** Extract article content from digest:
  - Headline
  - Category
  - Impact level (HIGH/MEDIUM/LOW)
  - Executive summary
  - Key takeaways
  - Strategic implications
  - Sources/URLs
- **Must Have:** Tag article with discussion potential score (meaty vs surface-level)
- **Should Have:** Identify articles that connect to previous episodes for callback potential

#### FR-3: Per-Host Research (Perplexity API)
- **Must Have:** Generate character-specific research prompt for each article from each host's perspective:
  - **Nigel:** Historical precedents, pattern recognition, "what went wrong last time this happened"
  - **Marcus:** Business model implications, tech disruption potential, market predictions
  - **Emily:** Community sentiment, player impact, developer intentions
- **Must Have:** Execute 3 Perplexity queries per article (one per host)
- **Must Have:** Save research results to structured files: `Podcast Research/YYYY-MM-DD/[Article-ID]-[Host-Name]-Research.md`
- **Must Have:** Log token usage per host per day
- **Should Have:** Cache research for debugging and future reference

### 3.2 Weekly Synthesis Phase

#### FR-4: Article Selection (OpenAI)
- **Must Have:** Sunday 5:00am ET execution
- **Must Have:** Compile all articles from past 7 days (Sat-Sat)
- **Must Have:** Load all 21 research files (7 days × 3 hosts)
- **Must Have:** Generate OpenAI prompt simulating "host voting" process:
  - Each host perspective evaluates article discussion potential
  - Nigel prioritizes historical significance and pattern recognition
  - Marcus prioritizes disruption potential and bold prediction opportunities
  - Emily prioritizes community impact and developer stories
- **Must Have:** Select 6-15 articles based on:
  - Host votes (weighted equally)
  - Impact level distribution (mix of HIGH/MEDIUM/LOW)
  - Category diversity (avoid 10 M&A articles)
  - Discussion potential (meaty enough for 8-12 min conversation)
  - Thematic connections (articles that relate to each other)
- **Must Have:** Order articles for episode flow (not always HIGH→MEDIUM→LOW, varies each week)
- **Should Have:** Identify callback opportunities to previous episodes

#### FR-5: Host Memory Loading (Claude)
- **Must Have:** Load summarized memory file for each host:
  - `Host_Memories/Nigel-Memory.md`
  - `Host_Memories/Marcus-Memory.md`
  - `Host_Memories/Emily-Memory.md`
- **Must Have:** Memory structure per host:
  - **Canonical Personality Traits** (updated as character evolves)
  - **Recent Opinions** (last 6 months, major stances taken)
  - **Active Predictions** (predictions made, not yet resolved)
  - **Resolved Predictions** (outcome: correct/incorrect, date resolved)
  - **Running Jokes** (callbacks, catchphrases, recurring bits)
  - **Key Insights** (notable analysis moments, quotable takes)
- **Must Have:** 6-month rolling window (auto-archive older content)
- **Should Have:** Prediction score tracking (% correct per host)

#### FR-6: Script Generation (Claude)
- **Must Have:** Generate 90-minute conversational script covering selected articles
- **Must Have:** Inject all context:
  - 6-15 selected articles with full content
  - 21 research files (all perspectives on selected articles)
  - 3 host memory summaries
  - Episode metadata (episode number, date range covered, previous episode callbacks)
- **Must Have:** Script format:
  ```
  NIGEL: [Dialogue line with natural speech patterns]
  MARCUS: [Response with character voice]
  EMILY: [Reaction]
  NIGEL: [Interrupts] Wait, hold on—
  MARCUS: [Overlaps] That's exactly what I'm saying!
  ```
- **Must Have:** Include natural speech patterns:
  - Ums, pauses, "you know", "I mean", "right"
  - Laughter: `[LAUGHS]`, `[CHUCKLES]`
  - Reactions: `[SIGHS]`, `[GROANS]`
  - Interruptions: `[INTERRUPTS]`, `[OVERLAPS]`
  - Emphasis: `[ANIMATED]`, `[SKEPTICAL]`
- **Must Have:** Include character-specific behaviors:
  - Nigel: Curses liberally (F-bombs, "bloody hell", "bollocks"), loud interruptions, historical references
  - Marcus: Bold predictions, data citations, overconfident tone, buzzwords
  - Emily: Community citations, optimistic framing, defers to experience, defends developers
- **Must Have:** Episode elements:
  - Cold open (brief banter leading into first article)
  - Natural topic transitions (not rigid segments)
  - Callbacks to previous episodes (when relevant)
  - New predictions (Marcus primarily, others occasionally)
  - Running jokes (organically integrated)
  - Episode wrap-up (brief closing thoughts, no formulaic outro)
- **Must Have:** Target 90 minutes of spoken audio (roughly 13,500-16,000 words)
- **Should Have:** Vary conversation structure each episode (not formulaic)

#### FR-7: Script Post-Processing
- **Must Have:** Parse script to extract:
  - All dialogue lines with speaker labels
  - Stage directions ([LAUGHS], [INTERRUPTS], etc.)
  - New predictions made (for memory update)
  - Notable opinions expressed (for memory update)
  - Running joke moments (for memory update)
- **Must Have:** Save script: `Podcast Scripts/YYYY-MM-DD - Episode [N] - Script.md`
- **Should Have:** Validate script length (word count within acceptable range)

### 3.3 Audio Production Phase

#### FR-8: Text-to-Speech Generation (OpenAI TTS)
- **Must Have:** Voice configuration per host:
  - **Nigel:** `alloy` voice (or best British male equivalent), slightly slower pace
  - **Marcus:** `onyx` voice (or best American male equivalent), normal pace
  - **Emily:** `shimmer` voice (or best British female equivalent), slightly faster pace
- **Must Have:** Process script line-by-line:
  1. Parse dialogue line
  2. Send to OpenAI TTS API with character voice
  3. Receive audio file (MP3)
  4. Save temporarily: `temp/episode-[N]-line-[X].mp3`
- **Must Have:** Handle stage directions:
  - `[LAUGHS]`: Insert 0.5s of silence (natural laugh pause)
  - `[INTERRUPTS]`: Reduce pause between lines to 0.1s
  - `[OVERLAPS]`: No pause between lines
  - `[SIGHS/GROANS]`: Insert 0.3s of silence
  - Default between speakers: 0.5s pause
- **Must Have:** Retry logic for TTS failures (3 attempts per line)
- **Should Have:** Voice consistency check (same voice ID across episode)

#### FR-9: Audio Stitching
- **Must Have:** Use `pydub` or similar library to concatenate audio files
- **Must Have:** Insert pauses based on stage directions
- **Must Have:** Add natural breath pauses between speakers (0.5s default)
- **Must Have:** Export final audio:
  - Format: MP3
  - Bitrate: 128 kbps (acceptable quality, reasonable file size)
  - Filename: `YYYY-MM-DD - Gaming Industry Podcast - Episode [N].mp3`
  - Save to: `~/podcast/episodes/`
- **Should Have:** Normalize audio levels across all lines
- **Could Have:** Basic noise gate to remove silence

#### FR-10: RSS Feed Management
- **Must Have:** Generate/update RSS 2.0 feed XML: `~/podcast/feed.xml`
- **Must Have:** Feed metadata:
  - Title: "Gaming Industry Podcast"
  - Description: "Weekly discussion of gaming industry developments by Nigel Thornbury, Marcus Webb, and Emily Bradford. Covers business, content, platforms, and market trends."
  - Language: en-us
  - No artwork (for now)
- **Must Have:** Episode item structure:
  - Title: "Episode [N]: [Date Range]" (e.g., "Episode 12: Feb 24-Mar 2")
  - Description: Brief summary of articles discussed
  - Publication date: Sunday 6:00am ET
  - Audio URL: `http://[tailscale-hostname]:8000/episodes/[filename].mp3`
  - Duration: Actual audio length
  - File size: Actual MP3 size
- **Must Have:** Keep last 52 episodes in feed (1 year of history)
- **Should Have:** Validate RSS feed against RSS 2.0 spec

#### FR-11: Local Web Server
- **Must Have:** Simple Python HTTP server serving `~/podcast/` directory
- **Must Have:** Start on boot via launchd/cron
- **Must Have:** Port: 8000 (or configurable)
- **Must Have:** Accessible via Tailscale: `http://macmini.tailnet:8000/feed.xml`
- **Must Have:** Serve RSS feed and MP3 files
- **Should Have:** Basic logging of access requests
- **Could Have:** Directory listing for manual file access

### 3.4 Memory & Prediction Tracking

#### FR-12: Memory Update (Post-Episode)
- **Must Have:** After script generation, extract for each host:
  - **New Opinions:** Major stances taken on articles (e.g., "Nigel believes Microsoft's AI hire will backfire within 2 years")
  - **New Predictions:** Explicit predictions made (e.g., "Marcus predicts Switch 2 will outsell PS5 by Q4 2026")
  - **Running Joke Moments:** New catchphrases, callbacks, recurring bits
  - **Character Evolution:** Notable personality developments
- **Must Have:** Append to host memory files with episode number and date
- **Must Have:** Archive opinions/predictions older than 6 months to `Host_Memories/Archive/`
- **Should Have:** Summarize after every 4 episodes to keep memory files manageable

#### FR-13: Prediction Resolution (Automated)
- **Must Have:** Weekly scan of news digests for prediction-relevant keywords
- **Must Have:** Match news developments to active predictions using Claude analysis
- **Must Have:** When match found:
  - Update prediction status: `RESOLVED - CORRECT` or `RESOLVED - INCORRECT`
  - Add resolution date and evidence (article headline + date)
  - Move from "Active Predictions" to "Resolved Predictions"
  - Calculate updated accuracy score per host
- **Must Have:** Claude integration to determine if prediction was correct:
  - Load prediction text
  - Load relevant news article
  - Evaluate: Did this prediction come true? (Yes/No/Partial)
- **Should Have:** Flag borderline resolutions for manual review
- **Could Have:** Notify in next episode when major prediction resolves

### 3.5 Error Handling & Recovery

#### FR-14: Daily Research Failures
- **Must Have:** If Perplexity API fails for a host:
  - Retry 3x with exponential backoff (2s, 4s, 8s)
  - Log failure
  - Continue with other hosts (don't block entire process)
  - If all hosts fail for an article: skip that article's research, continue with others

#### FR-15: Weekly Script Generation Failures
- **Must Have:** If Claude script generation fails:
  - Retry 2x with modified prompt (lower token limit or fewer articles)
  - If retries fail: Log error, skip week, send alert (future: email/Slack)
- **Must Have:** If script is too short (<10,000 words):
  - Retry with prompt: "Expand discussions, add more debate and analysis"
  - If still short: Accept and proceed (better than no episode)
- **Must Have:** If script is too long (>20,000 words):
  - Don't retry (long episode acceptable)
  - Log warning for future prompt adjustment

#### FR-16: Audio Production Failures
- **Must Have:** If TTS fails mid-episode:
  - Retry failed line 3x
  - If line repeatedly fails: Insert 1s silence and continue
  - Log failed lines for manual review
- **Must Have:** If audio stitching fails:
  - Retry entire stitching process 1x
  - If fails again: Keep individual line files, log error
- **Must Have:** If episode doesn't complete by 7am ET Sunday:
  - Log timeout error
  - Keep partial outputs for debugging
  - Skip week (don't publish incomplete episode)

---

## 4. Technical Architecture

### 4.1 System Components

```
news_bot.py (existing)
    ↓ (daily 12:00pm ET)
Daily Digest Published
    ↓ (daily 12:30pm ET)
podcast_bot.py --mode=daily-research
    ↓
Daily Research Files Saved
    ↓ (accumulates Mon-Sat)
    ↓ (Sunday 5:00am ET)
podcast_bot.py --mode=weekly-generate
    ↓
[OpenAI] Article Selection
    ↓
[Claude] Script Generation
    ↓
[OpenAI TTS] Audio Production
    ↓
RSS Feed Updated
    ↓
HTTP Server (always running)
    ↓
Accessible via Tailscale on Android
```

### 4.2 Tech Stack

**Languages & Frameworks:**
- Python 3.9+
- `pydub` for audio processing
- `feedgen` for RSS generation
- `http.server` for local web serving

**APIs:**
- Perplexity API (Sonar Pro) - Host research
- OpenAI API - Article selection prompts + TTS
- Anthropic Claude API (Sonnet 4) - Script generation

**Infrastructure:**
- Mac Mini (headless, macOS, Eastern Time)
- Cron scheduling
- Tailscale for network access
- Local file storage (no cloud dependencies)

**Dependencies (new):**
```
openai>=1.0.0
pydub>=0.25.1
feedgen>=0.9.0
```

### 4.3 File Structure

```
~/podcast/
├── episodes/
│   ├── 2026-03-02 - Gaming Industry Podcast - Episode 001.mp3
│   ├── 2026-03-09 - Gaming Industry Podcast - Episode 002.mp3
│   └── ...
├── scripts/
│   ├── 2026-03-02 - Episode 001 - Script.md
│   ├── 2026-03-09 - Episode 002 - Script.md
│   └── ...
├── research/
│   ├── 2026-02-24/
│   │   ├── article-001-nigel-research.md
│   │   ├── article-001-marcus-research.md
│   │   ├── article-001-emily-research.md
│   │   └── ...
│   ├── 2026-02-25/
│   └── ...
├── memories/
│   ├── Nigel-Memory.md
│   ├── Marcus-Memory.md
│   ├── Emily-Memory.md
│   └── Archive/
│       ├── Nigel-2025-Q4.md
│       └── ...
├── feed.xml (RSS feed)
├── logs/
│   └── podcast_bot.log
└── temp/ (temporary TTS files, cleaned after stitching)
```

### 4.4 Configuration

**Config File: `podcast_config.yaml`**
```yaml
apis:
  perplexity:
    model: "sonar-pro"
    max_tokens: 2000
    temperature: 0.3
    api_key_env: "PERPLEXITY_API_KEY"

  openai:
    model: "gpt-4"
    tts_model: "tts-1"
    tts_voices:
      nigel: "alloy"   # British male (closest available)
      marcus: "onyx"   # American male
      emily: "shimmer" # British female (closest available)
    max_tokens: 4000
    temperature: 0.4
    api_key_env: "OPENAI_API_KEY"

  claude:
    model: "claude-sonnet-4"
    max_tokens: 16000  # Long script generation
    temperature: 0.7   # More creative for dialogue
    api_key_env: "ANTHROPIC_API_KEY"

hosts:
  - name: "Nigel"
    full_name: "Nigel Thornbury"
    age: "late 60s"
    accent: "British"
    personality: "Bombastic, loud, curses, contrarian"
    voice: "alloy"
  - name: "Marcus"
    full_name: "Marcus Webb"
    age: "early 30s"
    accent: "American"
    personality: "Brash, know-it-all, jumps to conclusions"
    voice: "onyx"
  - name: "Emily"
    full_name: "Emily Bradford"
    age: "late 20s"
    accent: "British"
    personality: "Optimistic, rose-colored glasses, community-focused"
    voice: "shimmer"

episode:
  target_length_minutes: 90
  target_word_count: 14000  # ~90 min at 155 wpm
  min_word_count: 10000     # ~65 min
  max_word_count: 20000     # ~130 min
  article_count_range: [6, 15]
  coverage_period_days: 7   # Sat-Sat

scheduling:
  daily_research_time: "12:30"  # 30 min after digest publishes
  weekly_generation_time: "05:00"  # Sunday 5am ET
  timezone: "America/New_York"

audio:
  format: "mp3"
  bitrate: "128k"
  sample_rate: 24000
  pause_between_speakers: 0.5  # seconds
  pause_interrupt: 0.1
  pause_laugh: 0.5

memory:
  retention_months: 6
  summarize_every_episodes: 4
  prediction_auto_resolve: true

rss:
  title: "Gaming Industry Podcast"
  description: "Weekly discussion of gaming industry developments"
  language: "en-us"
  max_episodes_in_feed: 52
  base_url: "http://macmini.tailnet:8000"

server:
  port: 8000
  directory: "/Users/kylebillings/podcast"

paths:
  base: "/Users/kylebillings/podcast"
  episodes: "episodes"
  scripts: "scripts"
  research: "research"
  memories: "memories"
  temp: "temp"
  logs: "logs"
  digests: "/Users/kylebillings/Desktop/Tanalorr/50_OUTPUT/News Bot/Daily News"
```

### 4.5 Execution Flow

**Daily Research (Mon-Sat, 12:30pm ET):**
```python
def daily_research():
    # Load today's digest
    digest = load_digest(date.today())
    articles = parse_digest(digest)

    # For each article, each host does research
    for article in articles:
        for host in ["Nigel", "Marcus", "Emily"]:
            prompt = generate_research_prompt(article, host)
            research = query_perplexity(prompt)
            save_research(date.today(), article.id, host, research)

    log_completion()
```

**Weekly Generation (Sunday 5:00am ET):**
```python
def weekly_generation():
    # Collect past week's articles and research
    articles = collect_articles_from_week()  # Sat-Sat
    research_files = collect_research_files()  # 21 files

    # Article selection via OpenAI
    selection_prompt = generate_selection_prompt(articles, research_files)
    selected_articles = query_openai(selection_prompt)  # Returns 6-15 article IDs

    # Load host memories
    memories = {
        "Nigel": load_memory("Nigel"),
        "Marcus": load_memory("Marcus"),
        "Emily": load_memory("Emily")
    }

    # Generate script via Claude
    script_prompt = generate_script_prompt(
        selected_articles,
        research_files,
        memories,
        episode_number=get_next_episode_number()
    )
    script = query_claude(script_prompt)
    save_script(script)

    # Update memories
    update_memories(script)

    # Generate audio
    audio_files = []
    for line in parse_script(script):
        audio = query_openai_tts(line.text, line.speaker)
        audio_files.append(audio)

    # Stitch audio
    final_audio = stitch_audio(audio_files, script.stage_directions)
    save_episode(final_audio)

    # Update RSS
    update_rss_feed(final_audio)

    # Cleanup
    cleanup_temp_files()

    log_completion()
```

---

## 5. Prompt Engineering

### 5.1 Daily Research Prompts (Perplexity)

**Template Structure:**
```
You are [HOST_NAME], a gaming industry analyst with [PERSONALITY_DESCRIPTION].

ARTICLE TO RESEARCH:
[Full article content]

Your perspective: [PERSPECTIVE_DESCRIPTION]

Research this article from your unique perspective:
1. What historical precedents or patterns do you see? (Nigel)
   OR
   What business model disruptions or tech innovations are at play? (Marcus)
   OR
   What is the community sentiment and player impact? (Emily)

2. What are the key talking points you would raise in a podcast discussion?

3. What predictions or bold takes would you make about this development?

4. What questions would you want to debate with your co-hosts?

Provide detailed research notes (500-800 words) covering your perspective.
```

### 5.2 Article Selection Prompt (OpenAI)

**Template:**
```
You are helping select articles for a 90-minute gaming industry podcast.

AVAILABLE ARTICLES (past 7 days):
[List all articles with: ID, headline, impact level, category, summary]

RESEARCH AVAILABLE:
- Each article has research from 3 perspectives: Nigel (veteran), Marcus (tech analyst), Emily (community)

SELECTION CRITERIA:
- Select 6-15 articles for 90-minute discussion
- Mix of HIGH/MEDIUM/LOW impact
- Variety of categories
- Meaty topics that support 8-12 minute debates
- Articles that connect thematically for richer discussion
- Balance between hard business news and content/community topics

HOST PREFERENCES:
- Nigel prefers: M&A, leadership changes, industry patterns, historical significance
- Marcus prefers: Tech disruption, business models, bold prediction opportunities, data-driven stories
- Emily prefers: Game releases, community reactions, developer stories, positive developments

Simulate each host "voting" on article importance (1-10 scale).
Select 6-15 articles optimizing for:
1. Balanced host interest (all hosts should have articles they care about)
2. Discussion potential (controversial, complex, or trend-setting)
3. Episode flow (order articles for natural conversation progression)

Return: Ordered list of article IDs with brief rationale for selection and suggested order.
```

### 5.3 Script Generation Prompt (Claude)

**Template (high-level, actual will be 3000+ words):**
```
You are writing a 90-minute podcast script for "Gaming Industry Podcast."

HOSTS:
1. Nigel Thornbury (late 60s, British, bombastic, loud, curses liberally, interrupts, contrarian)
2. Marcus Webb (early 30s, American, brash know-it-all, jumps to conclusions, overconfident)
3. Emily Bradford (late 20s, British, optimistic, community-focused, rose-colored glasses)

SELECTED ARTICLES FOR THIS EPISODE:
[Full content of 6-15 articles]

RESEARCH NOTES:
[All 21 research files showing each host's perspective on selected articles]

HOST MEMORIES:
[Nigel's memory file - canonical personality, recent opinions, predictions, running jokes]
[Marcus's memory file - same structure]
[Emily's memory file - same structure]

EPISODE CONTEXT:
- Episode #: [N]
- Date range: [Sat-Sat]
- Previous episode callbacks: [List relevant topics from last episode]

SCRIPT REQUIREMENTS:

1. NATURAL CONVERSATION:
- 90 minutes of dialogue (target 14,000 words at 155 wpm)
- Natural interruptions, overlaps, cross-talk
- Include: ums, pauses, "you know", "I mean", laughter, groans
- Stage directions: [LAUGHS], [INTERRUPTS], [SIGHS], [ANIMATED], etc.

2. CHARACTER CONSISTENCY:
- Nigel: Curses frequently (F-bombs, "bloody hell", "bollocks"), loud, interrupts, references gaming history from 1980s-2000s, cuts through BS
- Marcus: Makes bold predictions ("I'm calling it now..."), overuses buzzwords, cites data/metrics, overconfident tone, doubles down when challenged
- Emily: Cites Reddit/Discord/Twitter sentiment, defends developers, optimistic framing, defers to experience, "But the community loves..."

3. CONVERSATION STRUCTURE:
- Cold open: Brief banter leading into first article (game-related only)
- 6-15 articles discussed naturally (not rigid segments)
- Each article: 6-12 minutes of discussion average
- Natural transitions between topics
- Callbacks to previous episodes when relevant
- New predictions (primarily Marcus, occasionally others)
- Running jokes integrated organically
- Closing thoughts (brief, not formulaic)

4. DEBATE DYNAMICS:
- Hosts disagree on approach/interpretation but usually converge on conclusions
- Nigel plays skeptic/devil's advocate
- Marcus plays provocateur with bold takes
- Emily plays defender/optimist
- They respect each other despite disagreement style
- Conversations 100% focused on games (no personal tangents)

5. MEMORY INTEGRATION:
- Reference past predictions when relevant
- Build on previous opinions
- Continue running jokes
- Show character consistency across episodes

6. PREDICTIONS:
- Marcus should make 2-4 bold predictions per episode
- Nigel occasionally makes predictions (more measured)
- Emily rarely predicts (more reactive analysis)
- Format clearly for extraction: "I'm predicting that [specific outcome] will happen by [timeframe]"

OUTPUT FORMAT:
NIGEL: [Dialogue with natural speech]
MARCUS: [Response]
EMILY: [Reaction]
NIGEL: [INTERRUPTS] Wait, hold on—
MARCUS: [OVERLAPS] That's exactly what I'm saying!

Write the complete 90-minute script covering all selected articles.
```

---

## 6. Cost Estimation

### 6.1 Daily Research Phase (Mon-Sat)

**Perplexity API:**
- Articles per day: ~15-25 (from digest)
- Hosts researching: 3
- Total queries per day: 45-75
- Tokens per query: ~2,500 (500 input + 2,000 output)
- Total tokens per day: 112,500 - 187,500
- Cost per day: $2.25 - $3.75
- **Monthly cost (26 days): $58.50 - $97.50**

### 6.2 Weekly Script Generation (Sunday)

**OpenAI (Article Selection):**
- Input: All articles from week (~30KB) + research summaries (~100KB) = ~130KB = ~30,000 tokens
- Output: Article selection list = ~1,000 tokens
- Total: ~31,000 tokens
- Cost: $0.31 (input) + $0.02 (output) = **$0.33 per week = $1.32/month**

**Claude (Script Generation):**
- Input: Selected articles (10KB) + research (40KB) + memories (10KB) + prompt (5KB) = ~65KB = ~15,000 tokens
- Output: 90-min script = ~16,000 tokens
- Cost (Sonnet 4):
  - Input: 15,000 × $0.015/1K = $0.225
  - Output: 16,000 × $0.075/1K = $1.20
  - Total: **$1.43 per week = $5.72/month**

**OpenAI TTS:**
- Script: ~14,000 words
- Cost: $0.015 per 1K characters
- Average: 14,000 words × 5 chars/word = 70,000 characters
- Cost: 70K × $0.015/1K = **$1.05 per week = $4.20/month**

### 6.3 Total Monthly Cost Estimate

| Component | Monthly Cost |
|-----------|-------------|
| Existing News Bot | $41.00 |
| Daily Research (Perplexity) | $58.50 - $97.50 |
| Article Selection (OpenAI) | $1.32 |
| Script Generation (Claude) | $5.72 |
| Audio Production (OpenAI TTS) | $4.20 |
| **Podcast Bot Total** | **$69.74 - $108.74** |
| **Combined System Total** | **$110.74 - $149.74** |

**Notes:**
- Highly variable based on article count per day
- Conservative estimate: ~$125/month total
- Could optimize by:
  - Reducing research depth (fewer tokens per query)
  - Selecting fewer articles per episode
  - Using cheaper TTS alternatives

---

## 7. Timeline & Milestones

### Phase 1: Core Infrastructure (Week 1)
- [ ] Set up `podcast_bot.py` project structure
- [ ] Implement daily research phase
  - [ ] Digest parsing
  - [ ] Per-host research prompt generation
  - [ ] Perplexity API integration
  - [ ] Research file storage
- [ ] Test daily research on 3 days of existing digests
- [ ] **Deliverable:** Research files generating successfully

### Phase 2: Script Generation (Week 2)
- [ ] Implement weekly synthesis phase
  - [ ] Article selection logic (OpenAI)
  - [ ] Host memory loading
  - [ ] Script generation (Claude)
  - [ ] Memory update extraction
- [ ] Create character prompt templates
- [ ] Test script generation with sample articles
- [ ] **Deliverable:** First draft script

### Phase 3: Character Refinement (Week 2-3)
- [ ] Review generated script for character consistency
- [ ] Refine character personalities in prompts
- [ ] Test natural conversation flow
- [ ] Iterate on debate dynamics
- [ ] **Deliverable:** Approved script with natural dialogue

### Phase 4: Audio Production (Week 3)
- [ ] Implement OpenAI TTS integration
- [ ] Build audio stitching pipeline
- [ ] Test stage direction handling (pauses, interruptions)
- [ ] Generate first full episode audio
- [ ] **Deliverable:** First complete 90-min MP3

### Phase 5: Distribution (Week 4)
- [ ] Implement RSS feed generation
- [ ] Set up local HTTP server
- [ ] Configure Tailscale access
- [ ] Test playback on Android device
- [ ] **Deliverable:** Podcast accessible on phone

### Phase 6: Memory System (Week 4)
- [ ] Implement host memory storage
- [ ] Build prediction tracking system
- [ ] Test prediction resolution automation
- [ ] Verify memory continuity across episodes
- [ ] **Deliverable:** Memory system functional

### Phase 7: Production Deployment (Week 5)
- [ ] Set up cron jobs (daily + weekly)
- [ ] Configure all API keys
- [ ] Test full automated workflow
- [ ] Monitor first 2 automated episodes
- [ ] **Deliverable:** Fully automated podcast system

### Phase 8: Optimization (Week 6+)
- [ ] Refine character prompts based on output
- [ ] Optimize token usage for cost reduction
- [ ] Improve audio quality (voice selection, pacing)
- [ ] Fine-tune article selection logic
- [ ] **Deliverable:** Stable, cost-effective production system

---

## 8. Open Questions

### Technical
- ✅ TTS service: OpenAI (easiest API integration)
- ✅ Voice quality: Test OpenAI voices for character fit, may need alternatives
- ⚠️ Audio stitching: Need to test pacing and naturalness
- ⚠️ Cost: $125/month estimated, may need optimization

### Content
- ⚠️ Character names: Nigel/Marcus/Emily - confirm approval
- ⚠️ Script length variance: Accept 70-110 min range or enforce 90 min target?
- ⚠️ Cursing frequency: How liberal with F-bombs for Nigel? (podcast not public)
- ⚠️ Article selection: Trust AI voting or manual override?

### Operational
- ✅ Hosting: Local Mac Mini + Tailscale
- ✅ Schedule: Sunday 5am generation
- ⚠️ Monitoring: How to detect failed episodes? (email alert, Slack, check logs manually?)
- ⚠️ Backfill: Generate episodes for past weeks or start fresh?

---

## 9. Risks & Mitigations

| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| Script feels robotic/unnatural | High | Medium | Extensive prompt engineering, character refinement, iterative testing |
| TTS voices don't match characters | Medium | Medium | Test multiple TTS services, voice cloning if needed |
| Cost exceeds $150/month | Medium | Low | Monitor daily, optimize token usage, reduce research depth |
| Character personalities drift | Medium | Medium | Memory system maintains consistency, regular prompt review |
| Prediction tracking fails | Low | Low | Manual fallback, regular audit of resolved predictions |
| Audio stitching artifacts | Medium | Low | Test pacing, adjust pause durations, use audio normalization |
| Episode generation timeouts | Medium | Low | Retry logic, reduce article count if needed |

---

## 10. Future Enhancements (Post-v1)

### Phase 2 Features
- Professional TTS with voice cloning (ElevenLabs or similar)
- Guest host episodes (simulated industry figures)
- Listener questions (manual submission via form)
- Chapter markers in RSS feed
- Show notes with article links

### Phase 3 Features
- Video version (static images + audio)
- Public distribution (Spotify, Apple Podcasts)
- Community poll integration (hosts discuss poll results)
- Live episode recording simulation (chat integration)

---

## 11. Related Documents

**Extends:**
- News Bot v2.0 PRD
- News Bot Deployment Guide

**Dependencies:**
- Daily Gaming Industry Digests (input)

**New Documents Needed:**
- Character Personality Guides (detailed)
- Prompt Library (research, selection, script templates)
- TTS Voice Testing Results
- Memory System Specification

---

## 12. Approval & Sign-Off

| Role | Name | Date | Status |
|------|------|------|--------|
| Product Owner | Kyle Billings | 2026-03-01 | ⏳ Pending Review |
| Technical Review | | | ⏳ Pending |
| Character Approval | | | ⏳ Pending |
| Budget Approval | | | ⏳ Pending |

---

**Document Status:** 🔄 Draft v1.0 - Pending Approval

**Next Steps:**
1. Review and approve character personalities
2. Confirm technical approach (OpenAI TTS vs alternatives)
3. Approve estimated budget ($110-150/month)
4. Proceed to implementation (Phase 1)

[[News Bot]] | [[Podcast]] | [[Automation]] | [[AI Agents]]
