---
type: readme
status: complete
version: 1.0
date: 2026-03-01
---

# Gaming Industry Podcast Bot - Documentation Package

## 📋 What This Is

Complete implementation and deployment documentation for an automated weekly gaming industry podcast featuring three AI-powered hosts with distinct personalities, debating news from the daily Gaming Industry Digest.

**System Overview:**
- **Daily Research:** Three hosts independently research articles via Perplexity (Mon-Sat)
- **Weekly Generation:** OpenAI selects articles, Claude writes 90-min natural conversation script, OpenAI TTS produces audio (Sunday 5am)
- **Distribution:** Self-hosted RSS feed accessible via Tailscale on Android

**Hosts:**
- **Nigel Thornbury** (60s British) - Bombastic contrarian veteran
- **Marcus Webb** (30s American) - Brash data-driven predictor
- **Emily Bradford** (20s British) - Optimistic community voice

---

## 📁 Documentation Structure

### Core Documents (Read First)

#### 1. **PRD - Gaming Industry Podcast Bot.md** (60 pages)
**Purpose:** Complete product requirements document
**Contains:**
- Full system architecture and data flow
- Technical requirements and API specifications
- Character profiles and personalities
- Cost estimates ($110-150/month total)
- Timeline and milestones
- Success criteria and quality metrics

**Read this to:** Understand what the system does and why

---

#### 2. **Implementation Guide.md** (40 pages)
**Purpose:** Technical implementation details
**Contains:**
- System architecture diagram
- Complete project structure
- Core component specifications with code examples
- Testing strategy and unit tests
- Monitoring and debugging tools
- Cost tracking implementation

**Read this to:** Understand how to build the system

**Key Sections:**
- `digest_parser.py` - Parse daily digests into article objects
- `research_generator.py` - Generate character-specific research via Perplexity
- `article_selector.py` - Select 6-15 articles via OpenAI voting
- `memory_manager.py` - Track host memories, predictions, running jokes
- `script_generator.py` - Generate 90-min natural dialogue via Claude
- `audio_producer.py` - Convert script to MP3 via OpenAI TTS

---

#### 3. **Deployment Guide.md** (45 pages)
**Purpose:** Mac Mini production deployment
**Contains:**
- Prerequisites and dependencies
- Step-by-step setup instructions
- Configuration files (API keys, yaml, cron)
- Testing procedures
- Production deployment checklist
- Monitoring and maintenance procedures
- Troubleshooting guide

**Read this to:** Understand how to deploy on Mac Mini

**Key Sections:**
- API key setup (secure .env)
- Cron job configuration (daily + weekly)
- HTTP server LaunchAgent
- Tailscale network access
- Full Disk Access permissions

---

#### 4. **QUICK START - Implementation Order.md** (15 pages)
**Purpose:** Step-by-step implementation path
**Contains:**
- Week-by-week implementation plan (6-8 weeks)
- Task breakdowns with validation steps
- File reference guide
- Pre-launch checklist
- Go-live procedure

**Read this to:** Know where to start and in what order

**Timeline:**
- Week 1: Core infrastructure
- Week 2: Daily research phase
- Week 3: Article selection & memory
- Week 4: Script generation
- Week 5: Audio production
- Week 6: RSS feed & distribution
- Week 7: Automation & scheduling
- Week 8: Character refinement

---

### Character Guides (Read Second)

#### 5. **Character Guides/Nigel Thornbury - Character Guide.md**
- Personality: Bombastic, profane, contrarian
- Speech patterns and vocabulary
- Debate style and interactions
- Running jokes and catchphrases
- Character arc over episodes

#### 6. **Character Guides/Marcus Webb - Character Guide.md**
- Personality: Brash know-it-all, data-obsessed
- Prediction philosophy and style
- Buzzword usage and metrics
- Overconfident delivery style
- Prediction accuracy tracking

#### 7. **Character Guides/Emily Bradford - Character Guide.md**
- Personality: Optimistic, community-focused
- Community sentiment analysis
- Defensive of developers
- Growth and confidence building
- Deference and persistence

---

### Prompt Templates (Implementation Reference)

#### 8. **Prompt Templates/Research Prompt - Nigel.md**
- Perplexity research from contrarian veteran perspective
- Focus: Historical precedents, pattern recognition
- Output: 800-1200 words per article

#### 9. **Prompt Templates/Research Prompt - Marcus.md**
- Perplexity research from data-driven disruptor perspective
- Focus: Business models, metrics, bold predictions
- Output: 800-1200 words per article

#### 10. **Prompt Templates/Research Prompt - Emily.md**
- Perplexity research from community-focused perspective
- Focus: Player sentiment, developer intent
- Output: 700-1000 words per article

#### 11. **Prompt Templates/Article Selection Prompt.md**
- OpenAI GPT-4 prompt for selecting 6-15 articles
- Simulates host voting on discussion potential
- Outputs ordered list with rationale
- JSON format for parsing

#### 12. **Prompt Templates/Script Generation Prompt.md** (CRITICAL)
- Claude Sonnet 4 prompt for 90-minute script
- Comprehensive instructions for natural dialogue
- Character consistency guidelines
- Interruption and overlap formatting
- Prediction extraction markers
- 13,000-16,000 word target

---

## 🚀 How to Get Started

### Option A: Quick Overview (30 minutes)
1. Read this README
2. Skim PRD (focus on System Overview section)
3. Review Character Guides (personality sections)
4. Read QUICK START for implementation path

### Option B: Deep Dive (3-4 hours)
1. Read PRD completely
2. Read Implementation Guide
3. Read all Character Guides
4. Review all Prompt Templates
5. Read Deployment Guide
6. Plan implementation timeline

### Option C: Jump Right In (Start Building)
1. Read QUICK START
2. Begin Week 1: Core Infrastructure
3. Reference other docs as needed
4. Test at each stage

---

## 💰 Cost Summary

**Monthly Operating Costs:**

| Component | Cost |
|-----------|------|
| Existing News Bot | $41/month |
| Daily Research (Perplexity) | $58-98/month |
| Article Selection (OpenAI) | $1/month |
| Script Generation (Claude) | $6/month |
| Audio Production (OpenAI TTS) | $4/month |
| **Total** | **$110-150/month** |

**One-Time Costs:**
- Development time: 6-8 weeks
- No hardware costs (existing Mac Mini)
- No hosting costs (self-hosted)

---

## 📊 System Specifications

**Input:**
- Daily Gaming Industry Digests (Mon-Sat, 12pm ET)
- 15-25 articles per digest
- HIGH/MEDIUM/LOW impact levels

**Processing:**
- Daily: 3 hosts × 15-25 articles = 45-75 Perplexity queries
- Weekly: Select 6-15 best articles
- Weekly: Generate 13,000-16,000 word script
- Weekly: Produce ~90-minute MP3

**Output:**
- One 90-minute podcast episode per week
- Covers Saturday-Saturday news cycle
- RSS feed with 52-episode history
- Accessible via Tailscale on Android

**Quality Targets:**
- Natural conversation (not robotic)
- Character consistency across episodes
- Memory and continuity (callbacks, predictions)
- Appropriate length (70-110 minutes acceptable)

---

## 🎭 Character Summary

### Nigel Thornbury
"Bloody hell, not another battle royale!"
- **Age:** Late 60s
- **Voice:** British male (posh but rough)
- **Role:** Cynical veteran who's "seen it all before"
- **Strength:** Historical context and pattern recognition
- **Weakness:** Too dismissive of genuine innovation
- **Catchphrase:** "I've seen this all before"

### Marcus Webb
"I'm calling it now—this is a paradigm shift!"
- **Age:** Early 30s
- **Voice:** American male (confident, fast-paced)
- **Role:** Data-driven predictor making bold claims
- **Strength:** Business model analysis and metrics
- **Weakness:** Jumps to conclusions, lacks historical context
- **Catchphrase:** "The data is crystal clear"

### Emily Bradford
"But the community loves this!"
- **Age:** Late 20s
- **Voice:** British female (bright, enthusiastic)
- **Role:** Optimistic community voice
- **Strength:** Player sentiment and developer empathy
- **Weakness:** Sometimes too optimistic about intentions
- **Catchphrase:** "I think they're really trying here"

---

## 🎯 Implementation Priorities

### Must Have (Core Functionality)
- [x] Daily research phase working
- [x] Article selection logic
- [x] Script generation with natural dialogue
- [x] Audio production pipeline
- [x] RSS feed and distribution

### Should Have (Quality)
- [x] Character consistency
- [x] Memory system (predictions, opinions)
- [x] Prediction auto-resolution
- [x] Natural interruptions and overlaps

### Nice to Have (Future)
- [ ] Voice cloning for better character voices
- [ ] Chapter markers in RSS feed
- [ ] Show notes with article links
- [ ] Public distribution (Spotify, Apple)

---

## 📈 Success Metrics

**Operational:**
- 95%+ automated episode generation success rate
- Episode length: 70-110 minutes (target 90)
- Zero manual intervention after setup

**Quality:**
- Characters feel distinct and consistent
- Conversations sound natural (not scripted)
- Predictions tracked and resolved automatically
- Community references feel authentic

**Cost:**
- Monthly costs <$150
- Per-episode cost <$35

---

## 🛠 Technical Stack

**Languages:**
- Python 3.9+

**APIs:**
- Perplexity (Sonar Pro) - Host research
- OpenAI (GPT-4 + TTS) - Article selection + audio
- Anthropic (Claude Sonnet 4) - Script generation

**Infrastructure:**
- Mac Mini (headless, macOS, Eastern Time)
- Cron scheduling (daily + weekly)
- Python HTTP server
- Tailscale for remote access

**Libraries:**
- `openai` - API client
- `anthropic` - API client
- `requests` - Perplexity API
- `pydub` - Audio stitching
- `feedgen` - RSS generation
- `PyYAML` - Configuration

---

## 📝 Next Actions

1. **Review Documentation**
   - [ ] Read this README
   - [ ] Read PRD (understand system)
   - [ ] Read Character Guides (understand hosts)
   - [ ] Read QUICK START (understand path)

2. **Approve Approach**
   - [ ] Confirm character personalities
   - [ ] Approve technical stack
   - [ ] Accept cost estimates
   - [ ] Agree on timeline

3. **Begin Implementation**
   - [ ] Week 1: Core infrastructure setup
   - [ ] Week 2: Daily research phase
   - [ ] Continue per QUICK START plan

---

## 📞 Support

**Documentation Files:**
- All questions answered in comprehensive guides
- Character details in Character Guides
- Technical specs in Implementation Guide
- Deployment steps in Deployment Guide

**Key References:**
- PRD: Overall system design
- Implementation Guide: Code structure
- Deployment Guide: Production setup
- Prompt Templates: AI interaction details

---

## 🎉 Documentation Complete

All necessary documentation has been created for full implementation and deployment of the Gaming Industry Podcast Bot.

**Package Includes:**
- 12 comprehensive documents
- ~200 pages of specifications
- Complete prompt templates
- Character personality guides
- Week-by-week implementation plan
- Production deployment procedures

**Status:** ✅ Ready for implementation

**Estimated Implementation Time:** 6-8 weeks

**Estimated Monthly Cost:** $110-150

---

**Questions?** Review the specific documentation section that covers your area of concern.

**Ready to Build?** Start with `QUICK START - Implementation Order.md`

[[Podcast Bot]] | [[Documentation]] | [[Start Here]]
